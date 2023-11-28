## Write fn init()
Same as default.
```rust,ignore
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
```rust,ignore
let caller_address = get_caller_address();
let mut game = get!(world, (position.x, position.y), (Game));
let timestamp = starknet::get_block_timestamp();
```

We call this `interact()` function when we click pixels. So, we use `pixel.alert` to call functions.

```rust,ignore
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

```rust,ignore
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
```rust,ignore
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

### Whole code in `minesweeper_actions` is like this
```rust,ignore
#[dojo::contract]
/// contracts must be named as such (APP_KEY + underscore + "actions")
mod minesweeper_actions {
    use starknet::{
        get_tx_info, get_caller_address, get_contract_address, get_execution_info, ContractAddress
    };

    use super::IMinesweeperActions;
    use pixelaw::core::models::pixel::{Pixel, PixelUpdate};

    use pixelaw::core::models::permissions::{Permission};
    use pixelaw::core::actions::{
        IActionsDispatcher as ICoreActionsDispatcher,
        IActionsDispatcherTrait as ICoreActionsDispatcherTrait
    };
    use super::{Game, State};
    use super::{APP_KEY, APP_ICON, APP_MANIFEST, GAME_MAX_DURATION};
    use pixelaw::core::utils::{get_core_actions, Direction, Position, DefaultParameters};

    use debug::PrintTrait;

    use poseidon::poseidon_hash_span;

    #[derive(Drop, starknet::Event)]
    struct GameOpened {
        game_id: u32,
        creator: ContractAddress
    }

    #[event]
    #[derive(Drop, starknet::Event)]
    enum Event {
        GameOpened: GameOpened
    }


    fn subu8(nr: u8, sub: u8) -> u8 {
        if nr >= sub {
            return nr - sub;
        } else {
            return 0;
        }
    }


    // ARGB
    // 0xFF FF FF FF
    // empty: 0x 00 00 00 00
    // normal color: 0x FF FF FF FF

    fn encode_color(r: u8, g: u8, b: u8) -> u32 {
        (r.into() * 0x10000) + (g.into() * 0x100) + b.into()
    }

    fn decode_color(color: u32) -> (u8, u8, u8) {
        let r = (color / 0x10000);
        let g = (color / 0x100) & 0xff;
        let b = color & 0xff;

        (r.try_into().unwrap(), g.try_into().unwrap(), b.try_into().unwrap())
    }

    // impl: implement functions specified in trait
    #[external(v0)]
    impl ActionsImpl of IMinesweeperActions<ContractState> {
        /// Initialize the MyApp App (TODO I think, do we need this??)
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


        /// Put color on a certain position
        ///
        /// # Arguments
        ///
        /// default_params: Default parameters for the action
        /// size: Size of the board
        /// mines_amount: Amount of mines to place
        fn interact(self: @ContractState, default_params: DefaultParameters, size: u64, mines_amount: u64) {
            'put_color'.print();

            // Load important variables
            let world = self.world_dispatcher.read();
            let core_actions = get_core_actions(world);
            let position = default_params.position;
            let player = core_actions.get_player_address(default_params.for_player);
            let system = core_actions.get_system_address(default_params.for_system);

            // Load the Pixel
            let mut pixel = get!(world, (position.x, position.y), (Pixel));

            let caller_address = get_caller_address();
            let mut game = get!(world, (position.x, position.y), (Game));
            let timestamp = starknet::get_block_timestamp();

            if (pixel.alert == 'reveal') {
                // call reveal function
                self.reveal(default_params);
            } else if (pixel.alert == 'explode') {
                // call explode function
                self.explode(default_params);
            } else if (self.ownerless_space(default_params, size: size) == true ){
                // start a new game
                let mut id = world.uuid();
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

				let mut random_number: u256 = 0;

				let mut num_mines = 0;
				loop {
					if num_mines >= mines_amount {
						break;
					}
					let timestamp_felt252 = timestamp.into();
					let x_felt252 = position.x.into();
					let y_felt252 = position.y.into();
					let m_felt252 = num_mines.into();

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

            } else {
                // we can't do anything, so we just return
                'find a free area'.print();
            }

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
                        alert: Option::None,
                        timestamp: Option::None,
                        text: Option::None,
                        app: Option::Some(system),
                        owner: Option::Some(player),
                        action: Option::None // Not using this feature for myapp
                    }
                );

            'put_color DONE'.print();
        }

        /// Reveal a pixel on a certain position
        ///
        /// # Arguments
        /// default_params: Default parameters for the action
        fn reveal(self: @ContractState, default_params: DefaultParameters) {

        }

        /// Explode a pixel on a certain position
        ///
        /// # Arguments
        /// default_params: Default parameters for the action
        fn explode(self: @ContractState, default_params: DefaultParameters) {

        }

        /// Check if a certain position is ownerless
        ///
        /// # Arguments
        /// default_params: Default parameters for the action
        /// size: Size of the board
        fn ownerless_space(self: @ContractState, default_params: DefaultParameters, size: u64) -> bool {
            return true; // for debug
        }
    }
}
```

### Test so far
Please do not foget adding grant_writer:
```rust,ignore
world.grant_writer('Game',minesweeper_actions_address);
```

Check your codes by this command:
```console
sozo test
```