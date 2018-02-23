#Qiime 2 script - 2 nov 2017



#Open Qiime2
```{r}
source activate qiime2-2017.10
qiime
```

# Import paired-end demultiplexed fastq
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path Run3 \
  --source-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path -Run3-61-156-demux-paired-end.qza


qiime demux summarize \
  --i-data -Run3-61-156-demux-paired-end.qza \
  --o-visualization -Run3-61-156-demux-paired-end.qzv

##qiime tools view https://view.qiime2.dorg

# Dada2 
##Start session in a screen (tmux) - (tmux detach    or Detach       ^b d)
tmux new -s marie
tmux att -t marie

##Run dada2
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs PeriphytonQ2-demux-paired-end.qza \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 0 \
  --p-trim-left-r 32 \
  --o-representative-sequences PeriphytonQ2-Run2-dada2.qza \
  --o-table PeriphytonQ2-Run2-table-dada2.qza \
  --p-n-threads 0 \
  --verbose


##Summary
qiime feature-table summarize \
  --i-table Simonin-2018-Run1-table-dada2.qza \
  --o-visualization Simonin-2018-Run1-visualization-table-dada2.qzv \
  --m-sample-metadata-file Simonin-2018-Run1-metadata.tsv

qiime feature-table tabulate-seqs \
  --i-data Simonin-2018-Run1-dada2.qza \
  --o-visualization Simonin-2018-Run1-dada2-rep-seqs.qzv

##Rarefy
qiime feature-table rarefy
  --i-table FlocQ2Q3-table-dada2.qza \
  --p-sampling-depth 10000 \
  --o-rarefied-table FlocQ2Q3-table-dada2-rarefied.qza

##Export
qiime tools export \
  FlocQ2Q3-dada2.qza \
  --output-dir FlocQ2Q3-dada2_rep_seqs

qiime tools export \
  FlocQ2Q3-table-dada2.qza \
  --output-dir FlocQ2Q3_table

qiime tools export \
  FlocQ2Q3-table-dada2-rarefied.qza \
  --output-dir FlocQ2Q3-table-dada2-rarefied


##To regroup different processed datasets - Merging denoised datasets
qiime feature-table merge \
  --i-table1 Run1-joined-filtered-table-deblur.qza \
  --i-table2 Perrotta-joined-filtered-table-deblur.qza \
  --o-merged-table merged_table_Run1_Perrota.qza


qiime feature-table merge-seq-data \
  --i-data1 Run1-joined-filtered-rep-seqs-deblur.qza \
  --i-data2 Perrotta-joined-filtered-rep-seqs-deblur.qza \
  --o-merged-data merged_rep-seqs_Run1_Perrotta.qza

##Export
qiime tools export \
  merged_table_Run1_Perrota.qza \
  --output-dir merged_table_Run1_Perrota-deblur

qiime tools export \
  merged_rep-seqs_Run1_Perrotta.qza \
  --output-dir merged_rep-seqs_Run1_Perrotta-deblur




##Make Summary
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv

  qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv

######################################################################################################



# Deblur - Sequences need to be paired first

## In Qiime1
multiple_join_paired_ends.py -i Run4bis -o Run4-157-259_PairedEndJoined

gzip M*

##In Qiime 2 Import & Filter & Deblur
source activate qiime2-2017.10
qiime


tmux new -s marie

tmux att -t marie

tmux kill-session -t marie

##Import
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path Run4A_pairedend \
  --source-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path Run4A-joined.qza 

qiime demux summarize \
  --i-data Run3-4-joined.qza \
  --o-visualization Run3-4-joined.qzv    

##Quality filter
qiime quality-filter q-score \
 --i-demux Run4A-joined.qza \
 --o-filtered-sequences Run4A-joined-filtered.qza \
 --o-filter-stats Run4A-joined-filtered-stats.qza \
  --verbose

