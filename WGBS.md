# How to run WGBS pipeline on Google Cloud

1) Install latest Java and set up for Google Cloud. Follow instruction [here](https://encode-dcc.github.io/wdl-pipelines/install.html#google-cloud-platform)

2) Create metadata CSV file and transfer to GC bucket
```
METADATA=metadata.csv
GC_PATH_METADATA=gs://encode-pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN
echo "sample_ENCSR750CPN,lib_test_ENCSR750CPN,flowcell,1,1" > metadata.csv
gsutil cp metadata.csv $GC_PATH_METADATA
```

3) Create input JSON file. Reference genome data for hg38 are shared on `gs://wgbs-pipeline`.
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
EOM
```

4) Download FASTQs from ENCODE portal, rename them and transfer to GC bucket.
FASTQs are available at https://www.encodeproject.org/experiments/ENCSR750CPN/
```
GC_PATH_FASTQ=gs://encode-pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/fastq
wget https://www.encodeproject.org/files/ENCFF961QCM/@@download/ENCFF961QCM.fastq.gz
wget https://www.encodeproject.org/files/ENCFF558LNP/@@download/ENCFF558LNP.fastq.gz
mv ENCFF961QCM.fastq.gz flowcell_1_1_1.fastq.gz	
mv ENCFF558LNP.fastq.gz flowcell_1_1_2.fastq.gz	
gsutil cp *.fastq.gz $GC_PATH_FASTQ
```

5) Download Cromwell and WGBS pipeline
```
cd $HOME
wget https://github.com/broadinstitute/cromwell/releases/download/33.1/cromwell-33.1.jar
git clone --branch develop_v1 https://github.com/ENCODE-DCC/wgbs-pipeline
```

6) Run pipeline
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
1) Install latest Java and set up for DNANexus. Follow instruction [here](https://encode-dcc.github.io/wdl-pipelines/install.html#dnanexus-platform) and log in.
```
dx login # choose your target (output) project for pipeline
```

2) Create metadata CSV file and transfer to DX
```
METADATA=metadata.csv
DX_PATH_DATA_ROOT=project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN
echo "sample_ENCSR750CPN,lib_test_ENCSR750CPN,flowcell,1,1" > metadata.csv
#dx cp $METADATA $DX_PATH_DATA_ROOT/$METADATA/ # it didn't work, manually transfered it to DX
```

3) Create input JSON file. Reference genome data for hg38 are shared on `gs://wgbs-pipeline`.
```
INPUT_JSON=ENCSR750CPN.dx.json
cat > $INPUT_JSON << EOM
{
  "wgbs.reference_fasta": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz",
  "wgbs.indexed_reference_gem": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.gem",
  "wgbs.indexed_reference_info": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.info",
  "wgbs.metadata": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/metadata.csv",
  "wgbs.fastq_files": [{"left": "sample_ENCSR750CPN", 
  						"right": [{
  									"left": "flowcell_1_1", 
  								   	"right": ["dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/flowcell_1_1_1.fastq.gz",
  				  							  "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/flowcell_1_1_2.fastq.gz"]
  								 }]
  					    }],
  "wgbs.chromosomes": ["chr1", "chr2", "chr3", "chr4", "chr5", "chr6", "chr7", "chr8", "chr9", "chr10", "chr11", "chr12", "chr13", "chr14", "chr15", "chr16", "chr17", "chr18", "chr19", "chr20", "chr21", "chr22", "chrX", "chrY"],
  "wgbs.organism": "Human",
  "wgbs.pyglob_nearness": 0
}
EOM
#dx cp $METADATA $DX_PATH_METADATA/$METADATA # it didn't work, manually transfered it to DX
```

4) Download FASTQs from ENCODE portal, rename them and transfer to DX project.
FASTQs are available at https://www.encodeproject.org/experiments/ENCSR750CPN/
```
DX_PATH_DATA_ROOT=project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN
wget https://www.encodeproject.org/files/ENCFF961QCM/@@download/ENCFF961QCM.fastq.gz
wget https://www.encodeproject.org/files/ENCFF558LNP/@@download/ENCFF558LNP.fastq.gz
mv ENCFF961QCM.fastq.gz flowcell_1_1_1.fastq.gz	
mv ENCFF558LNP.fastq.gz flowcell_1_1_2.fastq.gz	
#dx cp *.fastq.gz $DX_PATH_DATA_ROOT/ # it didn't work, manually transfered it to DX
```

