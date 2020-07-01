# Logic

Full example of the `logic` crate is available on our [CasperLabs/erc20](https://github.com/CasperLabs/erc20/tree/master/logic) repository

## ERC-20 Standard
The ERC-20 standard is defined in [an Ethereum Improvement Proposal (EIP)](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#). Read it carefully, as it defines the methods we'll implement:
* balance_of
* transfer
* total_supply
* approve
* allowance
* transfer_from
* mint

## Add new crate
One of the great benefits of writing smart contracts in Rust is that we can write a lot of code as a standalone library and use it later to implement smart contracts. In this section we will implement the logic of ERC-20. To make it testable, and easy to use later in smart contracts, we will make all the memory operations abstract.

Generate a new `logic` crate.
```
$ cargo new logic --lib
warning: compiling this new crate may not work due to invalid workspace configuration
```
Cargo reminds us to add `logic` to the current workspace, so let's modify `Cargo.toml` in the root directory.
```toml
# Cargo.toml

[workspace]

members = [
    "logic",
    "contract",
    "tests"
]
```
Run `logic` tests to see it works.
```bash
$ cargo test -p logic
```

## Cargo.toml
Prepare `logic/Cargo.toml`.
```toml
# logic/Cargo.toml

[package]
name = "logic"
version = "0.1.0"
authors = ["Maciej Zielinski <maciej@casperlabs.io>"]
edition = "2018"

[lib]
name = "logic"
doctest = false
bench = false

[dependencies]
num-traits = { version = "0.2.10", default-features = false }
```

## ERC20Trait
The logic will be implemented as an `ERC20Trait` trait.
```rust
// logic/src/lib.rs

pub trait ERC20Trait<
    Amount: num_traits::Zero + Add<Output = Amount> + Sub<Output = Amount> + PartialOrd + Copy,
    Address,
>
{}
```
The `Amount` and `Address` type parameters allow for flexibility in the types used in concrete implementations of the trait (e.g. implementation in tests and implementation in smart contracts).

## Reads and Writes
Next things to add are abstract functions, that handle data saves and reads.
```rust
// logic/src/lib.rs

pub trait ERC20Trait<
    Amount: num_traits::Zero + Add<Output = Amount> + Sub<Output = Amount> + PartialOrd + Copy,
    Address,
>
{
    fn read_balance(&mut self, address: &Address) -> Option<Amount>;
    fn save_balance(&mut self, address: &Address, balance: Amount);
    fn read_total_supply(&mut self) -> Option<Amount>;
    fn save_total_supply(&mut self, total_supply: Amount);
    fn read_allowance(&mut self, owner: &Address, spender: &Address) -> Option<Amount>;
    fn save_allowance(&mut self, owner: &Address, spender: &Address, amount: Amount);
}
```

## Total Supply, Balance and Allowance
We are ready now to define first ERC-20 methods. Below is the implementation of `balance_of`, `total_supply` and `allowance` inside the `ERC20Trait`. These are read-only methods.
```rust
// logic/src/lib.rs
pub trait ERC20Trait<...> {
    ...
    fn balance_of(&mut self, address: &Address) -> Amount {
        self.read_balance(address).unwrap_or_else(Amount::zero)
    }

    fn total_supply(&mut self) -> Amount {
        self.read_total_supply().unwrap_or_else(Amount::zero)
    }

    fn allowance(&mut self, owner: &Address, spender: &Address) -> Amount {
        self.read_allowance(owner, spender).unwrap_or_else(Amount::zero)
    }
}
```

## Mint
Next method to define inside the `ERC20Trait` is called `mint`. It's not a part of the ERC-20 specification, but it's present in almost every ERC-20 implementation. Its responsibility is incrementing the balance of tokens for the given `address`. It should update the total supply as well.
```rust
// logic/src/lib.rs
pub trait ERC20Trait<...> {
    ...
    fn mint(&mut self, address: &Address, amount: Amount) {
        let address_balance = self.balance_of(address);
        let total_supply = self.total_supply();
        self.save_balance(&address, address_balance + amount);
        self.save_total_supply(total_supply + amount);
    }
}
```

## Errors
Further implementation of `transfer` and `transfer_from` will be able to throw errors. Let's define them in the separate file and have them ready for later.
```rust
// logic/src/errors.rs

#[derive(PartialEq, Debug)]
pub enum ERC20TransferError {
    NotEnoughBalance,
}

#[derive(PartialEq, Debug)]
pub enum ERC20TransferFromError {
    TransferError(ERC20TransferError),
    NotEnoughAllowance,
}

impl From<ERC20TransferError> for ERC20TransferFromError {
    fn from(error: ERC20TransferError) -> ERC20TransferFromError {
        ERC20TransferFromError::TransferError(error)
    }
}
```

## Transfer
Finally we can implement `transfer` method, so it's possible to transfer tokens from `sender` address to `recipient` address. If the `sender` address has enough balance then tokens should be transferred to the `recipient` address. Otherwise return the `ERC20TransferError::NotEnoughBalance` error.
```rust
pub trait ERC20Trait<...> {
    ...
    fn transfer(
        &mut self,
        sender: &Address,
        recipient: &Address,
        amount: Amount,
    ) -> Result<(), ERC20TransferError> {
        let sender_balance = self.balance_of(sender);
        if amount > sender_balance {
            Err(ERC20TransferError::NotEnoughBalance)
        } else {
            let recipient_balance = self.balance_of(recipient);
            self.save_balance(&sender, sender_balance - amount);
            self.save_balance(&recipient, recipient_balance + amount);
            Ok(())
        }
    }
}
```

## Approve and Transfer From
The last missing functions are `approve` and `transfer_from`. `approve` is used to allow another address to spend tokens on my behalf.
```rust
pub trait ERC20Trait<...> {
    ...
    fn approve(&mut self, owner: &Address, spender: &Address, amount: Amount) {
        self.save_allowance(owner, spender, amount)
    }
}
```
`transfer_from` allows to spend approved amount of tokens.
```rust
pub trait ERC20Trait<...> {
    ...
    fn transfer_from(
        &mut self,
        spender: &Address,
        owner: &Address,
        recipient: &Address,
        amount: Amount,
    ) -> Result<(), ERC20TransferFromError> {
        let allowance = self.allowance(owner, spender);
        if amount > allowance {
            return Err(ERC20TransferFromError::NotEnoughAllowance);
        }
        self.transfer(owner, recipient, amount)?;
        self.approve(owner, spender, allowance - amount);
        Ok(())
    }
}
``` 
Note, that internaly it uses `transfer` function. If transfer fails, the `ERC20TransferError` is automatically converted to `ERC20TransferFromError` thanks to the `impl From<ERC20TransferError> for ERC20TransferFromError` implementation in `logic/src/error.rs`.
