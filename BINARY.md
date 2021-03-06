# Binary Format

This crate invents a binary format for the wasm interface types subsection. This
document is intended to document what's currently implemented. We'll hope to
keep this up to date!

## Custom Section

Currently the wasm interface types binary encoding is encoded into a WebAssembly
custom section with the name `wasm-interface-types`. A custom section is chosen
as a temporary stopgap to continue to work with all existing tooling. It's
imagined that as wasm interface types progressed we'll graduate to a
full-fledged section.

Only one of these custom sections can appear in the WebAssembly module and the
contents of the custom section are parsed as below:


The layout of the custom section is:

```
wit-section := version:str sections:wit-subsection*
```

First up is a string that indicates the version of the encoding. This is
intended as a temporary stopgap to improve error messages while this binary
format is evolved. This version is defined in the `wit-schema-version` crate and
is intended to be bumped whenever the binary format changes.

Subsections are defined as follows:

```
wit-subsection := id:u8 contents:bytes
```

This is intended to be similar to WebAssembly sections. A byte indicates the id
of the subsection and then there's a list of bytes, where the bytes are preceded
with a uleb-encoded `u32` of how many bytes are in the subsection.

All subsections must appear at most once and in order of id for now.

## Index spaces

Like WebAssembly the wasm interface types section has its own sets of index
spaces. The two currently are:

* Types - indexed in order of their appearance in the type subsection
* Functions - indexed with imports first and then function definitions next

Note that these index spaces are intended to be separate from the core wasm
index spaces.

## Type Subsection (0)

The first section is the type subsection. The type subsection lists type
signatures used by other sections. The format of the type subsection is:

```
type-subsection := 0x00 types:vec(type)
```

The ID of this subsection is 0, and then there's a uleb-encoded `u32` of how
many types are in the subsection. Each type is encoded as:

```
type := params:vec(valtype) results:vec(valtype)
```

Value type encodings are best consulted by looking at the source, namely the
`wit-writer` and `wit-parser` crates.

## Import Subsection (1)

The import subsection lists the imported functionality from the host environment
using wasm interface types signatures.

```
import-subsection : = 0x01 imports:vec(import)
```

Like the type subsection, this is a list of `import` objects to parse. Each
import is defined as:

```
import := module:str name:str type:u32
```

The `module` and `name` are the wasm module/name that the functionality is
imported from. The `type` is the type signature in the type section for the
function being imported. Currently only function imports are supported.

## Function Subsection (2)

The function subsection contains the bodies of functions which work with wasm
interface types.

```
func-subsection : = 0x02 funcs:vec(func)
```

where each `func` is defined as:

```
func := body:bytes
```

Like WebAssembly each `func` is a blob of bytes with its size specified so they
can all be skipped if necessary. A function body is parsed as:

```
func-body := ty:u32 instrs:instr* end
```

Here the `ty` is the type signature index, there's then a whole bunch of
instructions which follow, and the instructions are terminated by the `end`
instruction.

## Export Subsection (3)

The export subsection lists the names that are exported from the wasm interface
types subsection.

```
export-subsection : = 0x03 exports:vec(export)
```

Like the type subsection, this is a list of `export` objects to parse. Each
export is defined as:

```
export := name:str func:u32
```

The `name` is how this export is reference, and the `func` is the wasm interface
types function index of what's being exported.

## Implement Subsection (4)

The implement subsection contains a list of functions to use to implement the
core module's imports.

```
implement-subsection : = 0x04 funcs:vec(implement)
```

where each `implement` is defined as:

```
implement := core-func:u32 func:u32
```

Here `core-func` is required to be an import in the core wasm module and `func`
is a function in the WebAssembly interface types section. Note that `func`'s
type signature must match `core-func`'s type signature.

## Instructions

Instruction encodings are best consulted by looking at the source, namely the
`wit-writer` and `wit-parser` crates.
