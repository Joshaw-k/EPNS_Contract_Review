# EPNS(Ethereum Push Notification Service) Code Review

### Introduction

Ethereum Push Notification Service (EPNS), now known as the Push Protocol, is a communication protocol designed for the web3 ecosystem. It enables cross-chain notifications and messaging for decentralized applications (dApps), wallets, and services tied to wallet addresses in an open, gasless, and platform-agnostic fashion.

The protocol was created to address the lack of a notification mechanism in web3, which often results in users missing important updates, events, actions, and more. EPNS aims to solve this issue by providing a push notification service that can notify users or wallet addresses of important updates.

### Application Workflow

- **Integration:** To integrate EPNS into your application, you need to utilize the Push SDK. This SDK offers an abstraction layer for incorporating Push protocol features into both your frontend and backend systems

- **Subscription:** Users can subscribe to EPNS channels from a frontend application. This allows them to receive notifications from the channels they choose

- **Notification Creation:** Any dApp, service, or smart contract can send notifications to users (wallet addresses) in a platform-agnostic way. The notifications are triggered when a smart contract reaches certain conditions

- **Notification Delivery:** The notifications are delivered to the user's device via the platform they are using (mobile, tablet, web, etc.). The user has full control over what notifications they receive, allowing them to subscribe to or unsubscribe from the channels that they deem fit

- **User Interaction:** The user can interact with the notification, for example, by clicking on it to open the dApp, wallet, or smart contract that sent the notification

- **Governance:** The protocol governance is intended to incentivize continued adoption of the EPNS protocol, which is achieved by providing incentives for all users involved. These incentives are given in the form of PUSH tokens

### Code Walkthrough

The EPNS protocol consists of two distinct smart contracts, namely the "EPNS Core" and the "EPNS Communicator".

EPNSCore is the core protocol contract. It handles the creation of channels, channel state cycles, and channel verification features. Channels are created by dApps or smart contracts and require a minimum fee of 50 DAI. This fee forms a fund pool that generates constant interest (AAVE aDai). The generated interest from this goes to their subscribers as per their fair share.

EPNSCommunicator is the communicator protocol contract. It is responsible for sending notifications with EPNS CommV1. Any dApp, service, or smart contract can send notifications to users (wallet addresses) in a platform-agnostic way. The notifications are triggered when a smart contract reaches certain conditions

Here is the workflow for how these contracts work together:

- A dApp or smart contract creates a channel using the EPNSCore contract. This channel is tied to a specific wallet address and can send notifications to that address.

- The dApp or smart contract sends a notification using the EPNS Communicator contract. This notification is tied to the channel created.

- The notification is delivered to the user's device via the platform they are using (mobile, tablet, web, etc.). The user has full control over what notifications they receive, allowing them to subscribe to or unsubscribe from the channels that they deem fit.

#### EPNS_CORE_V1 Contract

This contract imports several interfaces to interact with AAVE Protocol and Uniswap and it also imports some of OpenZeppelin’s reusable libraries and contracts to build a secure smart contract.

##### A. IMPORTS

```
import "./interfaces/IPUSH.sol";
import "./interfaces/IADai.sol";
import "./interfaces/ILendingPool.sol";
import "./interfaces/IUniswapV2Router.sol";
import "./interfaces/IEPNSCommV1.sol";
import "./interfaces/ILendingPoolAddressesProvider.sol";

import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/proxy/Initializable.sol";
import "@openzeppelin/contracts/token/ERC20/SafeERC20.sol";
```

The interfaces imported are:

- **IPUSH:** This contract is used to interact with the PUSH token contract. The PUSH token is a utility token used by the Ethereum Push Notification Service (EPNS) to reward users for interacting with the platform.

- **IADai:** This contract interacts with Aave's lending pool. The redeem function is used to redeem or withdraw a certain amount of aDAI tokens from the Aave lending pool.

- **ILendingPool:** The epnscorev1 contract uses the ILendingPool interface to interact with the Aave lending protocol in order to manage the liquidity of PUSH tokens and generate revenue for the EPNS platform.
  IUniswapV2Router: The swapExactTokensForTokens function of the IUniswapV2Router interface is used in the epnscorev1 contract to swap PUSH tokens for other tokens.

