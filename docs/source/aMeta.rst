Instructions for installing and running `aMeta <https://github.com/NBISweden/aMeta>`
=====

aMeta is a Snakemake-based Metagenomic profiling workflow. 

Installing aMeta
------------

aMeta requires Conda for installation. 

Load Miniforge

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

8. Submit the test job by sbatch rather than running it directly on the login node (shell script here)










