SNPsea: an algorithm to identify cell types, tissues, and pathways affected by risk loci
========================================================================================

| <http://www.broadinstitute.org/mpg/snpsea> | Reference Manual [HTML] |
|:---:|:---:|

[HTML]: http://www.broadinstitute.org/mpg/snpsea/SNPsea_manual.html

- <a href="#example">Example</a>
- <a href="#quick-start">Quick Start</a>
- <a href="#citation">Citation</a>
- <a href="#description">Description</a>


Cartoon
-------

![Example of SNPsea results.][cartoon]

[cartoon]: https://raw.github.com/slowkow/snpsea/master/doc/figures/cartoon.png

This cartoon illustrates the key ideas of the algorithm:

A.  Each SNP in a set of disease-associated SNPs is in linkage disequilibrium
    (LD) with multiple genes. The genes are scored, in aggregate, for
    specificity to each tissue.

B.  The procedure is repeated with random null SNP sets that are not
    associated with any phenotype.

C.  The random SNP sets form the null distribution which allows us to
    determine the statistical significance of enrichment for specificity to
    a particular tissue.


Example
-------

![Example of SNPsea results.][example]

[example]: https://raw.github.com/slowkow/snpsea/master/doc/figures/Red_blood_cell_count-Harst2012-45_SNPs-GeneAtlas2004-single-pvalues_barplot.png

We identified *BM-CD71+Early Erythroid* as the cell type with most significant
enrichment (P < 2e-7) for cell type-specific gene expression relative to 78
other tissues in the Gene Atlas ([Su *et al.* 2004][Su2004]).

We used SNPsea to test the genes in linkage disequilibrium (LD) with 45 SNPs
associated with red blood cell count (P <= 5e-8) in GWAS of Europeans ([Harst
*et al.* 2012][Harst2012]). For each cell type, we tested a maximum of 1e7
null SNP sets by sampling random LD pruned SNP sets and scoring them. Each
set contains random SNPs matched to the input SNPs on the number of genes in
LD.

[Harst2012]: http://www.ncbi.nlm.nih.gov/pubmed/23222517
[Su2004]: http://www.ncbi.nlm.nih.gov/pubmed/15075390


We ran SNPsea like this:

```bash
options=(
    --snps              Red_blood_cell_count-Harst2012-45_SNPs.gwas
    --gene-matrix       GeneAtlas2004.gct.gz
    --gene-intervals    NCBIgenes2013.bed.gz
    --snp-intervals     TGP2011.bed.gz
    --null-snps         Lango2010.txt.gz
    --out               out
    --slop              10e3
    --threads           8
    --null-snpsets      0
    --min-observations  100
    --max-iterations    1e7
)
snpsea ${options[*]}

# Time elapsed: 2 minutes 36 seconds

# Create the figure shown above:
snpsea-barplot out
```


Quick Start
-----------

**Linux 64 bit executable**: <https://github.com/slowkow/snpsea/releases>

**Data**: <http://dx.doi.org/10.6084/m9.figshare.871430>

On Linux, you can get started right away:

```bash
mkdir snpsea; cd snpsea
curl -LOk https://github.com/slowkow/snpsea/releases/download/v1.0.3/snpsea_v1.0.3.zip
unzip snpsea_v1.0.3.zip
curl -LOk http://files.figshare.com/1382662/SNPsea_data_20140212.zip
unzip SNPsea_data_20140212.zip
bash example.sh
```

- - -

On other platforms, you have to build the source code.

First, install the [dependencies]:

```bash
#       Install Python libraries.
pip install docopt numpy pandas matplotlib
#       Change the graphical backend for matplotlib.
perl -i -pe 's/^(\s*(backend).*)$/#$1\n$2:Agg/' ~/.matplotlib/matplotlibrc

#       Install R libraries.
R -e 'install.packages(c("data.table", "reshape2", "gap", "ggplot2"))'

#       Install C++ libraries.
#       Ubuntu
sudo apt-get update; sudo apt-get install build-essential libopenmpi-dev libgsl0-dev
#       Mac
sudo port selfupdate; sudo port install gcc48 openmpi gsl
#       Broad Institute
use .gcc-4.8.1 .openmpi-1.4 .gsl-1.14
```

Next, download and compile SNPsea:

```bash
git clone https://github.com/slowkow/snpsea.git
cd snpsea/src
make
```

[dependencies]: http://www.broadinstitute.org/mpg/snpsea/SNPsea_manual.html#c-libraries


Citation
--------

If you benefit from this method, please cite:

> Slowikowski, K. et al. **SNPsea: an algorithm to identify cell types,
> tissues, and pathways affected by risk loci.** Bioinformatics (2014).
> doi:[10.1093/bioinformatics/btu326][Slowikowski2014]

See additional information and examples here:

> Hu, X. et al. *Integrating autoimmune risk loci with gene-expression data
> identifies specific pathogenic immune cell subsets.* The American Journal
> of Human Genetics 89, 496–506 (2011). [PubMed][Hu2011]

[Hu2011]: http://www.ncbi.nlm.nih.gov/pubmed/21963258
[Slowikowski2014]: http://bioinformatics.oxfordjournals.org/content/early/2014/05/10/bioinformatics.btu326


Description
-----------

SNPsea is a general algorithm to identify cell types, tissues, and pathways
likely to be affected by risk loci. The required input is a list of SNP
identifiers and a matrix of genes and conditions.

Suppose we have a gene expression matrix with expression profiles for multiple
cell types. Our goal may be to determine if some alleles associated to a trait
contain genes that are important for the function of a particular cell type.

First, we identify the genes in linkage disequilibrium with the given
trait-associated SNPs and score them for specificity to each cell type. To
evaluate significance of the specificity, we calculate an exact permutation
p-value as follows. We take the sum of the specificity scores and compare it
to a null distribution that is defined by sampling random matched SNP sets.
They are matched to the original trait-associated SNPs on the number of linked
genes.

This implementation is generalized, so you may provide (1) a continuous gene
matrix with gene expression (or any other values) or (2) a binary gene matrix
with presence/absence 1/0 values. We provide two continuous matrices and one
binary matrix for you.

The columns of the matrix could be tissues, cell types, GO annotation codes,
or any other types of *conditions*. Continuous matrices *must* be normalized
before running SNPsea so that columns are directly comparable to each other.

This analysis is able to detect if there is an enrichment of
condition-specificity of the genes linked with trait-associated SNPs.

If trait-associated alleles influence a small number of pathogenic tissues or
cell types, we hypothesize that the subset of genes with critical functions in
those pathogenic cell types are likely to be within trait-associated loci.

We assume that a gene's specificity to a given cell type or condition is
a reasonable indicator of the gene's importance to its function.

