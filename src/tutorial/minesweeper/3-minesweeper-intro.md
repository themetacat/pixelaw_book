## What we build: Minesweeper

For details, please see [wiki](https://en.wikipedia.org/wiki/Minesweeper_(video_game))

### Functions
We need following functions:
- init: initialize application settings.
- interact: interact with PixeLAW field. When we start game, we call this function to set up field.
- reveal: reveal the pixel if bomb or not.
- explode: if a player reveals a bomb, this function is called.
- ownerless_space: 

We can write interface like this:
```rust
#[starknet::interface]
trait IMinesweeperActions<TContractState> {
    fn init(self: @TContractState);
    fn interact(self: @TContractState, default_params: DefaultParameters, size: u64, mines_amount: u64);
    fn reveal(self: @TContractState, default_params: DefaultParameters);
    fn explode(self: @TContractState, default_params: DefaultParameters);
    fn ownerless_space(self: @TContractState, default_params: DefaultParameters, size: u64) -> bool;
}
```

### Set constants
We set some constants. We added `GAME_MAX_DURATION` to avoid parmanent sessions.
```rust
/// APP_KEY must be unique across the entire platform
const APP_KEY: felt252 = 'minesweeper';

/// Core only supports unicode icons for now
const APP_ICON: felt252 = 'U+1F4A3'; // bomb

/// prefixing with BASE means using the server's default manifest.json handler
const APP_MANIFEST: felt252 = 'BASE/manifests/minesweeper';

/// The maximum duration of a game in milliseconds
const GAME_MAX_DURATION: u64 = 20000;
```

We used [this](http://xahlee.info/comp/unicode_index.html?q=bom) to search the unicode icon.

### Create enum and struct

We use State and Game. State represents statement of pixel in minesweeper, and Game struct handle the components for this game.

```rust
#[derive(Serde, Copy, Drop, PartialEq, Introspect)]
enum State {
    None: (),
    Open: (),
    Finished: ()
}

#[derive(Model, Copy, Drop, Serde, SerdeLen)]
struct Game {
    #[key]
    x: u64,
    #[key]
    y: u64,
    id: u32,
    creator: ContractAddress,
    state: State,
    size: u64,
    mines_amount: u64,
    started_timestamp: u64
}  
```

### Import them
Don't forget to import them.
```rust
use super::{Game, State};
use super::{APP_KEY, APP_ICON, APP_MANIFEST, GAME_MAX_DURATION};
```

### Set up functions
In `ActionsImpl`, please declare functions.
```rust
#[dojo::contract]
mod minesweeper_actions {
    /// ...
    #[external(v0)]
    impl ActionsImpl of IMinesweeperActions<ContractState> {
        fn init(self: @TContractState) {}
        fn interact(self: @TContractState, default_params: DefaultParameters, size: u64, mines_amount: u64) {}
        fn reveal(self: @TContractState, default_params: DefaultParameters) {}
        fn explode(self: @TContractState, default_params: DefaultParameters) {}
        fn ownerless_space(self: @TContractState, default_params: DefaultParameters, size: u64) -> bool {}
    }
}
```