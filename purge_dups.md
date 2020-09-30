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
* [m64016_191223_193312.ccs.bam](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/genomic_data/pacbio/m64016_191223_193312.ccs.bam): file containing the raw HiFi reads. It's the traditional BAM extension used for representing reads mapping, however in this case it represents only the raw reads (no mapping). According to [PacBio](https://www.pacb.com/wp-content/uploads/3_DavidAlexander_SmrtDevMeeting.pdf), it can be thought as a better FASTQ format. 
    
## 3. Creating an artifially duplicated assembly  
* Since the original dataset has already had the duplicated regions purged, we first need to create an artificially duplicated assembly by merging the *primary* and *haplotigs*: 

```console  
cat 20200120.hicanu.purge.prim.fasta.gz 20200120.hicanu.purge.htig.fasta.gz > 20200120.hicanu.unpurged.fasta.gz
```  

Then, the file created (20200120.hicanu.unpurged.fasta.gz) has, for every heterozygous region, contigs representing both haplotypes. This is the file we will be using from now on to test purge_dups. 

## 4. Setting up a configuration file  
Before running purge_dups we need to create a configuration file, containing all the information that purge_dups needs to run the purging:  

```  
./scripts/pd_config.py -l iHelSar1.pri -s 10x.fofn -n config.iHelSar1.PB.asm1.json ~/vgp/release/insects/iHelSar1/iHlSar1.PB.asm1/iHelSar1.PB.asm1.fa.gz pb.fofn
```

# Linux commands
