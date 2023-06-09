## GEN 711 project 
Fecal Microbiota Transplant

Sophie Marek, Grace Drew, Casey Baumann 

## Background
Fecal microbiota transplantation is a common procedure used to treat patients suffering from C. diff which consists of an imbalance of normal healthy gut bacteria allowing for the C. diff bacteria population to increase causing symptoms of diarrhea, upset stomach, and signs of dehydration. Healthy bacteria from feces are collected from a carefully screened donor attemping to match the recipients lifestyle to the donor. The collected microbiota is then transferred to the recipient orally or rectally. The bacteria will restore healthy levels of bacteria in the lower intestine and therefore improve metabolic functions.

There is a link to gastrointestinal issues and autism spectrum disorder. The exact cause of ASD is not known but a correlation has been shown to ASD severity and GI symptoms. Previous studies include imporovement of patients' ASD and GI sympotoms after antibiotic treatment, as well as a mouse-model study which linked abnormal metabolism and behavior to the gut microbiota.

This specific study was an open-label clinical trial where both the patients and providers were aware of the treatment being administered. Some patients were either treated with microbiota treatment therapy or not and their gut microbiota was analyzed.

(Kang, DW., Adams, J.B., Gregory, A.C. et al. Microbiota Transfer Therapy alters gut ecosystem and improves gastrointestinal and autism symptoms: an open-label study. Microbiome 5, 10 (2017). )

## Methods
Children under the age of 18 with autism and gastrointestinal disorders were treated with fecal microbiota transplant to try and reduce the severity of their behavioral and gastrointestinal symptoms. 

Changes in their microbiome were tracked, as well as metrics that scale childhood autism, such as Parent Global Impressions-III (PGI-III) & Childhood Autism Rating Scale (CARS). Their gastrointestinal symptoms were tracked using GSRS for eighteen weeks.

Fecal swabs were collected by swabbing used toilet paper and occasionally testing the entire stool. Control groups were set up in twenty people to track normal temporal gut microbiome variation. Eighteen people received treatment. 

(“Fecal Transplant.” Fecal Transplant | Johns Hopkins Medicine, 4 Apr. 2022,)

## This command copies fasta to home directory and changes commands
cp /tmp/gen711_project_data/fastp-single.sh ~/fastp-single.sh
chmod +x ~/fastp-single.sh

## Performs quality control, trimming of adapters, filtering by quality, and read pruning  
./fastp-single.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 trimmed_fastqs
./fastp-single.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 trimmed_fastqs

## This command gets per base cov 
qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 --output-path demux1
qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 --output-path dumux2

## This performs trimming
qiime cutadapt trim-single --i-demultiplexed-sequences combined_fastqs.qza --p-front TACGTATGGTGCA --p-discard-untrimmed --p-match-adapter-wildcards --verbose --o-trimmed-sequences cutadapt1
qiime cutadapt trim-single --i-demultiplexed-sequences combined_fastqs.qza --p-front TACGTATGGTGCA --p-discard-untrimmed --p-match-adapter-wildcards --verbose --o-trimmed-sequences cutadapt2 

## This gives us visual information of the distribution of sequence qualities at each position
qiime demux summarize --i-data /home/users/ged1022/cutadapt1.qza --o-visualization  /home/users/ged1022/summarize1.qzv
qiime demux summarize --i-data /home/users/ged1022/cutadapt2.qza --o-visualization  /home/users/ged1022/summarize2.qzv

## This denoises single-end sequences, dereplicates them, and filters chimeras
qiime dada2 denoise-single --i-demultiplexed-seqs /home/users/ged1022/cutadapt1.qza --p-trunc-len 100 --p-trim-left 13 --p-n-threads 4 --o-denoising-stats denoising-stats1.qza --o-table feature_table.qza --o-representative-sequences rep-seqs1.qza
qiime dada2 denoise-single --i-demultiplexed-seqs /home/users/ged1022/cutadapt2.qza --p-trunc-len 100 --p-trim-left 13 --p-n-threads 4 --o-denoising-stats denoising-stats2.qza --o-table feature_table.qza --o-representative-sequences rep-seqs1.qza

