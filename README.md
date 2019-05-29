# Census RM QID Batch Runner [![Build Status](https://travis-ci.com/ONSdigital/census-rm-qid-batch-runner.svg?branch=master)](https://travis-ci.com/ONSdigital/census-rm-qid-batch-runner)

A python script to tell RM to generate unaddressed QID/UAC pairs from a CSV config file.

## Dependencies

Install dependencies with 
```bash
make install
```

The script needs a rabbit instance to talk to you can use the instance from [census-rm-docker-dev]() or the docker-compose file here to run just a rabbit container with

```bash
docker-compose up -d
``` 

Look in [rabbit_context.py](/rabbit_context.py) to see the rabbit config.

## Run Locally
Run the scripts locally with

```bash
pipenv run python generate_qid_batch.py <path to config csv>
```

to request the qid/uac pairs, then once all those message have been ingested

```bash
pipenv run python generate_print_files.py <path to config csv> <path to output directory>
```

this will read the generated qid/uac pairs and generate the print and manifest files in the specified directory

## Run in Kubernetes

### Prerequisites
The generate print files script needs a GCS bucket named `<PROJECT_ID>-print-files` in the project GCloud pointing at and bucket get/object create permissions.
To set this up:

1. Navigate to the storage section in the GCP web UI
1. Click create bucket and name it `<PROJECT_ID>-print-files`
1. In the new bucket, go to the permissions tab and edit the permissions of the `compute@...` service account to include `Storage Legacy Bucket Reader` and `Storage Object Creator`.

Also needs rabbit and case-processor working in order to generate the print files.

### Start a pod
To start up the pod in Kubernetes, point your kubectl context at the cluster you wish to run in, then run
```bash
make start-pod
```

Then once the pod is ready connect to it with
```bash
make connect-to-pod
```
This should connect you to a bash shell in the pod

### Request the QID/UAC pairs
Once you're in the pods shell you can queue the messages to the request the QID/UAC pairs by running
```bash
python generate_qid_batch.py unaddressed_batch.csv
```

You can watch the `unaddressedRequestQueue` from the rabbit management console to see when all these messages have been ingested by the case-processor.

If you need to run a different config file, you can copy it into the pod once it is started with kubectl. 
While the pod is running, in a different shell window run
```bash
kubectl cp <local path to csv> qid-batch-runner:/app
```

Return to the connected shell in the pod, the file should then be available in the pod in `/app`.

### Generate the print files
Once all the QID/UAC pair request messages have been ingested, you can generate the print files with
```bash
python generate_print_files.py unaddressed_batch.csv print_files
```

This should write the files out to the GCS bucket.

When you are finished exit the pod with `ctrl + D` or by running `exit`. This will disconnect, then you can delete the pod with `make delete-pod`.


## Tests and Checks

Run the unit tests and style/safety checks with

```bash
make test
```

## Docker
Build the image tagged as `eu.gcr.io/census-rm-ci/rm/census-rm-qid-batch-runner:latest` with
```bash
make build
```
