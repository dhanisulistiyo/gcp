# Serverless Cloud Run Development: Challenge Lab

Learn how to do the following using Cloud Run by connecting and leveraging data stored in Cloud Storage, building a resilient, asynchronous system with Cloud Run and Pub/Sub, building a REST API gateway using Cloud Run, building and exposing service using Cloud Run.

## Task 1

| FIELD	                | VALUE                              |
| --------------------- | ---------------------------------- |
| Billing Image         | billing-staging-api:0.1            |
| Billing Service       | public-billing-service-670         |
| Authentication        | unauthenticated                    |
| Code	                | pet-theory/lab07/unit-api-billing  |

- Build an image using Cloud Build
- Deploy a Cloud Run service as an unauthenticated service
- Test service responds when the endpoint is accessed

```bash
cd ~/pet-theory/lab07/unit-api-billing

gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1

gcloud run deploy public-billing-service-670 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1 \
  --platform managed \
  --region us-central1 \
  --max-instances=1
```

## Task 2

| FIELD	                | VALUE                                      |
| --------------------- | ------------------------------------------ |
| Image Name            | frontend-staging:0.1                       |
| Service Name          | frontend-staging-service-155               |
| Authentication        | unauthenticated                            |
| Code	                | pet-theory/lab07/staging-frontend-billing  |

- Build an image using Cloud Build
- Deploy the image to Cloud Run as unauthenticated service
- Service should respond when the endpoint is accessed

```bash
cd ~/pet-theory/lab07/staging-frontend-billing

gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1

gcloud run deploy frontend-staging-service-155 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 \
  --platform managed \
  --region us-central1 \
  --max-instances=1
```

## Task 3

| FIELD	                | VALUE                                      |
| --------------------- | ------------------------------------------ |
| Image Name            | billing-staging-api:0.2                    |
| Service Name          | private-billing-service-928                |
| Repository            | gcr.io                                     |
| Authentication        | authenticated                              |
| Code	                | pet-theory/lab07/staging-api-billing       |

- Delete the existing Billing Service
- Build an image using Cloud Build
- Deploy the image to Cloud Run requiring authentication
- Assign the SERVICE_URL to a environment variable

```bash
cd ~/pet-theory/lab07/staging-api-billing

gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2

gcloud run deploy private-billing-service-928 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2 \
  --platform managed \
  --region us-central1 \
  --no-allow-unauthenticated \
  --max-instances=1

```
optional 

```bash
BILLING_URL=$(gcloud run services describe private-billing-service-928 \
--platform managed \
--region us-central1 \
--format "value(status.url)")

curl -X get -H "Authorization: Bearer $(gcloud auth print-identity-token)" $BILLING_URL
```

## Task 4

| FIELD	                | VALUE                       |
| --------------------- | --------------------------- |
| Service Account       | billing-service-sa-928      |
| Display Name	        | Billing Service Cloud Run   |
| Service Name          | billing-service             |
| Role	                | N/A                         |

- Create a Service Account

```bash
gcloud iam service-accounts create billing-service-sa-928 --display-name "Billing Service Cloud Run"
```

## Task 5

| FIELD	                | VALUE                                      |
| --------------------- | ------------------------------------------ |
| Image Name            | billing-prod-api:0.1                       |
| Service Name          | billing-prod-service-751                   |
| Repository            | gcr.io                                     |
| Authentication        | authenticated                              |
| Code	                | pet-theory/lab07/prod-api-billing          |
| Service Account       | billing-service-sa-154                     |

- Deploy the image to Cloud Run
- Enable Authentication
- Enable Service Account
- Service should respond when the endpoint is accessed

```bash
cd ~/pet-theory/lab07/prod-api-billing

gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1

gcloud run deploy billing-prod-service-751 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1 \
  --platform managed \
  --region us-central1 \
  --no-allow-unauthenticated \
  --max-instances=1

gcloud iam service-accounts create billing-service-sa-154 --display-name "Billing Service Cloud Run"

gcloud run services add-iam-policy-binding billing-prod-service-751 \
--member=serviceAccount:billing-service-sa-154@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
--role=roles/run.invoker
```

optional

```bash
PROD_BILLING_URL=$(gcloud run services \
describe private-billing-service-207 \
--platform managed \
--region us-central1 \
--format "value(status.url)")

curl -X get -H "Authorization: Bearer \
$(gcloud auth print-identity-token)" \
$PROD_BILLING_URL
```

## Task 6

| FIELD	                | VALUE                               |
| --------------------- | ----------------------------------- |
| Service Account       | frontend-service-sa-267             |
| Display Name	        | Billing Service Cloud Run Invoker   |
| Service Name          | frontend-prod-service               |
| Role	                | run.invoker                         |

- Create Service Account
- Apply Service Account for Frontend Service
- Give Service Account run.invoker permission
- Bind Account to Service

```bash
gcloud iam service-accounts create frontend-service-sa-267 --display-name "Billing Service Cloud Run Invoker"
```

## Task 7

| FIELD	                | VALUE                                      |
| --------------------- | ------------------------------------------ |
| Image Name            | frontend-prod:0.1                          |
| Service Name          | frontend-prod-service-814                  |
| Repository            | gcr.io                                     |
| Authentication        | unauthenticated                            |
| Code	                | pet-theory/lab07/prod-frontend-billing     |
| Service Account       | frontend-service-sa-267                    |

- Deploy the image to Cloud Run
- Enable Authentication
- Enable Service Account
- Service should respond when the endpoint is accessed

```bash
cd ~/pet-theory/lab07/prod-frontend-billing

gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1

gcloud run deploy frontend-prod-service-814 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1 \
  --platform managed \
  --region us-central1 \
  --max-instances=1

gcloud run services add-iam-policy-binding frontend-prod-service-814 \
--member=serviceAccount:frontend-service-sa-267@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
--role=roles/run.invoker
```