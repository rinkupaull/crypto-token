# crypto-token

# CryptoToken Smart Contract

## Overview

The `CryptoToken` smart contract is an ERC20 token with additional functionalities for minting, burning, and redeeming entries for specific types (Bitcoin, Ethereum, Solana). It is built using OpenZeppelin libraries and includes owner-based access control.

## Features

1. **Token Initialization**: The contract is initialized with entry costs for different types of entries.
2. **Minting Tokens**: The owner can mint new tokens to any address.
3. **Burning Tokens**: Users can burn their own tokens.
4. **Transferring Tokens**: Users can transfer tokens to other addresses.
5. **Redeeming Entries**: Users can redeem tokens for specific entries.
6. **Query Functions**: Users can check their token balance and the last redeemed entry.

## Contract Details

### Inheritance
- Inherits from `ERC20`, `Ownable`, and `ERC20Burnable` from the OpenZeppelin library.

### State Variables
- `entryCosts`: A mapping of entry types to their respective costs.
- `userRedemptions`: A mapping of user addresses to their last redeemed entry type.

### Constructor

```solidity
constructor(address initialOwner) ERC20("Crypto Token", "CTK") Ownable(initialOwner) {
    initializeEntryCosts();
}
```
- **Parameters**:
  - `initialOwner`: The address of the initial owner of the contract.
- **Functionality**:
  - Initializes the entry costs.
  - Sets the initial owner of the contract.

### Functions

#### `initializeEntryCosts`

```solidity
function initializeEntryCosts() internal {
    entryCosts[EntryType.Bitcoin] = 800;
    entryCosts[EntryType.Ethereum] = 1200;
    entryCosts[EntryType.Solana] = 0; // Solana entry is free
}
```
- Sets the costs for each entry type.

#### `updateEntryCost`

```solidity
function updateEntryCost(EntryType entryType, uint256 cost) external onlyOwner {
    entryCosts[entryType] = cost;
}
```
- **Parameters**:
  - `entryType`: The type of entry to update the cost for.
  - `cost`: The new cost of the entry.
- **Access**:
  - Only the owner can call this function.

#### `mintTokens`

```solidity
function mintTokens(address recipient, uint256 amount) external onlyOwner {
    _mint(recipient, amount);
}
```
- **Parameters**:
  - `recipient`: The address to receive the minted tokens.
  - `amount`: The number of tokens to mint.
- **Access**:
  - Only the owner can call this function.

#### `transferTokens`

```solidity
function transferTokens(address recipient, uint256 amount) external {
    require(balanceOf(msg.sender) >= amount, "INSUFFICIENT TOKENS!");
    _transfer(msg.sender, recipient, amount);
}
```
- **Parameters**:
  - `recipient`: The address to transfer the tokens to.
  - `amount`: The number of tokens to transfer.

#### `burnTokens`

```solidity
function burnTokens(uint256 amount) external {
    require(balanceOf(msg.sender) >= amount, "INSUFFICIENT TOKENS!");
    _burn(msg.sender, amount);
    emit Redeemed(msg.sender, "Burned tokens", amount);
}
```
- **Parameters**:
  - `amount`: The number of tokens to burn.

#### `redeemEntryByName`

```solidity
function redeemEntryByName(string calldata entryName) external {
    EntryType entryType = getEntryTypeByName(entryName);
    uint256 cost = entryCosts[entryType];

    require(cost >= 0, "Invalid entry type"); // Allow free entries
    require(balanceOf(msg.sender) >= cost, "INSUFFICIENT TOKENS!");

    userRedemptions[msg.sender] = entryType;
    _burn(msg.sender, cost);
    emit Redeemed(msg.sender, entryName, cost);
}
```
- **Parameters**:
  - `entryName`: The name of the entry to redeem tokens for.

#### `getMyTokenBalance`

```solidity
function getMyTokenBalance() external view returns (uint256) {
    return balanceOf(msg.sender);
}
```
- Returns the token balance of the caller.

#### `getLastRedeemedEntry`

```solidity
function getLastRedeemedEntry() external view returns (EntryType) {
    return userRedemptions[msg.sender];
}
```
- Returns the last redeemed entry type for the caller.

#### `getEntryTypeByName`

```solidity
function getEntryTypeByName(string calldata entryName) internal pure returns (EntryType) {
    if (compareStrings(entryName, "Bitcoin")) {
        return EntryType.Bitcoin;
    } else if (compareStrings(entryName, "Ethereum")) {
        return EntryType.Ethereum;
    } else if (compareStrings(entryName, "Solana")) {
        return EntryType.Solana;
    } else {
        revert("Invalid entry name");
    }
}
```
- **Parameters**:
  - `entryName`: The name of the entry.
- **Returns**:
  - The corresponding entry type.

#### `compareStrings`

```solidity
function compareStrings(string memory a, string memory b) internal pure returns (bool) {
    return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
}
```
- **Parameters**:
  - `a`: First string to compare.
  - `b`: Second string to compare.
- **Returns**:
  - `true` if the strings are equal, `false` otherwise.

## Usage

### Deployment

Deploy the contract using any Ethereum development environment like Remix, Hardhat, or Truffle. Provide the initial owner address during deployment.

### Minting Tokens

The owner can mint new tokens to any address.

```solidity
mintTokens(addressOfRecipient, amount);
```

### Burning Tokens

Users can burn their own tokens.

```solidity
burnTokens(amount);
```

### Transferring Tokens

Users can transfer tokens to other addresses.

```solidity
transferTokens(addressOfRecipient, amount);
```

### Redeeming Entries

Users can redeem tokens for specific entries.

```solidity
redeemEntryByName("entryName");
```

### Querying Token Balance

Users can check their token balance.

```solidity
getMyTokenBalance();
```

### Querying Last Redeemed Entry

Users can check their last redeemed entry.

```solidity
getLastRedeemedEntry();
```

## License

This project is licensed under the MIT License.
