Instructions for installing and running aMeta
=====

`aMeta <https://github.com/NBISweden/aMeta>`_ is a Snakemake-based Metagenomic profiling workflow. 

Installing aMeta
------------

aMeta requires Conda for installation. 

1. Load Miniforge

.. code-block:: console  

   module load Miniforge/24.3.0-0

2. Initialize Conda

.. code-block:: console  

   conda init

3. Log out of Viking and back in again to initialise Conda upon start up. After logging in, you should see this in your Shell prompt:

.. code-block:: console  

   (base) [abc123@login1[viking2] ~]$

4. Clone the aMeta repository

.. code-block:: console

   git clone https://github.com/NBISweden/aMeta

5. Create the Conda environment

.. code-block:: console

   conda env create -f workflow/envs/environment.yaml

6. Activate aMeta

.. code-block:: console

   conda activate aMeta

7. Update Krona taxonomy

.. code-block:: console

   ktUpdateTaxonomy.sh

Running the test
------------

8. Submit the test job by sbatch rather than running it directly on the login node. 

Example shell script:

.. code-block:: console 

   #!/usr/bin/env bash
   #SBATCH --job-name=aMeta_test            # Job name
   #SBATCH --partition=nodes               # What partition the job should run on
   #SBATCH --time=05:00:00               # Time limit (DD-HH:MM:SS)
   #SBATCH --ntasks=1                      # Number of MPI tasks to request
   #SBATCH --cpus-per-task=16               # Number of CPU cores per MPI task
   #SBATCH --mem=64G                        # Total memory to request
   #SBATCH --account=arch-adna-2019       # Project account to use
   #SBATCH --mail-type=ALL            # Mail events (NONE, BEGIN, END, FAIL, ALL)
   #SBATCH --mail-user=abc@york.ac.uk   # Where to send mail
   #SBATCH --output=%x-%j.log              # Standard output log
   #SBATCH --error=%x-%j.err               # Standard error log
   #SBATCH -D /path/to/aMeta/.test
   
   # Abort if any command fails
   set -e
   
   # Purge any previously loaded modules
   module purge
   
   # Load modules
   . /opt/apps/eb/software/Miniforge/24.3.0-0/etc/profile.d/conda.sh
   
   conda activate aMeta
   
   # Commands to run
   echo Job started on $(date)
   
   ./runtest.sh -j 16
   
   echo Job ended on $(date)


Preparing fastq files for your run
------------

aMeta only accepts single end reads, so if you have paired-end reads, use `AdapterRemoval <https://adapterremoval.readthedocs.io/en/2.3.x/index.html>`_ to merge the reads (shell script here). Ensure that your reads end with the prefix `.fastq.gz`. 

You can also just concatenate the forward and reverse read pairs, or use `fastp <https://github.com/OpenGene/fastp>`_ to merge the reads as recommended `here <https://github.com/NBISweden/aMeta>`_ in the FAQ section.


Preparing the config files
------------

Creating the sample tsv
^^^^^^^^^^^^
Create the sample tsv file in the config folder in your aMeta directory. Ensure the sample names are concise and without an excess of special characters, e.g., ``SAMPLE1`` instead of ``Sample1_project_year_x%x``. 

.. code-block:: console

   sample   fastq
   SAMPLE1   /path/to/SAMPLE1.fastq.gz
   SAMPLE2   /path/to/SAMPLE2.fastq.gz
   SAMPLE3   /path/to/SAMPLE3.fastq.gz

To check if your sample file is tab separated, do

.. code-block:: console

   cat -A samples.tsv

The output should be the contents of the file and ``^I`` where tabs are expected, and ``$`` for end of line.


Editing the config file
^^^^^^^^^^^^
In the ``config`` folder, edit the ``config.yaml`` as follows (depending on if you want to run only prokaryotes or prokaryotes + eukaryotes):

.. code-block:: console
   
   samplesheet: "config/samples_kz.tsv"
   # Databases
   krakenuniq_db: /mnt/scratch/projects/biol-db/aMeta/KrakenUniq-part-NCBI-NT-June-2020
   pathogenomesFound: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-pathogenic-microbial-species-NCBI-NT-from-May-2020/pathogensFound.very_inclusive.tab
   #bowtie2_seqid2taxid_db: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-full-NCBI-NT-January-2021/seqid2taxid.map.orig
   bowtie2_seqid2taxid_db: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-pathogenic-microbial-species-NCBI-NT-from-May-2020/seqid2taxid.pathogen.map
   bowtie2_db: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-pathogenic-microbial-species-NCBI-NT-from-May-2020/library.pathogen.fna
   #bowtie2_db: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-full-NCBI-NT-January-2021/library.fna
   malt_seqid2taxid_db: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-full-NCBI-NT-January-2021/seqid2taxid.map.orig
   malt_nt_fasta: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-full-NCBI-NT-January-2021/library.fna
   malt_accession2taxid: /mnt/scratch/projects/biol-db/aMeta/Bowtie2-full-NCBI-NT-January-2021/nucl_gb.accession2taxid
   ncbi_db: resources/ncbi
   
   # Breadth and depth of coverage filters
   n_unique_kmers: 1000
   n_tax_reads: 200

Here, only the microbial NT database for Bowtie2 and KrakenUniq is used (the full NT database is commented out). You can find all the databases in ``/mnt/scratch/projects/biol-db/aMeta``


Editing the profile config file
^^^^^^^^^^^^

This is where you need to modify the runtime, resources, slurm partitions etc. 

