- such as detecting out of bounds stack reads, calculating the stack size for memory allocations, and other word-specific enforcements
- Each [[interpreter/interface/IInterpreterV1]] contract has at least one associated [[expression deployer/interface/IExpressionDeployerV1]]. The [[expression deployer]] has 1 exposed [[deployExpression]] function that is responsible for 2 things:
	- "Dry run" the expression to calculate both the [[net stack movement]] and [[gross stack movement]] and reads from the [[calling context]]
	- Provide an [[onchain]] address of the deployed [[expression]] in whatever format is compatible with the associated [[interpreter]].
- The [[StateConfig]] struct includes all the [[expression]] information as provided by the end user and the [[minOutputs]] is the number of outputs the [[interpreter/calling contract]] expects for each [[entrypoint]] it defines. This split prevents the end user from corrupting the [[expression]] by returning fewer results than the [[interpreter/calling contract]] needs.
  
  For example, if the 0th [[entrypoint]] to an [[expression]] requires at least 2 final values on the stack then index 0 of the [[minOutputs]] array MUST be 2. The length of the [[minOutputs]] array implicitly defines each [[entrypoint]] as all entrypoints MUST be at the start of the [[expression]] sources of a given [[StateConfig]]. The [[StateConfig]] MAY define more sources than there are entrypoints defined by the [[interpreter/calling contract]], and these are internally callable by the [[call]] [[word]]. This is similar to the distinction between [[internal]] vs. [[external]] functions in [[Solidity]].
  
  The [[interpreter/calling contract]] SHOULD respect the end user's [[StateConfig]] as provided and pass it directly to the [[expression deployer]] as-is without modification or interpretation.
  
  If the [[integrity check]] fails the [[expression deployer]] MUST error and the [[interpreter/calling contract]] MUST respect this error, rolling back the deployment. A failed [[integrity check]] can be a sign of [[out of bounds]] memory access and other [[security]] critical [[undefined behaviour]].
  
  **As EITHER the [[expression deployer]] or [[interpreter]] could be malicious, buggy or simply mismatched it is critical that [[expression]] only interacts with an [[expression]] that are paired [[immutably]] [[onchain]] with an [[expression deployer]] and [[interpreter]] they trust. Any mutability in the pairing onchain should be treated with extreme caution.**
  
  In practise the known pairings are matched at an [[expression metadata]] level offchain as [[interpreter lists]] similar to how [[token lists]] provide curated lists of tokens to mitigate phishing attempts in any [[front end]] that supports them. The [[expression metadata]] is fed into the subgraph indexer and made available and/or are filterable by any honest [[front end]] that wants to protect and assist their [[expression/auditor]] users.
  
  Of course, any [[phishing]]/malicious [[front end]] can also reference arbitrary [[phishing]]/malicious smart contracts, which is always the case with or without rain and so the [[DI pairing]] is no different or less important than selecting the correct "USDC" token address to approve/send/receive.
  
  **The [[interpreter/calling contract]] MUST pin the [[DI pairing]] along with the [[expression]] it is willing to [[eval]] so that [[expression/author]] cannot "[[bait and switch]]" the logic of [[expression]] by modifying the underlying execution environment.**