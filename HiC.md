# How to run HiC pipeline on Google Cloud

1) Install latest Java and set up for Google Cloud. Follow instruction [here](https://encode-dcc.github.io/wdl-pipelines/install.html#google-cloud-platform)

2) Create input JSON file. Restriction site information is shared on `gs://hic-pipeline`.
```
INPUT_JSON=ENCSR551IPY.google.json
cat > $INPUT_JSON << EOM
{
    "hic.fastq": [[[
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF599JDF.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF020XCQ.fastq.gz"],
    [
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF671BNU.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF024CDK.fastq.gz"],
    [
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF904WWS.fastq.gz",
        "gs://encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF029EAX.fastq.gz"]]],
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
    "hic.fastq": [[[
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF599JDF.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF020XCQ.fastq.gz"],
    [
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF671BNU.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF024CDK.fastq.gz"],
    [
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF904WWS.fastq.gz",
        "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF029EAX.fastq.gz"]]],
    "hic.chrsz": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/hg38_chr19_chrM.chrom.sizes",
    "hic.restriction_sites": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/restriction_sites/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt",
    "hic.reference_index": "dx://project-BKpvFg00VBPV975PgJ6Q03v6:pipeline-genome-data/hg38_chr19_chrM/hic_reference_index/index.tar.gz"
}
EOM
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

* if you use `import` to include another WDL script then both cromwell and DX will fail to find in the github repo directory. these compilers look for sub WDL files in the working directory (not in the github script directory) you ran `java -jar *.jar` command line.

* failed on GC with an error `/cromwell_root/script: line 17: docker: command not found` in the task `tads`. full log is here.

```
$ java -jar -Dconfig.file=$BACKEND_FILE -Dbackend.default=$BACKEND -Dbackend.providers.google.config.project=$GC_PRJ -Dbackend.providers.google.config.root=$GC_ROOT $CROMWELL run $WDL -i $INPUT_JSON -o $WF_OPT
Picked up _JAVA_OPTIONS: -Xms256M -Xmx1024M -XX:ParallelGCThreads=1
[2018-07-19 16:49:51,22] [info] Running with database db.url = jdbc:hsqldb:mem:f6b4a783-209f-412c-8072-4529b774cb14;shutdown=false;hsqldb.tx=mvcc
[2018-07-19 16:50:01,34] [info] Running migration RenameWorkflowOptionsInMetadata with a read batch size of 100000 and a write batch size of 100000
[2018-07-19 16:50:01,36] [info] [RenameWorkflowOptionsInMetadata] 100%
[2018-07-19 16:50:02,00] [info] Running with database db.url = jdbc:hsqldb:mem:61b7bde2-a286-4b2a-8b0a-ec72e8598d52;shutdown=false;hsqldb.tx=mvcc
[2018-07-19 16:50:02,41] [warn] This actor factory is deprecated. Please use cromwell.backend.google.pipelines.v1alpha2.PipelinesApiLifecycleActorFactory for PAPI v1 or cromwell.backend.google.pipelines.v2alpha1.PipelinesApiLifecycleActorFactory for PAPI v2
[2018-07-19 16:50:02,42] [warn] Couldn't find a suitable DSN, defaulting to a Noop one.
[2018-07-19 16:50:02,43] [info] Using noop to send events.
[2018-07-19 16:50:02,79] [info] Slf4jLogger started
[2018-07-19 16:50:03,03] [info] Workflow heartbeat configuration:
{
  "cromwellId" : "cromid-356a6b2",
  "heartbeatInterval" : "2 minutes",
  "ttl" : "10 minutes",
  "writeBatchSize" : 10000,
  "writeThreshold" : 10000
}
[2018-07-19 16:50:03,07] [info] Metadata summary refreshing every 2 seconds.
[2018-07-19 16:50:03,13] [info] WriteMetadataActor configured to flush with batch size 200 and process rate 5 seconds.
[2018-07-19 16:50:03,13] [info] KvWriteActor configured to flush with batch size 200 and process rate 5 seconds.
[2018-07-19 16:50:03,14] [info] CallCacheWriteActor configured to flush with batch size 100 and process rate 3 seconds.
[2018-07-19 16:50:05,57] [info] JobExecutionTokenDispenser - Distribution rate: 50 per 1 seconds.
[2018-07-19 16:50:05,59] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:50:05,59] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:50:05,59] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:50:05,59] [info] PAPIQueryManager Running with 3 workers
[2018-07-19 16:50:05,61] [info] SingleWorkflowRunnerActor: Submitting workflow
[2018-07-19 16:50:05,67] [info] Unspecified type (Unspecified version) workflow 3e908ddd-dcbb-4472-9aa3-84532ec3db4d submitted
[2018-07-19 16:50:05,72] [info] SingleWorkflowRunnerActor: Workflow submitted 3e908ddd-dcbb-4472-9aa3-84532ec3db4d
[2018-07-19 16:50:05,72] [info] 1 new workflows fetched
[2018-07-19 16:50:05,72] [info] WorkflowManagerActor Starting workflow 3e908ddd-dcbb-4472-9aa3-84532ec3db4d
[2018-07-19 16:50:05,73] [info] WorkflowManagerActor Successfully started WorkflowActor-3e908ddd-dcbb-4472-9aa3-84532ec3db4d
[2018-07-19 16:50:05,73] [info] Retrieved 1 workflows from the WorkflowStoreActor
[2018-07-19 16:50:05,74] [warn] SingleWorkflowRunnerActor: received unexpected message: Done in state RunningSwraData
[2018-07-19 16:50:05,75] [info] WorkflowStoreHeartbeatWriteActor configured to flush with batch size 10000 and process rate 2 minutes.
[2018-07-19 16:50:05,81] [info] MaterializeWorkflowDescriptorActor [3e908ddd]: Parsing workflow as WDL draft-2
[2018-07-19 16:50:07,34] [info] MaterializeWorkflowDescriptorActor [3e908ddd]: Call-to-Backend assignments: hic.hiccups -> google, hic.merge_pairs_file -> google, hic_sub.merge_sort -> google, hic_sub.dedup -> google, hic_sub.merge -> google, hic.tads -> google, hic_sub.align -> google, hic.create_hic -> google
[2018-07-19 16:50:10,17] [info] WorkflowExecutionActor-3e908ddd-dcbb-4472-9aa3-84532ec3db4d [3e908ddd]: Condition met: '!defined(input_hic)'. Running conditional section
[2018-07-19 16:50:16,36] [info] WorkflowExecutionActor-3e908ddd-dcbb-4472-9aa3-84532ec3db4d [3e908ddd]: Starting hic.merge_pairs_file
[2018-07-19 16:50:18,93] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.merge_pairs_file:NA:1]: sort -m -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n --parallel=8 -S 10%   > merged_pairs.txt
[2018-07-19 16:50:43,19] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.merge_pairs_file:NA:1]: job id: operations/EPvDz6fLLBjzmeDYzbLG-dYBIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 16:51:12,67] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.merge_pairs_file:NA:1]: Status change from - to Running
[2018-07-19 16:52:19,39] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.merge_pairs_file:NA:1]: Status change from Running to Success
[2018-07-19 16:52:21,38] [info] WorkflowExecutionActor-3e908ddd-dcbb-4472-9aa3-84532ec3db4d [3e908ddd]: Starting hic.create_hic
[2018-07-19 16:52:21,60] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.create_hic:NA:1]: /opt/scripts/common/juicer_tools pre -s inter_30.txt -g inter_30_hists.m -q 30 /cromwell_root/hic-pipeline-test-runs/jin/hic/3e908ddd-dcbb-4472-9aa3-84532ec3db4d/call-merge_pairs_file/glob-28a2611301572cea1d350945fbaa76d8/merged_pairs.txt inter_30.hic /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/hg38_chr19_chrM.chrom.sizes
[2018-07-19 16:52:58,15] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.create_hic:NA:1]: job id: operations/ENzY16fLLBjHwdv2xI_FrGIgvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 16:53:26,45] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.create_hic:NA:1]: Status change from - to Running
[2018-07-19 16:55:06,87] [info] PipelinesApiAsyncBackendJobExecutionActor [3e908dddhic.create_hic:NA:1]: Status change from Running to Success
[2018-07-19 16:55:07,90] [error] WorkflowManagerActor Workflow 3e908ddd-dcbb-4472-9aa3-84532ec3db4d failed (during ExecutingWorkflowState): Job hic.create_hic:NA:1 exited with return code 57 which has not been declared as a valid return code. See 'continueOnReturnCode' runtime attribute for more details.
Check the content of stderr for potential additional information: gs://hic-pipeline-test-runs/jin/hic/3e908ddd-dcbb-4472-9aa3-84532ec3db4d/call-create_hic/stderr.
 Picked up _JAVA_OPTIONS: -Djava.io.tmpdir=/cromwell_root/tmp.18d47473