qiime quality-filter visualize-stats \
  --i-filter-stats Run4A-joined-filtered-stats.qza \
  --o-visualization Run4A-joined-filtered-stats.qzv

## Trying to trim at 260 because (-1 no trimming) had an error
qiime deblur denoise-16S \
  --i-demultiplexed-seqs Run4A-joined-filtered.qza \
  --p-trim-length 260 \
  --o-representative-sequences Run4A-joined-filtered-rep-seqs-deblur.qza \
  --o-table Run4A-joined-filtered-table-deblur.qza \
  --p-sample-stats \
  --o-stats Run4A-joined-filtered-deblur-stats.qza \
  --verbose


qiime deblur visualize-stats \
  --i-deblur-stats Run4A-joined-filtered-deblur-stats.qza \
  --o-visualization Run4A-joined-filtered-deblur-stats.qzv

##Export
qiime tools export \
  Run3-4-joined-filtered-rep-seqs-deblur.qza \
  --output-dir Run3-4-joined-filtered-rep-seqs-deblur

qiime tools export \
  Run3-4-joined-filtered-table-deblur.qza \
  --output-dir Run3-4-joined-filtered-table-deblur

##convert biom file in txt file
biom convert -i feature-table.biom -o Run3-4-Deblur-SV-table.txt --to-tsv --table-type="OTU table"


##To regroup different processed datasets - Merging denoised datasets
qiime feature-table merge \
  --i-table1 Run4A-joined-filtered-table-deblur.qza \
  --i-table2 Run4B-joined-filtered-table-deblur.qza \
  --o-merged-table Run4all-joined-filtered-table-deblur.qza

qiime feature-table merge-seq-data \
  --i-data1 Run4A-joined-filtered-rep-seqs-deblur.qza \
  --i-data2 Run4B-joined-filtered-rep-seqs-deblur.qza \
  --o-merged-data Run4all-joined-filtered-rep-seqs-deblur.qza


###Combine Run 3 and 4
qiime feature-table merge \
  --i-table1 Run4all-joined-filtered-table-deblur.qza \
  --i-table2 Run3-joined-filtered-table-deblur.qza \
  --o-merged-table Run3-4-joined-filtered-table-deblur.qza

qiime feature-table merge-seq-data \
  --i-data1 Run4all-joined-filtered-rep-seqs-deblur.qza \
  --i-data2 Run3-joined-filtered-rep-seqs-deblur.qza \
  --o-merged-data Run3-4-joined-filtered-rep-seqs-deblur.qza

##Combine Run 3-4 and 1
qiime feature-table merge \
  --i-table1 Run3-4-joined-filtered-table-deblur.qza \
  --i-table2 Run1-joined-filtered-table-deblur.qza \
  --o-merged-table Run1-3-4-joined-filtered-table-deblur.qza

qiime feature-table merge-seq-data \
  --i-data1 Run3-4-joined-filtered-rep-seqs-deblur.qza \
  --i-data2 Run1-joined-filtered-rep-seqs-deblur.qza \
  --o-merged-data Run1-3-4-joined-filtered-rep-seqs-deblur.qza


##Summary
qiime feature-table summarize \
  --i-table Run1-3-4-joined-filtered-table-deblur.qza \
  --o-visualization Run1-3-4-joined-filtered-table-deblur.qzv \
  --m-sample-metadata-file Simonin-2018-Run1-3-4-metadata.tsv

qiime feature-table tabulate-seqs \
  --i-data Run1-3-4-joined-filtered-rep-seqs-deblur.qza \
  --o-visualization Run1-3-4-joined-filtered-rep-seqs-deblur.qzv

## Subsample a Biom table in Qiime1 based on a list of sample ID
filter_samples_from_otu_table.py -i Run1-3-4-SVtable.biom -o Nanocosm_only_SVtable.biom --sample_id_fp sample-id.txt --negate_sample_id_fp



