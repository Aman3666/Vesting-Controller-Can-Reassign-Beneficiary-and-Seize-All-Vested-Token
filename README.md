# Vesting-Controller-Can-Reassign-Beneficiary-and-Seize-All-Vested-Token
Affected Contract
src/vesting/LegionLinearVesting.sol


Source:
https://github.com/Legion-Team/legion-protocol-contracts/blob/master/src/vesting/LegionLinearVesting.sol#L103-L113

Severity

High — Total loss of funds

Root Cause
1. OpenZeppelin sets owner = beneficiary

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/finance/VestingWalletUpgradeable.sol#L53-L72

2. Release sends tokens to owner()

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/finance/VestingWalletUpgradeable.sol#L115-L137

3. Legion lets controller change the owner (beneficiary)

This function allows full takeover:

function emergencyTransferOwnership(address newOwner)
    external
    onlyVestingController
{
    _transferOwnership(newOwner);
}


Source:
https://github.com/Legion-Team/legion-protocol-contracts/blob/master/src/vesting/LegionLinearVesting.sol#L103-L113

Exploit Scenario

Alice is the beneficiary

Tokens are deposited

Controller calls emergencyTransferOwnership(attacker)

Anyone calls release(token)

All vested tokens go to the attacker instead of Alice

Result: Full theft. No revert.

Impact

Controller (or compromised controller) can steal all user vesting

Redirects all ERC20 + ETH to attacker

Breaks user trust assumptions

All deployed vesting contracts are at risk

Recommended Fix

Remove the emergencyTransferOwnership function
or

Require beneficiary signature approval
or

Add timelock + 2-step ownership transfer

Conclusion

This vulnerability enables complete takeover of vesting contracts and total loss of user funds. Immediate mitigation is required.
