- The integration point for [[offchain]] data is typically going to be [[signed context]] rather than a new [[onchain]] contract (e.g. some new [[oracle]] or [[interpreter]]).
  
  [[Signed context]] is a [[gas]] free operation for the [[signer]] that gives the [[expression]] caller a [[bearer proof]] of some data that is presumably relevant to the [[expression]].
- This can be used for many business flows:
	- Unlocking [[escrow]]
	- Reporting on outcomes of a [[game]]/[[event]]
	- [[Insurance]] style claims/proofs
	- Time bound commitments to a price from an [[offchain]] market
	- Etc.
- In most of these cases, as the source of truth is already [[offchain]] and probably [[centralised]], it makes little sense to attempt to replicate [[offchain]] data in some kind of [[oracle]]. If nothing else, the [[gas]] and [[infrastructure]] cost to maintain an [[oracle]] 24/7 are prohibitive compared to maintaining a [[server]] that can provide a [[signature]] ad-hoc upon request for key data.
  
  What is more important and useful than trying to write a new [[calling contract]] is that the [[expression]] defines who is allowed to sign ([[authorisation]]) and under what conditions, e.g. can a single signature be used twice or does it respect a [[nonce]]? is there an [[expiry time]]? does it have a [[domain separator]]? etc.