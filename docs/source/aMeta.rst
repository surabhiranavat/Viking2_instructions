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

8. Submit the test job by sbatch rather than running it directly on the login node (shell script here)


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

The output should be the contents of the file and ``^I`` where tabs are expected.


Editing the config file
^^^^^^^^^^^^
In the ``config`` folder, edit the ``config.yaml`` as follows:

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
     - runtime=120
     - mem_mb=16384
     - disk_mb=1000000
     - threads=8
     - slurm_partition=nodes
   
   set-threads:
     - Cutadapt_Adapter_Trimming=8
     - Bowtie2_Pathogenome_Alignment=10
     - Malt=10
     - KrakenUniq=32
   
   set-resources:
     - FastQC_BeforeTrimming:mem_mb=2024
     - FastQC_AfterTrimming:mem_mb=2024
     - Bowtie2_Alignment:mem_mb=102400
     - KrakenUniq:mem_mb=512000
     - KrakenUniq:runtime=80
     - KrakenUniq:slurm_partition=himem
     - Build_Malt_DB:mem_mb=512000
     - Build_Malt_DB:runtime=480
     - Build_Malt_DB:slurm_partition=himem
     - Malt:runtime=60
     - Malt:mem_mb=512000
   #  - Malt:slurm_partition=himem_week
   ##############################
   # Custom additions requiring additional scripts / resources
   ##############################
   ### Scripts for improving control of job submission
   #jobscript: "aMeta_ChristinaCarolus_sbatch.sh"
   #cluster: sbatch
   jobs: 40
   # cluster-status: "status.py"



















