# Outputs

This page summarizes the different output types available for Nextflow processes. Use these declarations to specify how data is emitted from your process scripts.

### val

Declare a variable output. The argument can be any value and can reference output variables defined in the process body.

- **Syntax:** `val(value)`
- **Output:** Any value.
- **Usage:**
    ```groovy
    process example {
        output:
        val(result)
        script:
        """
        result=42
        echo \$result
        """
    }
    ```

**See also:** [Inputs.md `val`](Inputs.md#val), the input-side counterpart for receiving a value into the process body.

### path

Declare a file output. Receives output files from the task environment matching the given pattern.

- **Syntax:** `path(pattern, [options])`
- **Output:** File(s) matching the pattern. Multiple patterns can be combined with `:` (e.g., `'*.bam:*.bai'`); the union of matches is collected.
- **Options:**
    - `arity`: Number or range of expected files (e.g., `1`, `1..*`). Task fails if invalid. If `1`, a single file is emitted; otherwise a list is always emitted, even for one file. Without `arity`, Nextflow emits a single file or a list depending on how many files are produced at runtime — this can lead to an output channel with a mix of files and lists, so set `arity` explicitly when the count matters.
    - `followLinks`: Return target files for symlinks (default: true).
    - `glob`: Interpret name as glob pattern (default: true).
    - `hidden`: Include hidden files (default: false).
    - `includeInputs`: Include input files matching the pattern (default: false).
    - `maxDepth`: Maximum directory levels to visit.
    - `type`: Type of paths returned: `file`, `dir`, or `any`.
- **Usage:**
    ```groovy
    process example {
        output:
        path('*.out', arity: 1)
        script:
        """
        echo foo > result.out
        """
    }
    ```

**See also:** [Inputs.md `path`](Inputs.md#path), the input-side counterpart — shares the same `arity` option. [Directives.md `publishDir`](Directives.md#publishdir)/[`storeDir`](Directives.md#storedir) decide where these files get copied once emitted. [Operators.md `collectFile`](Operators.md#collectfile) is another way to write channel contents to disk, from the channel side rather than the process side.

### env

Declare an environment variable output. Receives the value of the environment variable from the task environment.

- **Syntax:** `env(name)`
- **Output:** String.
- **Usage:**
    ```groovy
    process example {
        output:
        env(MY_VAR)
        script:
        """
        export MY_VAR=hello
        """
    }
    ```

**See also:** [Inputs.md `env`](Inputs.md#env), the input-side counterpart for exporting a variable into the task environment.

### stdout

Declare a stdout output. Receives the standard output of the task script.

- **Syntax:** `stdout`
- **Output:** String (stdout).
- **Usage:**
    ```groovy
    process example {
        output:
        stdout
        script:
        """
        echo Hello world
        """
    }
    ```

**See also:** [Inputs.md `stdin`](Inputs.md#stdin), the input-side counterpart for piping a string into the task as standard input. `eval` below is similar but captures a separate command's output rather than the script's own stdout.

### eval

Declare an eval output. Receives the standard output of the given command, executed in the task environment after the task script. If the command fails, the task also fails.

- **Syntax:** `eval(command)`
- **Output:** String (stdout of command).
- **Usage:**
    ```groovy
    process example {
        output:
        eval('cat result.txt')
        script:
        """
        echo foo > result.txt
        """
    }
    ```

### tuple

Declare a tuple output. Each argument should be an output declaration (`val`, `path`, `env`, `stdout`, or `eval`). Each tuple element is treated as a standalone output.

- **Syntax:** `tuple(arg1, arg2, ...)`
- **Output:** Tuple with elements matching the declarations.
- **Usage:**
    ```groovy
    process example {
        output:
        tuple(val(x), path(y))
        script:
        """
        echo foo > $y
        """
    }
    ```

**See also:** [Inputs.md `tuple`](Inputs.md#tuple), the input-side counterpart. [Operators.md `groupTuple`/`join`/`combine`/`cross`](Operators.md#combining-operators) are the main operators for working with tuple-shaped channels produced here.
