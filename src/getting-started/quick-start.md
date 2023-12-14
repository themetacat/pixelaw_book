# Quick Start

## Deploy the Pixel Core locally
Everything in PixeLAW starts with the core pixel layer into which you will deploy your own app.

Let's get started by deploying the core pixel layer, which includes both the contracts and the PixeLAW front-end.

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

## Wait for the Core to be deployed (Optional)

For convenience, you can run the following script that will print out "Ready for app deployment", once contracts fully initialised:
```console
scarb run ready_for_deployment
```

After some time (around approx. 10 minutes) you should be able to see PixeLAW running on http://localhost:3000. There is a docker-compose file in this repository specifically for running a local image of PixeLAW core. Wait until http://localhost:3000/manifests/core stops returning NOT FOUND. 





Awesome, you just successfully build your own PixeLAW App! If you fail, please read [app_template README](https://github.com/pixelaw/app_template) to see another way to deploy. Or you can check [here](./setup_old.md).

## Next Steps

The guide above should be enough to get your app ready on local. The next step would be:
- Deploy your app to https://demo.pixelaw.xyz/ [here]({#deploy_to_demo}).
- Check out [tutorials](../tutorial) of other PixeLAW Apps.
- Check out the [PixeLAW Core](https://github.com/pixelaw/core).