/cromwell_root/hic-pipeline-test-runs/jin/hic/3e908ddd-dcbb-4472-9aa3-84532ec3db4d/call-merge_pairs_file/glob-28a2611301572cea1d350945fbaa76d8/merged_pairs.txt does not exist or does not contain any reads.

[2018-07-19 16:55:07,90] [info] WorkflowManagerActor WorkflowActor-3e908ddd-dcbb-4472-9aa3-84532ec3db4d is in a terminal state: WorkflowFailedState
[2018-07-19 16:55:24,04] [info] SingleWorkflowRunnerActor workflow finished with status 'Failed'.
[2018-07-19 16:55:28,15] [info] Workflow polling stopped
[2018-07-19 16:55:28,17] [info] Shutting down WorkflowStoreActor - Timeout = 5 seconds
[2018-07-19 16:55:28,18] [info] Shutting down WorkflowLogCopyRouter - Timeout = 5 seconds
[2018-07-19 16:55:28,18] [info] Shutting down JobExecutionTokenDispenser - Timeout = 5 seconds
[2018-07-19 16:55:28,18] [info] Aborting all running workflows.
[2018-07-19 16:55:28,18] [info] JobExecutionTokenDispenser stopped
[2018-07-19 16:55:28,18] [info] WorkflowStoreActor stopped
[2018-07-19 16:55:28,19] [info] WorkflowLogCopyRouter stopped
[2018-07-19 16:55:28,19] [info] Shutting down WorkflowManagerActor - Timeout = 3600 seconds
[2018-07-19 16:55:28,19] [info] WorkflowManagerActor stopped
[2018-07-19 16:55:28,19] [info] WorkflowManagerActor All workflows finished
[2018-07-19 16:55:28,19] [info] Connection pools shut down
[2018-07-19 16:55:28,19] [info] Shutting down SubWorkflowStoreActor - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] Shutting down JobStoreActor - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] Shutting down CallCacheWriteActor - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] SubWorkflowStoreActor stopped
[2018-07-19 16:55:28,19] [info] Shutting down ServiceRegistryActor - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] Shutting down DockerHashActor - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] Shutting down IoProxy - Timeout = 1800 seconds
[2018-07-19 16:55:28,19] [info] CallCacheWriteActor Shutting down: 0 queued messages to process
[2018-07-19 16:55:28,19] [info] JobStoreActor stopped
[2018-07-19 16:55:28,19] [info] CallCacheWriteActor stopped
[2018-07-19 16:55:28,19] [info] KvWriteActor Shutting down: 0 queued messages to process
[2018-07-19 16:55:28,19] [info] WriteMetadataActor Shutting down: 0 queued messages to process
[2018-07-19 16:55:28,19] [info] DockerHashActor stopped
[2018-07-19 16:55:28,19] [info] IoProxy stopped
[2018-07-19 16:55:28,20] [info] ServiceRegistryActor stopped
[2018-07-19 16:55:28,22] [info] Database closed
[2018-07-19 16:55:28,22] [info] Stream materializer shut down
Workflow 3e908ddd-dcbb-4472-9aa3-84532ec3db4d transitioned to state Failed
[2018-07-19 16:55:28,26] [info] Automatic shutdown of the async connection
[2018-07-19 16:55:28,26] [info] Gracefully shutdown sentry threads.
[2018-07-19 16:55:28,26] [info] Shutdown finished.
leepc12@kadru:/users/leepc12/code/hic-pipeline$ java -jar -Dconfig.file=$BACKEND_FILE -Dbackend.default=$BACKEND -Dbackend.providers.google.config.project=$GC_PRJ -Dbackend.providers.google.config.root=$GC_ROOT $CROMWELL run $WDL -i $INPUT_JSON -o $WF_OPT
Picked up _JAVA_OPTIONS: -Xms256M -Xmx1024M -XX:ParallelGCThreads=1
[2018-07-19 16:56:39,07] [info] Running with database db.url = jdbc:hsqldb:mem:e67e651d-e3a7-46ca-a144-d99903ae208d;shutdown=false;hsqldb.tx=mvcc
[2018-07-19 16:56:52,20] [info] Running migration RenameWorkflowOptionsInMetadata with a read batch size of 100000 and a write batch size of 100000
[2018-07-19 16:56:52,21] [info] [RenameWorkflowOptionsInMetadata] 100%
[2018-07-19 16:56:52,31] [info] Running with database db.url = jdbc:hsqldb:mem:2004ceca-eea4-41f9-9791-f3bb674dcc3e;shutdown=false;hsqldb.tx=mvcc
[2018-07-19 16:56:52,73] [warn] This actor factory is deprecated. Please use cromwell.backend.google.pipelines.v1alpha2.PipelinesApiLifecycleActorFactory for PAPI v1 or cromwell.backend.google.pipelines.v2alpha1.PipelinesApiLifecycleActorFactory for PAPI v2
[2018-07-19 16:56:52,74] [warn] Couldn't find a suitable DSN, defaulting to a Noop one.
[2018-07-19 16:56:52,75] [info] Using noop to send events.
[2018-07-19 16:56:53,10] [info] Slf4jLogger started
[2018-07-19 16:56:53,35] [info] Workflow heartbeat configuration:
{
  "cromwellId" : "cromid-c080889",
  "heartbeatInterval" : "2 minutes",
  "ttl" : "10 minutes",
  "writeBatchSize" : 10000,
  "writeThreshold" : 10000
}
[2018-07-19 16:56:53,40] [info] Metadata summary refreshing every 2 seconds.
[2018-07-19 16:56:53,45] [info] WriteMetadataActor configured to flush with batch size 200 and process rate 5 seconds.
[2018-07-19 16:56:53,45] [info] KvWriteActor configured to flush with batch size 200 and process rate 5 seconds.
[2018-07-19 16:56:53,45] [info] CallCacheWriteActor configured to flush with batch size 100 and process rate 3 seconds.
[2018-07-19 16:56:54,94] [info] JobExecutionTokenDispenser - Distribution rate: 50 per 1 seconds.
[2018-07-19 16:56:54,96] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:56:54,97] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:56:54,97] [info] JES batch polling interval is 33333 milliseconds
[2018-07-19 16:56:54,97] [info] PAPIQueryManager Running with 3 workers
[2018-07-19 16:56:54,97] [info] SingleWorkflowRunnerActor: Submitting workflow
[2018-07-19 16:56:55,04] [info] Unspecified type (Unspecified version) workflow ec5840bf-c4aa-4c2e-a78e-02c64e807919 submitted
[2018-07-19 16:56:55,09] [info] SingleWorkflowRunnerActor: Workflow submitted ec5840bf-c4aa-4c2e-a78e-02c64e807919
[2018-07-19 16:56:55,09] [info] 1 new workflows fetched
[2018-07-19 16:56:55,09] [info] WorkflowManagerActor Starting workflow ec5840bf-c4aa-4c2e-a78e-02c64e807919
[2018-07-19 16:56:55,10] [info] WorkflowManagerActor Successfully started WorkflowActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919
[2018-07-19 16:56:55,10] [info] Retrieved 1 workflows from the WorkflowStoreActor
[2018-07-19 16:56:55,10] [warn] SingleWorkflowRunnerActor: received unexpected message: Done in state RunningSwraData
[2018-07-19 16:56:55,11] [info] WorkflowStoreHeartbeatWriteActor configured to flush with batch size 10000 and process rate 2 minutes.
[2018-07-19 16:56:55,17] [info] MaterializeWorkflowDescriptorActor [ec5840bf]: Parsing workflow as WDL draft-2
[2018-07-19 16:56:56,54] [info] MaterializeWorkflowDescriptorActor [ec5840bf]: Call-to-Backend assignments: hic.merge_pairs_file -> google, hic.create_hic -> google, hic.tads -> google, hic.hiccups -> google, hic_sub.merge_sort -> google, hic_sub.align -> google, hic_sub.dedup -> google, hic_sub.merge -> google
[2018-07-19 16:56:59,16] [info] WorkflowExecutionActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919 [ec5840bf]: Condition met: '!defined(input_hic)'. Running conditional section
[2018-07-19 16:57:08,48] [info] 50676269-5789-4195-bcd8-2a4f40e930cc-SubWorkflowActor-SubWorkflow-hic_sub:0:1 [50676269]: Starting hic_sub.align (3 shards)
[2018-07-19 16:57:11,33] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:0:1]:
   echo "Starting align"
   mkdir data && cd data && mkdir reference
   data_path=$(pwd)
   cd reference && tar -xvf /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/genome_reference_index/index.tar.gz
   index_folder=$(ls)
   cd $index_folder
   reference_fasta=$(ls | head -1)
   reference_folder=$(pwd)
   reference_index_path=$reference_folder/$reference_fasta
   cd ../..

   # Align reads
   echo "Running bwa command"
   bwa mem -SP5M -t 4 $reference_index_path /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF599JDF.fastq.gz /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF020XCQ.fastq.gz > result.sam
   # GOOD UNTIL HERE

