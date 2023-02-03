# Axiom-SNParray-Variant-and-Sample-QC-and-visualisation #
This repository provides the basic steps to export genotyping data from the Axiom Analysis Suite as well as it's manipulation using plink and population structure visualisation in the form of PCA's and Admixture plots.

## Axiom Analysis Suite ##

The [Axas](https://www.thermofisher.com/uk/en/home/life-science/microarray-analysis/microarray-analysis-instruments-software-services/microarray-analysis-software/axiom-analysis-suite.html) is a free-access software developed by ThermoFisher Scientific to analyse genotyping data generated by Axiom microarrays. 

## Generating genotype data from Axas ##

Whilst the software provides standard parameters for sample and array QC, it is possible to extract genotypes from all individuals and filter them manually using [plink](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1950838/). For this, genotyping data must be exported in PLINK PED, without specifying thresholds to the sample and variant CR. This will generate two files: a ".map" and ".ped" which must be imported to the HPC for furhter analysis.

## Screening for Sample CR in PLINK ##

Once the files have been imported to the HPC the sample CR can be explored using PLINK.
first, to explore data missingess within individuals and variants, we'll run the command:

```
plink --file (yourfilename) --missing 
```
This command will generate two files: <br />
**.imiss** (individual missingness) <br />
**.lmiss** (variant missingness) <br />

From these two files, you can estimate the average CR of your indivduals/variants as well as identify those that fall under the selected threshold. Whilst the threshold for sample CR is normally > 90%, and variant CR of > 95%, depending on your data and aims of your study, you can select different thresholds.

## Excluding individuals and variants with non-desirable CR ##

Individuals with CR below the selected threshold can be excluded using:
```
plink --file (yourfilename) --remove (textfile) --recode --allow-extra-chr --out (outputfile)
```

the flag **--remove** requires a space/tab-delimited text file with different columns including:  <br />
family IDs - column 1 <br />
within-family IDs - column 2 <br />

the **--recode** flag will generate a generate a new file, excluding the selected genotypes, as PLINK will preserve all genotypes. <br />
the **--allow-extra-chr** flag can be used when chromosome codes are not recognised (e.g. if you're using genomes assembled in a contig level) and their names do not begin with a digit.

```
plink --file (yourfilename) --exclude (textfile) --recode --allow-extra-chr --out (outputfile)
```
The **--exclude** flag will do the same for variants.

## Basic visualusation of population structure with Principal Component Analysis (PCA) ##
### 1. data prep for PCA ###
```
plink --file (yourfilename) --double-id --allow-extra-chr --set-missing-var-ids @:# --indep-pairwise 50 10 0.1 --out (outputfile)
plink --file (yourfilename) --double-id --allow-extra-chr --set-missing-var-ids @:# --extract (outputfile).prune.in --make-bed --pca --out (outputfile)
```
These commands will generate both **.eigenval** and **.eigenvec**  files, which are needed for the PCA.

## PCA analysis in R ##
The subsequent PCA analysis and data visulatisation will be done in [R](https://www.r-project.org/). The **.eigenval** and **.eingenvec** files can be exported to a local computer. Alternatively, R can be opened in the HPC, if available.
### Example of R script ###
```
library(tidyverse)
library(ggplot2)

#upload data
pca <- read_table2("./(yourfilename).eigenvec", col_names = FALSE)
eigenval <- scan("./(yourfilename).eigenval")

#remove first column
pca <- pca[,-1]

#check table
print(pca)

#prep table to name your individuals
names(pca)[2:ncol(pca)] <- paste0("PC", 1:(ncol(pca)-1))

#create a vector to name ind
names(pca)[1] <- "ind"

spp<- rep (NA, length(pca$ind))
spp[grep("Scilly", pca$ind)] <- "Isles_of_Scilly"
spp[grep("Bude", pca$ind)] <- "Bude"
spp[grep("Whistsandbay_HT", pca$ind)] <- "WhistBay_HT"
spp[grep("Whistsandbay_LT", pca$ind)] <- "WhistBay_LT"
spp[grep("Porthcotan_Bay", pca$ind)] <- "Porthcotan_Bay"
spp[grep("Porthleven", pca$ind)] <- "Porthleven"
spp[grep("Trevone", pca$ind)] <- "Trevone"
spp[grep("Polkeris", pca$ind)] <- "Polkeris"
spp[grep("Mevagissey", pca$ind)] <- "Mevagissey"
spp[grep("Mousehole", pca$ind)] <- "Mousehole"
spp[grep("Godrevy", pca$ind)] <- "Godrevy"
spp[grep("St_Anthonys_Head", pca$ind)] <- "St_Anthonys_Head"
spp[grep("Torquay", pca$ind)] <- "Torquay"
spp[grep("Feok", pca$ind)] <- "Feok"
spp[grep("Newquay", pca$ind)] <- "Newquay"
spp[grep("Portmeor", pca$ind)] <- "Portmeor"
spp[grep("Port_lune", pca$ind)] <- "Port_lune"
spp[grep("Tintangel_Castle", pca$ind)] <- "Tintangel_Castle"
spp[grep("Pinksoncreek_", pca$ind)] <- "Camel_estuary"
spp[grep("Budleigh", pca$ind)] <- "Budleigh"
spp[grep("Hallsends_north", pca$ind)] <- "Hallsends_north"
spp[grep("Helford", pca$ind)] <- "Helford"
spp[grep("St_Agnes", pca$ind)] <- "St_Agnes"
spp[grep("Exmouth_LT", pca$ind)] <- "Exmouth_LT"
spp[grep("Port_Gavirn_LT", pca$ind)] <- "Port_Gavirn_LT"
spp[grep("Appledore", pca$ind)] <- "Appledore"
spp[grep("Starcross", pca$ind)] <- "Starcross"
spp[grep("Port_Gavirn_HT", pca$ind)] <- "Port_Gavirn_HT"
spp[grep("Exmouth_HT", pca$ind)] <- "Exmouth_HT"
spp[grep("Millbay_Marina_", pca$ind)] <- "Millbay_Marina"
spp[grep("BBC", pca$ind)] <- "BBC"
spp[grep("DDE", pca$ind)] <- "DDE"
spp[grep("FIN", pca$ind)] <- "FIN"
spp[grep("GR", pca$ind)] <- "GR"
spp[grep("BO", pca$ind)] <- "BO"
spp[grep("IC", pca$ind)] <- "IC"
spp[grep("VG", pca$ind)] <- "VG"


pop<- rep (NA, length(pca$ind))
pop[grep("BBC", pca$ind)] <- "NE Pacific"
pop[grep("BBH", pca$ind)] <- "NE Pacific"
pop[grep("BO", pca$ind)] <- "NE Atlantic"
pop[grep("CAF", pca$ind)] <- "SE Pacific"
pop[grep("CC", pca$ind)] <- "SE Pacific"
pop[grep("CH", pca$ind)] <- "NE Pacific"
pop[grep("CRS", pca$ind)] <- "SE Pacific"
pop[grep("CSI", pca$ind)] <- "SE Pacific"
pop[grep("CV", pca$ind)] <- "SE Pacific"
pop[grep("DDE", pca$ind)] <- "NE Atlantic"
pop[grep("EX", pca$ind)] <- "NE Atlantic"
pop[grep("FAL", pca$ind)] <- "NE Atlantic"
pop[grep("Fin", pca$ind)] <- "Baltic"
pop[grep("GA", pca$ind)] <- "Baltic"
pop[grep("GK", pca$ind)] <- "Baltic"
pop[grep("GR", pca$ind)] <- "Mediterranean"
pop[grep("IC", pca$ind)] <- "NE Atlantic"
pop[grep("LF", pca$ind)] <- "NE Atlantic"
pop[grep("OA", pca$ind)] <- "NE Pacific"
pop[grep("PM", pca$ind)] <- "SE Pacific"
pop[grep("SBH", pca$ind)] <- "NE Pacific"
pop[grep("SW", pca$ind)] <- "NE Atlantic"
pop[grep("VG", pca$ind)] <-  "NE Atlantic"



pva <- as_tibble(data.frame(pca,spp))
pve <- data.frame(PC = 1:20, pve = eigenval/sum(eigenval)*100)
#new_order <- c("BBH", "BBC", "SBH", "OA", "CH", "CV", "CC", "PM", "CAF", "CSI", "CRS",  "DDE", "GR", "VG", "LF
", "EX", "FAL","IC", "BO", "SW", "GK", "GA", "FIN")
#spp <- factor(as.character(spp), levels=new_order)
#spp <- spp[order(spp$pop),]


#pdf("wgs_eigenval.pdf", height = 10, width = 10)
#ggplot(pve, aes(PC, pve)) +  geom_hline(yintercept = 0) + geom_vline(xintercept = 0) + geom_bar(stat = "identi
ty") + ylab("Percentage variance explained") + ggtheme
#dev.off()


cumsum(pve$pve)
## set ggplot2 theme for plotting
#ggtheme = theme(legend.title = element_blank(),axis.text.y = element_text(colour="black", size=15),axis.text.x
 = element_text(colour="black", size=15),axis.title = element_text(colour="black", size=15),legend.position = "
top",legend.text = element_text(size=15),legend.key.size = unit(0.7,"cm"),legend.box.spacing = unit(0, "cm"), p
anel.grid = element_blank(),panel.background = element_blank(),panel.border = element_rect(colour="black", fill
=NA, size=1),plot.title = element_text(hjust=0.5, size=25)) # title centered



# plot pca
pdf("sw-poly-pca.pdf", width = 15, height = 10)
ggplot(pca, aes(PC2, PC3, col= spp, label = spp)) +  geom_hline(yintercept = 0) + geom_vline(xintercept = 0) +
geom_point(size = 3) + xlim(-0.40,0.15) + ylim(-0.15,0.15) + coord_equal() +  geom_text(aes(label=spp), hjust=0
, vjust=0) +  theme_bw() + xlab(paste0("PC1 (", signif(pve$pve[1], 3), "%)")) + ylab(paste0("PC2 (", signif(pve
$pve[2], 3), "%)"))
dev.off()

