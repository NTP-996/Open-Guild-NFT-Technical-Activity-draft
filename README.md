# 📚 Tutorial: Building an NFT Card Game with Ink! Smart Contract

This tutorial will guide you through the process of building an NFT-based card game using Ink!, the smart contract language for the Polkadot ecosystem. You'll implement essential components of the game, including card creation, ownership transfer, and gameplay mechanics.

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
     - **Rust Extension**:
       - Go to the Extensions view (`Ctrl+Shift+X`).
       - Search for "Rust" and install the [Rust extension by rust-lang](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust).
     - **Ink! Extension**:
       - In the Extensions view, search for "Ink!" and install the [Ink! extension](https://marketplace.visualstudio.com/items?itemName=ParityTechnologies.ink-vscode).
     - **Rust Analyzer Extension (Recommended)**:
       - This extension provides enhanced Rust language support, including code completion and inline error checking.
       - Search for "Rust Analyzer" in the Extensions view and install the [Rust Analyzer extension](https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer).

### 📦 Setup on Different Operating Systems

### 🪟 Windows

1. **Install Rust**:
   - **Option 1: Native Windows Installation**
     - Download and install Rust using the installer provided at the [official Rust website](https://www.rust-lang.org/tools/install).
     - Open Command Prompt (cmd) or PowerShell and run:
       ```bash
       rustup target add wasm32-unknown-unknown
       ```

   - **Option 2: Using WSL2 (Recommended)**
     - Install WSL2 if you haven't already by following the [WSL installation guide](https://docs.microsoft.com/en-us/windows/wsl/install).
     - Install a Linux distribution from the Microsoft Store (e.g., Ubuntu).
     - Open your WSL2 terminal (e.g., Ubuntu) and install Rust:
       ```bash
       curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
       ```
     - After installation, add the wasm32 target:
       ```bash
       rustup target add wasm32-unknown-unknown
       ```

2. **Install Ink! CLI**:
   - **Option 1: Native Windows Installation**
     - Open Command Prompt or PowerShell and run:
       ```bash
       cargo install --force --locked cargo-contract
       ```

   - **Option 2: Using WSL2**
     - In your WSL2 terminal, run:
       ```bash
       cargo install --force --locked cargo-contract
       ```

3. **Install Node.js and NPM**:
   - **Option 1: Native Windows Installation**
     - Download the Windows installer from the [Node.js website](https://nodejs.org/) and run it.
     - Confirm installation by running in Command Prompt:
       ```bash
       node -v
       npm -v
       ```

   - **Option 2: Using WSL2**
     - In your WSL2 terminal, install Node.js using the package manager:
       ```bash
       sudo apt update
       sudo apt install nodejs npm
       ```
     - Confirm installation by running:
       ```bash
       node -v
       npm -v
       ```

4. **Install Inkathon and OpenBrush**:
   - **Option 1: Native Windows Installation**
     - In Command Prompt, run:
       ```bash
       npm install --save inkathon
       ```
     - For OpenBrush, run:
       ```bash
       cargo install cargo-contract
       ```

   - **Option 2: Using WSL2**
     - In your WSL2 terminal, run:
       ```bash
       npm install --save inkathon
       ```
     - For OpenBrush, run:
       ```bash
       cargo install cargo-contract
       ```

**Note:** Using WSL2 is recommended because it offers a more consistent development environment similar to Linux. This is particularly useful for developers working with tools that are often Linux-first, like Rust and Substrate. WSL2 also supports better file system performance and compatibility with various development tools compared to the native Windows environment.

#### 🍎 macOS

1. **Install Rust**:
   - Use the Terminal and run:
     ```bash
     curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
     ```
   - After installation, add the wasm32 target:
     ```bash
     rustup target add wasm32-unknown-unknown
     ```

2. **Install Ink! CLI**:
   - Run in Terminal:
     ```bash
     cargo install --force --locked cargo-contract
     ```

3. **Install Node.js and NPM**:
   - Install Node.js via [Homebrew](https://brew.sh/) by running:
     ```bash
     brew install node
     ```
   - Confirm installation by running:
     ```bash
     node -v
     npm -v
     ```

4. **Install Inkathon and OpenBrush**:
   - In Terminal, run:
     ```bash
     npm install --save inkathon
     ```
   - For OpenBrush, run:
     ```bash
     cargo install cargo-contract
     ```

#### 🐧 Linux (Ubuntu/Debian)

1. **Install Rust**:
   - Open the terminal and run:
     ```bash
     curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
     ```
   - After installation, add the wasm32 target:
     ```bash
     rustup target add wasm32-unknown-unknown
     ```

2. **Install Ink! CLI**:
   - Run in Terminal:
     ```bash
     cargo install --force --locked cargo-contract
     ```

3. **Install Node.js and NPM**:
   - Install via package manager:
     ```bash
     sudo apt update
     sudo apt install nodejs npm
     ```
   - Confirm installation by running:
     ```bash
     node -v
     npm -v
     ```

4. **Install Inkathon and OpenBrush**:
   - In Terminal, run:
     ```bash
     npm install --save inkathon
     ```
   - For OpenBrush, run:
     ```bash
     cargo install cargo-contract
     ```

## 🚀 Step 1: Set Up a New Ink! Project

Create a new Ink! project for your NFT card game contract.

```bash
cargo contract new nft_card_game
cd nft_card_game
```

This will create a basic Ink! project structure.

## 🛠️ Step 2: Add OpenBrush Dependency

Open the `Cargo.toml` file in your project and add OpenBrush as a dependency.

```toml
[dependencies]
ink = { version = "5.0.0", default-features = false }
openbrush = { tag = "4.0.0", git = "https://github.com/Brushfam/openbrush-contracts", default-features = false, features = [
    "psp34",
] }
```

## 🧑‍💻 Step 3: Implement NFT Card Game Logic

### ⚙️ Card Struct and NFT Implementation:

Create a new file `lib.rs` under the `src` directory and paste the following code:

```rust
use ink_lang as ink;
use openbrush::contracts::psp34::*;
use openbrush::traits::Storage;

#[ink::contract]
mod nft_card_game {
    use super::*;
    use openbrush::contracts::psp34::extensions::enumerable::*;

    #[ink(storage)]
    #[derive(Default, Storage)]
    pub struct NftCardGame {
        #[storage_field]
        psp34: psp34::Data,
    }

    impl PSP34 for NftCardGame {}
    impl PSP34Enumerable for NftCardGame {}

    impl NftCardGame {
        #[ink(constructor)]
        pub fn new() -> Self {
            Self::default()
        }

        #[ink(message)]
        pub fn mint_card(&mut self, to: AccountId, id: Id) -> Result<(), PSP34Error> {
            self._mint_to(to, id)
        }

        #[ink(message)]
        pub fn transfer_card(&mut self, from: AccountId, to: AccountId, id: Id) -> Result<(), PSP34Error> {
            self._transfer(from, to, id)
        }

        #[ink(message)]
        pub fn play_game(&self, player1: AccountId, player2: AccountId, card1: Id, card2: Id) -> bool {
            // Implement game logic here
            true // For demonstration purposes, player1 always wins
        }
    }
}
```

This code sets up the basic structure for minting and transferring NFT cards, as well as a placeholder for the game logic.

## 🔨 Step 4: Compile and Deploy

Compile your contract using the following command:

```bash
cargo +nightly contract build
```

This will generate a `.wasm` file that you can deploy on a Substrate-based chain.

## 🌐 Step 5: Deploy and Interact using Inkathon

Inkathon provides tools to interact with your smart contract from a TypeScript environment.

### ⚙️ Create a new Inkathon project:

```bash
npx create-inkathon-app my_inkathon_app
cd my_inkathon_app
npm install
```

### 🚀 Deploy the Contract:

Use the Inkathon framework to deploy your contract:

```typescript
import { deployContract } from '@scio-labs/inkathon';

async function deployNftCardGameContract() {
    const contractAddress = await deployContract({
        codeHash: '<your_contract_code_hash>',
        constructor: '<constructor>',
        gasLimit: '<gas_limit>',
        value: '<value>',
    });
    console.log('Contract deployed at:', contractAddress);
}

deployNftCardGameContract();
```

Replace `<your_contract_code_hash>`, `<constructor>`, `<gas_limit>`, and `<value>` with the appropriate values from your contract.

### 💬 Interact with the Contract:

Use Inkathon's functions to call your contract methods, such as minting cards, transferring cards, and initiating gameplay.

```typescript
import { callContractMethod } from '@scio-labs/inkathon';

async function mintNftCard(contractAddress: string, to: string, id: string) {
    const result = await callContractMethod({
        address: contractAddress,
        method: 'mint_card',
        args: [to, id],
        gasLimit: '<gas_limit>',
        value: '<value>',
    });
    console.log('Minting result:', result);
}

mintNftCard('<contract_address>', '<recipient_address>', '1');
```

## 🎉 Conclusion

You’ve now set up a basic environment to build and interact with an NFT card game using Ink! and OpenBrush. Feel free to expand on this tutorial by implementing more complex game mechanics or adding additional features to your NFT card game. For more information, explore the [Ink! documentation](https://use.ink/) and the [OpenBrush GitHub repository](https://github.com/727-Ventures/openbrush-contracts) for more advanced use cases and examples.
