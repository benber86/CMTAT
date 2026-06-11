# Configured snapshotEngine can halt all token updates after validation
Project: CMTAT Fork
## Alan finding
When snapshotEngine is nonzero, _update performs the ERC20 balance update and then calls snapshotEngine.operateOnTransfer with pre-state values. A reverting configured engine reverts the entire operation, so transfer/mint/burn liveness depends on a role-configured external callback.
## Potential exploit path
A SNAPSHOOTER_ROLE holder, or DEFAULT_ADMIN_ROLE through effective super-role behavior, calls setSnapshotEngine with an engine whose operateOnTransfer always reverts or conditionally reverts. Any later token operation reaching _update—ordinary transfer/transferFrom, mint, burn, forcedTransfer, or cross-chain mint/burn in full variants—passes earlier validation and ERC20 checks, executes ERC20Upgradeable._update, then calls the external snapshot engine. The revert rolls back the full operation, creating a role-configured denial of service for core token movement. Impact is configuration/role-limited and can be remediated only by an authorized change to the engine if such an actor remains available.

## Source references
- [contracts/modules/0_CMTATBaseCommon.sol:114-137 _update](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/0_CMTATBaseCommon.sol#L114-L137) - Reads pre-state, performs ERC20 update, then calls snapshotEngine.operateOnTransfer when configured.
- [contracts/modules/wrapper/extensions/SnapshotEngineModule.sol:57-77 setSnapshotEngine/snapshotEngine](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/wrapper/extensions/SnapshotEngineModule.sol#L57-L77) - Role-gated configuration selects the external callback target.
- [contracts/interfaces/engine/ISnapshotEngine.sol:8-21 operateOnTransfer](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/interfaces/engine/ISnapshotEngine.sol#L8-L21) - External callback interface invoked by token updates.
- [contracts/modules/1_CMTATBaseAccessControl.sol:101-104 _authorizeSnapshots](https://github.com/benber86/CMTAT/blob/49544f4de1993008acfc9e848d0bf03bd31d8579/contracts/modules/1_CMTATBaseAccessControl.sol#L101-L104) - Snapshot engine configuration is gated by SNAPSHOOTER_ROLE, with DEFAULT_ADMIN_ROLE effective authority through hasRole.


## PR status
This draft PR records an Alan fix request. Replace this review note with the code change before marking the PR ready for review.
