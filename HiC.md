# How to run HiC pipeline on Google Cloud

1) Install latest Java and set up for Google Cloud. Follow instruction [here](https://encode-dcc.github.io/wdl-pipelines/install.html#google-cloud-platform)

2) Create input JSON file. Restriction site information is shared on `gs://hic-pipeline`.
```
INPUT_JSON=ENCSR551IPY.google.json
cat > $INPUT_JSON << EOM
{
    "hic.fastq_files": [[
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF599JDF.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF020XCQ.fastq.gz"],
    [
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF671BNU.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF024CDK.fastq.gz"],
    [
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF904WWS.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF029EAX.fastq.gz"]],
    "hic.chrsz": "gs://encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/hg38_chr19_chrM.chrom.sizes",
    "hic.restriction_sites": "gs://encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt",
    "hic.reference_index": "gs://encode-pipeline-test-samples/encode-hic-pipeline/genome_reference_index/index.tar.gz"
}
EOM
```

3) Download FASTQs from ENCODE portal, rename them and transfer to GC bucket.
FASTQs are available at https://www.encodeproject.org/experiments/ENCSR551IPY/
```
GC_PATH_FASTQ=gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY
wget https://www.encodeproject.org/files/ENCFF599JDF/@@download/ENCFF599JDF.fastq.gz
wget https://www.encodeproject.org/files/ENCFF020XCQ/@@download/ENCFF020XCQ.fastq.gz
wget https://www.encodeproject.org/files/ENCFF671BNU/@@download/ENCFF671BNU.fastq.gz
wget https://www.encodeproject.org/files/ENCFF024CDK/@@download/ENCFF024CDK.fastq.gz
wget https://www.encodeproject.org/files/ENCFF904WWS/@@download/ENCFF904WWS.fastq.gz
wget https://www.encodeproject.org/files/ENCFF029EAX/@@download/ENCFF029EAX.fastq.gz
gsutil cp *.fastq.gz $GC_PATH_FASTQ
```

4) Download Cromwell and hic pipeline
```
cd $HOME
wget https://github.com/broadinstitute/cromwell/releases/download/33.1/cromwell-33.1.jar
git clone --branch restriction_enzyme https://github.com/ENCODE-DCC/hic-pipeline
```

5) Run pipeline
```
GC_ROOT=gs://hic-pipeline-test-runs/jin
GC_PRJ=hic-pipeline
BACKEND_FILE=backends/backend.conf
BACKEND=google
CROMWELL=~/cromwell-33.1.jar
WDL=hic.wdl
INPUT_JSON=ENCSR551IPY.google.json
WF_OPT=workflow_opts/docker.json

cd $HOME/hic-pipeline
java -jar -Dconfig.file=$BACKEND_FILE -Dbackend.default=$BACKEND -Dbackend.providers.google.config.project=$GC_PRJ -Dbackend.providers.google.config.root=$GC_ROOT $CROMWELL run $WDL -i $INPUT_JSON -o $WF_OPT
```

# How to run HiC pipeline on DNANexus
1) Install latest Java and set up for DNANexus. Follow instruction [here](https://encode-dcc.github.io/wdl-pipelines/install.html#dnanexus-platform) and log in.
```
dx login # choose your target (output) project for pipeline
```

2) Create input JSON file.
```
INPUT_JSON=ENCSR551IPY.dx.json
cat > $INPUT_JSON << EOM
{
    "hic.fastq_files": [[
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF599JDF.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF020XCQ.fastq.gz"],
    [
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF671BNU.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF024CDK.fastq.gz"],
    [
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF904WWS.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF029EAX.fastq.gz"]],
    "hic.chrsz": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/hg38_chr19_chrM.chrom.sizes",
    "hic.restriction_sites": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/restriction_sites/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt",
    "hic.reference_index": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/hic_reference_index/index.tar.gz"
}
EOM
```
#dx cp $METADATA $DX_PATH_METADATA/$METADATA # it didn't work, manually transfered it to DX
```

3) Download FASTQs from ENCODE portal, rename them and transfer to DX project.
FASTQs are available at https://www.encodeproject.org/experiments/ENCSR551IPY/
```
DX_PATH_DATA_ROOT=project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY
wget https://www.encodeproject.org/files/ENCFF599JDF/@@download/ENCFF599JDF.fastq.gz
wget https://www.encodeproject.org/files/ENCFF020XCQ/@@download/ENCFF020XCQ.fastq.gz
wget https://www.encodeproject.org/files/ENCFF671BNU/@@download/ENCFF671BNU.fastq.gz
wget https://www.encodeproject.org/files/ENCFF024CDK/@@download/ENCFF024CDK.fastq.gz
wget https://www.encodeproject.org/files/ENCFF904WWS/@@download/ENCFF904WWS.fastq.gz
wget https://www.encodeproject.org/files/ENCFF029EAX/@@download/ENCFF029EAX.fastq.gz
#dx cp *.fastq.gz $DX_PATH_DATA_ROOT/ # it didn't work, manually transfered it to DX
```

4) Download dxWDL and hic pipeline
```
cd $HOME
wget https://github.com/dnanexus/dxWDL/releases/download/0.70/dxWDL-0.70.jar
git clone --branch restriction_enzyme https://github.com/ENCODE-DCC/hic-pipeline
```

5) Compile workflow
```
DX_ROOT=/hic-pipeline/ENCSR551IPY/
DX_PRJ=encode-pipeline-challenge
DXWDL=~/dxWDL-0.70.jar
WDL=hic.wdl
INPUT_JSON=ENCSR551IPY.dx.json
WF_OPT=workflow_opts/docker.json

cd $HOME/hic-pipeline
java -jar $DXWDL compile $WDL -f -folder $DX_ROOT -defaults $INPUT_JSON -project $DX_PRJ -extras $WF_OPT
```

6) Run pipeline with DNANexus Web UI. Specify output folder and click launch.


# Comments

* ran successfully on GC. all outputs: https://console.cloud.google.com/storage/browser/hic-pipeline-test-runs/jin/hic/55d7d715-eccc-4830-9906-5c15cbeed554/?project=hic-pipeline

* transferred gemBS index on GC to DX: `project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-genome-data/hg38/gemBS_index/`


