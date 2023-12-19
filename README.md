# Interplay between Vite and SAP Approuter
This is a setup for a local development environment that uses the _SAP Approuter_ _in combination_ with the frontend-tooling _Vite_.

The Approuter will handle the authorization and authentication details, while one can still leverage the speed and Hot Module Replacement (HMR) of Vite's development server (dev server) to develop the User Interface (UI) quickly.

Say you prefer a standalone Approuter and you want to deploy the app later on to Business Technology Platform (BTP) in a Cloud Foundry Environment,
your local version of the app will have basically the same structure as your deployed version of the app.

Besides it is beneficial to require authentication and authorization locally—especially if any backend data requires it anyway.

The basic idea is the following. Let's say you have started the app and authenticated successfully. The Approuter will then forward the
browsers incoming request for the UI to the running Vite dev server. The latter returns the needed files to display the UI back to the Approuter—that in 
turn returns them back to the browser client. In other words the Approuter acts as a reverse-proxy here. If you now make changes to your UI code—because
the HMR is still working—your changes will reflect immediately in the webpage shown to you in the browser. This makes for a nice development experience
overall, while still incorporating authentication and authorization at the same time.

## Approuter Configuration
The configuration of the Approuter, which is going t obe applied during local development, is stored in its own folder in `router/dev`.
Here, define a "dummy" destination for local use in `router/dev/default-env.json` that points to the Vite dev server, that is also running locally.
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

## Vite Configuration

Change the port of Vite's web socket used for HMR, i.e. to _5174_ in our case, inside the `ui/vite.config.ts`. This way the web socket will
run on its own separate port. Otherwise, you will get an error at runtime and the HMR won't work.
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

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

## Installation and Startup
For this setup Node version 18 was used, but any newer Node LTS version should work as well. 
You further need to install _ts-node_ and _typescript_ globally with:
```
npm install --global typescript ts-node
```

Next, install the needed local packages by running the following command —while being in the root project folder:
```
npm run setup
```

Lastly, to start both the Vite dev server and the Approuter, run this command in the same folder:
```
npm run start
```

You should now be able to open up the application at http://localhost:5000 and see the UI there.

## Folder Structure
The `ui/` folder with the UI was generated with Vite as the underlying frontend-tooling. The `react-ts` template was used for React and Typescript.

Inside the `router/` folder you will find everything related to the Approuter,
including the config applied during local development (see `router/dev`).