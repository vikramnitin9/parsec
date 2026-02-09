# ParseC

ParseC parses a C program and writes a JSON representation of the parse tree (the AST).

ParseC uses the Clang compiler frontend to extract source-level syntactic information about the C program, and runs an LLVM pass to perform call-graph analysis. This allows it to sometimes resolve indirect calls (e.g., calls that involve function pointers), resulting an analysis that is potentially more complete than one that uses only Clang or libclang.

## Installation

### Prerequisites

ParseC is intended for use only with Linux. The following utilities and libraries are needed:

- `build-essential`
- `cmake`
- `bear`
- `llvm-14`, `llvm-14-dev`, `llvm-14-tools`, `clang-14`, `libclang-14-dev`

### Build

```sh
mkdir build && cd build
cmake .. && make
```

If compilation is successful, the `parsec` binary will be created in the `build` directory. You can add it to your `PATH`, or access it by its full file path.

## Usage

### Running on a single C file

`parsec` can be run on a single C file, say `hello.c`, as follows:

```sh
parsec hello.c
```

### Running on multiple C files

To run it on a full project, you need a `compile_commands.json` compilation database for your project. This can be obtained in a few different ways:

- If your project uses CMake, you can generate one automatically by setting [`CMAKE_EXPORT_COMPILE_COMMANDS`](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html).
- If your project uses `make`, you can use the `bear` utility to generate a compilation database:

  ```sh
  bear -- make
  ```

Once you have `compile_commands.json`, you can run `parsec` like this:

```sh
# From the directory containing compile_commands.json
parsec *.c
```
Here, `*.c` is a bash glob pattern representing all C files in your current directory. If your source files are in another directory, e.g., `src`, then run

```sh
# From the directory containing compile_commands.json
parsec src/*.c
```

### Options and Customization

`parsec` has two command-line flags, `--add-instr` and `--rename-main`. These are intended for internal use and debugging only.

## Output

The output of `parsec` is a single JSON file, `analysis.json`, created in the current working directory. The format of this file is described herein:

```json
{
  "files": [],
  "functions": [
    {
      "name": "str",
      "signature": "str",
      "num_args": "int",
      "argTypes": ["str"],
      "argNames": ["str"],
      "returnType": ["str"],
      "filename": "str",
      "startLine": "int",
      "endLine": "int",
      "startCol": "int",
      "endCol": "int",
      "callees": [
        {
          "name": "str"
        }
      ],
      "structs": [
        {
          "name": "str"
        }
      ],
      "enums": [
        {
          "name": "str"
        }
      ],
      "globals": [
        {
          "name": "str"
        }
      ]
    }
  ],
  "structs": [
    {
      "name": "str",
      "filename": "str",
      "startLine": "int",
      "endLine": "int",
      "startCol": "int",
      "endCol": "int"
    }
  ],
  "enums": [
    {
      "name": "str",
      "filename": "str",
      "startLine": "int",
      "endLine": "int",
      "startCol": "int",
      "endCol": "int"
    }
  ],
  "globals": [
    {
      "name": "str",
      "type": "str",
      "filename": "str",
      "startLine": "int",
      "endLine": "int",
      "startCol": "int",
      "endCol": "int",
      "isStatic": "bool"
    }
  ]
}
```

Note that the field `files` is intended to be populated with a list of source files and metadata about each one; however, this functionality is not yet implemented and the `files` field will currently always contain an empty list.
