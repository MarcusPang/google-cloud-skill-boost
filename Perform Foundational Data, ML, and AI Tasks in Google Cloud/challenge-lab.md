# Task 1
```bash
bq mk lab
gsutil mb gs://$BUCKET_NAME

gsutil cp gs://cloud-training/gsp323/lab.csv .
gsutil cp gs://cloud-training/gsp323/lab.schema .
cat lab.schema
```
Create customers datatable with lab.schema and lab.csv then make sure to run the _batch_ job to finish the task

# Task 2
```bash
gcloud config set dataproc/region us-central1
gcloud dataproc clusters create example-cluster --worker-boot-disk-size 500

# go to example-cluster under compute engine and add data.txt
hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt

# submit job
gcloud dataproc jobs submit spark --cluster example-cluster --class org.apache.spark.examples.SparkPageRank --jars file:///usr/lib/spark/examples/jars/spark-examples.jar --max-failures-per-hour=1 -- /data.txt
```

# Task 3
Import dataset and pass into recipe
```bash
# wrangle Recipe
rename type: manual mapping: [column2,'runid'], [column3,'userid'], [column4,'labid'], [column5,'lab_title'], [column6,'start'], [column7,'end'], [column8,'time'], [column9,'score'], [column10,'state']
filter type: custom rowType: single row: state == 'SUCCESS' action: Keep
filter type: contains col: score contains: /(^0$|^0\.0$)/ action: Delete
```
# Task 4
Only the first one works
```bash
# cloud natural language api
gcloud iam service-accounts create sample --display-name "my natural language service account"
gcloud iam service-accounts keys create ~/key.json --iam-account sample@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS="/home/$USER/key.json"
gcloud auth activate-service-account sample@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --key-file=$GOOGLE_APPLICATION_CREDENTIALS

gcloud ml language analyze-entities --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > task4-cnl.result

gcloud auth login

gsutil cp task4-cnl.result gs://qwiklabs-gcp-04-48b4ea256a62-marking/task4-cnl.result

# cloud speech api
cat > gcs_request.json <<EOF
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-training/gsp323/task4.flac"
  }
}
EOF
curl -s -X POST -H "Content-Type: application/json" --data-binary @gcs_request.json "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > task4-gcs.result
gsutil cp task4-gcs.result gs://qwiklabs-gcp-04-48b4ea256a62-marking/task4-gcs.result


# video
# setup iam account 
gcloud iam service-accounts create quickstart
gcloud iam service-accounts keys create key.json --iam-account quickstart@qwiklabs-gcp-04-48b4ea256a62.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file key.json
gcloud auth print-access-token

# create request
cat > gvi_request.json <<EOF
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
    -d @gvi_request.json > temp.txt

# replace PROJECTS, LOCATIONS, OPERATION_NAME with "name"
curl -s -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    'https://videointelligence.googleapis.com/v1/projects/561668572571/locations/asia-east1/operations/8514382717351512248' > task4-gvi.result
gsutil cp task4-gvi.result gs://qwiklabs-gcp-04-48b4ea256a62-marking/task4-gvi.result
gsutil cp temp.txt gs://qwiklabs-gcp-04-48b4ea256a62-marking/task4-gvi.result
```