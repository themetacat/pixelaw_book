# Set up Environment(Local Development)

This is a tutorial for local development. To deploy your app on Demo, please contact us on Discord.

To create your own application on PixeLAW, you must set up [PixeLAW Core](https://github.com/pixelaw/core) environ,ent locally.


## Prerequisites
-   Rust - install [here](https://www.rust-lang.org/tools/install)
-   Cairo language server - install [here](https://book.dojoengine.org/development/setup.html#3-setup-cairo-vscode-extension)
-   Dojo - install [here](https://book.dojoengine.org/getting-started/quick-start.html)
-   Scarb - install [here](https://docs.swmansion.com/scarb/download)
-   NodeJS - install [here](https://nodejs.org/en/download)
-   Docker - install [here](https://docs.docker.com/engine/install/)
-   Docker compose plugin - install [here](https://docs.docker.com/compose/install/)


## Clone app_template

### Option 1: Via Sozo
Run sozo init. This will initialize the project by cloning the repo.
````console
sozo init <YOUR-APP> pixelaw/app_template
````

### Option 2: Via Github
Use [this template](https://github.com/pixelaw/app_template) to create a new repository or clone this repository locally.
````console
git clone https://github.com/pixelaw/app_template <YOUR-APP>
````


## Set up PixeLAW/Core

### Option 1: Set up with Docker Compose

Running the following command will run core in a container.
````console
docker compose up -d
````

### Option 2: Set up with Docker

The following script will create the docker network for the container to run in:
````console
docker network create --driver bridge pixelaw
````
And this will run the actual container in http://localhost:3000:
````console
docker run -d --name pixelaw-core -p 5050:5050 -p 3000:3000 -p 8080:8080 -p 50051 --restart unless-stopped --network pixelaw oostvoort/pixelaw-core:v0.0.7
````


### Option 3: Set up in Manual way
This will take the most time. This entails cloning the PixeLAW core repos and running each individual component
manually. Refer to the [GitHub repository](https://github.com/pixelaw/core) for how to run it.