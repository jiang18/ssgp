# SSGP
## Simulate genotypes with [genosim](https://aipl.arsusda.gov/software/genosim/)
1. pedigree.file and genotype.data0
```perl
# sim_ped.pl

use strict;
use warnings;

@ARGV == 1 or die "One argument needed: sample size\n";
my ($np) = @ARGV;
my $st = 1000000;

open OUT,">pedigree.file";
for(1..$np){
        my $id = $_+$st;
        print OUT "F     $id        -1       -2 20000101  0.0    0\n";
}
close OUT;

open OUT,">genotype.data0";
print OUT "idnum  chip\n\n";
for(1..$np){
        my $id = $_+$st;
        print  OUT "    $id    1\n";
}
close OUT;
```
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
```perl
# aipl2plink.pl

use strict;
use warnings;

@ARGV == 1 or die "One argument needed: plink file prefix\n";
my ($plink) = @ARGV;

open IN,"genotypes.true";
open OUT,">$plink.ped";
my %gt = (0=>11, 1=>12, 2=>22);
while(<IN>)
{
        chomp;
        my @c = split /\s+/;
        my @g = split //,$c[-1];
        print OUT "0 $c[1] 0 0 2 0";
        for (@g) {
                print OUT ' ',$gt{$_};
        }
        print OUT "\n";
}
close IN;
close OUT;

open IN,"chromosome.data";
open OUT,">$plink.map";
$_=<IN>;
while(<IN>)
{
        chomp;
        my @c = split /\s+/;
        print OUT "$c[1]\t$c[0]\t0\t$c[4]\n";
}
close IN;
close OUT;
```
```console
foo@bar:~$ perl aipl2plink.pl 500k
foo@bar:~$ plink --file 500k --make-bed --out 500k --chr-set 30
```
## Simulate phenotypes based on genosim output files
```perl
# sim_snp_effects.pl

use strict;
use warnings;

my @snps;
my %freq;
open IN,"../true.frequency";
while(<IN>)
{
        chomp;
        my @c = split /\s+/;
        push @snps, $c[0];
        $freq{$c[0]} = $c[2];
}
close IN;

my $PI = 2 * atan2 1, 0;
my @nums = map {
    sqrt(-2 * log rand) * cos(2 * $PI * rand)
} 1..@snps;

@ARGV == 1 or die "One argument needed: output filename\n";
my ($out) = @ARGV;

open OUT,">$out";
print OUT "SNP,Chr,Pos,Allele1,Allele2,MAF,HWE_Pval,Group,Weight,Effect\n";

for(0..$#snps) {
        my $f = $freq{$snps[$_]};
        my $e = $nums[$_]/sqrt(2*$f*(1-$f));
        print OUT "$snps[$_],0,0,1,2,$f,0,NULL,1,$e\n";
}
close OUT;
```
```console
foo@bar:~$ mkdir pheno
foo@bar:~$ cd pheno
foo@bar:~$ mkdir snp
foo@bar:~$ mkdir gv
foo@bar:~$ perl sim_snp_effects.pl ./snp/1.csv
foo@bar:~$ ssgp --pred --binary_genotype ../500k --snp_estimate ./snp/1.csv --output ./gv/1
```
```R
# sim_phe.R

args <- commandArgs(trailingOnly=TRUE)
ii = args[1]
hsq = as.numeric(args[2])
print(paste("h2=",hsq,sep=""))

# sample size
n = 500000

gv = read.table(paste("gv/",ii,".csv",sep=""),sep=",")
vg = var(gv[,2])
ve = vg*(1-hsq)/hsq
print(paste("VarG=",vg,sep=""))

err = rnorm(n) * sqrt(ve)
phe = matrix(nrow=n, ncol=3)
phe[,1] = 0
phe[,2] = gv[,1]
phe[,3] = gv[,2] + err
print(paste("VarP=",var(phe[,3]),sep=""))

colnames(phe) = c("FID","IID","QT")
write.table(phe,paste(ii,".bolt.txt",sep=""),row.names = FALSE, quote=FALSE)
phe = phe[,2:ncol(phe)]
write.csv(phe,paste(ii,".ssgp.csv",sep=""),row.names = FALSE, quote=FALSE)
```
```console
foo@bar:~$ Rscript --no-save sim_phe.R 1 0.25
```
