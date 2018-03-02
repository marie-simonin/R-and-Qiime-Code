# 16S amplicon sequencing analysis with Qiime2 
# Dada2 and Deblur 


## Open Qiime2 (here for version 2017.10)
```{r}
source activate qiime2-2017.10
qiime
```

# Deblur 

## Import demultiplexed fastq (sequences not joined)
### Files should be under this format (Casava): sampleID_S50_L001_R1_001.fastq.gz
```{r}
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path Raw-data \
  --source-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path demux.qza
```
```{r}
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
```
## Visualize the qzv file on qiime tools view: https://view.qiime2.org/

## Join reads: - Sequences need to be paired first
```{r}
qiime vsearch join-pairs \
  --i-demultiplexed-seqs demux.qza \
  --o-joined-sequences demux-joined.qza
```

## or In Qiime1
```{r}
macqiime
multiple_join_paired_ends.py -i Raw-data -o Sequences_PairedEndJoined
```

### (Ignore if the pairing was done in Qiime2) Rename the joined fastQ files in the Casava format (Ex: sampleID_S50_L001_R1_001.fastq) and then gunzip the files
```{r}
gzip M*
```

### (Ignore if the pairing was done in Qiime2) In Qiime 2: Import & Filter & Deblur (if the joining was done in Qiime1)

```{r}
qiime
```
### (Ignore if the pairing was done in Qiime2) Import joined fastq files in Qiime 2 
```{r}
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path Sequences_PairedEndJoined \
  --source-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path demux-joined.qza 
```
```{r}
qiime demux summarize \
  --i-data demux-joined.qza \
  --o-visualization demux-joined.qzv 
```   
### (Ignore if the pairing was done in Qiime2) Visualize the qzv file on qiime tools view: https://view.qiime2.org/




## Quality filtering
```{r}
qiime quality-filter q-score \
 --i-demux demux-joined.qza \
 --o-filtered-sequences demux-joined-filtered.qza \
 --o-filter-stats demux-joined-filtered-stats.qza \
  --verbose
``` 
```{r}
qiime quality-filter visualize-stats \
  --i-filter-stats demux-joined-filtered-stats.qza \
  --o-visualization demux-joined-filtered-stats.qzv
``` 
## Visualize the qzv file on qiime tools view: https://view.qiime2.org/

##  Run Deblur, sequences trimmed at 260 bp long 
```{r}
qiime deblur denoise-16S \
  --i-demultiplexed-seqs demux-joined-filtered.qza \
  --p-trim-length 260 \
  --o-representative-sequences demux-joined-filtered-rep-seqs-deblur.qza \
  --o-table demux-joined-filtered-table-deblur.qza \
  --p-sample-stats \
  --o-stats demux-joined-filtered-deblur-stats.qza \
  --verbose
```

```{r}
qiime deblur visualize-stats \
  --i-deblur-stats demux-joined-filtered-deblur-stats.qza \
  --o-visualization demux-joined-filtered-deblur-stats.qzv
```

## Export SV table and representative sequences
```{r}
qiime tools export \
  demux-joined-filtered-rep-seqs-deblur.qza \
  --output-dir Deblur-output
```
```{r}
qiime tools export \
  demux-joined-filtered-table-deblur.qza \
  --output-dir Deblur-output
```

## In Qiime 2: To regroup different processed datasets - Merging denoised datasets (SV tables and representative sequences)
```{r}
qiime feature-table merge \
  --i-table1 demux-joined-filtered-table-deblur.qza \
  --i-table2 demux2-joined-filtered-table-deblur.qza \
  --o-merged-table Merged-joined-filtered-table-deblur.qza
```
```{r}
qiime feature-table merge-seq-data \
  --i-data1 demux-joined-filtered-rep-seqs-deblur.qza \
  --i-data2 demux-joined-filtered-rep-seqs-deblur.qza \
  --o-merged-data Merged-joined-filtered-rep-seqs-deblur.qza
```

## Summary
```{r}
qiime feature-table summarize \
  --i-table Merged-joined-filtered-table-deblur.qza \
  --o-visualization Merged-joined-filtered-table-deblur.qzv \
  --m-sample-metadata-file metadata.tsv
```
```{r}
qiime feature-table tabulate-seqs \
  --i-data Merged-joined-filtered-rep-seqs-deblur.qza \
  --o-visualization Merged-joined-filtered-rep-seqs-deblur.qzv
```


## Rarefy in Qiime2
```{r}
qiime feature-table rarefy \
  --i-table Merged-joined-filtered-table-deblur.qza \
  --p-sampling-depth 10000 \
  --o-rarefied-table Merged-joined-filtered-table-deblur-rarefied-10000.qza
```


## Export Merged dataset
```{r}
qiime tools export \
  Merged-joined-filtered-rep-seqs-deblur.qza \
  --output-dir Merged-deblur
```
```{r}
qiime tools export \
  Merged-joined-filtered-table-deblur.qza \
  --output-dir Merged-deblur
```
```{r}
qiime tools export \
  Merged-joined-filtered-table-deblur-rarefied-10000.qza \
  --output-dir Merged-deblur
```

