# Process Directives

This page summarizes all process directives available in Nextflow. Directives control process execution, resource allocation, environment, and output handling.

---

## Resource Management

Directives that control resource allocation for process execution.

### cpus

Set number of CPUs.

```groovy
process big_job {
    cpus 8
    script:
    """
    blastp -query input_sequence -num_threads ${task.cpus}
    """
}
```

### memory

Set memory requirement.

```groovy
process big_job {
    memory '2 GB'
    // ...
}
```

### time

Set maximum process runtime.

```groovy
process big_job {
    time '1h'
    // ...
}
```

### disk

Set local disk storage.

```groovy
process big_job {
    disk '2 GB'
    script:
    """
    your_command
    """
}
```

### accelerator

Request hardware accelerators (e.g. GPUs) for a process.

```groovy
process foo {
    accelerator 4, type: 'nvidia-tesla-k80'
    script:
    """
    your_gpu_enabled --command --line
    """
}
```
> Only supported by certain executors. See platform docs for details.

### machineType

Specify a cloud machine type.

```groovy
process foo {
    machineType 'n1-highmem-8'
    // ...
}
```

### resourceLimits

Set environment-specific resource limits.

```groovy
process my_task {
    resourceLimits cpus: 24, memory: 768.GB, time: 72.h
    // ...
}
```

### arch

Specify CPU architecture and microarchitecture.

```groovy
process cpu_task {
    arch 'linux/x86_64', target: 'cascadelake'
    script:
    """
    blastp -query input_sequence -num_threads ${task.cpus}
    """
}
```

---

## Execution Control

Directives that affect how and when processes are executed.

### executor

Override the executor for a process.

```groovy
process doSomething {
    executor 'sge'
    // ...
}
```

### queue

Set the job queue for grid executors.

```groovy
process grid_job {
    queue 'long'
    // ...
}
```

### clusterOptions

Pass native cluster options.

```groovy
process foo {
    clusterOptions '-x 1 -y 2'
    // or
    clusterOptions '-x 1', '-y 2', '--flag'
}
```
> Only for grid executors.

### penv

Set parallel environment for SGE.

```groovy
process big_job {
    cpus 4
    penv 'smp'
    // ...
}
```

### array

Submit tasks as job arrays (experimental).

```groovy
process cpu_task {
    executor 'slurm'
    array 100
    script:
    """
    your_command --here
    """
}
```
> Supported by AWS Batch, Google Cloud Batch, LSF, PBS, SGE, SLURM.

### maxForks

Limit parallel process instances.

```groovy
process doNotParallelizeIt {
    maxForks 1
    // ...
}
```

### maxSubmitAwait

Set max time in submission queue.

```groovy
process foo {
    maxSubmitAwait '10 mins'
    // ...
}
```

---

## Error Handling & Retry

Directives for error management and retry strategies.

### errorStrategy

Control error handling.

```groovy
process ignoreAnyError {
    errorStrategy 'ignore'
    // ...
}
```
- `terminate` (default)
- `finish`
- `ignore`
- `retry`

### maxRetries

Max retries for a failed task.

```groovy
process retryIfFail {
    errorStrategy 'retry'
    maxRetries 3
    // ...
}
```

### maxErrors

Max total errors allowed for a process.

```groovy
process retryIfFail {
    errorStrategy 'retry'
    maxErrors 5
    // ...
}
```

---

## Environment & Dependencies

Directives for specifying the process environment and dependencies.

### conda

Define Conda dependencies.

- **Specify packages:** List one or more packages (with optional versions) separated by spaces.
- **Specify channels:** Use the `channel::package=version` syntax to select a channel (e.g., `bioconda::bwa=0.7.15`).
- **Use environment files:** Provide a path to a Conda environment YAML file.
- **Use existing environments:** Provide a path to an existing Conda environment directory.

**Examples:**

```groovy
process foo {
    // Single package
    conda 'bwa=0.7.15'
    script:
    """
    bwa mem ...
    """
}

process bar {
    // Multiple packages
    conda 'bwa=0.7.15 fastqc=0.11.5'
    script:
    """
    bwa mem ... && fastqc ...
    """
}

process baz {
    // Specify channel
    conda 'bioconda::bwa=0.7.15'
    script:
    """
    bwa mem ...
    """
}

process envFileExample {
    // Use environment YAML file
    conda './envs/myenv.yaml'
    script:
    """
    my_tool ...
    """
}

process existingEnvExample {
    // Use existing environment directory
    conda '/path/to/conda/env'
    script:
    """
    my_tool ...
    """
}
```