# chimeric takes in $name$ext
   echo "Running chimeric script"
awk -v "fname"=result -f /opt/scripts/common/chimeric_blacklist.awk result.sam

   # if any normal reads were written, find what fragment they correspond
 # to and store that
 echo "Running fragment"
   echo $restriction
   /opt/scripts/common/fragment.pl result_norm.txt result_frag.txt /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt

   # convert sams to bams and delete the sams
   echo "Converting sam to bam"
samtools view -hb result_collisions.sam > collisions.bam
   samtools view -hb result_collisions_low_mapq.sam > collisions_low_mapq.bam
   samtools view -hb result_unmapped.sam > unmapped.bam
   samtools view -hb result_mapq0.sam > mapq0.bam
   samtools view -hb result_alignable.sam > alignable.bam

   #removed all sam files
   ##restriction used to be site_file
   rm result_collisions.sam result_collisions_low_mapq.sam result_unmapped.sam result_mapq0.sam result_alignable.sam

   # sort by chromosome, fragment, strand, and position
sort -T /opt/HIC_tmp -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n result_frag.txt > sort.txt
   if [ $? -ne 0 ]
then
       echo "***! Failure during sort"
       exit 1x
fi
[2018-07-19 16:57:11,33] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:2:1]:
   echo "Starting align"
   mkdir data && cd data && mkdir reference
   data_path=$(pwd)
   cd reference && tar -xvf /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/genome_reference_index/index.tar.gz
   index_folder=$(ls)
   cd $index_folder
   reference_fasta=$(ls | head -1)
   reference_folder=$(pwd)
   reference_index_path=$reference_folder/$reference_fasta
   cd ../..

   # Align reads
   echo "Running bwa command"
   bwa mem -SP5M -t 4 $reference_index_path /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF904WWS.fastq.gz /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF029EAX.fastq.gz > result.sam
   # GOOD UNTIL HERE

