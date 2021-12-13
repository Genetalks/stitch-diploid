## Introduction

This is an optimized C++ version of original [STITCH](https://github.com/rwdavies/STITCH), aims to genotype  and impute large cohorts for low-pass resequencing data. 

Compared to original STITCH, the optimized version highlights in:
- ***Higher accuracy***
  
   When training HMM model, AC-stitch adopts a `checkpoint && two-pass forward-backward` strategy, achieves genomewide genotype imputation without sharding chromosome into pieces.
- ***Smaller memory footprint***

   The memory footprint for single sample decreases more than 10x, enable to make full use of computing resources on a multi-core server.
- ***Faster***

   With the same parameter settings, the performance doubled.
- ***Incremental analysis***

   This version supports pre-training HMM model based on reference population, which can be used to directly imputation of new samples.

Compared to original STITCH, the optimized version defects in:

- Only the `diploid` model is supported.
- Only VCF output format is supported.
- No diagnostic plots is generated during training process.

## Installation

```sh
git clone https://github.com/Genetalks/stitch.git
cd stitch
sh stitch-*-centos-x86_64.run   # on ubuntu 16.04 / ubuntu 18.04 / ubuntu 20.04
sh stitch-*-centos-x86_64.run   # on centos7 / centos8
```

## Usage

1. Basic command syntax and usage stays the same as original STITCH, type `stitch --help` for details of command line interface and common options.

    A typical examples looks like:
    ```sh
    stitch --chr=chr19 --bamlist=bamlist.txt --posfile=posfile.txt --genfile=gen.txt --outputdir=test --K=4 --nGen=100
    ```

2. Generate intermediate reads from BAM/CRAM file
   
    If you specify option `--generateInputOnly`, the program will quit after generating intermediate reads file used in HMM-EM training process.

3. Training HMM model based on intermediate reads

   -  The output directory stays the same as that of used in generating intermediate reads.
   -  specify option `--no-regenerateInput`.
   -  specify option `--originalRegionName`, the option vaule stays the same as value of `--chr` in generating intermediate reads.

### `N+1` mode

1. Build reference model
    ```sh
    stitch --chr=chr19 \
        --bamlist=bamlist_ref.txt \
        --posfile=pos.txt \
        --outputdir=train_ref_model 
        --K=4 --nGen=100 --nCores=32
    ```

2. Imputation for new samples
    ```sh
    stitch --chr=chr19 \
        --bamlist=bamlist_test.txt \
        --posfile=pos.txt \
        --outputdir=test
        --model_file=train_ref_model/EM.all.chr19.tar.gz
        --K=4 --nGen=100 --nCores=32
    ```
***Note: the pos file `pos.txt` must stay the same as that of used in training process.***


### Key parameters affects performance and memory footprint

- --K
  
  On the premise of ensuring the imputation accuracy, the smaller `K` value is, the better.

- --iterations
  
  On the premise of ensuring model training convergency, the smaller the number of iterations, the better.

- --nCores
 
  * Set to maximum concurrency supported by the system.
  * Don't run multiple tasks on a single machine, the performance degrades due to limited memory bandwidth.
  * It's best to distribute data to different mechines by chromsome to maximize the utilization of multi-machine memory bandwidth.

- --outputSNPBlockSize
  
  * The larger the vaule, the larger the memory.
  * 128000 is recommended to balance memory and computing.

- --bindCPU 
  
  You are advised to specify this parameter to improve performance.


## License
