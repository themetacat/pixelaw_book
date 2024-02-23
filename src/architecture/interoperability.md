# Interoperability
The Core contract facilitates app-to-app communication via the Interoperability Trait:
````
use dojo::world::{IWorldDispatcher, IWorldDispatcherTrait};
use pixelaw::core::models::pixel::PixelUpdate;
use pixelaw::core::models::registry::App;
use starknet::ContractAddress;

#[starknet::interface]
trait IInteroperability<TContractState> {
  fn on_pre_update(
    self: @TContractState,
    pixel_update: PixelUpdate,
    app_caller: App,
    player_caller: ContractAddress
  );
  fn on_post_update(
    self: @TContractState,
    pixel_update: PixelUpdate,
    app_caller: App,
    player_caller: ContractAddress
  );
}
````
These functions are then called during update by the Core contract like so:
````
        fn update_pixel(
            self: @ContractState,
            for_player: ContractAddress,
            for_system: ContractAddress,
            pixel_update: PixelUpdate
        ) {
            'update_pixel'.print();
            let world = self.world_dispatcher.read();
            let mut pixel = get!(world, (pixel_update.x, pixel_update.y), (Pixel));

            assert(
                self.has_write_access(for_player, for_system, pixel, pixel_update), 'No access!'
            );

            let old_pixel_app = pixel.app;
            old_pixel_app.print();

            // pre update is done after checking if an update can be done
            if !old_pixel_app.is_zero() {
              let interoperable_app = IInteroperabilityDispatcher { contract_address: old_pixel_app };
              let app_caller = get!(world, for_system, (App));
              interoperable_app.on_pre_update(pixel_update, app_caller, for_player)
            }

            /// pixel updates are done here

            // post updates are called after the updates are done
            if !old_pixel_app.is_zero() {
              let interoperable_app = IInteroperabilityDispatcher { contract_address: old_pixel_app };
              let app_caller = get!(world, for_system, (App));
              interoperable_app.on_post_update(pixel_update, app_caller, for_player)
            }

            'update_pixel DONE'.print();
        }
````

## Implementing Interoperability
To use these functions an app just has to follow these steps:

### Step 1: Import the trait
Inside the dojo contract, import the interoperability trait
````
#[dojo::contract]
mod paint_actions {
    // put import here
    use pixelaw::core::traits::IInteroperability;
````

### Step 2: Implement the trait
Like any trait, just implement it like so:
````
#[dojo::contract]
mod paint_actions {
    // put import here
    use pixelaw::core::traits::IInteroperability;
    
    #[external(v0)] // makes sure that this can be called by the Core contract
    impl ActionsInteroperability of IInteroperability<ContractState> {
      fn on_pre_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ) {
       // put pre_update_code here
      }

      fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        // put post_update_code here
      }
    }
````

## Examples

### Snake passing through a Paint Pixel
When Snake is done passing through a pixel, it reverts the pixel back to its original pixel state. To demonstrate
interoperability, when a Snake reverts back a Paint Pixel, it gets the Paint pixel to use fade on it.

#### Snake first imports the trait
````
#[dojo::contract]
mod snake_actions {
    use pixelaw::core::traits::IInteroperability;
````

#### Snake implements the trait
````
#[external(v0)]
    impl ActionsInteroperability of IInteroperability<ContractState> {
      fn on_pre_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ) {
        // do nothing
      }

      fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        // put in code here
      }
    }
````

Snake first determines that the on_post_update is being called by the core contract:
````
    fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        let core_actions = get_core_actions(self.world_dispatcher.read());
        let core_actions_address = get_core_actions_address(self.world_dispatcher.read());
        assert(core_actions_address == get_caller_address(), 'caller is not core_actions');
      }
````

Next it makes sure that this is indeed a reversal of pixel state, and it's been called by the Snake Contract:
````
    fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        let core_actions = get_core_actions(self.world_dispatcher.read());
        let core_actions_address = get_core_actions_address(self.world_dispatcher.read());
        assert(core_actions_address == get_caller_address(), 'caller is not core_actions');
        
        // checks if this is a pixel state reversal called by the Snake Contract
        if pixel_update.app.is_some() && app_caller.system == get_contract_address() {
        
        }
      }
````

Then, Snake needs to check if it's reverting back to a Paint pixel
````
    fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        let core_actions = get_core_actions(self.world_dispatcher.read());
        let core_actions_address = get_core_actions_address(self.world_dispatcher.read());
        assert(core_actions_address == get_caller_address(), 'caller is not core_actions');
        
        // checks if this is a pixel state reversal called by the Snake Contract
        if pixel_update.app.is_some() && app_caller.system == get_contract_address() {
          let old_app = pixel_update.app.unwrap();
          let world = self.world_dispatcher.read();
          let old_app = get!(world, old_app, (App));
          
          // is this reverting back to paint
          if old_app.name == 'paint' {
          
          }
        }
      }
````

Lastly, it calls the paint contract to let it fade
````
    fn on_post_update(
        self: @ContractState,
        pixel_update: PixelUpdate,
        app_caller: App,
        player_caller: ContractAddress
      ){
        let core_actions = get_core_actions(self.world_dispatcher.read());
        let core_actions_address = get_core_actions_address(self.world_dispatcher.read());
        assert(core_actions_address == get_caller_address(), 'caller is not core_actions');
        
        // checks if this is a pixel state reversal called by the Snake Contract
        if pixel_update.app.is_some() && app_caller.system == get_contract_address() {
          let old_app = pixel_update.app.unwrap();
          let world = self.world_dispatcher.read();
          let old_app = get!(world, old_app, (App));
          
          // is this reverting back to paint
          if old_app.name == 'paint' {
            
            // creating fade params
            let mut calldata: Array<felt252> = ArrayTrait::new();
            let pixel = get!(world, (pixel_update.x, pixel_update.y), (Pixel));
            calldata.append(pixel.owner.into());
            calldata.append(old_app.system.into());
            calldata.append(pixel_update.x.into());
            calldata.append(pixel_update.y.into());
            calldata.append(pixel_update.color.unwrap().into());
            
            // 0x89ce6748d77414b79f2312bb20f6e67d3aa4a9430933a0f461fedc92983084 is fade as a selector
            starknet::call_contract_syscall(old_app.system, 0x89ce6748d77414b79f2312bb20f6e67d3aa4a9430933a0f461fedc92983084, calldata.span());
          }
        }
      }
````
