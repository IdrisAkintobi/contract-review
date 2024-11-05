# WildcatArchController Smart Contract Audit Report

## ⚠️ Disclaimer ⚠️

This audit of the smart contract is based on the information provided and my expertise in the field. It is important to note that this audit does not guarantee the security or functionality of the smart contract, nor does it eliminate all potential risks. I will not be held responsible for any actions taken based on this audit. Users are encouraged to conduct their own research and due diligence before relying on this audit for any specific purpose.

| Client         | Web3Bridge                                         |
| :------------- | :------------------------------------------------- |
| Title          | Smart Contract Audit Report                        |
| Target         | WildcatArchController                              |
| Version        | 0.0.1                                              |
| Author         | [Idris Akintobi](https://github.com/IdrisAkintobi) |
| Classification | Public                                             |
| Status         | Draft                                              |
| Date Created   | November 5, 2024                                   |

## Table of Contents

1. [Introduction](#introduction)
2. [Contract Overview](#contract-overview)
3. [Contract Dependencies](#contract-dependencies)
4. [Key Components](#key-components)
5. [State Variables](#state-variables)
6. [Events, Errors, and Modifiers](#events-errors-and-modifiers)
7. [Access Control](#access-control)
8. [Core Functionality Analysis](#core-functionality-analysis)
9. [SphereX Integration](#spherex-integration)
10. [Security Analysis](#security-analysis)
11. [Code Quality Analysis](#code-quality-analysis)
12. [Recommendations](#recommendations)
13. [Conclusion](#conclusion)

## Introduction

This report provides a comprehensive audit of the `WildcatArchController` contract from the Wildcat protocol repository. The Wildcat protocol enables borrowers to deploy highly configurable, un(der)collateralized on-chain credit solutions.

## Contract Overview

The `WildcatArchController` is a central registry and permission management contract for the Wildcat protocol. It manages borrowers, controllers, markets, and maintains a blacklist of assets. The contract inherits from SphereXConfig and Ownable, providing security and access control functionality.

```markdown
### Components

| 📝 Contracts | 📚 Libraries | 🔍 Interfaces | 🎨 Abstract |
| ------------ | ------------ | ------------- | ----------- |
| 1            | 0            | 0             | 0           |

**Exposed Functions**

This section lists functions that are explicitly declared public or payable. Please note that getter methods for public stateVars are not included.

| 🌐 Public | 💰 Payable |
| --------- | ---------- |
| 31        | 0          |

| External | Internal | Private | Pure | View |
| -------- | -------- | ------- | ---- | ---- |
| 31       | 16       | 0       | 0    | 20   |

**State Variables**

| Total | 🌐 Public |
| ----- | --------- |
| 5     | 0         |

**External Dependencies**

| Dependency / Import Path                               | Count |
| ------------------------------------------------------ | ----- |
| openzeppelin/contracts/utils/structs/EnumerableSet.sol | 1     |
| solady/auth/Ownable.sol                                | 1     |
```

## Contract Dependencies

1. **OpenZeppelin's EnumerableSet**: Used for efficient set operations
2. **Solady's Ownable**: Provides ownership management
3. **SphereXConfig**: Handles SphereX integration and configuration
4. **MathUtils**: Custom mathematics utilities
5. **ISphereXProtectedRegisteredBase**: Interface for SphereX protected contracts

#### EnumerableSet

```solidity
import { EnumerableSet } from 'openzeppelin/contracts/utils/structs/EnumerableSet.sol';
```

`EnumerableSet` is imported from OpenZeppelin's library and provides set operations specifically for address sets. It is used to store and manage sets of addresses such as `_markets`, `_controllerFactories`, `_borrowers`, `_controllers`, and `_assetBlacklist`. The library helps avoid duplicate entries and ensures efficient lookups, additions, and deletions.

#### Ownable

```solidity
import 'solady/auth/Ownable.sol';
```

The `Ownable` contract, sourced from the Solady library, provides ownership functionality, restricting access to specific functions to only the owner of the contract. In `WildcatArchController`, `onlyOwner` is used to restrict certain functions, like adding/removing borrowers and blacklisting assets, to be callable only by the contract owner.

#### SphereXConfig

```solidity
import './spherex/SphereXConfig.sol';
```

`SphereXConfig` is an abstract contract that handles administrative roles within the protocol. It allows the setup and management of the `SphereX` admin, operator, and engine roles, which are critical for protocol governance. `SphereXConfig` includes storage slots for these roles, events for changes, and functions to transfer admin privileges.

#### MathUtils

```solidity
import './libraries/MathUtils.sol';
```

`MathUtils` is a library containing utility functions for mathematical operations. In `WildcatArchController`, it is used for operations like setting boundaries on the length of arrays in functions.

#### ISphereXProtectedRegisteredBase

```solidity
import './interfaces/ISphereXProtectedRegisteredBase.sol';
```

`ISphereXProtectedRegisteredBase` defines the `changeSphereXEngine` function interface, enabling interaction with SphereX's engine to modify permissions dynamically. This interface allows `WildcatArchController` to use SphereX-protected features and manage access to protocol-sensitive actions.

---

## Key Components

The contract manages four main registries:

- Borrowers
- Controller Factories
- Controllers
- Markets
  And maintains an asset blacklist.

## State Variables

```solidity
EnumerableSet.AddressSet internal _markets;
EnumerableSet.AddressSet internal _controllerFactories;
EnumerableSet.AddressSet internal _borrowers;
EnumerableSet.AddressSet internal _controllers;
EnumerableSet.AddressSet internal _assetBlacklist;
```

All state variables use OpenZeppelin's EnumerableSet for efficient set operations and enumeration capabilities.

- **\_markets**: Stores registered market addresses.
- **\_controllerFactories**: Holds addresses of controller factories with deployment permissions.
- **\_borrowers**: Contains a set of approved borrowers.
- **\_controllers**: Manages addresses of active controllers.
- **\_assetBlacklist**: List of blacklisted assets.

## Events, Errors, and Modifiers

### Events

```solidity
event MarketAdded(address indexed controller, address market);
event MarketRemoved(address market);
event ControllerFactoryAdded(address controllerFactory);
event ControllerFactoryRemoved(address controllerFactory);
event BorrowerAdded(address borrower);
event BorrowerRemoved(address borrower);
event AssetBlacklisted(address asset);
event AssetPermitted(address asset);
event ControllerAdded(address indexed controllerFactory, address controller);
event ControllerRemoved(address controller);
```

These events signal changes within the protocol, assisting with traceability and enabling an external observer to track state changes.

### Errors

```solidity
error NotControllerFactory();
error NotController();
error BorrowerAlreadyExists();
error ControllerFactoryAlreadyExists();
error ControllerAlreadyExists();
error MarketAlreadyExists();
error BorrowerDoesNotExist();
error AssetAlreadyBlacklisted();
error ControllerFactoryDoesNotExist();
error ControllerDoesNotExist();
error AssetNotBlacklisted();
error MarketDoesNotExist();
```

Custom Errors are used to handle invalid actions, ensuring control over unauthorized access or duplicate entries.

### Modifiers

- **onlyControllerFactory**: Restricts functions to only callable by controller factories.
- **onlyController**: Limits function execution to verified controllers.

## Access Control

The contract implements multiple levels of access control:

1. **Owner Functions**:

   - registerBorrower
   - removeBorrower
   - addBlacklist
   - removeBlacklist
   - registerControllerFactory
   - removeControllerFactory
   - removeController
   - removeMarket

2. **Controller Factory Functions**:

   - registerController

3. **Controller Functions**:
   - registerMarket

## Core Functionality Analysis

### Borrower Management

```solidity
function registerBorrower(address borrower) external onlyOwner
function removeBorrower(address borrower) external onlyOwner
function isRegisteredBorrower(address borrower) external view returns (bool)
function getRegisteredBorrowers() external view returns (address[] memory)
```

- Proper access control via onlyOwner
- Events emitted for state changes
- View functions for querying borrower status

### Asset Blacklist

```solidity
function addBlacklist(address asset) external onlyOwner
function removeBlacklist(address asset) external onlyOwner
function isBlacklistedAsset(address asset) external view returns (bool)
function getBlacklistedAssets() external view returns (address[] memory)
```

- Maintains a blacklist of assets
- Owner-only access for modifications
- Appropriate event emissions

### Controller Factory Management

```solidity
function registerControllerFactory(address factory) external onlyOwner
function removeControllerFactory(address factory) external onlyOwner
function isRegisteredControllerFactory(address factory) external view returns (bool)
```

- Manages controller factory registration
- Integrates with SphereX through \_addAllowedSenderOnChain

### Controller Management

```solidity
function registerController(address controller) external onlyControllerFactory
function removeController(address controller) external onlyOwner
function isRegisteredController(address controller) external view returns (bool)
```

- Two-tier access control (owner and factory)
- Proper event emissions for tracking

### Market Management

```solidity
function registerMarket(address market) external onlyController
function removeMarket(address market) external onlyOwner
function isRegisteredMarket(address market) external view returns (bool)
```

- Controllers can register markets
- Owner can remove markets
- SphereX integration for registered markets

## SphereX Integration

The contract includes comprehensive SphereX integration:

```solidity
function updateSphereXEngineOnRegisteredContracts(
    address[] calldata controllerFactories,
    address[] calldata controllers,
    address[] calldata markets
) external spherexOnlyOperatorOrAdmin
```

- Allows updating SphereX engine for registered contracts
- Maintains security through spherexOnlyOperatorOrAdmin modifier
- Handles both engine updates and sender allowances

## Security Analysis

### Strengths

1. Comprehensive access control system
2. Clear error handling with custom errors
3. Event emission for all state changes
4. Use of battle-tested OpenZeppelin libraries
5. SphereX integration for additional security

### Potential Concerns

1. No time-delay or multi-signature requirements for critical operations
2. Centralized control through single owner
3. No explicit pause mechanism for emergency situations

## Code Quality Analysis

1. **Modularity**: Well-organized code with clear separation of concerns
2. **Documentation**: Good inline documentation for functions
3. **Naming**: Clear and consistent naming conventions
4. **Testing Coverage**: The contract test coverage is excellent

## Recommendations

### Security Improvements

1. Add emergency pause functionality

The contract lacks any pause mechanism. Given that it's the central registry controlling market deployments, this is crucial.

Recommended Addition:

```solidity
bool public paused;
modifier whenNotPaused() {
    require(!paused, "Contract is paused");
    _;
}

function setPaused(bool _paused) external onlyOwner {
    paused = _paused;
    emit PauseStatusChanged(_paused);
}

// Apply to critical functions
function registerMarket(address market) external onlyController whenNotPaused {
    // existing logic
}
```

### Code Quality Improvements

1. Add more detailed NatSpec documentation
2. Implement comprehensive input validation

Current Implementation:

```solidity
function registerControllerFactory(address factory) external onlyOwner {
    if (!_controllerFactories.add(factory)) {
        revert ControllerFactoryAlreadyExists();
    }
    _addAllowedSenderOnChain(factory);
    emit ControllerFactoryAdded(factory);
}
```

Recommended Implementation:

```solidity
function registerControllerFactory(address factory) external onlyOwner {
    if (factory == address(0)) revert InvalidAddress();
    if (factory.code.length == 0) revert NotAContract();
    if (!IControllerFactory(factory).supportsInterface(type(IControllerFactory).interfaceId)) {
        revert InvalidControllerFactory();
    }
    if (!_controllerFactories.add(factory)) {
        revert ControllerFactoryAlreadyExists();
    }
    _addAllowedSenderOnChain(factory);
    emit ControllerFactoryAdded(factory);
}
```

3. Add more detailed error messages
4. Optimized Getters with Pagination

Current Implementation:

```solidity
function getRegisteredBorrowers(
    uint256 start,
    uint256 end
) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
        arr[i] = _borrowers.at(start + i);
    }
}
```

Recommended Implementation:

```solidity
function getRegisteredBorrowers(
    uint256 start,
    uint256 pageSize
) external view returns (
    address[] memory arr,
    uint256 total,
    bool hasMore
) {
    uint256 len = _borrowers.length();
    uint256 end = MathUtils.min(start + pageSize, len);
    uint256 count = end - start;

    arr = new address[](count);
    unchecked {
        for (uint256 i = 0; i < count; ++i) {
            arr[i] = _borrowers.at(start + i);
        }
    }

    return (arr, len, end < len);
}
```

5. Cache external calls results when used multiple times

Current Implementation in SphereX update:

```solidity
function _updateSphereXEngineOnRegisteredContractsInSet(
    EnumerableSet.AddressSet storage set,
    address engineAddress,
    address[] memory contracts,
    bytes memory changeSphereXEngineCalldata,
    bytes memory addAllowedSenderOnChainCalldata,
    bytes4 notInSetErrorSelectorBytes
) internal {
    for (uint256 i = 0; i < contracts.length; i++) {
        address account = contracts[i];
        if (!set.contains(account)) {
            uint32 notInSetErrorSelector = uint32(notInSetErrorSelectorBytes);
            assembly {
                mstore(0, notInSetErrorSelector)
                revert(0x1c, 0x04)
            }
        }
        _callWith(account, changeSphereXEngineCalldata);
        if (engineAddress != address(0)) {
            assembly {
                mstore(add(addAllowedSenderOnChainCalldata, 0x24), account)
            }
            _callWith(engineAddress, addAllowedSenderOnChainCalldata);
            emit_NewAllowedSenderOnchain(account);
        }
    }
}
```

Recommended Implementation:

```solidity
function _updateSphereXEngineOnRegisteredContractsInSet(
    EnumerableSet.AddressSet storage set,
    address engineAddress,
    address[] memory contracts,
    bytes memory changeSphereXEngineCalldata,
    bytes memory addAllowedSenderOnChainCalldata,
    bytes4 notInSetErrorSelectorBytes
) internal {
    bool hasEngine = engineAddress != address(0);
    uint256 len = contracts.length;

    for (uint256 i = 0; i < len;) {
        address account = contracts[i];
        if (!set.contains(account)) {
            uint32 notInSetErrorSelector = uint32(notInSetErrorSelectorBytes);
            assembly {
                mstore(0, notInSetErrorSelector)
                revert(0x1c, 0x04)
            }
        }
        _callWith(account, changeSphereXEngineCalldata);
        if (hasEngine) {
            assembly {
                mstore(add(addAllowedSenderOnChainCalldata, 0x24), account)
            }
            _callWith(engineAddress, addAllowedSenderOnChainCalldata);
            emit_NewAllowedSenderOnchain(account);
        }
        unchecked { ++i; }
    }
}
```

### Operational Improvements

1. Add logging for administrative actions

## Conclusion

The `WildcatArchController` contract is well-structured, with tests covering key functionalities. Test cases validate the protocol’s access control mechanisms, registry functions, and error handling effectively. Applying recommended optimizations can further enhance performance and security.