- **IEPNSCommV1:** The EPNSCorev1 contract uses the subscribeViaCore function to subscribe users to channels on behalf of the EPNS platform. This allows the EPNS platform to send notifications to users about important events, such as new updates to the platform or new messages from channels that they are subscribed to.

- **ILendingPoolAddressesProvider:** The ILendingPoolAddressesProvider interface is used in the epnscorev1 contract to get the addresses of the Aave lending pool core and lending pool contracts. The epnscorev1 contract uses these addresses to interact with the Aave lending protocol.

The Openzeppelin imports are:

- **Strings.sol:** This library provides a number of functions for working with strings, such as converting strings to and from numbers, converting strings to uppercase and lowercase, and concatenating strings.

- **SafeMath.sol:** This library provides a number of functions for performing arithmetic operations safely, such as addition, subtraction, multiplication, and division.

- **IERC20.sol:** This interface defines the standard interface for an ERC20 token.

- **Initializable.sol:** This base contract is used to create upgradeable smart contracts.

- **SafeERC20.sol:** This library provides a number of wrapper functions for interacting with ERC20 tokens that check for success and revert on error. This can help to prevent unexpected behavior and security vulnerabilities in your Solidity contracts.

##### B. ENUMS

```
// For Message Type
enum ChannelType {
    ProtocolNonInterest,
    ProtocolPromotion,
    InterestBearingOpen,
    InterestBearingMutual
}
enum ChannelAction {
    ChannelRemoved,
    ChannelAdded,
    ChannelUpdated
}
```

**ChannelType:** This enum is used in this contract to determine the type of channel that is being created or interacted with. The type of channel determines a number of factors, such as the following: ProtocolNonInterest, ProtocolPromotion, InterestBearingOpen, InterestBearingMutual

**ChannelAction:** enum is used in the epnscorev1 contract to represent the different actions that can be performed on a channel. These actions are: ChannelRemoved, ChannelAdded, ChannelUpdated

##### C. STRUCTS

```
struct Channel {
    ChannelType channelType;

    uint8 channelState;

    address verifiedBy;

    uint256 poolContribution;

    uint256 channelHistoricalZ;

    uint256 channelFairShareCount;

    uint256 channelLastUpdate;

    uint256 channelStartBlock;

    uint256 channelUpdateBlock;

    uint256 channelWeight;
}
```

**channelType:** This is an enum that denotes the type of channel being created.

**channelState:** This is an unsigned integer that denotes the current state of a particular channel. The channel can have any of the following states: INACTIVE, ACTIVE, DEACTIVATED, BLOCKED.

**verifiedBy:** This is the address of the user who verified the channel. This is the user who is responsible for ensuring that the channel is legitimate and that the content that is sent through the channel is accurate and up-to-date.

**poolContribution:** This is the amount of PUSH tokens that the channel has contributed to the EPNS pool. This amount is used to calculate the channel's fair share of the rewards that are distributed from the pool.

**channelHistoricalZ:** The channel's historical Z-score. This is a measure of the channel's engagement and popularity.
channelFairShareCount: The number of fair shares that the channel has earned. This number is used to calculate the channel's share of the rewards that are distributed from the pool.

**channelLastUpdate:** The block number at which the channel was last updated.

**channelStartBlock:** The block number at which the channel was created.

**channelUpdateBlock:** The block number at which the channel was last updated.

**channelWeight:** The channel's weight. This is a measure of the channel's importance and influence.

##### D. MAPPING

```
mapping(address => Channel) public channels;
mapping(uint256 => address) public channelById;
mapping(address => string) public channelNotifSettings;
```

**channels:** This mapping is used to store all of the channels in the EPNS platform. The key of the mapping is the address of the channel, and the value of the mapping is the Channel struct that represents the channel.

**channelById:** This mapping is used to store the address of the channel for a given channel ID. The key of the mapping is the channel ID, and the value of the mapping is the address of the channel.

