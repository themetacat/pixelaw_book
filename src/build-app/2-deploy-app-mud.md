# Deploy your app locally

## Building your contracts
Before deploy your smart contracts, you should run the following to compile the solidity contracts.
Make sure you are currently in the `pixelaw_app_template_mud/contracts/`
````sh
pnpm mud build
````

## Deploy/Update your App:
This will deploy your contracts to the local PixeLAW world and init your app.
##### If the app contract is deployed for the first time: 
```sh
pnpm run deploy
```

##### If you want to update a deployed app：
First comment out the registerNamespace and registerFunctionSelector parts in ./script/MyAppExtension.s.sol:
```solidity
// world.registerNamespace(namespaceResource);

// world.registerFunctionSelector(systemResource, "init()");
// world.registerFunctionSelector(systemResource, "interact((address,string,(uint32,uint32),string))");
```
then:
```sh
pnpm run deploy INIT=false
```

#### Upload your ABI JSON:
```sh
pnpm run upload
```
Awesome, you just successfully deployed your own PixeLAW App! If you fail, please read [app_template README](https://github.com/themetacat/pixelaw_app_template_mud/tree/main) to see another way to deploy.

### Deploying to Demo

#### Building your contracts:
```sh
pnpm mud build
```

#### Deploy/Update your App:
##### If the app contract is deployed for the first time: 
```sh
pnpm run deploy RPC_URL=<replace-this-with-provided-rpc-url> CHAIN_ID=<replace-this-with-chain-id>
```

##### If you want to update a deployed app：
First comment out the registerNamespace and registerFunctionSelector parts in ./script/MyAppExtension.s.sol:
```solidity
// world.registerNamespace(namespaceResource);

// world.registerFunctionSelector(systemResource, "init()");
// world.registerFunctionSelector(systemResource, "interact((address,string,(uint32,uint32),string))");
```
then:
```sh
pnpm run deploy INIT=false RPC_URL=<replace-this-with-provided-rpc-url> CHAIN_ID=<replace-this-with-chain-id>
```

#### Upload your ABI JSON:
```sh
pnpm run upload
```

## Command
### pnpm run deploy
##### if you set RPC_URL, you should set CHAIN_ID
```sh
pnpm run deploy
    INIT if INIT=false,update the system, default true
    RPC_URL, default http://127.0.0.1:8545
    CHAIN_ID, default 31337
```