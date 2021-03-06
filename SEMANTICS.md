# Semantics of WebAssembly Interface Types

This is intended largely to be a document of notes for the implemented and/or
envisioned semantics of the wasm interface types section. Be sure to consult
[BINARY.md](./BINARY.md) as well as the [official
repo](https://github.com/WebAssembly/interface-types) as well.

This is pretty unstructured, so beware.

* There's a wasm interface types type index space. It's separate from the core
  wasm index space.

* There's a wasm interface types function index space. It's separate from the
  core wasm function space.

* The presence of the wasm interface types section means that the core module's
  `export` section is basically ignored for semantic reasons. The exports of the
  module are exclusively looked up through the wasm interface types section.

* The imports of a module with wasm interface types is the set of imports from
  the wasm interface types section, plus the set of imports from the core wasm
  module, minus the set of `implements` items in the wasm interface types
  section.

* References to the core module are done through indices which are resolved
  relative the to the core module's index spaces.

* Can't implement the same function twice in the `implement` subsection

* Currently the `s32` type matches the `i32` type in wasm, same for `s64` and
  `i64`. This is used during validation when adapters hook up to core functions.
