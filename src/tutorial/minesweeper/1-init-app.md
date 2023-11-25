# Deploying default app locally

As a deafault, there is a paint game contarct.

Following these instructions, you can deploy your first app on your local PixeLAW.

## Deploying locally

### Check response

Whichever way you've chosen to start up PixeLAW core, it will take a while for all the core contracts to get deployed
and start running. Wait until http://localhost:3000/manifests/core stops returning NOT FOUND. To check if you can
start deploying your app, use the following script (this will print out "Ready for app deployment" when the core
contracts have finished initialization):

````console
scarb run ready_for_deployment
````

### Test your contract
Before building your smart contracts, you should test them.
````console
sozo test
````

As a current issue, there might be the following problem due to Dojo. Just ignore it.
- `test myapp::tests::tests::test_myapp_actions ... fail (gas usage est.: 1044740)
  failures:
  myapp::tests::tests::test_myapp_actions - panicked with [6445855526543742234424738320591137923774065490617582916 ('CLASS_HASH_NOT_DECLARED'), 23583600924385842957889778338389964899652 ('ENTRYPOINT_FAILED'), 23583600924385842957889778338389964899652 ('ENTRYPOINT_FAILED'), ]`


### Build your contracts
````console
sozo build
````

### Deploy your contracts
This will deploy your app to the local PixeLAW using sozo migrate.
````console
sozo migrate --name pixelaw
````

### Initializing your contracts
This will grant writer permission to your app for any custom models made.
````console
scarb run initialize
````

For zsh users, 
````console
scarb run initialize_zsh
````

### Uploading your manifest
````console
scarb run upload_manifest
````

For zsh users,
````console
scarb run upload_manifest_zsh
````

## Deploying to Demo
Deploying to demo is almost the same as local development. The only difference is needing
the RPC_URL of the Demo environment as well as the DEMO_URL (NOTE: this must end in a slash i.e. '/')
of the Demo App to upload your manifest to. Both URLs can be provided by us. Please reach out through [Discord](https://t.co/jKDjNbFdZ5).


### Build your contracts
````console
sozo build
````

### Deploy your contracts
This will deploy your app to the local PixeLAW using sozo migrate.
````console
sozo migrate --name pixelaw --rpc-url <replace-this-with-provided-rpc-url>
````

### Initializing your contracts
This will grant writer permission to your app for any custom models made.
````console
scarb run initialize <replace-this-with-provided-rpc-url>
````

### Uploading your manifest
````console
scarb run upload_manifest <replace-this-with-provided-demo-url>
````