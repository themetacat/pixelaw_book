# Set up local environment

## Clone app_template

Please go to [app_template](https://github.com/pixelaw/app_template). And clone it.

```bash
$ git clone git@github.com:pixelaw/app_template.git
```

## Prerequisites

Download these libraries.

-   Rust - install [here](https://www.rust-lang.org/tools/install)
-   Cairo language server - install [here](https://book.dojoengine.org/development/setup.html#3-setup-cairo-vscode-extension)
-   Dojo - install [here](https://book.dojoengine.org/getting-started/quick-start.html)
-   Scarb - install [here](https://docs.swmansion.com/scarb/download)
-   NodeJS - install [here](https://nodejs.org/en/download)
-   Docker - install [here](https://docs.docker.com/engine/install/)
-   Docker compose plugin - install [here](https://docs.docker.com/compose/install/)

## Developing Locally

Easiest way to deploy PixeLAW is just running this command in `app_template` folder:
```bash
$ docker compose up -d
```

If you fail, please read [app_template README](https://github.com/pixelaw/app_template) to see another way to deploy. Or you can check [here](./setup_old.md).