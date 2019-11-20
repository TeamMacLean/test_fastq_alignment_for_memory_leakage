## README

Two paired end fastq datasets were consuming >=120 GB RAM memory while analysed using galaxy platform. The datasets were 575 MB (dataset1) and 4.9 GB (dataset2) both R1 and R2 combined. The reference sequence was 1005 MB in size.

The datasets should not be using so much of the memory. We want to check if there is any weird characters in the FASTQ file that is causing memory leakage.

## Method

1) split the paired end data to smaller datasets
2) Align the split files using bwa mem and/or bowtie2 to a reference sequence.
3) check the log files for each split file alignment showing any error due to low memory allocation

## Results

Paired end reads were split such that each split file has 100,000 fastq reads. The command used for splitting the files

```
gunzip -c FASTQFile.fq.gz | split -l 400000
```

Each fastq record has 4 lines, thus, splitting 400000 lines per file to get 100000 fastq reads per file

The split files are in the folders

```
/tsl/scratch/shrestha/test_fastq/D_2_FDPL190728232-1a_HTM22DSXX_L1/R1 and /tsl/scratch/shrestha/test_fastq/D_2_FDPL190728232-1a_HTM22DSXX_L1/R2
/tsl/scratch/shrestha/test_fastq/D_2_FDPL190728232-1a_HTM22DSXX_L4/R1 and /tsl/scratch/shrestha/test_fastq/D_2_FDPL190728232-1a_HTM22DSXX_L4/R2
```

Alignment commands are:

```
cd /tsl/scratch/shrestha/test_fastq/D_2_FDPL190728232-1a_HTM22DSXX_L1
for name in R1/x*; do 
	basefile=$(basename $name)
	sbatch -o ${basefile}_bwamem.log -J $basefile --wrap "bwa mem -o ${basefile}_bwamem.bam refindex R1/${basefile} R2/${basefile}"
done


for name in R1/x*; do 
	basefile=$(basename $name)
	sbatch -o ${basefile}_bowtie2.log -J $basefile --wrap "bowtie2 -x refindex -1 R1/${basefile} -2 R2/${basefile} -S ${basefile}_bowtie2.sam"
done

```

Alignment of all split files using both bowtie2 and bwa mem ran successfully. The suggests that the files are fine and that didn't cause memory leakage.

