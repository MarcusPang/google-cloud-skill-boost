# Task 1 and 2

```bash
export PROJECT=$PROJECT
gcloud iam service-accounts create my-account --display-name my-account

gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/bigquery.admin
gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/storage.admin

gcloud iam service-accounts keys create key.json --iam-account=my-account@$PROJECT.iam.gserviceaccount.com

export GOOGLE_APPLICATION_CREDENTIALS=key.json
```

# Task 3+

The documentation is in python2

```python
# TBD: Create a Vision API image object called image_object
# Ref: https://googleapis.dev/python/vision/latest/gapic/v1/types.html#google.cloud.vision_v1.types.Image
image_object = vision.types.Image()
image_object.content = file_content

# TBD: Detect text in the image and save the response data into an object called response
# Ref: https://googleapis.dev/python/vision/latest/gapic/v1/api.html#google.cloud.vision_v1.ImageAnnotatorClient.document_text_detection
response = vision_client.document_text_detection(image=image_object)
```

Translation part, remember to unhide second last line

```python
# TBD: For non EN locales pass the description data to the translation API
# ref: https://googleapis.dev/python/translation/latest/client.html#google.cloud.translate_v2.client.Client.translate
# Set the target_language locale to 'en')
translation = translate_client.translate(desc, target_language="en")
```

Then run the query

```sql
SELECT locale, COUNT(locale) as lcount FROM image_classification_dataset.image_text_detail GROUP BY locale ORDER BY lcount DESC
```
