# Quick Start: Build your own PixeLAW App

## Deploy the Pixel Core locally
Everything in PixeLAW starts with the core pixel layer into which you will deploy your own app.

Let's get started by both deploying the core pixel layer, and the app from the app_template locally.


## Clone app_template

Please go to [app_template](https://github.com/pixelaw/app_template). And clone it.

```console
git clone https://github.com/pixelaw/app_template.git app_template
```

## Prerequisites

Download these libraries.

-   Dojo - install [here](https://book.dojoengine.org/getting-started/quick-start.html)
-   Scarb - install [here](https://docs.swmansion.com/scarb/download)
-   Docker - install [here](https://docs.docker.com/engine/install/)
-   Docker compose plugin - install [here](https://docs.docker.com/compose/install/)

## Run your own tests

To make sure everything is working correctly, run the following command to run tests:
```console
sozo test
```

## Deploy the Pixel Core

In order to simplify deploying PixeLAW locally, we created a docker container with all dependencies. Simply run this command in the `app_template` folder:
```console
docker compose up -d
```

After some time (around approx. 10 minutes) you should be able to see PixeLAW running on http://localhost:3000. There is a docker-compose file in this repository specifically for running a local image of PixeLAW core. Wait until http://localhost:3000/manifests/core stops returning NOT FOUND. 

## Deploy your own App to the Pixel Core

Now that you have deployed the pixel core, you are ready to build and migrate your own contracts in `app_template/src`.

- `src/app.cairo` contains your apps core logic.
- `src/lib.cairo` contains a module declaration.
- `src/tests.cairo` contains your tests.

### Building your contracts

Run the following to compile the cairo contracts, generating the necessary artifacts for deployment.
```console
sozo build
```

### Migrate/Deploy your App

This will deploy your contracts to the local PixeLAW world.
```console
sozo migrate --name pixelaw
```

### Initialise your App

This will run `scripts/default_auth.sh` and provide necessary authorization and writer permission to your app.
```console
scarb run initialize
```

### Uploading your manifest

This will run `scripts/upload_manifest.sh` and upload your manifest.json required by the front-end.
```console
scarb run upload_manifest
```

Awesome, you just successfully build your own PixeLAW App! If you fail, please read [app_template README](https://github.com/pixelaw/app_template) to see another way to deploy. Or you can check [here](./setup_old.md).

## Next Steps

The guide above should be enough to get your app ready on local. The next step would be:
- Deploy your app to https://demo.pixelaw.xyz/ [here]({#deploy_to_demo}).
- Check out [tutorials](../tutorial) of other PixeLAW Apps.
- Check out the [PixeLAW Core](https://github.com/pixelaw/core).