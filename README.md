# SSGP
## Simulate genotypes and phenotypes with [genosim](https://aipl.arsusda.gov/software/genosim/)
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
## Simulate phenotypes based on genosim output
```R
# sim_phe.R

lines =  readLines("genomic.bv")
oddlines = lines[seq (1, length(lines),2)]
dat = strsplit(oddlines, split = " +")
phe = matrix(unlist(dat), ncol = 102, byrow = TRUE)
phe[,1] = 0
colnames(phe) = c("FID","IID","N",1:99)

vg = apply(phe[,4:102],2,var)
hsq = c(rep(0.05,33), rep(0.25,33), rep(0.55,33))
ve = vg*(1-hsq)/hsq

err = matrix(rnorm(500000*99), nrow = 500000) %*% diag(sqrt(ve))
phe[,4:102] = apply(phe[,4:102],2, as.numeric) + err

write.table(phe,"500k.bolt.txt",row.names = FALSE, quote=FALSE)
phe = phe[,2:ncol(phe)]
write.csv(phe,"500k.ssgp.csv",row.names = FALSE, quote=FALSE)
```

