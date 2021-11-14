# AI and Machine Learning tools on Google CLoud
- AI Platform enables for models to be deployed and trained locally and on the cloud
- Dataprep is used to prepare and clean data
- Dataflow

## Dataprep
Cloud Dataprep by Trifacta is an intelligent data service for visually exploring, cleaning, and preparing data for analysis. Cloud Dataprep is serverless and works at any scale. There is no infrastructure to deploy or manage. Easy data preparation with clicks and no code!
- Create flows which allows for data transformation through recipes
- These recipes can be merged and saved for future use

## Dataflow Pipeline
```bash
# create dataset with BigQuery 
bq mk $BIG_QUERY_TABLE
bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t $BIG_QUERY_TABLE.realtime

# storage bucket
gsutil mb gs://$BUCKET_NAME/

# settings
Dataflow template: Pub/Sub Topic to BigQuery
Input Pub/Sub topic: projects/pubsub-public-data/topics/$BIG_QUERY_TABLE-realtime
BigQuery Output table: $PROJECT_ID:$BIG_QUERY_TABLE.realtime
temp location: gs://$BUCKET_NAME/temp

# run query in BigQuery to test
SELECT * FROM `$PROJECT_ID.$BIG_QUERY_TABLE.realtime` LIMIT 1000
```

## Dataproc
Cloud Dataproc is a fast, easy-to-use, fully-managed cloud service for running Apache Spark and Apache Hadoop clusters in a simpler, more cost-efficient way. Cloud Dataproc clusters can be created quickly and resized any time.

```bash
# config
gcloud config set dataproc/region us-central1
gcloud dataproc clusters create example-cluster --worker-boot-disk-size 500

# example job which calculates pi
gcloud dataproc jobs submit spark --cluster example-cluster \
--class org.apache.spark.examples.SparkPi \
--jars file:///usr/lib/spark/examples/jars/spark-examples.jar -- 1000
gcloud dataproc clusters update example-cluster --num-workers 4

# update cluster after deployment
gcloud dataproc clusters update example-cluster --num-workers 4
```

## Cloud Natural Language API
Cloud Natural Language API lets you extract information about people, places, events, (and more) mentioned in text documents, news articles, or blog posts.

Features:
- Syntax Analysis: Extract tokens and sentences, identify parts of speech (PoS) and create dependency parse trees for each sentence.
- Entity Recognition: Identify entities and label by types such as person, organization, location, events, products and media.
- Sentiment Analysis: Understand the overall sentiment expressed in a block of text.
- Content Classification: Classify documents in predefined 700+ categories.
- Multi-Language: Enables you to easily analyze text in multiple languages including English, Spanish, Japanese, Chinese (Simplified and Traditional), French, German, Italian, Korean and Portuguese.
- Integrated REST API: Access via REST API. Text can be uploaded in the request or integrated with Cloud Storage.

```bash
# setup
export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)
gcloud iam service-accounts keys create ~/key.json --iam-account quickstart@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS="/home/USER/key.json"

# ssh into compute engine and test entity analysis
gcloud ml language analyze-entities --content="Michelangelo Caravaggio, Italian painter, is known for 'The Calling of Saint Matthew'." > result.json
```

## Cloud Speech API
First create API key for curl under API & Services, then ssh inside the Compute Engine instance

```bash 
# create request
cat > request.json <<EOF
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-samples-tests/speech/brooklyn.flac"
  }
}
EOF

# export API Key inside ssh instance
export API_KEY=<YOUR_API_KEY>

# call Speech API and save to result.json
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json
```

## Cloud Video Intelligence
Google Cloud Video Intelligence makes videos searchable and discoverable by extracting metadata with an easy to use REST API. You can now search every moment of every video file in your catalog. It quickly annotates videos stored in Cloud Storage, and helps you identify key entities (nouns) within your video; and when they occur within the video. Separate signal from noise by retrieving relevant information within the entire video, shot-by-shot, -or per frame.

```bash
# setup iam account 
gcloud iam service-accounts create quickstart
gcloud iam service-accounts keys create key.json --iam-account quickstart@<your-project-123>.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file key.json
gcloud auth print-access-token

# create request
cat > request.json <<EOF
{
   "inputUri":"gs://spls/gsp154/video/train.mp4",
   "features": [
       "LABEL_DETECTION"
   ]
}
EOF

# test video and save "name" output
curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/videos:annotate' \
    -d @request.json

# replace PROJECTS, LOCATIONS, OPERATION_NAME with "name"
curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/projects/PROJECTS/locations/LOCATIONS/operations/OPERATION_NAME'
```