**channelNotifSettings:** This mapping is used to store the notification settings for a given channel. The key of the mapping is the address of the channel, and the value of the mapping is a string that represents the channel's notification settings.

```
string public constant name = "EPNS CORE V1";
bool oneTimeCheck;
bool public isMigrationComplete;

address public pushChannelAdmin;
address public governance;
address public daiAddress;
address public aDaiAddress;
address public WETH_ADDRESS;
address public epnsCommunicator;
address public UNISWAP_V2_ROUTER;
address public PUSH_TOKEN_ADDRESS;
address public lendingPoolProviderAddress;

uint256 public REFERRAL_CODE;
uint256 ADJUST_FOR_FLOAT;
uint256 public channelsCount;

//  @notice Helper Variables for FSRatio Calculation | GROUPS = CHANNELS
uint256 public groupNormalizedWeight;
uint256 public groupHistoricalZ;
uint256 public groupLastUpdate;
uint256 public groupFairShareCount;

// @notice Necessary variables for Keeping track of Funds and Fees
uint256 public POOL_FUNDS;
uint256 public PROTOCOL_POOL_FEES;
uint256 public ADD_CHANNEL_MIN_FEES;
uint256 public CHANNEL_DEACTIVATION_FEES;
uint256 public ADD_CHANNEL_MIN_POOL_CONTRIBUTION;
```

**name:** The name of the contract.

**oneTimeCheck:** A boolean variable that is used to ensure that certain functions are only called once.

**isMigrationComplete:** A boolean variable that indicates whether or not the migration process has been completed.

**pushChannelAdmin:** The address of the Push channel admin. This is the address of the account that is responsible for managing the Push channel.

**governance:** The address of the governance contract. This is the contract that is responsible for managing the EPNS platform.

**daiAddress:** The address of the DAI token contract.

**aDaiAddress:** The address of the aDAI token contract.

**WETH_ADDRESS:** The address of the WETH token contract.

**epnsCommunicator:** The address of the EPNS communicator contract. This is the contract that is responsible for sending notifications to users.

**UNISWAP_V2_ROUTER:** The address of the Uniswap V2 router contract. This is the contract that is used to swap tokens on the Uniswap V2 decentralized exchange.

**PUSH_TOKEN_ADDRESS:** The address of the PUSH token contract.

**lendingPoolProviderAddress:** The address of the Aave lending pool provider contract.

**REFERRAL_CODE:** The referral code for the EPNS platform.

**ADJUST_FOR_FLOAT:** A value that is used to adjust for floating-point errors.

**channelsCount:** The number of channels in the EPNS platform.

**groupNormalizedWeight:** The normalized weight of the group.

**groupHistoricalZ:** The historical Z-score of the group.

**groupLastUpdate:** The block number at which the group was last updated.

**groupFairShareCount:** The number of fair shares that the group has earned.

**POOL_FUNDS:** The amount of funds in the EPNS pool.

**PROTOCOL_POOL_FEES:** The amount of protocol fees that have been collected from the EPNS pool.

**ADD_CHANNEL_MIN_FEES:** The minimum amount of fees that must be paid to add a new channel to the EPNS platform.

**CHANNEL_DEACTIVATION_FEES:** The amount of fees that must be paid to deactivate a channel in the EPNS platform.

**ADD_CHANNEL_MIN_POOL_CONTRIBUTION:** The minimum amount of PUSH tokens that must be contributed to the EPNS pool to add a new channel.

##### E. EVENTS

```
event UpdateChannel(address indexed channel, bytes identity);
event ChannelVerified(address indexed channel, address indexed verifier);
event ChannelVerificationRevoked(address indexed channel, address indexed revoker);

event DeactivateChannel(
    address indexed channel,
    uint256 indexed amountRefunded
);
event ReactivateChannel(
    address indexed channel,
    uint256 indexed amountDeposited
);
event ChannelBlocked(
    address indexed channel
);
event AddChannel(
    address indexed channel,
    ChannelType indexed channelType,
    bytes identity
);
event ChannelNotifcationSettingsAdded(
    address _channel,
    uint256 totalNotifOptions,
    string _notifSettings,
    string _notifDescription
);
```