## Rarefy in Qiime2
qiime feature-table rarefy \
  --i-table Run1-3-4-joined-filtered-table-deblur.qza \
  --p-sampling-depth 2887 \
  --o-rarefied-table Run1-3-4-joined-filtered-table-deblur-rarefied.qza

## Rarefy in Qiime1
single_rarefaction.py -i Nanocosm_only_SVtable.biom -o Nanocosm_only_SVtable-rarefied2887.biom -d 2887



qiime tools export \
  Run1-3-4-joined-filtered-rep-seqs-deblur.qza \
  --output-dir Run1-3-4-joined-filtered-rep-seqs-deblur

qiime tools export \
  Run1-3-4-joined-filtered-table-deblur.qza \
  --output-dir Run1-3-4-joined-filtered-table-deblur


qiime tools export \
  Run1-3-4-joined-filtered-table-deblur-rarefied.qza \
  --output-dir Run1-3-4-joined-filtered-table-deblur-rarefied2887

##convert biom file in txt file
biom convert -i Nanocosm_only_SVtable-rarefied2887.biom -o Nanocosm_only_SVtable-rarefied2887.txt --to-tsv --table-type="OTU table"




#Make tree in Qiime1
make_phylogeny.py -i Merge_final_alignment.fasta -t fasttree -o Merge_final.tre


# In R for microbial community analysis

## On an Amazon Instance
Installation in AWS for R:
sudo apt-get install libcairo2-dev
sudo apt install gfortran

## Open SV table in R
SV2 <- read.csv('Merge_rarefied_2000.csv', header=TRUE)
SVt2 <- as.data.frame(t(SV2))
write.csv(SVt2, "Merge_rarefied_2000_transposed.csv")
SVt2 <- read.csv('Merge_rarefied_2000_transposed.csv', skip=1, header=TRUE, check.names=F)
meta=read.csv('Prelim_Map_corrected.csv')
SVmeta <- merge(meta, SVt2, by="X.OTU.ID")
matrix<-SVt2[c(2:46336)]
matrixU=matrix[ , !names(matrix) %in% c("26598f2696a3d194c60e64460461c45c","3fb1e71a8e4b45671bfda485076b6285","6605dfa2f92b283dda21518aa22405a1","8baf35d49f99b843e8d867c241eff050", "a94b259b67a7e6e3422ffcec2e902463", "bd36ba5b48add586efdbcddbf1ad2a9a")]

#Distance matrix Unifrac
unifracs <- GUniFrac(matrixU, tree, alpha=c(0, 0.5, 1))$unifracs


# Taxonomy in Qiime2
##Assign Taxonomy: feature classifier (need SILVA or Greengene rep seq + taxonomy file + dataset rep seq.qza)

###Import reference files
qiime tools import \
  --type 'FeatureData[Sequence]' \
  --input-path 99_otus_16S.fasta \
  --output-path 99_otus_16S.qza

qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --source-format HeaderlessTSVTaxonomyFormat \
  --input-path consensus_taxonomy_7_levels.txt \
  --output-path ref-taxonomy.qza

### Extract reference reads based on primers to work on the same 16S fragment
qiime feature-classifier extract-reads \
  --i-sequences 85_otus.qza \
  --p-f-primer GTGCCAGCMGCCGCGGTAA \
  --p-r-primer GGACTACHVGGGTWTCTAAT \
  --p-trunc-len 100 \
  --o-reads ref-seqs.qza

### Train the classifier
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads ref-seqs.qza \
  --i-reference-taxonomy ref-taxonomy.qza \
  --o-classifier classifier.qza

### test the classifier
qiime feature-classifier classify-sklearn \
  --i-classifier classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv

## Assign taxonomy with Qiime1
assign_taxonomy.py -i repr_set_seqs.fasta -r 99_otus_16S.fasta -t consensus_taxonomy_7_levels.txt








#### Debug Deblur
# To find extra space or missing quality scores:
for f in *.fastq; do r=$(( $(wc -l < $f | tr -d '[:space:]') % 4 )); echo $r $f; done



