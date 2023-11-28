## Write fn init()
Same as default.
```rust
fn init(self: @ContractState) {
    let world = self.world_dispatcher.read();
    let core_actions = pixelaw::core::utils::get_core_actions(world);

    core_actions.update_app(APP_KEY, APP_ICON, APP_MANIFEST);

    // TODO: replace this with proper granting of permission

    core_actions.update_permission('snake',
        Permission {
            alert: false,
            app: false,
            color: true,
            owner: false,
            text: true,
            timestamp: false,
            action: false
        });
    core_actions.update_permission('paint',
        Permission {
            alert: false,
            app: false,
            color: true,
            owner: false,
            text: true,
            timestamp: false,
            action: false
        });
}
```

### Write a draft of fn interact()

Initially, load some information
```rust
let caller_address = get_caller_address();
let caller_app = get!(world, caller_address, (App));
let mut game = get!(world, (position.x, position.y), (Game));
let timestamp = starknet::get_block_timestamp();
```

We call this `interact()` function when we click pixels. So, we use `pixel.alert` to call functions.

```rust
if pixel.alert = 'reveal' {
    // call reveal function
    self.reveal(default_params);
} else if pixel.alert = 'explode' {
    // call explode function
    self.explode(default_params);
} else if self.ownerless_space(default_params, size: size) == true {
    // declare a new game
} else {
    // we can't do anything, so we just return
    'find a free area'.print();
}
```

### Implement function when we start game
If `self.ownerless_space()` returns true, we can start new game. Let's start new game.

```rust
let mut id = world.uuid();

// set game information
game = 
    Game {
        x: position.x,
        y: position.y,
        id,
        creator: player,
        state: State::Open,
        size: size,
        mines_amount: mines_amount,
        started_timestamp: timestamp
    };


emit!(world, GameOpened {game_id: game.id, creator: player});

set!(world, (game));

let mut i: u64 = 0;
let mut j: u64 = 0;

// update pixels to set color and alert. Then, if player click the pixel, they call reveal function.
loop { 
    if i >= size {
        break;
    }
    j = 0;
    loop { 
        if j >= size {
            break;
        }
        core_actions
            .update_pixel(
            player,
            system,
            PixelUpdate {
                x: position.x + j,
                y: position.y + i,
                color: Option::Some(default_params.color), //should I pass in a color to define the minesweepers field color?
                alert: Option::Some('reveal'),
                timestamp: Option::None,
                text: Option::None,
                app: Option::Some(system),
                owner: Option::Some(player),
                action: Option::None,
                }
            );
            j += 1;
    };
    i += 1;
};
```

Then, set mines.
```rust
let mut num_mines = 0;
loop {
    if num_mines >= mines_amount {
        break;
    }
    let timestamp_felt252 = timestamp.into();
    let x_felt252 = position.x.into();
    let y_felt252 = position.y.into();
    let m_felt252 = num_mines.into();

    //random = (timestamp + i) + position.x.into() + position.y.into();

    let hash: u256 = poseidon_hash_span(array![timestamp_felt252, x_felt252, y_felt252, m_felt252].span()).into();
    random_number = hash % (size * size).into();

    core_actions
        .update_pixel(
            player,
            system,
            PixelUpdate {
                //x: (position.x + random_x),
                x: position.x + (random_number / size.into()).try_into().unwrap(),
                //y: (position.y + random_y),
                y: position.y + (random_number % size.into()).try_into().unwrap(),
                color: Option::Some(default_params.color),
                alert: Option::Some('explode'),
                timestamp: Option::None,
                text: Option::None,
                app: Option::Some(system),
                owner: Option::Some(player),
                action: Option::None,
            }
        );
    num_mines += 1;
};
```