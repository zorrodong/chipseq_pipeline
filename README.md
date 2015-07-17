ENCODE ChIP-Seq Pipelines
===============================================

ENCODE ChIP-Seq pipelines are based on https://docs.google.com/document/d/1lG_Rd7fnYgRpSIqrIfuVlAz2dW1VaSQThzk836Db99c/edit# .

Taking advandatge of the powerful pipeline language BigDataScript (http://pcingola.github.io/BigDataScript/index.html), ENCODE ChIP-Seq pipelines have the following features:

```
1) One-command-line installation for all dependencies for ChIP-Seq pipeline
2) One command line (or one configuration file) to run the whole pipeline
3) Resuming from the point of failure for failed jobs (by comparing timestamps of input/output files)
4) Automatically optimize parallel jobs for the pipeline
5) Sun Grid Engine cluster support
6) Realtime HTML Progress report to monitor the pipeline jobs
```

### Installation instruction (pipelines and their dependencies)

```
# get the latest version of chipseq pipelines
$git clone https://github.com/kundajelab/ENCODE_chipseq_pipeline

# find the tf chipseq pipeline script
$cd ENCODE_chipseq_pipeline
$ls -l tf_chipseq.bds

# install dependencies
$./install_dependencies.sh

# IMPORTANT! Move bds.config to BigDataScript (BDS) directory
$mkdir -p $HOME/.bds
$cp bds.config $HOME/.bds/
```

Add the following lines to your $HOME/.bashrc or $HOME/.bash_profile:

```
# Java settings
export _JAVA_OPTIONS="-Xms256M -Xmx512M -XX:ParallelGCThreads=1"
export MAX_JAVA_MEM="8G"
export MALLOC_ARENA_MAX=4

# BigDataScript settings
export PATH=$PATH:$HOME/.bds
```

### Usage

There are two ways to define parameters for ChIP-Seq pipelines. For most of the parameters, they already have default values. If they are not defined in command line argument or in a configuration file, default value will be used.

1) From command line arguments 
```
# general usage
$bds tf_chipseq.bds [OPTS_FOR_PIPELINE]

# help for parameters
$bds tf_chipseq.bds -h

# example cmd. line args (human, no replicate-2 control fastq and using Anshul Kundaje's IDR)
$bds tf_chipseq.bds \
-prefix ENCSR000EGM \
-input fastq \
-fastq1 /DATA/ENCSR000EGM/ENCFF000YLW.fastq.gz \
-fastq2 /DATA/ENCSR000EGM/ENCFF000YLY.fastq.gz \
-ctl_fastq1 /DATA/ENCSR000EGM/Ctl/ENCFF000YRB.fastq.gz \
-idx_bwa /INDEX/encodeHg19Male_v0.7.10/encodeHg19Male_bwa-0.7.10.fa \
-idr_nboley false
```

2) From a configuration file
```
$bds tf_chipseq.bds [CONF_FILE]

# example configuration file (human, no replicate-2 control fastq and using Nathan Boley's IDR)
$cat [CONF_FILE]

PREFIX=ENCSR000EGM
INPUT_TYPE=fastq
INPUT_FASTQ_REP1= /DATA/ENCSR000EGM/ENCFF000YLW.fastq.gz
INPUT_FASTQ_REP2= /DATA/ENCSR000EGM/ENCFF000YLY.fastq.gz
INPUT_FASTQ_CTL_REP1= /DATA/ENCSR000EGM/Ctl/ENCFF000YRB.fastq.gz
BWA_INDEX_NAME= /INDEX/encodeHg19Male_v0.7.10/encodeHg19Male_bwa-0.7.10.fa
USE_IDR_NBOLEY=true
```

IMPORTANT! For generating bwa index, we recommend to use bwa 0.7.10.

### For cluster use (Sun Grid Engine only)

Add "-s sge" to the command line.

```
$bds -s sge tf_chipseq.bds ...
```

### Debugging and logging for BDS

```
# make BDS verbose
$bds -v tf_chipseq.bds ...

# display debugging information
$bds -d tf_chipseq.bds ...

# test run (this actually does nothing) to check input/output file names and commands
$bds -dryRun tf_chipseq.bds ...
```

For better debugging, an HTML progress report in the working directory (where you run the pipeline command) will be useful. You can monitor your BDS jobs real time.

### Signal track generation for tagAlign files (example for human, hg19)

Add the following command line argument to generate signal tracks.

```
# to generate wig or bedgraph, and convert bedgraph to bigwig. if you don't want wig, remove -wig true \
bds tf_chipseq.bds \
... \
-wig true \
-bedgraph true \
-bigwig true \
-seq /DATA/encodeHg19Male \
-umap /DATA/encodeHg19Male/globalmap_k20tok54 \
-chrsz /DATA/hg19.chrom.sizes
```