# chimeric takes in $name$ext
   echo "Running chimeric script"
awk -v "fname"=result -f /opt/scripts/common/chimeric_blacklist.awk result.sam

   # if any normal reads were written, find what fragment they correspond
 # to and store that
 echo "Running fragment"
   echo $restriction
   /opt/scripts/common/fragment.pl result_norm.txt result_frag.txt /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt

   # convert sams to bams and delete the sams
   echo "Converting sam to bam"
samtools view -hb result_collisions.sam > collisions.bam
   samtools view -hb result_collisions_low_mapq.sam > collisions_low_mapq.bam
   samtools view -hb result_unmapped.sam > unmapped.bam
   samtools view -hb result_mapq0.sam > mapq0.bam
   samtools view -hb result_alignable.sam > alignable.bam

   #removed all sam files
   ##restriction used to be site_file
   rm result_collisions.sam result_collisions_low_mapq.sam result_unmapped.sam result_mapq0.sam result_alignable.sam

   # sort by chromosome, fragment, strand, and position
sort -T /opt/HIC_tmp -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n result_frag.txt > sort.txt
   if [ $? -ne 0 ]
then
       echo "***! Failure during sort"
       exit 1x
fi
[2018-07-19 16:57:11,33] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:1:1]:
   echo "Starting align"
   mkdir data && cd data && mkdir reference
   data_path=$(pwd)
   cd reference && tar -xvf /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/genome_reference_index/index.tar.gz
   index_folder=$(ls)
   cd $index_folder
   reference_fasta=$(ls | head -1)
   reference_folder=$(pwd)
   reference_index_path=$reference_folder/$reference_fasta
   cd ../..

   # Align reads
   echo "Running bwa command"
   bwa mem -SP5M -t 4 $reference_index_path /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF671BNU.fastq.gz /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/ENCSR551IPY/ENCFF024CDK.fastq.gz > result.sam
   # GOOD UNTIL HERE

