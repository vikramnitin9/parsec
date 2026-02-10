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
# [Method 1] If all your source files are located at the top level of some directory, say `src`
# From the directory containing compile_commands.json, run
parsec src/*.c
```

Here, `*.c` is a bash glob pattern representing all C files in your current directory. The glob pattern will be expanded _by your shell_ before being passed to `parsec`. Note that this will not work if you have source files in multiple nested directories below the current directory. In that case, if your shell supports recursive globs, use this:

```sh
# [Method 2] Only if your shell supports recursive globs!
parsec **/*.c
```

If your shell does not support recursive globs, then you have no choice but to list all the subdirectories individually like this:

```sh
# [Method 3] 
parsec src/a/*.c src/a/b/*.c src/d/*.c
```

### Options and Customization

`parsec` has two command-line flags, `--add-instr` and `--rename-main`. These are intended for internal use and debugging only.

## Output

The output of `parsec` is a single JSON file, `analysis.json`, created in the current working directory. This file contains a structured representation of information extracted from the analyzed source code.

The sections below document the full structure and meaning of each field in the output. The JSON shown is illustrative; actual output will contain concrete values.

---

## Example Structure

```json
{
  "files": [],
  "functions": [],
  "structs": [],
  "enums": [],
  "globals": []
}
```

---

## Top‑Level Fields

| Field       | Type  | Description                                                                              |
| ----------- | ----- | ---------------------------------------------------------------------------------------- |
| `files`     | array | List of source files and their metadata. **Currently not implemented** and always empty. |
| `functions` | array | Functions discovered in the analyzed source code.                                        |
| `structs`   | array | Struct definitions found in the codebase.                                                |
| `enums`     | array | Enum definitions found in the codebase.                                                  |
| `globals`   | array | Global variables found in the codebase.                                                  |

---

## `files[]`

> **Status:** Not yet implemented

This field is reserved for future use and will eventually contain metadata for each source file processed by `parsec`.

Expected fields may include (subject to change):

| Field      | Type   | Description                     |
| ---------- | ------ | ------------------------------- |
| `filename` | string | Path or name of the source file |
| `language` | string | Detected programming language   |
| `hash`     | string | Content hash of the file        |

---

## `functions[]`

Each entry represents a single function definition.

### Fields

| Field        | Type     | Description                                  |
| ------------ | -------- | -------------------------------------------- |
| `name`       | string   | Function name                                |
| `signature`  | string   | Full function signature as written in source |
| `num_args`   | integer  | Number of arguments                          |
| `argTypes`   | string[] | Types of each argument                       |
| `argNames`   | string[] | Names of each argument                       |
| `returnType` | string[] | Return type(s)                               |
| `filename`   | string   | Source file containing the function          |
| `startLine`  | integer  | Line number where the function starts        |
| `endLine`    | integer  | Line number where the function ends          |
| `startCol`   | integer  | Column where the function starts             |
| `endCol`     | integer  | Column where the function ends               |
| `callees`    | object[] | Functions called by this function            |
| `structs`    | object[] | Structs referenced by this function          |
| `enums`      | object[] | Enums referenced by this function            |
| `globals`    | object[] | Globals referenced by this function          |

### `callees[]`, `structs[]`, `enums[]`, `globals[]`

Each object contains:

| Field  | Type   | Description            |
| ------ | ------ | ---------------------- |
| `name` | string | Referenced symbol name |

---

## `structs[]`

Each entry represents a struct definition.

| Field       | Type    | Description                       |
| ----------- | ------- | --------------------------------- |
| `name`      | string  | Struct name                       |
| `filename`  | string  | Source file containing the struct |
| `startLine` | integer | Starting line number              |
| `endLine`   | integer | Ending line number                |
| `startCol`  | integer | Starting column                   |
| `endCol`    | integer | Ending column                     |

---

## `enums[]`

Each entry represents an enum definition.

| Field       | Type    | Description                     |
| ----------- | ------- | ------------------------------- |
| `name`      | string  | Enum name                       |
| `filename`  | string  | Source file containing the enum |
| `startLine` | integer | Starting line number            |
| `endLine`   | integer | Ending line number              |
| `startCol`  | integer | Starting column                 |
| `endCol`    | integer | Ending column                   |

---

## `globals[]`

Each entry represents a global variable.

| Field       | Type    | Description                               |
| ----------- | ------- | ----------------------------------------- |
| `name`      | string  | Global variable name                      |
| `type`      | string  | Variable type                             |
| `filename`  | string  | Source file containing the variable       |
| `startLine` | integer | Starting line number                      |
| `endLine`   | integer | Ending line number                        |
| `startCol`  | integer | Starting column                           |
| `endCol`    | integer | Ending column                             |
| `isStatic`  | boolean | Whether the variable is declared `static` |

---

## Notes

- Line and column numbers are 1‑indexed.
- All the `filename` fields are based on the source file names as passed to ParseC. So if you were to run:
  
```sh
parsec src/*.c
```

Then the filenames would be `src/file1.c`, etc. However, if you were to run:

```sh
parsec /home/user/proj/src/*.c
```

Then the filenames would be `/home/user/proj/src/file1.c`, etc.

- Arrays may be empty if no corresponding entities are found.
- The output format may evolve; consumers should ignore unknown fields for forward compatibility.