Or add the following lines to the configuration file (for human, hg19).
```
CREATE_WIG= true 	// to create wig
CREATE_BEDGRAPH= true	// to create bedgraph
CONVERT_TO_BIGWIG= true	// to convert bedgraph to bigwig

SEQ_DIR=/DATA/encodeHg19Male
UMAP_DIR=/DATA/encodeHg19Male/globalmap_k20tok54
CHROM_SIZES=/DATA/hg19.chrom.sizes
```

Seq_dir is the directory where reference genome files exist. Umap files are provided at http://www.broadinstitute.org/~anshul/projects/encode/rawdata/umap/


### List of all parameters for TF ChIP-Seq pipeline

For advanced users, all command line parameters for the pipeline is listed and explained below:

```
# general
	-c <string>              : Configuration file path (if not specified, define parameters in command line argument).
	-prefix <string>         : Prefix for all outputs.
	-o <string>              : Output directory. (default: out)
	-tmp <string>            : Temporary directory for intermediate files. (default: tmp).
	-wt <int>                : Default walltime in seconds for all cluster jobs (default: 10800).
	-nth <int>               : Default number of threads for all cluster jobs (default: 1).
	-mem <int>               : Default max. memory in MB for all cluster jobs (default: 4000).

# handling environment variables
	-mod <string>            : Modules separated by ; (example: "bowtie/2.2.4; bwa/0.7.7; picard-tools/1.92").
	-shcmd <string>          : Shell cmds separated by ;. Env. vars should be written as ${VAR} not as $VAR (example: "export PATH=${PATH}:/usr/test; VAR=test").
	-addpath <string>        : Path to be PREPENDED to env. var. PATH. Multiple paths should be separated by ; (example: "/bin/test:/bin/test2")

# inputs
	-input <string>          : Input file type: two options (fastq: including mapping of fastqs, tagalign)

	# if inputs are fastqs
	-fastq1 <string>         : Path for fastq for replicate 1 (single ended).
	-fastq2 <string>         : Path for fastq for replicate 2 (single ended).
	-ctl_fastq1 <string>     : Path for control fastq for replicate 1 (single ended).
	-ctl_fastq2 <string>     : Path for control fastq for replicate 2 (single ended, if not exists leave this blank).

	-fastq1_1 <string>       : Path for fastq for replicate 1 pair 1 (paired-end).
	-fastq1_2 <string>       : Path for fastq for replicate 1 pair 2 (paired-end).
	-fastq2_1 <string>       : Path for fastq for replicate 2 pair 1 (paired-end).
	-fastq2_2 <string>       : Path for fastq for replicate 2 pair 2 (paired-end).
	-ctl_fastq1_1 <string>   : Path for control fastq for replicate 1 pair 1 (paired-end).
	-ctl_fastq1_2 <string>   : Path for control fastq for replicate 1 pair 2 (paired-end).
	-ctl_fastq2_1 <string>   : Path for control fastq for replicate 2 pair 1 (paired-end, if not exists leave this blank).
	-ctl_fastq2_2 <string>   : Path for control fastq for replicate 2 pair 2 (paired-end, if not exists leave this blank).

	# if inputs are tagaligns
	-tagalign_PE <bool>      : Set it true if tagaligns are paired end (default: false).
	-tagalign1 <string>      : Path for tagAlign for replicate 1.
	-tagalign2 <string>      : Path for tagAlign for replicate 2.	
	-ctl_tagalign1 <string>  : Path for control tagAlign for replicate 1.
	-ctl_tagalign2 <string>  : Path for control tagAlign for replicate 2 (if not exists leave this blank).

# bwa
	-idx_bwa <string>        : Path for bwa index.
	-param_bwa <string>      : Parameters for bwa align (default: "-q 5 -l 32 -k 2").
	-nth_bwa_aln <int>       : Number of threads for bwa aln (default: 2).
	-wt_bwa_aln <int>        : Walltime in seconds for bwa aln (default: 36000).
	-mem_bwa_aln <int>       : Max. memory in MB for bwa aln (default: 8000).
	-wt_bwa_sam <int>        : Walltime in seconds for bwa sampe/samse (default: 36000).
	-mem_bwa_sam <int>       : Max. memory in MB for bwa sampe/samse (default: 8000).

# signal track generation
	-wig <bool>              : Set it true to create wig (default: false).
	-bedgraph <bool>         : Set it true to create bedgraph (default: false).
	-bigwig <bool>           : Set it true to convert bedgraph to bigwig signal track (default: false).
	-umap <string>           : Path for umap (for hg19, path for globalmap_k20tok54).
	-seq <string>            : Path for sequence files (for hg19, directory where chr*.fa exist).
	-chrsz <string>          : Path for chrom.sizes file for your sequence files.

# etc.	
	-mapq_thresh <int>       : MAPQ_THRESH (default: 30).
	-nreads <int>            : NREADS (default. 15000000).
	-qc <bool>               : Set it true to test-run and stop before peak calling, false: keep going through IDR (default: false).
	-num_rep <int>           : Number of replicates, define it for qc = true only. (default: 2).

# spp	
	-nth_spp <int>           : Number of threads for spp (run_spp.R) (default: 2).
	-wt_spp <int>            : Walltime in seconds for spp (default: 40000).
	-mem_spp <int>           : Max. memory in MB for spp (default: 8000).
	-npeak <int>             : Parameter for -npeak in phantompeakqual tool run_spp.R (default: 300000).

# idr	
	-idr_thresh <string>     : IDR thresh (default: 0.02).
	-idr_nboley <bool>       : Use Nathan Boley's code for IDR, otherwise Anshul Kundaje's code (default: true)	
```

