- You may have noticed in the [[API documentation]] that [[eval]] and stateChanges both have `FooWithNamespace` variants. The [[interpreter/namespace]] is used to further disambiguate [[word/get]] and [[word/set]] *between any two [[expression]] from the same [[calling contract]]*.
  
  For a simple [[factory]] based deployment [[interpreter/namespace]] is typically NOT required as all the [[expression]] are deployed together by the same [[expression/author]] and are not changed and cannot be impacted by another [[calling contract/malicious]].
  
  For a more complex setup such as [[Orderbook]] there are many [[expression/author]] deploying many [[expression]] under the same [[calling context]] . In this case we cannot allow a [[word/set]] from one [[expression/author]] to overwrite the same key as used by another [[expression/author]]. That would be a critical security issue as users could attack each other's orders by carefully constructing keys to collide, thus changing prices and amounts of live orders of an [[order book/counterparty]].
  
  In such cases the [[calling contract]] can define an [[interpreter/namespace]] ([[Orderbook]] uses the [[Orderbook/order/owner]] as an [[interpreter/namespace]] for all their [[Orderbook/order]]) that sandbox state changes beyond the default per-caller.
  
  The default behaviour of the reference [[interpreter]] is to sandbox all state as a [[mapping]].
  
  ```solidity
  // state is several tiers of sandbox
  // 0. address is msg.sender so that callers cannot attack each other
  // 1. StateNamespace is caller-provided namespace so that expressions cannot attack each other
  // 2. uint is expression-provided key
  // 3. uint is expression-provided value
  mapping(address => mapping(StateNamespace => mapping(uint => uint)))
      internal state;
  ```
  
  This provides one tier of [[mapping]] for each [[onchain]] entity that could mount an attack on another at the same level.
- Two [[calling contract/malicious]] calling contracts could try to attack each other by attempting to deploy expressions under the same namespace
- Two [[expression/author/malicious]] deploying [[expression/malicious]] under a single [[calling contract]] could try to attack each other by crafting a [[key collision]] with their [[expression/malicious]].
- Two [[interpreter/malicious]] could try to attack each other if they shared some [[storage]] contract other than their own internal [[storage]] [[mapping]]
  
  Note that [[expression]] from the same [[calling contract]] and [[interpreter/namespace]] MAY have a [[key collision]] if their keys are the same. [[expression/author]] MUST take care to ensure keys don't _accidentally_ collide across [[word/get]] and [[word/set]] within the same [[interpreter/namespace]]. The simplest way to achieve this is to use [[word/hash]] to build a [[compound key]] out of [[constant]] and [[dynamic]] values at [[runtime]].
  
  The possibility of [[expression]] [[key collision]] is intentional as [[expression/author]] MAY deploy a set of related [[expression]] that all share some [[state]] even though they [[eval]] independently. An example of this could be imposing global token mint/transfer limits across several independent methods of minting/transferring.