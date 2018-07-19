# How to run WGBS pipeline on Google Cloud

1) Create metadata CSV file and transfer to GC bucket
```
METADATA=metadata.csv
GC_PATH_METADATA=gs://encode-pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN
echo "sample_ENCSR750CPN,lib_test_ENCSR750CPN,flowcell,1,1" > metadata.csv
gsutil cp metadata.csv $GC_PATH_METADATA
```

2) Create input JSON file. Reference genome data for hg38 are shared on `gs://wgbs-pipeline`.
```
INPUT_JSON=ENCSR750CPN.google.json
cat > $INPUT_JSON << EOM
{
  "wgbs.reference_fasta": "gs://wgbs-pipeline/hg38/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz",
  "wgbs.indexed_reference_gem": "gs://wgbs-pipeline/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.gem",
  "wgbs.indexed_reference_info": "gs://wgbs-pipeline/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.info",
  "wgbs.metadata": "gs://encode-pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/metadata.csv",
  "wgbs.fastq_files": [{"left": "sample_ENCSR750CPN", 
  						"right": [{
  									"left": "flowcell_1_1", 
  								   	"right": ["gs://wgbs-pipeline/input_data/ENCSR750CPN/fastq/flowcell_1_1_1.fastq.gz",
  				  							  "gs://wgbs-pipeline/input_data/ENCSR750CPN/fastq/flowcell_1_1_2.fastq.gz"]
  								 }]
  					    }],
  "wgbs.chromosomes": ["chr1", "chr2", "chr3", "chr4", "chr5", "chr6", "chr7", "chr8", "chr9", "chr10", "chr11", "chr12", "chr13", "chr14", "chr15", "chr16", "chr17", "chr18", "chr19", "chr20", "chr21", "chr22", "chrX", "chrY"],
  "wgbs.organism": "Human",
  "wgbs.pyglob_nearness": 0
}
```

3) Download FASTQs from ENCODE portal and transfer to GC bucket.
FASTQs are available at https://www.encodeproject.org/experiments/ENCSR750CPN/
```
GC_PATH_FASTQ=gs://encode-pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/fastq
wget https://www.encodeproject.org/files/ENCFF961QCM/@@download/ENCFF961QCM.fastq.gz
wget https://www.encodeproject.org/files/ENCFF558LNP/@@download/ENCFF558LNP.fastq.gz
mv ENCFF961QCM.fastq.gz flowcell_1_1_1.fastq.gz	
mv ENCFF558LNP.fastq.gz flowcell_1_1_2.fastq.gz	
gsutil cp *.fastq.gz $GC_PATH_FASTQ
```

4) Download Cromwell and WGBS pipeline
```
cd $HOME
wget https://github.com/broadinstitute/cromwell/releases/download/33.1/cromwell-33.1.jar
git clone --branch develop_v1 https://github.com/ENCODE-DCC/wgbs-pipeline
```

5) Run pipeline
```
GC_ROOT=gs://wgbs-pipeline-test-runs/jin
GC_PRJ=wgbs-pipeline
BACKEND_FILE=backends/backend.conf
BACKEND=google
CROMWELL=~/cromwell-33.1.jar
WDL=wgbs-pipeline.wdl
INPUT_JSON=ENCSR750CPN.google.json
WF_OPT=workflow_opts/docker_google.json

cd $HOME/wgbs-pipeline
java -jar -Dconfig.file=$BACKEND_FILE -Dbackend.default=$BACKEND -Dbackend.providers.google.config.project=$GC_PRJ -Dbackend.providers.google.config.root=$GC_ROOT $CROMWELL run $WDL -i $INPUT_JSON -o $WF_OPT
```

# How to run WGBS pipeline on DNANexus