## Visualization of column types in metadata
qiime metadata tabulate --m-input-file /home/users/ged1022/denoising-stats1.qza --o-visualization /home/users/ged1022/denoising-stats1.qzv
qiime metadata tabulate --m-input-file /home/users/ged1022/denoising-stats2.qza --o-visualization /home/users/ged1022/denoising-stats2.qzv

## Summarize the feature table and feature data
qiime feature-table tabulate-seqs --i-data /home/users/ged1022/rep-seqs1.qza --o-visualization /home/users/ged1022/rep-seqs1.qzv
qiime feature-table tabulate-seqs --i-data /home/users/ged1022/rep-seqs.qza --o-visualization /home/users/ged1022/rep-seqs2.qzv

## Combines feature data objects which may or may not contain data for the same features
qiime feature-table merge-seqs --i-data /home/users/ged1022/rep-seqs1.qza --i-data /home/users/ged1022/rep-seqs.qza --o-merged-data /home/users/ged1022/merged.rep-seqs.qza

## Run the classifier and use that to create a table of the output
qiime feature-classifier classify-sklearn --i-classifier /tmp/gen711_project_data/reference_databases/classifier.qza --i-reads /home/users/ged1022/merged.rep-seqs.qza --o-classification /home/users/ged1022/FMT-taxonomy.qza

## Visualize the taxonomic profiles of each sample using an interactive qiime taxa barplot
qiime taxa barplot --i-table /home/users/ged1022/feature_table1.qza --m-metadata-file sample-metadata.tsv --i-taxonomy /home/users/ged1022/FMT-taxonomy.qza --o-visualization /home/users/ged1022/barplot-1.qzv
qiime taxa barplot --i-table /home/users/ged1022/feature_table2.qza --m-metadata-file sample-metadata.tsv --i-taxonomy /home/users/ged1022/FMT-taxonomy.qza --o-visualization /home/users/ged1022/barplot-2.qzv

## Results Figures
<img width="1686" alt="Screen Shot 2023-05-16 at 1 19 17 PM" src="https://github.com/sophiemarek/poop-project/assets/130415363/7bba0d17-1404-4f38-8d7d-8bb88c9894af">
Figure 1- Plot generated from Qiime using the FASTA sequence. This graph shows the amount of different bacteria in each of the samples gut microbiome 
<img width="1299" alt="Screen Shot 2023-05-17 at 1 01 39 PM" src="https://github.com/sophiemarek/poop-project/assets/130415363/2ad2fbe9-23ec-4918-8717-403ceed9a6ce">
Figure 2- Plot generated from Qiime using FASTA sequence. This graph shows the bacteria found in the gut microbiome of the control group in comparison to the treatment group

## Results Interpretation 
The most common phyla in the gut microbiome are Firmicutes, Bacteroidetes, Actinobacteria and Proteobacteria.These phyla make up over 90% of the microbiota found in the human gut. As seen in Figure 1 the majority of the phyla found in the samples coincide with what was expected.

As seen in Figure 2 the treatment had more firmicutes present in their gut microbiome than the control group did. Firmicutes are important for producing endospores which help ensure that bacteria is able to survive in harsh environments such as one with a lack of nutrients and are able to promote transmission. The treatment group would need more firmicutes because of the harsher gut environment and that is seen when comparing the two groups. 

At the end of this study there was a reported 80% decrease in GI symptoms such as diarrhea, indigestion,contripation and abdominal pain. It was also reported that behavioral Autism Spectrum Disorder symptoms increased and remained improved after the treatment had ended. 


(Trevor O. Kirby, The gut microbiota as a therapeutic approach for obesity. Microbiome and Metabolome in Diagnosis, Therapy, and other Strategic Applications.(2019, February 15).)
