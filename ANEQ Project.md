
Try to make directory first. I name it pempek
```
mkdir pempek
cd pempek
```


launch an interactive session: 
```
ainteractive --ntasks=6 --time=02:00:00
```

activate qiime.
```
module purge  
  
module load qiime2/2024.10_amplicon
```

make sub directory
```
`mkdir slurm`

`mkdir taxonomy`

`mkdir tree`

`mkdir taxaplots`

`mkdir dada2`

`mkdir demux`

`mkdir metadata`

`mkdir core_metrics`
```

change to metadata directory
```
cd metadata
```


insert the pempek_metadata_3_26.txt to the folder

rename the metadata
```
mv pempek_metadata_3_26.txt metadata.txt
```

visualize the metadata file
```
qiime metadata tabulate \--m-input-file metadata.txt \--o-visualization metadata.qzv
```

Insert the demux_sr49_pempek_nonasal.qza in the demux directory and rename the file
```
cd demux
mv demux_sr49_pempek_nonasal.qza demux_pempek.qza
```

visualize the demux 
```
qiime demux summarize \--i-data demux_pempek.qza \--o-visualization demux_pempek.qzv
```

change directory to dada2
```
cd ../dada2
```

run the dada2 (denoising)
```

qiime dada2 denoise-paired \--i-demultiplexed-seqs ../demux/demux_pempek.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--p-n-threads 6 \--o-representative-sequences pempek_seqs_dada2.qza \--o-denoising-stats pempek_dada2_stats.qza \--o-table pempek_table_dada2.qza

```

Visualize the dada2
```
#Visualize the denoising results:

qiime metadata tabulate \--m-input-file pempek_dada2_stats.qza \--o-visualization dada2_stats.qzv

qiime feature-table summarize \--i-table pempek_table_dada2.qza \--m-sample-metadata-file ../metadata/metadata.txt \--o-visualization dada2_table.qzv

qiime feature-table tabulate-seqs \--i-data pempek_seqs_dada2.qza \--o-visualization dada2_seqs.qzv

```

Filter sequence longer than 300
```
qiime feature-table filter-seqs \--i-data pempek_seqs_dada2.qza \--m-metadata-file pempek_seqs_dada2.qza \--p-where 'length(sequence) < 300' \--o-filtered-data pempek_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \--i-data pempek_seqs_dada2_filtered300.qza \--o-visualization pempek_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \--i-table pempek_table_dada2.qza \--m-metadata-file pempek_seqs_dada2_filtered300.qza \--o-filtered-table pempek_table_dada2_filtered300.qza
  
qiime feature-table summarize \--i-table pempek_table_dada2_filtered300.qza \--m-sample-metadata-file ../metadata/metadata.txt \--o-visualization pempek_table_dada2_filtered300.qzv
    
```

Change to taxonomy directory
cd taxonomy
run the taxonomy
```
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/pempek_seqs_dada2_filtered300.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2_filtered.qza
```

Visualize the taxonomy
```
qiime metadata tabulate \
--m-input-file taxonomy_gg2_filtered.qza \
--o-visualization taxonomy_gg2_filtered.qzv
```

Filter the chloroplast and mitochondria
```
qiime taxa filter-table \
--i-table ../dada2/pempek_table_dada2_filtered300.qza \
--i-taxonomy taxonomy_gg2_filtered.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

Generate Taxa Barplot
```
qiime taxa barplot \
--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \
--i-taxonomy taxonomy_gg2_filtered.qza \
--m-metadata-file ../metadata/metadata.txt \
--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```

sbatch for phylogenetic tree --> make a pempek.sh at slurm directory
```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=05:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=c837856475@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal

#Activate qiime
#Insert the two commands you need to load qiime2
module purge  
  
module load qiime2/2024.10_amplicon

#Get reference
wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


#Command
qiime fragment-insertion sepp \
--i-representative-sequences ../dada2/pempek_seqs_dada2_filtered300.qza \
--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \
--o-tree ../tree/tree_gg2.qza \
--o-placements ../tree/tree_placements_gg2.qza
```

remove the controls
```
qiime feature-table filter-samples \
--i-table dada2/table_nomitochloro_gg2_filtered300.qza \
--m-metadata-file metadata/metadata.txt \
--p-where "NOT [sample_type] IN ('control') " \
--o-filtered-table dada2/table_nomitochloro_nocontrol.qza
```

```
sbatch pempek.sh
```

visualise without control
```
qiime taxa barplot \
--i-table ../dada2/table_nomitochloro_nocontrol.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered.qza \
--m-metadata-file ../metadata/metadata.txt \
--o-visualization table_nomitochloro_nocontrol.qzv
```

Make directiory rarefaction
```
mkdir alpha_rarefaction

cd alpha_rarefaction

```

Run rarefaction (check for depth)
```
qiime diversity alpha-rarefaction \
--i-table ../dada2/table_nomitochloro_nocontrol.qza \
--m-metadata-file ../metadata/metadata.txt \
--p-min-depth 50 \
--p-max-depth 66000 \
--o-visualization alpha_rarefaction_curves_16S.qzv \
```

Run core metric (check depth)
```
cd ../

qiime diversity core-metrics-phylogenetic \
--i-table dada2/table_nomitochloro_nocontrol.qza \
--i-phylogeny tree/tree_gg2.qza \
--m-metadata-file metadata/metadata.txt \
--p-sampling-depth 7000 \
--output-dir core_metrics_results
```

Visualize observe features
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/observed_features_vector.qza \
--m-metadata-file metadata/metadata.txt \
--o-visualization core_metrics_results/observed_features_statistics.qzv
```

