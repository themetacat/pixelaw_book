# Build your first Pixel App

> **_REQUIRED:_** To create your own application in PixeLAW, you must have [set up the PixeLAW Core](../getting-started/quick-start-mud.md) environment locally.

## PixeLAW in 15 minutes

Lets check out the folder structure first of the [App Template](https://github.com/themetacat/pixelaw_app_template_mud):

- **`contracts/` contains all your solidity contracts and scripts for deployment**
  - `src/` contains your app's codegen and logic.
    - `codegen/` contains your app's store codegen and contracts interface.
    - `core_codegen/` contains PixeLAW Core's store codegen and contracts interface.
    - `systems/` contains your app's logic contract.
  - `test/` contains your app's test contract.
  - `script/` contains your app's extension contract.
  - `scripts/` contains scripts for deployment.
    - `deploy.sh` deploy your app contract and init your app.
    - `upload_json.sh` upload your abi to github.
  - `mud.config.ts` your app's table and namespace config.
  - `.env` contains the values required to deploy the MUD and PixeLAW core

The default App Template includes the contract code of Paint App which allows users to paint any pixel with any color.

You can either directly [deploy it to the Core](2-deploy-app-mud.md) or adjust the code as you see fit.

Let's dive into the code of the [App Template](https://github.com/themetacat/pixelaw_app_template_mud/tree/main).

### Imports

At first we require certain imports from the core to enable our paint app. Depending on your use case you might require different imports. Refer to the [PixeLAW Core](https://github.com/themetacat/pixelaw_core),

```solidity
import { System } from "@latticexyz/world/src/System.sol";
import { ICoreSystem } from "../core_codegen/world/ICoreSystem.sol";
import { PermissionsData, DefaultParameters, Position, PixelUpdateData, Pixel, PixelData, TestParameters } from "../core_codegen/index.sol";
```

### Declaring your App

1. The **APP_ICON** will define the icon shown on the front-end to select your app.


2. The **NAMESPACE** is the namespace you set in mud.config.ts, which is the namespace after the current app contract is deployed.

3. The **SYSTEM_NAME** is the system name set for the current contract in systems of mud.config.ts

4. The **APP_NAME** is the unique username of your app, and has to be the same across the entire platform.

5. The **APP_MANIFEST** simply has to be adjusted according to your APP_NAME.

```solidity
// Core only supports unicode icons for now
string constant APP_ICON = 'U+1F58B';

// The NAMESPACE and SYSTEM_NAME of the current contract in mudConfig
string constant NAMESPACE = 'myapp';
string constant SYSTEM_NAME = 'MyAppSystem';

// APP_NAME must be unique across the entire platform
string constant APP_NAME = 'myapp';

// prefixing with BASE means using the server's default abi.json handler, the following is consistent with the current contract name.
string constant APP_MANIFEST = 'BASE/MyAppSystem';
```

### App Contract
The `init` and `interact` function are required for any PixeLAW App.

Additionally, we provide the permission to another app called Snake to interact with any Pixel occupied by our app.
```solidity
function init() public {
    
    // init my app
    ICoreSystem(_world()).update_app(APP_NAME, APP_ICON, APP_MANIFEST, NAMESPACE, SYSTEM_NAME);

    // Grant permission to the snake App
    ICoreSystem(_world()).update_permission("snake", 
    PermissionsData({
      app: true, color: true, owner: true, text: true, timestamp: false, action: false
      })); 
  }

```
Now that we get to the interact function, which is called by default by the front end unless otherwise specified. 

Most importantly it calls the `ICoreSystem(_world()).update_pixel` to change the color of a pixel that has been clicked.

When calling update_pixel, if you do not want to set a value for one of the parameters or change the original value of the pixel, please do this:

If the parameter type is address, please pass in address(1),
If the parameter type is string, please pass in "_Null"
This will automatically skip the permission check and assignment of the parameter.

```solidity
  //Put color on a certain position
  // Arguments
  //`position` - Position of the pixel.
  //`new_color` - Color to set the pixel to.
  function interact(DefaultParameters memory default_parameters) public {
    // Load important variables
    Position memory position = default_parameters.position;
    address player = default_parameters.for_player;
    string memory app = default_parameters.for_app;

    // Load the Pixel
    PixelData memory pixel = Pixel.get(position.x, position.y);

    // TODO: Load MyApp App Settings like the fade steptime
    // For example for the Cooldown feature
    uint256 COOLDOWN_SECS = 5;

    // Check if 5 seconds have passed or if the sender is the owner
    require(pixel.owner == address(0) || pixel.owner == player || block.timestamp - pixel.timestamp < COOLDOWN_SECS, 'Cooldown not over');

    // We can now update color of the pixel

    // If you don't want to assign a value of type address(like owner), you should pass in address(1)
    // If you don't want to assign a value of type string(like app、color、text...), you should pass in "_Null"
    ICoreSystem(_world()).update_pixel(
      PixelUpdateData({
        x: position.x,
        y: position.y,
        color: default_parameters.color,
        timestamp: 0,
        text: "_Null",
        app: app,
        owner: player,
        action: "_Null"
      }));
  }
  
```

The above specifies the same functionality like our Paint App. Feel free to [deploy it locally](./2-deploy-app-mud.md), make changes and try it out.

## Next Steps

The guide above should get you familiar with how the Paint App is structured. The next step would be to:
- [Deploy your app](./2-deploy-app-mud.md) to the front-end.
- Check out other [tutorials](../how-to-play/tutorials.md) of other PixeLAW Apps.
- Check out the [PixeLAW Core](https://github.com/themetacat/pixelaw_core/tree/main).