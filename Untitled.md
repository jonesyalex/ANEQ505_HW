



qiime dada2 denoise-paired \ --i-demultiplexed-seqs ../demux/demux_cow.qza \--p-trim-left-f 0 \--p-trim-left-r 0 \--p-trunc-len-f 250 \--p-trunc-len-r 250 \--p-n-threads 6 \--o-representative-sequences cow_seqs_dada2.qza \  
--o-denoising-stats cow_dada2_stats.qza \  
--o-table cow_table_dada2.qza