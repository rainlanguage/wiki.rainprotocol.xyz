- contracts in scope
	- stake/Stake.sol
		- interpreter/run/LibContext.sol
		- tier/TierV2.sol
			- ITierV2.sol
		- tier/libraries/TierConstants.sol
		- tier/libraries/TierReport.sol
		- interpreter/shared/Rainterpreter.sol
			- interpreter/run/IInterpreterV1.sol
			- interpreter/ops/AllStandardOps.sol
				- interpreter/ops/core/OpReadMemory.sol
					- interpreter/run/LibStackPointer.sol
					- interpreter/run/LibInterpreterState.sol
					- interpreter/deploy/LibIntegrityCheck.sol
					- math/Binary.sol
				- interpreter/ops/math/OpAdd.sol
					- array/LibUint256Array.sol
				- interpreter/ops/core/OpContext.sol
				- type/LibCast.sol
				- type/LibConvert.sol
			- interpreter/run/LibEncodedDispatch.sol
			- interpreter/ops/core/OpGet.sol
			- kv/LibMemoryKV.sol
		- interpreter/shared/RainterpreterExpressionDeployer.sol
			- interpreter/deploy/LibIntegrityCheck.sol
			- interpreter/deploy/IExpressionDeployerV1.sol
- want to get deposit/withdraw throttling on [[Stake]] off to audit by EOY 2022
- calling these modifications [[stakepreter]] because everything needs to be either a pun on "rain" or "interpreter" of course
- It's already the 12th so not much time!
- luckily we're basically already in QA
	- PR is up for an [ERC4626](https://ethereum.org/en/developers/docs/standards/tokens/erc-4626/) compliant extension to `maxDeposit` and `maxWithdraw` to simply inject the [[interpreter/interface/IInterpreterV1]] interface on top of what [[Open Zeppelin]] does, with a cap of whatever OZ does
	- `maxMint` and `maxRedeem` are simply calculated by converting `maxDeposit` and `maxWithdraw` to shares from the asset throttling calculations
- auditors are [[Xord]] and they already said that auditing all the opcodes would be too large for scope
	- we decided to get just the core of the [[expression deployer]] and [[interpreter]] loops audited + whichever words are actually relevant to the first [[expression]] that we want to deploy with [[Stake]]
	- main thing will be deciding on the [[expression]] that we want to write so we know exactly what subset of the [[word/list]] would be relevant to get audited
- for audit will need
	- sweep of relevant [[interpreter]] and [[Stake]] contracts for documentation accuracy
	- appropriate QA/testing
	- deciding on the [[expression]]
	- fix known issues with [[interpreter]] that prevent prod deployment
		- notably [[expression registry]] and [[stack highwater]] plus removing deprecated code such as [[StandardInterpreter]]
- we MAY send to audit in parallel to test writing if we feel we've covered the basics enough with our inhouse QA, and can continue to backfill tests while the auditors review it all
- Some things worth reading for audit
	- [[interpreter/integration]]
	- [[word/list]]