## convert biom file in txt file in Qiime 1
```{r}
macqiime
biom convert -i feature-table.biom -o Merged-Deblur-SV-table.txt --to-tsv --table-type="OTU table"
```



---

# DADA2


## Import demultiplexed fastq (sequences not joined)
### Files should be under this format (Casava): sampleID_S50_L001_R1_001.fastq.gz
```{r}
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path Raw-data \
  --source-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path demux.qza
```
```{r}
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
```
## Visualize the qzv file on qiime tools view: https://view.qiime2.org/

# Dada2 
## If running on a server, start session in a screen (tmux) 
```{r}
tmux new -s dada
tmux att -t dada
```

## Run dada2
```{r}
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \
  --p-trunc-len-f 0 \
  --p-trunc-len-r 0 \
  --p-trim-left-r 32 \
  --o-representative-sequences demux-rep-seq-dada2.qza \
  --o-table demux-table-dada2.qza \
  --p-n-threads 0 \
  --verbose
```

## Summary
```{r}
qiime feature-table summarize \
  --i-table demux-table-dada2.qza \
  --o-visualization demux-table-dada2.qzv \
  --m-sample-metadata-file metadata.tsv
```
```{r}
qiime feature-table tabulate-seqs \
  --i-data demux-rep-seq-dada2.qza \
  --o-visualization demux-rep-seq-dada2.qzv
```

## Rarefy
```{r}
qiime feature-table rarefy
  --i-table demux-table-dada2.qza \
  --p-sampling-depth 10000 \
  --o-rarefied-table demux-table-dada2-rarefied-10000.qza
```

## Export SV table (biom table) and representative sequences (txt file)
```{r}
qiime tools export \
  demux-table-dada2.qza \
  --output-dir Dada2-output
```
```{r}
qiime tools export \
  demux-rep-seq-dada2.qza \
  --output-dir Dada2-output
```
```{r}
qiime tools export \
  demux-table-dada2-rarefied-10000.qza \
  --output-dir Dada2-output
```
## convert biom file in txt file in Qiime 1
```{r}
macqiime
biom convert -i feature-table.biom -o Dada2-SV-table.txt --to-tsv --table-type="OTU table"
```

## To regroup different processed datasets - Merging denoised datasets
```{r}
qiime feature-table merge \
  --i-table1 demux-table-dada2.qza \
  --i-table2 demux2-table-dada2.qza \
  --o-merged-table merged_table_dada2.qza
```
```{r}
qiime feature-table merge-seq-data \
  --i-data1 demux-rep-seq-dada2.qza \
  --i-data2 demux2-rep-seq-dada2.qza \
  --o-merged-data merged_rep-seq-dada2.qza
```



# Misc in Qiime 1
## Subsample a Biom table in Qiime1 based on a list of sample ID
```{r}
filter_samples_from_otu_table.py -i feature-table.biom -o feature-table-filtered.biom --sample_id_fp sample-id.txt --negate_sample_id_fp
```

## Make tree in Qiime1
```{r}
make_phylogeny.py -i Merge_final_alignment.fasta -t fasttree -o Merge_final.tre
```
##  Rarefy in Qiime1
```{r}
single_rarefaction.py -i feature-table.biom -o Deblur-SVtable-rarefied-10000.biom -d 10000
```


# Taxonomy in Qiime2 or Qiime 1

## In Qiime 2 - Assign Taxonomy: feature classifier (need SILVA or Greengene rep seq + taxonomy file + dataset rep seq.qza)
### Import reference files from Silva 128
```{r}
qiime tools import \
  --type 'FeatureData[Sequence]' \
  --input-path 99_otus_16S.fasta \
  --output-path 99_otus_16S.qza
```
```{r}
qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --source-format HeaderlessTSVTaxonomyFormat \
  --input-path consensus_taxonomy_7_levels.txt \
  --output-path ref-taxonomy.qza
```

### Extract reference reads based on primers to work on the same 16S fragment
```{r}
qiime feature-classifier extract-reads \
  --i-sequences 99_otus_16S.qza \
  --p-f-primer GTGYCAGCMGCCGCGGTAA \
  --p-r-primer GGACTACNVGGGTWTCTAAT \
  --p-trunc-len 260 \
  --o-reads reference-seqs.qza
```

### Train the classifier
```{r}
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads reference-seqs.qza \
  --i-reference-taxonomy ref-taxonomy.qza \
  --o-classifier classifier.qza
```
### test the classifier and assign the taxonomy
```{r}
qiime feature-classifier classify-sklearn \
  --i-classifier classifier.qza \
  --i-reads Merged-joined-filtered-rep-seqs-deblur.qza \
  --o-classification taxonomy.qza
```
```{r}
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```

## Assign taxonomy with Qiime1
```{r}
assign_taxonomy.py -i repr_set_seqs.fasta -r 99_otus_16S.fasta -t consensus_taxonomy_7_levels.txt
```


## Debug Deblur or Quality filtering
### To find extra space or missing quality scores in fastq files (in the terminal):
### Problem if a 0 is not returned
```{r}
for f in *.fastq; do r=$(( $(wc -l < $f | tr -d '[:space:]') % 4 )); echo $r $f; done
```