Equivalent parameters in a configuration file is listed and explained below:

```
# general
	PREFIX 		: Prefix for all outputs.
	OUTPUT_DIR 	: Output directory. (default: out)
	TMP_DIR 	: Temporary directory for intermediate files. (default: tmp).

	WALLTIME 	: Default walltime in seconds for all cluster jobs (default: 10800).
	NTHREADS 	: Default number of threads for all cluster jobs (default: 1).
	MEMORY 		: Default max. memory in MB for all cluster jobs (default: 4000).

# handling environment variables
	MODULE 		: Modules separated by ; (example: "bowtie/2.2.4; bwa/0.7.7; picard-tools/1.92").
	SHELLCMD	: Shell cmds separated by ;. Env. vars should be written as ${VAR} not as $VAR (example: "export PATH=${PATH}:/usr/test; VAR=test")
	ADDPATH		: Paths to be added to env. var. PATH separated by ; or :. (a quicker way to add PATH)

# inputs
        INPUT_TYPE              : Input file type: two options (fastq: including mapping of fastqs, tagalign: starting from tagaligns)

        # if inputs are fastqs
	INPUT_FASTQ_REP1        : Path for input fastq for replicate 1 (single ended).
	INPUT_FASTQ_REP2        : Path for input fastq for replicate 2 (single ended).
	INPUT_FASTQ_CTL_REP1    : Path for control fastq for replicate 1 (single ended).
	INPUT_FASTQ_CTL_REP2    : Path for control fastq for replicate 2 (single ended, if not exists leave this blank).).

	INPUT_FASTQ_REP1_PE1    : Path for input fastq for replicate 1 pair 1 (paired-end).
	INPUT_FASTQ_REP1_PE2    : Path for input fastq for replicate 1 pair 2 (paired-end).
	INPUT_FASTQ_REP2_PE1    : Path for input fastq for replicate 2 pair 1 (paired-end, if not exists leave this blank).
	INPUT_FASTQ_REP2_PE2    : Path for input fastq for replicate 2 pair 2 (paired-end, if not exists leave this blank).
	INPUT_FASTQ_CTL_REP1_PE1: Path for control fastq for replicate 1 pair 1 (paired-end).
	INPUT_FASTQ_CTL_REP1_PE2: Path for control fastq for replicate 1 pair 2 (paired-end).
	INPUT_FASTQ_CTL_REP2_PE1: Path for control fastq for replicate 2 pair 1 (paired-end).
	INPUT_FASTQ_CTL_REP2_PE2: Path for control fastq for replicate 2 pair 2 (paired-end).

        # if inputs are tagaligns
	TAGALIGN_PE: Set it true if tagaligns are paired end

	INPUT_TAGALIGN_REP1    : Path for input tagalign for replicate 1.
	INPUT_TAGALIGN_REP2    : Path for input tagalign for replicate 2.
	INPUT_TAGALIGN_CTL_REP1: Path for control tagalign for replicate 1.
	INPUT_TAGALIGN_CTL_REP2: Path for control tagalign for replicate 2 (if not exists, leave this blank).

# bwa
	BWA_INDEX_NAME         : Path for bwa index.
	BWA_ALN_PARAM          : Parameters for bwa align (default: "-q 5 -l 32 -k 2").
	NTHREADS_BWA_ALN       : Number of threads for bwa aln (default: 2).
	WALLTIME_BWA_ALN       : Walltime in seconds for bwa aln (default: 36000).
	MEMORY_BWA_ALN         : Max. memory in MB for bwa aln (default: 8000).
	WALLTIME_BWA_SAM       : Walltime in seconds for bwa sampe/samse (default: 36000).
	MEMORY_BWA_SAM         : Max. memory in MB for bwa sampe/samse (default: 8000).

# signal track generation
	CREATE_WIG             : Set it true to create wig (default: false).
	CREATE_BEDGRAPH        : Set it true to create bedgraph (default: false).
	CONVERT_TO_BIGWIG      : Set it true to convert bedgraph to bigwig signal track (default: false).
	UMAP_DIR               : Path for umap (for hg19, path for globalmap_k20tok54).
	SEQ_DIR                : Path for sequence files (for hg19, directory where chr*.fa exist).
	CHROM_SIZES            : Path for chrom.sizes file for your sequence files.

# etc.	
	MAPQ_THRESH            : MAPQ_THRESH (default: 30).
	NREADS                 : NREADS (default. 15000000).
	QC_ONLY                : Set it true to test-run and stop before peak calling, false: keep going through IDR (default: false).
	NUM_REP                : Number of replicates, define it for qc = true only. (default: 2).

# spp	
	NTHREADS_SPP           : Number of threads for spp (run_spp.R) (default: 2).
	WALLTIME_SPP           : Walltime in seconds for spp (default: 40000).
	MEMORY_SPP             : Max. memory in MB for spp (default: 8000).
	NPEAK                  : Parameter for -npeak in phantompeakqual tool run_spp.R (default: 300000).

# idr	
	IDR_THRESH             : IDR thresh (default: 0.02).
	USE_IDR_NBOLEY         : Use Nathan Boley's code for IDR, otherwise Anshul Kundaje's code (default: true)	
```


