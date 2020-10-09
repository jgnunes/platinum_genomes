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
Once we have the depth histogram, containing distribution of each value of read depth coverage, we can use the *calcuts* function (that comes with *purge_dups*) to automatically set three cutoffs:  
1. A low cutoff that will be used to identify contigs that are likely to be assembly artefacts  
2. A medium cutoff that will be used to separate diploid and haploid coverage depths  
3. A high cutoff that will be used to identify contigs that are likely to be collapsed repeats 

```console
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/calcuts PB.stat > cutoffs 2>calcults.log
```  

## 8. Inspect automatic cutoff  
It is good practice to inspect the cutoffs that were automatically defined by *purge_dups*. You can do that by analyzing the plot generated with the script *hist_plot.py*:  

```console  
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/scripts/hist_plot.py -c cutoffs PB.stat PB.cov.png
```

<img src="https://user-images.githubusercontent.com/22843614/94845666-623dfd80-03f6-11eb-8850-6e175bea773b.png" width=90%></img>  

In an assembly that has both haploid and diploid sequences, we should have a peak with higher coverage and another peak with approximately 50% read-depth as the first, representing the diploid sequences (since half of the reads will map to one haplotype and the other half to the other haplotype). However, our plot only seems to have one peak, and the question that remains is if this peak represents an assembly that is entirely diploid or entirenly haploid. This is not very relevant for us to place the low and high cutoffs, but it is for the middle cutoff, since it should separate the haploid and diploid depths. In other words, the middle cutoff should be placed so that what is to its left represents the diploid peak, that has an 0.5X depth, and what is to its right represents the haploid peak, that has an 1.0X depth. This is confirmed by [this github answer](https://github.com/dfguan/purge_dups/issues/27), where the developer of *purge_dups* gives similar interpretation to the same situation.  

One way to decide if our only peak represents the diploid or haploid peak is to compare its coverage with the expected coverage, by dividing the number of bases that were sequenced by the size of *Pieris rapae*'s genome. We can check the number of sequenced bases by inspecting the PacBio FASTA sequence with the command:  

```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/input $ zcat m64016_191223_193312.ccs.fasta.gz | grep -v ">" | awk '{x+=length($0)}END{print x}'
```
Which returns 14576396674 bases, i.e., approximately 14 billion bases!  

We can find out the size of *Pieris rapae*'s genome by looking at previous genome papers from this species, like [this](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5247789/), which gives us a genome size of 246 Mbp for a haploid assembly. 

Then we divide the number of sequenced bases by the size of the genome:  
```  
expected coverage = 14576396674/246000000 = 59X
```

I.e, the expected coverage is 59X for an haploid assembly. Since we known from our plot that our peak is around 30X, it likely represents diploid sequences. This is exactly what expected, since our assembly was obtained after merging the two haploid assemblies. However, we now know that our middle cutoff needs to be shifted right, in order to place our peak mostly to its left (remember, the left peak represents the 0.5X depth). 


## 9. Adjusting cutoffs  
In order to adjust our cutoffs, we will run the *calcuts* function again, but now with different options:  

```console  
bioma@bioma-XPS-8300:~/joao_ferreira/platinum_genome_training/purge_dups/manual$ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/calcuts -l5 -m45 -u63 P.stat > cutoffs_adjusted
```  

Notice that we have only changed the position of the middle cutoff. We are now placing it at 45X coverage, because it represents the approximate middle point between the haploid coverage (59X, here rounded to 60X for easiness reasons) and the diploid coverage (30X):  

<img src="https://user-images.githubusercontent.com/22843614/94849612-0f674480-03fc-11eb-8afe-ba881851d2c3.png" widt=90%></img>

## 10. Split assembly  
Now we will split our assembly into contigs by cutting at blocks of 'N's:  
```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/split_fa ../../input/20200120.hicanu.unpurged.fasta > 20200120.hicanu.unpurged.split
```  

## 11. Do a sel-self alignment  
Then we will use minimap2 to align the assembly contigs against themselves:  

```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ minimap2 -xasm5 -DP 20200120.hicanu.unpurged.split 20200120.hicanu.unpurged.split | gzip -c - > 20200120.hicanu.unpurged.split.self.paf.gz
```  

## 12. Purge haplotigs and overlaps  
Then we will use the function *purge_dups* to purge the unproperly duplicated regions from the original assembly. In order to do that we need to input some information to *purge_dups*: i) the read depth cutoffs set manually (*cutoffs_adjusted*), ii) the read depth per base in the original assembly (*PB.base.cov*), and iii) the self-self alignment of contigs of the original assembly (*20200120.hicanu.unpurged.split.self.paf.gz*):

```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/purge_dups -2 -T cutoffs_adjusted -c PB.base.cov 20200120.hicanu.unpurged.split.self.paf.gz > dups.bed 2> purge_dups.log
```  

## 13. Retrieving the primary assembly and haplotig sequences from the original assembly  
The main file generated by the previous step is *dups.bed*, which contains all the information describing what are the duplicated regions to be purged. Once we have this information and the original assembly, we can use the *get_seqs* function to create the final purged assembly (*purged.fa*), as well as the assembly containing the haplotigs that were separated from the original assembly (*hap.fa*).  

