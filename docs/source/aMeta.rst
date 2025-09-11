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

Create a sample tsv file in the config folder in your aMeta directory. Ensure the sample names are concise and without an excess of special characters, e.g., SAMPLE1 instead of Sample1_project_year_x%x. 

.. code-block:: console

   sample   fastq
   SAMPLE1   /path/to/SAMPLE1.fastq.gz
   SAMPLE2   /path/to/SAMPLE2.fastq.gz
   SAMPLE3   /path/to/SAMPLE3.fastq.gz

To check if your sample file is tab separated, do

.. code-block:: console

   cat -A samples.tsv

The output should be the contents of the file and `^I` where tabs are expected.

















