Instructions for installing and running nf-core/eager v2.5.3
=====

`nf-core/eager <https://nf-co.re/eager/2.5.3>`_ is a Nextflow-based pipeline for processing ancient DNA data.


Prerequisites
-------------

- Paired-end FASTQ files named with pattern ``*_1.fastq.gz`` and ``*_2.fastq.gz`` or ``*_1.fq.gz`` and ``*_2.fq.gz``
- Or prepare a TSV file as per the instructions `here <https://nf-co.re/eager/2.5.3/docs/usage/#tsv-input-method>`_ using this `template <https://raw.githubusercontent.com/nf-core/test-datasets/eager/reference/TSV_template.tsv>`_ .

Setup and Execution
------------

1. Start a new tmux session
^^^^^^^^^^^^^^^^^^^

Using tmux allows your pipeline to continue running even if you disconnect from Viking.

.. code-block:: console  

   tmux new -s eager

2. Load the required modules
^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   module purge
   module load Nextflow/22.10.6
   module load Apptainer/latest

3. Set up environment variables for nextflow and singularity
^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Set the path to the directory where you want to run eager from 
   BASE_DIR="/users/$USER/scratch/eager"

   # Create user directories
   mkdir -p "$BASE_DIR"/{nextflow_home,tmp,singularity_tmp}

   # User specific paths
   export NXF_HOME="$BASE_DIR/nextflow_home"
   export NXF_TEMP="$BASE_DIR/tmp"
   export SINGULARITY_TMPDIR="$BASE_DIR/singularity_tmp"

   # Shared paths where pre-installed nf-core/eager image is stored
   export NXF_SINGULARITY_CACHEDIR="/mnt/scratch/projects/arch-adna-2019/eager/singularity_cache"
   export SINGULARITY_CACHEDIR="/mnt/scratch/projects/arch-adna-2019/eager/singularity_cache"

   # Link project code associated with user account to run jobs on SLURM
   export USER_ACCOUNT=arch-adna-2019

4. Set paths for the input files to be used in the pipeline
^^^^^^^^^^^^^^^^^^^

**CHANGE THESE** to match your data and reference genome locations.

.. code-block:: console

   SHARED_CONFIG="/mnt/scratch/projects/arch-adna-2019/eager/configs/viking.config"
   INPUT_DATA="/path/to/data/*_{1,2}.fastq.gz"  # or .fq.gz
   REF_DIR="/path/to/reference"
   REFERENCE="${REF_DIR}/genome.fasta"
   BWA_INDEX="${REF_DIR}"
   OUTPUT_DIR="${BASE_DIR}/results"

5. Run the pipeline 
^^^^^^^^^^^^^^^^^^^

We are using a combination of the `University of York institutional profile <https://nf-co.re/configs/york_viking/>`_ , singularity, and a custom configuration to be able to run the script using Slurm as an executor and the container for eager. 

.. code-block:: console

   nextflow run nf-core/eager \
   --input "${INPUT_DATA}" \
   --fasta "${REFERENCE}" \
   --fasta_index "${REFERENCE}.fai" \
   --bwa_index "${BWA_INDEX}" \
   --outdir "${OUTPUT_DIR}" \
   -profile york_viking,singularity \
   -with-singularity "${SINGULARITY_CACHEDIR}/nfcore-eager-2.5.3.img" \
   -c "${SHARED_CONFIG}"
 
The ``-resume`` flag allows the pipeline to continue from the last completed step if it fails or is interrupted.

Monitoring Progress
-------------------

.. code-block:: console

   # To detach from your tmux session
   # Press: Ctrl+b, then d

   # Check SLURM job status
   squeue --me
   
   # Watch Nextflow log in real-time in your $BASE_DIR
   tail -f .nextflow.log

   # Reattach to tmux session later
   tmux attach -t eager

Cleaning up the directories
-------------------

To clean up the intermediate files, run

.. code-block:: console

   nextflow clean -f -k #-f is for force and -k is to keep the most recent successful run's cached result

You can still resume your job after cleaning

After your run has successfully completed and you don't need to resume or repeat the pipeline for any of the samples, you can safely delete the work directory. 

.. code-block:: console

   rm -rf work # or rm -r work if you want the safety prompts