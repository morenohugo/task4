# Task4: Distribution of read depth (DP) qualities INDELS vs. SNPs

This script (`task4.sh`) filters and processes variant data from a VCF file to examine the read depth (DP) distribution for INDELs and SNPs. The analysis focuses on two chromosomes: `chr1` and `chrZ`.

## Requirements

- **Data**: A compressed VCF file (`luscinia_vars.vcf.gz`) in the `/data-shared/vcf_examples/` directory.
- **Tools**: Bash, `zcat`, `grep`, `cut`, `egrep`, `sed`, `awk`, `paste`, and `wc`.

## Usage
cd ...
chmod +x workflow.sh
./workflow.sh

### Step-by-Step

1. **Filter VCF Data**:
   The script filters for records from chromosomes `chr1` and `chrZ` and removes headers beginning with `##`.

   ```bash
   < /data-shared/vcf_examples/luscinia_vars.vcf.gz zcat | grep -v '^##' | tail -c+2 | grep -e 'chr1\s' -e 'chrZ\s' > data_filtered.tsv

