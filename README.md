# Blobtools_for_removing_contaminations_in_genome_assembly
This repository contains scripts I used to create blob database for a de novo genome assembly for visualisation and removing contaminants
## 1. Installing blobtools and dependencies
It can be installed with conda following the instructions provided [here](https://blobtoolkit.genomehubs.org/install/)
Instead of running `sudo` command as instructed in above link. you can also install `firefox` `xvfb` with conda in the same environment where you install blobtools. [firefox](https://anaconda.org/conda-forge/firefox) [xvfb](https://anaconda.org/conda-forge/xvfbwrapper)

In Canon the quickest approach to use blobtools is using an existing Docker image.  you can use singularity pull to download the Docker (technically OCI image) and convert it into a singularity SIF file
```
singularity pull --disable-cache docker://genomehubs/blobtoolkit:3.4.0
```
Singularity is not on the login node in canon so start an interactive job and use the compute node to run above command.
Also `--disable-cache` option avoids having the downloaded Docker image layers occupy space in the Singularity cache directory, which is in your home directory by default.
To run the blobtools image:
```
singularity exec --cleanenv /path/to/blobtoolkit_3.4.0.sif blobtool <options>
```

## Creating blobtools datasets for Euborellia genome
```
singularity exec --cleanenv /n/home12/upendrabhattarai/bin/blobtoolkit/blobtools3/blobtoolkit_3.4.2.sif blobtools create --fasta ASSEMBLY_Euborellia.fasta --taxid 146833 --taxdump taxdump --meta ASSEMBLY_Euborellia.yaml ./DATASETS/ASSEMBLY_Euborellia
```
## Adding blastn and diamond blastx hits to the dataset
```
singularity exec --cleanenv /n/home12/upendrabhattarai/bin/blobtoolkit/blobtools3/blobtoolkit_3.4.2.sif blobtools add --hits ../blob_blastn/results/blob.blastn.euborellia.ncbi.out --hits ../blob_diamond/Euborellia.diamond.blastx.out --taxrule bestsumorder --taxdump taxdump/ ./DATASETS/ASSEMBLY_Euborellia
```

## Filter the dataset
Trying to filter both the bacterial hits and viruses hits from superkingdom in blobtools was not working.
So I removed bacterial reads with:
```
singularity exec --cleanenv /n/home12/upendrabhattarai/bin/blobtoolkit/blobtools3/blobtoolkit_3.4.2.sif blobtools filter --param bestsumorder_superkingdom--Keys=Bacteria --fasta ASSEMBLY_Euborellia.fasta --output filter.json/no.bacteria.json --summary STDOUT DATASETS/ASSEMBLY_Euborellia/
```
Need to rename the filtered output, it comes as: ASSEMBLY_Euborellia.filtered.fasta
Then I extracted the reads from virus hits using --invert option in blobtools filter:
```
singularity exec --cleanenv /n/home12/upendrabhattarai/bin/blobtoolkit/blobtools3/blobtoolkit_3.4.2.sif blobtools filter --param bestsumorder_superkingdom--Keys=Viruses --invert --fasta ASSEMBLY_Euborellia.fasta --output filter.json/only.viruses.json --summary STDOUT DATASETS/ASSEMBLY_Euborellia/
```
To remove virus reads from the filtered assembly (filtered for bacterial reads). I used BBMap software.
Module can be loaded in fasrc as:
```
module load GCC/7.3.0-2.30 OpenMPI/3.1.1 BBMap/38.35
```
Run the filterbyname.sh script of BBMap tools. It can take only virus readssfasta file as input to remove those reads from assembly with no. bacterial reads to get final filtered assembly.
```
filterbyname.sh in=ASSEMBLY_Euborellia.filter.no.bacteria.fasta out=ASSEMBLY_Euborellia.filter.no.bacteria.no.viruses.fasta names=ASSEMBLY_Euborellia.filtered.only.viruses.fasta
java -ea -Xmx2949m -cp /n/sw/eb/apps/centos7/MPI/GCC/7.3.0-2.30/OpenMPI/3.1.1/BBMap/38.35/current/ driver.FilterReadsByName in=ASSEMBLY_Euborellia.filter.no.bacteria.fasta out=ASSEMBLY_Euborellia.filter.no.bacteria.no.viruses.fasta names=ASSEMBLY_Euborellia.filtered.only.viruses.fasta
```
