### GSP311 : Automate Interactions with Contact Center AI: Challenge Lab :-

----------------------------------------------------------------------------------------------------------------------------------------------

Follow on Twitter @techiezilla , Instagram @techiezilla.


### Task - 0 : Download the necessary files :-

```yaml
export PROJECT=$(gcloud info --format='value(config.project)')
git clone https://github.com/GoogleCloudPlatform/dataflow-contact-center-speech-analysis.git
```

### Task - 1 : Create a Regional Cloud Storage bucket :-

```yaml
gsutil mb -l us-central1 gs://${PROJECT}/
```

### Task - 2 : Create a Cloud Function :-

Go to the “saf-longrun-job-func” directory.

```yaml
gcloud functions deploy safLongRunJobFunc --runtime nodejs8 --trigger-resource ${PROJECT} --region us-central1 --trigger-event google.storage.object.finalize
```

### Task - 3 : Create a BigQuery Dataset :-

```yaml
bq mk saf
```

### Task - 4 : Create Cloud Pub/Sub Topic :-

```yaml
gcloud pubsub topics create helpdesk
```

### Task - 5 : Create a Cloud Storage Bucket for Staging Contents :-

```yaml
gsutil mb -l ${PROJECT} gs://${PROJECT}/
mkdir DFaudio
```

### Task - 6 : Deploy a Cloud Dataflow Pipeline :-

Go to the saf-longrun-job-dataflow directory.

```yaml
python -m virtualenv env -p python3
source env/bin/activate
pip install apache-beam[gcp]
pip install dateparser

python saflongrunjobdataflow.py \
--project=[PROJECT_ID] \
--region=us-central1 \
--input_topic=projects/${PROJECT}/topics/helpdesk \
--runner=DataflowRunner \
--temp_location=gs://[BUCKET_NAME_FOR_STAGING_CONTENT]/[FOLDER] \
--output_bigquery=${PROJECT}:saf.transcripts \
--requirements_file=requirements.txt
```

### Task - 7 : Upload Sample Audio Files for Processing :-

Mono flac audio sample :-

```yaml
# mono flac audio sample
gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:false -h x-goog-meta-pubsubtopicname:helpdesk -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_mono.flac gs://${PROJECT}
```

Stereo wav audio sample :-

```yaml
# stereo wav audio sample
gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:true -h x-goog-meta-pubsubtopicname:helpdesk -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_stereo.wav gs://${PROJECT}
```

After uploading sample audio files for processing, we have to wait....until we see output in bigquery > dataset > table.

### Task - 8 : Run a Data Loss Prevention Job :-

Run the following query in the BigQury query editor :-

```yaml
select * from (SELECT entities.name,entities.type, COUNT(entities.name) AS count FROM saf.transcripts, UNNEST(entities) entities GROUP BY entities.name, entities.type ORDER BY count ASC ) Where count > 5
```

### Congratulations!

Follow on Twitter @techiezilla , Instagram @techiezilla.
