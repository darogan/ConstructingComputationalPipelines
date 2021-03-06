
# Constructing Computational Pipelines


**Russell S. Hamilton**

*Centre for Trophoblast Research, Department of Physiology, Development and Neuroscience, University of Cambridge, Downing Site, Cambridge, CB2 3DY, UK*


**ChuangKee Ong**

*Open Targets, European Bioinformatics Institute (EMBL-EBI), Wellcome Trust Genome Campus, Cambridge, CB10 1SD, UK*

## Publication

Ong, C-K & Hamilton, R.S. (2018) Constructing Computational Pipelines Encyclopedia of Bioinformatics and Computational Biology, 3, 135-143 [[DOI](https://doi.org/10.1016/B978-0-12-809633-8.20090-3)]

## Worked Example Code and Documentation

Implementation of a simple 5 step RNA-Seq workflow in a selection of pipeline tools: Bash, Clusterflow and eHive

Pipeline Tools | URL
-------------- | --------------
ClusterFlow    | http://clusterflow.io
eHive          | https://github.com/Ensembl/ensembl-hive


The software packages for the basic workflow steps are in the table below and are required to be installed prior to running the pipeline tools.

Resource       | Brief Description | URL
-------------- | ----------------- | ---
FastQC         | Quality assessment on Fastq files               | http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
Trim_galore    | Trim low quality and adapters from Fastq files  | http://www.bioinformatics.babraham.ac.uk/projects/trim_galore/
HiSat2         | Performs alignment of reads to reference genome | https://ccb.jhu.edu/software/hisat2
HTSeq-counts   | Gene level quantification of aligned reads      | http://www-huber.embl.de/HTSeq/doc/count.html
QualiMap       | Quality assessment on alignned reads            | http://qualimap.bioinfo.cipf.es/
MultiQC        | Aggregates results from analyses performed      | http://multiqc.info/



## Bash Script Example of Simple RNA-Seq
In this very simple bash shell script the read files and reference genome should be edited manually at the top of the file:

Change the filenames to match the names of the samples to be run
````
READ1="RNA-Seq-Project.R1.fq.gz"
READ2="RNA-Seq-Project.R2.fq.gz"
````

Change the filenames to correspond to the annotation (GTF) and indexed reference genome appropriate for the samples being analysed
````
GTF="reference_genome.gtf"
INDEX="reference_genome.hisat2.idx"
````

Ensure the script has the executable permissions

    $ chmod 755 SimpleRNA-Seq.sh

Run the script from the command line

    $ ./SimpleRNA-Seq.sh

## Clusterflow Example of Simple RNA-Seq

Download and install clusterflow from the link in the table above. Clusterflow modules from each of the pipeline steps are already included in Clusterflow, with the exception of `qualimap_rnaseq`. This file (`qualimap_rnaseq.cfmod`) is provided in the Clusterflow directory and should be copied to the Clusterflow installation module directory. The simple RNA-Seq pipeline is provided as a file (`SimpleRNA-Seq.config`) and should be copied to the clusterflow installation pipeline directory

Once installed Clusterflow can be run on a set of sample with the following command. Replace the genome reference as appropriate (note these should be specified as part of the Clusterflow install).

````
$ cf --genome <YOURGENOME REF> SimpleRNA-Seq *.fq.gz
````


Results will be written into the directory the `cf` command was run from


## eHive Example of Simple RNA-Seq

Pre-requisite: A Mysql instance

Clone eHive module from github repository in the table above. Setup $PERL5LIB pointing to the ehive cloned repository, next setup eHive Mysql database parameters $HIVE_HOST, $HIVE_PORT, $HIVE_USER, $HIVE_PASS, $hive_dbname.

* Run the pipeline initialisation step using the configuration file:

````
init_pipeline.pl ehiveRNAseq::RNAseq_conf \
  -hive_host ${HIVE_HOST}   \
  -hive_port ${HIVE_PORT}   \
  -hive_user ${HIVE_USER}   \
  -hive_password ${HIVE_PASS} \
  -hive_dbname ${hive_dbname} \
  -data_dir [where your raw data resides] \
  -output_dir [where the output will be] \
  -hive_force_init 1 \
  -flag_pe 1   
````

* After successfully initialised the eHive database for the pipeline, execute the beekeeper script to start creating worker to run jobs.
````
beekeeper.pl -url mysql://[mysql_username]:[mysql_password]@[mysql_hostname]:[mysql_port]/[hive_dbname] -sync
beekeeper.pl -url mysql://[mysql_username]:[mysql_password]@[mysql_hostname]:[mysql_port]/[hive_dbname] -loop
````
