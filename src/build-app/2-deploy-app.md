# Deploy your app locally

## Test your contract
Before building your smart contracts, you should test them.
````console
sozo test
````

There might be the following problem due to an issue with Dojo. Just ignore it.
- `test myapp::tests::tests::test_myapp_actions ... fail (gas usage est.: 1044740)
  failures:
  myapp::tests::tests::test_myapp_actions - panicked with [6445855526543742234424738320591137923774065490617582916 ('CLASS_HASH_NOT_DECLARED'), 23583600924385842957889778338389964899652 ('ENTRYPOINT_FAILED'), 23583600924385842957889778338389964899652 ('ENTRYPOINT_FAILED'), ]`


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

For zsh users, 
````console
scarb run initialize_zsh
````

Awesome, you just successfully deployed your own PixeLAW App! If you fail, please read [app_template README](https://github.com/pixelaw/app_template) to see another way to deploy.

# Deploying your app to PixeLAW
Deploying to https://demo.pixelaw.xyz/ is almost the same process as developing in your local development. The only difference is that you will require the RPC_URL of the Demo environment. For that, please reach out through [Discord](https://t.co/jKDjNbFdZ5).


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
scarb run upload_manifest http://demo.pixelaw.xyz/manifests/ 
````