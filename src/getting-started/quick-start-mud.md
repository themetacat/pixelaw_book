# Quick Start(For MUD)

## Deploy the Pixel Core locally
Everything in PixeLAW starts with the [core pixel layer](https://github.com/themetacat/pixelaw_core) into which you will deploy your own app.

Let's get started by deploying the core pixel layer, which includes both the contracts and the PixeLAW front-end.

## Prerequisites
- [Node.js v18](https://nodejs.org/en/download/package-manager)
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- [pnpm](https://pnpm.io/)
`
sudo npm install -g pnpm
`

## Clone repo

Please go to [pixelaw\_core\_mud](https://github.com/themetacat/pixelaw_core). And clone it.

```console
git clone https://github.com/themetacat/pixelaw_core pixelaw_core
```

## Build and deploy locally

To make sure everything is working correctly, run the following command to do all things:

```sh
cd pixelaw_core && pnpm install
```
```sh
pnpm run start
```

After some time (around 10 minute) you should be able to see PixeLAW running on [http://localhost:3000](http://localhost:3000).

You should be able to see PixeLAW in its true glory: ![PixelCore](../images/PixelCore.png)

If you run into any issues you can check out the [github repo](https://github.com/themetacat/pixelaw_app_template_mud/tree/main), and read [start.sh](https://github.com/themetacat/pixelaw_core/blob/main/start.sh) to see the build and deploy details.

## Next Step

Awesome, you just successfully deployed the Pixel Core. 

The next step should be for you to build your own PixeLAW App. We will remain in the `app_template_mud` repo.

Go and be a Pixel Builder and [deploy your own App to the core](../build-app/1-build-app-mud.md)!