**UpdateChannel event:** This event is emitted when a channel's identity is updated. This event includes the address of the channel and the new identity of the channel.

**ChannelVerified event:** This event is emitted when a channel is verified by another user. This event includes the address of the channel and the address of the user who verified the channel.

**ChannelVerificationRevoked event:** This event is emitted when a channel's verification is revoked by another user. This event includes the address of the channel and the address of the user who revoked the channel's verification.

**DeactivateChannel event:** This event is emitted when a channel is deactivated. This event includes the address of the channel and the amount of money that was refunded to the channel owner when the channel was deactivated.

**ReactivateChannel event:** This event is emitted when a channel is reactivated. This event includes the address of the channel and the amount of money that was deposited into the channel when it was reactivated.

**ChannelBlocked event:** This event is emitted when a channel is blocked. This event includes the address of the channel.

**AddChannel event:** This event is emitted when a new channel is added. This event includes the address of the channel, the type of channel, and the identity of the channel.

**ChannelNotifcationSettingsAdded event:** This event is emitted when notification settings are added to a channel. This event includes the address of the channel, the total number of notification options, the notification settings, and the notification description.

These events can be used by users to track changes to channels and their settings. For example, a user can subscribe to the ChannelVerified event to be notified when a channel is verified. Or, a user can subscribe to the ChannelBlocked event to be notified when a channel is blocked.

##### F. MODIFIER

```
modifier onlyPushChannelAdmin() {
    require(msg.sender == pushChannelAdmin, "EPNSCoreV1::onlyPushChannelAdmin: Caller not pushChannelAdmin");
    _;
}

modifier onlyGovernance() {
    require(msg.sender == governance, "EPNSCoreV1::onlyGovernance: Caller not Governance");
    _;
}

modifier onlyInactiveChannels(address _channel) {
    require(
        channels[_channel].channelState == 0,
        "EPNSCoreV1::onlyInactiveChannels: Channel already Activated"
    );
    _;
}

modifier onlyActivatedChannels(address _channel) {
    require(
        channels[_channel].channelState == 1,
        "EPNSCoreV1::onlyActivatedChannels: Channel Deactivated, Blocked or Does Not Exist"
    );
    _;
}

modifier onlyDeactivatedChannels(address _channel) {
    require(
        channels[_channel].channelState == 2,
        "EPNSCoreV1::onlyDeactivatedChannels: Channel is not Deactivated Yet"
    );
    _;
}

modifier onlyUnblockedChannels(address _channel) {
    require(
        ((channels[_channel].channelState != 3) &&
        (channels[_channel].channelState != 0)),
        "EPNSCoreV1::onlyUnblockedChannels: Channel is BLOCKED Already or Not Activated Yet"
    );
    _;
}

modifier onlyChannelOwner(address _channel) {
    require(
    ((channels[_channel].channelState == 1 && msg.sender == _channel) ||
    (msg.sender == pushChannelAdmin &&
    _channel == address(0x0))),
    "EPNSCoreV1::onlyChannelOwner: Channel not Exists or Invalid Channel Owner"
    );
    _;
}

modifier onlyUserAllowedChannelType(ChannelType _channelType) {
    require(
        (_channelType == ChannelType.InterestBearingOpen ||
        _channelType == ChannelType.InterestBearingMutual),
        "EPNSCoreV1::onlyUserAllowedChannelType: Channel Type Invalid"
    );

    _;
}
```

**onlyPushChannelAdmin():** This modifier function restricts access to functions to the Push Channel Admin.

**onlyGovernance():** This modifier function restricts access to functions to the Governance contract.

**onlyInactiveChannels():** This modifier function restricts access to functions to channels that are inactive.

**onlyActivatedChannels():** This modifier function restricts access to functions to channels that are activated.

**onlyDeactivatedChannels():** This modifier function restricts access to functions to channels that are deactivated.

**onlyUnblockedChannels():** This modifier function restricts access to functions to channels that are not blocked.

**onlyChannelOwner():** This modifier function restricts access to functions to the owner of the channel.

