# Task 1
Compute Engine > Create Instance > Series: N1, machine: f1 micro

# Task 2
```bash
gcloud config set compute/zone us-east1-b
gcloud container clusters create nucleus-jumphost-webserver1
gcloud container clusters get-credentials nucleus-jumphost-webserver1
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8082
kubectl get service
```

# Task 3
After storing startup.sh
```bash
# instance template
gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh

# target pool
gcloud compute target-pools create --region=us-east1 nginx-pool

# managed instance group
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool 

# gcloud compute instances list

# firewall
gcloud compute firewall-rules create allow-tcp-rule-527 --allow tcp:80 

gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool

# gcloud compute forwarding-rules 

# healthcheck
gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

# backend service
gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

# URL map and target HTTP proxy
gcloud compute url-maps create web-map \
--default-service nginx-backend 

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

# forwarding rule
gcloud compute forwarding-rules create http-content-rule\
--global \
--target-http-proxy http-lb-proxy \
--ports 80 

# gcloud compute forwarding-rules list
```