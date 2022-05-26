# (PART\*) Chapter 3 Environmental <br>Health Database Mining {-}

# 3.1 Comparative Toxicogenomics Database

This training module was developed by Lauren E. Koval, Dr. Kyle R. Roell, and Dr. Julia E. Rager

Fall 2021








#### Background on Training Module


#### Introduction to Comparative Toxicogenomics Database (CTD)
CTD is a publicly available, online database that provides manually curated information about chemical-gene/protein interactions, chemical-disease and gene-disease relationships. CTD also recently incorporated curation of exposure data and chemical-phenotype relationships.

CTD is located at: http://ctdbase.org/. Here is a screenshot of the CTD homepage (as of August 5, 2021):
<img src="_book/TAME_Toolkit_files/figure-html/Module3_1_CTD_homepage.jpg" width="750" style="display: block; margin: auto;" />



## Introduction to Training Module
In this activity, we will be using CTD to access and download data to perform data organization and analysis as an applications-based example towards environmental health research. This activity represents a demonstration of basic data manipulation, filtering, and organization steps in R, while highlighting the utility of CTD to identify novel genomic/epigenomic relationships to environmental exposures. Example visualizations are also included in this training module's script, providing visualizations of gene list comparison results.



#### Training Module's **Environmental Health Questions**
This training module was specifically developed to answer the following environmental health questions:

(1) Which genes show altered expression in response to arsenic exposure?
(2) Of the genes showing altered expression, which may be under epigenetic control?



#### Script Preparations



#### Cleaning the global environment

```r
rm(list=ls())
```



#### Installing required R packages
If you already have these packages installed, you can skip this step, or you can run the below code which checks installation status for you.

```r
if (!requireNamespace("tidyverse"))
  install.packages("tidyverse")
if (!requireNamespace("VennDiagram"))
install.packages("VennDiagram")
if (!requireNamespace("grid"))
install.packages("grid")
```



#### Loading R packages required for this session

```r
library(tidyverse)
library(VennDiagram)
library(grid)
```



#### Set your working directory

```r
setwd("/filepath to where your input files are")
```

 

## CTD Data in R

#### Organizing Example Dataset from CTD

CTD requires manual querying of its database, outside of the R scripting environment. Because of this, let's first manually pull the data we need for this example analysis. We can answer both of the example questions by pulling all chemical-gene relationship data for arsenic, which we can do by following the below steps:

Navigate to the main CTD website: http://ctdbase.org/.

  
Select at the top, 'Search' -> 'Chemical-Gene Interactions'. 
<br>
<img src="_book/TAME_Toolkit_files/figure-html/Module3_1_CTD_Step1.jpg" width="630" style="display: block; margin: auto;" />

<br>
  
Select to query all chemical-gene interaction data for arsenic.
<br>
<img src="_book/TAME_Toolkit_files/figure-html/Module3_1_CTD_Step2.jpg" width="600" style="display: block; margin: auto;" />
<br>

  
Note that there are lots of results, represented by many many rows of data! Scroll to the bottom of the webpage and select to download as 'CSV'.
<br>
<img src="_book/TAME_Toolkit_files/figure-html/Module3_1_CTD_Step3.jpg" width="600" style="display: block; margin: auto;" />
  
<br>
    
This is the file that we can now use to import into the R environment and analyze!
Note that the data pulled here represent data available on August 1, 2021



#### Loading the Example CTD Dataset into R



#### Read in the csv file of the results from CTD query

```r
ctd = read_csv("Module3_1/Module3_1_CTDOutput_ArsenicGene_Interactions.csv")
```



#### Data Viewing



Let's first see how many rows and columns of data this file contains:

```r
dim(ctd)
```

```
## [1] 6280    9
```
This dataset includes 6280 observations (represented by rows) linking arsenic exposure to gene-level alterations
With information spanning across 9 columns



Let's also see what kind of data are organized within the columns:

```r
colnames(ctd)
```

```
## [1] "Chemical Name"       "Chemical ID"         "CAS RN"             
## [4] "Gene Symbol"         "Gene ID"             "Interaction"        
## [7] "Interaction Actions" "Reference Count"     "Organism Count"
```



```r
# Viewing the first five rows of data, across all 9 columns
ctd[1:9,1:5] 
```

```
## # A tibble: 9 × 5
##   `Chemical Name` `Chemical ID` `CAS RN`  `Gene Symbol` `Gene ID`
##   <chr>           <chr>         <chr>     <chr>             <dbl>
## 1 Arsenic         D001151       7440-38-2 AACSP1           729522
## 2 Arsenic         D001151       7440-38-2 AADACL2          344752
## 3 Arsenic         D001151       7440-38-2 AAGAB             79719
## 4 Arsenic         D001151       7440-38-2 AAK1              22848
## 5 Arsenic         D001151       7440-38-2 AAMDC             28971
## 6 Arsenic         D001151       7440-38-2 AAR2              25980
## 7 Arsenic         D001151       7440-38-2 AASS              10157
## 8 Arsenic         D001151       7440-38-2 ABCA1                19
## 9 Arsenic         D001151       7440-38-2 ABCA12            26154
```




#### Filtering Data for Genes with Altered Expression



To identify genes with altered expression in association with arsenic, we can leverage the results of our CTD query and filter this dataset to include only the rows that contain the term "expression" in the "Interaction Actions" column.

```r
exp_filt = ctd %>% filter(grepl("expression", `Interaction Actions`))
```

We now have 2586 observations, representing instances of arsenic exposure causing a changes in a target gene's expression levels.

```r
dim(exp_filt)
```

```
## [1] 2586    9
```



Let's see how many unique genes this represents:

```r
length(unique(exp_filt$`Gene Symbol`))
```

```
## [1] 1878
```
This reflects 1878 unique genes that show altered expression in association with arsenic.



Let's make a separate dataframe that includes only the unique genes, based on the "Gene Symbol" column.

```r
exp_genes = exp_filt %>% distinct(`Gene Symbol`, .keep_all=TRUE)

# Removing columns besides gene identifier
exp_genes = exp_genes[,4] 

# Viewing the first 10 genes listed
exp_genes[1:10,] 
```

```
## # A tibble: 10 × 1
##    `Gene Symbol`
##    <chr>        
##  1 AADACL2      
##  2 AAK1         
##  3 AASS         
##  4 ABCA12       
##  5 ABCC1        
##  6 ABCC2        
##  7 ABCC3        
##  8 ABCC4        
##  9 ABCG4        
## 10 ABHD12B
```
This now provides us a list of 1878 genes showing altered expression in association with arsenic.


##### Technical notes on running the distinct function within tidyverse:
By default, the distinct function keeps the first instance of a duplicated value. This does have implications if the rest of the values in the rows differ. You will only retain the data associated with the first instance of the duplicated value (which is why we just retained the gene column here). It may be useful to first find the rows with the duplicate value and verify that results are as you would expect before removing observations. For example, in this dataset, expression levels can increase or decrease. If you were looking for just increases in expression, and there were genes that showed increased and decreased expression across different samples, using the distinct function just on "Gene Symbol" would not give you the results you wanted. If the first instance of the gene symbol noted decreased expression, that gene would not be returned in the results even though it might be one you would want. For this example case, we only care about expression change, regardless of direction, so this is not an issue. The distinct function can also take multiple columns to consider jointly as the value to check for duplicates if you are concerned about this.

<br>

<!-- #### With this, we can answer **Environmental Health Question #1**: -->
<!-- #### (1) Which genes show altered expression in response to arsenic exposure? -->
<!-- #### *Answer: This list of 1878 genes have published evidence supporting their altered expression levels associated with arsenic exposure.* -->

:::question
<i>With this, we can answer **Environmental Health Question 1**:</i>
Which genes show altered expression in response to arsenic exposure?
:::
:::answer
**Answer**: This list of 1878 genes have published evidence supporting their altered expression levels associated with arsenic exposure.
:::

<br>

## Identifying Genes  under Epigenetic Control



For this dataset, let's focus on gene-level methylation as a marker of epigenetic regulation. Let's return to our main dataframe, representing the results of the CTD query, and filter these results for only the rows that contain the term "methylation" in the "Interaction Actions" column.

```r
met_filt = ctd %>% filter(grepl("methylation",`Interaction Actions`))
```

We now have 3211 observations, representing instances of arsenic exposure causing a changes in a target gene's methylation levels.

```r
dim(met_filt)
```

```
## [1] 3211    9
```


Let's see how many unique genes this represents.

```r
length(unique(met_filt$`Gene Symbol`))
```

```
## [1] 3142
```
This reflects 3142 unique genes that show altered methylation in association with arsenic



Let's make a separate dataframe that includes only the unique genes, based on the "Gene Symbol" column.

```r
met_genes = met_filt %>% distinct(`Gene Symbol`, .keep_all=TRUE)

# Removing columns besides gene identifier
met_genes = met_genes[,4] 
```
This now provides us a list of 3142 genes showing altered methylation in association with arsenic.



With this list of genes with altered methylation, we can now compare it to previous list of genes with altered expression to yeild our final list of genes of interest. To achieve this last step, we present two different methods to carry out list comparisons below.



#### Method 1 for list comparisons: Merging



Merge the expression results with the methylation resuts on the Gene Symbol column found in both datasets.

```r
merge_df = merge(exp_genes, met_genes, by = "Gene Symbol")
```
We end up with 315 rows reflecting the 315 genes that show altered expression and altered methylation

Let's view these genes:

```r
merge_df[1:315,]
```

