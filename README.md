# Vesting Controller Vulnerability 

## 📖 Overview
- **Github guide:** https://github.com/Legion-Team/legion-protocol-contracts/blob/master/src/vesting/LegionLinearVesting.sol
- **Affected contract:** src/vesting/LegionLinearVesting.sol
- **Vulnerability:** Controller can reassign beneficiary (owner) → steal all vested tokens  
- **Severity:** High — total loss of funds  
- **Status:** Unpatched (as of [commit hash])

---

## 🔎 Root Cause

1. `owner() == beneficiary` in OpenZeppelin’s `VestingWalletUpgradeable`.  
2. `release()` transfers to `owner()`.  
3. `emergencyTransferOwnership()` allows controller to change `owner()` arbitrarily.

See full details below.

---

## 🚨 Full Description

### 1. Initialization  
- `__VestingWallet_init(beneficiary, …)` sets `owner = beneficiary`.  

### 2. Token/ETH Release  
- `release(token)` / `release()` sends to `owner()`.  

### 3. Dangerous Ownership Transfer  
```solidity
function emergencyTransferOwnership(address newOwner)
    external
    onlyVestingController
{
    _transferOwnership(newOwner);
}

```

### Impact

This function allows the vestingController to change the contract owner at any time. Since `owner()` is also the beneficiary in `VestingWalletUpgradeable`, replacing the owner immediately replaces the beneficiary. As a result, all vested and future vesting releases are redirected to the `newOwner` address. This enables a complete takeover of the vesting contract and allows an attacker (or compromised controller) to steal all vested tokens in a single step, without user approval or any security checks.
