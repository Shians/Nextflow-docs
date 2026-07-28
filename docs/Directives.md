# Process Directives

This page summarizes all process directives available in Nextflow. Directives control process execution, resource allocation, environment, and output handling.

---

## Resource Management

Directives that control resource allocation for process execution.

### cpus

Use `cpus` to specify the number of CPU cores required for the process. The allocated value is accessible via `${task.cpus}` in the script, allowing you to pass it to multi-threaded tools.

**Supported values:** Positive integers (e.g., `1`, `4`, `16`)

```groovy
process alignment {
    cpus 8

    input:
    path genome
    path reads

    output:
    path 'aligned.bam'

    script:
    """
    # Access allocated CPUs via task.cpus
    bwa mem -t ${task.cpus} ${genome} ${reads} | \\
        samtools sort -@ ${task.cpus} -o aligned.bam
    """
}
```

**See also:** `resourceLimits` for setting maximum CPU constraints across environments.

### memory

Use `memory` to specify the amount of RAM required for the process. This ensures proper resource allocation on cluster systems and prevents out-of-memory errors.

**Supported units:** `B`, `KB`, `MB`, `GB`, `TB` (case-insensitive)

```groovy
process assembly {
    memory '32 GB'

    input:
    path reads

    output:
    path 'contigs.fa'

    script:
    """
    # Large memory operations
    spades.py -o assembly -1 ${reads[0]} -2 ${reads[1]} -m ${task.memory.toMega()}
    """
}
```

**Dynamic memory allocation on retry:**
```groovy
process retry_with_more_memory {
    errorStrategy 'retry'
    maxRetries 3
    memory { 4.GB * task.attempt }  // Start with 4GB, double on each retry

    input:
    path data

    output:
    path 'result.txt'

    script:
    """
    echo "Attempt ${task.attempt} with ${task.memory}"
    memory_intensive_analysis.sh ${data} > result.txt
    """
}
```

**See also:** `resourceLimits` for setting maximum memory constraints.

### time

Use `time` to set the maximum execution time for the process. If the process exceeds this limit, it will be terminated. This is essential for cluster schedulers that enforce time limits.

**Supported units:** `s` (seconds), `m` (minutes), `h` (hours), `d` (days)

```groovy
process long_simulation {
    time '6h'

    input:
    val simulation_id

    output:
    path 'simulation_*.out'

    script:
    """
    run_simulation.sh ${simulation_id}
    """
}
```

**Dynamic time allocation:**
```groovy
process adaptive_runtime {
    errorStrategy 'retry'
    maxRetries 2
    time { 2.h * task.attempt }  // Start with 2h, increase on retry

    script:
    """
    long_running_task.sh
    """
}
```

**See also:** `maxRetries` for handling time limit exceeded errors.

### disk

Use `disk` to specify the amount of local disk storage required for temporary files during process execution. Particularly important for cloud executors and systems with limited local storage.

**Supported units:** `B`, `KB`, `MB`, `GB`, `TB`

```groovy
process sort_large_file {
    disk '100 GB'

    input:
    path unsorted_data

    output:
    path 'sorted_data.txt'

    script:
    """
    # Sorting requires substantial temporary disk space
    sort -T . ${unsorted_data} > sorted_data.txt
    """
}
```

**See also:** `scratch` for using temporary local directories.

### accelerator

Use `accelerator` to request hardware accelerators such as GPUs for compute-intensive processes like machine learning, molecular dynamics, or image processing.

**Arguments:**
- First argument: Number of accelerators (required)
- `type`: Accelerator type/model (optional, executor-dependent)
- `request`: Number to request vs. number to use (optional)

```groovy
process gpu_training {
    accelerator 1, type: 'nvidia-tesla-v100'

    input:
    path training_data

    output:
    path 'model.h5'

    script:
    """
    # GPU-accelerated training
    python train_model.py \\
        --data ${training_data} \\
        --gpu ${task.accelerator.device} \\
        --output model.h5
    """
}
```

**Multiple GPUs:**
```groovy
process multi_gpu {
    accelerator 4, type: 'nvidia-tesla-k80'

    script:
    """
    # Utilize all 4 GPUs
    mpirun -np 4 gpu_simulation --gpus 4
    """
}
```

> **Note:** Only supported by certain executors (AWS Batch, Google Cloud Batch, Kubernetes). Check your executor documentation for available accelerator types.

**See also:** `machineType` for cloud-specific instance selection.

### machineType

Use `machineType` to explicitly specify the virtual machine type when running on cloud platforms (Google Cloud, AWS). This allows you to select instances optimized for specific workloads (compute, memory, GPU).

```groovy
process high_memory_analysis {
    machineType 'n1-highmem-8'  // Google Cloud: 8 vCPUs, 52 GB RAM

    input:
    path large_dataset

    output:
    path 'analysis_results.txt'

    script:
    """
    analyze_large_data.sh ${large_dataset} > analysis_results.txt
    """
}
```

**AWS example:**
```groovy
process compute_optimized {
    machineType 'c5.4xlarge'  // AWS: 16 vCPUs, 32 GB RAM

    script:
    """
    compute_intensive_task.sh
    """
}
```

> **Note:** Machine type names are cloud provider-specific. This directive is only used by cloud executors, and when set, it overrides the `cpus` and `memory` directives for that process.

**See also:** `cpus` and `memory` for executor-agnostic resource requests.

### resourceLimits

Use `resourceLimits` to set maximum resource constraints that apply across different execution environments. This prevents processes from requesting more resources than available in your infrastructure.

**Arguments:**
- `cpus`: Maximum CPU cores
- `memory`: Maximum memory
- `disk`: Maximum disk space
- `time`: Maximum runtime

```groovy
process constrained_task {
    cpus 32
    memory '128 GB'
    time '24h'

    // Ensure limits are respected across all environments
    resourceLimits cpus: 24, memory: 64.GB, time: 12.h

    script:
    """
    # Will use min(requested, limit) = 24 CPUs, 64 GB, 12h
    analysis_pipeline.sh
    """
}
```

**Common use case - setting organization limits:**
```groovy
process org_compliant {
    resourceLimits cpus: 48, memory: 768.GB, time: 72.h

    cpus 64      // Will be capped at 48
    memory '1 TB'  // Will be capped at 768 GB

    script:
    """
    your_command
    """
}
```

