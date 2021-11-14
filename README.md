# Google Cloud Skill Boost

Courses that <a href="https://www.cloudskillsboost.google/public_profiles/eb520619-9940-4183-8f7b-8c5f590ce045">I have completed</a> during the month where I had a free subscription!

## Helpful commands
```bash
# details on project incl. zone and region
gcloud compute project-info describe --project $PROJECT_ID

# create bq
bq mk $BQ_NAME # (optional) --time_partitioning_field timestamp --schema ...

# create bucket
gsutil mb gs://$BUCKET_NAME/

# create kubernetes cluster
gcloud container clusters create $CLUSTER_NAME
gcloud container clusters get-credentials $CLUSTER_NAME

# gcloud interactive
sudo apt-get install google-cloud-sdk
gcloud beta interactive
gcloud compute instances describe $VM_NAME

# connect VM using ssh
gcloud compute ssh gcelab2 --zone $ZONE
```