```
##   [1] "ABCC4"     "ABHD17A"   "ABLIM2"    "ACAD9"     "ACKR2"     "ACP3"     
##   [7] "ADAMTS1"   "AFF1"      "AGO2"      "ALDH3B2"   "ANPEP"     "AOPEP"    
##  [13] "AP3D1"     "APBB3"     "APP"       "AQP1"      "ARF1"      "ARID5B"   
##  [19] "AS3MT"     "ASAP1"     "ATF2"      "ATG7"      "ATP6V1C2"  "ATXN1"    
##  [25] "ATXN7"     "BACH1"     "BCAR1"     "BCL2"      "BCL6"      "BDNF"     
##  [31] "BECN1"     "BMI1"      "BMPR1A"    "C1GALT1C1" "C1S"       "C2CD3"    
##  [37] "CAMP"      "CARD18"    "CASP8"     "CASTOR1"   "CBR4"      "CBS"      
##  [43] "CCDC68"    "CCL14"     "CCL20"     "CCL24"     "CCR2"      "CD2"      
##  [49] "CD27"      "CD40"      "CDC42"     "CDH1"      "CDK2"      "CDK4"     
##  [55] "CDK5"      "CDK6"      "CDKN1B"    "CDKN2A"    "CELF1"     "CENPM"    
##  [61] "CEP72"     "CERK"      "CES4A"     "CFAP300"   "CHORDC1"   "CLEC4D"   
##  [67] "CLIC5"     "CMBL"      "CNTNAP2"   "CRCP"      "CREBBP"    "CUX2"     
##  [73] "CYP1B1"    "CYP26B1"   "CYP2U1"    "DAPK1"     "DAXX"      "DCAF7"    
##  [79] "DDB2"      "DHX32"     "DLK1"      "DNMT1"     "DSG1"      "DYNC2I2"  
##  [85] "ECHS1"     "EDAR"      "EFCAB2"    "EHMT2"     "EML2"      "EPHA1"    
##  [91] "EPHA2"     "EPM2AIP1"  "ERBB4"     "ERCC2"     "ERN2"      "ESR1"     
##  [97] "ETFB"      "ETFDH"     "F3"        "FAM25A"    "FAM43A"    "FAM50B"   
## [103] "FAM53C"    "FAS"       "FBLN2"     "FBXO32"    "FGF2"      "FGFR3"    
## [109] "FGGY"      "FOSB"      "FPR2"      "FTH1P3"    "FTL"       "GAK"      
## [115] "GAS1"      "GFRA1"     "GGACT"     "GLI2"      "GLI3"      "GNPDA1"   
## [121] "GOLGA4"    "GSTM3"     "GTSE1"     "H2AC6"     "H6PD"      "HAPLN2"   
## [127] "HCRT"      "HDAC4"     "HGF"       "HLA-DQA1"  "HOTAIR"    "HSD17B2"  
## [133] "HSPA1B"    "HSPA1L"    "HYAL1"     "IER3"      "IFNAR2"    "IFNG"     
## [139] "IGF1"      "IKBKB"     "IL10"      "IL16"      "IL1R1"     "IL1RAP"   
## [145] "IL20RA"    "INPP4B"    "IRF1"      "ITGA8"     "ITGAM"     "ITGB1"    
## [151] "JMJD6"     "JUP"       "KCNQ1"     "KEAP1"     "KLC1"      "KLHL21"   
## [157] "KRT1"      "KRT18"     "KRT27"     "LAMB1"     "LCE2B"     "LEPR"     
## [163] "LGALS7"    "LMF1"      "LMNA"      "LRP8"      "LRRC20"    "MALAT1"   
## [169] "MAOA"      "MAP2"      "MAP2K6"    "MAP3K8"    "MAPT"      "MARVELD3" 
## [175] "MBNL2"     "MEF2C"     "MEG3"      "MGMT"      "MICB"      "MLC1"     
## [181] "MLH1"      "MMP19"     "MOSMO"     "MPG"       "MRAP"      "MSH2"     
## [187] "MSI2"      "MT1M"      "MTOR"      "MUC1"      "MYH14"     "MYL9"     
## [193] "MYRIP"     "NCL"       "NEBL"      "NEDD4"     "NES"       "NEU1"     
## [199] "NFE2L2"    "NLRP3"     "NOS2"      "NPM1"      "NRF1"      "NRG1"     
## [205] "NRP2"      "NTM"       "NUAK2"     "NUP62CL"   "OASL"      "OCLN"     
## [211] "OSBPL5"    "PALS1"     "PCSK6"     "PDZD2"     "PECAM1"    "PFKFB3"   
## [217] "PGAP2"     "PGK1"      "PIAS1"     "PLA2G4D"   "PLCD1"     "PLEC"     
## [223] "PLEKHA6"   "PLEKHG3"   "PPFIA4"    "PPFIBP2"   "PPTC7"     "PRDX1"    
## [229] "PRKCQ"     "PRMT6"     "PRR5L"     "PRSS3"     "PTGS2"     "PTPRE"    
## [235] "PVT1"      "PYROXD2"   "RAB11FIP3" "RAMP1"     "RAP1GAP2"  "RAPGEF1"  
## [241] "RASAL2"    "RELCH"     "RGMA"      "RHEBL1"    "RHOH"      "RIPOR1"   
## [247] "RNF213"    "RNF216"    "ROBO1"     "S100P"     "S1PR1"     "SBF1"     
## [253] "SBNO2"     "SCGB3A1"   "SCHIP1"    "SELENOW"   "SEMA5B"    "SGMS1"    
## [259] "SH2B2"     "SKP2"      "SLC22A5"   "SLC44A2"   "SLC6A6"    "SNCA"     
## [265] "SNHG32"    "SNX1"      "SORL1"     "SPHK1"     "SPINK1"    "SPSB1"    
## [271] "SPTBN1"    "SQSTM1"    "SRGAP1"    "SSU72"     "STAT3"     "STK17B"   
## [277] "STX1A"     "STX3"      "SULT2B1"   "TCEA3"     "TERT"      "TGFB1"    
## [283] "TGFB3"     "TGFBR2"    "THNSL2"    "TIMP2"     "TLR10"     "TMEM86A"  
## [289] "TNFRSF10B" "TNFRSF10D" "TNFRSF1B"  "TNFSF10"   "TNNC2"     "TP53"     
## [295] "TRIB1"     "TRNP1"     "TSC22D3"   "TSLP"      "TXNRD1"    "UAP1"     
## [301] "UBE2J2"    "ULK1"      "USP36"     "VAV3"      "VWF"       "WDR26"    
## [307] "WDR55"     "WNK1"      "WWTR1"     "XDH"       "ZBTB25"    "ZEB1"     
## [313] "ZNF200"    "ZNF267"    "ZNF696"
```



<!-- #### With this, we can answer **Environmental Health Question #2**: -->
<!-- #### (2) Of the genes showing altered expression, which may be under epigenetic control? -->
<!-- #### *Answer: We identified 315 genes with altered expression resulting from arsenic exposure, that also demonstrate epigenetic modifications from arsenic. These genes include many high interest molecules involved in regulating cell health, including several cyclin dependent kinases (e.g., CDK2, CDK4, CDK5, CDK6), molecules involved in oxidative stress (e.g., FOSB, NOS2), and cytokines involved in inflammatory response pathways (e.g., IFNG, IL10, IL16, IL1R1, IR1RAP, TGFB1, TGFB3).* -->

:::question
<i>With this, we can answer **Environmental Health Question 2**:</i>
Of the genes showing altered expression, which may be under epigenetic control?
:::
:::answer
**Answer**:  We identified 315 genes with altered expression resulting from arsenic exposure, that also demonstrate epigenetic modifications from arsenic. These genes include many high interest molecules involved in regulating cell health, including several cyclin dependent kinases (e.g., CDK2, CDK4, CDK5, CDK6), molecules involved in oxidative stress (e.g., FOSB, NOS2), and cytokines involved in inflammatory response pathways (e.g., IFNG, IL10, IL16, IL1R1, IR1RAP, TGFB1, TGFB3).
:::



#### Method 2 for list comparisons: Intersection
For further training, shown here is another method for pulling this list of interest, through the use of the 'intersection' function.



Obtain a list of the overlapping genes in the overall expression results and the methylation results.

```r
inxn = intersect(exp_filt$`Gene Symbol`,met_filt$`Gene Symbol`)
```
Again, we end up with a list of 315 unique genes that show altered expression and altered methylation.



This list can be viewed on its own or converted to a dataframe (df).

```r
inxn_df = data.frame(genes=inxn)
```



This list can also be conveniently used to filter the original query results.

```r
inxn_df_all_data = ctd %>% filter(`Gene Symbol` %in% inxn)
```



Note that in this last case, the same 315 genes are present, but this time the results contain all records from the original query results, hence the 875 rows (875 records observations reflecting the 315 genes).

```r
summary(unique(sort(inxn_df_all_data$`Gene Symbol`))==sort(merge_df$`Gene Symbol`))
```

```
##    Mode    TRUE 
## logical     315
```

```r
dim(inxn_df_all_data)
```

```
## [1] 875   9
```


Visually we can represent this as a Venn diagram. Here, we use the ["VennDiagram"](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-12-35) R package. 


```r
# Use the data we previously used for intersection in the venn diagram function
venn.plt = venn.diagram(
  x = list(exp_filt$`Gene Symbol`, met_filt$`Gene Symbol`),
  category.names = c("Altered Expression" , "Altered Methylation"),
  filename = "venn_diagram.png",
  
  # Change font size, type, and position
  cat.cex = 1.15,
  cat.fontface = "bold",
  cat.default.pos = "outer",
  cat.pos = c(-27, 27),
    cat.dist = c(0.055, 0.055),
  
  # Change color of ovals
  col=c("#440154ff", '#21908dff'),
  fill = c(alpha("#440154ff",0.3), alpha('#21908dff',0.3)),
)
```

<img src="03-Chapter3_files/figure-html/print-venn-1.png" width="672" />


## Concluding Remarks
In conclusion, we identified 315 genes that show altered expression in response to arsenic exposure that may be under epigenetic control. These genes represent critical mediators of oxidative stress and inflammation, among other important cellular processes. Results yielded an important list of genes representing potential targets for further evaluation, to better understand mechanism of environmental exposure-induced disease. Together, this example highlights the utility of CTD to address environmental health research questions.

For more information, see the recently updated primary CTD publication:  

