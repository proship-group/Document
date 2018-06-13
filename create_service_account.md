
# Create a GCP Service Account

> NOTE: **Anyone with the secret key can access your GCP Project with those permissions.**

#### via Google Cloud Console


    1. Go to https://console.cloud.google.com/iam-admin/serviceaccounts/project and click `CREATE SERVICE ACCOUNT`
    2. On the dialog, do the folling:
        2.1. Enter `Service account name`
        2.2. Select Role > `Storage Admin`. 
        2.3. Check the `Furnish a new private key`; and 
        2.4. Click `CREATE`
    3. After creating, a `json` file will be automatically downloaded. Please keep a copy and secure it.


#### via `gcloud` CLI

Set parameters first:

```
# set parameters
$ export PROJECT_ID=<project id>
$ export SERVICE_ACCOUNT_NAME=<service account name>
$ export SERVICE_ACCOUNT=$SERVICE_ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com
$ export SERVICE_ACCOUNT_KEYFILE=<filename.json>
$ SERVICE_ROLE=<roles/service.role>
# example: for gcs management use `roles/storage.admin`
```

Then, create the service account and assign a role/s

```
# create the service account
$ gcloud iam service-accounts create $SERVICE_ACCOUNT_NAME --display-name $SERVICE_ACCOUNT_NAME
$ gcloud iam service-accounts keys create $SERVICE_ACCOUNT_KEYFILE --iam-account $SERVICE_ACCOUNT
$ gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$SERVICE_ACCOUNT --role $SERVICE_ROLE
```

> NOTE: **Anyone with the secret key can access your GCP Project with those permissions.**