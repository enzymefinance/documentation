# Fund

## Hub & Spoke

## General

The Hub and Spoke contracts make up the core contract framework of the Melon fund. Hub and Spoke is an architecture design to help reason about all the moving parts of fund as constructed by an interconnected system of linked smart contracts. The Hub will contain all relevant information for all Spokes registered with the Hub. Each Spoke will independently contain all relevant information about the Hub. Individual Spokes will have programmatic access to information in specific other Spokes.

In general, there is one Hub contract and separate, distinct Spoke contracts for each fund. (CHECK: Implementation for shared components as spoke) The Hub forms the core of the individual Melon fund with each Spoke contributing specific services to the fund. The Hub contract stores the manager's address and the fund name permanently in state, and also contains the functionality to permanently and irreversibly shut the Melon fund down.

## Hub.sol

#### Description

The Hub maintains the specific Spoke components of the fund in terms of their routing and their permissioning. Permissioning is of utmost importance as it ensures only design-intended access to specific components of the Melon fund smart contract system. The permissioning has a very low-level granularity, exclusively permitting only specific Spokes to invoke specific functions on specific target Spokes. This surgical permissioning system ensures that sensitive function execution can occur as intended and not by external actors.

#### Inherits from

DSGuard (link)
&nbsp;

#### On Construction

