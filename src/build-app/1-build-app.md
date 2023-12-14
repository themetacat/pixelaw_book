# Build your first Pixel App

> **_REQUIRED:_** To create your own application in PixeLAW, you must have [set up the PixeLAW Core](../getting-started/quick-start.md) environment locally.

## PixeLAW in 15 minutes

Lets check out the folder structure first of the [App Template](https://github.com/pixelaw/app_template/tree/main):

- **`app_template/src` contains all your cairo contracts.**
 - `src/app.cairo` contains your apps core logic.
 - `src/lib.cairo` contains a module declaration.
 - `src/tests.cairo` contains your tests.
- `app_template/scripts` contains scripts for deployment.
 - `scripts/default_auth.sh` provides necessary authorization and writer permission to your app.
 - `scripts/upload_manifest.sh` upload your manifest.json required by the front-end
 - `scripts/ready_for_deployment.sh` helper script used to notify final docker deployment.
- `app_template/scarb.toml` specifies all dependencies.
- `app_template/docker-compose.yml` required for docker compose.

The default App Template includes the contract code of the Paint App which allows users to paint any pixel with any color.

You can either directly [deploy it to the Core](2-deploy-app.md) or adjust the code as you see fit.

Let's dive into the code of the [App Template](https://github.com/pixelaw/app_template/tree/main).

### Imports

At first we require certain imports from the core to enable our paint app. Depending on your use case you might require different imports. Refer to the [Pixel Core](https://github.com/pixelaw/core), other [App examples](https://github.com/pixelaw/examples) or talk to us on [Discord](https://t.co/jKDjNbFdZ5).

```rust,ignore
use dojo::world::{IWorldDispatcher, IWorldDispatcherTrait};
use pixelaw::core::models::pixel::{Pixel, PixelUpdate};
use pixelaw::core::utils::{get_core_actions, Direction, Position, DefaultParameters};
use starknet::{get_caller_address, get_contract_address, get_execution_info, ContractAddress};
```

### Interface

In the interface we prototype our functions and expose them publicly. **Every PixeLAW App requires the init and interact function.** The init functions adds your App to the world registry, whereas the interact function defines the default functionality of your app. The core will by default call the interact function of your app.

The DefaultParameters parameter is a convention we use to input all pixel-related information to the function.

```rust,ignore
#[starknet::interface]
trait IMyAppActions<TContractState> {
    fn init(self: @TContractState);
    fn interact(self: @TContractState, default_params: DefaultParameters);
}
```

### Declaring your App

1. The **APP_KEY** is the unique username of your app, and has to be the same across the entire platform.

2. The **APP_ICON** will define the icon shown on the front-end to select your app.

3. The **APP_MANIFEST** simply has to be adjusted according to your APP_KEY.

```rust,ignore
/// APP_KEY must be unique across the entire platform
const APP_KEY: felt252 = 'myapp';
/// Core only supports unicode icons for now
const APP_ICON: felt252 = 'U+263A';
/// prefixing with BASE means using the server's default manifest.json handler
const APP_MANIFEST: felt252 = 'BASE/manifests/myapp';
```

### App Contract

```rust,ignore
#[dojo::contract]
///PixeLAW Naming Convention: contracts must be named as such (APP_KEY + underscore + "actions")
mod myapp_actions {
    use starknet::{
        get_tx_info, get_caller_address, get_contract_address, get_execution_info, ContractAddress
    };
    use super::IMyAppActions;
    use pixelaw::core::models::pixel::{Pixel, PixelUpdate};

    use pixelaw::core::models::permissions::{Permission};
    use pixelaw::core::actions::{
        IActionsDispatcher as ICoreActionsDispatcher,
        IActionsDispatcherTrait as ICoreActionsDispatcherTrait
    };
    use super::{APP_KEY, APP_ICON, APP_MANIFEST};
    use pixelaw::core::utils::{get_core_actions, Direction, Position, DefaultParameters};

    use debug::PrintTrait;
```
Below we continue with function implementations.

As mentioned above, the `init` and `interact` function are required for any PixeLAW App.

Additionally, we provide the permission to another app called Snake to interact with any Pixel occupied by our app.

```rust,ignore
    // impl: implement functions specified in trait
    #[external(v0)]
    impl ActionsImpl of IMyAppActions<ContractState> {
        /// Initialize the MyApp App
        fn init(self: @ContractState) {
            let world = self.world_dispatcher.read();
            let core_actions = pixelaw::core::utils::get_core_actions(world);

            //add app to the world registry
            core_actions.update_app(APP_KEY, APP_ICON, APP_MANIFEST);
            //Grant permission to the snake App
            core_actions
                .update_permission(
                    'snake',
                    Permission {
                        app: false,
                        color: true,
                        owner: false,
                        text: true,
                        timestamp: false,
                        action: false
                    }
                );
        }
```
Now that we get to the interact function, which is called by default by the front end unless otherwise specified. 

Most importantly it calls the `core_actions.update_pixel` to change the color of a pixel that has been clicked..

```rust,ignore
        /// Put color on a certain position
        ///
        /// # Arguments
        ///
        /// * `position` - Position of the pixel.
        /// * `new_color` - Color to set the pixel to.
        fn interact(self: @ContractState, default_params: DefaultParameters) {
            'put_color'.print();

            // Load important variables
            let world = self.world_dispatcher.read();
            let core_actions = get_core_actions(world);
            let position = default_params.position;
            let player = core_actions.get_player_address(default_params.for_player);
            let system = core_actions.get_system_address(default_params.for_system);

            // Load the Pixel
            let mut pixel = get!(world, (position.x, position.y), (Pixel));

            let COOLDOWN_SECS = 5;

            // Check if 5 seconds have passed or if the sender is the owner
            assert(
            pixel.owner.is_zero() || (pixel.owner) == player || starknet::get_block_timestamp()
            - pixel.timestamp < COOLDOWN_SECS,
            'Cooldown not over'
            );

            // We can now update color of the pixel
            core_actions
                .update_pixel(
                    player,
                    system,
                    PixelUpdate {
                        x: position.x,
                        y: position.y,
                        color: Option::Some(default_params.color),
                        timestamp: Option::None,
                        text: Option::None,
                        app: Option::Some(system),
                        owner: Option::Some(player),
                        action: Option::None // Not using this feature for myapp
                    }
                );
            'put_color DONE'.print();
        }
    }
}
```

The above specifies the same functionality like our Paint App. Feel free to [deploy it locally](2-deploy-app.md), make changes and try it out.

## Next Steps

The guide above should get you familiar with how the Paint App is structured. The next step would be to:
- [Deploy your app](2-deploy-app.md) to the front-end.
- Check out other [tutorials](../tutorial) of other PixeLAW Apps.
- Check out the [PixeLAW Core](https://github.com/pixelaw/core).