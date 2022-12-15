- https://github.com/beehive-innovation/rain-protocol/pull/491
-
- 2 rules:
- all [[multioutput opcodes]] MUST move the [[stack highwater]] past ALL their outputs
- [[stack]] MUST move the highwater past the index it reads from
- the idea is to prevent nesting multioutputs or consuming things that have been copied from the stack
- it is an [[integrity error]] to pop below the highwater, it will be treated exactly as a stack underflow has been treated up to this point
- the net result is that certain operations suck as [[stack]] and anything multioutput such as [[call]] or even inputs to a [[call]] [[entrypoint]] become immutable on the stack during an [[eval]] and can only be copied in order to be read
- # Context
-
- previously in [[rainlang]] we allowed `_` on the RHS when some [[word]] [[pushes]] more than one value on the [[stack]], this is confusing
	- if the mere presence of a [[word]] implies 1 value on the [[stack]] it is hard to say what the correct behaviour and visual representation should be for things that put 0 values on the stack e.g. [[word/ensure]]
	- having multiple outputs can break otherwise reasonable expectations/assumptions like addition being commutative consider:
	  ```
	  _ _ _: _ _ add(_ call<1 4 1>(x) 1 2);
	  ```
	  vs
	  ```
	  _ _ _: _ _ add(1 2 _ call<1 4 1>(x));
	  ```
	  where the latter is actually impossible and would error as it implies that the values `1 2` somehow inject themselves _between_ the outputs, but the onchain logic has no way to do that because stacking constants also requires calling words, even if it did it would be gas prohibitive to be detecting and rearranging or linking values together in memory
- There's a related issue with copying things from the stack, in rainlang that is most naturally done by naming something on the LHS then referencing it later on the RHS e.g.
  ```
  /* name foo */
  foo: add(1 2),
  /* use foo */
  bar: add(foo 3); 
  ```
  this expands the RHS label to `stack(n)` where n is the index of the same label on the LHS.
  there is an issue where the raw bytes representation can consume values on the RHS that are higher up on the LHS, which would mutate what `n` points to e.g.
  ```
  /* foo @ n = 0 */
  foo: 1,
  /* add with 2 inputs consumes 1 and also foo */
  /*  now bar is @ n = 0 */
  bar: add<2> 1,
  /* baz is 3 not 2! */
  /* this is because foo is an alias to an index, which bar was written to */ 
  baz: add(foo 1);
  ```
  this seems to be solvable by having the parser simply disallow "raw" operands that behave the same as parens without the parens, e.g. only the following is allowed, much clearer as to why the final result is 3.
  ```
  foo: add(1 1),
  bar: add(foo 1);
  ```
  however there's still a legibility issue when `stack` appears in a nested context because the formatter has to deal with raw bytes that can be put onchain as long as they pass integrity checks, so it has no control over operands, meaning this is allowable without a highwater:
  ```
  /* result is 4 */
  _: add(add(1 stack(0)) stack(0));
  ```
  in this case we have nested `stack` calls where the nested stacks point by index to values that were pushed to the stack in the same nesting, i.e. within a single nesting some value is both popped and copied by stack such that `stack(n)` returns a different value for a given `n`. This isn't a correctness issue really, but it requires some brain juices to track that `stack(0)` is 1 then on the same line it is 2. 
  
  **We'd like to be able to enforce that `stack(n)` is [[idempotent]].**
  
  The following would be allowed, giving the same final value of 4 but enforcing idempotence of `stack`:
  ```
  _: 1,
  _: add(1 stack(0)),
  _ add(stack(1) stack(1));
  ```
  This is a gas/memory tradeoff against legibility as the final stack height is 3 rather than 1 and there is an extra call to `stack` to copy the inner addition to the outer addition. Luckily stack is one of, if not the cheapest opcode, and is almost a noop relative to the base cost of looping over its [[function pointer]].
-
- We also want to apply the same idempotence rules to inputs to [[call]] , if there are n inputs then the first n values are copy-on-read, all the same reasoning as [[stack]] applies here because in [[rainlang]] [[call]] inputs always appear on the LHS with no corresponding RHS, which is the same as saying that they cannot be nested, so we should enforce their highwater also