5) Download gemBS index for hg38 from GC bucket and transfer to DX project.
```
DX_PATH_GENOME_DATA_ROOT=project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-genome-data/hg38/
gsutil cp -r dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38/gemBS_index .
#dx cp -r gemBS_index $DX_PATH_GENOME_DATA_ROOT/ # it didn't work
```

6) Download dxWDL and WGBS pipeline
```
cd $HOME
wget https://github.com/dnanexus/dxWDL/releases/download/0.70/dxWDL-0.70.jar
git clone --branch develop_v1 https://github.com/ENCODE-DCC/wgbs-pipeline
```

5) Compile workflow
```
DX_ROOT=/wgbs-pipeline/ENCSR750CPN/
DX_PRJ=encode-pipeline-challenge
DXWDL=~/dxWDL-0.70.jar
WDL=wgbs-pipeline.wdl
INPUT_JSON=ENCSR750CPN.dx.json
WF_OPT=workflow_opts/docker_google.json

cd $HOME/wgbs-pipeline
java -jar $DXWDL compile $WDL -f -folder $DX_ROOT -defaults $INPUT_JSON -project $DX_PRJ -extras $WF_OPT
```

6) Run pipeline with DNANexus Web UI. Specify output folder and click launch.


# Comments

* ran successfully on GC. all outputs are [here](https://console.cloud.google.com/storage/browser/wgbs-pipeline-test-runs/jin/wgbs/55d7d715-eccc-4830-9906-5c15cbeed554/?project=wgbs-pipeline)

* transferred gemBS index on GC to DX: `dx://project-BKpvFg00VBPV975PgJ6Q03v6:/pipeline-genome-data/hg38/gemBS_index/`

* failed on DX due to unsupported data type for the workflow input variable `wgbs.fastq_files`. dxWDL uses latest cromwell compiler internally but complicated data type cannot be supported yet.
```
Array[Pair[String, Array[Pair[String,Array[File]]]]] fastq_files
```

```
$ java -jar $DXWDL compile $WDL -f -folder $DX_ROOT -defaults $INPUT_JSON -project $DX_PRJ -extras $WF_OPT
Picked up _JAVA_OPTIONS: -Xms256M -Xmx1024M -XX:ParallelGCThreads=1
Unsupported runtime attribute preemptible,we currently support Set(dx_instance_type, disks, docker, cpu, memory)
Unsupported runtime attribute bootDiskSizeGb,we currently support Set(dx_instance_type, disks, docker, cpu, memory)
Unsupported runtime attribute noAddress,we currently support Set(dx_instance_type, disks, docker, cpu, memory)
Unsupported runtime attribute zones,we currently support Set(dx_instance_type, disks, docker, cpu, memory)
Runtime attribute preemptible for task index is unknown
Runtime attribute preemptible for task merging_sample is unknown
Runtime attribute preemptible for task bscall_concatenate is unknown
Runtime attribute preemptible for task methylation_filtering is unknown
Runtime attribute preemptible for task bscall_job is unknown
Runtime attribute preemptible for task mapping_job is unknown
lookupObject: pipeline-genome-data/hg38/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz
lookupObject: pipeline-genome-data/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.gem
lookupObject: pipeline-genome-data/hg38/gemBS_index/GRCh38_no_alt_analysis_set_GCA_000001405.15.BS.info
lookupObject: pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/metadata.csv
lookupObject: pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/flowcell_1_1_1.fastq.gz
lookupObject: pipeline-test-samples/encode-wgbs-pipeline/ENCSR750CPN/flowcell_1_1_2.fastq.gz
java.lang.Exception: Unsupported/Invalid type/JSON combination in input file
  womType= Pair[String, Array[Pair[String, Array[File]]]]
  JSON= {
  "left": "sample_ENCSR750CPN",
  "right": [{
    "left": "flowcell_1_1",
    "right": [{
      "$dnanexus_link": "file-FJ8VBy00kJ12v6PzF37PZPkq"
    }, {
      "$dnanexus_link": "file-FJ8VJQQ0xj8383kF358FGg78"
    }]
  }]
}
        at dxWDL.WdlVarLinks$.importFromCromwell(WdlVarLinks.scala:684)
        at dxWDL.WdlVarLinks$.$anonfun$importFromCromwell$2(WdlVarLinks.scala:674)
        at scala.collection.TraversableLike.$anonfun$map$1(TraversableLike.scala:234)
        at scala.collection.Iterator.foreach(Iterator.scala:944)
        at scala.collection.Iterator.foreach$(Iterator.scala:944)
        at scala.collection.AbstractIterator.foreach(Iterator.scala:1432)
        at scala.collection.IterableLike.foreach(IterableLike.scala:71)
        at scala.collection.IterableLike.foreach$(IterableLike.scala:70)
        at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
        at scala.collection.TraversableLike.map(TraversableLike.scala:234)
        at scala.collection.TraversableLike.map$(TraversableLike.scala:227)
        at scala.collection.AbstractTraversable.map(Traversable.scala:104)
        at dxWDL.WdlVarLinks$.importFromCromwell(WdlVarLinks.scala:673)
        at dxWDL.WdlVarLinks$.importFromCromwellJSON(WdlVarLinks.scala:691)
        at dxWDL.compiler.InputFile.dxWDL$compiler$InputFile$$translateValue(InputFile.scala:60)
        at dxWDL.compiler.InputFile.$anonfun$addDefaultsToStage$2(InputFile.scala:103)
        at scala.collection.TraversableLike.$anonfun$map$1(TraversableLike.scala:234)
        at scala.collection.Iterator.foreach(Iterator.scala:944)
        at scala.collection.Iterator.foreach$(Iterator.scala:944)
        at scala.collection.AbstractIterator.foreach(Iterator.scala:1432)
        at scala.collection.IterableLike.foreach(IterableLike.scala:71)
        at scala.collection.IterableLike.foreach$(IterableLike.scala:70)
        at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
        at scala.collection.TraversableLike.map(TraversableLike.scala:234)
        at scala.collection.TraversableLike.map$(TraversableLike.scala:227)
        at scala.collection.AbstractTraversable.map(Traversable.scala:104)
        at dxWDL.compiler.InputFile.addDefaultsToStage(InputFile.scala:98)
        at dxWDL.compiler.InputFile.$anonfun$embedDefaultsIntoWorkflow$1(InputFile.scala:169)
        at scala.collection.TraversableLike.$anonfun$map$1(TraversableLike.scala:234)
        at scala.collection.Iterator.foreach(Iterator.scala:944)
        at scala.collection.Iterator.foreach$(Iterator.scala:944)
        at scala.collection.AbstractIterator.foreach(Iterator.scala:1432)
        at scala.collection.IterableLike.foreach(IterableLike.scala:71)
        at scala.collection.IterableLike.foreach$(IterableLike.scala:70)
        at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
        at scala.collection.TraversableLike.map(TraversableLike.scala:234)
        at scala.collection.TraversableLike.map$(TraversableLike.scala:227)
        at scala.collection.AbstractTraversable.map(Traversable.scala:104)
        at dxWDL.compiler.InputFile.embedDefaultsIntoWorkflow(InputFile.scala:166)
        at dxWDL.compiler.InputFile.embedDefaults(InputFile.scala:211)
        at dxWDL.compiler.Top$.compileIR(Top.scala:166)
        at dxWDL.compiler.Top$.apply(Top.scala:272)
        at dxWDL.Main$.compile(Main.scala:386)
        at dxWDL.Main$.dispatchCommand(Main.scala:589)
        at dxWDL.Main$.delayedEndpoint$dxWDL$Main$1(Main.scala:646)
        at dxWDL.Main$delayedInit$body.apply(Main.scala:13)
        at scala.Function0.apply$mcV$sp(Function0.scala:34)
        at scala.Function0.apply$mcV$sp$(Function0.scala:34)
        at scala.runtime.AbstractFunction0.apply$mcV$sp(AbstractFunction0.scala:12)
        at scala.App.$anonfun$main$1$adapted(App.scala:76)
        at scala.collection.immutable.List.foreach(List.scala:389)
        at scala.App.main(App.scala:76)
        at scala.App.main$(App.scala:74)
        at dxWDL.Main$.main(Main.scala:13)
        at dxWDL.Main.main(Main.scala)
```
