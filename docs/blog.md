
# Interplay between Vite and SAP Approuter

Hi there folks,

not too long ago we wanted to utilize the UI5 Web Components in React. In their
documentation they recommend to use Vite—a relatively new frontend-tooling framework
—to create an initial app as a starting point based on one of its templates.

At the same time, we commonly use the SAP Approuter during development to require authorization, authentication—also locally. There was no guidance here, really, on how to
use both in combination. Eventually, after some research
and exploring the possible configuration options of Vite, we came up with our
own setup that has been working quite well for us so far. Therefore, I want to share it here.

First, I will link to the respective git repository which showcases this setup, in case you want to jump there immediately.

Now, Vite is a tooling which is used for frontend development. It focuses on speed and developer happiness and for that it leverages the fact that the latest stable versions of browsers
support Native ES Modules out of the box. Think of them as normal JavaScript Modules for simplicity. This eliminates the need for a large part of bundling.
Interestingly, during development, Vite does not seem to create a dist folder or any bundled files at all—unlike many other tooling.

As for the Approuter, it is what's called a reverse-proxy server. It acts as the central entry point to the app,
meaning it is the first thing the client's browser makes contact with. Here it forwards any incoming
requests coming from the browser, the client, to the respective service in the backend. Later on, it returns the responses
of those services back to the browser of the client.

More importantly for our scenario, it ensures you as a developer also need to authenticate and be authorized to access the app
—even when developing locally. This is useful if you need to access some data which requires authorization anyway.

The other main benefit is that the structure of the app and the authentication flow locally is basically the same as for the deployed version
of the app later on. Just imagine a scenario in which you want to have a standalone Approuter and later on deploy your app to
Business Technology Platform (BTP) on the Cloud-Foundry environment. Lastly, you can still benefit from the speed and Hot Module Replacement (HMR) of Vite's development server (dev server).

To be fair, the Approuter of course does not handle authentication/authorization completely alone, but
rather makes use of the XSUAA service (extended services for UAA, where the
latter stands for User Account and Authentication) for that. I won't go into
the details of the XSUAA, but you should by now have a basic idea as to why we wanted to
use the Approuter during local development as well.

Let's get now to the details of the setup. This following is going to be the folder structure.
The `ui/` folder containing the UI was generated with the help of the frontend-tooling Vite and its `react-ts` template for React and Typescript.
The `router/` folder is intended for everything related to the Approuter.

#img here#

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

This configuration is only used locally. When it comes time to deploy the app later on to BTP on Cloud Foundry you can ignore it. To do that, add the line `ignore: [./dev]`
to the `build-parameters` section of your module definition inside the respective `mta.yaml` descriptor file.

## Vite Configuration

As for Vite you only need to change the port of Vite's web socket used for HMR (i.e., to _5174_ in our case). This way the web socket runs on its own separate port.
You do this inside the `ui/vite.config.ts`.

For production, you can create a build as usual with the build command. Here you configure Vite to store the build output in `router/dist/`. This way, Approuter can point to the created build in productive scenario (see `router/xs-app.json` for that).
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
First install all packages which are required locally by running—while being in the root project folder:
```
npm run setup
```

Then you can start both the Vite dev server and the Approuter by running in the same folder:
```
npm run start
```

You should now be able to open up the application at http://localhost:5000 and see the UI there.

Maybe this will be useful to someone out there.

Cheers,
Julian