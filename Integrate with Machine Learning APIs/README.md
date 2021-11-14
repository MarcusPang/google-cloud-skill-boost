# Introduction to APIs to Google
## Service Accounts
A service account is a special type of Google account that belongs to your application or a virtual machine (VM) instead of to an individual end user. Your application assumes the identity of the service account to call Google APIs, so that the users aren't directly involved.

https://developers.google.com/oauthplayground/ generates OAuth tokens for APIs

Possible to create buckets and upload files using OAuth

# Extract, Analyze, and Translate Text from Images with the Cloud ML APIs
Example using OCR API to retrieve text detected
```bash
cat > ocr-request.json <<EOF
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://$BUCKET_NAME/sign.jpg"
          }
        },
        "features": [
          {
            "type": "TEXT_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
EOF

curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
```
Translation API
```bash
cat > translation-request.json <<EOF
{
  "q": "your_text_here",
  "target": "en"
}
EOF

STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json

curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json
```
Natural Language API using analyzeEntities method
```bash
cat > nl-request.json <<EOF
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"your_text_here"
  },
  "encodingType":"UTF8"
}
EOF

STR=$(jq .data.translations[0].translatedText  translation-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" nl-request.json

curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl-request.json
```

# Classify Text into Categories with the Natural Language API
Example
```bash
cat > request.json <<EOF
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"A Smoky Lobster Salad With a Tapa Twist. This spin on the Spanish pulpo a la gallega skips the octopus, but keeps the sea salt, olive oil, pimentón and boiled potatoes."
  }
}
EOF

curl "https://language.googleapis.com/v1/documents:classifyText?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json > result.json
```

Authenticate Natural Language API & BigQuery using python
```bash
gcloud iam service-accounts create my-account --display-name my-account
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:my-account@$PROJECT_ID.iam.gserviceaccount.com --role=roles/bigquery.admin
gcloud iam service-accounts keys create key.json --iam-account=my-account@$PROJECT_ID.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS=key.json
```
Sample python script
```python
from google.cloud import storage, language, bigquery
# Set up our GCS, NL, and BigQuery clients
storage_client = storage.Client()
nl_client = language.LanguageServiceClient()
# TODO: replace YOUR_PROJECT with your project name below
bq_client = bigquery.Client(project='$PROJECT_ID')
dataset_ref = bq_client.dataset('news_classification_dataset')
dataset = bigquery.Dataset(dataset_ref)
table_ref = dataset.table('article_data')
table = bq_client.get_table(table_ref)
# Send article text to the NL API's classifyText method
def classify_text(article):
        response = nl_client.classify_text(
                document=language.Document(
                        content=article,
                        type_=language.Document.Type.PLAIN_TEXT
                )
        )
        return response
rows_for_bq = []
files = storage_client.bucket('qwiklabs-test-bucket-gsp063').list_blobs()
print("Got article files from GCS, sending them to the NL API (this will take ~2 minutes)...")
# Send files to the NL API and save the result to send to BigQuery
for file in files:
        if file.name.endswith('txt'):
                article_text = file.download_as_bytes()
                nl_response = classify_text(article_text)
                if len(nl_response.categories) > 0:
                        rows_for_bq.append((str(article_text), nl_response.categories[0].name, nl_response.categories[0].confidence))
print("Writing NL API article data to BigQuery...")
# Write article text + category data to BQ
errors = bq_client.insert_rows(table, rows_for_bq)
assert errors == []
```

```bash
python3 classify-text.py
```

# Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API
```bash
# LABEL_DETECTION can be changed to WEB_DETECTION
cat > request.json <<EOF
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://$BUCKET_NAME/donuts.png"
          }
        },
        "features": [
          {
            "type": "LABEL_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
EOF

# Face detection
cat > request.json <<EOF 
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://qwiklabs-gcp-00-81d8a8584a10/selfie.png"
          }
        },
        "features": [
          {
            "type": "FACE_DETECTION"
          },
          {
            "type": "LANDMARK_DETECTION"
          }
        ]
      }
  ]
}
EOF
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
```

# Awwvision: Cloud Vision API from a Kubernetes Cluster

```bash
# setup
gcloud config set compute/zone us-central1-a
gcloud container clusters create awwvision \
    --num-nodes 2 \
    --scopes cloud-platform
gcloud container clusters get-credentials awwvision
kubectl cluster-info

# update python and change venv
sudo apt-get update
sudo apt-get install -y virtualenv
virtualenv -p python3 venv
source venv/bin/activate

# run
gsutil -m cp -r gs://spls/gsp066/cloud-vision .
cd cloud-vision/python/awwvision
make all
```