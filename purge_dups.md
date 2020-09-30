# Goal  
To remove from an assembly overlaps and haplotigs caused by sequence divergence in heterozygous regions.  

# References  
* [Github repository](https://github.com/dfguan/purge_dups)  
* [Paper](https://academic.oup.com/bioinformatics/article/36/9/2896/5714742)

# Software testing  

# Commands  

## 1. purge_dups installation  
Installing *purge_dups* and dependencies (minimap2, runner and KMC (zlib was already installed) ):  
```console  
conda create --name purge_dups
cd /home/bioma/anaconda3/envs/purge_dups  
git clone https://github.com/dfguan/purge_dups.git  
cd purge_dups/src && make  
cd ../..
git clone https://github.com/dfguan/runner.git  
cd runner && python3 setup.py install --user  
cd ..  
git clone https://github.com/dfguan/KMC.git  
cd KMC && make -j 16  
conda install -c bioconda minimap2
```
## 2. Dataset  
For this tutorial, we will be using the genome assembly of the butterfly species [*Pieris rapae*](https://en.wikipedia.org/wiki/Pieris_rapae), which was publicly released as part of the Darwin Tree of Life (DToL) [project](https://www.darwintreeoflife.org/). Three files need to be downloaded:    
* [20200120.hicanu.purge.prim.fasta.gz](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/assemblies/working/20200120.hicanu.purge/20200120.hicanu.purge.prim.fasta.gz): the primary assembly  
* [20200120.hicanu.purge.htig.fasta.gz](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/assemblies/working/20200120.hicanu.purge/20200120.hicanu.purge.htig.fasta.gz): the haplotigs, i.e., the variant forms of heterozygous regions represented in the primary assembly.  
* [m64016_191223_193312.ccs.bam](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/genomic_data/pacbio/m64016_191223_193312.ccs.bam): file containing the raw HiFi reads. It's [compatible](https://pacbiofileformats.readthedocs.io/en/3.0/BAM.html) with the traditional BAM extension used for representing reads mapping, however in this case it represents only the raw reads (no mapping). According to [PacBio](https://www.pacb.com/wp-content/uploads/3_DavidAlexander_SmrtDevMeeting.pdf), it can be thought as a better FASTQ format. 
    
## 3. Creating an artifially duplicated assembly  
* Since the original dataset has already had the duplicated regions purged, we first need to create an artificially duplicated assembly by merging the *primary assembly* and *haplotigs*: 

```console  
cat 20200120.hicanu.purge.prim.fasta.gz 20200120.hicanu.purge.htig.fasta.gz > 20200120.hicanu.unpurged.fasta.gz
```  

Then, the file created (20200120.hicanu.unpurged.fasta.gz) has, for every heterozygous region, contigs representing both haplotypes. This is the file we will be using from now on to test purge_dups. 

## 4. Converting HiFi reads to FASTA format  
Since purge_dups is not compatible with the BAM format, we need to convert our raw HiFi reads from BAM to FASTQ format. We will do it using the [*BAM2fastx*](https://github.com/PacificBiosciences/bam2fastx) tools by PacBio:  
```console  
bam2fasta -o m64016_191223_193312.ccs m64016_191223_193312.ccs.bam
```

## 5. Run minimap2 to align pacbio data and generate paf files  
Then we align the PacBio HiFi reads against the duplicated assembly using the program *minimap2*:  
```console  
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ minimap2 -xmap-pb ../../input/20200120.hicanu.unpurged.fasta.gz ../../input/m64016_191223_193312.ccs.fasta.gz | gzip -c - > PieRapa.paf.gz
```  

## 6. Calculate base-level depth and read depth histogram 
Then once the alignment is done, we will calculate the per base depth (PB.base.cov) and depth histogram (PB.stat) from it, using the function *pbcstat* that comes with *purge_dups*:

```console
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/pbcstat PieRapa.paf.gz
```  

## 7. Generate cutoff
```console
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/calcuts PB.stat > cutoffs 2>calcults.log
```  

## Verifying the cutoffs 
```console
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/scripts/hist_plot.py PB.stat hist_depth
```

# Linux commands  