generate a plot to visualize
``` 
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/shannon_vector.qza \
--m-metadata-file metadata/metadata.txt \
--o-visualization core_metrics_results/shannon_statistics.qzv  
  
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \
--m-metadata-file metadata/metadata.txt \
--o-visualization core_metrics_results/faiths_pd_statistics.qzv

qiime diversity alpha-correlation \
--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \
--m-metadata-file metadata/metadata.txt \
--o-visualization core_metrics_results/faith_pd_correlation_statistics.qzv

```

Longitudinal (Volatility)
```
mkdir longitudinal

cd longitudinal

qiime longitudinal volatility \
--m-metadata-file ../metadata/metadata.txt \
--m-metadata-file ../core_metrics_results/weighted_unifrac_pcoa_results.qza \
--p-state-column day \
--p-individual-id-column calf_id \
--p-default-group-column 'sample_type' \
--p-default-metric 'Axis 2' \
--o-visualization pc_vol_sample_type.qzv
```

Export
```
cd ../

mkdir export

unzip core_metrics_results/shannon_vector.qza -d export/shannon

cd export 

ls

cd shannon

ls

cd */data

ls

cd ../ # back to pempek
```
```

# Observed Features

unzip core_metrics_results/observed_features_vector.qza -d export/observed_features

# Faith's PD

unzip core_metrics_results/faith_pd_vector.qza -d export/faith_pd

# Pielou's evenness

unzip core_metrics_results/evenness_vector.qza -d export/evenness

```

Export transpose
```
cd dada2

qiime feature-table transpose \
--i-table table_nomitochloro_gg2_filtered300.qza \
--o-transposed-feature-table table_nomitochloro_transposed.qza

qiime metadata tabulate \
--m-input-file table_nomitochloro_transposed.qza \
--m-input-file pempek_seqs_dada2.qza \
--m-input-file ../taxonomy/taxonomy_gg2_filtered.qza \
--o-visualization tabulated_results.qzv
```

Beta Metrics
```
# Bray Curtis

unzip core_metrics_results/bray_curtis_pcoa_results.qza -d export/bray_curtis

# Jaccard

unzip core_metrics_results/jaccard_pcoa_results.qza -d export/jaccard

# Unweighted Unifrac

unzip core_metrics_results/unweighted_unifrac_pcoa_results.qza -d export/unweighted_unifrac

# Weighted Unifrac

unzip core_metrics_results/weighted_unifrac_pcoa_results.qza -d export/weighted_unifrac
```

```
cd export

mkdir alpha_div

# define alpha metrics

metrics=("shannon" "evenness" "faith_pd" "observed_features")

# copy their tsv files into alpha_div/

for metric in "${metrics[@]}"; do

 cp $metric/*/data/alpha-diversity.tsv alpha_div/${metric}.tsv

done
```

```
mkdir beta_div

# define beta metrics

metrics=("bray_curtis" "jaccard" "unweighted_unifrac" "weighted_unifrac")

# copy their txt files into beta_div/

for metric in "${metrics[@]}"; do

 cp $metric/*/data/ordination.txt beta_div/${metric}.txt

done
```

ANCOM BC2
```
cd ../

module purge

module load qiime2/2026.1_amplicon

```

```
mkdir ancombc2

cd ancombc2

qiime feature-table filter-samples \
--i-table ../dada2/table_nomitochloro_nocontrol.qza \
--p-min-frequency 7000 \
--o-filtered-table table_nomitochloro_7000.qza
```

```
qiime feature-table filter-features \
--i-table table_nomitochloro_7000.qza \
--p-min-frequency 50 \
--p-min-samples 4 \
--o-filtered-table table_nomitochloro_7000_abund.qza
```

```
qiime taxa collapse \
--i-table table_nomitochloro_7000_abund.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered.qza \
--p-level 7 \
--o-collapsed-table table_nomitochloro_7000_abund_L7.qza
```


STILL FAILED. (the metadata should be have no controls.)
```
ANCOMBC USING NEW METADATA
qiime composition ancombc2 \
--i-table table_nomitochloro_7000_abund_L7.qza \
--m-metadata-file pempek_metadata_noEC.txt \
--p-fixed-effects-formula 'treatment + age_w' \
--p-reference-levels treatment::pair age_w::birth \
--p-random-effects-formula '(1 | calf_number)' \
--o-ancombc2-output ancombc2_sampletype_day_treatment_L7.qza

qiime composition tabulate \
--i-data ancombc2_sampletype_day_L7.qza \
--o-visualization ancombc2_sampletype_day_L7.qzv

qiime composition ancombc2-visualizer \
--i-data ancombc2_sampletype_day_L7.qza \
--o-visualization ancombc2_barplot_sampletype_day_L7.qzv
```

ML treatment
```
cd pempek

mkdir ml

cd ml

qiime taxa collapse \
--i-table ../core_metrics_results/rarefied_table.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered.qza \
--p-level 7 \
--o-collapsed-table rare_table_L7.qza
```

```
qiime sample-classifier classify-samples \
--i-table rare_table_L7.qza \
--m-metadata-file ../metadata/metadata.txt \
--m-metadata-column treatment \
--p-random-state 123 \
--p-n-jobs 1 \
--output-dir sample_classifier_results_treatment
```

ML age (days)

```
qiime sample-classifier classify-samples \
--i-table rare_table_L7.qza \
--m-metadata-file ../metadata/metadata.txt \
--m-metadata-column age_d \
--p-random-state 123 \
--p-n-jobs 1 \
--output-dir sample_classifier_results_age