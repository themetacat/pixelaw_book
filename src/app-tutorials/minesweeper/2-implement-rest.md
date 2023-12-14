## Implement `reveal()`

```rust,ignore
/// Reveal a pixel on a certain position
///
/// # Arguments
/// default_params: Default parameters for the action
fn reveal(self: @ContractState, default_params: DefaultParameters) {
    let world = self.world_dispatcher.read();
    let core_actions = get_core_actions(world);
    let position = default_params.position;
    let player = core_actions.get_player_address(default_params.for_player);
    let system = core_actions.get_system_address(default_params.for_system);
    let mut pixel = get!(world, (position.x, position.y), (Pixel));

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
                text: Option::Some('U+1F30E'),
                app: Option::None,
                owner: Option::None,
                action: Option::None,
            }
        );
}
```

## Implement `explode()`
```rust,ignore
/// Explode a pixel on a certain position
///
/// # Arguments
/// default_params: Default parameters for the action
fn explode(self: @ContractState, default_params: DefaultParameters) {
    let world = self.world_dispatcher.read();
    let core_actions = get_core_actions(world);
    let position = default_params.position;
    let player = core_actions.get_player_address(default_params.for_player);
    let system = core_actions.get_system_address(default_params.for_system);
    let mut pixel = get!(world, (position.x, position.y), (Pixel));

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
                text: Option::Some('U+1F4A3'),
                app: Option::None,
                owner: Option::None,
                action: Option::None,
            }
        );
}
```

## Implement `owner_less()`
```rust,ignore
fn ownerless_space(self: @ContractState, default_params: DefaultParameters, size: u64) -> bool {
    let world = self.world_dispatcher.read();
    let core_actions = get_core_actions(world);
    let position = default_params.position;
    let mut pixel = get!(world, (position.x, position.y), (Pixel));

    let mut i: u64 = 0;
    let mut j: u64 = 0;
    let mut check_test: bool = true;

    let check = loop {
        if !(pixel.owner.is_zero() && i <= size)
        {
            break false;
        }
        pixel = get!(world, (position.x, (position.y + i)), (Pixel));
        j = 0;
        loop {
            if !(pixel.owner.is_zero() && j <= size)
            {
                break false;
            }
            pixel = get!(world, ((position.x + j), position.y), (Pixel));
            j += 1;
        };
        i += 1;
        break true;
    };
    check
}
```

## Test
Test code is gonna be like this:
```rust,ignore
fn test_create_minefield() {
    // Deploy everything
    let (world, core_actions, minesweeper_actions) = deploy_world();

    core_actions.init();
    minesweeper_actions.init();

    // Impersonate player
    let player = starknet::contract_address_const::<0x1337>();
    starknet::testing::set_account_contract_address(player);

    //computer variables
    let size: u64 = 5;
    let mines_amount: u64 = 10;

    // Player creates minefield
    minesweeper_actions
        .interact(
            DefaultParameters {
                for_player: Zeroable::zero(),
                for_system: Zeroable::zero(),
                position: Position { x: 1, y: 1 },
                color: 0
            },
            size,
            mines_amount
        );
}
```


Please check your implementation so far
```console
sozo test
```

## Deploy
After this, please deploy by following [this](../setup-deploy/README.md)