+ Davis AP, Grondin CJ, Johnson RJ, Sciaky D, Wiegers J, Wiegers TC, Mattingly CJ. Comparative Toxicogenomics Database (CTD): update 2021. Nucleic Acids Res. 2021 Jan 8;49(D1):D1138-D1143. PMID: [33068428](https://pubmed.ncbi.nlm.nih.gov/33068428/).

Additional case studies relevant to environmental health research include the following:

+ An example publication leveraging CTD findings to identify mechanisms of metals-induced birth defects: Ahir BK, Sanders AP, Rager JE, Fry RC. Systems biology and birth defects prevention: blockade of the glucocorticoid receptor prevents arsenic-induced birth defects. Environ Health Perspect. 2013 Mar;121(3):332-8. PMID: [23458687](https://pubmed.ncbi.nlm.nih.gov/23458687/).  

+ An example publication leveraging CTD to help fill data gaps on data poor chemicals, in combination with ToxCast/Tox21 data streams, to elucidate environmental influences on disease pathways: Kosnik MB, Planchart A, Marvel SW, Reif DM, Mattingly CJ. Integration of curated and high-throughput screening data to elucidate environmental influences on disease pathways. Comput Toxicol. 2019 Nov;12:100094. PMID: [31453412](https://pubmed.ncbi.nlm.nih.gov/31453412/).  

+ An example publication leveraging CTD to extract chemical-disease relationships used to derive new chemical risk values, with the goal of prioritizing connections between environmental factors, genetic variants, and human diseases: Kosnik MB, Reif DM. Determination of chemical-disease risk values to prioritize connections between environmental factors, genetic variants, and human diseases. Toxicol Appl Pharmacol. 2019 Sep 15;379:114674. [PMID: 31323264](https://pubmed.ncbi.nlm.nih.gov/31323264/).










# 3.2 Gene Expression Omnibus

This training module was developed by Dr. Kyle R. Roell and Dr. Julia E. Rager

Fall 2021




#### Background on Training Module

#### Introduction to the Environmental Health Database, Gene Expression Omnibus (GEO)
[GEO](https://www.ncbi.nlm.nih.gov/geo/) is a publicly available database repository of high-throughput gene expression data and hybridization arrays, chips, and microarrays that span genome-wide endpoints of genomics, transcriptomics, and epigenomics. The repository is organized and managed by the [The National Center for Biotechnology Information (NCBI)](https://www.ncbi.nlm.nih.gov/), which seeks to advance science and health by providing access to biomedical and genomic information. The three [overall goals](https://www.ncbi.nlm.nih.gov/geo/info/overview.html) of GEO are to: (1) Provide a robust, versatile database in which to efficiently store high-throughput functional genomic data, (2) Offer simple submission procedures and formats that support complete and well-annotated data deposits from the research community, and (3) Provide user-friendly mechanisms that allow users to query, locate, review and download studies and gene expression profiles of interest.

Of high relevance to environmental health, data organized within GEO can be pulled and analyzed to address new environmental health questions, leveraging previously generated data. For example, we have pulled gene expression data from acute myeloid leukemia patients and re-analyzed these data to elucidate new mechanisms of epigenetically-regulated networks involved in cancer, that in turn, may be modified by environmental insults, as previously published in [Rager et al. 2012](https://pubmed.ncbi.nlm.nih.gov/22754483/). We have also pulled and analyzed gene expression data from published studies evaluating toxicity resulting from hexavalent chromium exposure, to further substantiate the role of epigenetic mediators in hexavelent chromium-induced carcinogenesis (see [Rager et al. 2019](https://pubmed.ncbi.nlm.nih.gov/30690063/)). This training exercise leverages an additional dataset that we published and deposited through GEO to evaluate the effects of formaldehyde inhalation exposure, as detailed below.


## Introduction to Training Module
This training module provides an overview on pulling and analyzing data deposited in GEO.  As an example, data are pulled from the published GEO dataset recorded through the online series [GSE42394](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE42394). This series represents Affymetrix rat genome-wide microarray data generated from our previous study, aimed at evaluating the transcriptomic effects of formaldehyde across three tissues: the nose, blood, and bone marrow. For the purposes of this training module, we will focus on evaluating gene expression profiles from nasal samples after 7 days of exposure, collected from rats exposed to 2 ppm formaldehyde via inhalation. These findings, in addition to other epigenomic endpoint measures, have been previously published (see [Rager et al. 2014](https://pubmed.ncbi.nlm.nih.gov/24304932/)).

This training module specifically guides trainees through the loading of required packages and data, including the manual upload of GEO data as well as the upload/organization of data leveraging the [GEOquery package](https://www.bioconductor.org/packages/release/bioc/html/GEOquery.html). Data are then further organized and combined with gene annotation information through the merging of platform annotation files. Example visualizations are then produced, including boxplots to evaluate the overall distribution of expression data across samples, as well as heat map visualizations that compare unscaled versus scaled gene expression values. Statistical analyses are then included to identify which genes are significantly altered in expression upon exposure to formaldehyde. Together, this training module serves as a simple example showing methods to access and download GEO data and to perform data organization, analysis, and visualization tasks through applications-based questions.


#### Training Module's **Environmental Health Questions**
This training module was specifically developed to answer the following environmental health questions:

(1) What kind of molecular identifiers are commonly used in microarray-based -omics technologies?
(2) How can we convert platform-specific molecular identifiers used in -omics study designs to gene-level information?
(3)	Why do we often scale gene expression signatures prior to heat map visualizations?
(4) What genes are altered in expression by formaldehyde inhalation exposure?
(5) What are the potential biological consequences of these gene-level perturbations?



#### Script Preparations

#### Cleaning the global environment

```r
rm(list=ls())
```


#### Installing required R packages
If you already have these packages installed, you can skip this step, or you can run the below code which checks installation status for you

```r
if (!requireNamespace("tidyverse"))
  install.packages("tidyverse")
if (!requireNamespace("reshape2"))
    install.packages("reshape2")

# GEOquery, this will install BiocManager if you don't have it installed
if (!requireNamespace("BiocManager"))
  install.packages("BiocManager")
BiocManager::install("GEOquery")
```

```
## Warning: package(s) not installed when version(s) same as current; use `force = TRUE` to
##   re-install: 'GEOquery'
```


#### Loading R packages required for this session

```r
library(tidyverse)
library(reshape2)
library(GEOquery)
```
For more information on the **tidyverse package**, see its associated [CRAN webpage](https://cran.r-project.org/web/packages/tidyverse/index.html), primary [webpage](https://www.tidyverse.org/packages/), and peer-reviewed [article released in 2018](https://onlinelibrary.wiley.com/doi/10.1002/sdr.1600).

For more information on the **reshape2 package**, see its associated [CRAN webpage](https://cran.r-project.org/web/packages/reshape2/index.html), [R Documentation](https://www.rdocumentation.org/packages/reshape2/versions/1.4.4), and [helpful website](https://seananderson.ca/2013/10/19/reshape/) providing an introduction to the reshape2 package.

For more information on the **GEOquery package**, see its associated [Bioconductor website](https://www.bioconductor.org/packages/release/bioc/html/GEOquery.html) and [R Documentation file](https://www.rdocumentation.org/packages/GEOquery/versions/2.38.4).



#### Set your working directory

```r
setwd("/filepath to where your input files are")
```





#### Loading and Organizing the Example Dataset
Let's start by loading the GEO dataset needed for this training module. As explained in the introduction, this module walks through two methods of uploading GEO data: manual option vs automatic option using the GEOquery package. These two methods are detailed below.



## GEO Data in R

#### 1. Manually Downloading and Uploading GEO Files
In this first method, we will navigate to the datasets within the GEO website, manually download its associated text data file, save it in our working directory, and then upload it into our global environment in R.

For the purposes of this training exercise, we manually downloaded the GEO series matrix file from the GEO series webpage, located at: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE42394. The specific file that was downloaded was noted as *GSE42394_series_matrix.txt*, pulled by clicking on the link indicated by the red arrow from the GEO series webpage:

<img src="_book/TAME_Toolkit_files/figure-html/Module3_2_GEO_Screenshot1.png" width="506" />


For simplicity, we also have already pre-filtered this file for the samples we are interested in, focusing on the rat nasal gene expression data after 7 days of exposure to gaseous formaldehyde. This filtered file was saved as: *GSE42394_series_matrix_filtered.txt*


At this point, we can simply read in this pre-filtered text file for the purposes of this training module

```r
geodata_manual = read.table(file="Module3_2/Module3_2_GSE42394_series_matrix_filtered.txt",
                             header=T)
```


Because this is a manual approach, we have to also manually define the treated and untreated samples (based on manually opening the surrounding metadata from the GEO webpage)

Manually defining treated and untreated for these samples of interest:

```r
exposed_manual = c("GSM1150940", "GSM1150941", "GSM1150942")
unexposed_manual = c("GSM1150937", "GSM1150938", "GSM1150939")
```



#### 2. Uploading and Organizing GEO Files through the GEOquery Package
In this second method, we will leverage the GEOquery package, which allows for easier downloading and reading in of data from GEO without having to manually download raw text files, and manually assign sample attributes (e.g., exposed vs unexposed). This package is set-up to automatically merge sample information from GEO metadata files with raw genome-wide datasets.


Let's first use the getGEO function (from the GEOquery package) to load data from our series matrix.
*Note that this line of code may take a couple of minutes*

```r
geo.getGEO.data = getGEO(filename='Module3_2/Module3_2_GSE42394_series_matrix.txt')
```



One of the reasons the getGEO package is so helpful is that we can automatically link a dataset with nicely organized sample information using the pData() function.

```r
sampleInfo = pData(geo.getGEO.data)
```


Let's view this sample information / metadata file, first by viewing what the column headers are.

```r
colnames(sampleInfo)
```

```
##  [1] "title"                   "geo_accession"          
##  [3] "status"                  "submission_date"        
##  [5] "last_update_date"        "type"                   
##  [7] "channel_count"           "source_name_ch1"        
##  [9] "organism_ch1"            "characteristics_ch1"    
## [11] "characteristics_ch1.1"   "characteristics_ch1.2"  
## [13] "characteristics_ch1.3"   "characteristics_ch1.4"  
## [15] "characteristics_ch1.5"   "treatment_protocol_ch1" 
## [17] "growth_protocol_ch1"     "molecule_ch1"           
## [19] "extract_protocol_ch1"    "label_ch1"              
## [21] "label_protocol_ch1"      "taxid_ch1"              
## [23] "hyb_protocol"            "scan_protocol"          
## [25] "description"             "data_processing"        
## [27] "platform_id"             "contact_name"           
## [29] "contact_email"           "contact_department"     
## [31] "contact_institute"       "contact_address"        
## [33] "contact_city"            "contact_zip/postal_code"
## [35] "contact_country"         "supplementary_file"     
## [37] "data_row_count"          "age:ch1"                
## [39] "cell type:ch1"           "gender:ch1"             
## [41] "strain:ch1"              "time:ch1"               
## [43] "treatment:ch1"
```

Then viewing the first five columns.

```r
sampleInfo[1:10,1:5]
```

```
##                                                    title geo_accession
## GSM1150937            Nose_7DayControl_Rep1 [Affymetrix]    GSM1150937
## GSM1150938            Nose_7DayControl_Rep2 [Affymetrix]    GSM1150938
## GSM1150939            Nose_7DayControl_Rep3 [Affymetrix]    GSM1150939
## GSM1150940            Nose_7DayExposed_Rep1 [Affymetrix]    GSM1150940
## GSM1150941            Nose_7DayExposed_Rep2 [Affymetrix]    GSM1150941
## GSM1150942            Nose_7DayExposed_Rep3 [Affymetrix]    GSM1150942
## GSM1150943 WhiteBloodCells_7DayControl_Rep1 [Affymetrix]    GSM1150943
## GSM1150944 WhiteBloodCells_7DayControl_Rep2 [Affymetrix]    GSM1150944
## GSM1150945 WhiteBloodCells_7DayControl_Rep3 [Affymetrix]    GSM1150945
## GSM1150946 WhiteBloodCells_7DayExposed_Rep1 [Affymetrix]    GSM1150946
##                           status submission_date last_update_date
## GSM1150937 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150938 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150939 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150940 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150941 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150942 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150943 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150944 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150945 Public on Jan 07 2014     May 29 2013      Jan 07 2014
## GSM1150946 Public on Jan 07 2014     May 29 2013      Jan 07 2014
```
This shows that each sample is provided with a unique number starting with "GSM", and these are described by information summarized in the "title" column. We can also see that these data were made public on Jan 7, 2014.


Let's view the next five columns.

```r
sampleInfo[1:10,6:10]
```

```
##            type channel_count                                 source_name_ch1
## GSM1150937  RNA             1        Nasal epithelial cells, 7 day, unexposed
## GSM1150938  RNA             1        Nasal epithelial cells, 7 day, unexposed
## GSM1150939  RNA             1        Nasal epithelial cells, 7 day, unexposed
## GSM1150940  RNA             1          Nasal epithelial cells, 7 day, exposed
## GSM1150941  RNA             1          Nasal epithelial cells, 7 day, exposed
## GSM1150942  RNA             1          Nasal epithelial cells, 7 day, exposed
## GSM1150943  RNA             1 Circulating white blood cells, 7 day, unexposed
## GSM1150944  RNA             1 Circulating white blood cells, 7 day, unexposed
## GSM1150945  RNA             1 Circulating white blood cells, 7 day, unexposed
## GSM1150946  RNA             1   Circulating white blood cells, 7 day, exposed
##                 organism_ch1 characteristics_ch1
## GSM1150937 Rattus norvegicus        gender: male
## GSM1150938 Rattus norvegicus        gender: male
## GSM1150939 Rattus norvegicus        gender: male
## GSM1150940 Rattus norvegicus        gender: male
## GSM1150941 Rattus norvegicus        gender: male
## GSM1150942 Rattus norvegicus        gender: male
## GSM1150943 Rattus norvegicus        gender: male
## GSM1150944 Rattus norvegicus        gender: male
## GSM1150945 Rattus norvegicus        gender: male
## GSM1150946 Rattus norvegicus        gender: male
```
We can see that information is provided here surrounding the type of sample that was analyzed (i.e., RNA), more information on the collected samples within the column 'source_name_ch1', and the organism (rat) is provided in the 'organism_ch1' column.


More detailed metadata information is provided throughout this file, as seen when viewing the column headers above.


#### Defining Samples

Now, we can use this information to define the samples we want to analyze. Note that this is the same step we did manually above.

In this training exercise, we are focusing on responses in the nose, so we can easily filter for cell type = Nasal epithelial cells (specifically in the "cell type:ch1" variable). We are also focusing on responses collected after 7 days of exposure, which we can filter for using time = 7 day (specifically in the "time:ch1" variable). We will also define exposed and unexposed samples using the variable "treatment:ch1".

First, let's subset the sampleInfo dataframe to just keep the samples we're interested in

```r
# Define a vector variable (here we call it 'keep') that will store rows we want to keep
keep = rownames(sampleInfo[which(sampleInfo$`cell type:ch1`=="Nasal epithelial cells" 
                                  & sampleInfo$`time:ch1`=="7 day"),])

# Then subset the sample info for just those samples we defined in keep variable
sampleInfo = sampleInfo[keep,]
```


Next, we can pull the exposed and unexposed animal IDs. Let's first see how these are labeled within the "treatment:ch1" variable.

```r
unique(sampleInfo$`treatment:ch1`)
```

```
## [1] "unexposed"          "2 ppm formaldehyde"
```


And then search for the rows of data, pulling the sample animal IDs (which are in the variable 'geo_accession').

```r
exposedIDs = sampleInfo[which(sampleInfo$`treatment:ch1`=="2 ppm formaldehyde"), 
                          "geo_accession"]
unexposedIDs = sampleInfo[which(sampleInfo$`treatment:ch1`=="unexposed"), 
                            "geo_accession"]
```


The next step is to pull the expression data we want to use in our analyses. The GEOquery function, exprs(), allows us to easily pull these data. Here, we can pull the data we're interested in using the exprs() function, while defining the data we want to pull based off our previously generated 'keep' vector.

```r
# As a reminder, this is what the 'keep' vector includes 
# (i.e., animal IDs that we're interested in)
keep
```

```
## [1] "GSM1150937" "GSM1150938" "GSM1150939" "GSM1150940" "GSM1150941"
## [6] "GSM1150942"
```


```r
# Using the exprs() function
geodata = exprs(geo.getGEO.data[,keep])
```


Let's view the full dataset as is now:

```r
head(geodata)
```

```
##          GSM1150937 GSM1150938 GSM1150939 GSM1150940 GSM1150941 GSM1150942
## 10700001    5786.60    5830.08    5637.34    5313.33    5557.04    5469.90
## 10700002     192.92     206.86     220.83     183.12     177.16     198.64
## 10700003    1820.98    1795.79    1735.70    1578.02    1681.58    1632.20
## 10700004      66.95      65.61      64.41      60.19      60.41      60.67
## 10700005     770.07     753.41     731.20     684.53     657.25     667.66
## 10700006       5.80       5.35       5.48       5.58       5.39       5.35
```
This now represents a matrix of data, with animal IDs as column headers and expression levels within the matrix.


#### Simplifying column names
These column names are not the easiest to interpret, so let's rename these columns to indicate which animals were from the exposed vs. unexposed groups.

We need to first convert our expression dataset to a dataframe so we can edit columns names, and continue with downstream data manipulations that require dataframe formats.

```r
geodata = data.frame(geodata)
```


Let's remind ourselves what the column names are:

```r
colnames(geodata)
```

```
## [1] "GSM1150937" "GSM1150938" "GSM1150939" "GSM1150940" "GSM1150941"
## [6] "GSM1150942"
```

Which ones of these are exposed vs unexposed animals can be determined by viewing our previously defined vectors.

```r
exposedIDs
```

```
## [1] "GSM1150940" "GSM1150941" "GSM1150942"
```

```r
unexposedIDs
```

```
## [1] "GSM1150937" "GSM1150938" "GSM1150939"
```
With this we can tell that the first three listed IDs are from unexposed animals, and the last three IDs are from exposed animals.

Let's simplify the names of these columns to indicate exposure status and replicate number.

```r
colnames(geodata) = c("Control_1", "Control_2", "Control_3", "Exposed_1", 
                       "Exposed_2", "Exposed_3")
```


And we'll now need to re-define our 'exposed' vs 'unexposed' vectors for downstream script.

```r
exposedIDs = c("Exposed_1", "Exposed_2", "Exposed_3")
unexposedIDs = c("Control_1", "Control_2", "Control_3")
```



Viewing the data again:

```r
head(geodata)
```

```
##          Control_1 Control_2 Control_3 Exposed_1 Exposed_2 Exposed_3
## 10700001   5786.60   5830.08   5637.34   5313.33   5557.04   5469.90
## 10700002    192.92    206.86    220.83    183.12    177.16    198.64
## 10700003   1820.98   1795.79   1735.70   1578.02   1681.58   1632.20
## 10700004     66.95     65.61     64.41     60.19     60.41     60.67
## 10700005    770.07    753.41    731.20    684.53    657.25    667.66
## 10700006      5.80      5.35      5.48      5.58      5.39      5.35
```
These data are now looking easier to interpret/analyze. Still, the row identifiers include 8 digit numbers starting with "107...". We know that this dataset is a gene expression dataset, but these identifiers, in themselves, don't tell us much about what genes these are referring to. These numeric IDs specifically represent microarray probesetIDs, that were produced by the Affymetrix platform used in the original study.

**But how can we tell which genes are represented by these data?!**


#### Adding Gene Symbol Information
<!-- #### Adding Gene Symbol Information to -Omic Data Sets through Platform Annotation Files -->
Each -omics dataset contained within GEO points to a specific platform that was used to obtain measurements.
In instances where we want more information surrounding the molecular identifiers, we can merge the platform-specific annotation file with the molecular IDs given in the full dataset.

For example, let's pull the platform-specific annotation file for this experiment.  

+ Let's revisit the [website](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE42394) that contained the original dataset on GEO 
+ Scroll down to where it lists "Platforms", and there is a hyperlinked platform number "GPL6247" (see arrow below)

<img src="_book/TAME_Toolkit_files/figure-html/Module3_2_GEO_Screenshot2.png" width="509" />


+ Click on this, and you will be navigated to a different GEO website describing the Affymetrix rat array platform that was used in this analysis
+ Note that this website also includes information on when this array became available, links to other experiments that have used this platform within GEO, and much more
+ Here, we're interested in pulling the corresponding gene symbol information for the probeset IDs
+ To do so, scroll to the bottom, and click "Annotation SOFT table..." and download the corresponding .gz file within your working directory
+ Unzip this, and you will find the master annotation file: "GPL6247.annot"

In this exercise, we've already done these steps and unzipped the file in our working directory. So at this point, we can simply read in this annotation dataset, still using the GEOquery function to help automate.

```r
geo.annot = GEOquery::getGEO(filename="Module3_2/Module3_2_GPL6247.annot")
```

Now we can use the Table function from GEOquery to pull data from the annotation dataset.

```r
id.gene.table = GEOquery::Table(geo.annot)[,c("ID", "Gene symbol")]
id.gene.table[1:10,1:2]
```

```
##          ID                        Gene symbol
## 1  10701620 Vom2r67///Vom2r5///Vom2r6///Vom2r4
## 2  10701630                                   
## 3  10701632                                   
## 4  10701636                                   
## 5  10701643                                   
## 6  10701648                             Vom2r5
## 7  10701654                             Vom2r6
## 8  10701663                                   
## 9  10701666                                   
## 10 10701668                   Vom2r65///Vom2r1
```
With these two columns of data, we now have the needed IDs and gene symbols to match with our dataset.



Within the full dataset, we need to add a new column for the probeset ID, taken from the rownames, in preparation for the merging step.

```r
geodata$ID = rownames(geodata)
```

We can now merge the gene symbol information by ID with our expression data.

```r
geodata_genes = merge(geodata, id.gene.table, by="ID")
head(geodata_genes)
```

```
##         ID Control_1 Control_2 Control_3 Exposed_1 Exposed_2 Exposed_3
## 1 10700001   5786.60   5830.08   5637.34   5313.33   5557.04   5469.90
## 2 10700002    192.92    206.86    220.83    183.12    177.16    198.64
## 3 10700003   1820.98   1795.79   1735.70   1578.02   1681.58   1632.20
## 4 10700004     66.95     65.61     64.41     60.19     60.41     60.67
## 5 10700005    770.07    753.41    731.20    684.53    657.25    667.66
## 6 10700006      5.80      5.35      5.48      5.58      5.39      5.35
##   Gene symbol
## 1            
## 2            
## 3            
## 4            
## 5            
## 6
```
Note that many of the probeset IDs do not map to full gene symbols, which is shown here by viewing the top few rows - this is expected in genome-wide analyses based on microarray platforms.

Let's look at the first 25 unique genes in these data:

```r
UniqueGenes = unique(geodata_genes$`Gene symbol`)
UniqueGenes[1:25]
```

```
##  [1] ""                                           
##  [2] "Vom2r67///Vom2r5///Vom2r6///Vom2r4"         
##  [3] "Vom2r5"                                     
##  [4] "Vom2r6"                                     
##  [5] "Vom2r65///Vom2r1"                           
##  [6] "Vom2r65///Vom2r5///Vom2r1///Vom2r6///Vom2r4"
##  [7] "Vom2r65///Vom2r5///Vom2r6///Vom2r4"         
##  [8] "Raet1e"                                     
##  [9] "Lrp11"                                      
## [10] "Katna1"                                     
## [11] "Ppil4"                                      
## [12] "Zc3h12d"                                    
## [13] "Shprh"                                      
## [14] "Fbxo30"                                     
## [15] "Epm2a"                                      
## [16] "Sf3b5"                                      
## [17] "Plagl1"                                     
## [18] "Fuca2"                                      
## [19] "Adat2"                                      
## [20] "Hivep2"                                     
## [21] "Nmbr"                                       
## [22] "Cited2"                                     
## [23] "Txlnb"                                      
## [24] "Reps1"                                      
## [25] "Perp"
```
Again, you can see that the first value listed is blank, representing probesetIDs that do not match to fully annotated gene symbols. Though the rest pertain for gene symbols annotated to the rat genome.

You can also see that some gene symbols have multiple entries, separated by "///"

To simplify identifiers, we can pull just the first gene symbol, and remove the rest by using gsub().

```r
geodata_genes$`Gene symbol` = gsub("///.*", "", geodata_genes$`Gene symbol`)
```

Let's alphabetize by main expression dataframe by gene symbol.

```r
geodata_genes = geodata_genes[order(geodata_genes$`Gene symbol`),]
```

And then re-view these data:

```r
geodata_genes[1:5,]
```

```
##         ID Control_1 Control_2 Control_3 Exposed_1 Exposed_2 Exposed_3
## 1 10700001   5786.60   5830.08   5637.34   5313.33   5557.04   5469.90
## 2 10700002    192.92    206.86    220.83    183.12    177.16    198.64
## 3 10700003   1820.98   1795.79   1735.70   1578.02   1681.58   1632.20
## 4 10700004     66.95     65.61     64.41     60.19     60.41     60.67
## 5 10700005    770.07    753.41    731.20    684.53    657.25    667.66
##   Gene symbol
## 1            
## 2            
## 3            
## 4            
## 5
```

In preparation for the visualization steps below, let's reset the probeset IDs to rownames.

```r
rownames(geodata_genes) = geodata_genes$ID

# Can then remove this column within the dataframe
geodata_genes$ID = NULL
```

Finally let's rearrange this dataset to include gene symbols as the first column, right after rownames (probeset IDs).

```r
geodata_genes = geodata_genes[,c(ncol(geodata_genes),1:(ncol(geodata_genes)-1))]
geodata_genes[1:5,]
```

```
##          Gene symbol Control_1 Control_2 Control_3 Exposed_1 Exposed_2
## 10700001               5786.60   5830.08   5637.34   5313.33   5557.04
## 10700002                192.92    206.86    220.83    183.12    177.16
## 10700003               1820.98   1795.79   1735.70   1578.02   1681.58
## 10700004                 66.95     65.61     64.41     60.19     60.41
## 10700005                770.07    753.41    731.20    684.53    657.25
##          Exposed_3
## 10700001   5469.90
## 10700002    198.64
## 10700003   1632.20
## 10700004     60.67
## 10700005    667.66
```

```r
dim(geodata_genes)
```

```
## [1] 29214     7
```

Note that this dataset includes expression measures across **29,214 probes, representing 14,019 unique genes**.
For simplicity in the final exercises, let's just filter for rows representing mapped genes.


```r
geodata_genes = geodata_genes[!(geodata_genes$`Gene symbol` == ""), ]
dim(geodata_genes)
```

```
## [1] 16024     7
```
Note that this dataset now includes 16,024 rows with mapped gene symbol identifiers.


<!-- #### With this, we can now answer **Environmental Health Question #1**: -->
<!-- #### (1) What kind of molecular identifiers are commonly used in microarray-based -omics technologies? -->
<!-- #### *Answer: Platform-specific probeset IDs.* -->

<br>

:::question
<i>With this, we can now answer **Environmental Health Question 1**:</i>
What kind of molecular identifiers are commonly used in microarray-based -omics technologies?
:::
:::answer
**Answer**:  Platform-specific probeset IDs.
:::

<!-- #### We can also answer **Environmental Health Question #2**: -->
<!-- #### (2) How can we convert platform-specific molecular identifiers used in -omics study designs to gene-level information? -->
<!-- #### *Answer: We can merge platform-specific IDs with gene-level information using annotation files.* -->

<br>

:::question
<i>We can also answer **Environmental Health Question 2**:</i>
How can we convert platform-specific molecular identifiers used in -omics study designs to gene-level information?
:::
:::answer
**Answer**: We can merge platform-specific IDs with gene-level information using annotation files.
:::
<br>

## Visualizing Data

#### Visualizing Gene Expression Data using Boxplots and Heat Maps

+ To visualize the -omics data, we can generate boxplots, heat maps, any many other types of visualizations
+ Here, we provide an example to plot a boxplot, which can be used to visualize the variability amongst samples
+ We also provide an example to plot a heat map, comparing unscaled vs scaled gene expression profiles
+ These visualizations can be useful to both simply visualize the data as well as identify patterns across samples or genes

#### Boxplot Visualizations
For this example, let's simply use R's built in boxplot() function.

We only want to use columns with our expression data (2 to 7), so let's pull those columns when running the boxplot function.

```r
boxplot(geodata_genes[,2:7])
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-67-1.png" width="480" />

There seem to be a lot of variability within each sample's range of expression levels, with many outliers. This makes sense given that we are analyzing the expression levels across the rat's entire genome, where some genes won't be expressed at all while others will be highly expressed due to biological and/or potential technical variability.
  
To show plots without outliers, we can simply use outline=F.

```r
boxplot(geodata_genes[,2:7], outline=F)
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-68-1.png" width="480" />
  

#### Heat Map Visualizations
Heat maps are also useful when evaluating large datasets.

There are many different packages you can use to generate heat maps. Here, we use the *superheat* package.

It also takes awhile to plot all genes across the genome, so to save time for this training module, let's randomly select 100 rows to plot.


```r
# To ensure that the same subset of genes are selected each time
set.seed = 101                                     

# Random selection of 100 rows
row.sample = sample(1:nrow(geodata_genes),100) 

# Heat map code
superheat::superheat(geodata_genes[row.sample,2:7], # Only want to plot non-id/gene symbol columns (2 to 7)
                     pretty.order.rows = TRUE,
                     pretty.order.cols = TRUE,
                     col.dendrogram = T,
                     row.dendrogram = T)
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-69-1.png" width="864" />

This produces a heat map with sample IDs along the x-axis and probeset IDs along the y-axis. Here, the values being displayed represent normalized expression values.


One way to improve our ability to distinguish differences between samples is to **scale expression values** across probes. 


#### Scaling data
Z-score is a very common method of scaling that transforms data points to reflect the number of standard deviations they are from the overall mean. Z-score scaling data results in the overall transformation of a dataset to have an overall mean = 0 and standard deviation = 1.

Let's see what happens when we scale this gene expression dataset by z-score across each probe. This can be easily done using the *scale* function.

This specific *scale* function works by centering and scaling across columns, but since we want to use it across probesets (organized as rows), we need to first transpose our dataset, then run the scale function.

```r
geodata_genes_scaled = scale(t(geodata_genes[,2:7]), center=T, scale=T)
```

Now we can transpose it back to the original format (i.e., before it was transposed).

```r
geodata_genes_scaled = t(geodata_genes_scaled)
```


And then view what the normalized and now scaled expression data look like for now a random subset of 100 probesets (representing genes).
<img src="03-Chapter3_files/figure-html/unnamed-chunk-72-1.png" width="864" />

With these data now scaled, we can more easily visualize patterns between samples.


<!-- #### We can also answer **Environmental Health Question #3**: -->
<!-- #### (3) Why do we often scale gene expression signatures prior to heat map visualizations? -->
<!-- #### *Answer: To better visualize patterns in expression signatures between samples.* -->
<br>

:::question
<i>We can also answer **Environmental Health Question 3**:</i>
Why do we often scale gene expression signatures prior to heat map visualizations?
:::
:::answer
**Answer**: To better visualize patterns in expression signatures between samples.
:::

<br>
Now, with these data nicely organized, we can see how statistics can help find which genes show trends in expression associated with formaldehyde exposure.

---

## Statistical Analyses

#### Statistical Analyses to Identify Genes altered by Formaldehyde
A simple way to identify differences between formaldehyde-exposed and unexposed samples is to use a t-test. Because there are so many tests being performed, one for each gene, it is also important to carry out multiple test corrections through  a p-value adjustment method. 

We need to run a t-test for each row of our dataset. This exercise demonstrates two different methods to run a t-test:

+ Method 1: using a 'for loop'
+ Method 2: using the apply function (more computationally efficient)

#### Method 1 (m1): 'For Loop'
Let's first re-save the molecular probe IDs to a column within the dataframe, since we need those values in the loop function.

```r
geodata_genes$ID = rownames(geodata_genes)
```


We also need to initially create an empty dataframe to eventually store p-values.

```r
pValue_m1 = matrix(0, nrow=nrow(geodata_genes), ncol=3)
colnames(pValue_m1) = c("ID", "pval", "padj")
head(pValue_m1)
```

```
##      ID pval padj
## [1,]  0    0    0
## [2,]  0    0    0
## [3,]  0    0    0
## [4,]  0    0    0
## [5,]  0    0    0
## [6,]  0    0    0
```
You can see the empty dataframe that was generated through this code.


Then we can loop through the entire dataset to acquire p-values from t-test statistics, comparing n=3 exposed vs n=3 unexposed samples.

```r
for (i in 1:nrow(geodata_genes)) {
  
  #Get the ID
  ID.i = geodata_genes[i, "ID"];
  
  #Run the t-test and get the p-value
  pval.i = t.test(geodata_genes[i,exposedIDs], geodata_genes[i,unexposedIDs])$p.value;
  
  #Store the data in the empty dataframe
  pValue_m1[i,"ID"] = ID.i;
  pValue_m1[i,"pval"] = pval.i
  
}
```

View the results:

```r
# Note that we're not pulling the last column (padj) since we haven't calculated these yet
pValue_m1[1:5,1:2] 
```

```
##      ID         pval                
## [1,] "10903987" "0.0812802229304083"
## [2,] "10714794" "0.757311314118124" 
## [3,] "10858408" "0.390952310869689" 
## [4,] "10872252" "0.0548937136005506"
## [5,] "10905819" "0.173539535577791"
```



#### Method 2 (m2): Apply Function
For the second method, we can use the *apply()* function to calculate resulting t-test p-values more efficiently labeled. 


```r
pValue_m2 = apply(geodata_genes[,2:7], 1, function(x) t.test(x[unexposedIDs],
                                                              x[exposedIDs])$p.value)
names(pValue_m2) = geodata_genes[,"ID"]
```

We can convert the results into a dataframe to make it similar to m1 matrix we created above.

```r
pValue_m2  = data.frame(pValue_m2)

# Now create an ID column
pValue_m2$ID = rownames(pValue_m2)
```

Then we can view at the two datasets to see they result in the same pvalues.

```r
head(pValue_m1)
```

```
##      ID         pval                 padj
## [1,] "10903987" "0.0812802229304083" "0" 
## [2,] "10714794" "0.757311314118124"  "0" 
## [3,] "10858408" "0.390952310869689"  "0" 
## [4,] "10872252" "0.0548937136005506" "0" 
## [5,] "10905819" "0.173539535577791"  "0" 
## [6,] "10907585" "0.215200167867295"  "0"
```

```r
head(pValue_m2)
```

```
##           pValue_m2       ID
## 10903987 0.08128022 10903987
## 10714794 0.75731131 10714794
## 10858408 0.39095231 10858408
## 10872252 0.05489371 10872252
## 10905819 0.17353954 10905819
## 10907585 0.21520017 10907585
```
We can see from these results that both methods (m1 and m2) generate the same statistical p-values.

#### Interpreting Results
Let's again merge these data with the gene symbols to tell which genes are significant.

First, let's convert to a dataframe and then merge as before, for one of the above methods as an example (m1).

```r
pValue_m1 = data.frame(pValue_m1)
pValue_m1 = merge(pValue_m1, id.gene.table, by="ID")
```

We can also add a multiple test correction by applying a false discovery rate-adjusted p-value; here, using the Benjamini Hochberg (BH) method.

```r
# Here fdr is an alias for B-H method
pValue_m1[,"padj"] = p.adjust(pValue_m1[,"pval"], method=c("fdr"))
```

Now, we can sort these statistical results by adjusted p-values.

```r
pValue_m1.sorted = pValue_m1[order(pValue_m1[,'padj']),]
head(pValue_m1.sorted)
```

```
##            ID                 pval       padj Gene symbol
## 9143 10837582 4.57288413593085e-07 0.00732759      Olr633
## 5640 10783648 1.93688668590855e-06 0.01551834      Slc7a8
## 8    10701699   0.0166773380386967 0.13089115       Lrp11
## 17   10701817   0.0131845685452954 0.13089115       Fuca2
## 19   10701830  0.00586885826460337 0.13089115      Hivep2
## 23   10701880  0.00749149990409956 0.13089115       Reps1
```

Pulling just the significant genes using an adjusted p-value threshold of 0.05.

```r
adj.pval.sig = pValue_m1[which(pValue_m1[,'padj'] < .05),]

# Viewing these genes
adj.pval.sig       
```

```
##            ID                 pval       padj Gene symbol
## 5640 10783648 1.93688668590855e-06 0.01551834      Slc7a8
## 9143 10837582 4.57288413593085e-07 0.00732759      Olr633
```


<!-- #### With this, we can answer **Environmental Health Question #4**: -->
<!-- #### (4) What genes are altered in expression by formaldehyde inhalation exposure? -->
<!-- #### *Answer: Olr633 and Slc7a8.* -->
<br>

:::question
<i>With this, we can answer **Environmental Health Question 4**:</i>
What genes are altered in expression by formaldehyde inhalation exposure?
:::
:::answer
**Answer**: Olr633 and Slc7a8.
:::

<br>

<!-- #### With this, we can answer **Environmental Health Question #5**: -->
<!-- #### (5) What are the potential biological consequences of these gene-level perturbations? -->
<!-- #### *Answer: Olr633 stands for 'olfactory receptor 633'. Olr633 is up-regulated in expression, meaning that formaldehyde inhalation exposure has a smell that resulted in 'activated' olfactory receptors in the nose of these exposed rats. Slc7a8 stands for 'solute carrier family 7 member 8'. Slc7a8 is down-regulated in expression, and it plays a role in many biological processes, that when altered, can lead to changes in cellular homeostasis and disease.* -->


Finally, let's plot these using a mini heat map.
Note that we can use probesetIDs, then gene symbols, in rownames to have them show in heat map labels.
<img src="03-Chapter3_files/figure-html/unnamed-chunk-84-1.png" width="672" />

Note that this statistical filter is pretty strict when comparing only n=3 vs n=3 biological replicates. If we loosen the statistical criteria to p-value < 0.05, this is what we can find:

```r
pval.sig = pValue_m1[which(pValue_m1[,'pval'] < .05),]
nrow(pval.sig)
```

```
## [1] 5327
```
5327 genes with significantly altered expression!

Note that other filters are commonly applied to further focus these lists (e.g., background and fold change filters) prior to statistical evaluation, which can impact the final results. See [Rager et al. 2013](https://pubmed.ncbi.nlm.nih.gov/24304932/)  for further statistical approaches and visualizations.

<br>

:::question
<i>With this, we can answer **Environmental Health Question 5**:</i>
What are the potential biological consequences of these gene-level perturbations?
:::
:::answer
**Answer**: Olr633 stands for 'olfactory receptor 633'. Olr633 is up-regulated in expression, meaning that formaldehyde inhalation exposure has a smell that resulted in 'activated' olfactory receptors in the nose of these exposed rats. Slc7a8 stands for 'solute carrier family 7 member 8'. Slc7a8 is down-regulated in expression, and it plays a role in many biological processes, that when altered, can lead to changes in cellular homeostasis and disease.
:::

<br>

## Concluding Remarks
In conclusion, this training module provides an overview of pulling, organizing, visualizing, and analyzing -omics data from the online repository, Gene Expression Omnibus (GEO). Trainees are guided through the overall organization of an example high dimensional dataset, focusing on transcriptomic responses in the nasal epithelium of rats exposed to formaldehyde. Data are visualized and then analyzed using standard two-group comparisons. Findings are interpreted for biological relevance, yielding insight into the effects resulting from formaldehyde exposure. 

For additional case studies that leverage GEO, see the following publications that also address environmental health questions from our research group:

+ Rager JE, Fry RC. The aryl hydrocarbon receptor pathway: a key component of the microRNA-mediated AML signalisome. Int J Environ Res Public Health. 2012 May;9(5):1939-53. doi: 10.3390/ijerph9051939. Epub 2012 May 18. PMID: 22754483; PMCID: [PMC3386597](https://pubmed.ncbi.nlm.nih.gov/22754483/).

+ Rager JE, Suh M, Chappell GA, Thompson CM, Proctor DM. Review of transcriptomic responses to hexavalent chromium exposure in lung cells supports a role of epigenetic mediators in carcinogenesis. Toxicol Lett. 2019 May 1;305:40-50. PMID: [30690063](https://pubmed.ncbi.nlm.nih.gov/30690063/).








# 3.3 Database Integration: Air Quality, Mortality, and Environmental Justice Data


The development of this training module was led by Dr. Cavin Ward-Caviness.

Fall 2021

*Disclaimer: The views expressed in this document are those of the author and do not necessarily reflect the views or policies of the U.S. EPA.*




#### Introduction to Exposure and Health Databases used in this Module

In this R example we will use publicly available exposure and health databases to examine associations between air quality and mortality across the entire U.S. Specific databases that we will query include the following:

+ EPA Air Quality data: As an example air pollutant exposure dataset, 2016 annual average data from the EPA Air Quality System database will be analyzed,  using data downloaded and organized from the following website: <https://aqs.epa.gov/aqsweb/airdata/annual_conc_by_monitor_2016.zip>

+ CDC Health Outcome data: As an example health outcome dataset, the 2016 CDC Underlying Cause of Death dataset, from the WONDER (Wide-ranging ONline Data for Epidemiologic Research) website  will be analyzed, using All-Cause Mortality Rates downloaded and organized from the following website: <https://wonder.cdc.gov/ucd-icd10.html>

+ Human covariate data: The potential influence of covariates (e.g., race) and other confounders will be analyzed using data downloaded and organized from the following 2016 county-level resource: <https://www.countyhealthrankings.org/explore-health-rankings/rankings-data-documentation/national-data-documentation-2010-2019>


## Introduction to Training Module

This training module provides an example analysis based on the integration of data across multiple environmental health databases. This module specifically guides trainees through an explanation of how the data were downloaded and organized, and then details the loading of required packages and datasets. Then, this module provides code for visualizing county-level air pollution measures obtained through U.S. EPA monitoring stations throughout the U.S. Air pollution measures include PM2.5, NO2, and SO2, are visualized here as the yearly average. Air pollution concentrations are then evaluated for potential relationship to the health outcome, mortality. Specifically, age adjusted mortality rates are organized and statistically related to PM2.5 concentrations through linear regression modeling. Crude statistical models are first provided that do not take into account the influence of potential confounders. Then, statistical models are used that adjust for potential confounders, including adult smoking rates, obesity, food environment indicators, physical activity, employment status, rural vs urban living percentages, sex, ethnicity, and race. Results from these models point to the finding that areas with higher percentages of African-Americans may be experiencing higher impacts from PM2.5 on mortality. This relationship is of high interest, as it represents a potential Environmental Justice issue.


#### Training Module's **Environmental Health Questions**:
This training module was specifically developed to answer the following environmental health questions:

(1) What areas of the U.S. are most heavily monitored for air quality?
(2) Is there an association between long-term, ambient PM2.5 concentrations and mortality at the county level? Stated another way we are asking: Do counties with higher annual average PM2.5 concentrations also have higher all-cause mortality rates?
(3) What is the difference when running crude statistical models vs. statistical models that adjust for potential confounding, when evaluating the relationship between PM2.5 and mortality?
(4) Do observed associations differ when comparing between counties with a higher vs. lower percentage of African-Americans which can indicate environmental justice concerns?


#### Script Preparations

#### Cleaning the global environment

```r
rm(list=ls())
```


#### Installing required R packages
If you already have these packages installed, you can skip this step, or you can run the below code which checks installation status for you:

```r
if (!requireNamespace("sf"))
  install.packages("sf");
if (!requireNamespace("dplyr"))
  install.packages("dplyr");
if (!requireNamespace("tidyr"))
  install.packages("tidyr");
if (!requireNamespace("ggplot2"))
  install.packages("ggplot2");
if (!requireNamespace("ggthemes"))
  install.packages("ggthemes");
```


#### Loading R packages required for this session

```r
library(sf)
library(dplyr)
library(tidyr)
library(ggplot2)
library(ggthemes)
```



#### Set your working directory

```r
setwd("/filepath to where your input files are")
```


#### Loading Example Datasets
Let's start by loading the datasets needed for this training module. As detailed in the introduction, these data were previously downloaded and organized, and specifically made available for this training excercise as a compiled RDataset, containing organized dataframes ready to analyze.

We can now read in these organized data using the 'load' function:

```r
load("Module3_3/Module3_3_Data_AirQuality_Mortality_EJ.RData")
```



#### Data Viewing & Plotting
The air pollution data has already been aggregated to the county level so let's plot it and see what it looks like.

First let's take a look at the data, starting with the county-level shapefile:

```r
head(counties_shapefile)
```

```
## Simple feature collection with 6 features and 9 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -102 ymin: 37.39 xmax: -84.8 ymax: 43.5
## Geodetic CRS:  NAD83
##   STATEFP COUNTYFP COUNTYNS       AFFGEOID GEOID      NAME LSAD     ALAND
## 1      19      107 00465242 0500000US19107 19107    Keokuk   06 1.500e+09
## 2      19      189 00465283 0500000US19189 19189 Winnebago   06 1.037e+09
## 3      20      093 00485011 0500000US20093 20093    Kearny   06 2.255e+09
## 4      20      123 00485026 0500000US20123 20123  Mitchell   06 1.818e+09
## 5      20      187 00485055 0500000US20187 20187   Stanton   06 1.762e+09
## 6      21      005 00516849 0500000US21005 21005  Anderson   06 5.227e+08
##     AWATER                       geometry
## 1  1929323 MULTIPOLYGON (((-92.41 41.5...
## 2  3182052 MULTIPOLYGON (((-93.97 43.5...
## 3  1133601 MULTIPOLYGON (((-101.5 37.9...
## 4 44979981 MULTIPOLYGON (((-98.49 39.2...
## 5   178555 MULTIPOLYGON (((-102 37.54,...
## 6  6311537 MULTIPOLYGON (((-85.17 38, ...
```
This dataframe contains the location information for the counties which we will use for visualizations.


Now let's view the EPA Air Quality Survey (AQS) data collected from 2016:

```r
head(epa_ap_county)
```

```
##   State.Code State.Name County.Code County.Name State_County_Code
## 1          1    Alabama           3     Baldwin              1003
## 2          1    Alabama          27        Clay              1027
## 3          1    Alabama          33     Colbert              1033
## 4          1    Alabama          49      DeKalb              1049
## 5          1    Alabama          55      Etowah              1055
## 6          1    Alabama          69     Houston              1069
##   Parameter.Name            Units.of.Measure County_Avg
## 1           PM25 Micrograms/cubic meter (LC)      7.226
## 2           PM25 Micrograms/cubic meter (LC)      7.364
## 3           PM25 Micrograms/cubic meter (LC)      7.492
## 4           PM25 Micrograms/cubic meter (LC)      7.696
## 5           PM25 Micrograms/cubic meter (LC)      8.196
## 6           PM25 Micrograms/cubic meter (LC)      7.062
```
This dataframe represents county-level air quality measures, collected through the Air Quality Survey (2016), as detailed above. This dataframe is in melted (or long) format, meaning that different air quality measures are organized across rows, with variable measure indicators in the 'Parameter.Name', 'Units.of.Measure', and 'County_Avg' columns.


These data can be restructured to view air quality measures as separate variables labeled across columns using:

```r
# transform from the "long" to "wide" format for the pollutants
epa_ap_county <- epa_ap_county %>% select(-Units.of.Measure) %>% unique() %>% tidyr::spread(Parameter.Name, County_Avg)
head(epa_ap_county)
```

```
##   State.Code State.Name County.Code County.Name State_County_Code NO2  PM25 SO2
## 1          1    Alabama           3     Baldwin              1003  NA 7.226  NA
## 2          1    Alabama          27        Clay              1027  NA 7.364  NA
## 3          1    Alabama          33     Colbert              1033  NA 7.492  NA
## 4          1    Alabama          49      DeKalb              1049  NA 7.696  NA
## 5          1    Alabama          55      Etowah              1055  NA 8.196  NA
## 6          1    Alabama          69     Houston              1069  NA 7.062  NA
```
Note that we can now see the specific pollutant variables 'NO2', 'PM25', and 'SO2' on the far right.

## Population-weighted vs Unweighted Exposures

Here we pause briefly to speak on population-weighted vs unweighted exposures. The analysis we will be undertaking is known as an "ecological" analysis where we are looking at associations by area, e.g. county. When studying environmental exposures by area a common practice is to try to weight the exposures by the population so that exposures better represent the "burden" faced by the population. Ideally for this you would want a systematic model or assessment of the exposure that corresponded with a fine-scale population estimate so that for each county you could weight exposures within different areas of the county by the population exposed. This sparse monitor data (we will examine the population covered later in the tutorial) is not population weighted, but should you see similar analyses with population weighting of exposures you should simply be aware that this better captures the "burden" of exposure experienced by the population within the area estimated, typically zip code or county.

Now let's view the CDC's mortality dataset collected from 2016:

```r
head(cdc_mortality)
```

```
##   Notes             County County.Code Deaths Population Crude.Rate
## 1    NA Autauga County, AL        1001    520      55416     938.36
## 2    NA Baldwin County, AL        1003   1974     208563     946.48
## 3    NA Barbour County, AL        1005    256      25965     985.94
## 4    NA    Bibb County, AL        1007    239      22643    1055.51
## 5    NA  Blount County, AL        1009    697      57704    1207.89
## 6    NA Bullock County, AL        1011    133      10362    1283.54
##   Age.Adjusted.Rate Age.Adjusted.Rate.Standard.Error
## 1             884.4                            39.46
## 2             716.9                            16.58
## 3             800.7                            51.09
## 4             927.7                            61.03
## 5             989.4                            38.35
## 6            1063.0                            93.69
```


We can create a visualization of values throughout the U.S. to further inform what these data look like:

```r
# Can merge them by the FIPS county code which we need to create for the counties_shapefile
counties_shapefile$State_County_Code <- as.character(as.numeric(paste0(counties_shapefile$STATEFP, counties_shapefile$COUNTYFP)))

# Let's merge in the air pollution and mortality data and plot it
counties_shapefile <- merge(counties_shapefile, epa_ap_county, by.x="State_County_Code", by.y="State_County_Code", all.x=TRUE)
counties_shapefile <- merge(counties_shapefile, cdc_mortality, by.x="State_County_Code", by.y="County.Code")

# Will remove alaska and hawaii just so we can look at the continental USA
county_plot <- subset(counties_shapefile, !STATEFP %in% c("02","15"))

# We can start with a simple plot of age-adjusted mortality rate, PM2.5, NO2, and SO2 levels across the U.S.
plot(county_plot[,c("Age.Adjusted.Rate","PM25","NO2","SO2")])
```

<img src="03-Chapter3_files/figure-html/county plot-1.png" width="576" />

You can see that these result in the generation of four different nation-wide plots, showing the distributions of age adjusted mortality rates, PM2.5 concentrations, NO2 concentrations, and SO2 concentrations, averaged per-county.


Let's make a nicer looking plot with ggplot, looking just at PM2.5 levels:

```r
# Using ggplot we can have more control
p <- ggplot(data=county_plot) + 
  geom_sf(aes(fill=PM25)) +
  scale_fill_viridis_c(option="plasma", name="PM2.5",
                       guide = guide_colorbar(
                         direction = "horizontal",
                         barheight = unit(2, units = "mm"),
                         barwidth = unit(50, units = "mm"),
                         draw.ulim = F,
                         title.position = 'top',
                         # some shifting around
                         title.hjust = 0.5,
                         label.hjust = 0.5)) +
  ggtitle("2016 Annual PM25 (EPA Monitors)") +
  theme_map() +
  theme(plot.title = element_text(hjust = 0.5, size=22)) 

p
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-96-1.png" width="576" />




<br>

:::question
<i>With this, we can answer **Environmental Health Question 1**:</i>
What areas of the U.S. are most heavily monitored for air quality?
:::
:::answer
**Answer**: We can tell from the PM2.5 specific plot that air monitors are densely located in California, and other areas with high populations (including the East Coast), while large sections of central U.S. lack air monitoring data.
:::

<br>

#### Analyzing Relationships between PM2.5 and Mortality

Now the primary question is whether counties with higher PM2.5 also have higher mortality rates. To answer this question, first we need to run some data merging in preparation for this analysis:

```r
model_data <- merge(epa_ap_county, cdc_mortality, by.x="State_County_Code", by.y="County.Code")
```


As we saw in the above plot, only a portion of the USA is covered by PM2.5 monitors. Let's see what our population coverage is:

```r
sum(model_data$Population, na.rm=TRUE)
```

```
## [1] 232169063
```

```r
sum(cdc_mortality$Population, na.rm=TRUE)
```

```
## [1] 323127513
```

```r
sum(model_data$Population, na.rm=TRUE)/sum(cdc_mortality$Population, na.rm=TRUE)*100
```

```
## [1] 71.85
```


We can do a quick visual inspection of this using a scatter plot which will also let us check for unexpected distributions of the data (always a good idea):

```r
plot(model_data$Age.Adjusted.Rate, model_data$PM25, main="PM2.5 by Mortality Rate", xlab="Age Adjusted Mortality Rate", ylab="PM25")
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-98-1.png" width="576" />




The univariate correlation is a simple way of quantifying this potential relationships, though it does not nearly tell the complete story. Just as a starting point, let's run this simple univariate correlation calculation using the 'cor' function:

```r
cor(model_data$Age.Adjusted.Rate, model_data$PM25, use="complete.obs")
```

```
## [1] 0.1785
```


## Regression Modeling

Now, let's obtain a more complete estimate of the data through regression modeling. As an initial starting point, let's run this model without any confounders (also known as a 'crude' model).


A simple linear regression model in R can be carried out using the 'lm' (linear model) function. Here, we are evaluating age adjusted mortality rate (age.adjusted.rate) as the dependent variable, and PM2.5 as the independent variable. Values used in evaluating this model were weighted to adjust for the fact that some counties have higher precision in their age adjusted mortality rate (represented by a smaller age adjusted rate standard error).

```r
# Running the linear regression model
m <- lm(Age.Adjusted.Rate ~ PM25,
        data = model_data, weights=1/model_data$Age.Adjusted.Rate.Standard.Error)   
# Viewing the model results through the summary function
summary(m)   
```

```
## 
## Call:
## lm(formula = Age.Adjusted.Rate ~ PM25, data = model_data, weights = 1/model_data$Age.Adjusted.Rate.Standard.Error)
## 
## Weighted Residuals:
##     Min      1Q  Median      3Q     Max 
## -110.10   -9.79    8.36   25.09   84.32 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   661.04      22.53   29.34   <2e-16 ***
## PM25            8.96       2.88    3.11   0.0019 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 29.9 on 608 degrees of freedom
##   (88 observations deleted due to missingness)
## Multiple R-squared:  0.0157,	Adjusted R-squared:  0.0141 
## F-statistic: 9.68 on 1 and 608 DF,  p-value: 0.00195
```
Shown here are summary level statistics summarizing the results of the linear regression model.

In the model summary we see several features. The "Estimate" column is the regression coefficient which tells us the relationship between a 1 ug/m3 change (elevation) in PM2.5 and the age-adjusted all-cause mortality rate. "Std. Error" is the standard error of the estimate. The column "t value" represents the T-statistic which is the test statistic for linear regression models and is simply the "Estimate" divied by "Std. Error". This "t value" is compared with the Student's T distribution in order to determine the p-value ("Pr(>|t|)").

The residuals are the difference between the predicted outcome (age-adjusted mortality rate) and known outcome from the data. For linear regression to be valid this should be normally distributed. The residuals from a linear model can be extracted using the residuals() command and plotted to see their distribution.


<br>

:::question
<i>With this, we can answer **Environmental Health Question 2**:</i>
Is there an association between long-term, ambient PM2.5 concentrations and mortality at the county level? Stated another way we are asking: Do counties with higher annual average PM2.5 concentrations also have higher all-cause mortality rates?
:::
:::answer
**Answer**: Based on these model results, there may indeed be an association between PM2.5 concentrations and mortality (p=0.0019).
:::

<br>


To more thoroughly examine the potential relationship between PM2.5 concentrations and mortality it is absolutely essential to adjust for confounders.

```r
# First we merge the covariate data in with the AQS data
model_data <- merge(model_data, county_health, by.x="State_County_Code", by.y="County.5.digit.FIPS.Code", all.x=TRUE)

# Now we add some relevant confounders to the linear regression model
m <- lm(Age.Adjusted.Rate ~ PM25 + Adult.smoking + Adult.obesity + Food.environment.index + Physical.inactivity +
          High.school.graduation + Some.college + Unemployment + Violent.crime + Percent.Rural + Percent.Females +
          Percent.Asian + Percent.Non.Hispanic.African.American + Percent.American.Indian.and.Alaskan.Native + Percent.NonHispanic.white,
        data = model_data, weights=1/model_data$Age.Adjusted.Rate.Standard.Error)

# And finally we check to see if the statistical association persists
summary(m)
```

```
## 
## Call:
## lm(formula = Age.Adjusted.Rate ~ PM25 + Adult.smoking + Adult.obesity + 
##     Food.environment.index + Physical.inactivity + High.school.graduation + 
##     Some.college + Unemployment + Violent.crime + Percent.Rural + 
##     Percent.Females + Percent.Asian + Percent.Non.Hispanic.African.American + 
##     Percent.American.Indian.and.Alaskan.Native + Percent.NonHispanic.white, 
##     data = model_data, weights = 1/model_data$Age.Adjusted.Rate.Standard.Error)
## 
## Weighted Residuals:
##    Min     1Q Median     3Q    Max 
## -70.06  -7.05   0.96   8.12  64.05 
## 
## Coefficients:
##                                             Estimate Std. Error t value
## (Intercept)                                 616.4798   157.2084    3.92
## PM25                                          3.8485     1.6932    2.27
## Adult.smoking                               859.2734   137.8131    6.24
## Adult.obesity                               605.8431    97.7195    6.20
## Food.environment.index                      -28.6554     3.9336   -7.28
## Physical.inactivity                         117.6092    91.4152    1.29
## High.school.graduation                       55.1445    40.7849    1.35
## Some.college                               -244.4126    49.7876   -4.91
## Unemployment                                 97.9816   161.8902    0.61
## Violent.crime                                 0.0755     0.0152    4.98
## Percent.Rural                               -12.5531    20.8483   -0.60
## Percent.Females                            -271.4550   299.1693   -0.91
## Percent.Asian                                74.5433    55.8926    1.33
## Percent.Non.Hispanic.African.American       153.3991    39.4962    3.88
## Percent.American.Indian.and.Alaskan.Native  200.7001   102.7503    1.95
## Percent.NonHispanic.white                   240.1798    26.3054    9.13
##                                            Pr(>|t|)    
## (Intercept)                                 1.0e-04 ***
## PM25                                        0.02344 *  
## Adult.smoking                               9.4e-10 ***
## Adult.obesity                               1.2e-09 ***
## Food.environment.index                      1.2e-12 ***
## Physical.inactivity                         0.19883    
## High.school.graduation                      0.17694    
## Some.college                                1.2e-06 ***
## Unemployment                                0.54529    
## Violent.crime                               8.6e-07 ***
## Percent.Rural                               0.54736    
## Percent.Females                             0.36464    
## Percent.Asian                               0.18290    
## Percent.Non.Hispanic.African.American       0.00012 ***
## Percent.American.Indian.and.Alaskan.Native  0.05133 .  
## Percent.NonHispanic.white                   < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 14.2 on 514 degrees of freedom
##   (168 observations deleted due to missingness)
## Multiple R-squared:  0.784,	Adjusted R-squared:  0.778 
## F-statistic:  124 on 15 and 514 DF,  p-value: <2e-16
```


<br>

:::question
<i>With this, we can answer **Environmental Health Question 3**:</i>
What is the difference when running crude statistical models vs statistical models that adjust for potential confounding, when evaluating the relationship between PM2.5 and mortality?
:::
:::answer
**Answer**: The relationship between PM2.5 and mortality remains statistically significant when confounders are considered (p=0.023), though is not as significant as when running the crude model (p=0.0019).
:::

<br>

## Environmental Justice Considerations
Environmental Justice is the study of how societal inequities manifest in differences in environmental health risks either due to greater exposures or a worse health response to exposures. Racism and racial discrimination are major factors in both how much pollution people are exposed to as well what their responses might be due to other co-existing inequities (e.g. poverty, access to healthcare, food deserts). Race is a commonly used proxy for experiences of racism and racial discrimination.

Here we will consider the race category of 'Non-Hispanic African-American' to investigate if pollution levels differ by percent African-Americans in a county and if associations between PM2.5 and mortality also differ by this variable, which could indicate Environmental Justice-relevant issues revealed by this data. We will specifically evaluate data distributions across counties with the highest percentage of African-Americans (top 25%) vs lowest percentage of African-Americans (bottom 25%).  

First let's visualize the distribution of % African-Americans in these data:

```r
hist(model_data$Percent.Non.Hispanic.African.American*100, main="Percent African-American by County",xlab="Percent")
```

<img src="03-Chapter3_files/figure-html/unnamed-chunk-101-1.png" width="576" />

And let's look at a summary of the data:

```r
summary(model_data$Percent.Non.Hispanic.African.American)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    0.00    0.01    0.04    0.09    0.12    0.70      67
```


We can compute quartiles of the data:

```r
model_data$AA_quartile <- with(model_data, cut(Percent.Non.Hispanic.African.American, 
                                breaks=quantile(Percent.Non.Hispanic.African.American, probs=seq(0,1, by=0.25), na.rm=TRUE), 
                                include.lowest=TRUE, ordered_result=TRUE, labels=FALSE))
```


Then we can use these quartiles to see that as the Percent African-American increases so does the PM2.5 exposure by county:

```r
AA_summary <- model_data %>% filter(!is.na(Percent.Non.Hispanic.African.American)) %>% group_by(AA_quartile) %>% summarise(Percent_AA = mean(Percent.Non.Hispanic.African.American, na.rm=TRUE), Mean_PM25 = mean(PM25, na.rm=TRUE))

AA_summary
```

```
## # A tibble: 4 × 3
##   AA_quartile Percent_AA Mean_PM25
##         <int>      <dbl>     <dbl>
## 1           1    0.00671      5.97
## 2           2    0.0238       7.03
## 3           3    0.0766       7.93
## 4           4    0.269        8.05
```

Now that we can see this trend, let's add some statistics. Let's specifically compare the relationships between PM2.5 and mortality within the bottom 25% AA counties (quartile 1); and also the highest 25% AA counties (quartile 4):

```r
# First need to subset the data by these quartiles of interest
low_AA <- subset(model_data, AA_quartile==1)
high_AA <- subset(model_data, AA_quartile==4)

# Then we can run the relevant statistical models
m.low <- lm(Age.Adjusted.Rate ~ PM25 + Adult.smoking + Adult.obesity + Food.environment.index + Physical.inactivity +
          High.school.graduation + Some.college + Unemployment + Violent.crime + Percent.Rural + Percent.Females +
          Percent.Asian + Percent.American.Indian.and.Alaskan.Native + Percent.NonHispanic.white,
        data = low_AA, weights=1/low_AA$Age.Adjusted.Rate.Standard.Error)

m.high <- lm(Age.Adjusted.Rate ~ PM25 + Adult.smoking + Adult.obesity + Food.environment.index + Physical.inactivity +
          High.school.graduation + Some.college + Unemployment + Violent.crime + Percent.Rural + Percent.Females +
          Percent.Asian + Percent.American.Indian.and.Alaskan.Native + Percent.NonHispanic.white,
        data = high_AA, weights=1/high_AA$Age.Adjusted.Rate.Standard.Error)


# We see a striking difference in the associations
rbind(c("Bottom 25% AA Counties",round(summary(m.low)$coefficients["PM25",c(1,2,4)],3)),
      c("Top 25% AA Counties",round(summary(m.high)$coefficients["PM25",c(1,2,4)],3)))
```

```
##                               Estimate Std. Error Pr(>|t|)
## [1,] "Bottom 25% AA Counties" "4.782"  "3.895"    "0.222" 
## [2,] "Top 25% AA Counties"    "14.552" "4.13"     "0.001"
```


<br>

:::question
<i>With this, we can answer **Environmental Health Question 4**:</i>
Do observed associations differ when comparing between counties with a higher vs. lower percentage of African-Americans which can indicate environmental justice concerns?
:::
:::answer
**Answer**: Yes. Counties with the highest percentage of African-Americans (top 25%) demonstrated a highly significant association between PM2.5 and age adjusted mortality, even when adjusting for confounders (p=0.001), meaning that the association between PM2.5 and mortality within these counties may be exacerbated by factors relevant to race. Conversely, counties with the lowest percentages of African-Americans (bottom 25%) did not demonstrate a significant association between PM2.5 and age adjusted mortality, indicating that these counties may have lower environmental health risks due to factors correlated with race.
:::

<br>

## Concluding Remarks
In conclusion, this training module serves as a novel example data integration effort of high relevance to environmental health issues. Databases that were evaluated here span exposure data (i.e., Air Quality System data), health outcome data (i.e., mortality data), and county-level characteristics on healthcare, food environment, and other potentially relevant confounders (i.e., county-level variables that may impact observed relationships), and environmental justice data (e.g., race). Many different visualization and statistical approaches were used, largely based on linear regression modeling and county-level characteristic stratifications. These example statistics clearly support the now-established relationship between PM2.5 concentrations in the air and mortality. Importantly, these related methods can be tailored to address new questions to increase our understanding between exposures to chemicals in the environment and adverse health outcomes, as well as the impact of different individual or area charateristics on these relationships - particularly those that might relate to environmental justice concerns.

