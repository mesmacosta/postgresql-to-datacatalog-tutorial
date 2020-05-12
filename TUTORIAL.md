<!---
Note: This tutorial is meant for Google Cloud Shell, and can be opened by going to
http://gstatic.com/cloudssh/images/open-btn.svg)](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/mesmacosta/postgresql-to-datacatalog-tutorial&tutorial=TUTORIAL.md)--->
# PostgreSQL to Data Catalog Tutorial

<!-- TODO: analytics id? -->
<walkthrough-author name="mesmacosta@gmail.com" tutorialName="PostgreSQL to Data Catalog Tutorial" repositoryUrl="https://github.com/mesmacosta/postgresql-to-datacatalog-tutorial"></walkthrough-author>

## Intro

This tutorial will walk you through the execution of the PostgreSQL to Data Catalog Tutorial.

## Environment

Let's start by setting up your environment.

## Set Up your Project

Start by setting your project ID. Replace the placeholder to your project.
```bash
gcloud config set project MY_PROJECT_PLACEHOLDER
```

Next load it in a environment variable.
```bash
export PROJECT_ID=$(gcloud config get-value project)
```

## Create the PostgreSQL Database

You can use the open source scripts at [Cloud SQL PostgreSQL Tooling](https://github.com/mesmacosta/cloudsql-postgresql-tooling) to create and populate your PostgreSQL instance.

Clone the github repository:
```bash
git clone https://github.com/mesmacosta/cloudsql-postgresql-tooling
```
Go to the cloned repo directory:
```bash
cd cloudsql-postgresql-tooling
```

Next execute the `init-db.sh` script.
This will create your PostgreSQL instance and populate it with random schema.
```bash
source init-db.sh
```

## Set Up the Service Account

Create a Service Account.
```bash
gcloud iam service-accounts create postgresql2dc-credentials \
--display-name  "Service Account for PostgreSQL to Data Catalog connector" \
--project $PROJECT_ID
```

Next create and download the Service Account Key.
```bash
gcloud iam service-accounts keys create "postgresql2dc-credentials.json" \
--iam-account "postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" 
```

Next add Data Catalog admin role to the Service Account.
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member "serviceAccount:postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
--quiet \
--project $PROJECT_ID \
--role "roles/datacatalog.admin"
```

## Execute the PostgreSQL connector

You can build the PostgreSQL connector yourself by going to
[this GitHub repository](https://github.com/GoogleCloudPlatform/datacatalog-connectors-rdbms/tree/master/postgresql2datacatalog).

To facilitate its usage, we are going to use a docker image. 
The environment variables were loaded by the `init-db.sh` script.

Execute the connector:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/postgresql2datacatalog:stable \
--datacatalog-project-id=$PROJECT_ID \
--datacatalog-location-id=us-central1 \
--postgresql-host=$public_ip_address \
--postgresql-user=$username \
--postgresql-pass=$password \
--postgresql-database=$database
```

## Check the results of the script

After the script finishes, you can go to Go to Data Catalog
[Search UI](https://console.cloud.google.com/datacatalog?q=system=postgresql)
 and search for PostgreSQL metadata.

## Cleaning up

The easiest way to avoid incurring charges to your Google Cloud account for the resources used in this tutorial is to delete 
the project you created. Otherwise you can run the clean up scripts below.

**To delete the project**, follow the steps below:

1.  In the Cloud Console, [go to the Projects page](https://console.cloud.google.com/iam-admin/projects).

2.  In the project list, select the project that you want to delete and click **Delete project**.

    ![N|Solid](https://storage.googleapis.com/gcp-community/tutorials/partial-redaction-with-dlp-and-gcf/img_delete_project.png)
    
3.  In the dialog, type the project ID, and then click **Shut down** to delete the project.

**To delete the created resources** and mantain the project,
 follow the steps below:

Delete the PostgreSQL metadata:
```bash
./cleanup-db.sh
```

Execute the cleaner container:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/postgresql-datacatalog-cleaner:stable \
--datacatalog-project-ids=$PROJECT_ID \
--rdbms-type=postgresql \
--table-container-type=schema
```

Delete the PostgreSQL database:
```bash
./delete-db.sh
```
## Search for the PostgreSQL Entries

Go to Data Catalog search UI:
[Search UI](https://console.cloud.google.com/datacatalog?q=system=postgresql)

Check the search results, and verify that there are no results. Entries for the system `postgresql` 
have been deleted.

## Congratulations!

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

You've successfully finished the PostgreSQL to Data Catalog Tutorial.