The constructor requires and sets the manager address and the fund name as well as the `creator` and `creationTime.
&nbsp;

#### Structs

`Routes`

Member variables:

`address accounting`
`address feeManager`
`address participation`
`address policyManager`
`address shares`
`address trading`
`address vault`
`address priceSource`
`address registry`
`address version`
`address engine`
`address mlnToken`

This struct stores the contract addresses of the listed components and spokes.
&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

`modifier onlyCreator()`

A modifier requiring that the `msg.sender` is the `creator`.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`Routes public routes`

A `Routes` struct containing Spoke and component addresses.
&nbsp;

`address public manager`

The address of the fund manager.
&nbsp;

`string public name`

The name of the fund as defined by the manager at fund set up.
&nbsp;

`bool public isShutDown`

A boolean variable defining the operational status of the fund.
&nbsp;

`bool public spokesSet`

A boolean variable defining the completeness status of all spoke settings.
&nbsp;

`bool public routingSet`

A boolean variable defining the completeness status of all routing settings.
&nbsp;

`bool public permissionsSet`

A boolean variable defining the completeness status of all permissions settings.
&nbsp;

`address public creator`

An address variable defining the creator of the Melon fund, i.e. `msg.sender`.
&nbsp;

`uint public creationTime`

An integer variable representing the Melon fund's creation time.
&nbsp;

mapping (address => bool) public isSpoke

`A public mapping associating an address to a boolean indicating that the address is a Spoke of the Melon fund.`
&nbsp;

#### Public Functions

`function shutDownFund() public`

This function sets the `ìsShutDown` state variable to `true`, effectively disabling the fund for all activities except investor redemptions. This function can only be successfully called by the manager address. The function's actions are permanent and irreversible.
&nbsp;

`function setSpokes(address[12] _spokes) onlyCreator`

This function takes an array of addresses and sets all member variables of the `routes` state variable struct if they have not already been set as part of the fund initialization sequence. The function then sets the `spokesSet` state variable to `true`. This function implements the `onlyCreator` modifier.
&nbsp;

`function setRouting() onlyCreator`

This function requires that the Spokes and Routings have been set. It then initializes (see Spoke `initialize()` below) all registered Spokes and finally sets the `routingSet` state variable to `true`. This function implements the `onlyCreator` modifier.
&nbsp;

`function setPermissions() onlyCreator`

This function requires that the Spokes and Routings have been set, but that permissions have not been set. Next, the functions _explicitly_ sets specific, design-intended permissions between the calling contracts and called contracts' functions. The function then finally sets the `permissionsSet` state variable to `true`. This function implements the `onlyCreator` modifier.
&nbsp;

`function vault() view returns (address)`

This view function returns the vault Spoke address.
&nbsp;

`function accounting() view returns (address)`

This view function returns the accounting Spoke address.
&nbsp;

`function priceSource() view returns (address)`

This view function returns the priceSource contract address.
&nbsp;

`function participation() view returns (address)`

This view function returns the participation Spoke address.
&nbsp;

`function trading() view returns (address)`

This view function returns the trading Spoke address.
&nbsp;

`function shares() view returns (address)`

This view function returns the shares Spoke address.
&nbsp;

`function policyManager() view returns (address)`

This view function returns the policyManager Spoke address.
&nbsp;

## Spoke.sol

#### Description

All Spoke contracts are registered with only one Hub and are initialized, storing the routes to all other Spokes registered with the Hub. Spokes serve as an abstraction of functionality and each Spoke has distinct and separate business logic domains, but may pragmatically access other Spokes registered with their Hub.
&nbsp;

#### Inherits from

DSAuth (link)

&nbsp;

#### On Construction

Sets the `hub` state variable and sets the the `hub` as the `authority` and `owner` as defined for permissioning within DSAuth.

&nbsp;

#### Structs

`Routes`

Member variables:

`address accounting`
`address feeManager`
`address participation`
`address policyManager`
`address shares`
`address trading`
`address vault`
`address priceSource`
`address canonicalRegistrar`
`address version`
`address engine`
`address mlnAddress`

This struct stores the contract addresses of the listed components and spokes.
&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

`modifier notShutDown()`

This modifier requires that the fund is not shut down and is evaluated prior to the execution of the functionality of the implementing function.
&nbsp;

`modifier onlyInitialized()`

This modifier requires that the `initialized` state variable is set to `true`.
&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`Hub public hub`

A variable of type `Hub`, defining the specific Hub contract to which the Spoke is connected.
&nbsp;

`Routes public routes`

A `Routes` struct storing all initialized Spoke and component addresses.
&nbsp;

`bool public initialized`

A boolean variable defining the initialization status of the Spoke.
&nbsp;

#### Public Functions

`function initialize(address[12] _spokes) public auth`

This function requires firstly that the Spoke not be initialized, then takes an array of addresses of all Spoke- and component contracts, sets all members of the `routes` struct state variable and finally sets `initialized` = `true` and the `owner` to address "0".
&nbsp;

`function engine() view returns (address)`

This view function returns the engine component contract address.
&nbsp;

`function mlnToken() view returns (address)`

This view function returns the MLN token contract address.
&nbsp;

`function priceSource() view returns (address)`

This view function returns the priceSource component contract address.
&nbsp;

`function version() view returns (address)`

This view function returns the version component contract address.
&nbsp;

---

## Fund Manager

### Overview

The Manager (Investment Manager) is the central actor in the Melon Fund. The Investment Manager is the creator of the Melon Fund and as such, the `owner` of the Melon Fund smart contract. In this capacity, the Investment Manager designs the initial set up of the Melon Fund, including:

- Compliance rules regarding Investor participation
- Risk Engineering rules regarding Investments guidelines
- Management and Performance fees
- Exchanges to be used
- Asset Tokens eligible for Subscription and Redemption
- Fund name

Once the Melon Fund is set up, it will be open for investment from Investors. When Investors transfer investment capital to the Melon Fund, the Investment Manager has full discretionary investment allocation authority over the assets, although Investors retain ultimate control over their respective shares in the Melon Fund. The discretionary investment allocation authority is, however, constrained by those Risk Engineering rules put in place by the Investment Manager during fund set up.

The critical point about a Melon Fund is that Investors retain control of their investment in the fund, while delegating asset management activity to the Investment Manager, who has the fine-grained trade and asset management authority, but no ability to remove asset tokens from the segregated safety of the Melon Fund's smart contact custody.

The Investment Manager address can only be the Investment Manager for a single Melon Fund per protocol Version, i.e. one address cannot manage multiple Melon Funds on a Version.

### Fund Interaction

The Investment Manager in the capacity of `owner` of the Melon Fund smart contract has the ability to interact with the following smart contract functions:

- `enableInvestment()`

- `disableInvestment)()`

- `shutDown()` - Fund shutdown

- `callOnExchange()` - Make-, Take-, and Cancel Order

Please refer to the Fund components documentation for further details on these functions.

---

## Investors

A Melon Fund is setup to serve Investors. That is, an Investment Manager creates a Melon Fund in order to provide the service of managing capital on behalf of participating Investors.

Investors are individuals or institutions participating in the performance of Melon Funds by sending cryptographic Asset tokens to the specific Melon Fund via the fund's `requestInvestment()` function.

In the context of the platform, the Investor is the Ethereum address from which the `requestInvestment()` was called and tokens sent (i.e. `msg.sender`).

### Investor Address

The individual Investor must retain the responsibility of secure custody of the Investor address's private key. The private key is required to sign valid Ethereum transactions which are ultimately responsible for calling specific functions and sending token amounts, covering the activities of investing, redeeming and redeeming by slice.

WARNING: Loss or poor security of the Investor address's private key will result in the irreversible and permanent loss of all crypto asset tokens associated with that address.

### Ethereum Addresses and Keys

An Ethereum address takes the hexidecimal format of 20 bytes prefixed by "0x". Ethereum addresses are all lowercase, but may be mixed case if a checksum is implemented. The private key is 256 random bits.

Example Ethereum Address:  
`0xdd4A3d9D44A670a80bAbbd60FD300a4C8C38561f`

Example Ethereum Private Key:
`d43689ae52d6a4e0e95d41bc638e95a66c4e7d0852f6db83d44c234ce9267d0d`

## Participation.sol

#### Description

The Participation contract encompasses the entire Melon fund interface to the investor and manages or delegates all functionality pertaining to fund subscription and redemption activities including the creation/destruction of Melon fund shares, fee calculations, asset token transfers and enabling/disabling specific asset tokens for subscription.

#### Inherits from

ParticipationInterface, TokenUser, AmguConsumer, Spoke (links)

#### On Construction

The contract requires the hub address and an array of default asset addresses. These inputs set the hub and enable investment for the assets passed in to the constructor.

The Participation contract is created from the ParticipationFactory contract, which creates a new instance of `Participation` given the `hub` address, registering the address of the newly created Participation contract as a child of the ParticipationFactory.

#### Structs

`Request`

Member variables:

`address investmentAsset` - The address of the asset token with which the subscription is made.
`uint investmentAmount` - The quantity of tokens of the `investmentAsset`
`uint requestedShares` - The quantity of fund shares for the request
`uint timestamp` - The timestamp of the block containing the subscription transaction
`uint atUpdateId` - [CHECK]

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`uint constant public SHARES_DECIMALS = 18`

An integer constant which specifies the decimal precision of a single share. The value is set to 18.
&nbsp;

`uint constant public INVEST_DELAY = 10 minutes`

An integer constant which specifies the number of seconds a valid subscription request must be delayed before a subscription is executed.
&nbsp;

`uint constant public REQUEST_LIFESPAN = 1 days`

An integer constant which specifies the time duration where an investment subscription request is live. This constant is set to 1 day in seconds.
&nbsp;

`uint constant public REQUEST_INCENTIVE = 10 finney`

An integer constant which specifies the amount for the subscription request provided as an incentive for triggering the delayed subscription request.
&nbsp;

`mapping (address => Request) public requests`

A public mapping associating an investor’s address to a `Request` struct.
&nbsp;

`mapping (address => bool) public investAllowed`

A public mapping which specifies all asset token addresses which have been enabled for subscription to the Melon fund.
&nbsp;

`mapping (address => mapping (address => uint)) public lockedAssetsForInvestor`

A public (compound) mapping associating a subscription asset token address to an investor address to a quantity of the asset token. This mapping is used by the contract to lock and hold the investor's subscription token until the request can be either executed or cancelled.

&nbsp;

#### Public Functions

`function() public payable`

This public function is the contract's fallback function enabling the contract to receive ETH sent directly to the contract for the `REQUEST_INCENTIVE`.

`function enableInvestment(address[] _assets) public auth`

This function requires that the caller is the `owner` or the manager or the current contract. The function iterates over an array of provided asset token addresses, ensuring each is registered with the Melon fund's PriceFeed, and ensures that registered asset token addresses are set to `true` in the `investAllowed` mapping. Finally the function emits the `EnableInvestment()` event, logging the `_assets`.
&nbsp;

`function disableInvestment(address[] _assets) public auth`

This function requires that the caller is the `owner` or the manager or the current contract. The function iterates over an array of provided asset token addresses and ensures that asset token addresses are set to `false` in the `investAllowed` mapping. Finally the function emits the `DisableInvestment()` event, logging the `_assets`.
&nbsp;

`function requestInvestment(uint requestedShares, uint investmentAmount, address investmentAsset) external notShutDown payable amguPayable(true) onlyInitialized`

This function ensures that the fund is not shutdown and that subscription is permitted in the provided `ìnvestmentAsset`. The function then creates and populates a `Request` struct (see details above) from the function parameters provided and adds this to the `requests` mapping corresponding to the `msg.sender`. Finally, this function emits the `InvestmentRequest` event. Execution of this functions requires payment of AMGU ETH to the Melon Engine to provide the investment request execution incentive. This function implements the `notShutDown`, `payable`, `amguPayable` and `onlyInitialized` modifiers.
&nbsp;

`function cancelRequest() external payable amguPayable(0)`

This function removes the request from the `requests` mapping for the request corresponding to the `msg.sender` address. The function requires that a request exists and that at least one of the conditions to cancel an investment request are met: invalid investment asset price, expired request or fund is shut down. Investment asset tokens are transferred back to the investor address. Finally, the function emits the `CancelRequest` event, logging `msg.sender`. The function is `payable` and also implements the `amguPayable` modifier, requiring amgu payment. Here, the `deductFromRefund` parameter is set to `0`.
&nbsp;

`function executeRequestFor(address requestOwner) public notShutDown amguPayable(0) payable`

This function:
ensures that the fund is not shutDown,
ensures that `requestOwner` has a valid subscription request,
ensures that the subscription asset has a valid price,
triggers a management fee calculation and reward,
gets the current share cost in terms of the subscription asset,
ensures that the total share cost <= subscription amount,
transfers the subscription assets to the fund vault,
calculates change (remainder value) and returns any non-zero asset quantity to the investor address,
reset the `lockedAssetsForInvestor` mapping to "0" for the investor address,
transfers the REQUEST_INCENTIVE to `msg.sender`,
creates the new shares and allocates them to the investor address,
adds the subscription asset token to the Melon fund's list of owned assets,
emits the RequestExecution() event logging `requstOwner`, `msg.sender`, `investmentAsset`, `investmentAmount` and `requestedShares`.
Finally, the function removes the `Request` from the `requests` mapping for the `requestOwner` address. The function is `payable` and also implements the `amguPayable` modifier, requiring amgu payment. Here, the `deductFromRefund` parameter is set to `0`.
&nbsp;

`function getOwedPerformanceFees(uint shareQuantity) view returns (uint remainingShareQuantity)`

This view function calculates and returns the quantity of shares owed for payment of accrued performance fees given the provided quantity of shares being redeemed.
&nbsp;

`function redeem() public`

This function determines the quantity of shares owned by `msg.sender` and calls `redeemQuantity()`. A share-commensurate quantity of all token assets in the fund are transferred to `msg.sender`, i.e. the investor. This function redeems _all_ shares owned by `msg.sender`.
&nbsp;

`function redeemQuantity(uint shareQuantity) public`

This function allows the investor to redeem a specified quantity of shares held by the `msg.sender`, i.e. the investor.
&nbsp;

`function redeemWithConstraints(uint shareQuantity, address[] requestedAssets) public`

This function:
ensures that the requested redemption share quantity is less than or equal to the share quantity owned by the investor (`msg.sender`),
ensures that management fee shares have been calculated and allocated immediately prior to redemption,
ensures that accrued performance fee shares have been calculated, allocated to the manager, removed from the investor's balance and added to total share supply immediately prior to redemption,
maintains parity between `requestedAssets[]` array parameter and the `redeemedAssets[]` internal array, ensuring that individual asset tokens are only redeemed once,
calculates the proportionate quantity of each token asset owned by the investor (`msg.sender`) based on share quantity,
destroys the investor's shares and reduces the fund's total share supply by the same amount,
safely transfers all token assets in the correct quantities to the `msg.sender` (investor) address using the ERC20 `transfer()` function,
and finally, the function emits the `SuccessfulRedemption()` event along with the quantity of successfully redeemed shares.
&nbsp;

`function hasRequest(address _who) view returns (bool)`

This view function returns a boolean indicating whether the provided address has a corresponding active request with a positive quantity of requested shares in the `requests` mapping.
&nbsp;

`function hasValidRequest(address _who) public view returns (bool)`

This public view function returns a boolean indicating the `_who` address parameter has a valid request, meaning that either this is the address's first subscription or the INVEST_DELAY is respected, the subscription amount is greater than "0" and the requested share quantity is greater than "0".
&nbsp;

`function hasExpiredRequest(address _who) view returns (bool)`

This view function returns a boolean indicating that the address provided has a `requests` entry which is currently older than `REQUEST_LIFESPAN`.
&nbsp;

---

## Shares

## Shares.sol

#### Description

In fund management, shares assign proportionate ownership of the collective underlying assets held in the fund. Each share in a fund has equal claim to the fund's assets. Shareholders then have a claim to the underlying fund assets in proportion to the quantity of shares under their control, i.e. ownership.

In a Melon fund, fund shares are represented as ERC20 tokens. Each token has equal and proportionate claim to the underlying assets. The share tokens are fungible but are not transferrable [CHECK if this will remain so.] between owners.

As subscribed capital enters the fund, the Shares contract will create a proportionate quantity of share tokens based on the current market prices of the fund's underlying assets and the market price of the subscribed capital and assign allocate these newly created share tokens to the subscribing address.

When share tokens are redeemed. A proportionate quantity of the fund's underlying asset tokens are transferred to the redeeming address and the redeeming address's redeemed share tokens are destroyed (i.e. set to 0) and the fund's total share quantity is reduced by that same quantity.

#### Inherits from

Spoke, StandardToken (links)

&nbsp;

#### On construction

The Shares contract requires the address of the `hub` and becomes a `spoke` of the `hub`. The Shares contract describes the Melon fund's shares as _ERC20 tokens_. The share tokens take their name as defined by the `hub`. The share token's symbol and decimals are hardcoded in the contract.

The Shares contract is created from the sharesFactory contract, which creates a new instance of `Shares` given the `hub` address, register the address of the newly created shares contract as a child of the sharesFactory.
&nbsp;

#### Public State variables

`string public symbol`

A public string variable denoting the Melon fund's share token symbol, currently defaulting to "MLNF".
&nbsp;

`string public name`

A public string variable denoting the Melon fund's name as specified by the manager at set up.
&nbsp;

`uint8 public decimals`

A public integer variable storing the decimal precision of the Melon fund's share quantity, currently defaulting to 18, the same decimal precision as ETH and many other ERC20 tokens.
&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Public Functions

`function createFor(address who, uint amount) auth`

This function requires that the caller is the `owner` or the participation component contract or the fee manager component contract or the current contract. This function calls internal function `_mint()`, which increases both `totalSupply` and the `balance` of the address by the same quantity.
&nbsp;

`function destroyFor(address who, uint amount) auth`

This function requires that the caller is the `owner` or the participation component contract or the current contract. This function calls internal function `_burn()`, which decreases both `totalSupply` and the `balance` of the address by the same quantity.
&nbsp;

#### Reverting functions:

create
`function transfer(address to, uint amount) public returns (bool)`
&nbsp;

`function transferFrom(address from, address to, uint amount) public returns (bool)`
&nbsp;

`function approve(address spender, uint amount) public returns (bool)`
&nbsp;

`function increaseApproval(address spender, uint amount) public returns (bool)`
&nbsp;

`function decreaseApproval(address spender, uint amount) public returns (bool)`
&nbsp;

---

## Vault

## General

## Vault.sol

#### Description

The Vault makes use of the `auth` modifier from the DSAuth dependency. The `auth` functionality ensures the precise provisioning of call permissions, and focuses on granting `owner` or the current contract call permissions.

The Vault contract is created from the vaultFactory contract, which creates a new instance of the `Vault` given the `hub` address, register the address of the newly created vault contract as a child of the vaultFactory and finally emits an event, broadcasting the creation event along with the address of the vault contract.

&nbsp;

##### Inherits from

VaultInterface, TokenUser, Spoke (link)

&nbsp;

##### On construction

Requires the `Hub` address and uses this to instantiate itself as a `Spoke`.
&nbsp;

#### Structs

None

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

None.

&nbsp;

#### Public Functions

`function withdraw(address token, uint amount) external auth`

This external function requires that the caller is the `owner` or the current contract. This function calls the `safeTransfer()` function from `TokenUser.sol`, safely transferring ownership of the provided amount from the vault to the custody of `msg.sender`.

&nbsp;

---

## Accounting

## General

## Accounting.sol

#### Description

The Accounting contract defines the accounting rules implemented by the fund. All operations concerning the underlying fund positions, fund position maintenance, asset token pricing, fees, Gross- and Net Asset value calculations and per-share calculations come together in this contract's business logic.
&nbsp;

#### Inherits from

AccountingInterface, AmguConsumer, Spoke (links)

&nbsp;

#### On Construction

The contract requires the hub address, the denomination asset address, the native asset address and an array of default asset addresses. These inputs set the accounting spoke's denomination asset, denomination asset decimals, the default share price (1.0 in denomination asset terms) and add all default assets passed in to the `ownedAssets` array state variable.

The Accounting contract is created from the AccountingFactory contract, which creates a new instance of `Accounting` given the `hub` address, registering the address of the newly created Accounting contract as a child of the AccountingFactory.
&nbsp;

#### Structs

`Calculations`

Member Variables:

`uint gav` - The Gross Asset Value of all fund positions  
 `uint nav` - The Net Asset Value of all fund positions  
 `uint allocatedFees` - Fee shares accrued since the previous fee calculation about to be allocated
`uint totalSupply` - The quantity of fund shares  
 `uint timestamp` - The timestamp of the current transactions block
&nbsp;

#### Enums

None.

#### Modifiers

None.

#### Public State Variables

`uint constant public MAX_OWNED_ASSETS = 20`

A constant integer determining the maximum quantity of individual asset token positions able to be held at any point in time by a Melon fund. This constant is set to "20".
&nbsp;

`address[] public ownedAssets`

A pubic array containing the addresses of tokens currently held by the fund.
&nbsp;

`mapping (address => bool) public isInAssetList`

A mapping defining the status of an asset's membership in the `ownedAssets` array.
&nbsp;

`uint public constant SHARES_DECIMALS = 18`

A public constant representing the decimal precision of Melon fund share token.
&nbsp;

`address public NATIVE_ASSET`

The address of the asset token native to the platform.
&nbsp;

`address public DENOMINATION_ASSET`

The address of the token defined to be the denomination asset, or base currency of the fund. NAV, performance and all fund-level metrics will be denominated in this asset.
&nbsp;

`uint public DENOMINATION_ASSET_DECIMALS`

An integer determining the decimal precision, or the degree of divisibility, of the denomination asset.
&nbsp;

`uint public DEFAULT_SHARE_PRICE`

An integer determining the initial "sizing" of one share in the fund relative to the denomination asset. The share price at fund inception will always be one unit of the denomination asset.
&nbsp;

`Calculations public atLastAllocation`

A `Calculations` structure holding the latest state of the member fund calculations described above.
&nbsp;

#### Public functions

`function getOwnedAssetsLength() view returns (uint)`

This public view function returns the length of the `ownedAssets` array state variable.

`function getFundHoldings() returns (uint[], address[])`

This function returns the current quantities and corresponding addresses of the funds token positions as two distinct order-dependent arrays.
&nbsp;

`function calcAssetGAV(address _queryAsset) returns (uint)`

This function calculates and returns the current fund position GAV (in denomination asset terms) of the individual asset token as specified by the address provided.
&nbsp;

`function assetHoldings(address _asset) public returns (uint)`

This function returns the fund position quantity of the asset token as specified by the address provided.
&nbsp;

`function calcGav() public returns (uint gav)`

This function calculates and returns the current Gross Asset Value (GAV) of all fund assets in denomination asset terms.
&nbsp;

`function calcNav(uint gav, uint unclaimedFees) public pure returns (uint)`

This function calculates and returns the fund's Net Asset Value (NAV) given the provided fund GAV and current quantity of unclaimed fee shares.
&nbsp;

`function valuePerShare(uint totalValue, uint numShares) view returns (uint)`

This function calculates and returns the value (in denomination asset terms) of a single share in the fund given the fund total value and the total number of shares provided.
&nbsp;

`function performCalculations() returns (uint gav, uint unclaimedFees, uint feesInShares, uint nav, uint sharePrice)`

This view function returns bundled calculations for GAV, NAV, unclaimed fees, fee share quantity and current share price (in denomination asset terms).
&nbsp;

`function calcSharePrice() returns (uint sharePrice)`

This function calculates and returns the current price (in denomination asset terms) of a single share in the fund.
&nbsp;

`calcGavPerShareNetManagementFee() returns (uint gavPerShareNetManagementFee)`

This function calculates and returns the GAV (in denomination asset terms) of a single share in the fund net of the Management Fee due.
&nbsp;

`function getShareCostInAsset(uint _numShares, address _altAsset) returns (uint)`

This public function calculates and returns the quantity of the `_altAsset` asset token commensurate with the value of `_numShares` quantity of the Melon fund's shares.
&nbsp;

`function triggerRewardAllFees() public amguPayable(0) payable`

This public function updates `ownedAssets` and rewards all fees accrued to the current point in time. The function then updates the `atLastAllocation` struct state variable. The function is `payable` and also implements the `amguPayable` modifier, requiring amgu payment. Here, the `deductFromRefund` parameter is set to `0`.
&nbsp;

`function updateOwnedAssets() public`

This function maintains the `ownedAssets` array by removing or adding asset addresses as the fund holdings composition changes.
&nbsp;

`function addAssetToOwnedAssets(address _asset) public auth`

This function requires that the caller is the `owner` or the trading component contract or the participation component contract or current contract. The function maintains the `ownedAssets` array and the `isInOwedAssets` mapping by adding asset addresses as the fund holdings composition changes.
&nbsp;

`function removeFromOwnedAssets(address _asset) public auth`

This function requires that the caller is the `owner` or the trading component contract or the current contract. The function maintains the `ownedAssets` array and the `isInOwedAssets` mapping by removing asset addresses as the fund holdings composition changes.
&nbsp;

---

## Fees

## General

Fees are charges levied against a Melon fund's assets for:

- management services rendered, in the form of Management Fees
- fund performance achieved, in the form of Performance Fees.

Legacy investment funds are typically run as legal vehicle; a business with income and expenses. As a traditional fund incurs expenses (not only those listed above, but others like administration-, custody-, directors-, audit fess), traditional funds must liquidate assets to cash or pay the expenses out of cash held by the fund. This necessarily reduces the assets of the fund and incurs further trading costs.

Melon funds employ a novel and elegant solution: Instead of using cash, or liquidating positions to pay fees, the Melon fund smart contract itself calculates and creates new shares (or share fractions), allocating them to the investment manager's own account holdings within the fund as payment.

At that point, the investment manager can continue holding an increased number of shares in their own fund, or redeem the shares to pay their own operating costs, expand their research activities or whatever else they deem appropriate. The Melon Fund smart contract autonomously and verifiably maintains the shareholder accounting impact in a truly reliable and transparent manner.

Paying fees in this manner has a few interesting side-effects: Assets do not leave the fund, as would a cash payment. There are no unnecessary trading or cash management transactions. Investors and managers have access to real-time fee accrual metrics. Finally, the incentives to the manager are reinforced beyond a cash performance fee by being paid in the currency that is their own product.

The Management Fee and Performance Fee are each represented by an individual contract. The business logic and functionality of each fee is defined within the respective contract. The contract will interact with the various components of the fund to calculate and return the quantity of shares to create. This quantity is then added to the Manager address balance and to the total supply balance.

The Fee Manager component manages the individual fee contracts which have been configured for the Melon fund at set up. The core of the Fee Manager is an array of Fee contract instances. Functions exist to prime this array at fund setup with the selected and configured fee contracts, as well as basic query functionality to aid the user interface in calculating and representing the share NAV.

Fees are calculated and allocated when fund actions such as subscribe, redeem or claim fees are executed by a participant. Calling `rewardAllFees()` will iterate over the array of registered fees in the fee manager, calling each fee's calculation logic. This returns the quantity of shares to create and allocate to the manager.

It is important to note that fee calculations take place before the fund's share quantity is impacted by subscriptions or redemptions. To this end, when a subscription or redemption action is initiated by an investor, the execution order first calculates fee amounts and creates the corresponding share quantity, as the elapsed time and share quantity at the start is known. Essentially, the Melon fund calculates and records a reconciled state immediately and in the same transaction where share quantity changes due subscription/redemption.

## Management Fees

Management Fees are earned with the passage of time, irrespective of performance. The order of fee calculations is important. The Management Fee share quantity calculation is a prerequisite to the Performance Fee calculation, as fund performance must be reduced by the Management Fee expense to fairly ascertain net performance.

The Management Fee calculation business logic is fully encapsulated by the Management Fee contract. This logic can be represented as follows.

First, the time-weighted, pre-dilution share quantity is calculated:
&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;PD_{f}$=($T_{n}$)($\frac{t_{e}}{t_{y}}$)($f_{m})"/>

then, this figure is scaled such that investors retain their original share holdings quantity, but newly created shares represent the commensurate fee percentage amount:
&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;SMF_{e}=\frac{PD_{f}T_{n}}{T_{n}-PD_{f}}"/>

where,

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;PD_{f}"/> = pre-dilution quantity of shares

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;T_{n}"/> = current shares outstanding

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;t_{e}"/> = number of seconds elapsed since previous conversion event

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;t_{y}"/> = number of seconds in a year ( = 60 _ 60 _ 24 \* 365 )

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;f_{m}"/> = Management Fee rate

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;SMF_{e}"/> = number shares to create to compensate Management Fees earned during the conversion period
&nbsp;

## Performance Fees

Performance Fees accrue over time with performance, but can only be harvested after regular, pre-determined points in time. This period is referred to as the Measurement Period and is decided by the fund manager and configured at fund set up.

Performance is assessed at the end of the Measurement Period by comparing the fund's current share price net of Management Fees to the fund's current high-water mark (HWM).

The HWM represents the highest share valuation which the Melon fund has historically achieved _at a Measurement Period ending time_. More clearly, it is not a fund all-time-high, but rather the maximum share valuation of all Measurement Period-end snapshot valuations.

If the difference to the HWM is positive, performance has been achieved and a Performance Fee is due to the fund manager. This calculation is straightforward and is the aforementioned difference multiplied by the Performance Fee rate. The Performance Fee is _not_ an annualized fee rate.

The calculation of the Performance Fee requires that, at that moment, no Management Fees are due. This will be true as the code structure always invokes the Management Fee calculation and allocation immediately preceding, and in the same transaction, the Performance Fee calculation.

The Performance Fee calculation business logic is fully encapsulated by the Performance Fee contract. This logic can be represented as follows.
&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?HWM_{MP}$=\begin{cases}S_{n},&S_{n}>HWM_{MP-1}\\HWM_{MP-1},&S_{n}\leq{HWM_{MP-1}}\end{cases}"/>

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?P_{MP}$=\begin{cases}GAV_{s}-HWM_{MP-1},&GAV_{s}>HWM_{MP-1}\\0,&GAV_{s}\leq{HWM_{MP-1}}\end{cases}"/>

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?PD_{f}=\frac{P_{MP}T_{n}^2f_{p}}{GAV}"/>

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?SPF_{e}=\frac{PD_{f}T_{n}}{T_{n}-PD_{f}}"/>

where,

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?HWM_{MP}"/> = high-water mark for the Measurement Period

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?S_{n}"/> = current share price net of Management Fee

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?GAV"/> = current fund Gross Asset Value

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?GAV_{s}=\frac{GAV}{T_{n}}"/> = current fund GAV per share

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?P_{MP}"/> = performance for the Measurement Period

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?PD_{f}"/> = pre-dilution quantity of shares

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?\Large&space;T_{n}"/> = current shares outstanding

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?f_{p}"/> = Performance Fee rate

&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://ibm.codecogs.com/svg.latex?SPF_{e}"/> = number shares to create to compensate Performance Fees earned during the conversion period
&nbsp;

While Performance Fees are only crystalized at the end of each measurement period, there must be a mechanism whereby redeeming investors compensate for _their_ current share of accrued performance fees prior to redemption.

In the case where an investor redeems prior to the current Measurement Period's end and where the current share price exceeds the fund HWM, a Performance Fee is due. The Redemption business logic calculates the accrued Performance Fee for the entire fund at the time of redemption and weights this by the redemption share quantity's proportion to the fund's total share quantity. The resulting percentage is the proportion of the redemption share quantity due as the redeeming investor's Performance Fee payment. This quantity is deducted from the redeeming share quantity and credited to the manager's share account balance. The remaining redeeming share quantity is destroyed as the proportionate individual token assets are transferred out of the fund and the fund's total share quantity and the investor's share quantity is reduced by this net redeeming share quantity.

**Note on Fee Share Allocation**

The intended behavior is for the Manager to immediately redeem fee shares as they are created. This will ensure a fair and precise share allocation. The implemented code that represents the fee calculations above contain smart contract optimizations for state variable storage. By not immediately redeeming fee shares, a negligible deviation to the manager fee payout will arise due to this optimization.

## FeeManager.sol

#### Description

The Fee Manager is a spoke which is initialized and permissioned in the same manner as all other spokes. The Fee Manager registers and administers the execution order of the individual fee contracts.

##### Inherits from

Spoke, DSMath (link)

&nbsp;

##### On Construction

The FeeManager contract constructor requires the hub address, the denomination asset token contract address, an array of addresses representing fee contract addresses, an array of integers representing corresponding fee rates, an array of integers representing corresponding performance fee periods and the address of the registry contract.

&nbsp;

##### Structs

None.

&nbsp;

##### Enums

None.

&nbsp;

##### Events

FeeReward(uint shareQuantity)

This event is triggered when a Fee has been successfully allocated to the fund manager. The event logs the quantity of new fee shares created.
&nbsp;

FeeRegistration(address fee)

This event is triggered when a Fee contract is registered with the FeeManager contract. The event logs the Fee contract address.
&nbsp;

##### Public State Variables

`Fee[] public fees`

An array of type fees storing the defined fees.
&nbsp;

`mapping (address => bool) public feeIsRegistered`

A mapping storing the registration status of a fee address.
&nbsp;

##### Public Functions

`function register(address feeAddress, uint feeRate, uint feePeriod, address denominationAsset) internal`

This function adds the feeAddress provided to the `fees` array and sets the `feeIsRegistered` mapping for that address to `true`. The fee contract is initialized with the `feeRate`, `feePeriod` and `denominationAsset` token address. Finally, the FeeRegistration event is emitted.
&nbsp;

`function totalFeeAmount() public view returns (uint total)`

This function returns the total amount of fees incurred for the hub.
&nbsp;

`function rewardAllFees() public auth`

This function requires that the caller is the `owner` or the accounting component contract or the current contract. This function creates shares commensurate with all fees stored in the `Fee[]` state variable array.
&nbsp;

`function rewardManagementFee() public`

This public function calculates, creates and allocates the quantity of shares currently due as the management fee.
&nbsp;

`function managementFeeAmount() public view returns (uint)`

This public view function calculates and returns the quantity of shares currently due as the management fee.
&nbsp;

`function performanceFeeAmount() public view returns (uint)`

This public view function calculates and returns the quantity of shares currently due as the performance fee.
&nbsp;

## ManagementFee.sol

#### Description

The ManagementFee contract contains the complete business logic for the creation of fund shares based on assets managed over a specified time period.
&nbsp;

#### Inherits from

Fee and DSMath (link)

&nbsp;

#### On Construction

None.

&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Public State Variables

`uint public DIVISOR = 10 ** 18`

An integer defining the standard divisor. The variable is set to 10^18.
&nbsp;

`mapping (address => uint) public managementFeeRate`

An public mapping associating fund address with the management fee percentage rate.
&nbsp;

`mapping (address => uint) public lastPayoutTime`

An public mapping associating fund address with the block time in UNIX epoch seconds when the previous fee payout was executed for this fund.
&nbsp;

#### Public Functions

`function feeAmount(address hub) public view returns (uint feeInShares)`

This function calculates and returns the number of shares to be created given the amount of time since the previous fee payment, asset value and the defined management fee rate.
&nbsp;

`function initializeForUser(uint feeRate, uint feePeriod, address denominationAsset) external`

This function ensures that no previous fee payout to the manager address on this Version has been affected. The manager address then receives a fee rate- and timestamp entry in the `managementFeeRate` and `lastPayoutTime` mappings, respectively.
&nbsp;

`function updateState(address hub) external`

This function sets `lastPayoutTime` to the current block timestamp.
&nbsp;

## PerformanceFee.sol

#### Description

The PerformanceFee contract contains the complete business logic for the creation of fund shares based on fund performance over a specified Measurement Period and relative to the fund-internally-defined HWM.

#### Inherits from

Fee, DSMath (link)

&nbsp;

#### On Construction

None.

&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Events

`HighWaterMarkUpdate(address indexed feeManager, uint indexed hwm)`

This event is triggered when the Melon fund's high-watermark is updated with a new value. The event logs the fee manager contract address and the new high-watermark value.
&nbsp;

#### Public State Variables

`uint public constant DIVISOR = 10 ** 18`

A constant integer defining the standard divisor.
&nbsp;

`uint public constant INITIAL_SHARE_PRICE = 10 ** 18`

A constant integer defining the initial share price to "1".
&nbsp;

`uint public constant REDEEM_WINDOW = 1 weeks`

A constant integer defining a window of time after a measurement period where a performance fee due can be harvested. The constant is set to one week (in seconds).
&nbsp;

`mapping(address => uint) public initializeTime`

A public mapping associating the Melon fund address address to the Melon fund's initialization block time.
&nbsp;

`mapping(address => uint) public highWaterMark`

A public mapping associating the Melon fund address to the Melon fund's `highWaterMark`, which defines the asset value which must be exceeded at the measurement period's end to facilitate the determination of the performance fee due to the manager.
&nbsp;

`mapping(address => uint) public lastPayoutTime`

A public mapping associating the Melon fund address to the Melon fund's last previous time a performance fee calculation and payout was executed. It is an integer defining the block time in UNIX epoch seconds when the previous fee payout was executed.
&nbsp;

`mapping(address => uint) public performanceFeeRate`

A public mapping associating the Melon fund address to the Melon fund's configured performance fee rate.
&nbsp;

`mapping(address => uint) public performanceFeePeriod`

A public mapping associating the Melon fund address to the Melon fund's configured performance measurement period. This integer express the performance measurement period in seconds.
&nbsp;

#### Public Functions

`function initializeForUser(uint feeRate, uint feePeriod, address denominationAsset) external`

This external function is executed once at the initialization of the Melon fund. The function sets the Melon fund's performance fee rate, the performance measurement period, the fund denomination asset, the initial high-watermark, the `initializeTime` and the `lastPayoutTime`.

`function feeAmount() public view returns (uint feeInShares)`

This function calculates and returns the number of shares to be created given the fund performance since the previous measurement period payment, asset value and the defined performance fee rate.
&nbsp;

`function updateState() external`

This function sets the `highwatermark` and `lastPayoutTime` if applicable. The function requires the `canUpdate()` function to return `true`. Finally, the function emits the `HighWaterMarkUpdate()` event logging the fee manager address and the fund's new high-watermakr, i.e. the new GAV per share value.
&nbsp;

`function canUpdate(address _who) public view returns (bool)`

This public view function returns a boolean indicating whether conditions are met to trigger the collection of a performance fee. Theses conditions are: the current time is within the update window and that an update has not already been performed for the current measurement period.
&nbsp;

---

## Trading

The Melon Protocol integrates with decentralized exchanges to facilitate the trading of Assets, one of the essential functionalities of a Melon Fund. This means that a Melon Fund must accommodate multiple different decentralized exchanges smart contracts if the fund is to draw from a wider pool of liquidity.

## Trading.sol

#### Description

The Trading contract is created by the tradingFactory, which holds a queryable array of all created Trading contracts and emits the `instanceCreated` event along with the trading contract's address.

The trading contract:

-manages the set up of the interface infrastructure to the various selected exchanges.

-manages open make orders that the fund has submitted to the various registered exchanges.

-houses the common function for calling specific exchange functions through each exchange's respective exchange adapters. Note that each exchange will have their own unique interface and required parameters, which the adapters accommodate.

-provides a public function to transfer an array of token assets to the fund's vault.

-provides view functions to get order information regarding specific assets on specific exchanges as well as specific order detail.

The contract has a fallback function to retrieve ETH.

Inherits from
Spoke, DSMath, TradingInterface, TokenUser (link)

On Construction

The contract requires the hub address, an array of exchange addresses, an array of exchange adapter addresses, an array of booleans indicating if an exchange takes custody of tokens for make orders and the address of the Registry contract. The contract becomes a spoke to the hub. The constructor of the trading contract requires that the length of the exchange array matches the length of exchange adapter array and the length of the exchange array matches the length of boolean custody status array. Finally, the constructor builds the public variable `exchanges[]` array of `Exchange` structs.
&nbsp;

#### Structs

`Exchange`

Member Variables

`address exchange` - The address of the decentralized exchange smart contact.

`address adapter` - The address of the corresponding adapter smart contract.

`bool takesCustody` - An flag specifying whether the exchange holds the asset token in its own custody for make orders.
&nbsp;

`Order`

Member Variables

`address exchangeAddress` - The address of the decentralized exchange smart contact.

`bytes32 orderId` - A unique identifier for a specific order on the specific exchange.

`UpdateType updateType` - A struct indicating the update behavior/action of the order. Permitted values are `make`, `take` and `cancel`.

`address makerAsset` - The address of the asset token owned and provided in exchange.

`address takerAsset` - The address of the asset token to be received in exchange.

`uint makerQuantity` - An integer representing the quantity of the asset token owned and provided in exchange.

`uint takerQuantity` - An integer representing the quantity of the asset token owned and provided in exchange.

`uint timestamp` - The timestamp of the block in which the submitted order transaction was mined.

`uint fillTakerQuantity` - An integer representing the quantity of the maker asset token traded in the make order. This value is not necessarily the same as the `makerQuantity` because taker participants can choose to only partially execute the order, i.e. take a lower quantity of the `makerAsset` token than specified by the order's `makerQuantity`.
&nbsp;

`OpenMakeOrder`

Member Variables

`uint id` - An integer originating from the exchange which uniquely identifies the order within the exchange.

`uint expiresAt` - An integer representing the time when the order expires. The timestamp is represented by the Ethereum blockchain as a UNIX Epoch. After order expiration, the order will no longer [exist/be active] on the exchange and custody, if the exchange held custody, returns from the exchange contract to the fund contract.

`uint orderIndex` - An integer representing the index of the order in the `orders` array.

`address buyAsset` - An address representing the token contract of the buy asset in the order.
&nbsp;

#### Enums

`UpdateType` - An enum which characterizes the type of update to the order.

Member Types

`make` - Indicates a make order type, where a quantity of a specific asset is offered at a specified price.

`take` - Indicates a take order type, where the offered quantity (or less) of a specific asset is accepted at the specified price by the counterparty. Taking less than the offered quantity in the order would be considered a "partial fill" of the order.

`cancel` - A type indicating that the update performed will cancel the order.
&nbsp;

#### Public State variables

`Exchange[] public exchanges`

A public array of `Exchange` structs which stores all exchanges with which the fund has been initialized.
&nbsp;

`Order[] public orders`

A public array of `Order` structs which stores all active orders [CHECK] on the
&nbsp;

`mapping (address => bool) public adapterIsAdded`

A public mapping which indicates that a specific exchange adapter (as identified by the adapter address) is registered for the fund.
&nbsp;

`mapping (address => mapping(address => OpenMakeOrder)) public exchangesToOpenMakeOrders`

A public compound mapping associating an exchange address to open make orders from the fund on the specific exchange.
&nbsp;

`mapping (address => uint) public openMakeOrdersAgainstAsset`

A public mapping associating an asset token contract address to the quantity of open make orders for that asset token.
&nbsp;

`mapping (address => bool) public isInOpenMakeOrder`

A public mapping indicating that the specified token asset is currently offered in an open make order on an exchange.
&nbsp;

`mapping (bytes32 => LibOrder.Order) public orderIdToZeroExOrder`

A public mapping a ZeroEx order identifier to a ZeroEx Order struct.
&nbsp;

`uint public constant ORDER_LIFESPAN = 1 days`

A public constant specifying the number of seconds that an order will remain active on an exchange. This number is added to the order creation date's timestamp to fully specify the order's expiration date. `1 days` is equal to 86400 ( 60 _ 60 _ 24 ).
&nbsp;

#### Modifiers

`delegateInternal()`

A modifier which requires that the caller (`msg.sender`) is the current contract `Trading.sol` before the implementing function executes its functionality. This ensures that only the current contract can call a function implementing this modifier.
&nbsp;

#### Public functions

`function() public payable`

The contracts fallback function making the contract able to receive ETH sent to the contract without a funciton call.
&nbsp;

`function isOrderExpired(address exchange, address asset) public view returns (bool)`

This public view function returns a boolean indicating whether an order for the asset token and exchange provided is currently expired.
&nbsp;

`function addExchange(address _exchange, address _adapter, bool _takesCustody) internal`

This is an internal function which is called for each exchange address passed to the constructor, adding the full exchange struct to the exchanges array state variable. The function ensures that an exchange has not previously been registered.
&nbsp;

`function callOnExchange( uint exchangeIndex, string methodSignature, address[6] orderAddresses, uint[8] orderValues, bytes32 identifier, bytes makerAssetData, bytes takerAssetData, bytes signature ) public onlyInitialized`

This is the fund's general interface to each registered exchange for trading asset tokens. The client will call this function for specific exchange/trading interactions. This function first calls the policyManager to ensure that function-specific policies are pre- or post-executed to ensure that the exchange trade adheres to the policies configured for the fund. This function implements the `onlyInitialized` modifier. Finally, the function emits the `ExchangeMethodCall()` event, logging the specified parameters.
&nbsp;

`function addOpenMakeOrder( address ofExchange, address sellAsset, address buyAsset, uint orderId, uint expirationTime ) public delegateInternal`

This public function can only be called from within the current contract. The function ensures that the sell asset token does not already have a current open sell order and that there are one or more orders in the `orders` array. If the `expirationTime` is set to "0", the order's expiration time is set to `ORDER_LIFESPAN`. The expiration time is required to be greater that the current `block.timestamp` and less than or equal to the sum of the `ORDER_LIFESPAN` and `block.timestamp`. The function then sets the `isInOpenMakeOrder` mapping for the sell asset token address to `true`, and sets the details of the address's `openMakeOrder` struct on the contracts `exchangesToOpenMakeOrders` mapping.
&nbsp;

`function removeOpenMakeOrder( address exchange, address sellAsset ) public delegateInternal`

This public function can only be called from within the current contract. The function removes the provided sell asset token address entry for the provided exchange address.
&nbsp;

`function orderUpdateHook( address ofExchange, bytes32 orderId, UpdateType updateType, address[2] orderAddresses, uint[3] orderValues ) public delegateInternal`

This public function can only be called from within the current contract. The function used the input parameters and the current execution block's timestamp to push make- or take orders to the `orders` array. [Why only make or take orders??]
&nbsp;

`function updateAndGetQuantityBeingTraded(address _asset) public returns (uint)`

This public function returns the sum of the quantity of the provided asset token address held by the current contract and the quantity of the provided asset token held across all registered exchanges in the fund's make orders. The sum returned excluded quantities in make orders where the exchange does not take custody of the tokens.
&nbsp;

`function updateAndGetQuantityHeldInExchange(address ofAsset) public returns (uint)`

This public function sums and returns all quantities of the provided asset token address in make orders across all registered exchanges, excluding, however, quantities in make orders where the exchange does not take custody of the tokens, but uses the ERC-20 "approve" functionality. The rationale is that token quantities in "approve" status are not actually held by the exchange. The function also maintains the `exchangesToOpenMakeOrders` and `isInOpenMakeOrder` mappings.
&nbsp;

`function returnAssetToVault(address _token) public`

This public function transfers all token quantities of the provided array of token asset addresses from the current current contract's custody back to the fund's vault.
&nbsp;

`function addZeroExOrderData(bytes32 orderId, LibOrder.Order zeroExOrderData) delegateInternal`

This function adds the data provided by the parameters to the orderIdToZeroExOrder mapping state variable.
&nbsp;

`function returnBatchToVault(address[] _tokens) public`

This public function returns all asset tokens represented by the `_tokens` address array parameter and returns the asset tokens to the Melon fund's vault.
&nbsp;

`function getExchangeInfo() public view returns (address[], address[], bool[])`

This public view function returns two address arrays and one boolean array with all corresponding registered exchange contract addresses, adapter contract addresses and the `takesCustoday` indicators.
&nbsp;

`function getOpenOrderInfo(address ofExchange, address ofAsset) public view returns (uint, uint, uint)`

This public view function takes the exchange contract address and an asset token contract address and returns three integers: the order identifier, the order expiration time and the order index.
&nbsp;

`function getOrderDetails(uint orderIndex) public view returns (address, address, uint, uint)`

This public view function takes the order index as a parameter and returns the maker asset token contract address, the taker asset token contract address, the maker asset token quantity and the taker asset token quantity.
&nbsp;

`function getZeroExOrderDetails(bytes32 orderId) public view returns (LibOrder.Order)`

This public view function takes the order identifier as a parameter and returns the corresponding populated `Order` struct.
&nbsp;

## Exchange Adapters

Exchange Adapters are smart contracts which communicate directly and on-chain with the intended DEX smart contract. They serve as a translation bridge between the Melon Fund and the DEX.

Currently, the Melon Protocol has adapters to integrate the following DEXs:

- Oasis DEX
- 0x (enabling interaction on the orderbooks of all 0x relayers)
- Kyber Network
- Ethfinex

Each exchange is tied to a specific adapter by the canonical registrar. A fund can be setup to use multiple exchanges, provided they are registered by the registrar.

### Adapter Function Details

The `Fund.sol` smart contract in the Melon Protocol (the blockchain fund instance) commonly uses the following functions in the Exchange Adapter to interact with the intended DEX for trading purposes:

- `makeOrder()` Creates a new order in the DEX's order book. The order may not be immediately executed. Note that this function will not be implemented for the 0x adapter until 0x Version 2 is released.

- `takeOrder()` Represents implicit agreement with a standing make order on the DEX's order book. The order will be immediately executed.

- `cancelOrder()` Retracts a standing make order from the DEX's order book. The cancelation will be immediately executed.

A single event is emitted by the Exchange Adapter:

- `OrderUpdated()` Event to inform other layers (e.g. web page) that the order has been updated in some way.

The following functions are public view functions:

- `getLastOrderId()` - Constant view function which returns the last order Id on a specific exchange.

- `getOrder()` - Constant view function which returns the order's sell asset address, buy asset address, sell asset quantity and buy asset quantity on a given exchange for a given order Id.

Note that fund is not limited to these functions and can call arbitrary functions on the exchange adapters using delegate calls, provided the function signature is whitelisted by the canonical registrar.
&nbsp;

---

## Fund Policy

### Policy Manager

The policyManager contract is the core of risk management and compliance policies. Policies are individual contracts that define and enforce specific business logic codified within. Policies are registered with the Melon fund's policyManager contract for specific function call pertaining to trading fund positions or investor subscriptions.

The policyManager is embedded into specific function calls in other Spokes of the Melon fund as required through the modifiers described below.
&nbsp;

## PolicyManager.sol

#### Description

In many of the functions below an array of address `addresses`, an array of uint `values` and an `identifier` are passed as parameters. For the arrays, the order of the address or value in the array is semantically significant. The individual array positions, [n], are defined within the policyManager contract as follows:

`address[5] addresses`:

[0] order maker - the address initiating or offering the trade
[1] order taker - the address filling or partially filling the offered trade
[2] maker asset - the token address of the asset token intended by the order maker to exchange for another token asset, i.e. the taker asset
[3] taker asset - the token address of the asset token to be received by the offer maker in exchange for the maker asset
[4] exchange address - address of the exchange contract on which the order is to be placed

`uint[3] values`:

[0] maker token quantity - the maximum quantity of the maker asset token to be given by the order maker in exchange for the specified taker asset token
[1] taker token quantity - the maximum quantity of the taker asset token to be received by the order maker in exchange for the specified maker asset token
[2] Fill amount - the quantity of the taker token exchanged in the transaction. Must be less than or equal to the taker token quantity [1]

Finally, `identifier` and `sig` parameters are described below:

`bytes32 identifier` - order id for exchanges utilizing a unique, exchange-specific order identifier

`bytes4 sig` - the keccak256 hash of the plain text function signature of the function which triggers the specific policy validation.
&nbsp;

#### Inherits from

Spoke (link)

&nbsp;

#### On Construction

The PolicyManager contract is passed the corresponding Hub address and sets this as the hub state variable inherited from Spoke.

The PolicyManager contract is created from the PolicyManagerFactory contract, which creates a new instance of `PolicyManager` given the `hub` address, registering the address of the newly created PolicyManager contract as a child of the PolicyManagerFactory.
&nbsp;

#### Structs

`Entry`

Member variables:

`Policy[] pre` - An array of Policy contract addresses which are registered to be validated as pre-conditions to defined function calls.

`Policy[] post` - An array of Policy contract addresses which are registered to be validated as post-conditions to defined function calls.
&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

`modifier isValidPolicyBySig(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier)`

This modifier ensures that `preValidate()` is called prior to applied function code and that `postValidate()` is called after the applied function code, sending the function signature hash `sig` as provided.
&nbsp;

`modifier isValidPolicy(address[5] addresses, uint[3] values, bytes32 identifier)`

This modifier ensures that `preValidate()` is called prior to applied function code and that `postValidate()` is called after the applied function code, sending the calling function signature hash `msg.sig`.
&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`mapping(bytes4 => Entry) policies`

A mapping of bytes4 to an `Entry` struct.
&nbsp;

#### Public Functions

`function registerBatch(bytes4[] sig, address[] _policies) public auth`

This function requires that the caller is the `owner` or the current contract. This public function requires equal length of both array parameters. The function then iterates over the `sig` array calling the register() function, providing each function signature hash and the corresponding Policy contract address.  
&nbsp;

`function register(bytes4 sig, address _policy) public auth`

This function requires that the caller is the `owner` or the current contract. This public function first ascertains whether the Policy being registered with the PolicyManager is to be executed as a pre- or post condition and then pushes the Policy with the corresponding signature hash on to the respective pre or post Policy array within the policies mapping. Once a Policy is registered, the condition defined within the Policy will be the standard against which the policy-registered function's consequential state changes will be tested. If the state changes pass the policy test, the function will continue execution unhindered and the state changes, e.g. a trade and the respective changes to token allocation) will become final as part of a transaction in a mined block. If the state changes do not pass the policy test, the transaction will revert and no state change will be affected.

&nbsp;

`function getPoliciesBySig(bytes4 sig) public view returns (address[], address[])`

This view function returns two address arrays (pre and post, respectively) containing all Policy contract addresses registered for the provided function signature hash. This gives the ability to query registered policies for a specific function call.
&nbsp;

`function preValidate(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) view public`

This view function calls the `validate()` function, explicitly passing the `pre` array from the `policies` mapping state variable filtered for Policies registered for the provided function signature hash `sig`.
&nbsp;

`function postValidate(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) view public`

This view function calls the `validate()` function, explicitly passing the `post` array from the `policies` mapping state variable filtered for Policies registered for the provided function signature hash `sig`.
&nbsp;

`function validate(Policy[] storage aux, bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) view internal`

This internal view function receives a function-specific filtered array of Policy contract addresses and iterates over the array, calling each Policy's implemented `rule()` function. If the call to a Policy's `rule()` evaluates to `true`, execution and any state transition proceeds. If the call to a Policy's `rule()` evaluates to `false`, all execution and preliminary state transitions are reverted.
&nbsp;

## Policies

Polices are individual smart contracts which define rule or set of rules to be compared to the state of the Melon fund. Policies simply assess the current state of the Melon fund and resolve to a boolean decision, whether the action may be executed or not, returning `true` for allowed actions and `false` for disallowed actions. The Melon fund-specific Policies are deployed with parameterized values which the defined Policy logic uses to assess the permissibility of the action.

### Pre- and Post-Conditionality

Policies are intended to be rules; they are intended to permit or prevent specific behavior or action depending on the _state_ as compared to specified criteria.

Some Policies, due to the nature of the data required, can be immediately resolved based on the _current_ state. The resolution of the Policy result is trivial because all data needed is at hand and must nod be derived or calculated. Such Policies can be defined as "pre-condition" Policies.

Other Policies may need to assess the consequence of the behavior or action before the logic can assess its permissibility relative to the defined rule. To do this, the result of the action must be derived or calculated, essentially asking, "What _will_ the state be if this action is executed?" Such Policies can be defined as "post-condition" Policies.

On the blockchain and in smart contracts, we can use a fortunate side-effect of the process of mining and block finalization to help determine the validity of post-condition Policies. With post-condition Policies, the action or behavior is executed with the smart contract logic and the changed (but not yet finalized or mined) state is assessed against the logic and defined parameters of the post-condition Policy. In the case where this new state complies with the logic and criteria of the Policy, the action is allowed, meaning the smart contract execution is allowed to run to completion, the block is eventually mined and this new compliant state is finalized in that mined block. In the case where the new state does not comply with the logic and criteria of the Policy, the action is disallowed and the revert() function is called, stopping execution and discarding (or rolling back) all state changes. In calling the revert() function, gas is consumed to arrive at the reference state, but any unused gas is returned to the caller as the reference state is discarded.
&nbsp;

## Policy.sol

#### Description

The Policy contract is inherited by implemented Policies. This contract will be changed to an interface when upgraded to Solidity 0.5, as enums and structs are allowed to be defined in interfaces in that version.
&nbsp;

#### Inherits from

None.

&nbsp;

#### On Construction

None.

&nbsp;

#### Structs

None.

&nbsp;

#### Enums

`Applied` - An enum which characterizes the conditionality type of the Policy.

Member Types

`pre` - Indicates that the Policy will be evaluated prior to the corresponding function's execution.

`post` - Indicates that the Policy will be evaluated after the corresponding function's execution.
&nbsp;

#### Modifiers

None.

&nbsp;

##### Events

None.

&nbsp;

##### Public State Variables

None.

&nbsp;

##### Public Functions

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function is called by the PolicyManager to ensure that specific registered function calls are validated by their respective registered Policy validation logic. The function returns `true` when conditions defined in the Policy are met and execution continues to successful completion. If conditions defined in the Policy are no met, the function returns `false`, all execution up to that point is reverted and execution cannot complete, returning all remaining gas.

The `rule()` function takes a function signature hash `sig`, an array of address `addresses`, an array of uint `values` and an `identifier` are passed as parameters. For the arrays, the order of the address or value in the array is semantically significant. The individual array positions, [n], are defined as follows:

`address[5] addresses`:

[0] order maker - the address initiating or offering the trade
[1] order taker - the address filling or partially filling the offered trade
[2] maker asset - the token address of the asset token intended by the order maker to exchange for another token asset, i.e. the taker asset
[3] taker asset - the token address of the asset token to be received by the offer maker in exchange for the maker asset
[4] exchange address - address of the exchange contract on which the order is to be placed

`uint[3] values`:

[0] maker token quantity - the maximum quantity of the maker asset token to be given by the order maker in exchange for the specified taker asset token
[1] taker token quantity - the maximum quantity of the taker asset token to be received by the order maker in exchange for the specified maker asset token
[2] Fill amount - the quantity of the taker token exchanged in the transaction. Must be less than or equal to the taker token quantity [1]

Finally, `identifier` and `sig` parameters are described below:

`bytes32 identifier` - order id for exchanges utilizing a unique, exchange-specific order identifier

`bytes4 sig` - the keccak256 hash of the plain text function signature of the function which triggers the specific policy validation.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `Applied` which is defined for each specific Policy contract. The `Applied` enum indicates whether the Policy logic is applied prior to, or after the corresponding function's execution. This function is called when the Policy contract is registered with the PolicyManager contract.
&nbsp;

---

## Compliance

## General

The contracts below define the business logic for the screening of investor addresses allowed to- or prevented from subscribing to a Melon fund.

In the Melon Protocol, the term "Compliance" revolves around the specifics of investing as an Investor _into_ Melon Fund. This is often referred to as a "Subscription". Details around which addresses, amounts and when Investors may subscribe to the Melon Fund are configured in-, and managed by the Compliance module. To clarify, in the traditional investment management industry, "compliance" is also used to refer to trading of underlying fund assets, as in a whether a certain asset or trade is compliant with portfolio guidelines, laws and regulations. This type of intra-portfolio asset level compliance is handled by the Risk Engineering module in the Melon Protocol.

The main use case for the Melon Fund Compliance module is the ability to create and maintain a whitelist for specific addresses. That is, the Investment Manager can explicitly define specific addresses which will then have the ability to subscribe to the Melon Fund.

Note that any listing of an address on the Compliance module whitelist only impacts the Investor's _ability_ to subscribe to the Melon Fund. An existing investment from a specific address will remain invested irrespective of any change in that address's whitelist status. The current implementation does not affect an address status after the fact. Whitelist removal does not affect an invested address's current invested status or ability to redeem, but will prevent future subscriptions. The whitelist can be seen as a positive filter, creating a known universe of allowed investor addresses. Investor addresses can be added to- or removed from the whitelist, individually or in batch by the fund manager (`owner`).

### Future Directions

The Melon Fund Compliance module potentially has great significance for regulators and regulatory frameworks. Possible use cases are: Regulators or KYC/AML providers may create and maintain their own whitelist of Investor addresses in a Melon Compliance module that individual Melon Funds could query; the ability for the Investment Manager to "Hard Close" or "Soft Close" subscriptions; the ability to cap subscription amounts.

Hard Close - A fund refuses all new investment subscriptions, usually due to capacity limitations for a specific strategy.

Soft Close - A fund refuses investment subscriptions new Investors, but accepts investment top-ups from existing Investors (addresses).

## UserWhitelist.sol

#### Description

This contract defines a positive filter list, against which subscribing addresses are verified for membership. Member addresses are permitted to subscribe to the fund.

#### Inherits from

Policy, DSAuth (links)

&nbsp;

#### On Construction

The UserWhitelist contract requires an array of addresses (`_preApproved`) which are added to the `whitelisted` mapping.

&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

`event ListAddition(address indexed who)`

This event is triggered when an address is added to the `whitelisted` mapping. The event logs the newly added address.
&nbsp;

`event ListRemoval(address indexed who)`

This event is triggered when an address is removed from the `whitelisted` mapping. The event logs the removed address.
&nbsp;

#### Public State Variables

`mapping (address => bool) whitelisted`

Mapping which designates an investor address as being eligible to subscribe to the fund.
&nbsp;

#### Public Functions

`function addToWhitelist(address _who) public auth`

This function requires that the caller is the `owner` or the current contract. This function sets the `whitelisted` mapping for the provided address to `true`, then emits the `ListAddition()` event logging the newly added address.
&nbsp;

`function removeFromWhitelist(address _who) public auth`

This function requires that the caller is the `owner` or the current contract. This function sets the `whitelisted` mapping for the provided address to `false`. Addresses which had previously subscribed and are invested can not subsequently subscribe further amounts. Finally, the function emits the `ListRemoval()` event logging the removed address.
&nbsp;

`function batchAddToWhitelist(address[] _members) public auth`

This function requires that the caller is the `owner` or the current contract. The function, with one transaction, enables multiple addresses in the `whitelisted` mapping to be set to `true`.
&nbsp;

`function batchRemoveFromWhitelist(address[] _members) public auth`

This function requires that the caller is the `owner` or the current contract. The function, with one transaction, enables multiple addresses in the `whitelisted` mapping to be set to `false`. Addresses which had previously subscribed and are invested can not subsequently subscribe further amounts.
&nbsp;

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This function is called by the Policy Manager when functions registered for this specific policy are called. See further documentation under Policy Manager (link).
&nbsp;

`function position() external view returns (Applied)`

This function is called by the Policy Manager to determine whether the policy should be executed and evaluated as a pre-condition (`Applied.pre`) or as a post-condition (`Applied.post`). Compliance contracts are called as a pre-condition to subscription and therefore return the enum `Applied.pre` during the whitelist compliance check. See further documentation under Policy Manager (link).
&nbsp;

---

## Risk Engineering Policies

## General

### Risk Management vs Risk Engineering

_Risk_ is the effect of uncertainty on objectives; the risk is the possibility that an event will occur and adversely affect the achievement of an objective.

Risk is uncertainty with implicit consequences. In general, the investor cares about two types of risk: 1) those for which she receives compensation in return for bearing, and 2) those for which no compensation is received. It is obvious that the latter must be eliminated to the extent possible.

For example, the following shows risk dissected into systematic risk and unsystematic risk:

![Related image](https://www.tytoncapital.com/wp-content/uploads/2016/05/types-of-risk.gif)

We can observe that unsystematic risk can be _completely_ eliminated from the portfolio by sufficiently diversifying or simply having a sufficient number of assets in the portfolio. Of course, these assets must be reasonably uncorrelated with each other.

Critically, an investor is NOT compensated for bearing unsystematic risk, and this would be detrimental to the portfolio's return (and the investor's wealth) over time.

### Ex Post -> "Management"

Risks which have materialized should be managed to the extent possible. Examples:

- A position in the portfolio experiences positive returns to the point that the allocation breaches a concentration guideline. Managing this concentration risk means liquidating a portion of the position to return the position to a tolerable percentage of the portfolio. The proceeds are then allocated to other existing or new positions at the discretion of the investment manager.
- A position's price volatility increases to the point that it becomes too "risky" for the portfolio. This could be regarding the allocation or in general. In the former case, the position would be reduced so that the asset-weighted volatility of the portfolio returns to tolerable levels

### Ex Ante -> "Engineering"

There are potential risks that we can explicitly prohibit with blockchain tooling. This is where we want to shine.

Portfolio guidelines are rules which are laid out in a legal document (Offering Memorandum or Prospectus) prior to fund inception. Here, the parties agree on the types of instruments to be invested, strategy, rules about concentration and other rule-based metrics.

All institutional fund managers use software to help them manage portfolios. They will say, "Our software will not allow a trade which conflicts with the guidelines", or they will have a process in place which has a name like "pre-trade clearance" or "hypothetical trading". These methods have served the industry to a satisfactory level, but non-compliant trades do find their way into a portfolio.

Melon funds approach risk differently. Melon fund currently implement _risk engineering_ i.e. anticipating a specific risk and handling it in the code of the smart contract. Essentially, this is a proactive posture, as opposed to a reactive posture.

_Risk engineering_ is the anticipation and identification of different risks, and the action of preventing their occurrence through code. Rules coded into smart contracts that are meant to ensure that something **cannot not** happen in the structure of a Melon fund.

### What is risk management ?

- the **forecasting and evaluation of financial risks** together with the **identification of procedures to avoid or minimize their impact**
- **identification, evaluation and prioritization of risks** followed by coordinated and economical application of resources to minimize, monitor and control the probability or impact of unfortunate events.
- Risk management's objective is to **assure uncertainty does not deflect the endeavor from the business goals**.
- Strategies to manage threats, typically includes: **avoiding the threat, reducing the negative effect or probability of the threat, transferring all or part of the threat to another party** and even retaining some or all of the potential or actual consequences of a particular threat.
- Risk with greatest loss or impact and the greatest probability of occurring are handled first, and risks with lower probability of occurrence and lower loss are handled in descending order.

In Melon, it's different. We're not doing risk management, rather are we doing _risk engineering_ ie. anticipating the risk ahead of time, and handling it in the code of the smart contract. Being proactive vs reactive.

_risk engineering_ = the anticipation and identification of different risks ahead of time, and the action of preventing their occurrence through code. Rules coded into smart contracts that are meant to make sure that something **will not** happen in the context of a Melon fund.

**What risks can we identify within the context of a Melon fund?**

- Malevolent behavior of fund manager (trying to embezzle funds through orders on exchanges)
- Bad trading decisions from fund manager
- Deviation from initial strategy as laid out in the prospectus from fund manager
- Fund manager doesn't adhere to the level of risk that was communicated to the investors (volatility risk, sharpe ratio)
- Redemption risk
- Liquidity risk

**What could on-chain risk engineering entail?**

- Prevent investors from fund embezzlement through exchanges.
- Ensure that the fund operates accordingly to approved fund characteristics and other risk guidelines.
- Trading limits: Approve or reject trades based on: - is asset authorized? - is the asset liquid enough? - is the post trade holdings allocation acceptable? -Nominal (gross/net positions, concentration) - is the price _reasonable_ (ie. best execution)? - is the volatility implications of this trade acceptable according to portfolio guidelines?
- Price feed providing additional data on assets, useful to risk management such as historical volatility, average annual rates of return and average standard deviation. Is that conceivable?

**Which kind of off-chain risk engineering tooling can we provide?**

- Risk engineering should also encompass monitoring of portfolio's risk in light of market risk. Simple monitoring can be done off-chain at the discretion of the fund manager. Monitoring of indicators such as downside capture [1], drawdown [2], leverage etc.
- Possible to link up your fund with risk engineering trading bot such as rebalancing bots (eg. rebalancing based on allocation or rebalancing based on risk parity). Could be simple trading bots, not necessarily enforced on-chain.
- Tools to monitor VaR: measures the dollar-loss expectation that can occur with a 5% probability.
- Tools to monitor redemption risk (based on redemption requests, redemption history and in light of current portfolio allocation)

[1] ([**Downside Capture**](https://www.investopedia.com/university/hedge-fund/risks.asp))
In relation to hedge funds, and in particular those that claim absolute return objectives, the measure of [downside capture](https://www.investopedia.com/terms/d/down-market-capture-ratio.asp) can indicate how correlated a fund is to a market when the market declines. The lower the downside capture, the better the fund preserves wealth during market downturns. This metric is figured by calculating the cumulative return of the fund for each month that the market/benchmark was down, and dividing it by the cumulative return of the market/benchmark in the same time frame. Perfect correlation with the market will equate to a 100% downside capture and typically is only possible when comparing the benchmark to itself.

[2](<[**Drawdown**](https://www.investopedia.com/university/hedge-fund/risks.asp)>)
Another measure of a fund's risk is maximum [drawdown](https://www.investopedia.com/terms/d/drawdown.asp). Maximum drawdown measures the percentage drop in cumulative return from a previously reached high. This metric is good for identifying funds that preserve wealth by minimizing drawdowns throughout up/down cycles, and gives an analyst a good indication of the possible losses that this fund can experience at any given point in time. Months to recover, on the other hand, gives a good indication of how quickly a fund can recuperate losses. Take the case where a hedge fund has a maximum drawdown of 4%, for example. If it took three months to reach that maximum drawdown, as investors, we would want to know if the returns could be recovered in three months or less. In some cases where the drawdown was sharp, it should take longer to recover. The key is to understand the speed and depth of a drawdown with the time it takes to recover these losses. Do they make sense given the strategy?

### Rule set identification and specification

This section contains all the possible on-chain rules we have identified. Only a few of them will be implemented in v1 (see implemented policies).

#### Rule 1: Maximum number of positions

**Definition**: A fund may not exceed X positions.
**Rationale**: (i) Avoid over-diversification. (ii) Avoid excessive rebalancing transactions in the event of a new subscription (or redemption).
**Implementation**: Before a trade is authorized, the system must get the number of different assets held by the fund, and make sure that adding 1 asset does not breach the X maximum number of positions.
**Parameters**: X as the maximum number of underlying positions.

#### Rule 2: Whitelisted assets

**Definition**: The fund can only invest in explicitly permitted assets (included in the whitelisted asset universe defined at fund setup).
**Rationale**: (i) Investors have a clear understanding of which assets can ever be invested in by the fund. (ii) Restrict the investment manager. Tighten scope of the investment universe. (iii) Fund can only invest in a niche classification (eg. ESG, utility tokens, virtual currencies, commodities only etc.)
**Implementation**: Before a trade is authorized, the system must check that the asset being bought is included in the whitelisted asset universe of the fund. The asset registrar may contain an additional field _categories_ that informs about the nature/type of the asset. This _category_ field can then be queried by the contract before authorizing trading of the asset.
Other relevant categories: type (utility, security etc.), sector, country (of jurisdiction) etc.
**Parameters**: Whitelisted assets list or list of category (with specification of _AND_ or _OR_ relationship)
**Side note**: There could be a method to remove an asset from the whitelist. Fund manager should not be able to add an asset to the whitelist.

#### Rule 3: Blacklisted assets

**Definition**: The fund is explicitly prohibited from investing in some assets.
**Rationale**: (i) Investors have a guarantee that the fund will never invest in a type of asset. (ii) Restricts Investment manager (iii) Regulatory compliance (iv) Investor preferences.
**Implementation**: Before a trade is authorized, the system must check that the asset being bought is not included in the blacklisted assets list. The asset registrar may contain an additional field _categories_ that informs about the nature/type of the asset. This _category_ field can then be queried by the contract before authorizing trading of the asset.
**Parameters**: Blacklisted assets list
**Side note**: There could be a method to add an asset from the backlist. Fund manager should not be able to remove an asset from the blacklist.

#### Rule 4: Max concentration (%)

**Definition**: A position (or positions in a single category) can not actively exceed X% of the NAV.
**Rationale**: (i) Avoid cluster risk. (ii) Portfolio diversification (iii) Reduce unsystematic risk.
**Implementation**: Before a trade is authorized, the system must check the current proportion of NAV that this asset (or category) represents and computes the post trade proportion of NAV. If the post trade proportion of NAV exceeds X%, the trade will not be authorized.
**Parameters**: X as the maximum % a position's value can represent of the NAV.

#### Rule 5: Early redemption penalty (this may be contained in the compliance contract)

**Definition**: An investor can be incurred a penalty if he redeems his shares before the end of the specified period.
**Rationale**: VC funds, private equity funds. Longer time horizon funds.
**Implementation**: This is not part of the risk management contract per se, rather should be implemented as a compliance/participation module.
**Parameters**: period (probably expressed in # of blocks before redemption is possible) and penalty.

#### Rule 6: Best price execution

**Definition**: A fund manager is not allowed to deviate more than X% away from the reference price provided by the price feed.
**Rationale**: (i) Prohibit predatory investment manager behavior. (ii) Wash trading mitigation
**Implementation**: Before a trade is authorized, the system must check the price of the order against the reference price and ensure the former does not deviate away more than X% from the reference price.
**Parameters**: x as % of allowed deviation.

#### Rule 7: Turnover (expressed in # of trades)

**Definition**: Restrict the number of trades. The fund manager is restricted from exceeding the maximum number of trades allowed.
**Rationale**: (i) Prevent predatory behavior from the investment manager. (ii) Wash trading prevention. (iii) Keep transactions fees to the minimum.
**Implementation**: Before a trade is authorized, the system must check if this additional trade does not exceed the maximum # of trades allowed on the defined time period.
**Parameters**: max number of trades, time period.

Consider volume turnover in terms of portfolio base currency.

Note, top-ups (reductions) due to subscriptions (redemptions) should not count as turnover. Only discretionary investment decisions

#### Rule 8: Turnover (expressed in volume traded)

**Definition**: Restrict the number of trades. The fund manager is restricted from exceeding the maximum volume in trades allowed in a given timeframe.
**Rationale**: (i) Prevent predatory behavior from the investment manager. (ii) Wash trading prevention. (iii) Keep transactions fees to the minimum.
**Implementation**: Before a trade is authorized, the system must check if this additional trade does not exceed the maximum volume of trades allowed on the defined time period.
**Parameters**: max number volume of trades (expressed in the currency of denomination of the fund), time period.

Consider volume turnover in terms of portfolio base currency.

Note, top-ups (reductions) due to subscriptions (redemptions) should not count as turnover. Only discretionary investment decisions

#### Rule 9: Market cap range

**Definition**: A fund can only invest in assets whose market cap is contained in a certain range [x;y] (ie x < market cap < y).
**Rationale**: Strategy focus enforce
**Implementation**: Before trade is authorized, the system must check the market cap of the assets being traded and verify that the market cap is compliant with the market cap range defined at fund setup. The asset registrar may contain an additional field _market cap_ for each asset. Open question: how to measure Market Cap. Only influences the trade permission at time of trade. That is, there is no consequence when an owned asset's market cap breaches the upper or lower specified bounds.
**Parameters**: x as low boundary of the range and y as high boundary of the range.

#### Rule 10: Volatility threshold for a specific asset

**Definition**: A fund can only invest (purchase) in an asset whose volatility reference indicator is contained in a certain range [x;y] (ie x < volatility < y).
**Rationale**: Investors have an understanding of the volatility level of individual assets that are allowed in the portfolio at the time of inclusion.
**Implementation**: Before a trade is authorized, the system must check the volatility reference indicator of the said asset and make sure it is included in the specified [x;y] range.
The volatility reference indicator could be standard deviation but also downside deviation. Possible relevant indicators: standard deviation, Sharpe ratio, Sortino indicator.
**Parameters**: x as low boundary of the acceptable volatility range and y as high boundary of the acceptable volatility range.

Portfolio base currency cannot be considered for this rule.

#### Rule 11: Portfolio volatility threshold

**Definition**: A fund is prevented from performing a trade if the effect on the portfolio volatility is undesirable (ie if volatility post trade exceeds X% level of volatility)
**Rationale**: Investors have an understanding of the volatility level of the portfolio maintained at the time of trading.
**Implementation**: Before a trade is authorized, the system must check the post volatility of the total portfolio and make sure it is included in the specified [x;y] range.
**Parameters**: x as low boundary of the acceptable portfolio volatility range and y as high boundary of the acceptable portfolio volatility range.
**Side note**: This computation may be too expensive on Ethereum smart contract. We might be able to use a simplified computation. But this rule is to be envisioned in the Melon chain context.

Edge case: Could also allow trades which incrementally reduce (increase) portfolio volatility. Basically, is the trade beneficial to the investor and in the spirit of the guidelines.

#### Rule 12: Liquidity

**Definition**: A fund can only invest in assets whose liquidity is contained in a certain range [x;y] (ie x < liquidity < y).  
**Rationale**: (i) enforce strategy focus
**Implementation**: Before a trade is authorized, the system must check the liquidity available for that asset and make sure it is included in the specified [x;y] range. The liquidity data is retrieved from the exchanges the fund is allowed to trade on. Liquidity can be expressed as the % of your trade volume over the 30-days ADV (average daily volume traded).
**Parameters**: x as low boundary and y as high boundary of range.

#### Rule 13: Correlation of adding an asset to portfolio

**Definition**: A fund can invest in an asset if its correlation with the current portfolio is contained between a range [x;y]
**Rationale**: (i) making sure that adding an asset helps optimizing the portfolio or at least move it in the right direction.
**Implementation**: Before a trade is authorized, the system must compute the correlation of that asset to the portfolio and make sure it is included in the specified [x;y] range.
**Parameters**: x as low boundary and y as high boundary of the correlation range.

#### Rule 14: Time horizon

**Definition**: A fund manager needs to hold an asset for specific average period of time.
**Implementation**: We voluntarily decide not to encode that rule in a smart contract as it can lead to dangerous situations for the fund. Could be as a guideline for informational purposes but won't be enforced.

(not seen in practice, could be gamed by investor redemption and subsequent re-investment; must be thoughtfully aligned with lockup.)

#### Rule 15: Full investment

**Definition**: A fund manager can defined the full investment threshold ie. the proportion of assets invested vs the proportion of cash held.
**Rationale**: The fund manager should not stay passive collecting management fees just by holding too large proportion of cash (taking no risk) risk.
**Implementation**: We do not want to enforce that rule as the fund manager should have the flexibility to put everything in safety in case of imminent market crash or similar event. It could simply be an informational rule that can be used in monitoring/reporting tool to compare with actual investment levels of the fund.
**Parameters**: X as the percentage of NAV invested. Remove, unimportant; could cause sub-optimal behavior - keep as indicator for reporting.

#### Rule 16: Generalizable 5/10/40 rule.

This is a current UCITS rule. Positions may not exceed 10% of NAV, and that assets which exceed 5% of NAV summed together may not exceed 40% as a group.

#### Rule 17: Specific Asset/Category Max (%)

This rule assigns a maximum allocation as a percentage of NAV to a specific token or category. This is slightly different from Rule 4 which is an unspecified, blanket maximum allocation for any specific asset token or category.

#### Rule 18: Fund Cap

This rule could be implemented by an Investment Manager to cap the overall AuM of a Melon Fund for various reasons, in particular when a specific strategy may have no further market capacity within its mandate.

#### Rule 19: Fund Audit

This rule could be implemented by an Investment Manager to enforce fund audits by a pre-determined auditor at defined time intervals. Funds implementing this rule would be restricted from trading (except into the denomination asset) when an audit has not been performed within the defined time interval.

## Implemented Risk Policies

## AssetBlacklist.sol

#### Description

The Policy explicitly prohibits the Melon fund from investing in or hold any asset token which is part of the contract's blacklist. The rationale for this Policy my be: (i) investors have a guarantee that the fund will never invest in an asset. (ii) explicit restrictions on the manager (iii) regulatory compliance (iv) investor preferences. Assets token addresses can be added, but not removed from the contract's blacklist.
&nbsp;

#### Inherits from

TradingSignatures, Policy, AddressList (Link)

&nbsp;

#### On Construction

The contract requires an array of addresses, where each address is added to the contract's internal list.
&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

None.

&nbsp;

#### Public Functions

`function addToBlacklist(address _asset) external auth`

This function requires that the caller is the `owner` or the current contract. The function then requires that the token asset address is not a member of the list. If it is not a member of the list, the address provided is then added to the contract's internal list. Once a token asset address is blacklisted, it cannot be removed.
&nbsp;

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function returns `true` if the taker asset address (position [3] in the `addresses` array parameter) is not a member of the blacklist. The function returns `false` if the taker asset is a member of the blacklist. For a full description of the parameters, please refer to `rule()` in the general Policy contract above.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `pre`, indicating the the Policy logic be evaluated prior to execution of the Policy's corresponding registered function call.
&nbsp;

## AssetWhitelist.sol

#### Description

The Policy explicitly permits the Melon fund to invest in and hold any asset token which is part of the contract's whitelist as defined at fund setup. The rationale for this Policy my be: (i) Investors have a clear understanding of which assets can ever be invested in by the fund. (ii) restricting the investment manager - tightening the scope of the investment universe. (iii) the fund can only invest in a niche classification (eg. ESG, utility tokens, virtual currencies, commodities only etc.). Assets token addresses can be removed from, but not added to the contract's whitelist.
&nbsp;

#### Inherits from

TradingSignatures, Policy, AddressList (Link)

&nbsp;

#### On Construction

The contract requires an array of addresses, where each address is added to the contract's internal list.

&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

None.

&nbsp;

#### Public Functions

`function removeFromWhitelist(address _asset) external auth`

This function requires that the caller is the `owner` or the current contract. The function then requires that the token asset address is a member of the contract's list. If it is a member of the list, the address provided is then removed from the contract's internal list. Whitelisted token asset addresses can be removed from, but not added to, the contract's list.
&nbsp;

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function returns `true` if the taker asset address (position [3] in the `addresses` array parameter) is a member of the whitelist. The function returns `false` if the taker asset is not a member of the whitelist. For a full description of the parameters, please refer to `rule()` in the general Policy contract above.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `pre`, indicating the the Policy logic be evaluated prior to execution of the Policy's corresponding registered function call.
&nbsp;

## MaxConcentration.sol

#### Description

The Policy explicitly prevents a position from actively exceeding the specified percentage of fund value. The rationale for this Policy is to: (i) avoid cluster risk, (ii) aid portfolio diversification and (iii) reduce unsystematic risk. This Policy is evaluated after execution of the registered corresponding function.
&nbsp;

#### Inherits from

TradingSignatures, DSMath, Policy (Link)

&nbsp;

#### On Construction

The contract requires the parameter representing the concentration upper limit as a percentage be less than 100%. The public state variable `maxConcentration` is then set to the value provided by this parameter.
&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`uint internal constant ONE_HUNDRED_PERCENT = 10 ** 18`

This constant is used to correctly represent 100%. It is set to 10^18.
&nbsp;

`uint public maxConcentration`

This public variable stores the upper limit, in percentage of fund value, which an individual asset token position can have. The value is stored using 18 decimal precision, i.e. 100000000000000000 would represent a maximum concentration for any individual token asset position of 10% of fund value.
&nbsp;

#### Public Functions

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function returns `true` if Melon fund's denomination asset is the taker asset. The rationale for this is that denomination asset should not be subject to the maximum concentration rule as the denomination asset represents non-exposure to the wider market. In cases where the taker asset is not the denomination asset, the function evaluates the post-trade allocation percentage of the position to fund value, returning `true` if the entire fund post-trade position in the taker asset does not exceed the maximum concentration percentage. If the fund's post-trade position in the taker asset does exceed the maximum concentration percentage, the function returns `false` and the transaction is reverted. For a full description of the parameters, please refer to `rule()` in the general Policy contract above.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `post`, indicating the the Policy logic be evaluated after execution of the Policy's corresponding registered function call.
&nbsp;

## MaxPositions.sol

#### Description

The Policy explicitly prevents a position from entering the fund if the quantity of non-denomination asset token positions exceed the specified `maxPositions` state variable. The rationale for this is to: (i) avoid over-diversification and (ii) avoid excessive rebalancing transactions in the event of a new subscription (or redemption). This Policy is evaluated after execution of the registered corresponding function.
&nbsp;

#### Inherits from

TradingSignatures, Policy (Link)

&nbsp;

#### On Construction

The contract requires a parameter representing the maximum number of non-denomination asset token positions permitted in the Melon fund. The public state variable `maxPositions` is then set to the value provided by this parameter. A value of 0 means that no non-denomination assets positions are permitted.
&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`uint public maxPositions`

This public variable stores the upper limit of the number of non-denomination asset positions permitted in the Melon fund. For example, `maxPositions` == 5 means that a maximum of five non-denomination asset token positions are permitted at any time.
&nbsp;

#### Public Functions

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function returns `true` if Melon fund's denomination asset is the taker asset. The rationale for this is that denomination asset should not be subject to the maximum positions rule as the denomination asset represents non-exposure to the wider market. In cases where the taker asset is not the denomination asset, the function evaluates the post-trade number of non-denomination asset positions in the fund, returning `true` if the post-trade number of non-denomination asset positions in the fund does not exceed the maximum positions configuration. If the fund's post-trade position in the taker asset causes the fund's non-denomination asset position quantity to exceed the maximum positions quantity, the function returns `false` and the transaction is reverted. For a full description of the parameters, please refer to `rule()` in the general Policy contract above.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `post`, indicating the the Policy logic be evaluated after execution of the Policy's corresponding registered function call.
&nbsp;

## PriceTolerance.sol

#### Description

The PriceTolerance Policy explicitly prevents a make order from being submitted to an exchange, or explicitly prevents a take order trade from executing depending on a comparison between the implicit order price, adjusted for the specified tolerance, and the current reference price as provided by the price feed. The rationale is that the Policy: (i) prevents predatory investment manager behavior and (ii) mitigates wash trading. This Policy is evaluated prior to execution of the registered corresponding functions.
&nbsp;

#### Inherits from

TradingSignatures, Policy, DSMath (Link)

&nbsp;

#### On Construction

The contract requires a parameter representing the maximum price feed deviation of an asset trade to the detriment of fund investors. The `tolerance` state variable is then set to this value. A parameter value of 10 would represent a maximum order price deviation to the current reference price of 10%. The contract constructor requires the `_tolerancePercent` parameter to range between "0" and "100".
&nbsp;

#### Structs

None.

&nbsp;

#### Enums

None.

&nbsp;

#### Modifiers

None.

&nbsp;

#### Events

None.

&nbsp;

#### Public State Variables

`uint tolerance`

This state variable stores the permitted deviation in percentage of the order price to the current reference price. A value of 10 would represent a maximum order price deviation to the current reference price of 10%.
&nbsp;

`uint constant MULTIPLIER = 10 ** 16`

This constant is used to scale the `_tolerancePercent` parameter to correctly represent a percentage. It is set to 10^16.
&nbsp;

`uint constant DIVISOR = 10 ** 18`

This constant is used to correctly represent token quantities/prices. It is set to 10^18.
&nbsp;

#### Public Functions

`function takeOasisDex(address ofExchange, bytes32 identifier, uint fillTakerQuantity) view returns (bool)`

This view function gathers data from the existing Oasis Dex order and respective amounts filled, then compares the resulting price with the tolerance percentage-adjusted reference price from the price feed. If the order price does not exceed the tolerance percentage-adjusted reference price, the function returns `true` and the fund is permitted to take the order (partial- or complete fill); otherwise the function returns `false`, the trade is not permitted and the transaction is reverted.
&nbsp;

`function takeGenericOrder(address makerAsset, address takerAsset, uint[3] values) view returns (bool)`

This view function accommodates generic exchange take order functionality, calculating the maker fill quantity given the taker quantity, maker quantity and the taker fill quantity, then compares the resulting price (fill quantity ratio) with the tolerance percentage-adjusted reference price from the price feed. If the take order price does not exceed the tolerance percentage-adjusted reference price, the function returns `true` and the fund is permitted to take the order (partial- or complete fill); otherwise the function returns `false`, the trade is not permitted and the transaction is reverted.
&nbsp;

`function takeOrder(address[5] addresses, uint[3] values, bytes32 identifier) public view returns (bool)`

This view function is evaluated prior to executing take orders on exchanges and determines the execution routing for take orders depending on value of the `identifier` parameter. If the parameter `identifier == 0x0` (denoting a 0x-integrated exchange), the function calls `takeGenericOrder()` and `takeOasisDex()` otherwise, passing all input parameters on to the subsequent function call.
&nbsp;

`function makeOrder(address[5] addresses, uint[3] values, bytes32 identifier) public view returns (bool)`

This view function is evaluated prior to executing make orders on exchanges. The function determines the current reference price for the asset token pair, then determines the price as determined by the make offer quantities. If the make order price does not exceed the tolerance percentage-adjusted reference price, the function returns `true` and the fund is permitted to submit the make order to the exchange; otherwise the function returns `false`, the make order is not permitted and the transaction is reverted.
&nbsp;

`function rule(bytes4 sig, address[5] addresses, uint[3] values, bytes32 identifier) external view returns (bool)`

This view function is evaluated prior to executing make orders or take orders on exchanges and determines the execution routing for these order types depending on value of the `sig` parameter, which is the keccak256 hash of the plain text function signature of `makeOrder()` or `takeOrder()`. The function returns `true` for permitted orders, i.e. make orders can be submitted to the exchange and take orders can be traded. The function returns `false` for prohibited orders which violate the price tolerance criteria, reverting the transaction. For a full description of the parameters, please refer to `rule()` in the general Policy contract above.
&nbsp;

`function position() external view returns (Applied)`

This view function returns the enum `pre`, indicating the the Policy logic be evaluated prior to execution of the Policy's corresponding registered function call.
&nbsp;
