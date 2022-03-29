Detecting unexpected bugs in Clarity contracts is possible, as described in [Clarity model-based testing primer](https://blog.nikosbaxevanis.com/2022/03/15/clarity-clarity-model-based-testing-primer) ([tl;dr](https://blog.nikosbaxevanis.com/2022/03/15/clarity-clarity-model-based-testing-primer/#summary)).

* Define one or more commands for each one of the public functions of the contract ([example](https://github.com/LNow/testing-example/blob/5f7eedf3edd9ee174650b59b4c181ddfdf713e41/tests/ci/example.commands.ts)).
  * Pick zero, one or more commands and run them sequentially if they can be executed on the current state ([example](https://github.com/LNow/testing-example/blob/5f7eedf3edd9ee174650b59b4c181ddfdf713e41/tests/example.test-ci.ts)).
* Define a TypeScript model ([clarinet tests are essentially deno tests](https://blog.nikosbaxevanis.com/2022/03/05/clarity-property-based-testing-primer/#internals)) as a simplified version of the contract. Commands can then check for potential problems or inconsistencies between the model and the contract.

Sample output taken from https://github.com/kenrogers/cargo/pull/1:

![image](https://user-images.githubusercontent.com/287532/164042854-66b780a4-fdcf-4147-861e-0ece51a4c1f1.png)

Sample output taken from https://github.com/LNow/testing-example/pull/2:

![image](https://user-images.githubusercontent.com/287532/165757840-bd2ea188-1d32-443a-a178-95afe45d61f9.png)

### Resources

* More info about the technique using fast-check and clarinet
  * [Detecting unexpected bugs](https://github.com/kenrogers/cargo/pull/1)
  * [Clarity model-based testing primer](https://blog.nikosbaxevanis.com/2022/03/15/clarity-clarity-model-based-testing-primer)
* Example code
  * https://github.com/kenrogers/cargo/pull/1/files
  * https://gist.github.com/moodmosaic/ec138d99346e11a3416b5ee216b20033#file-sup_test_model-ts
  * https://github.com/LNow/testing-example/pull/2/files
* Papers, talks, lectures, and books
  * [Testing the Hard Stuff and Staying Sane](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quviq-testing.pdf), John Hughes
  * [Do not write tests, generate them! / Curry On conf '17](https://www.youtube.com/watch?v=hXnS_Xjwk2Y), John Hughes
  * [Testing the Hard Stuff and Staying Sane / Clojure conf '14](https://www.youtube.com/watch?v=zi0rHwfiX1Q), John Hughes
  * [Model Checking](https://www.youtube.com/playlist?list=PLwabKnOFhE38C0o6z_bhlF_uOUlblDTjh), Joost-Pieter Katoen
  * [Principles of Model Checking (The MIT Press)](https://www.amazon.com/Principles-Model-Checking-MIT-Press/dp/026202649X), Christel Baier, Joost-Pieter Katoen
  * [Model Checking in Haskell, Part 1: Transition Systems and Invariants](https://benjaminselfridge.github.io/posts/2022-05-10-model-checking-1.html)

