- # context
- main concern is that we haven't done enough R&D into the threat vector of allowing untrusted expressions to run on the shared interpreter
- current flow of logic is something like this
	- [[expression deployer]] runs [[expression/integrity check]] over [[index opcodes]]
	- indexes are mapped to [[function pointers]] and the function pointers are the bytes that are deployed onchain with sstore2
	- [[calling contract]] receives address to onchain [[expression]] and passes it back to [[interpreter]] to run
	- [[interpreter]] loops over the [[function pointers]] it reads from the [[expression]] and runs them according to the [[function signature]] that it expects all [[opcodes]] to have during [[eval]]
- until now everything in the interpreter has been read only, all the opcode functions have been [[view]] or [[pure]] so any undefined behaviour during [[eval]] could arbitrarily corrupt the stack, but honest [[calling contracts]] can guard against return values that are so malformed as to be unreadable and have no control over stack values anyway, so arbitrary due to bad [[function pointers]] is just as arbitrary as arbitrary due to malicious [[expression]]
- now the standard shared [[interpreter]] writes to storage which means that there are some code paths in the compiled [[interpreter]] that modify storage that is written/read across all expressions
	- suddenly the attack surface allows a malicious expression to potentially run a codepath that would modify storage values that other expressions will later read, this would be a critical security issue, consider the case where alice places an order on the order book and bob manages to trick the [[interpreter]] into modifying alice's price lower to give bob arbitrarily cheap tokens
- the [[interpreter/interface/IInterpreterV1]] interface _appears_ safe as the only [[external]] function that accepts [[function pointers]] is [[view]] but how does [[Solidity]] actually enforce this?
	- [EIP-214](https://eips.ethereum.org/EIPS/eip-214) introduced [[STATICCALL]] which is the logic that the _caller_ uses to enforce rollback upon state changes, but what can the _callee_ do to protect itself?
	  
	  > This proposal adds a new opcode that can be used to call another contract (or itself) while disallowing any modifications to the state during the call (and its subcalls, if present). Any opcode that attempts to perform such a modification (see below for details) will result in an exception instead of performing the modification.
	  
	  It appears the expectation is that the callee would static call itself to invoke chain level storage protection. Given that the [[msg.sender]] within the [[interpreter]] during [[eval]] is the [[calling contract]] it appears either [[Solidity]] is doing nothing or a lot of magic here. I'd expect to see a static call to self manifest as the [[msg.sender]] being the [[interpreter]] itself as there's no "delegate static call" as far as i know.
	- As far as i know there's also no way to set the static flag in the evm outside of a static call, i.e. i can't just call some opcode directly that enables the flag inline with the current execution context (maybe this exists and i don't know about it yet?)
	- I assume what is happening here is that [[Solidity]] does NOT protect against using [[Yul]] assembly to directly coerce `uint` values to [[function pointers]]. i.e. normal operation is that the [[Solidity]] compiler treats [[function pointers]] as an internal type only (which makes sense as trying to return a [[function pointer]] from any [[external]] function is a compiler error), and uses a compile time proof to respect [[view]] on [[external]] functions and NOT an onchain static flag.
	- **We can test the assumption by spinning up a test contract that is designed to write to itself under a function that can be called directly by its [[function pointer]], then use [[Yul]] to coerce a `uint` passed in from an [[external]] and [[view]] call to an [[internal]] and [[view]] function, but then actually point this call to the [[internal]] function that writes to storage.**
- Given time constraints for this year, the R&D required to test the security issue, canvas potential fixes, assess the tradeoffs, etc. etc. seems a bit much, at the least we could
	- Create an [[expression registry]] where the [[expression deployer]] explicitly notifies the [[interpreter]] about each safe [[expression]]
		- Pros
			- Battle tested and simple approach, write a [[mapping]] and set an [[owner]] and be done with it
			- Very quick to deploy and easy to explain to an external auditor
			- Ability to avoid re-deploying expressions with identical bytecode as the registry can also behave as a cache
		- Cons
			- Requires a whole extra cold mapping write (lots of gas) for deployment, mostly this will impact [[Orderbook]] where adding expressions is part off [[add order]]
			- Requires modest additional gas on read to check the registry
			- Requires jumping through circular access checks during construction and initialization that are pretty clunky relative to current status quo, complicates initial deployments and care must be taken to ensure the deployment itself doesn't become a new attack vector
	- Apply a self static call as per EIP-214 so that there would be an [[external]] [[eval]] and an [[internal]] [[eval]], although the "internal" one would still be [[external]] but access gated to a self-call. This would still allow corrupt function pointers to be passed to [[eval]] and arbitrarily corrupt memory and the final stack, but would restrict the damage to something the [[calling contract]] can mitigate, rather than allow any [[expression]] to modify the state for any other [[expression]] globally.
		- Pros
			- Elegant in that it doesn't require juggling circular access logic and initialization/ownership between [[interpreter]] and [[expression deployer]]
			- Zero additional deployment gas
			- Automatically guards against all potential vulnerabilities of this nature now and forever at the chain layer rather than relying on the [[Solidity]] compiler to catch problems
		- Cons
			- Requires more low level concepts to understand and apply correctly for anyone implementing an [[interpreter]]
			- Does NOT require an additional trust relationship/assumption between [[interpreter]] and [[expression deployer]] so avoids the potential for a bad deployment due to code or otherwise
			- Maybe??? more gas??? no idea what the gas would be to forward [[calldata]] as-is to a self static call compared to reading an expression registry, neither sound either tiny or massive (assuming that the outer [[eval]] doesn't copy a lot of data to memory on either input or output)
			- Requires ensuring that [[msg.sender]] and [[address(this)]] are always provided by the [[calling contract]] although this seems like it's probably a good pattern anyway given that we're even considering doing this kind of thing, just to future proof the [[calling contract]] at the [[base context]] level