# chimeric takes in $name$ext
   echo "Running chimeric script"
awk -v "fname"=result -f /opt/scripts/common/chimeric_blacklist.awk result.sam

   # if any normal reads were written, find what fragment they correspond
 # to and store that
 echo "Running fragment"
   echo $restriction
   /opt/scripts/common/fragment.pl result_norm.txt result_frag.txt /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/GRCh38_no_alt_analysis_set_GCA_000001405.15.chr19_chrM_MboI.txt

   # convert sams to bams and delete the sams
   echo "Converting sam to bam"
samtools view -hb result_collisions.sam > collisions.bam
   samtools view -hb result_collisions_low_mapq.sam > collisions_low_mapq.bam
   samtools view -hb result_unmapped.sam > unmapped.bam
   samtools view -hb result_mapq0.sam > mapq0.bam
   samtools view -hb result_alignable.sam > alignable.bam

   #removed all sam files
   ##restriction used to be site_file
   rm result_collisions.sam result_collisions_low_mapq.sam result_unmapped.sam result_mapq0.sam result_alignable.sam

   # sort by chromosome, fragment, strand, and position
sort -T /opt/HIC_tmp -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n result_frag.txt > sort.txt
   if [ $? -ne 0 ]
then
       echo "***! Failure during sort"
       exit 1x
