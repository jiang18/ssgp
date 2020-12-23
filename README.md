# Executables
https://drive.google.com/drive/folders/1SwDld404x6r4p3nIvf6g0VJ9Nh2ZRHtK?usp=sharing

# Simulation
The procedures below generate genotypes and phenotypes of 500K individuals. Modify the sample size if needed. 
## Simulate genotypes with [genosim](https://aipl.arsusda.gov/software/genosim/)
1. pedigree.file and genotype.data0
```console
foo@bar:~$ perl sim_ped.pl 500000
```
2. markersim.options
```
  50000      0      1.0         0           1
  loci     qtls    curvparm   correlated   traits

 96     30        .01       .99       0
seed  chromes  minfreq  maxfreq  QTLfreq
```
3. chips.txt
```
chip    reduce1   offset1    reduce2    offset2    depth1    error1
  1        1         0          1          0          35        .00
```
4. genosim.options
```
  1.0      .965      0
Morgans  LDparam  hapout

  30      30      .00      .000   .00000003
chromes Xchrome  notread   errate   mutate
```
```console
foo@bar:~$ ./markersim
foo@bar:~$ ./genosim
```
## Convert genosim output files to plink files
```console
foo@bar:~$ perl aipl2plink.pl 500k
foo@bar:~$ plink --file 500k --make-bed --out 500k --chr-set 30
```
## Simulate phenotypes based on genosim output files
```console
foo@bar:~$ mkdir pheno
foo@bar:~$ cd pheno
foo@bar:~$ perl sim_snp_effects.pl 1.snp.csv
foo@bar:~$ ssgp --pred --binary_genotype ../500k --snp_estimate 1.snp.csv --output 1.gv
```
```console
foo@bar:~$ Rscript --no-save sim_phe.R 1 0.3
```
# SSGP
Note that the simulation procedures above generate a sample of 500K while the commands below work for a sample of 10K.
## REML
```console
foo@bar:~$ mkdir ssgp
foo@bar:~$ cd ssgp
foo@bar:~$ ssgp --phenotype_file ../10k/pheno/1.ssgp.csv --binary_genotype_file ../10k/10k --trait QT --reml --snp_info_file snp_info.csv --out 1 --num_threads 10 --num_random_probes 30
```
## LMM and GWA
```console
foo@bar:~$ ssgp --phenotype_file ../10k/pheno/1.ssgp.csv --binary_genotype_file ../10k/10k --trait QT --lmm --snp_info_file snp_info.csv --out 1 --num_threads 10 --num_qf_markers 30
foo@bar:~$ OMP_NUM_THREADS=10 ssgp_gamma.py --pfile ../10k/10k --ssgp 1 --out 1.gamma.txt
foo@bar:~$ OMP_NUM_THREADS=10 ssgp_gwa.py --pfile ../10k/10k --ssgp 1 --out 1.chr1.txt --chr 1
```
# GCTA-fastGWA
# BOLT
```console
foo@bar:~$ mkdir bolt
foo@bar:~$ cd bolt
foo@bar:~$ ~/software/BOLT-LMM_v2.3.4/bolt \
  --lmmInfOnly --bed=../10k/10k.bed --bim=../10k/10k.bim --fam=../10k/10k.fam \
  --phenoFile=../10k/pheno/1.bolt.txt --phenoCol=QT \
  --numThreads=10 --Nautosomes=30 --LDscoresUseChip \
  --verboseStats --statsFile=10k.1.lmm.txt
```
 

