# Inputs

This page summarizes the different input types available for Nextflow processes. Use these declarations to specify how data is provided to your process scripts.

### val

Declare a variable input. The received value can be any type and is made available to the process body as a variable with the given identifier.

- **Syntax:** `val(identifier)`
- **Input:** Any value.
- **Usage:**
    ```groovy
    process example {
        input:
        val(x)
        script:
        """
        echo $x
        """
    }
    ```

**See also:** [Outputs.md `val`](Outputs.md#val), the output-side counterpart for emitting a value from the process body.

### path

Declare a file input. The value should be a file or collection of files and is staged into the task directory.

- **Syntax:** `path(identifier | stageName)`
- **Input:** File or collection of files.
- **Options:**
    - `arity`: Number or range of expected files (e.g., `1`, `1..*`). Task fails if invalid.
    - `name` / `stageAs`: Name or pattern for the file(s) in the task directory.
- **Usage:**
    ```groovy
    process example {
        input:
        path('input.txt', stageAs: 'renamed.txt')
        script:
        """
        cat renamed.txt
        """
    }
    ```

**See also:** [Outputs.md `path`](Outputs.md#path), the reverse declaration for emitting files instead of receiving them — shares the same `arity` option. [Directives.md `stageInMode`](Directives.md#stageinmode) controls how these files are staged into the task directory. [Operators.md `splitCsv`/`splitFasta`/`splitFastq`/`splitText`/`splitJson`](Operators.md#splitting-and-parsing-operators) are the usual way to turn a `path` input's contents into a per-record channel.

### env

Declare an environment variable input. The value should be a string and is exported to the task environment as the given variable name.

- **Syntax:** `env(name)`
- **Input:** String.
- **Usage:**
    ```groovy
    process example {
        input:
        env(MY_VAR)
        script:
        """
        echo \$MY_VAR
        """
    }
    ```

**See also:** [Outputs.md `env`](Outputs.md#env), the output-side counterpart for capturing an environment variable from the task.

### stdin

Declare a stdin input. The value should be a string and is provided as standard input to the task script. Only one stdin input can be declared per process.

- **Syntax:** `stdin`
- **Input:** String.
- **Usage:**
    ```groovy
    process example {
        input:
        stdin
        script:
        """
        cat
        """
    }
    ```

**See also:** [Outputs.md `stdout`](Outputs.md#stdout), the output-side counterpart for capturing a task's standard output.

### tuple

Declare a tuple input. Each argument should be an input declaration (`val`, `path`, `env`, or `stdin`). The received value should be a tuple matching the declaration.

- **Syntax:** `tuple(arg1, arg2, ...)`
- **Input:** Tuple with elements matching the declarations.
- **Usage:**
    ```groovy
    process example {
        input:
        tuple(val(x), path(y))
        script:
        """
        echo $x
        cat $y
        """
    }
    ```

**See also:** [Outputs.md `tuple`](Outputs.md#tuple), the output-side counterpart. [Operators.md `groupTuple`/`join`/`combine`/`cross`](Operators.md#combining-operators) are the main operators for building and manipulating tuple-shaped channels feeding into this declaration.