.. code-block:: console

   keep-going: true
   restart-times: 1
   max-jobs-per-second: 1
   max-status-checks-per-second: 0.2
   local-cores: 1
   latency-wait: 600
   rerun-incomplete: true
   ##############################
   # Resources; fine-tune at will
   ##############################
   default-resources:
     - runtime=360
     - mem_mb=16384
     - disk_mb=1000000
     - threads=10
     - slurm_partition=nodes
   
   set-threads:
     - Cutadapt_Adapter_Trimming=8
     - Bowtie2_Pathogenome_Alignment=10
     - Malt=16
     - Build_Malt_DB=16
     - KrakenUniq=32
   
   set-resources:
     - FastQC_BeforeTrimming:mem_mb=2024
     - FastQC_AfterTrimming:mem_mb=2024
     - Bowtie2_Alignment:mem_mb=102400
     - KrakenUniq:mem_mb=512000
     - KrakenUniq:runtime=1440
     - KrakenUniq:slurm_partition=himem
     - Build_Malt_DB:mem_mb=512000
     - Build_Malt_DB:runtime=480
     - Build_Malt_DB:slurm_partition=himem
     - Malt:runtime=480
     - Malt:mem_mb=512000
   ##############################
   # Custom additions requiring additional scripts / resources
   ##############################
   ### Scripts for improving control of job submission
   #jobscript: ""
   #cluster: sbatch
   jobs: 40
   # cluster-status: "status.py"

Modify any of the parameters as per your files. The full NT database will require a lot more memory and runtime than just the pathogens database. If you want to know more about the partitions and runtime limits click `here <https://vikingdocs.york.ac.uk/using_viking/resource_partitions.html>`_.

Preparing for the run
------------

Once this is done, create a new ``tmux`` session, and vertically divide the terminal using the commands ``Ctrl B`` +  ``%``. To move between panes, use ``Ctrl B`` + ``→`` to move to the right pane or ``Ctrl B`` + ``←`` to move to the left. For more ``tmux`` options look up tmux cheatsheets.

In the right pane, activate the aMeta Conda environment

.. code-block:: console

   conda activate aMeta

Next, go into the aMeta folder or your work directory and create all the Conda environments such as MapDamage, Krona, FastQC, etc (a total of nine environments). This will take a bit of time to install. You can also use local Viking modules by editing ``envmodules.yaml`` in the ``config`` folder. To avoid dependency conflicts, I use the Conda environments. 

.. code-block:: console

   cd aMeta
   # install job-specific environments, and mention conda-frontend else it will try to use libmamba
   snakemake --snakefile workflow/Snakefile --use-conda --conda-create-envs-only -j 20 --conda-frontend conda

Then update the Krona taxonomy

.. code-block:: console

   env=$(grep krona .snakemake/conda/*yaml | awk '{print $1}' | sed -e "s/.yaml://g" | head -1)
   cd $env/opt/krona/
   ./updateTaxonomy.sh taxonomy
   cd -

Modify default Java heap space parameters for Malt jobs

.. code-block:: console

   env=$(grep hops .snakemake/conda/*yaml | awk '{print $1}' | sed -e "s/.yaml://g" | head -1)
   conda activate $env
   version=$(conda list malt --json | grep version | sed -e "s/\"//g" | awk '{print $2}')
   cd $env/opt/malt-$version
   sed -i -e "s/-Xmx64G/-Xmx512G/" malt-build.vmoptions
   sed -i -e "s/-Xmx64G/-Xmx512G/" malt-run.vmoptions
   cd -
   conda deactivate

Running aMeta
------------

Time to run aMeta! But before that, do a Snakemake dry run, which will check if the workflow is defined properly. This can be done by adding ``-n`` in the command (dry run). It will tell you if there are an issues with your workflow, and will estimate computation times. 

.. code-block:: console

   snakemake --snakefile ../workflow/Snakefile --use-conda --profile profile \
   --cluster "sbatch --time={resources.runtime} --mem={resources.mem_mb} --cpus-per-task={resources.threads} \
   --partition={resources.slurm_partition} \
   --job-name={rule} --output=slurm_logs/{rule}/{rule}_%j.log --error=slurm_logs/{rule}/{rule}_%j.err" \
   --printshellcmds --rerun-incomplete --conda-frontend conda  --cluster-cancel "scancel" -n

Now, run the same command as above but without ``-n``. This should submit a series of jobs to the Slurm scheduler. 

To monitor which jobs are running, navigate to the second pane of your vertically split tmux session, and type 

.. code-block:: console

   watch squeue --me

This will update every two seconds. 


Optional steps
------------

If the jobs are cancelled on Slurm due to insufficient runtime or memory, Snakemake does not automatically stop, but will hang. In order to prevent this, copy the following script and name it ``slurm_script.py`` where a failed Slurm job wil flag Snakemake, and the workflow will either keep going or terminate (depending on the step). 

.. code-block:: console

   #!/usr/bin/env python
   import subprocess
   import sys
   
   jobid = sys.argv[1]
   
   output = str(subprocess.check_output("sacct -j %s --format State --noheader | head -1 | awk '{print $1}'" % jobid, shell=True).strip())
   
   running_status=["PENDING", "CONFIGURING", "COMPLETING", "RUNNING", "SUSPENDED"]
   if "COMPLETED" in output:
     print("success")
   elif any(r in output for r in running_status):
     print("running")
   else:
     print("failed")

Change permissions on the file so it is executable:

.. code-block:: console

   chmod +x slurm_script.py

In ``profile/config.yaml``, at the bottom of the file, change ``# cluster-status: "status.py"`` to ``cluster-status: "slurm_status.py"``, and add ``--parsable`` inside the ``--cluster "sbatch ..."`` section of the Snakemake command. This will ensure that Snakemake receives the job status from Slurm, and will terminate or continue depending on the job. 