fi
[2018-07-19 16:57:33,51] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:0:1]: job id: operations/ENTC6KfLLBjX3o-qk_e57ZcBIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 16:57:33,51] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:2:1]: job id: operations/EJHD6KfLLBiMhbXm_cq_4NwBIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 16:57:33,51] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:1:1]: job id: operations/EM3C6KfLLBj2j8W7oemh2Bcgvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 16:58:02,04] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:1:1]: Status change from - to Running
[2018-07-19 16:58:02,05] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:2:1]: Status change from - to Running
[2018-07-19 16:58:02,06] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:0:1]: Status change from - to Running
[2018-07-19 16:59:42,53] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:1:1]: Status change from Running to Success
[2018-07-19 17:04:10,04] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:2:1]: Status change from Running to Success
[2018-07-19 17:04:43,42] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.align:0:1]: Status change from Running to Success
[2018-07-19 17:04:47,88] [info] 50676269-5789-4195-bcd8-2a4f40e930cc-SubWorkflowActor-SubWorkflow-hic_sub:0:1 [50676269]: Starting hic_sub.merge_sort
[2018-07-19 17:04:48,38] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge_sort:NA:1]: sort -m -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n --parallel=8 -S 10% /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt  > merged_sort.txt
[2018-07-19 17:04:48,91] [info] 50676269-5789-4195-bcd8-2a4f40e930cc-SubWorkflowActor-SubWorkflow-hic_sub:0:1 [50676269]: Starting hic_sub.merge
[2018-07-19 17:04:48,98] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge:NA:1]: samtools merge merged_collisions.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam
samtools merge merged_collisions_lowmapq.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam
samtools merge merged_unmapped.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam
samtools merge merged_mapq0.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-0e332e42297c61a239c20058b776c301/mapq0.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-0e332e42297c61a239c20058b776c301/mapq0.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-0e332e42297c61a239c20058b776c301/mapq0.bam
samtools merge merged_alignable.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam
[2018-07-19 17:05:18,47] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge:NA:1]: job id: operations/EL-LhajLLBju_bHegLPlluwBIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 17:05:18,47] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge_sort:NA:1]: job id: operations/ELmLhajLLBjPhKuUn573sjggvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 17:05:50,46] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge:NA:1]: Status change from - to Running
[2018-07-19 17:05:50,46] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge_sort:NA:1]: Status change from - to Running
[2018-07-19 17:06:57,21] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge_sort:NA:1]: Status change from Running to Success
[2018-07-19 17:06:59,47] [info] 50676269-5789-4195-bcd8-2a4f40e930cc-SubWorkflowActor-SubWorkflow-hic_sub:0:1 [50676269]: Starting hic_sub.dedup
[2018-07-19 17:06:59,98] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.dedup:NA:1]: touch dups.txt
touch optdups.txt
touch merged_nodups.txt
awk -f /opt/scripts/common/dups.awk /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge_sort/glob-3fca55958f9e521cf685cb69bb0cba48/merged_sort.txt
[2018-07-19 17:07:33,47] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.dedup:NA:1]: job id: operations/EKOhjajLLBiu-N6cnsj7tw8gvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 17:08:04,25] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.dedup:NA:1]: Status change from - to Running
[2018-07-19 17:09:11,03] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.dedup:NA:1]: Status change from Running to Success
[2018-07-19 17:10:18,04] [info] PipelinesApiAsyncBackendJobExecutionActor [50676269hic_sub.merge:NA:1]: Status change from Running to Success
[2018-07-19 17:10:20,43] [info] 50676269-5789-4195-bcd8-2a4f40e930cc-SubWorkflowActor-SubWorkflow-hic_sub:0:1 [50676269]: Workflow hic_sub complete. Final Outputs:
{
  "hic_sub.out_unmapped": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-3e8d0b0ad8d7b395c65e466caea64369/unmapped.bam"],
  "hic_sub.out_collisions_low": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-a07e75a530220ee3ab115c6819e710f8/collisions_low_mapq.bam"],
  "hic_sub.out_merge_sort": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge_sort/glob-3fca55958f9e521cf685cb69bb0cba48/merged_sort.txt",
  "hic_sub.out_merged_collisions_low": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge/glob-48a8cc5b582f5a9b84d09795d92566c4/merged_collisions_lowmapq.bam",
  "hic_sub.out_alignable": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-1890eb591bb2a7d7b20ad8674fb9113a/alignable.bam"],
  "hic_sub.out_mapq0": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-0e332e42297c61a239c20058b776c301/mapq0.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-0e332e42297c61a239c20058b776c301/mapq0.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-0e332e42297c61a239c20058b776c301/mapq0.bam"],
  "hic_sub.out_sort_file": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-d1c69b66aa3b2f1a221725e00cd2ae8c/sort.txt"],
  "hic_sub.out_merged_unmapped": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge/glob-424a3f9a758fbfe7eb85506bcfc0051c/merged_unmapped.bam",
  "hic_sub.out_merged_align": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge/glob-4709a86f566f9145e550ae2160fbf105/merged_alignable.bam",
  "hic_sub.out_merged_collisions": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge/glob-f2d22b942f49793caf6b0993e2b0610c/merged_collisions.bam",
  "hic_sub.out_dedup": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-dedup/glob-4597e8834cf225f807c30b0caec03863/merged_nodups.txt",
  "hic_sub.out_collisions": ["gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-0/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-1/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam", "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-align/shard-2/glob-b76556d4d7bc657e18489b542e50e2dd/collisions.bam"],
  "hic_sub.out_merged_mapq0": "gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-merge/glob-49d2d8d8c6612815b97beab28ee8c9c0/merged_mapq0.bam"
}
[2018-07-19 17:10:24,48] [info] WorkflowExecutionActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919 [ec5840bf]: Starting hic.merge_pairs_file
[2018-07-19 17:10:24,97] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.merge_pairs_file:NA:1]: sort -m -k2,2d -k6,6d -k4,4n -k8,8n -k1,1n -k5,5n -k3,3n --parallel=8 -S 10% /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hic_sub/shard-0/hic_sub/50676269-5789-4195-bcd8-2a4f40e930cc/call-dedup/glob-4597e8834cf225f807c30b0caec03863/merged_nodups.txt  > merged_pairs.txt
[2018-07-19 17:10:53,47] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.merge_pairs_file:NA:1]: job id: operations/ELfAmajLLBjxguTr87XGsHkgvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 17:11:24,82] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.merge_pairs_file:NA:1]: Status change from - to Running
[2018-07-19 17:12:31,80] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.merge_pairs_file:NA:1]: Status change from Running to Success
[2018-07-19 17:12:34,02] [info] WorkflowExecutionActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919 [ec5840bf]: Starting hic.create_hic
[2018-07-19 17:12:34,97] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.create_hic:NA:1]: /opt/scripts/common/juicer_tools pre -s inter_30.txt -g inter_30_hists.m -q 30 /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-merge_pairs_file/glob-28a2611301572cea1d350945fbaa76d8/merged_pairs.txt inter_30.hic /cromwell_root/encode-pipeline-test-samples/encode-hic-pipeline/reference_grch38_chr10_chrM/hg38_chr19_chrM.chrom.sizes
[2018-07-19 17:13:08,47] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.create_hic:NA:1]: job id: operations/EKTWoajLLBjKw-vA39mr9G8gvcv_iN8bKg9wcm9kdWN0aW9uUXVldWU
[2018-07-19 17:13:38,63] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.create_hic:NA:1]: Status change from - to Running
[2018-07-19 17:15:19,11] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.create_hic:NA:1]: Status change from Running to Success
[2018-07-19 17:15:21,30] [info] WorkflowExecutionActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919 [ec5840bf]: Starting hic.tads, hic.hiccups
[2018-07-19 17:15:21,97] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.tads:NA:1]: docker run --runtime=nvidia --entrypoint /opt/scripts/common/juicer_tools arrowhead /cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-create_hic/glob-ba0c8fce6341d8aa09affa9f1df33235/inter_30.hic contact_domains
[2018-07-19 17:15:21,97] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.hiccups:NA:1]: DIR=$(dirname "/cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-create_hic/glob-ba0c8fce6341d8aa09affa9f1df33235/inter_30.hic")
FILE=$(basename "/cromwell_root/hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-create_hic/glob-ba0c8fce6341d8aa09affa9f1df33235/inter_30.hic")
docker run --runtime=nvidia --entrypoint /opt/scripts/common/juicer_tools -v $DIR:/input quay.io/anacismaru/nvidia_juicer:test hiccups /input/$FILE loops
[2018-07-19 17:15:53,47] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.hiccups:NA:1]: job id: operations/EKXxq6jLLBiNrcLmycuWtqIBIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 17:15:53,47] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.tads:NA:1]: job id: operations/ELHxq6jLLBjsuvSi3fS6i_0BIL3L_4jfGyoPcHJvZHVjdGlvblF1ZXVl
[2018-07-19 17:16:25,99] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.hiccups:NA:1]: Status change from - to Running
[2018-07-19 17:16:26,00] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.tads:NA:1]: Status change from - to Running
[2018-07-19 17:18:06,35] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.hiccups:NA:1]: Status change from Running to Success
[2018-07-19 17:18:06,36] [info] PipelinesApiAsyncBackendJobExecutionActor [ec5840bfhic.tads:NA:1]: Status change from Running to Success
[2018-07-19 17:18:08,02] [error] WorkflowManagerActor Workflow ec5840bf-c4aa-4c2e-a78e-02c64e807919 failed (during ExecutingWorkflowState): Job hic.tads:NA:1 exited with return code 127 which has not been declared as a valid return code. See 'continueOnReturnCode' runtime attribute for more details.
Check the content of stderr for potential additional information: gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-tads/stderr.
 /cromwell_root/script: line 17: docker: command not found

