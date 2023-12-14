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