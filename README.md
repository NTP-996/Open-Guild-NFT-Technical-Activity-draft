# 📚 Tutorial: Building an NFT Card Game with Ink!, Inkathon, and OpenBrush

This tutorial will guide you through the process of building an NFT-based card game using Ink!, the smart contract language for the Polkadot ecosystem, along with Inkathon for deployment and interaction, and OpenBrush for standardized NFT implementation. You'll implement essential components of the game, including card creation, ownership transfer, and basic gameplay mechanics.

## 📝 Prerequisites

Before we start, ensure you have the following:

1. **🦀 Rust**: Install Rust and set up your environment.
   - Install Rust: [Rust Installation Guide](https://www.rust-lang.org/tools/install)
   - Add the wasm32-unknown-unknown target:
     ```bash
     rustup target add wasm32-unknown-unknown
     ```

2. **💻 Visual Studio Code (VS Code)**: Set up VS Code for Rust and Ink! development.
   - Download and install VS Code from the [official website](https://code.visualstudio.com/).
   - Install the following extensions in VS Code:
     - Rust Extension
     - Ink! Extension
     - Rust Analyzer Extension (Recommended)

3. **📦 Node.js and npm**: Required for Inkathon.
   - Download and install from the [official Node.js website](https://nodejs.org/).

### 📦 Setup on Different Operating Systems

(Setup instructions for Windows, macOS, and Linux as provided in the original document)

## 🚀 Step 1: Set Up a New Ink! Project

Create a new Ink! project for your NFT card game contract.

```bash
cargo contract new nft_card_game
cd nft_card_game
```

## 🛠️ Step 2: Add OpenBrush Dependency

Open the `Cargo.toml` file in your project and add OpenBrush as a dependency.

```toml
[dependencies]
ink = { version = "5.0.0", default-features = false }
openbrush = { tag = "4.0.0", git = "https://github.com/Brushfam/openbrush-contracts", default-features = false, features = ["psp34"] }

[lib]
name = "nft_card_game"
path = "lib.rs"
crate-type = [
    "cdylib",
]

[features]
default = ["std"]
std = [
    "ink/std",
    "openbrush/std",
]
```

## 🧑‍💻 Step 3: Implement NFT Card Game Logic

Create a new file `lib.rs` in the `src` directory and implement the NFT card game logic:

```rust
#![cfg_attr(not(feature = "std"), no_std, no_main)]

#[openbrush::implementation(PSP34, Ownable)]
#[openbrush::contract]
pub mod nft_card_game {
    use openbrush::contracts::ownable::*;
    use openbrush::contracts::psp34::*;
    use openbrush::traits::Storage;
    use ink::prelude::string::String;
    use ink::prelude::vec::Vec;

    #[derive(Default, Storage)]
    #[ink(storage)]
    pub struct NftCardGame {
        #[storage_field]
        psp34: psp34::Data,
        #[storage_field]
        ownable: ownable::Data,
        cards: Vec<Card>,
    }

    #[derive(Clone, scale::Encode, scale::Decode)]
    #[cfg_attr(feature = "std", derive(scale_info::TypeInfo, ink::storage::traits::StorageLayout))]
    pub struct Card {
        name: String,
        attack: u32,
        defense: u32,
    }

    impl NftCardGame {
        #[ink(constructor)]
        pub fn new() -> Self {
            let mut instance = Self::default();
            instance._init_with_owner(Self::env().caller());
            instance
        }

        #[ink(message)]
        pub fn create_card(&mut self, name: String, attack: u32, defense: u32) -> Result<(), PSP34Error> {
            self.ensure_owner()?;
            let card = Card { name, attack, defense };
            self.cards.push(card);
            let id = PSP34Id::U32(self.cards.len() as u32);
            self._mint_to(self.env().caller(), id)?;
            Ok(())
        }

        #[ink(message)]
        pub fn get_card(&self, token_id: u32) -> Option<Card> {
            if token_id > 0 && token_id <= self.cards.len() as u32 {
                Some(self.cards[(token_id - 1) as usize].clone())
            } else {
                None
            }
        }

        #[ink(message)]
        pub fn play_game(&self, player1_card: u32, player2_card: u32) -> Option<u32> {
            let card1 = self.get_card(player1_card)?;
            let card2 = self.get_card(player2_card)?;
            
            let player1_power = card1.attack + card1.defense;
            let player2_power = card2.attack + card2.defense;
            
            if player1_power > player2_power {
                Some(player1_card)
            } else if player2_power > player1_power {
                Some(player2_card)
            } else {
                None // It's a tie
            }
        }
    }
}
```

This implementation includes basic card creation, retrieval, and a simple game mechanic.

## 🔨 Step 4: Compile and Deploy

Compile your contract using the following command:

```bash
cargo +nightly contract build
```

This will generate a `.wasm` file and metadata JSON that you can use for deployment.

## 🌐 Step 5: Deploy and Interact using Inkathon

### ⚙️ Create a new Inkathon project:

```bash
npx create-inkathon-app my_nft_card_game
cd my_nft_card_game
npm install
```

### 🚀 Deploy the Contract:

Create a new file `deploy.ts` in your Inkathon project:

```typescript
import { deployContract } from '@scio-labs/use-inkathon'
import { initPolkadotJs } from './initPolkadotJs'

async function main() {
  // Initialize Polkadot.js
  const { api, keyring } = await initPolkadotJs()

  // Deploy the contract
  const { address } = await deployContract(api, keyring, {
    contractName: 'nft_card_game',
    constructorArgs: [],
    value: '0',
    gasLimit: '100000000000',
  })

  console.log(`Contract deployed at: ${address}`)
}

main().catch(console.error)
```

Run the deployment script:

```bash
npx ts-node deploy.ts
```

### 💬 Interact with the Contract:

Create an `interact.ts` file to interact with your deployed contract:

```typescript
import { ContractPromise } from '@polkadot/api-contract'
import { initPolkadotJs } from './initPolkadotJs'

async function main() {
  const { api, keyring } = await initPolkadotJs()

  const contractAddress = 'YOUR_DEPLOYED_CONTRACT_ADDRESS'
  const contract = new ContractPromise(api, ABI, contractAddress)

  const alice = keyring.addFromUri('//Alice')

  // Create a card
  await contract.tx.createCard({ gasLimit: '1000000000' }, 'Dragon', 100, 50).signAndSend(alice)

  // Get card details
  const { result, output } = await contract.query.getCard(alice.address, { gasLimit: '1000000000' }, 1)
  if (result.isOk && output) {
    console.log('Card details:', output.toHuman())
  }

  // Play a game
  const { result: gameResult, output: gameOutput } = await contract.query.playGame(alice.address, { gasLimit: '1000000000' }, 1, 2)
  if (gameResult.isOk && gameOutput) {
    console.log('Game result:', gameOutput.toHuman())
  }
}

main().catch(console.error)
```

Run the interaction script:

```bash
npx ts-node interact.ts
```

## 🎉 Conclusion

Congratulations! You've now created an NFT card game using Ink! smart contracts, deployed it using Inkathon, and interacted with it using Polkadot.js. This tutorial provides a foundation for building more complex blockchain-based games. You can expand on this by adding more game mechanics, implementing a user interface, or integrating with other blockchain features.

For more advanced features and best practices, refer to the official documentation:
- [Ink! Documentation](https://use.ink/)
- [OpenBrush GitHub Repository](https://github.com/Brushfam/openbrush-contracts)
- [Inkathon GitHub Repository](https://github.com/scio-labs/inkathon)
