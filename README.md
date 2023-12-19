# Interplay between Vite and SAP Approuter
This is one possible setup for a local development environment that involves using the SAP _Approuter_ _in combination_ with the frontend-tooling _Vite_.

The Approuter will handle the authorization and authentication details, while one can still leverage the speed and Hot Module Replacement (HMR) of Vite's development server (dev server) to develop the UI quickly.

Also, say you want to deploy the complete app later on to Business Technology Platform (BTP) in a Clond Foundry Environment, then your locally and your deployed version of the app basically have the same structure.
This is one of the main benefits of this setup—besides incorporating also authentication/authorization during local development.

Now, to use this setup when developing locally, you first start the app. After you have authenticated, the Approuter will forward any incoming request for the UI to the running Vite dev server.
This dev server returns all the necessary files to display the webpage in the browser back to the Approuter. In your browser you should now see the UI on http://localhost:5000

## Approuter Configuration
The configuration for the Approuter used locally is stored separately in the `router/dev folder`. Firstly, define a dummy destination
for local use in `router/dev/default-env.json` that points to the locally running Vite dev server.
```json
{
    "destinations": [
        {
            "name": "vite-dev-server",
            "url": "http://localhost:5173"
        }
    ]
}
```

Next, tell the Approuter to forward any incoming requests to this destination. This is done in the `router/dev/xs-app.json`.
```json
{
    "welcomeFile": "/index.html",
    "routes": [
        {
            "source": "^/(.*)$",
            "target": "/$1",
            "destination": "vite-dev-server",
            "authenticationType": "none"
        }
    ]
}
```

This configuration is only used locally. When it comes time to deploy the app later on to BTP on Cloud Foundry you can ignore it. To do that, add the line `ignore: [./dev]` to the `build-parameters` section of your
module definition inside the respective `mta.yaml` descriptor file.

## Vite Configuration

As for Vite you only need to change the port of Vite's websocket used for HMR (i.e., to 5174_ in our case). This way the web socket runs on its own separate port.
You do this inside the `ui/vite.config.ts`. For production, you can create a build as usual with the build command. Here you configure Vite to store the build output in `router/dist/`. This way, Approuter can point to the created build in productive scenario (see `router/xs-app.json` for that).
```typescript
export default defineConfig({
  plugins: [react()],
  server: {
    hmr: {
      port: 5174,
    },
  },
  build: {
    outDir: "../router/dist",
    emptyOutDir: true,
  },
});
```

## Requirements
For this setup Node version 18 was used, but any newer Node LTS version should work as well. 
You further need to install _ts-node_ and _typescript_ globally with:
```
npm install --global typescript ts-node
```

## Installation and Startup
First install all packages which are required locally by running:
```
npm run setup
```

Then you can start both the Vite dev server and the Approuter by running—while being in the root project folder:
```
npm run start
```

You should now be able to open up the application at http://localhost:5000 and see the UI there.

## Folder Structure
The `ui/` folder containing the UI was generated with the help of the frontend-tooling Vite and its `react-ts` template for React and Typescript.
The other `router/` folder is intended for everything related to the Approuter.