**See also:** `cpus`, `memory`, `time` for setting resource requests.

### arch

Use `arch` to specify the CPU architecture and optionally the microarchitecture for process execution. This is used by the `spack` directive to build microarchitecture-optimized software, and by the Wave container service to build containers for the given architecture — it does not perform general node/hardware selection (use `machineType` for that).

**Arguments:**
- First argument: Architecture string (e.g., `'linux/x86_64'`, `'linux/arm64'`)
- `target`: Specific microarchitecture for optimizations (optional)

```groovy
process x86_optimized {
    arch 'linux/x86_64', target: 'cascadelake'

    input:
    path sequences

    output:
    path 'aligned.sam'

    script:
    """
    # Run on Intel Cascade Lake or newer for AVX-512 instructions
    bwa mem -t ${task.cpus} ref.fa ${sequences} > aligned.sam
    """
}
```

**ARM architecture example:**
```groovy
process arm_process {
    arch 'linux/arm64'

    script:
    """
    # Run on ARM-based instances (e.g., AWS Graviton)
    compute_task.sh
    """
}
```

> **Note:** Microarchitecture targets depend on your execution environment and available hardware.

**See also:** `machineType` for cloud-specific instance selection.

---

## Execution Control

Directives that affect how and when processes are executed.

### executor

Use `executor` to override the default executor for a specific process. This allows you to run different processes on different compute platforms within the same pipeline.

**Common executors:** `local`, `sge`, `slurm`, `pbs`, `lsf`, `awsbatch`, `azurebatch`, `google-batch`, `k8s`

```groovy
process local_preprocessing {
    executor 'local'

    input:
    path raw_data

    output:
    path 'preprocessed.txt'

    script:
    """
    # Run locally for quick preprocessing
    preprocess.sh ${raw_data} > preprocessed.txt
    """
}

process cluster_analysis {
    executor 'slurm'
    cpus 32
    memory '128 GB'

    input:
    path preprocessed

    output:
    path 'results.txt'

    script:
    """
    # Submit to SLURM cluster for heavy computation
    analyze.sh ${preprocessed} > results.txt
    """
}
```