However, beware if the sequence headers of your original assembly FASTA file have spaces and if that is the case, remove the space and every character that follows it. For instance, if the headers were ">some_random_name_00X EXTRAINFO" turn all of them into something like ">some_random_name_00X". This is because for some reason those spaces may cause *get_seqs* function to not properly recognize the target sequences and end up not purging sequences from the original file even when the *dups.bed* instructs to do so. Then you will end up with a *purge.fa* file which is exactly like the original assembly and an empty *hap.fa* file. 

```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ /home/bioma/anaconda3/envs/purge_dups/purge_dups/bin/get_seqs dups.bed 20200120.hicanu.unpurged.split
```  

*get_seqs* execution has generated a *purged.fa* file of 245 MB and a *hap.fa* file of 257 MB, i.e, with comparable sizes. Since the assembly file that was processed by *purge_dups* is the result of artificially merging two different haploid assemblies, that should be expected.   

```console  
257M out  2 18:16 hap.fa
```  

## 14. Validating the purged assembly  
We can use different tools to validate the final purged assembly in order to check if it is clean enough or even if it was overpurged. Our choice here is going to be [KMC](https://github.com/dfguan/KMC), which does a k-mer analysis of our final assembly.  

The first step is to create a folder where we will store the KMC results:  
```console    
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ mkdir kmc 
```  
Then inside that folder we will create a temporary file that will be used for KMC to store temporary files necessary for KMC calculations:  
```console    
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual $ cd kmc  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ mkdir tmp
```
Then we will finally run the KMC analysis:
```console    
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ kmc -k21 -ci0 -fm -sm ../purged.fa purged tmp  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ kmc -k21 -ci0 -sm ../../../input/m64016_191223_193312.ccs.fastq.gz m64016_191223_193312.ccs tmp
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ kmc_tools analyze m64016_191223_193312.ccs purged purged.matrix  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ ~/anaconda3/envs/purge_dups/KMC/spectra.py purged.matrix purged.png
```  
In the first command we calculated the k-mer profile for the *purged.fa* assembly, which was also calculated for the input *Hi-Fi* reads in the second command. In the third command we analyze the k-mer profiles of one against the other, and finally in the last command we use a python script to plot the analysis, generating a final plot image in PNG format.  

It will also be useful to plot a KMC analysis for the original assembly that was purged (*20200120.hicanu.unpurged.fasta*), in order to evaluate what was the actual work done with *purge_dups*:  
```console  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ kmc -k21 -ci0 -fm -sm ../../../input/20200120.hicanu.unpurged.fasta 20200120.hicanu.unpurged tmp
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ kmc_tools analyze m64016_191223_193312.ccs 20200120.hicanu.unpurged 20200120.hicanu.unpurged.matrix  
bioma@bioma-XPS-8300 ~/joao_ferreira/platinum_genome_training/purge_dups/manual/kmc $ python ~/anaconda3/envs/purge_dups/KMC/spectra.py 20200120.hicanu.unpurged.matrix 20200120.hicanu.unpurged.png
```  
The KMC plot for the original assembly exhibits a 1X representing k-mers that were only found in one place of the genome, likely representing heterozygous regions. The 2X represents k-mers found in two places of the genome, representing those sequences that are equal in both haplotypes (and therefore present in two copies in our diploid assembly). There are even k-mers found three times in the assembly (3X), which may represent erroneous duplications present in the assembly.  

<img src="https://user-images.githubusercontent.com/22843614/95625348-bb3d0f80-0a4e-11eb-8195-d4db0c26edb8.png" width=70%></img>  

The KMC plot for the purged assembly exhibits significantly different pattern. First there is no 2X curve, indicating that our assembly was properly de-duplicated by *purge_dups*. Since *purge_dups* has removed one of the haplotype copies from the assembly, when we try to map this purged sequence, we obviously cannot find it in the assembly any longer, what explains the 0X curve. Regarding the 1X curve, it has two peaks: the first one has lower coverage and represents heterozygous regions. The decrease in the coverage is explained by the fact that the divergent variants that were purged from the assembly obviously did not map to it. The second peak however represents homozygous regions that had twice the coverage of the heterozygous ones since all their chromosome copies will map perfectly to the assembly:  

<img src="https://user-images.githubusercontent.com/22843614/95626591-f17b8e80-0a50-11eb-8aca-c62c40f47838.png" width=70%></img> 

## 15. Re-run all *purge_dups* steps with *hap.fa* as input to get a decent haplotig set

It is good practice to run one more *purge_dups* execution for the haplotig assembly (the file *hap.fa*) in order to get a decent haplotig set. That means re-running all the steps replacing the original assembly (*20200120.hicanu.unpurged.fasta*) by *hap.fa*. This is a [recommendation](https://github.com/dfguan/purge_dups#step-4-merge-hapfa-and-hap_asm-and-redo-the-above-steps-to-get-a-decent-haplotig-set) from *purge_dups* developer, Dengfeng Guan, however due to time constraints we will not execute it in this tutorial.  