**onlyUserAllowedChannelType():** This modifier function restricts access to functions to channels of a certain type, in this case, InterestBearingOpen or InterestBearingMutual.

These modifier functions are important for protecting the EPNS protocol and ensuring that only authorized users can perform certain actions.

##### G. FUNCTIONS

**Function `initialize()`**

The initialize() function in the EPNS CORE V1 contract is a special function that is used to initialize the contract when it is first deployed. This function can only be called once, and it is used to set up the contract's initial state.

The initialize() function performs the following tasks:

- Sets up the contract's addresses.
- Sets the contract's fees and other parameters.
- Returns a boolean value once all the values are initialized successfully.

Here is a detailed explanation of what each step does:

**Sets up the contract's addresses.**

The initialize() function sets up the contract's addresses by storing the following values in the contract's state:

- pushChannelAdmin: The address of the Push channel admin.
- governance: The address of the governance contract.
- daiAddress: The address of the DAI token contract.
- aDaiAddress: The address of the aDAI token contract.
- WETH_ADDRESS: The address of the WETH token contract.
- REFERRAL_CODE: The referral code for the EPNS platform.
- PUSH_TOKEN_ADDRESS: The address of the PUSH token contract.
- UNISWAP_V2_ROUTER: The address of the Uniswap V2 router contract.
- lendingPoolProviderAddress: The address of the Aave lending pool provider contract.

These addresses are used by the contract to interact with other contracts and to perform various operations.

**Sets the contract's fees and other parameters.**

The initialize() function sets the contract's fees and other parameters by storing the following values in the contract's state:

- CHANNEL_DEACTIVATION_FEES: The amount of fees that must be paid to deactivate a channel on the EPNS platform.
- ADD_CHANNEL_MIN_POOL_CONTRIBUTION: The minimum amount of PUSH tokens that must be contributed to the EPNS pool to add a new channel.
- ADD_CHANNEL_MIN_FEES: The minimum amount of fees that must be paid to add a new channel to the EPNS platform.
- ADJUST_FOR_FLOAT: A value that is used to adjust for floating-point errors.
- groupLastUpdate: The block number at which the group was last updated.
- groupNormalizedWeight: The normalized weight of the group

These fees and parameters are used by the contract to calculate various costs and determine how to distribute rewards.

**Returns a boolean value once all the values are initialized successfully.**

- The initialize() function returns a boolean value to indicate whether or not the initialization was successful. If the initialization was successful, the function will return true. Otherwise, the function will return false.

The initialize() function is a very important function in the EPNS CORE V1 contract. It is responsible for setting up the contract's initial state.

##### SETTER FUNCTONS

**updateWETHAddress():** This function allows the Push Channel Admin to update the address of the Wrapped Ether (WETH) contract. The WETH contract is used to wrap Ether so that it can be used in decentralized finance (DeFi) protocols.

**updateUniswapRouterAddress():** This function allows the Push Channel Admin to update the address of the Uniswap V2 router contract. The Uniswap V2 router contract is used to swap tokens on the Uniswap decentralized exchange.

**setEpnsCommunicatorAddress():** This function allows the Push Channel Admin to set the address of the EPNS Communicator contract. The EPNS Communicator contract is responsible for sending and receiving notifications on the EPNS protocol.

**setGovernanceAddress():** This function allows the Push Channel Admin to set the address of the Governance contract. The Governance contract is responsible for managing the EPNS protocol.

**setMigrationComplete():** This function allows the Push Channel Admin to set the isMigrationComplete flag to true. This flag is used to indicate that the migration of the EPNS protocol to the new contract is complete.

**setChannelDeactivationFees():** This function allows the Governance contract to set the fees that are charged when a channel is deactivated. These fees are used to fund the development and maintenance of the EPNS protocol.

**setMinChannelCreationFees():** This function allows the Governance contract to set the minimum fees that must be paid when creating a new channel. These fees are used to fund the development and maintenance of the EPNS protocol.

**transferPushChannelAdminControl():** This function allows the Push Channel Admin to transfer control of the Push Channel Admin role to a new address. This role is responsible for managing the global channel on the EPNS protocol.
