- Ethereum.org lists [5 possible upgrade methods](https://ethereum.org/fr/developers/docs/smart-contracts/upgrading/).
	- 1. Creating multiple versions of a smart contract and [[migrating]] users/state
	  2. Separate [[logic]]/[[state]] contracts
	  3. [[Proxy pattern]] to [[delegate call]]
	  4. [[Immutable]] main contract that reads from "satellite" contracts
	  5. [[Diamond pattern]] to [[delegate call]]
	- Of these 3 and 5 are examples of [[delegate call]], wherein the [[callee]] can modify the state of the [[word/caller]]. Upgrades that rely on this mechanism also force a [[trust relationship]] between the [[state]] and [[logic]] contracts, where the [[logic]] contract can arbitrarily manipulate/corrupt the [[state]] contract. This [[trust relationship]] extends to whoever (some [[admin key]]) can decide which [[logic]] contract is used.
	  Option 2 also requires "you" (read: centralised [[admin key]]) to [[authorise]] changes and prevent malicious/buggy code modifying [[state]]. It also elevates the [[admin key]] to a single point of failure/manipulation for the contract.
	  Option 1 is a hardcore [[decentralised]] option, a high profile successful historical example is the SAI to DAI [[migration]] where the ability to support collateral other than ETH was added to the DAI ecosystem by launching an entirely new token and burning the old one. These migrations tend to be very simple at the technical layer and very messy at the social layer.
	- [[Rain]] [[interpreter]] is an example of option 4:
		- > The main contract in this case contains the core business [[logic]], but interfaces with other smart contracts ("satellite contracts") to execute certain functions. This main contract also stores the address for each satellite contract and can switch between different implementations of the satellite contract.
		- > You can build a new satellite contract and configure the main contract with the new address. This allows you to change strategies (i.e., implement new [[logic]]) for a smart contract.
		- > Although similar to the [[proxy pattern]] discussed earlier, the [[strategy pattern]] is different because the main contract, which users interact with, holds the business [[logic]]. Using this pattern affords you the opportunity to introduce limited changes to a smart contract without affecting the core [[infrastructure]].
		- The combination of a deployed [[expression]] and the [[interpreter]] it is intended for allows end users to write their own satellites, with the main contract providing a high level intent and safety rails for users.
		- By allowing end users to specify their own [[interpreter]] and [[expression]] the [[interpreter/calling contract]] can remain immutable and benefit from each new [[word]] and other features as they become available in newer [[interpreter]], or users can benefit from sticking with higher [[lindy]] [[interpreter]] versions when they can achieve what they need with an older version.
	- So far we have identified 2 patterns where end users can self-select [[interpreter]].
	  The `[[Flow]]` contract is an example of the first and `[[Orderbook]]` the second.
		- 1. Upon creation of a [[child contract]] from a [[factory]] the [[interpreter]] and [[expression]] can be set as [[initialisation]] parameters
		  2. When the calculations and [[state]] changes can ONLY damage the [[interpreter]] selector they are free to select any [[interpreter]] they want, as by definition a bad choice cannot harm anyone else
		-
	- Note also that these approaches are NOT mutually exclusive. Some team MAY choose to use BOTH an [[admin key]] based upgrade path in addition to using the [[Rain]] [[interpreter/interface]] . Using [[Rain]] is neither a guarantee nor a requirement that the [[interpreter/calling contract]] is [[immutable]] and [[trustless]] re: upgrade paths.