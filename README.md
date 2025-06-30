# FundMe Smart Contract

A Solidity smart contract for crowdfunding and donations with USD-based minimum contribution requirements using Chainlink price feeds.

## Overview

The FundMe contract allows users to donate ETH while ensuring all contributions meet a minimum USD value threshold. It uses Chainlink's decentralized price oracles to convert ETH amounts to USD in real-time, making it resistant to cryptocurrency price volatility.

## Features

- **USD-Based Minimum**: Requires a minimum of $5 USD equivalent in ETH per donation
- **Real-time Price Conversion**: Integrates with Chainlink price feeds for accurate ETH/USD conversion
- **Owner-Only Withdrawals**: Only the contract deployer can withdraw collected funds
- **Flexible Funding**: Accepts direct ETH transfers through fallback and receive functions
- **Gas Optimized**: Includes optimized withdrawal function to reduce gas costs
- **Transparent Tracking**: Maintains records of all funders and their contribution amounts

## Contract Architecture

### Dependencies
- **Chainlink AggregatorV3Interface**: For ETH/USD price data
- **PriceConverter Library**: Custom library for price conversion logic

### Key Components
- `MINIMUM_USD`: Constant minimum donation amount (5 * 10^18 wei equivalent to $5)
- `i_owner`: Immutable address of the contract owner
- `s_funders`: Array tracking all funder addresses
- `s_addressToAmountFunded`: Mapping of funder addresses to their total contributions
- `s_priceFeed`: Chainlink price feed interface

## Functions

### Public Functions

#### `fund()`
Allows users to donate ETH to the contract.
- **Requirements**: ETH value must be equivalent to at least $5 USD
- **Effects**: Records the funder's address and contribution amount

#### `withdraw()`
Allows the owner to withdraw all collected funds.
- **Access**: Owner only
- **Effects**: Resets all funder data and transfers contract balance to owner

#### `cheaperWithdraw()`
Gas-optimized version of the withdraw function.
- **Access**: Owner only
- **Benefits**: Reduces gas costs by caching array length

### View Functions

#### `getVersion()`
Returns the version of the Chainlink price feed being used.

#### `getAddressToAmountFunded(address)`
Returns the total amount funded by a specific address.

#### `getFunder(uint256)`
Returns the funder address at a specific index.

#### `getOwner()`
Returns the contract owner's address.

### Special Functions

#### `fallback()` and `receive()`
Automatically call the `fund()` function when ETH is sent directly to the contract.

## Deployment

### Prerequisites
1. Chainlink price feed contract address for ETH/USD pair
2. Sufficient ETH for gas fees

### Constructor Parameters
- `priceFeed`: Address of the Chainlink ETH/USD price feed contract

### Example Deployment (Ethereum Mainnet)
```solidity
// ETH/USD price feed address on Ethereum Mainnet
address constant ETH_USD_PRICE_FEED = 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;

FundMe fundMe = new FundMe(ETH_USD_PRICE_FEED);
```

## Usage Examples

### Funding the Contract
```solidity
// Option 1: Call fund() function directly
fundMe.fund{value: 0.1 ether}();

// Option 2: Send ETH directly to contract address
// This automatically triggers the fund() function
(bool success,) = address(fundMe).call{value: 0.1 ether}("");
require(success, "Transfer failed");
```

### Checking Contribution
```solidity
// Check how much a specific address has funded
uint256 contribution = fundMe.getAddressToAmountFunded(userAddress);

// Get a funder by index
address funder = fundMe.getFunder(0);
```

### Withdrawing Funds (Owner Only)
```solidity
// Standard withdrawal
fundMe.withdraw();

// Gas-optimized withdrawal
fundMe.cheaperWithdraw();
```

## Security Features

- **Access Control**: Uses custom error `FundMe__NotOwner()` for gas-efficient owner verification
- **Safe Transfers**: Uses the recommended `call()` method for ETH transfers
- **State Reset**: Properly resets funder data during withdrawals
- **Input Validation**: Ensures minimum funding requirements are met

## Gas Optimization

The contract implements several gas optimization techniques:
- Uses `immutable` and `constant` keywords for variables that don't change
- Caches array length in local variables during loops
- Uses custom errors instead of require statements with strings
- Provides both standard and optimized withdrawal functions

## Network Compatibility

The contract can be deployed on any EVM-compatible network with Chainlink price feeds available:
- Ethereum Mainnet
- Polygon
- Avalanche
- Binance Smart Chain
- And other supported networks

**Note**: Ensure you use the correct Chainlink price feed address for your target network.

## License

This project is licensed under the MIT License.

## Dependencies

- Solidity ^0.8.18
- @chainlink/contracts
- Custom PriceConverter library

## Testing Considerations

When testing this contract, consider:
- Mock price feed contracts for local testing
- Different ETH price scenarios
- Edge cases around the minimum funding amount
- Owner and non-owner access patterns
- Gas cost comparisons between withdrawal functions

## Potential Enhancements

- Add events for better transaction tracking
- Implement pause/unpause functionality
- Add support for multiple cryptocurrencies
- Create a withdrawal schedule system
- Add refund functionality for contributors