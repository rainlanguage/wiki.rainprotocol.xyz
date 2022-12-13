- main workhorse for any [[rainlang]] [[word]] (and is also a word itself) that wants to be function-like in that it has:
	- a reference to a source index that behaves like an [[entrypoint]]
	- 0+ inputs, which differentiates it from main entrypoints that only have 0 inputs
	- 0+ outputs, where 0 outputs may be useful in the case of [[set]]
	- an internal scope/stack while it runs
	- deallocates memory unused by outputs after it runs
- is not trying to be "a real function" (whatever that means in your language of choice), mostly only concerned with building then discarding a substack
	- optimised for looping where we set aside a known number of outputs after each loop iteration then discard the rest of the substack, this allows us to do [[stack deallocation]] and reuse a single region of memory for arbitrarily many loops
	- the _caller_ defines the number of outputs in a [[destructuring]] style on the LHS of [[rainlang]], this means the same source index could be [[variadic]] on its _outputs_ even though it has a fixed number of inputs. The following are equivalent but the latter uses less gas as it copies to and retains less data on the outer stack.
	  ```
	  _ _ _ foo: call<1 4 1>(1);
	  ```
	  vs.
	  ```
	  foo: call<1 1 1>(1);
	  ```
	  Note that generally entrypoints operate on the assumption that they produce a certain number of meaningful outputs and are free to refactor the rest of their substack including adding and removing values to their LHS, but this is a convention rather than something enforced by the [[expression deployer]].