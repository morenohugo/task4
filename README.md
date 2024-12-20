# Task4: Distribution of read depth (DP) qualities INDELS vs. SNPs

This script (`workflow.sh`) filters and processes variant data from a VCF file (`luscinia_vars.vcf.gz`) to examine the read depth (DP) distribution for INDELs and SNPs. The analysis focuses on two chromosomes: `chr1` and `chrZ`.

## Running workflow.sh
```bash
# go to the working directory
cd projects/unix-course-2024-2025-fix-morenohugo/

# make the workflow.sh executable
chmod +x workflow.sh

# Running the script
./workflow.sh
```

### Step-by-Step

1. **Filter VCF Data**
   
   Filter the VCF data to get only chromosome 1 and Z information, and remove the metadata (headers beginning with `##`).

   ```bash
   < /data-shared/vcf_examples/luscinia_vars.vcf.gz zcat | grep -v '^##' | tail -c+2 | grep -e 'chr1\s' -e 'chrZ\s' > results/data_filtered.tsv

3. **Define INPUT**
   
   Set data_filtered.tsv as the input for all other tests
   
   ```bash
   INPUT=results/data_filtered.tsv

5. **Check the data**
   
   Check if the subset data was properly created and only chromosome 1 and Z are present in $INPUT
   
   ```bash
   < $INPUT cut -f1 | uniq

7. **Extract the columns of interest**
   
   Extract the first six columns
   ```bash
   < $INPUT cut -f1-6 > results/first_six.tsv
   ```

   Extract the read depth (DP) from the info column
   ```bash
   < $INPUT egrep -o 'DP=[^;]*' | sed 's/DP=//' > results/DP.tsv
   ```
   
   Check if the lines are INDEL or SNP
   ```bash
   < $INPUT awk '{if($0 ~ /INDEL/) print "INDEL"; else print "SNP"}' > results/indel.tsv
   ```
   
8. **Check the data**
   
   Check if the three subsets (`first_six.tsv`, `DP.tsv`, `indel.tsv`)
   ```bash
   wc -l results/first_six.tsv results/DP.tsv results/indel.tsv
   ```

10. **Create the final output**
    
   Merge the three subsets and create the final output
   ```bash
   paste results/first_six.tsv results/DP.tsv results/indel.tsv > results/results.tsv
   echo "Output saved to results/results.tsv"
   ```

## data-analysis.R
1. **Import the required libraries**
```R
library(tidyverse)
library(ggplot2)
````

2. **Load the data**

Load the data in R with explicit column names
```R
d <- read_tsv("results/results.tsv", 
         col_names = c("CHROM", "POS", "ID", "REF", "ALT", "QUAL", "DEPTH", "TYPE")) 
```

3. **Plot the distribution of DP for SNPs and INDELs**

Since we are interested in the distribution of DP in SNPs vs INDELS, the most appropriate graph types are boxplots/violinplots and histograms

The **<ins>boxplot</ins>** shows the distribution of log-transformed DP values for INDELs and SNPs. The SNPs have a lower read depth (DP) median than the INDELs.

![Image of the boxplot](https://github.com/morenohugo/task4/blob/main/boxplot_DP_TYPE_log10.png)

```R
boxplot <- ggplot(d, aes(TYPE, DEPTH, fill = TYPE)) + 
  geom_boxplot() +
  labs(title = "Distribution of Read Depth (DP) for SNPs vs. INDELs",
       y = "log10(Read Depth (DP))") +
  theme(legend.position = "top") +
  theme_bw() + 
  scale_y_log10()

# Save the boxplot
ggsave("results/boxplot_DP_TYPE_log10.png", plot = boxplot)
```


The **<ins>violin plot</ins>** shows the distribution of DP values for INDELs and SNPs. 

![Image of the violinplot](https://github.com/morenohugo/task4/blob/main/violinplot_DP_TYPE_log10.png)

```R
violinplot <- ggplot(d, aes(TYPE, DEPTH, fill = TYPE)) + 
  geom_violin() +
  labs(title = "Distribution of Read Depth (DP) for SNPs vs. INDELs",
       y = "log10(Read Depth (DP))") + 
  theme(legend.position = "top") +
  theme_bw() + 
  scale_y_log10()

# Save the violin plot
ggsave("results/violinplot_DP_TYPE_log10.png", plot = violinplot)
```

The **<ins>histogram</ins>** shows the distribution of log-transformed DP values for INDELs and SNPs. More SNPs have a lower read depth (DP) when compared to the INDELs.

![Image of the histogram](https://github.com/morenohugo/task4/blob/main/histogram_DP_TYPE_density.png)

```R
histogram <- ggplot(d, aes(x = DEPTH, fill = TYPE)) +
  geom_histogram(alpha = 0.5, binwidth = 5, position = "identity", stat = "density") +
  labs(title = "Distribution of Read Depth (DP) for SNPs vs. INDELs",
       x = "Read Depth (DP)") +
  theme_bw() +
  theme(legend.position = "top")

# Save the histogram
ggsave("results/histogram_DP_TYPE_density.png", plot = histogram)
```

