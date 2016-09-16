---

layout: default
title: Exome Sequencing Analysis using Galaxy - Advanced
---

Advanced Exome Analysis using Galaxy
===

## Aims

In this practical you will use several additional features not covered in the previous sessions. This will help you to:

 * check whether your favourite gene is covered by your exome capture kit
 * perform an accurate quality control of your aligned reads
 * quickly examine your VCF file
 * flag low quality variants
 * use SnpEFF for variant annotation
 * annotate variants with your custom annotations (e.g. clinical significance from ClinVar, allele frequency from your reference paned)
 * focus on regions of interest for consanguineous parents

## Before starting

Using the **Copy datasets** function, copy the following datasets to a new history:

 * Pedigree
 * NexteraRapidCaptureExpandedExome_Target.hg19.chr8.padding200.bed
 * BAMs used as input for Unified Genotyper (output of **Print reads**)
 * VCF from **Unified Genotyper**

### Check the coverage of your favourite genes

* Download **NexteraRapidCaptureExpandedExome_Target.hg19.chr8.padding200.bed** from your Galaxy history.
  Please note that this file only contains target regions on chr8, with 200bp padding.
* Go to [UCSC Genome Browser](http://genome.ucsc.edu/cgi-bin/hgTracks?db=hg19&position=lastDbPos) and select **Add/manage custom tracks**
* Upload your BED file

This check can be performed before running the experiment as it only requires a BED files containing the regions covered by the Exome Capture Kit.
These files are freely available from the vendor sites. 

**Question**:

* You are strongly interested in PPP2R2A gene. How is the coverage of the different isoforms?

### Quality control of aligned reads

 * Run **BAM QC using QPLOT** on your 3 BAM files at the same time. Provide a **label** (i.e. `father`, `mother` or `proband`)
   for each input file. For more info see the QPLOT [website](http://genome.sph.umich.edu/wiki/QPLOT).

**Questions**:

* Were most reads properly paired?
* Did you notice any GC bias in sequencing depth? Check both pdf and stats (GCBiasMSE, GC Bias Mean Squared Error) from QPLOT.

### Quickly examine your VCF and BAM file

The next Galaxy release will include two direct links to quickly display VCF and BAM file at [iobio.io](http://iobio.io/applications.html), a platform for immediate visual feedback of complex genomic datasets.

In the meantime, you can display your data at iobio.io as follows:

* Send your VCF or BAM file to UCSC using the **display at UCSC main** link
* Go to UCSC, and select **manage custom tracks** button
* Click on the dataset name and copy the URL of the dataset (e.g. http:<i></i>//90.147.170.113/display_application/ddf83cf807e6e774/ucsc_vcf/main/23701f153218a357/data/galaxy_ddf83cf807e6e774.vcf.gz)
* Go to [vcf.iobio.io](http://vcf.iobio.io/) or [bam.iobio.io](http://bam.iobio.io/) and paste the URL


### Flag low quality variants

The aim of this step is to reduce the false positive calls by identifying low quality variants.
The best solution is to apply GATK Variant Quality Score Recalibration. If it cannot be applied (e.g. for small sample sets), low
quality variants can be flagged using the following criteria, according to GATK Best Practices (aka GATK hard-filtering):

**Filters for SNPS:**

 * QualByDepth < 2.0
 * RMSMappingQuality < 40.0
 * FS > 60.0
 * HaplotypeScore > 13.0
 * MQRankSum < -12.5
 * ReadPosRankSum < -8.0

**Filters for INDELs**:

 * QualByDepth < 2.0
 * FS > 200.0 
 * ReadPosRankSum <  -20.0 

Note that you need to apply different filters to SNPs and INDELs.
Browse the **Published workflows** section and run **GATK Hard Filters**. Edit the workflow to inspect the different
sections and execute. The output now includes different variants whose value in column FILTER is different from PASS:
these variants are considered as low-quality variants and are assigned a low priority.

**Question**:

 * How many variants are flagged as PASS? You can aggregate and count the variants by the flag in column FILTER with
**Join, Subtract and Group -> Group** as follows:
  * Select data: VCF output of the workflow
  * Group by column: select the value (c1, 1st column; c2, 2nd column; ...) corresponding to the column FILTER
  * Ignore lines beginning with these characters: # (skip header lines)
  * Operations: `count` on column `[value corresponding to the column FILTER]`, `do not round results`.

### Annotations with SnpEFF

**SnpEFF Variant effect and annotation** is a popular tool for the annotation of VCF files.
This will populate the INFO column of your file with the new annotations, and the header of the VCF with a short description.

**Question**:

* Can you find the new annotations in your output VCF files?

### Annotate with your internal resources

To annotate your VCF with info extracted from internal resources, i.e. allele frequency from a reference population,
you can run **GATK Variant annotator**. Briefly, it takes a VCF as input and adds the annotations extracted from
the INFO column of multiple VCF files.

Let's assume you want to annotate which variants in your set are present in NCBI ClinVar,
a database of variants of clinical relevance. To do that, execute **GATK Variant annotator** as follows:

 * Copy to your current history **clinvar_20160831.hg19.vcf** (from **Data Libraries -> Training -> Input**)
 * Run **GATK Variant Annotator** with the following parameters:
 
   * Choose the reference genome from the history: `hg19_chr8.fa`
   * Variant file to annotate: your VCF from Unified Genotyper
   * Provide a dbSNP Reference-Ordered Data (ROD) file: **don't set dbSNP** (reduces the computation time)
   * Binding for reference-ordered resource data:

     * ROD file: `clinvar_YYYYMMDD_hg19.vcf`
     * ROD name: a shortname to be used in the next step (no spaces allowed) - `clinvar`

   * Expressions: to annotate with the CLNSIG (Variant Clinical Significance, from 0 to 7) and CLNDBN (Variant disease name) parameters from ClinVar, enter the two following expressions:

     * `clinvar.CLNSIG`
     * `clinvar.CLNDBN`
     
   * Choose the bed file with target regions: add `NexteraRapidCaptureExpandedExome_Target.hg19.chr8.padding200.bed` in 
    **Advanced GATK options -> Operate on genomic interval**

If you want to export the final VCF in a Excel-compatible file, run the **VCFtoTab-delimited** tool.

### Runs of Homozygosity

Identification of Runs of Homozygosity (RoH) is a strategy to limit the search for candidate genes to specific chromosomal
regions in consanguineous families. You can identify RoH in your family with the following tools:

 * RoH with **H3M2** - Input: BAM from the proband
 * KGGSeq - Input: VCF and pedigree. Open section **Specify homozygosity filters**, and enter a value (i.e. `10`, corresponding to 10Kb) for **Filter by Runs of Homozygosity (ROH)**.
   The software will return only the variants located in a RoH with length greater than this value. In the **tabular** output the last two columns contain the number of SNPs and length of the RoH. 

### bedtools

The [bedtools](http://bedtools.readthedocs.io/en/latest/) utilities are a suite of tools for a wide-range of operations with genomic data. Using bedtools you can, for example, intersect, merge and count genomic intervals from files in different formats such as BAM, BED and VCF.

**Question**:

 * Using **MultiCovBed** tool, count the number of reads mapped to ERICH1 gene
