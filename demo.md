# Toy data
[QTL-MAS 2012](https://figshare.com/articles/QTL-MAS-2012/12336866)

# Input files
- Genotypes
- Phenotypes
- Covariates
- SNP info (group and weight)

# How-to
```console
ssgp.avx2 --phenotype_file phen.csv --trait_name milk --snp_info_file snp_info.csv --snp_set_name group --binary_genotype_file geno --snp_scale_value 1 --output out --num_threads 2

```
