# wBTC Aave Rewards Farm Badger V1 Strategy

Demo Strategy for Badger V1 Vaults.

This strategy will deposit wBTC on AAVE to earn interest and rewards
It will then claim the rewards to increase the amount of wBTC

## Deposit
Deposit funds in the AAVE Lending Pool, so that we earn interest as well as rewards

## Tend
If there's any wBTC in the strategy, it will be deposited in the pool

## Harvest
The Strategy will harvest stkAAVE, then swap it into AAVE, it will then swap the AAVE into wBTC.

## Expected Yield
The expected yield is:
* Interest of wBTC on AAVE (0.02%)
* Rewards on the wBTC Deposit (1.20%)

For an expected yield of around 1.22% APY

## What you'll find here

- Basic Solidity Smart Contract for creating your own Badger Strategy ([`contracts/MyStrategy.sol`](contracts/MyStrategy.sol))

- Interfaces for some of the most used DeFi protocols on ethereum mainnet. ([`interfaces`](interfaces))
- Dependencies for OpenZeppelin and other libraries. ([`deps`](deps))

- Sample test suite that runs on mainnet fork. ([`tests`](tests))

This mix is configured for use with [Ganache](https://github.com/trufflesuite/ganache-cli) on a [forked mainnet](https://eth-brownie.readthedocs.io/en/stable/network-management.html#using-a-forked-development-network).

## Installation and Setup

1. Use this code by clicking on Use This Template

2. Download the code with ```git clone URL_FROM_GITHUB```

3. [Install Brownie](https://eth-brownie.readthedocs.io/en/stable/install.html) & [Ganache-CLI](https://github.com/trufflesuite/ganache-cli), if you haven't already.

4. Copy the `.env.example` file, and rename it to `.env`

5. Sign up for [Infura](https://infura.io/) and generate an API key. Store it in the `WEB3_INFURA_PROJECT_ID` environment variable.

6. Sign up for [Etherscan](www.etherscan.io) and generate an API key. This is required for fetching source codes of the mainnet contracts we will be interacting with. Store the API key in the `ETHERSCAN_TOKEN` environment variable.


## Basic Use

To deploy the demo Badger Strategy in a development environment:

1. Open the Brownie console. This automatically launches Ganache on a forked mainnet.

```bash
  brownie console
```

2. Run Scripts for Deployment
```
  brownie run deploy
```

Deployment will set up a Vault, Controller and deploy your strategy

3. Run the test deployment in the console and interact with it
```python
  brownie console
  deployed = run("deploy")

  ## Takes a minute or so
  Transaction sent: 0xa0009814d5bcd05130ad0a07a894a1add8aa3967658296303ea1f8eceac374a9
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 9
  UniswapV2Router02.swapExactETHForTokens confirmed - Block: 12614073   Gas used: 88626 (0.74%)

  ## Now you can interact with the contracts via the console
  >>> deployed
  {
      'controller': 0x602C71e4DAC47a042Ee7f46E0aee17F94A3bA0B6,
      'deployer': 0x66aB6D9362d4F35596279692F0251Db635165871,
      'lpComponent': 0x028171bCA77440897B824Ca71D1c56caC55b68A3,
      'rewardToken': 0x7Fc66500c84A76Ad7e9c93437bFc5Ac33E2DDaE9,
      'sett': 0x6951b5Bd815043E3F842c1b026b0Fa888Cc2DD85,
      'strategy': 0x9E4c14403d7d9A8A782044E86a93CAE09D7B2ac9,
      'vault': 0x6951b5Bd815043E3F842c1b026b0Fa888Cc2DD85,
      'want': 0x6B175474E89094C44Da98b954EedeAC495271d0F
  }
  >>>

  ## Deploy also uniswaps want to the deployer (accounts[0]), so you have funds to play with!
  >>> deployed.want.balanceOf(a[0])
  240545908911436022026

```

## Adding Configuration

To ship a valid strategy, that will be evaluated to deploy on mainnet, with potentially $100M + in TVL, you need to:
1. Write the Strategy Code in MyStrategy.sol
2. Customize the StrategyResolver so that snapshot testing can verify that operations happened correctly
3. Write any extra test to confirm that the strategy is working properly

## Implementing Strategy Logic

[`contracts/MyStrategy.sol`](contracts/MyStrategy.sol) is where you implement your own logic for your strategy. In particular:

* Customize the `initialize` Method
* Set a name in `MyStrategy.getName()`
* Set a version in `MyStrategy.version()`
* Write a way to calculate the want invested in `MyStrategy.balanceOfPool()`
* Write a method that returns true if the Strategy should be tended in `MyStrategy.isTendable()`
* Set a version in `MyStrategy.version()`
* Invest your want tokens via `Strategy._deposit()`.
* Take profits and repay debt via `Strategy.harvest()`.
* Unwind enough of your position to payback withdrawals via `Strategy._withdrawSome()`.
* Unwind all of your positions via `Strategy._withdrawAll()`.
* Rebalance the Strategy positions via `Strategy.tend()`.
* Make a list of all position tokens that should be protected against movements via `Strategy.protectedTokens()`.


## Specifying checks for ordinary operations in config/StrategyResolver
In order to snapshot certain balances, we use the Snapshot manager.
This class helps with verifying that ordinary procedures (deposit, withdraw, harvest), happened correctly.

See `/helpers/StrategyCoreResolver.py` for the base resolver that all strategies use
Edit `/config/StrategyResolver.py` to specify and verify how an ordinary harvest should behave

## Add your custom testing
Check the various tests under `/tests`
The file `/tests/test_custom` is already set up for you to write custom tests there


## Testing

To run the tests:

```
brownie test
```


## Debugging Failed Transactions

Use the `--interactive` flag to open a console immediatly after each failing test:

```
brownie test --interactive
```

Within the console, transaction data is available in the [`history`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#txhistory) container:

```python
>>> history
[<Transaction '0x50f41e2a3c3f44e5d57ae294a8f872f7b97de0cb79b2a4f43cf9f2b6bac61fb4'>,
 <Transaction '0xb05a87885790b579982983e7079d811c1e269b2c678d99ecb0a3a5104a666138'>]
```

Examine the [`TransactionReceipt`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#transactionreceipt) for the failed test to determine what went wrong. For example, to view a traceback:

```python
>>> tx = history[-1]
>>> tx.traceback()
```

To view a tree map of how the transaction executed:

```python
>>> tx.call_trace()
```

See the [Brownie documentation](https://eth-brownie.readthedocs.io/en/stable/core-transactions.html) for more detailed information on debugging failed transactions.


## Deployment

When you are finished testing and ready to deploy to the mainnet:

1. [Import a keystore](https://eth-brownie.readthedocs.io/en/stable/account-management.html#importing-from-a-private-key) into Brownie for the account you wish to deploy from.
2. Run [`scripts/deploy.py`](scripts/deploy.py) with the following command

```bash
$ brownie run deployment --network mainnet
```

You will be prompted to enter your keystore password, and then the contract will be deployed.


## Known issues

### No access to archive state errors

If you are using Ganache to fork a network, then you may have issues with the blockchain archive state every 30 minutes. This is due to your node provider (i.e. Infura) only allowing free users access to 30 minutes of archive state. To solve this, upgrade to a paid plan, or simply restart your ganache instance and redploy your contracts.

# Resources

- Badger [Discord channel](https://discord.gg/phbqWTCjXU)
- Yearn [Discord channel](https://discord.com/invite/6PNv2nF/)
- Brownie [Gitter channel](https://gitter.im/eth-brownie/community)
- Alex The Entreprenerd on [Twitter](https://twitter.com/GalloDaSballo)
