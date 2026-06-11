# forcedTransfer mutates allowance(from, recipient), not allowance(from, caller)
Project: CMTAT Fork
## Alan finding
Authorized forcedTransfer skips ordinary compliance checks by design, but its nonzero-recipient branch reduces allowance(from,to). That recipient allowance is unrelated to the forced-transfer manager's authority and can surprise allowance/accounting integrations.
## Potential exploit path
A forced-transfer manager, effectively including DEFAULT_ADMIN_ROLE in full variants, calls forcedTransfer(from, to, value). _forcedTransfer first unfreezes enough source tokens, then for nonzero to reads allowance(from, to), not allowance(from, msg.sender). If that allowance is positive and finite, it is set to zero when below value or reduced by value otherwise, then tokens are transferred directly. The privileged actor can therefore corrupt the user's pre-existing allowance relationship with the recipient while not spending any allowance granted to the administrative caller. Direct token movement is already privileged, so asset-transfer impact is role-limited, but allowance state and downstream accounting can be unexpectedly changed.

## Source references
- [contracts/modules/wrapper/extensions/ERC20EnforcementModule.sol:56-75 forcedTransfer](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/wrapper/extensions/ERC20EnforcementModule.sol#L56-L75) - Public forced-transfer entrypoints are only forced-transfer-manager gated.
- [contracts/modules/internal/ERC20EnforcementModuleInternal.sol:100-126 _forcedTransfer](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/internal/ERC20EnforcementModuleInternal.sol#L100-L126) - Implements unfreeze, burn/transfer branch, allowance(from,to) adjustment, and events.
- [contracts/modules/1_CMTATBaseAccessControl.sol:94-97 _authorizeForcedTransfer](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/1_CMTATBaseAccessControl.sol#L94-L97) - Full variants authorize forced transfer through DEFAULT_ADMIN_ROLE.


## PR status
This draft PR records an Alan fix request. Replace this review note with the code change before marking the PR ready for review.
