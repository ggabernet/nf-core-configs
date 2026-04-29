# nf-core/configs: Cambridge HPC Configuration

All nf-core pipelines have been successfully configured for use on the Cambridge HPC cluster at the [The University of Cambridge](https://www.cam.ac.uk/).
To use, run the pipeline with `-profile cambridge`. This will download and launch the [`cambridge.config`](../conf/cambridge.config) which has been pre-configured
with a setup suitable for the Cambridge HPC cluster. Using this profile, either a docker image containing all of the required software will be downloaded,
and converted to a Singularity image or a Singularity image downloaded directly before execution of the pipeline.

### Install Nextflow

The latest version of Nextflow is not installed by default on the Cambridge HPC
cluster CSD3.

The recommended option is the standard Nextflow self-installing package:

```
# Check that Java 17+ is available
java -version

# Download Nextflow
curl -s https://get.nextflow.io | bash

# Make it executable
chmod +x nextflow

# Move it to a personal bin directory
mkdir -p $HOME/bin
mv nextflow $HOME/bin/

# Add that directory to your PATH if needed
export PATH="$HOME/bin:$PATH"

# Confirm the installation
nextflow info
```

See the official installation guide for the latest details and Java
requirements:

- [nf-core / Nextflow installation guide](https://nf-co.re/docs/usage/getting_started/installation)

If you prefer a user-managed package manager, a simple option is to install
`micromamba` and then follow the nf-core conda-style instructions for creating
an environment with `nextflow`:

- [micromamba installation guide](https://mamba.readthedocs.io/en/stable/installation/micromamba-installation.html)
- [nf-core conda installation instructions](https://nf-co.re/docs/usage/getting_started/installation#conda-installation)

`pixi` (see more info [here](https://pixi.prefix.dev/latest/)) may also work well for personal environment management, but this profile
does not currently provide tested `pixi` instructions, so `micromamba` is the
more conservative recommendation here.

### Set up Singularity cache

Singularity allows the use of containers and will use a caching strategy. First, you might want to set the `NXF_SINGULARITY_CACHEDIR` bash environment variable, pointing at your hpc-work location. If not, it will be automatically assigned to the current directory.

```
# do this once per login, or add these lines to .bashrc
export NXF_SINGULARITY_CACHEDIR=/home/<username>/singularity
```

Once done, and ready to use Nextflow, you can check that the Singularity module is loaded by default when logging on the cluster.


### Run Nextflow

Here is an example with the nf-core pipeline sarek ([read documentation here](https://nf-co.re/sarek/3.3.2)).
The profile defaults to the `icelake` partition, but users can switch to
`icelake-himem` or `sapphire` with `--partition`. The user should also provide
their SLURM project / account with `--project`.

```
# Launch the nf-core pipeline for a test database
# with the Cambridge profile
nextflow run nf-core/sarek -profile test,cambridge --partition icelake --project NAME-SL2-CPU --outdir nf-sarek-test
```

If the project name contains `-SL3-`, the profile applies a 12 h walltime cap.
Otherwise it assumes the standard SL1 / SL2 36 h limit.

**For long runs**, we recommend starting Nextflow inside a `screen` or `tmux`
session so that the Nextflow manager process keeps running after you disconnect
your SSH session.

```
# Start a tmux session
tmux new -s nextflow

# Or start a screen session
screen -S nextflow

# Re-attach later if needed
tmux attach -t nextflow
screen -r nextflow
```

Detaching from `tmux` leaves the workflow running in the background with
`Ctrl-b` then `d`. For `screen`, use `Ctrl-a` then `d`.

**For large runs**, especially with many samples, the Nextflow manager process can
itself use substantial memory. In those cases, it is better to launch `nextflow run ...` inside an interactive `srun` session or submit it via `sbatch`, rather than running it directly on a login node.

All of the intermediate files required to run the pipeline will be stored in the `work/` directory. It is recommended to delete this directory after the pipeline has finished successfully because it can get quite large, and all of the main output files will be saved in the `--outdir` directory anyway.

> NB: You will need an account to use the Cambridge HPC cluster in order to run the pipeline. If in doubt contact IT.
> NB: Nextflow will need to submit the jobs via SLURM to the Cambridge HPC cluster and as such the commands above will have to be executed on one of the login  nodes. If in doubt contact IT.