**See also:** Configure default executor in `nextflow.config` rather than per-process for consistency. See [PipelineConfiguration.md "Slurm Profile Example"](PipelineConfiguration.md#slurm-profile-example) for a full cluster-executor config.

### queue

Use `queue` to specify which job queue or partition to submit the process to when using grid executors (SGE, SLURM, PBS, LSF). Queues often have different resource limits and priorities.

```groovy
process quick_task {
    executor 'slurm'
    queue 'short'
    time '1h'

    script:
    """
    fast_analysis.sh
    """
}

process long_task {
    executor 'slurm'
    queue 'long'
    time '72h'

    script:
    """
    extended_simulation.sh
    """
}
```

**Multiple queues (priority order):**
```groovy
process flexible_submission {
    queue 'high-priority,normal,low-priority'

    script:
    """
    # Nextflow will try queues in order until one accepts the job
    your_command
    """
}
```

**See also:** `clusterOptions` for additional queue-specific settings.

### clusterOptions

Use `clusterOptions` to pass native scheduler options directly to the underlying grid executor. This allows you to use executor-specific features not directly supported by Nextflow directives.

```groovy
process slurm_specific {
    executor 'slurm'
    clusterOptions '--account=myproject --qos=high'

    script:
    """
    research_computation.sh
    """
}
```

**Multiple options (alternative syntaxes):**
```groovy
process sge_options {
    executor 'sge'

    // Single string
    clusterOptions '-P myproject -l h_rt=24:00:00'

    // Or multiple arguments
    // clusterOptions '-P', 'myproject', '-l', 'h_rt=24:00:00'

    script:
    """
    your_command
    """
}
```

**Dynamic options using task properties:**
```groovy
process dynamic_options {
    executor 'slurm'
    clusterOptions { "--account=${params.account} --constraint=${task.cpus > 16 ? 'highmem' : 'normal'}" }

    script:
    """
    analysis.sh
    """
}
```

> **Note:** Only applicable to grid executors (SGE, SLURM, PBS, LSF). Options are executor-specific.

**See also:** `queue` for selecting job queues.

### penv

Use `penv` to set the parallel environment for SGE (Sun Grid Engine) when requesting multiple CPUs. The parallel environment must be configured in your SGE installation.

```groovy
process sge_parallel {
    executor 'sge'
    cpus 8
    penv 'smp'  // Shared memory parallel environment

    script:
    """
    # SGE will allocate 8 cores in the 'smp' parallel environment
    parallel_app -threads ${task.cpus}
    """
}
```

**MPI parallel environment:**
```groovy
process mpi_job {
    executor 'sge'
    cpus 16
    penv 'mpi'

    script:
    """
    mpirun -np ${task.cpus} mpi_application
    """
}
```

> **Note:** Only applies to SGE executor. Common parallel environments: `smp` (shared memory), `mpi` (MPI), `orte` (OpenMPI).

**See also:** `cpus` directive and your SGE administrator for available parallel environments.

### array

Use `array` to submit multiple tasks as a job array, which can improve scheduler efficiency and reduce submission overhead when running many similar tasks.

**Supported executors:** AWS Batch, Google Cloud Batch, LSF, PBS, SGE, SLURM

```groovy
process batch_analysis {
    executor 'slurm'
    array 100  // Submit in batches of 100 tasks

    input:
    val sample_id

    output:
    path "${sample_id}_result.txt"

    script:
    """
    # Each sample runs as part of a job array
    analyze_sample.sh ${sample_id} > ${sample_id}_result.txt
    """
}

workflow {
    Channel.of(1..1000) | batch_analysis
}
```

**Dynamic array size:**
```groovy
process adaptive_array {
    executor 'slurm'
    array { task.executor == 'slurm' ? 50 : 1 }

    script:
    """
    compute_task.sh
    """
}
```

> **Experimental feature:** Behavior may vary between executors. Job arrays can significantly reduce scheduler overhead for pipelines with many small tasks.
>
> **Gotcha:** Because a job array is submitted as a single job, directives like `cpus`, `memory`, `disk`, `queue`, `machineType`, `resourceLabels`, `resourceLimits`, `time`, `clusterOptions`, and `accelerator` must be uniform across every task in the array — dynamic per-attempt values (e.g. `memory { 4.GB * task.attempt }`) won't vary by task within the same array.

**See also:** Your cluster documentation for job array limits and best practices.

### maxForks

Use `maxForks` to limit the number of process instances that can run in parallel. This is useful for rate-limiting, controlling resource usage, or when accessing shared resources with limited capacity. By default, it is equal to the number of available CPU cores minus 1.

```groovy
process database_query {
    maxForks 5  // Only 5 concurrent database connections

    input:
    val query_id

    output:
    path 'query_${query_id}.csv'

    script:
    """
    # Limit concurrent database load
    psql -c "SELECT * FROM data WHERE id=${query_id}" > query_${query_id}.csv
    """
}
```

**Serial execution:**
```groovy
process write_to_shared_file {
    maxForks 1  // Only one instance at a time

    input:
    val data

    script:
    """
    # Prevent race conditions when writing to shared resource
    echo "${data}" >> shared_output.txt
    """
}
```

**Rate limiting API calls:**
```groovy
process api_call {
    maxForks 3  // Respect API rate limits

    input:
    val record_id

    output:
    path 'record_${record_id}.json'

    script:
    """
    curl "https://api.example.com/records/${record_id}" > record_${record_id}.json
    """
}
```

**See also:** Consider using process output channels and sequential operators instead of `maxForks 1` for better pipeline design.

### maxSubmitAwait

Use `maxSubmitAwait` to set the maximum time a task can wait in the submission queue before being considered failed. This helps detect scheduler issues and prevents tasks from waiting indefinitely.

**Supported units:** `s`, `m`, `h`, `d`

```groovy
process time_sensitive {
    maxSubmitAwait '10m'

    script:
    """
    # If not started within 10 minutes, fail the task
    time_critical_analysis.sh
    """
}
```

**With retry strategy:**
```groovy
process retry_on_queue_timeout {
    maxSubmitAwait '15m'
    errorStrategy 'retry'
    maxRetries 2

    script:
    """
    # Retry if stuck in queue too long
    analysis.sh
    """
}
```

> **Note:** This directive is most useful with busy cluster schedulers where queue wait times can be unpredictable.

**See also:** `time` for process execution time limits, `errorStrategy` for handling submission timeouts.

---

## Error Handling & Retry

Directives for error management and retry strategies.

### errorStrategy

Use `errorStrategy` to control how the pipeline responds when a process task fails. This allows you to build robust pipelines that can handle transient failures or continue despite errors.

**Available strategies:**
- `terminate` (default): Stop the entire pipeline immediately when a task fails
- `finish`: Wait for running tasks to complete, then stop the pipeline
- `ignore`: Continue the pipeline, skip failed tasks. The pipeline still exits `0` unless `workflow.failOnIgnore = true` is set in the config, in which case it reports a non-zero exit code
- `retry`: Automatically retry failed tasks up to `maxRetries` times

```groovy
process transient_failures {
    errorStrategy 'retry'
    maxRetries 3

    script:
    """
    # Retry on network errors, out-of-memory, etc.
    download_from_remote.sh
    """
}
```

**Ignore errors for optional processes:**
```groovy
process optional_qc {
    errorStrategy 'ignore'

    input:
    path data

    output:
    path 'qc_report.html', optional: true

    script:
    """
    # Pipeline continues even if QC fails
    generate_qc_report.sh ${data}
    """
}
```

**Finish gracefully:**
```groovy
process cleanup_on_error {
    errorStrategy 'finish'

    script:
    """
    # Allow other tasks to complete before stopping
    critical_analysis.sh
    """
}
```

**Dynamic error strategy:**
```groovy
process smart_retry {
    errorStrategy { task.exitStatus == 137 ? 'retry' : 'terminate' }
    maxRetries 2
    memory { 4.GB * task.attempt }

    script:
    """
    # Retry only on out-of-memory errors (exit code 137)
    # Terminate immediately on other errors
    memory_sensitive_process.sh
    """
}
```

**See also:** `maxRetries` for retry limits, `maxErrors` for total error thresholds. When retrying, [`task.attempt`](ProccessProperties.md#taskattempt) and [`task.previousException`](ProccessProperties.md#taskpreviousexception) are available to adapt behavior on subsequent attempts.

### maxRetries

Use `maxRetries` to set the maximum number of times a task will be retried after failure. Must be used with `errorStrategy 'retry'`. Common pattern: increase resources on each retry attempt.

```groovy
process retry_with_backoff {
    errorStrategy 'retry'
    maxRetries 3

    input:
    path input_file

    output:
    path 'output.txt'

    script:
    """
    echo "Attempt ${task.attempt} of ${task.maxRetries + 1}"
    flaky_process.sh ${input_file} > output.txt
    """
}
```

**Increase resources on retry:**
```groovy
process adaptive_resources {
    errorStrategy { task.exitStatus in [137, 140] ? 'retry' : 'terminate' }
    maxRetries 3
    memory { 4.GB * task.attempt }      // 4GB, 8GB, 12GB, 16GB
    time { 2.h * task.attempt }         // 2h, 4h, 6h, 8h
    cpus { task.attempt > 2 ? 8 : 4 }   // 4, 4, 8, 8

    script:
    """
    echo "Attempt ${task.attempt}: ${task.cpus} CPUs, ${task.memory}, ${task.time}"
    resource_intensive_task.sh
    """
}
```

**Retry with exponential backoff (delay):**
```groovy
process api_with_backoff {
    errorStrategy 'retry'
    maxRetries 5

    script:
    """
    # Wait longer between retries: 2^attempt seconds
    sleep $((2 ** (${task.attempt} - 1)))
    curl https://api.example.com/data
    """
}
```

**Exit codes indicating retry-able failures:**
- `137`: Out of memory (SIGKILL)
- `140`: Exceeded time limit
- `143`: Terminated by signal (SIGTERM)
- Custom application exit codes

**See also:** `errorStrategy` for enabling retries, `maxErrors` for process-wide error limits. See [`task.attempt`](ProccessProperties.md#taskattempt) and [`task.previousTrace`](ProccessProperties.md#taskprevioustrace) for tuning resources per retry, as shown above.

### maxErrors

Use `maxErrors` to set the maximum number of failed task instances allowed for a process before terminating the pipeline. This is useful when processing many samples and you want to tolerate some failures without stopping everything.

```groovy
process sample_analysis {
    errorStrategy 'ignore'
    maxErrors 10  // Stop if more than 10 samples fail

    input:
    val sample_id

    output:
    path "${sample_id}_result.txt"

    script:
    """
    # Analyze sample, allow up to 10 failures total
    analyze_sample.sh ${sample_id} > ${sample_id}_result.txt
    """
}

workflow {
    Channel.of(1..1000) | sample_analysis
    // Pipeline continues until 10 samples fail, then terminates
}
```

**Combine with retry:**
```groovy
process fault_tolerant {
    errorStrategy 'retry'
    maxRetries 2
    maxErrors 5

    input:
    val item

    script:
    """
    # Each item retried up to 2 times
    # Pipeline stops if 5 items fail (after all retries)
    process_item.sh ${item}
    """
}
```

**Unlimited errors (use with caution):**
```groovy
process best_effort {
    errorStrategy 'ignore'
    maxErrors -1  // Never stop, ignore all failures

    input:
    val id

    script:
    """
    # Process as many as possible, never fail pipeline
    optional_task.sh ${id}
    """
}
```

**See also:** `errorStrategy` must be set to `ignore` or `finish` for `maxErrors` to be useful. With `terminate` (default), the first error stops the pipeline regardless of `maxErrors`.

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

**See also:** [PipelineConfiguration.md "Conda Options"](PipelineConfiguration.md#conda-options) for pipeline-wide settings (cache dir, mamba, channels) that control how this directive's environments are built.

### spack

Use `spack` to define software dependencies managed by Spack, a package manager for supercomputers and scientific software. Similar to `conda` but optimized for HPC environments.

```groovy
process alignment {
    spack 'bwa@0.7.15'

    input:
    path genome
    path reads

    output:
    path 'aligned.sam'

    script:
    """
    bwa mem ${genome} ${reads} > aligned.sam
    """
}
```

**Multiple packages:**
```groovy
process multi_tool {
    spack 'bwa@0.7.15 samtools@1.9'

    script:
    """
    bwa mem genome.fa reads.fq | samtools sort -o aligned.bam
    """
}
```

**See also:** `conda` for Conda-based dependency management, `module` for environment modules.

### module

Use `module` to load software via environment modules (commonly used on HPC clusters). Environment modules must be installed and configured on your system.

```groovy
process blast_search {
    module 'ncbi-blast/2.12.0'

    input:
    path query
    path database

    output:
    path 'results.txt'

    script:
    """
    blastp -query ${query} -db ${database} -out results.txt
    """
}
```

**Multiple modules:**
```groovy
process pipeline_step {
    module 'bwa/0.7.17'
    module 'samtools/1.14'

    script:
    """
    bwa mem genome.fa reads.fq | samtools sort -o aligned.bam
    """
}
```

**Module with dependencies:**
```groovy
process r_analysis {
    module 'R/4.1.0:rstudio/2021.09.0'  // Load R and RStudio modules

    script:
    """
    Rscript analysis.R
    """
}
```

**See also:** `conda` and `spack` for package-manager based dependencies, `container` for containerized environments.

### container

Use `container` to run the process inside a container (Docker, Singularity, Podman). This ensures reproducibility by packaging all dependencies with the software environment.

```groovy
process containerized_analysis {
    container 'quay.io/biocontainers/bwa:0.7.17--h5bf99c6_8'

    input:
    path genome
    path reads

    output:
    path 'aligned.sam'

    script:
    """
    bwa mem ${genome} ${reads} > aligned.sam
    """
}
```

**Docker Hub image:**
```groovy
process python_script {
    container 'python:3.9-slim'

    script:
    """
    python -c "print('Hello from container')"
    """
}
```

**Multiple container registries:**
```groovy
process multi_registry {
    container 'docker://ubuntu:20.04'          // Docker Hub
    // container 'library://alpine:latest'     // Sylabs Cloud Library
    // container 'oras://ghcr.io/user/image'   // GitHub Container Registry

    script:
    """
    echo "Running in container"
    """
}
```

**Container per conda environment:**
```groovy
process auto_container {
    conda 'bioconda::salmon=1.9.0'
    // Automatically use container if enabled in config
    // container 'quay.io/biocontainers/salmon:1.9.0--...'

    script:
    """
    salmon --version
    """
}
```

> **Note:** Enable containers in your config with `docker.enabled = true` or `singularity.enabled = true`.

**See also:** `containerOptions` for additional container settings, `conda` for auto-container feature. See [PipelineConfiguration.md "Container Engines"](PipelineConfiguration.md#container-engines-docker-singularity) for enabling Docker/Singularity globally.

### containerOptions

Use `containerOptions` to pass additional options to the container engine (Docker, Singularity, Podman). This allows mounting volumes, setting environment variables, or specifying runtime parameters.

```groovy
process mount_volume {
    container 'ubuntu:20.04'
    containerOptions '--volume /data:/mnt/data:ro'

    script:
    """
    # Access host directory at /mnt/data (read-only)
    ls /mnt/data
    """
}
```

**Multiple options:**
```groovy
process docker_gpu {
    container 'tensorflow/tensorflow:latest-gpu'
    containerOptions '--gpus all --shm-size=16g'

    script:
    """
    python train_model.py
    """
}
```

**Singularity options:**
```groovy
process singularity_bind {
    container 'docker://ubuntu:20.04'
    containerOptions '--bind /scratch:/scratch --writable-tmpfs'

    script:
    """
    # Singularity-specific binding
    echo "Using /scratch"
    """
}
```

**Dynamic options:**
```groovy
process conditional_mount {
    container 'alpine:latest'
    containerOptions { params.mount_data ? "--volume ${params.data_dir}:/data" : '' }

    script:
    """
    ls /data
    """
}
```

> **Note:** Not supported by Kubernetes executor. Options are container engine-specific.

**See also:** `container` for specifying container images.

### pod

Use `pod` to configure Kubernetes-specific pod settings when using the Kubernetes executor. This allows setting environment variables, resource limits, security contexts, and other pod specifications.

```groovy
process k8s_task {
    pod env: 'FOO', value: 'bar'

    script:
    """
    echo "FOO is $FOO"
    """
}
```

**Multiple environment variables:**
```groovy
process k8s_multi_env {
    pod env: 'API_URL', value: 'https://api.example.com'
    pod env: 'TIMEOUT', value: '30'

    script:
    """
    curl \$API_URL
    """
}
```

**Pod annotations:**
```groovy
process annotated_pod {
    pod annotation: 'iam.amazonaws.com/role', value: 'my-role'

    script:
    """
    aws s3 ls
    """
}
```

**Node selector:**
```groovy
process specific_node {
    pod nodeSelector: 'disktype', value: 'ssd'

    script:
    """
    # Run on nodes with SSD storage
    intensive_io_task.sh
    """
}
```

> **Note:** Only applicable when using the Kubernetes executor.

**See also:** Kubernetes executor documentation for available pod options.

### shell

Use `shell` to specify a custom shell interpreter and options for executing the process script. By default, Nextflow uses `/bin/bash -ue`.

```groovy
process custom_shell {
    shell '/bin/bash', '-euo', 'pipefail'

    script:
    """
    # Fail on undefined variables, pipe failures, and errors
    cat file.txt | grep pattern | sort
    """
}
```

**Using sh instead of bash:**
```groovy
process posix_shell {
    shell '/bin/sh'

    script:
    """
    # POSIX-compliant shell script
    echo "Running in sh"
    """
}
```

**Python as shell:**
```groovy
process python_shell {
    shell '/usr/bin/env python3'

    script:
    """
    print("Hello from Python")
    import sys
    print(f"Python version: {sys.version}")
    """
}
```

**Perl script:**
```groovy
process perl_shell {
    shell '/usr/bin/env perl', '-e'

    script:
    """
    print "Hello from Perl\\n";
    """
}
```

**See also:** Use `script:` block for bash/shell scripts, `exec:` block for compiled executables.

### secret

Use `secret` to securely expose secrets (API keys, passwords, tokens) as environment variables in the process. Secrets must be defined in your Nextflow secrets store or cloud provider's secret manager.

```groovy
process api_call {
    secret 'MY_API_KEY'

    script:
    """
    # Secret exposed as environment variable
    curl -H "Authorization: Bearer \$MY_API_KEY" https://api.example.com/data
    """
}
```

**Multiple secrets:**
```groovy
process authenticated_upload {
    secret 'AWS_ACCESS_KEY_ID'
    secret 'AWS_SECRET_ACCESS_KEY'

    input:
    path data_file

    script:
    """
    # AWS credentials from secrets
    aws s3 cp ${data_file} s3://my-bucket/
    """
}
```

**Database credentials:**
```groovy
process db_query {
    secret 'DB_HOST'
    secret 'DB_USER'
    secret 'DB_PASSWORD'

    output:
    path 'query_results.csv'

    script:
    """
    psql -h \$DB_HOST -U \$DB_USER -c "SELECT * FROM data" > query_results.csv
    """
}
```

**Defining secrets:**
```bash
# Store secrets using Nextflow CLI
nextflow secrets set MY_API_KEY
```

> **Note:** Secrets are not logged or displayed in console output. Only supported by local and grid executors (e.g., SLURM, Grid Engine). AWS Batch supports secrets only when deployed via Seqera Platform; other cloud executors are not supported.

**See also:** Nextflow secrets documentation for setting up secret stores.

---

## Input/Output Staging

Directives for handling input and output files.

### stageInMode

Use `stageInMode` to control how input files are transferred into the process working directory. Different modes offer trade-offs between performance, storage usage, and reliability.

**Available modes:**
- `'symlink'` (default): Create symbolic links (fastest, minimal disk usage)
- `'link'`: Create hard links (fast, but requires same filesystem)
- `'copy'`: Copy files (slower, but most reliable and portable)
- `'rellink'`: Create relative symbolic links

```groovy
process copy_inputs {
    stageInMode 'copy'

    input:
    path input_file

    output:
    path 'modified.txt'

    script:
    """
    # Safe to modify input_file as it's a copy
    sed 's/foo/bar/' ${input_file} > modified.txt
    """
}
```

**When to use each mode:**

**`symlink` (default):**
```groovy
process fast_staging {
    stageInMode 'symlink'  // Default, can be omitted

    input:
    path reference_genome

    script:
    """
    # Fast staging, minimal disk usage
    # WARNING: Don't modify input files (original will be affected)
    index_genome.sh ${reference_genome}
    """
}
```

**`copy` for file modification:**
```groovy
process modify_files {
    stageInMode 'copy'

    input:
    path config_template

    output:
    path 'modified_config.txt'

    script:
    """
    # Safe to modify since it's a copy
    sed "s/PLACEHOLDER/${params.value}/" ${config_template} > modified_config.txt
    """
}
```

**`link` for same filesystem:**
```groovy
process efficient_local {
    stageInMode 'link'

    input:
    path large_file

    script:
    """
    # Hard link - saves space, faster than copy
    # Only works if input and work dir are on same filesystem
    process_file.sh ${large_file}
    """
}
```

**`rellink` for portable workflows:**
```groovy
process relative_links {
    stageInMode 'rellink'

    input:
    path data

    script:
    """
    # Relative symlinks - more portable than absolute
    analyze.sh ${data}
    """
}
```

**See also:** `stageOutMode` for output file handling, `scratch` for temporary execution directories. Applies to files declared via [Inputs.md `path`](Inputs.md#path).

### stageOutMode

Use `stageOutMode` to control how output files are transferred from the process working directory to their final destination. Choose based on performance needs and storage constraints.

**Available modes:**
- `'copy'` (default): Copy files to target location
- `'move'`: Move files (faster, but working directory is modified)
- `'rsync'`: Use rsync for transfer (efficient for large files)
- `'rclone'`: Use rclone for cloud storage transfers
- `'fcp'`: Use FCP (Fast Copy) for high-performance transfers

```groovy
process move_outputs {
    stageOutMode 'move'
    publishDir '/results'

    output:
    path 'result_*.txt'

    script:
    """
    generate_results.sh
    """
}
```

**`copy` (default) - safe and reliable:**
```groovy
process standard_output {
    stageOutMode 'copy'  // Default, can be omitted
    publishDir '/results'

    output:
    path 'output.txt'

    script:
    """
    analysis.sh > output.txt
    """
}
```

**`move` - faster, but destructive:**
```groovy
process fast_transfer {
    stageOutMode 'move'
    publishDir '/final/location'

    output:
    path 'large_output.bam'

    script:
    """
    # Move is faster than copy for large files
    # WARNING: work directory will be modified
    alignment_pipeline.sh > large_output.bam
    """
}
```

**`rsync` - efficient for large files:**
```groovy
process large_outputs {
    stageOutMode 'rsync'
    publishDir '/shared/storage'

    output:
    path 'genome_assembly/*'

    script:
    """
    assemble_genome.sh
    """
}
```

**`rclone` - for cloud storage:**
```groovy
process cloud_output {
    stageOutMode 'rclone'
    publishDir 's3://my-bucket/results'

    output:
    path 'analysis_results.tar.gz'

    script:
    """
    # Efficient transfer to cloud storage
    tar -czf analysis_results.tar.gz results/
    """
}
```

**See also:** `publishDir` for specifying output destinations, `stageInMode` for input handling. Applies to files declared via [Outputs.md `path`](Outputs.md#path).

### scratch

Use `scratch` to execute the process in a temporary local directory (often on fast local storage) and then copy results back. This improves I/O performance when working directory is on slow network storage.

```groovy
process fast_io {
    scratch true

    input:
    path data

    output:
    path 'sorted_data.txt'

    script:
    """
    # Runs in /tmp or $TMPDIR (fast local disk)
    # Results copied back to work directory when complete
    sort -T . ${data} > sorted_data.txt
    """
}
```

**Specify custom scratch directory:**
```groovy
process custom_scratch {
    scratch '/fast/local/ssd'

    script:
    """
    # Use specific fast storage location
    intensive_io_operation.sh
    """
}
```

**Conditional scratch based on executor:**
```groovy
process adaptive_scratch {
    scratch { task.executor == 'slurm' ? '/scratch/$USER' : true }

    script:
    """
    # Use cluster scratch on SLURM, default temp otherwise
    compute_task.sh
    """
}
```

**Global scratch in config:**
```groovy
// In nextflow.config
process {
    scratch = true  // Enable for all processes
}

// Or per-executor
executor {
    name = 'slurm'
    scratch = '/scratch/$USER'
}
```

**Common scratch patterns:**
```groovy
process sort_large_file {
    scratch true
    disk '500 GB'  // Ensure enough scratch space

    input:
    path unsorted_file

    output:
    path 'sorted_file.txt'

    script:
    """
    # Sort benefits greatly from local scratch
    # -T . uses current directory (the scratch space)
    sort -T . -S ${task.memory.toMega()}M ${unsorted_file} > sorted_file.txt
    """
}
```

> **Note:** Scratch directories are automatically cleaned up after successful completion. On failure, they may be retained for debugging depending on configuration.

**See also:** `disk` for specifying scratch space requirements, `stageInMode`/`stageOutMode` for file transfer control.

---

## Output Publishing & Caching

Directives for publishing and caching process outputs.

### publishDir

Publish output files to a directory. The `publishDir` directive allows you to automatically copy, move, or link process output files to a specified directory or remote storage (e.g., S3 bucket). This is useful for collecting results in a single location for downstream analysis or sharing.

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

#### Available Options

You can specify options as a map, e.g. `publishDir path: '/some/dir', mode: 'copy', overwrite: true`.

| Option         | Description |
|----------------|-------------|
| `path`         | Directory or remote location where files are published. Shortcut: `publishDir '/some/dir'` is equivalent to `publishDir path: '/some/dir'`. |
| `mode`         | File publishing method:<br> - `'copy'`: Copy files to publish directory.<br> - `'copyNoFollow'`: Copy files without following symlinks.<br> - `'link'`: Create hard links.<br> - `'move'`: Move files (use only for terminal processes).<br> - `'rellink'`: Create relative symlinks.<br> - `'symlink'`: Create absolute symlinks (default). |
| `overwrite`    | Overwrite existing files in the target directory. Default: `true` during normal execution, `false` when resuming. |
| `pattern`      | [Glob](https://docs.oracle.com/javase/tutorial/essential/io/fileOps.html#glob) pattern to select which output files to publish. |
| `saveAs`       | Closure to rename or change the destination path of published files. Return `null` to skip publishing a file. Useful for dynamic naming or selective publishing.<br>Example:<br>```publishDir '/results', saveAs: { filename -> filename.endsWith('.txt') ? "renamed_${filename}" : null }```<br>Rename `.command.log` to include the process name:<br>```publishDir '/results', saveAs: { filename -> filename == '.command.log' ? "${task.process}_${filename}" : filename }``` |
| `enabled`      | Enable or disable publishing for this rule. Default: `true`.<br>Example: `enabled: params.publish_results` |
| `failOnError`  | Abort execution if publishing fails for any file. Default: `true` (since v24.03.0-edge; was `false` before). |
| `contentType`  | *(Experimental, S3 only)*<br>Specify the media (MIME) type of the published file. If set to `true`, inferred from file extension. Default: `false`. |
| `storageClass` | *(Experimental, S3 only)*<br>Specify the storage class for the published file (e.g., `STANDARD`, `GLACIER`). |
| `tags`         | *(Experimental, S3 only)*<br>Associate arbitrary tags with the published file.<br>Example: `tags: [FOO: 'Hello world']` |

#### Examples

**Basic usage:**
```groovy
process foo {
    publishDir '/results'
    output:
    path 'output.txt'
    script:
    """
    echo result > output.txt
    """
}
```

**Publish only `.txt` files and rename them:**
```groovy
process bar {
    publishDir '/results', pattern: '*.txt', saveAs: { name -> "renamed_${name}" }
    output:
    path '*.txt'
    path '*.log'
    script:
    """
    echo foo > foo.txt
    echo bar > bar.log
    """
}
```

**Publish to S3 with custom storage class and tags:**
```groovy
process s3_publish {
    publishDir 's3://my-bucket/results', storageClass: 'STANDARD_IA', tags: [project: 'nf-demo']
    output:
    path 'result.txt'
    script:
    """
    echo data > result.txt
    """
}
```

**Disable publishing conditionally:**
```groovy
process conditional_publish {
    publishDir '/results', enabled: params.publish_results
    output:
    path 'output.txt'
    script:
    """
    echo something > output.txt
    """
}
```

**Fail pipeline if publishing fails:**
```groovy
process strict_publish {
    publishDir '/results', failOnError: true
    output:
    path 'output.txt'
    script:
    """
    echo fail > output.txt
    """
}
```

> For more details, see the [Nextflow documentation on publishDir](https://www.nextflow.io/docs/latest/process.html#publishdir).

**See also:** [Outputs.md `path`](Outputs.md#path) declares which files exist; `publishDir` decides where they get copied. `storeDir` below is the alternative for permanent, cached results rather than one-off publishing.

### storeDir

Use `storeDir` to permanently store process outputs in a specified directory and reuse them across pipeline runs. Unlike `publishDir`, `storeDir` is used for caching expensive computations and enables true result persistence.

**Key differences from `publishDir`:**
- Results stored in `storeDir` are **never re-executed** if they exist
- `publishDir` is for final results; `storeDir` is for permanent caching
- `storeDir` bypasses normal caching logic - checks only if output files exist

```groovy
process buildReferenceDatabase {
    storeDir '/db/references'

    input:
    path genome_fasta

    output:
    path 'genome.index.*'

    script:
    """
    # Build index - expensive operation cached permanently
    # Only runs if genome.index.* files don't exist in /db/references
    bwa index -p genome.index ${genome_fasta}
    """
}
```

**Use cases:**

**Reference database generation:**
```groovy
process formatBlastDatabase {
    storeDir "${params.db_dir}/blast"

    input:
    path protein_sequences

    output:
    path 'proteins.db.*'

    script:
    """
    # Format database once, reuse forever
    makeblastdb -in ${protein_sequences} -dbtype prot -out proteins.db
    """
}
```

**Download and cache external resources:**
```groovy
process downloadGenome {
    storeDir '/cache/genomes'

    output:
    path 'hg38.fa.gz'

    script:
    """
    # Download once, cache permanently
    wget https://ftp.ncbi.nlm.nih.gov/genomes/hg38.fa.gz
    """
}
```

**Preprocessing shared across runs:**
```groovy
process preprocessDataset {
    storeDir "${params.cache_dir}/preprocessed"
    tag "${dataset_id}"

    input:
    val dataset_id
    path raw_data

    output:
    path "${dataset_id}_preprocessed.rds"

    script:
    """
    # Expensive preprocessing - cache by dataset ID
    Rscript preprocess.R ${raw_data} ${dataset_id}_preprocessed.rds
    """
}
```

**Important considerations:**
- Files in `storeDir` are **never deleted** by Nextflow
- No automatic cleanup - manage disk space manually
- Output file names must be deterministic for proper caching
- Changes to input files won't trigger re-execution (use unique output names)

**See also:** `publishDir` for copying final results, `cache` for controlling normal result caching.

### cache

Use `cache` to control how Nextflow determines if a process needs to be re-executed when resuming a pipeline. The cache mode affects what changes trigger re-execution.

**Available modes:**
- `true` (default): Cache based on inputs, parameters, and file metadata
- `false`: Disable caching, always re-execute
- `'deep'`: Include file content in cache key (slower but more accurate)
- `'lenient'`: Only check input file name and size (not timestamp) — a workaround for shared filesystems with inconsistent timestamps

```groovy
process alwaysRun {
    cache false

    script:
    """
    # Runs every time, never cached
    echo "Current timestamp: $(date)"
    """
}
```

**`true` (default) - standard caching:**
```groovy
process standard_cache {
    cache true  // Default, can be omitted

    input:
    path input_file

    output:
    path 'output.txt'

    script:
    """
    # Cached based on:
    # - Task name and script
    # - Input file paths and metadata (size, timestamp)
    # - Parameters and environment
    analyze.sh ${input_file} > output.txt
    """
}
```

**`'deep'` - content-based caching:**
```groovy
process deep_cache {
    cache 'deep'

    input:
    path config_file

    output:
    path 'result.txt'

    script:
    """
    # Cached based on file CONTENT, not just metadata
    # Re-runs if file content changes, even if timestamp unchanged
    # Slower (must hash file contents) but more accurate
    process_config.sh ${config_file} > result.txt
    """
}
```

**`'lenient'` - minimal caching:**
```groovy
process lenient_cache {
    cache 'lenient'

    input:
    val parameter

    output:
    path 'output.txt'

    script:
    """
    # Only checks input file name and size, not timestamp
    # Useful on shared filesystems where timestamps are unreliable
    generate_output.sh ${parameter} > output.txt
    """
}
```

**`false` - disable caching:**
```groovy
process random_seed {
    cache false

    output:
    path 'random_values.txt'

    script:
    """
    # Always re-execute, never use cached results
    # Useful for: random number generation, timestamps, API calls
    python generate_random.py > random_values.txt
    """
}
```

**When to use each mode:**

| Mode | Use Case | Trade-off |
|------|----------|-----------|
| `true` | Standard workflows | Balanced: checks metadata, not content |
| `'deep'` | Critical reproducibility, small config files | Slow: hashes file contents |
| `'lenient'` | Fast iteration, large input files | Fast: may miss some changes |
| `false` | Non-deterministic processes, debugging | Always re-runs |

**Dynamic caching:**
```groovy
process conditional_cache {
    cache { params.use_cache ? 'deep' : false }

    script:
    """
    computation.sh
    """
}
```

**Caching with retry:**
```groovy
process retry_no_cache {
    errorStrategy 'retry'
    maxRetries 3
    cache false  // Don't cache failed attempts

    script:
    """
    # Re-execute on retry, don't use potentially corrupted cache
    flaky_process.sh
    """
}
```

> **Note:** Cache is stored in the `work/` directory. Cleaning work directory removes cached results.

**See also:** `storeDir` for permanent result storage, `-resume` option for using cached results.

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

**See also:** [PipelineConfiguration.md "Process Scope"](PipelineConfiguration.md#process-scope) for the full `withLabel`/`withName` selector syntax used to target labeled processes.

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

### ext

Use `ext` as a namespace for custom, user-defined process directives — most commonly `ext.args` for passing extra command-line arguments into a script without hardcoding them.

```groovy
process mapping {
    container "biocontainers/star:${task.ext.version}"
    ext version: '2.7.10b', args: '--outSAMtype BAM SortedByCoordinate'

    input:
    path genome
    tuple val(sample_id), path(reads)

    script:
    """
    STAR --genomeDir ${genome} --readFilesIn ${reads} ${task.ext.args ?: ''}
    """
}
```

`ext` values can also be set per-process in `nextflow.config` (the common nf-core pattern), which lets you override arguments without touching the pipeline code:
```groovy
process {
    withName: mapping {
        ext.args = '--outSAMtype BAM SortedByCoordinate --twopassMode Basic'
    }
}
```

### resourceLabels

Use `resourceLabels` to attach custom name-value tags to the cloud compute resource used for a task — useful for cost tracking and attribution.

```groovy
process billed_task {
    resourceLabels region: 'us-east-1', team: 'genomics'

    script:
    """
    your_command --here
    """
}
```

> **Note:** Supported by AWS Batch, Azure Batch, Google Cloud Batch, and Kubernetes. Check your cloud provider's tag naming limits before using this directive.

**See also:** `label` for pipeline-level process labels used in config selectors.

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

Use `beforeScript` to execute a shell command before the main process script runs. Common uses include environment setup, loading modules, creating directories, or initializing resources.

```groovy
process cluster_setup {
    beforeScript 'source /cluster/bin/setup'

    script:
    """
    # Cluster environment loaded by beforeScript
    echo "Running analysis"
    blastp -query input.fa -db database
    """
}
```

**Loading environment modules:**
```groovy
process with_modules {
    beforeScript 'module load python/3.9 samtools/1.14'

    script:
    """
    # Modules loaded before this runs
    python analysis.py
    samtools view alignment.bam
    """
}
```

**Creating working directories:**
```groovy
process prepare_workspace {
    beforeScript 'mkdir -p temp output logs'

    script:
    """
    # Directories created by beforeScript
    process_data.sh --temp temp --output output --log logs/run.log
    """
}
```

**Downloading reference data:**
```groovy
process download_reference {
    beforeScript '''
        if [ ! -f reference.fa ]; then
            wget https://example.com/reference.fa
        fi
    '''

    script:
    """
    # Reference file available
    bwa index reference.fa
    """
}
```

**Setting up conda environment manually:**
```groovy
process custom_conda {
    beforeScript 'source activate myenv'

    script:
    """
    python script.py
    """
}
```

**Global beforeScript in config:**
```groovy
// In nextflow.config
process {
    beforeScript = 'source /etc/profile.d/modules.sh'
}

// Or per-label
process {
    withLabel: cluster {
        beforeScript = 'module load java/11'
    }
}
```

> **Note:** `beforeScript` runs in the same shell session as the main script, so environment changes persist.

**See also:** `afterScript` for cleanup operations, `module` directive for loading environment modules.

### afterScript

Use `afterScript` to execute a shell command after the main process script completes. Common uses include cleanup, logging, notifications, or moving temporary files. Runs regardless of whether the main script succeeds or fails.

```groovy
process cleanup_temp {
    afterScript 'rm -rf temp_dir'

    script:
    """
    # Create and use temporary directory
    mkdir temp_dir
    process_data.sh --tmp temp_dir
    # temp_dir automatically cleaned by afterScript
    """
}
```

**Logging execution time:**
```groovy
process timed_process {
    afterScript 'echo "Process completed at $(date)" >> /logs/timeline.txt'

    script:
    """
    long_running_analysis.sh
    """
}
```

**Cleanup scratch space:**
```groovy
process scratch_cleanup {
    scratch '/fast/scratch'
    afterScript 'rm -rf /fast/scratch/${USER}/*'

    script:
    """
    intensive_io_task.sh
    """
}
```

**Send notifications:**
```groovy
process notify_completion {
    afterScript '''
        curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK \
            -d '{"text":"Process completed"}'
    '''

    script:
    """
    critical_analysis.sh
    """
}
```

**Archive logs:**
```groovy
process archive_logs {
    afterScript 'tar -czf logs.tar.gz *.log && mv logs.tar.gz /archive/'

    script:
    """
    generate_output.sh > process.log 2>&1
    """
}
```

**Conditional cleanup:**
```groovy
process conditional_cleanup {
    afterScript '''
        if [ -d temp_* ]; then
            rm -rf temp_*
        fi
    '''

    script:
    """
    analysis.sh
    """
}
```

**Important notes:**
- `afterScript` runs **outside the container** if using containerized processes
- Executes regardless of main script success/failure
- Cannot access variables or outputs from the main script
- Exit status of `afterScript` does not affect task success/failure

**Global afterScript in config:**
```groovy
// In nextflow.config
process {
    afterScript = 'echo "Completed: ${task.name}" >> /logs/completed.txt'
}
```

> **Warning:** When using containers, `afterScript` runs on the host system, not inside the container. Use it for host-level cleanup, not for operations requiring container software.

**See also:** `beforeScript` for setup operations, `publishDir` with `saveAs` for selective file handling.

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

### fair

Use `fair` to guarantee that a process emits its outputs in the same order its inputs were received, even though tasks run concurrently and may finish out of order.

```groovy
process foo {
    fair true

    input:
    val x
    output:
    tuple val(task.index), val(x)

    script:
    """
    sleep \$((RANDOM % 3))
    """
}

workflow {
    Channel.of('A', 'B', 'C', 'D') | foo | view
}
// Output is always in order: [1, A], [2, B], [3, C], [4, D]
```

Without `fair`, faster tasks can emit before slower ones that were submitted earlier, so output order is not guaranteed.