Job hic.hiccups:NA:1 exited with return code 127 which has not been declared as a valid return code. See 'continueOnReturnCode' runtime attribute for more details.
Check the content of stderr for potential additional information: gs://hic-pipeline-test-runs/jin/hic/ec5840bf-c4aa-4c2e-a78e-02c64e807919/call-hiccups/stderr.
 /cromwell_root/script: line 19: docker: command not found

[2018-07-19 17:18:08,02] [info] WorkflowManagerActor WorkflowActor-ec5840bf-c4aa-4c2e-a78e-02c64e807919 is in a terminal state: WorkflowFailedState
[2018-07-19 17:18:41,46] [info] SingleWorkflowRunnerActor workflow finished with status 'Failed'.
[2018-07-19 17:18:43,47] [info] Workflow polling stopped
[2018-07-19 17:18:43,49] [info] Shutting down WorkflowStoreActor - Timeout = 5 seconds
[2018-07-19 17:18:43,50] [info] Shutting down WorkflowLogCopyRouter - Timeout = 5 seconds
[2018-07-19 17:18:43,50] [info] Shutting down JobExecutionTokenDispenser - Timeout = 5 seconds
[2018-07-19 17:18:43,50] [info] Aborting all running workflows.
[2018-07-19 17:18:43,50] [info] JobExecutionTokenDispenser stopped
[2018-07-19 17:18:43,50] [info] WorkflowStoreActor stopped
[2018-07-19 17:18:43,51] [info] WorkflowLogCopyRouter stopped
[2018-07-19 17:18:43,51] [info] Shutting down WorkflowManagerActor - Timeout = 3600 seconds
[2018-07-19 17:18:43,51] [info] WorkflowManagerActor stopped
[2018-07-19 17:18:43,51] [info] WorkflowManagerActor All workflows finished
[2018-07-19 17:18:43,51] [info] Connection pools shut down
[2018-07-19 17:18:43,51] [info] Shutting down SubWorkflowStoreActor - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] Shutting down JobStoreActor - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] Shutting down CallCacheWriteActor - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] SubWorkflowStoreActor stopped
[2018-07-19 17:18:43,51] [info] Shutting down ServiceRegistryActor - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] Shutting down DockerHashActor - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] Shutting down IoProxy - Timeout = 1800 seconds
[2018-07-19 17:18:43,51] [info] CallCacheWriteActor Shutting down: 0 queued messages to process
[2018-07-19 17:18:43,51] [info] JobStoreActor stopped
[2018-07-19 17:18:43,51] [info] KvWriteActor Shutting down: 0 queued messages to process
[2018-07-19 17:18:43,51] [info] WriteMetadataActor Shutting down: 0 queued messages to process
[2018-07-19 17:18:43,51] [info] CallCacheWriteActor stopped
[2018-07-19 17:18:43,51] [info] DockerHashActor stopped
[2018-07-19 17:18:43,51] [info] IoProxy stopped
[2018-07-19 17:18:43,52] [info] ServiceRegistryActor stopped
[2018-07-19 17:18:43,54] [info] Database closed
[2018-07-19 17:18:43,54] [info] Stream materializer shut down
Workflow ec5840bf-c4aa-4c2e-a78e-02c64e807919 transitioned to state Failed
[2018-07-19 17:18:43,58] [info] Automatic shutdown of the async connection
[2018-07-19 17:18:43,58] [info] Gracefully shutdown sentry threads.
[2018-07-19 17:18:43,58] [info] Shutdown finished.
```
