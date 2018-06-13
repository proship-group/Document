# Create GCS Buckets

#### via Google Cloud Console

You need to create two (2) buckets. The bucket names should be `${GCS_BUCKET_NAME}` and `${GCS_BUCKET_NAME}-mailfetcher`

1. [Open the Cloud Storage Browser](https://console.cloud.google.com/storage/browser)
2. Click **Create bucket**.
3. Enter the Bucket **Name**.
4. For the **Default storage class**, choose **Regional**.
5. Under **Location**, select the **zone of the cluster**.
6. Click **Create**.

#### via `gsutil` CLI

To create a bucket using `gsutil` command-line tool, execute the following command

```bash
$ gsutil mb -c regional -l $(awk -F'-' '{a=$1"-"$2; print a}' <<< ${ZONE}) gs://${GCS_BUCKET_NAME}/
$ gsutil mb -c regional -l $(awk -F'-' '{a=$1"-"$2; print a}' <<< ${ZONE}) gs://${GCS_BUCKET_NAME}-mailfetcher/
```

##### References:
 - https://cloud.google.com/storage/docs/creating-buckets