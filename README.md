<img src="https://github.com/user-attachments/assets/1b4adeec-c46b-446c-ab59-61b9235cf6e6" alt="investigut" width="555" height="128"/>

## What is InvestiGUT?
InvestiGUT has been designed to allow users to study the ecological prevalence of a protein sequence across human gut microbiome samples.

[![DOI](https://zenodo.org/badge/766999755.svg)](https://doi.org/10.5281/zenodo.14973403)

## How does InvestiGUT work?
InvestiGUT (v1.1) accepts either a single protein sequence or a set of sequences, as defined by the options ‘–s’ or ‘–m’ respectively. User-provided protein sequences are matched to the collection of human gut microbiome proteins via DIAMOND (v2.1.11) alignment with a default minimum query and subject coverage of 90% and identity of 90%, which can be defined by the user. User proteins can be analysed individually (-s), or as a group (-m) where all proteins must be present within a sample to be considered. The group analysis allows users to determine how frequently all enzymes in a certain pathway or all subunits in a protein complex are present. Output is divided into metagenomic and species-based analysis results, with both being available as raw data in the form of TSV files and as automatically generated vector figures. 

Metadata for each metagenomic sample included age, sexgender, BMI, smoker-status, westernised-diet status, use of antibiotics, and geographical location. Additionally, disease prevalence includes output for Crohn’s disease, ulcerative colitis, colorectal cancer, Clostridioides difficile infection, type-2 diabetes, rheumatoid arthritis, and fatty liver disease. Metadata vocabulary is consistent with that used in the curatedMetagenomeData repository. Prevalence of the queried sequence in each of these groups is quantified, and compared either between categories, or healthy controls, using Fisher’s exact test from the scipy.stats module in Python. 

Querying the user input protein against the 3,594 nonredundant genomes provides a list of species which contain the protein, termed “function-positive species”. The taxonomic range of the protein is determined by the prevalence of the protein with each genus, family, and phylum of positive species. The function-positive fraction is obtained as the cumulative relative abundance of function-positive genomes across 4,624 individuals for which pre-computed relative abundance values are available.


## Instructions
1. Clone the repository.  
```bash
git clone https://github.com/Matt-Schmitz/InvestiGUT.git
```
2. Enter the InvestiGUT folder.  
```bash
cd InvestiGUT
```
3. Create the environment with [mamba](https://mamba.readthedocs.io/en/latest/installation/mamba-installation.html).  
```bash
mamba create --no-channel-priority -n investigut \
    -c bioconda -c conda-forge \
    "python=3.11" "numpy=1.24.3" "scipy=1.10.1" \
    "conda-forge::matplotlib-base" "seaborn=0.13.0" \
    "libsqlite=3.48.0" "pandas=1.5.3" "statsmodels=0.13.5" \
    "ete3=3.1.2" "openpyxl=3.0.10" "bioconda::diamond=2.1.11"
```
4. Activate the environment.  
```bash
mamba activate investigut
```
5. Download proteins and create DIAMOND database.  After the DIAMOND database has been created, the compressed fasta data can be deleted.

> [!WARNING]
> The compressed data take up 79GB (two files).
> The DIAMOND database will take up 311GB.

```bash
./download_fasta.sh
```
6. Add the InvestiGUT folder to your `~/.bashrc` and apply the changes.  
- Add to .bashrc: `export PATH="/PATH/TO/InvestiGUT:$PATH"`
- Enter in terminal:
```bash
source ~/.bashrc
```
- Reactivate the mamba environment:
```bash
mamba activate investigut
```

7. Run InvestiGUT on chosen proteins.  
Basic usage:

```bash
investigut.py -i /PATH/TO/FASTA/FILE
```

  
Options list:
> [!TIP]
> Any arguments not recognized by InvesitGut will be passed on to DIAMOND (e.g., `--threads`, `--faster`, `--ultra-sensitive`).
```
-h, --help            show this help message and exit
-i {INPUT}            input fasta file
-o {OUTPUT}           output folder
-s                    single mode (default)
-m                    multi mode
-d {DMND_TSV_OUTPUT}  optional DIAMOND .tsv output file to skip alignment
--query-cover {NUM}   DIAMOND query-coverage (default 90.0)
--subject-cover {NUM}
                      DIAMOND subject-coverage (default 90.0)
--id {NUM}            DIAMOND %id (default 90)
--low                 overrides default DIAMOND query/sujbect coverage and %id giving all a
                      value of 50
```
## Example Usage

In the example folder, a file called "seaweed.fa" contains two bacterial proteins, Bp1670 and Bp1689, that are involved in seaweed digestion (taken from https://www.nature.com/articles/nature08937). In multi mode (`-m`), metagenomes and metagenome-assembled-genomes (MAGs) containing all proteins in the fasta file are counted as positive, whereas in single mode (`-s`), metagenomes and MAGs are separately examined for each protein sequence. The following command searches for matches to both proteins (`-m`) using a subject/query-coverage of 50% and a percetage identity threshhold of 50% (`--low`). 
```bash
investigut.py -i /PATH/TO/seaweed.fa -m --low
```
The resulting output can be found under `./examples/Bp1670 + Bp1689`. In this folder, the "overview.txt" file contains global prevalence within the metagenomes, disease prevalence statistics, country prevalence statistics, and other demographic factor data (smokers vs. non-smokers, BMI, gender, age by decade of life, and antibiotic usage). The MAG overview data include a list of positive gut bacteria, occurrence of the protein(s) by taxonomic rank, and the cumulative relative abundance of species containing the protein(s) of interest within the two cohorts of the origin data (https://www.nature.com/articles/s41467-022-31502-1). 

The following is an example of an auto-generated figure of prevalence by country found within the output folder after running the seaweed digestion proteins mentioned above through InvestiGUT.

![Prevalence_of_Bp1670 + Bp1689_by_Country](https://github.com/Matt-Schmitz/InvestiGut/assets/34464190/12cb4a4a-4cd5-47cd-af14-803e949c310c)

## Database Creation

> [!TIP]
> A premade database is automatically downloaded when following the instructions above. The steps below are only necessary to create a custom database.

1. Clone the repository.  
```bash
git clone https://github.com/Matt-Schmitz/InvestiGUT.git
```
2. Enter the InvestiGut folder.  
```bash
cd InvestiGUT
```
3. Create the environment with [mamba](https://mamba.readthedocs.io/en/latest/installation/mamba-installation.html).  
```bash
mamba create --no-channel-priority -n investigut_database \
    -c conda-forge -c bioconda \
    "python=3.9" "conda-forge::gsl=2.7" \
    "bioconda::augustus=3.5.0" fraggenescan phanotate \
    "bioconda::prodigal=2.6.3" kraken2 "bioconda::pyrodigal" \
    glimmerhmm metagene_annotator snap ete3=3.1.2 \
    "bioconda::nextflow=24.04.2"
```
4. Activate the environment.  
```bash
mamba activate investigut_database
```
5. With Kraken 2 install in the mamba environment, you need to create the taxonomy prediction database.
> [!TIP]
> Check out the Kraken 2 [manual](https://github.com/DerrickWood/kraken2/wiki/Manual) for a more detailed list of options.

Build the standard database.
```bash
kraken2-build --standard --db $DBNAME
```
Alternatively, a custom database can be created. The following database includes bacteria, archaea, and viruses from NCBI.
```bash
kraken2-build --download-taxonomy --db $DBNAME
kraken2-build --download-library bacteria --db $DBNAME
kraken2-build --download-library archaea --db $DBNAME
kraken2-build --download-library viral --db $DBNAME
kraken2-build --build --db $DBNAME
```



6. Make the Kraken 2 database available in RAM for fast taxonomy prediciton.

Make the tmpfs folder that will be mounted.
```bash
mkdir /PATH/TO/tmpfs
```

Edit the system configuration file.
```bash
sudo nano /etc/fstab
```

In the fstab file, add the following line. Modify the path to the tmpfs directory and the size to be large enough for the hash.k2d, opts.k2d, and taxo.k2d files created in step 5.

```bash
tmpfs /PATH/TO/tmpfs tmpfs rw,nodev,noexec,nosuid,size=120g,user,noauto 0 0
```

Reload the modified fstab file.
```bash
sudo systemctl daemon-reload
```

Move the Kraken 2 files into the tmpfs directory.
```bash
mv /PATH/TO/hash.k2d /PATH/TO/tmpfs/hash.k2d
mv /PATH/TO/opts.k2d /PATH/TO/tmpfs/opts.k2d
mv /PATH/TO/taxo.k2d /PATH/TO/tmpfs/taxo.k2d
```

7. Make GeneMark* software available to InvestiGUT. 
The lisence of GeneMark* software does not allow them to be packaged with InvestiGUT, so they must be manually downloaded.

[https://genemark.bme.gatech.edu/GeneMark/license_download.cgi](https://genemark.bme.gatech.edu/GeneMark/license_download.cgi)
```text
GeneMarkS     -> Add    /PATH/TO/gmsn.pl to PATH for usage with InvestiGUT
GeneMarkS2    -> Add    /PATH/TO/gms2.pl to PATH for usage with InvestiGUT
GeneMarkST    -> Add    /PATH/TO/gmst.pl to PATH for usage with InvestiGUT
MetaGeneMark  -> Add     /PATH/TO/gmhmmp to PATH for usage with InvestiGUT
MetaGeneMark2 -> Add /PATH/TO/run_mgm.pl to PATH for usage with InvestiGUT
```

8. Set parameters and run the pipeline.

In the pipeline.nf file, there are the following options.

```bash
params.inputDir = '/PATH/TO/input'
inputDirMAGs = params.inputDir + "/MAGs"
inputDirMetagenomes = params.inputDir + "/metagenomes"
params.outputDir = '/PATH/TO/output'
params.krakendbDir = '/PATH/TO/tmpfs'

params.threads = Runtime.runtime.availableProcessors() // maximum number available
params.memoryMapping = true
pathToNodesDmp = "/PATH/TO/nodes.dmp"

// GeneMark* tools are available for use after download:
params.tools_for_org_group = """{
    "Archaea":["FragGeneScan", "GeneMarkST","MetaGeneMark"], 
    "Bacteria":["MetaGeneAnnotator","MetaGeneMark","Pyrodigal"], 
    "Eukaryota":["Augustus","Pyrodigal","Snap"], 
    "Viruses":["MetaGeneAnnotator","GeneMarkS","GeneMarkS2"]
    }"""


// Leaving these paths blank will assume that they have been added to PATH
// The lisence of GeneMark* software does not allow them to be packaged with InvestiGUT
pathToGeneMarkS = ""     // "/PATH/TO/gmsn.pl"
pathToGeneMarkS2 = ""    // "/PATH/TO/gms2.pl"
pathToGeneMarkST = ""    // "/PATH/TO/gmst.pl"
pathToMetaGeneMark = ""  // "/PATH/TO/gmhmmp"
pathToMetaGeneMark2 = "" // "/PATH/TO/run_mgm.pl"
```

> [!WARNING]
> The input files must be in the format {study_name}__{sample_id}.fa(.gz)? as study_name and sample_id appear in 
[https://waldronlab.io/curatedMetagenomicData/](https://waldronlab.io/curatedMetagenomicData/)

Run the pipeline.
```bash
nextflow pipeline.nf
```
