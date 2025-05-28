# Outputs

This document summarizes the different output types available for Nextflow processes. Use these declarations to specify how data is emitted from your process scripts.

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

### path

Declare a file output. Receives output files from the task environment matching the given pattern.

- **Syntax:** `path(pattern, [options])`
- **Output:** File(s) matching the pattern.
- **Options:**
    - `arity`: Number or range of expected files (e.g., `1`, `1..*`). Task fails if invalid.
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

### eval

Declare an eval output. Receives the standard output of the given command, executed in the task environment after the task script.

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