> See [Conda environments](https://www.nextflow.io/docs/latest/conda.html) for more details.

### spack

Define Spack dependencies.

```groovy
process foo {
    spack 'bwa@0.7.15'
    // ...
}
```

### module

Load environment modules.

```groovy
process basicExample {
    module 'ncbi-blast/2.2.27'
    // ...
}
```

### container

Run the process in a Docker container.

```groovy
process runThisInDocker {
    container 'dockerbox:tag'
    script:
    """
    your_command
    """
}
```

### containerOptions

Extra options for the container engine.

```groovy
process runThisWithDocker {
    container 'busybox:latest'
    containerOptions '--volume /data/db:/db'
    script:
    """
    your_command --data /db
    """
}
```
> Not supported by Kubernetes executor.

### pod

Kubernetes pod-specific settings.

```groovy
process your_task {
    pod env: 'FOO', value: 'bar'
    // ...
}
```

### shell

Set a custom shell for the script.

```groovy
process doMoreThings {
    shell '/bin/bash', '-euo', 'pipefail'
    // ...
}
```

### secret

Expose secrets as environment variables.

```groovy
process someJob {
    secret 'MY_ACCESS_KEY'
    secret 'MY_SECRET_KEY'
    script:
    """
    your_command --access \$MY_ACCESS_KEY --secret \$MY_SECRET_KEY
    """
}
```

---

## Input/Output Staging

Directives for handling input and output files.

### stageInMode

Control how input files are staged.

- `'copy'`
- `'link'`
- `'rellink'`
- `'symlink'` (default)

### stageOutMode

Control how output files are staged out.

- `'copy'`
- `'fcp'`
- `'move'`
- `'rclone'`
- `'rsync'`

### scratch

Run process in a temporary local directory.

```groovy
process simpleTask {
    scratch true
    output:
    path 'data_out'
    script:
    """
    your_command
    """
}
```

---

## Output Publishing & Caching

Directives for publishing and caching process outputs.

### publishDir

Publish output files to a directory.

```groovy
process foo {
    publishDir '/data/chunks', mode: 'copy', overwrite: false
    output:
    path 'chunk_*'
    script:
    """
    printf 'Hola' | split -b 1 - chunk_
    """
}
```

### storeDir

Permanent cache for process results.

```groovy
process formatBlastDatabases {
    storeDir '/db/genomes'
    // ...
}
```

### cache

Control process result caching.

```groovy
process noCacheThis {
    cache false
    // ...
}
```
- `false`: Disable caching
- `true`: Enable (default)
- `'deep'`: Use file content
- `'lenient'`: Minimal metadata

---

## Metadata & Customization

Directives for process labeling, tagging, and custom metadata.

### label

Annotate processes for grouping/configuration.

- **Purpose:** Use `label` to assign one or more static labels to a process. These labels are mainly used for configuration and resource selection in your `nextflow.config` file (e.g., to apply settings to all processes with a given label).
- **Scope:** Labels are static and set at the process definition level.

```groovy
process bigTask {
    label 'big_mem'
    // ...
}
```

**Example: Use label for configuration**
```groovy
// In nextflow.config
process {
    withLabel: big_mem {
        cpus = 16
        memory = '64 GB'
    }
}
```

### tag

Custom label for each process execution.

- **Purpose:** Use `tag` to dynamically assign a string to each process execution (task). This is useful for tracking, logging, and making trace files more informative.
- **Scope:** Tags can use process properties and are evaluated per task.

```groovy
process foo {
    tag "$foo"
    input:
    val foo
    script:
    """
    echo $foo
    """
}
```

**How tag influences job names (e.g. SLURM):**

When using cluster executors like SLURM, the `tag` value is often included in the job name submitted to the scheduler. By default, Nextflow sets the job name to include the process name and, if a tag is specified, the tag value. This makes it easier to identify jobs in the cluster queue.

**Example:**
If you set:
```groovy
process foo {
    tag { "SAMPLE_${sample_id}" }
    input:
    val sample_id
    script:
    """
    echo "Processing $sample_id"
    """
}
```
The SLURM job name will include `foo (SAMPLE_sample1)` for a task with `sample_id = 'sample1'`.

You can further customize the job name using the `executor.jobName` config option:
```groovy
executor {
    jobName = { "${task.process} ${task.tag}" }
}
```

**Example: Use tag with process properties**
```groovy
process foo {
    tag { "sample:${task.id} (${task.process})" }
    input:
    val sample_id
    script:
    """
    echo "Processing sample $sample_id"
    """
}
```
- Here, `task.id` and `task.process` are process properties available for use in the tag.

**Common process properties for tag:**

- `task.id`: Unique pipeline task index
- `task.index`: Process-level task index
- `task.process`: Process name
- `task.attempt`: Current retry attempt
- `task.cpus`, `task.memory`: Allocated resources

**Summary Table**

| Directive | Purpose | Scope | Example Usage |
|-----------|---------|-------|--------------|
| `label`   | Static grouping/configuration | Process definition | Resource selection in config |
| `tag`     | Dynamic per-task annotation   | Each task         | Logging, trace, reporting   |

---

## Script Hooks

Directives for running code before or after the main script.

### beforeScript

Run a Bash snippet before the main process script.

```groovy
process foo {
    beforeScript 'source /cluster/bin/setup'
    script:
    """
    echo bar
    """
}
```

### afterScript

Run a Bash snippet immediately after the main process script.

```groovy
process foo {
    afterScript 'rm -rf temp_dir'
    script:
    """
    your_command
    """
}
```
> Runs outside the container if `container` is used.

---

## Miscellaneous

### debug

Show process stdout in the terminal.

```groovy
process sayHello {
    debug true
    script:
    """
    echo Hello
    """
}
```