### What are MODULE, SHELLCMD and ADDPATH? (handling environment variables)

It is important to define enviroment variables (like $PATH) to make bioinformatics softwares in the pipeline work properly. MODULE, SHELLCMD and ADDPATH are three convenient ways to define environment variables. Environment variables defined with MODULE, SHELLCMD and ADDPATH are preloaded for all tasks on the pipeline. For example, if you define environment variables for bwa/0.7.10 with MODULE. bwa of version 0.7.10 will be used throughout the whole pipeline (including bwa aln, bwa same and bwa sampe).

1) MODULE

There are different versions of bioinformatics softwares (eg. samtools, bedtools and bwa) and <a href="http://modules.sourceforge.net/">Enviroment Modules</a> is the best way to manage environemt variables for them. For example, if you want to add environment variables for bwa 0.7.10 by using Environment Modules. You can simply type the following:

```
$module add bwa/0.7.10;
```

The equivalent setting in the pipeline configuration file should look like:
```
MODULE= bwa/0.7.10;
```

You can have multiple lines for MODULE since any suffix is allowed. Use ; as a delimiter.
```
MODULE_BIO= bwa/0.7.10; bedtools/2.x.x; samtools/1.2
MODULE_LANG= r/2.15.1; java/latest
```

2) SHELLCMD

If you have softwares locally installed on your home, you may need to add to them environment variables like $PATH, $LD_LIBRARY_PATH and so on. <b>IMPORTANT!</b> Note that any pre-defined enviroment variables (like $PATH) should be referred in a curly bracket like ${PATH}. This is because BDS distinguishes environment variables from BDS variables by a curly bracket ${}.
```
SHELLCMD= export PATH=${PATH}:path_to_your_program
```

You can have multiple lines for SHELLCMD since any suffix is allowed. Use ; as a delimiter. 
```
SHELLCMD_R= export PATH=${PATH}:/home/userid/R-2.15.1;
SHELLCMD_LIB= export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${HOME}/R-2.15.1/lib
```

SHELLCMD is not just for adding environemt variables. It can execute any bash shell commands prior to any jobs on the pipeline. For example, to give all jobs peaceful 10 seconds before running.
```
SHELLCMD_SLEEP_TEN_SECS_FOR_ALL_JOBS= echo "I am sleeping..."; sleep 10
```

3) ADDPATH

If you just want to add something to your $PATH, use ADDPATH instead of SHELLCMD. It's much simpler. Use : or ; as a delimiter.

```
ADDPATH= ${HOME}/program1/bin:${HOME}/program1/bin:${HOME}/program2/bin:/usr/bin/test
```

### What are -mod, -shcmd and -addpath?

They are command line argument versions of MODULE, SHELLCMD and ADDPATH. For example,

```
$bds tf_chipseq.bds -mod 'bwa/0.7.10; samtools/1.2' -shcmd 'export PATH=${PATH}:/home/userid/R-2.15.1' -addpath '${HOME}/program1/bin'
```

### Contributors

* Jin wook Lee - PhD Student, Mechanical Engineering Dept., Stanford University
* Anshul Kundaje - Assistant Professor, Dept. of Genetics, Stanford University
