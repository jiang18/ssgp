# SSGP
## Simulate genotypes and phenotypes with [genosim](https://aipl.arsusda.gov/software/genosim/)
1. pedigree.file and genotype.data0
```perl
# sim_ped.pl

use strict;
use warnings;

my $st = 1000000;
my $np = 500000;

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
2. markersim.options
```
  60000     10000      1.0         0          100
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
## Convert genosim output files to plink files
```perl
# aipl2plink.pl

use strict;
use warnings;

open IN,"genotypes.true";
open OUT,">500k.ped";

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
open OUT,">500k.map";
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
```
plink --file 500k --make-bed --out 500k --chr-set 30
```
