# NFT Marketplace on EVM

Solidity marketplace contracts for buying and selling NFTs on EVM chains, with support for both ERC-721 and ERC-1155 tokens through a shared interface.

## What this project includes

- `Marketplace`: core marketplace logic (asks, bids, accept/cancel, escrow withdrawals)
- `MarketplaceFeeCollector`: extension contract with marketplace fee logic
- `ETHMarketplace`: thin wrapper over the fee-collector marketplace
- `NFTCommon`: helper library that abstracts ERC-721 and ERC-1155 ownership/transfer checks
- `E721` and `E1155`: simple faucet-style NFT contracts for local testing

## Supported flows

- Create asks (optionally reserved to a specific buyer)
- Create bids with ETH escrowed in the contract
- Cancel asks and bids
- Accept asks (buyer pays ETH and receives NFT)
- Accept bids (seller transfers NFT and receives ETH, net of fee extension if enabled)
- Withdraw funds from escrow

## Project structure

```text
contracts/
  Marketplace.sol
  ETHMarketplace.sol
  NFTCommon.sol
  extensions/
    MarketplaceFeeCollector.sol
  mint/
    E721.sol
    E1155.sol
interfaces/
  IMarketplace.sol
  INFTContract.sol
hardhat.config.js
```

## Tech stack

- Solidity `^0.8.11` (Hardhat configured with `0.8.17`)
- Hardhat
- OpenZeppelin Contracts

## Getting started

### 1) Install dependencies

```bash
npm install
```

### 2) Compile contracts

```bash
npx hardhat compile
```

### 3) Run local node (optional)

```bash
npx hardhat node
```

### 4) Deploy (example)

No deployment script is included yet. You can deploy from Hardhat console:

```bash
npx hardhat console --network localhost
```

Then in console:

```js
const [deployer] = await ethers.getSigners();
const Marketplace = await ethers.getContractFactory("Marketplace");
const market = await Marketplace.deploy(deployer.address); // beneficiary
await market.deployed();
market.address;
```

## Contracts overview

### `Marketplace`

Core state:

- `asks[nft][tokenId]`
- `bids[nft][tokenId]`
- `escrow[user]`

Main external functions:

- `createAsk(INFTContract[] nft, uint256[] tokenID, uint256[] price, address[] to)`
- `createBid(INFTContract[] nft, uint256[] tokenID, uint256[] price)` (payable)
- `cancelAsk(INFTContract[] nft, uint256[] tokenID)`
- `cancelBid(INFTContract[] nft, uint256[] tokenID)`
- `acceptAsk(INFTContract[] nft, uint256[] tokenID)` (payable)
- `acceptBid(INFTContract[] nft, uint256[] tokenID)`
- `withdraw()`

Admin functions:

- `changeBeneficiary(address payable newBeneficiary)`
- `revokeAdmin()`

### `MarketplaceFeeCollector` / `ETHMarketplace`

- Adds configurable fee in basis points (`fee`)
- Sends fee cut to `beneficiary`
- Exposes `changeFee(uint256 newFee)` (admin only)

### `NFTCommon` library

Unifies token operations across standards by:

- trying ERC-721 transfer first, then ERC-1155 fallback
- checking ownership via `ownerOf` fallback to `balanceOf`

## Local testing NFTs

### `E721`

- `faucet()` mints one ERC-721 token to caller

### `E1155`

- `faucet()` increments token id and mints quantity `10` of that id to caller
- `totalSupply()` returns latest minted token id counter

## Notes and caveats

- The repository currently has no test files or deployment scripts.
- `package.json` still contains the default `test` script placeholder.
- Always validate business logic with thorough tests before mainnet deployment.
- Review fee math and escrow update behavior carefully for production hardening.

## Recommended next steps

- Add Hardhat tests for all bid/ask and escrow paths
- Add deployment scripts under `scripts/`
- Add CI checks (`compile`, `test`, optional linting)
- Add event indexing examples for frontend integration
