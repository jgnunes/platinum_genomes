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
For this tutorial, we will be using the genome assembly of the butterfly species [*Pieris rapae*](https://en.wikipedia.org/wiki/Pieris_rapae), which was publicly released as part of the Darwin Tree of Life (DToL) [project](https://www.darwintreeoflife.org/). Two files need to be downloaded for this tutorial:
    * [20200120.hicanu.purge.prim.fasta.gz](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/assemblies/working/20200120.hicanu.purge/20200120.hicanu.purge.prim.fasta.gz): the primary assembly
    * [20200120.hicanu.purge.htig.fasta.gz](https://darwin.cog.sanger.ac.uk/insects/Pieris_rapae/ilPieRapa1/assemblies/working/20200120.hicanu.purge/20200120.hicanu.purge.htig.fasta.gz): the alternate assembly, containing duplicated heterozygous regions that were purged from the primary assembly  
    


# Linux commands
