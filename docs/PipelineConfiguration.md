# Process Configuration in Nextflow

A Nextflow configuration file (`nextflow.config`) allows you to control pipeline behavior and resource usage without modifying pipeline code. This is especially useful for adapting pipelines to different environments (local, cluster, cloud) or for tuning performance.

## Basic Structure

A config file consists of assignments, blocks, and includes. Comments are allowed using `//` or `#`.

```groovy
// Simple assignment
workDir = 'work'
process.maxErrors = 10

// Block syntax for grouping options
process {
    executor = 'sge'
    queue = 'long'
    memory = '8 GB'
}
```

## Process Scope

The `process` scope is used to set defaults for all processes in your pipeline. You can also use selectors to target specific processes or labels.

```groovy
process {
    cpus = 4
    memory = '8 GB'
    withLabel: big_mem {
        cpus = 16
        memory = '64 GB'
    }
    withName: align_reads {
        queue = 'short'
    }
}
```

- Settings in the process definition override config defaults.
- `withLabel` applies settings to processes with a given label.
- `withName` applies settings to processes with a specific name.

## Profiles

Profiles allow you to define sets of configuration options for different environments. Select a profile at runtime with `-profile`.

```groovy
profiles {
    standard {
        process.executor = 'local'
    }
    cluster {
        process.executor = 'sge'
        process.queue = 'long'
    }
}
```

Activate with:
```
nextflow run main.nf -profile cluster
```

### Configuring Profiles

Profiles are defined in the `profiles` block of your configuration file. Each profile is a named block containing configuration settings that override the defaults when the profile is activated. You can specify multiple profiles, and select one or more at runtime.

**Example:**

```groovy
profiles {
    local {
        process.executor = 'local'
        docker.enabled = false
    }
    slurm {
        process.executor = 'slurm'
        process.queue = 'batch'
        docker.enabled = true
    }
    test {
        params.run_mode = 'test'
        process.cpus = 1
    }
}
```

- To activate a single profile:
  ```
  nextflow run main.nf -profile slurm
  ```
- To activate multiple profiles (settings are merged in order):
  ```
  nextflow run main.nf -profile test,slurm
  ```

**Notes:**
- Profile settings override the base configuration.
- Use profiles to easily switch between local, cluster, or cloud environments, or to set up testing and production modes.
- Avoid mixing dot and block syntax for the same scope within a profile to prevent unexpected overrides.

For more details, see the official [Nextflow configuration documentation](https://www.nextflow.io/docs/latest/config.html#config-profiles).

## Including Other Config Files

You can modularize configuration using `includeConfig`:

```groovy
includeConfig 'path/to/cluster.config'
```

## Useful Constants

- `projectDir`: Directory where the main script is located.
- `launchDir`: Directory where the workflow was launched.
- `workDir`: Directory for intermediate files.

## Conda Options

Nextflow supports configuring Conda environments for process execution. The `conda` scope in your config file controls how Conda environments are created and managed.

Common options include:

```groovy
conda {
    enabled = true                // Enable Conda support (default: false)
    cacheDir = '/path/to/conda'   // Directory to store Conda environments
    channels = ['bioconda','conda-forge'] // List of Conda channels
    createOptions = '--override-channels' // Extra options for `conda create`
    createTimeout = '30 min'      // Timeout for environment creation (default: 20 min)
    useMamba = true               // Use mamba instead of conda (default: false)
    useMicromamba = false         // Use micromamba (default: false)
}
```

- `enabled`: Enables or disables Conda environments.
- `cacheDir`: Path where Conda environments are stored (should be shared if using a cluster).
- `channels`: List or comma-separated string of Conda channels to use.
- `createOptions`: Additional command-line options for `conda create`.
- `createTimeout`: Maximum time allowed for environment creation.
- `useMamba`: Use [mamba](https://mamba.readthedocs.io/) for faster environment creation.
- `useMicromamba`: Use [micromamba](https://micromamba.readthedocs.io/) for lightweight environments.

See the [official documentation](https://www.nextflow.io/docs/latest/conda.html) for more details and advanced usage.

## Summary

- Use config files to separate environment-specific settings from pipeline logic.
- Use process selectors for fine-grained control.
- Use profiles for easy switching between environments.

For more details, see the official [Nextflow configuration documentation](https://www.nextflow.io/docs/latest/config.html).
