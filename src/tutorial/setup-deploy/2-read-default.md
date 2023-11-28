## Understand default app
Here, we dive into the default app_template.


### Interface
We need interface. As a default, we use this.

- init: the contract initialize the states and settings.
- interace: the contract interact with cores system.
- fade: original function for app_template/paint game. If you want to create your specific function, please implement like this.

```rust,ignore
#[starknet::interface]
trait IMyAppActions<TContractState> {
    fn init(self: @TContractState);
    fn interact(self: @TContractState, default_params: DefaultParameters);
    fn fade(self: @TContractState, default_params: DefaultParameters);
}
```

As a default parameters, we use this for now.
```rust,ignore
#[derive(Copy, Drop, Serde, SerdeLen)]
struct DefaultParameters {
    for_player: ContractAddress,
    for_system: ContractAddress,
    position: Position,
    color: u32
}
```

### Init function

We need to call `update_app` for initializing application.
```rust,ignore
core_actions.update_app(APP_KEY, APP_ICON, APP_MANIFEST)
```

Also, we can set the permissions for another applications.

```rust,ignore
core_actions
    .update_permission(
        'snake',
        Permission {
            alert: false,
            app: false,
            color: true,
            owner: false,
            text: true,
            timestamp: false,
            action: false
        }
    );
```

### Interact function

By calling `core_actions.update_pixel()`, we can update pixels wiht our logic.

```rust,ignore
// logic or assertion
core_actions
    .update_pixel(
        player,
        system,
        PixelUpdate {
            x: position.x,
            y: position.y,
            color: Option::Some(default_params.color),
            alert: Option::None,
            timestamp: Option::None,
            text: Option::None,
            app: Option::Some(system),
            owner: Option::Some(player),
            action: Option::None // Not using this feature for myapp
        }
    );
```

### Fade function

As same as interact function, we can update pixels.

Here, we use `core_actions.schedule_queue()` instead of `core_actions.update_pixels()` to use queue system.
```rust,ignore
core_actions
    .schedule_queue(
        queue_timestamp, // When to fade next
        THIS_CONTRACT_ADDRESS, // This contract address
        get_execution_info().unbox().entry_point_selector, // This selector
        calldata.span() // The calldata prepared
    );
```

We use queue_timestamp to handle the time when the queued transaction is executed.
```rust,ignore
let queue_timestamp = starknet::get_block_timestamp() + FADE_SECONDS;
```
