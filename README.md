# MGMT 590 (Prod Scale Data Products) - Assignment #4 Batch Pipeline

This repo contains the master solution for Assignment #4 in the Summer 2021 edition of MGMT 590 (Production Scale Data Products) at Purdue University. Specifically, this repo includes code and config for running a Bacth Pipeline using Dockerhub and Pachyderm for question answering using pre-trained models from [Hugging Face Transformers](https://huggingface.co/models).

## Outline

- Part 1 - Overview 

- Part 2 - Architecture

- Part 3 - Docker Hub

- Part 4 - Pachyderm

- Part 5 - Environment variables and scripts that need to be in place

- Part 6 - Steps Followed

- Part 7 - Create Service Accounts/Roles

- Part 8 - Run the file upload REST API Route

- Part 9 - Connect to Pachctl and How to Deploy batch pipeline using Pachyderm Hub

- Part 10 - Create and Manage Secrets in Pachyderm

- Part 11 - Running the commands in pachctl



## 1.Overview 

![architecture](./images/K8.png)

Batch pipelining is very useful in our case, since processing large amount of data in a single API call on a single system would not be feasible. We are using Kubernetes + Pachyderm for our pipeline. Kubernetes helps us in optimizing the resource utilization by automatically routing a particular container to the most suitable available node in the cluster and minimizing CPU and GPU wastage. Kubernetes also helps in parallel processing and reducing the processing time. We have used Pachyderm to incorporate the pipeline flow, since we need to connect information from different containers running on different nodes. Below are the 2 pipelines which we have created in our project:

Pipeline 1: This pipeline refers to the GCS bucket (contains the uploaded CSV files) periodically every 5 minutes and runs the question answering function on the new data available in the bucket. Then the pipeline stores the obtained answers in the output repo of Pachyderm.

Pipeline 2: This pipeline collects the answers in the output repo of Pachyderm and inserts them to a Cloud SQL (PostgreSQL) through SSL connection. All the questions answered by the API are stored in this database.

This is a REST API integrated with batch pipelining which can answer questions when provided with a context using one of the state-of-the-art models in Natural Language Processing (NLP). Question answering is a task in information retrieval and Natural Language Processing (NLP) that investigates software that can answer questions asked by humans in natural language. In Extractive Question Answering, a context is provided so that the model can refer to it and make predictions on where the answer lies within the passage. It uses “distilled-bert” algorithm to get the answers to all the questions in the uploaded file. But the standout functionality of this API is, all the steps of the question answering process are automated by integrating GitHub, Google cloud, Docker Hub, and Pachyderm. We also used PostgresSQL from google cloud for storing our answering records. We are using our Laptop/Cloud Shell to push our code GitHub which is version controlled and contains the repositories corresponding to our REST API, Web App(These were a part of our assignment 3), additionally which we have included a Batch processing Pipeline Repo. The REST API and Web App are triggered via GitHub actions workflow and build and deployed to Google cloud Run. We also have a Cloud SQL instance of POSTGRES for our CRUD operations on flask and streamlit. There's also a container Registry which contains the images of our build deploys. The GitHub actions workflow for Batch Pipelines builds and deploys to Docker Hub instead of Google cloud. We're using Pachyderm Hub to host Pachyderm and Kubernetes Cluster.


## 2.Architeture

![architecture](./images/architect.png)

We are using our Laptop/Cloud Shell to push our code Github which is version controlled and contains the repositories corresponding to our REST API,Web App(These were a part of of our assignment 3), additionally which we have included a Batch processing Pipeline Repo. The REST API and Web App are triggred via github actions workflow and build and deployed to Google cloud Run. 

We also have a Cloud SQL instance of POSTGRES for our CRUD operations on flask and streamlit. There's also a container Registry which contains the images of our buid deploys.The github actions workflow for Batch Pipelines builds and deploys to DockerHub instead of Google cloud. We're using Pachyderm Hub to host Pachyderm and Kubernetes Cluster.

In our assignment we are going to have a GCS(Google Cloud Storage), which is going to have .csv files stored and these csv will be put in by our REST API by a new route /uploads  allowing a user to upload a CSV file with a column of questions and a column of contexts. This CSV file should be pushed by the REST API into a GCS bucket (where your Pachyderm pipeline will read it in for processing). The new route should return a 200 OK code and a message to the user indicating that it successfully pushed the file to the GCS bucket


# 3.Docker Hub

![DOCKER](./images/docker.jpeg)

Docker Hub ``is a cloud-based registry service`` which provides a centralized resource for container image discovery, distribution and change management, user and team collaboration, and workflow automation throughout the development pipeline. There are both private and public repositories. Private repository can only be used by people 
within their own organization.

Docker Hub is ``hardcoded into Docker as the default registry``, which means that the docker pull command will initialize the download automatically from Docker Hub.

# Pachyderm: Data Versioning, Data Pipelines, and Data Lineage

![Pachyderm](./images/pachyderm.jpeg)

Pachyderm is a tool for version-controlled, automated, end-to-end data pipelines for data science. If you need to chain together data scraping, ingestion, cleaning, munging, wrangling, processing,
modeling, and analysis in a sane way, while ensuring the traceability and provenance of your data, Pachyderm is for you. If you have an existing set of scripts which do this in an ad-hoc fashion and you're looking
for a way to "productionize" them, Pachyderm can make this easy for you.

## Features

- Containerized: Pachyderm is built on Docker and Kubernetes. Whatever
  languages or libraries your pipeline needs, they can run on Pachyderm which
  can easily be deployed on any cloud provider or on prem.
- Version Control: Pachyderm version controls your data as it's processed. You
  can always ask the system how data has changed, see a diff, and, if something
  doesn't look right, revert.
- Provenance (aka data lineage): Pachyderm tracks where data comes from. Pachyderm keeps track of all the code and  data that created a result.
- Parallelization: Pachyderm can efficiently schedule massively parallel
  workloads.
- Incremental Processing: Pachyderm understands how your data has changed and
  is smart enough to only process the new data
  

## 4.Environment variables and scripts that need to be in place 


```shell 
    
DOCKERHUB_TOKEN
    
DOCKERHUB_USERNAME
    
GCS_CREDS
```

Here, the DOCKERHUB_TOKEN is the Docker Hub password and DOCKERHUB_USERNAME is the user id of the DockerHub. This is needed as there's an authentication process involved in our workflow. The third secret GCS_CREDS is used to set the environment variable GOOGLE_APPLICATION_CREDENTIALS.

Apart from these there are certain pipeline specs that we need to create that are pipeline1_spec.json and pipeline2_spec.json which are used to create the pipelines respectively. There are scripts which aid us in setting the environment variables during the pachyderm run of our images from DockerHub. Below is the folder structure , for our project folder:

```shell 
    
File Type
Name

YAML File
build_and_push.yml

Pipeline 1
 

Pipeline 1
Dockerfile

Pipeline 1
pipeline1_spec.json

Pipeline 1
question_answer_pd.py

Pipeline 1
requirements.txt

Pipeline 2
 

Pipeline 2
Dockerfile

Pipeline 2
answer_insert.py

Pipeline 2
pipeline2_spec.json

Pipeline 2
requirements.txt

Home
create_secret.sh

Home
secret_template.json

Home
secret_template_db.json

```

## DEPENDENCIES:

```python

transformers==4.6.1

flask==2.0.1

pysqlite3

psycopg2-binary

pytest==6.2.2

pandas

google-cloud-storage
```

# 5.Steps Followed

## 1. Create a GCS Bucket for Artifact Storage

We can create a GCS Bucket either using the UI and going to Cloud Storage or through console as following:

a. Log in with `gcloud`:

  ```
  $ gcloud auth login
  ```

b. Run the `gsutil mb` command to create a bucket within your GCP project, giving your project’s ID (`PROJECT_ID` below) and the name of the bucket to create (`BUCKET`):

  ```
  $ PROJECT_ID=<Insert Project ID>
  $ BUCKET=gs://<Insert Bucket Name>
  $ gsutil mb -p $PROJECT_ID $BUCKET
  ```

# 2.Cloud Storage client library

Once our bucket is setup we can use the Google cloud storage client to interatct with it.

This tutorial shows how to get started with the [Cloud Storage Python client library](https://googleapis.github.io/google-cloud-python/latest/storage/index.html).

## Create a storage bucket

Buckets are the basic containers that hold your data. Everything that you store in Cloud Storage must be contained in a bucket. You can use buckets to organize your data and control access to your data.

Start by importing the library:


```python
from google.cloud import storage
```

The `storage.Client` object uses your default project. Alternatively, you can specify a project in the `Client` constructor. For more information about how the default project is determined, see the [google-auth documentation](https://google-auth.readthedocs.io/en/latest/reference/google.auth.html).

Run the following to create a client with your default project:

## Upload a local file to a bucket

Objects are the individual pieces of data that you store in Cloud Storage. Objects are referred to as "blobs" in the Python client library. There is no limit on the number of objects that you can create in a bucket.

An object's name is treated as a piece of object metadata in Cloud Storage. Object names can contain any combination of Unicode characters (UTF-8 encoded) and must be less than 1024 bytes in length.

For more information, including how to rename an object, see the [Object name requirements](https://cloud.google.com/storage/docs/naming#objectnames).


```python
blob_name = "us-states.txt"
blob = bucket.blob(blob_name)

source_file_name = "resources/us-states.txt"
blob.upload_from_filename(source_file_name)

print("File uploaded to {}.".format(bucket.name))
```

## List blobs in a bucket


```python
blobs = bucket.list_blobs()

print("Blobs in {}:".format(bucket.name))
for item in blobs:
    print("\t" + item.name)
```

## Get a blob and display metadata

See [documentation](https://cloud.google.com/storage/docs/viewing-editing-metadata) for more information about object metadata.


```python
blob = bucket.get_blob(blob_name)

print("Name: {}".format(blob.id))
print("Size: {} bytes".format(blob.size))
print("Content type: {}".format(blob.content_type))
print("Public URL: {}".format(blob.public_url))
```

## Download a blob to a local directory


```python
output_file_name = "resources/downloaded-us-states.txt"
blob.download_to_filename(output_file_name)

print("Downloaded blob {} to {}.".format(blob.name, output_file_name))
```

## Cleaning up

### Delete a blob


```python
blob = client.get_bucket(bucket_name).get_blob(blob_name)
blob.delete()

print("Blob {} deleted.".format(blob.name))
```

## 5.Set Environment Variable

```python
export GOOGLE_APPLICATION_CREDENTIALS="KEY_PATH"
Replace KEY_PATH with the path of the JSON file that contains your service account key.

For example:


export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/service-account-file.json"
```

## 6.Create Service Accounts/Roles

We then proceed to create a service account the below three user roles as displayed in the image.

![user](./images/user.png)


## 7.Run the file upload REST API Route

We then run the file upload REST API route which leads to the uploading of the csv file to the google cloud bucket that we made earlier.A sample request and response is shown below:

![fileuplolad](./images/fileupload.png)

## 8.Connect to Pachctl and How to Deploy batch pipeline using Pachyderm Hub

![pipeline](./images/pipeline.png)

Our next step is to connect to pachctl. The below image tells the step to connect to pachctl.

The following steps tell us how to deploy our pipeline to Pachyderm hub:

![pachctl](./images/pachctl.png)

## Step 1: Write Your Analysis Code
Because Pachyderm is completely language-agnostic, the code that is used to process data in Pachyderm can be written in any language and can use any libraries of choice. Whether your code is as simple as a bash command or as complicated as a TensorFlow neural network, it needs to be built with all the required dependencies into a container that can run anywhere, including inside of Pachyderm

a)Read files from a local file system. Pachyderm automatically mounts each input data repository as /pfs/<repo_name> in the running containers of your Docker image. Therefore, the code that you write needs to read input data from this directory, similar to any other file system.

b)Write files into a local file system, such as saving results. Your code must write to the /pfs/out directory that Pachyderm mounts in all of your running containers. Similar to reading data, your code does not have to manage parallelization or sharding.

## Step 2: Build Your Docker Image
When you create a Pachyderm pipeline, you need to specify a Docker image that includes the code or binary that you want to run. Therefore, every time you modify your code, you need to build a new Docker image, push it to your image registry, and update the image tag in the pipeline spec. This section describes one way of building Docker images, but if you have your own routine, feel free to apply it.

To build a Docker image, complete the following steps:

1)If you do not have a registry, create one with a preferred provider. I have decided to use DockerHub.

2)Create a Dockerfile for your project.

3)Build a new image from the Dockerfile by specifying a tag:

```shell
docker build -t <image>:<tag> .
```

## Step 3: Push Your Docker Image to a Registry
After building your image, you need to upload the image into a public or private image registry, such as DockerHub.

Alternatively, you can use the Pachyderm's built-in functionality to tag, build, and push images by running the pachctl update pipeline command with the --build flag. 

Since, we are using DockerHub, run:

```shell
docker login --username=<dockerhub-username> --password=<dockerhub-password> <dockerhub-fqdn>
```
Push your image to your image registry.

If you use DockerHub, run:

```shell
docker push <image>:tag
```

## Step 4: Create/Edit the Pipeline Config
Pachyderm's pipeline specifications store the configuration information about the Docker image and code that Pachyderm should run. Pipeline specifications are stored in JSON or YAML format.

A standard pipeline specification must include the following parameters:

```shell
name
transform
parallelism
input
Note
```

Some special types of pipelines, such as a spout pipeline, do not require you to specify all of these parameters.

You can store your pipeline locally or in a remote location, such as a GitHub repository.

To create a Pipeline, complete the following steps:

Create a pipeline specification. Here is an example of a pipeline spec:

```json
{
  "pipeline": {
    "name": "my-pipeline"
  },
  "transform": {
    "image": "<image>:<tag>",
    "cmd": ["/binary", "/pfs/data", "/pfs/out"]
  },
  "input": {
      "pfs": {
        "repo": "data",
        "glob": "/*"
      }
  }
}
```


## Step 5: Deploy/Update the Pipeline
As soon as you create a pipeline, Pachyderm immediately spins up one or more Kubernetes pods in which the pipeline code runs. By default, after the pipeline finishes running, the pods continue to run while waiting for the new data to be committed into the Pachyderm input repository. You can configure this parameter, as well as many others, in the pipeline specification.

Create a Pachyderm pipeline from the spec:

```shell
pachctl create pipeline -f my-pipeline.json
```
You can specify a local file or a file stored in a remote location, such as a GitHub repository. For example, https://raw.githubusercontent.com/pachyderm/pachyderm/1.13.x/examples/opencv/edges.json. 
1. If your pipeline specification changes, you can update the pipeline by running

```shell
pachctl create pipeline -f my-pipeline.json
```


## 9.Create and Manage Secrets in Pachyderm

The next step is to create secrets in pachyderm. Pachyderm uses Kubernetes' Secrets to store and manage sensitive data, such as passwords, OAuth tokens, or ssh keys. You can use any of Kubernetes' types of Secrets that match your use case. Namely, generic (or Opaque), tls, or docker-registry.

As of today, Pachyderm only supports the JSON format for Kubernetes' Secrets files.
To use a Secret in Pachyderm, you need to:

Create it.

Reference it in your pipeline's specification file.

## Create a Secret

The creation of a Secret in Pachyderm requires a JSON configuration file.

A good way to create this file is:

1. To generate it by calling a dry-run of the kubectl create secret ... --dry-run=client --output=json > myfirstsecret.json command.
2. Then call pachctl create secret -f myfirstsecret.json.

## Reference a Secret in Pachyderm's specification file¶

Now that your secret is created on Pachyderm cluster, you will need to notify your pipeline by updating your pipeline specification file. In Pachyderm, a Secret can be used in many different ways. Below, I'm describing the way we used :

As a container environment variable:

In this case, in Pachyderm's pipeline specification file, you need to reference Kubernetes' Secret by its name
and specify an environment variable named env_var that the value of your key should be bound to.
This makes for easy access to your Secret's data in your pipeline's code. For example, this is useful for passing the password to a third-party system to your pipeline's code.

```json
   "description":"A pipeline that pushes answers to the database",
   "transform":{
      "cmd":[
         "python",
         "/app/answer_insert.py"
      ],
      "image":"xyz",
      "secrets":[
         {
            "name":"dbaccess",
            "env_var":"PG_HOST",
            "key":"host"
         },
```
## 10.Running the commands in pachctl

After connecting the to the pachctl and creating the repo we can list it using ``pchctl list repo``

```shell
mj48345@docker-ubuntu-1-vm:~$ pachctl list repo
NAME                 CREATED     SIZE (MASTER) ACCESS LEVEL
push-answers-to-sql  3 hours ago 3.235KiB      OWNER        Output repo for pipeline push-answers-to-sql.
question_answer      3 hours ago 3.235KiB      OWNER        Output repo for pipeline question_answer.
question_answer_tick 3 hours ago 0B            OWNER        Cron tick repo for pipeline question_answer.
```

After that we can create the pipeline using the  ``pachctl create pipeline -f pipeline1_spec.json`` command

We can view the pipeline created using the following command: ``pachctl list pipeline -f pipeline1_spec.json``
```shell
pachctl list pipeline
mj48345@docker-ubuntu-1-vm:~$ pachctl list pipeline
NAME                VERSION INPUT             CREATED        STATE / LAST JOB   DESCRIPTION
push-answers-to-postgre 1       question_answer:/ 6 seconds ago  running / starting A pipeline that pushes answers to the database
question_answer     1       tick:@every 300s  11 seconds ago running / starting A pipeline that dowloads files from GCS and answers questions.
```

We can then view the logs using the following command: ``pachctl logs -v -j``


```shell    
mj48345@docker-ubuntu-1-vm:~$ pachctl logs -v -j 5d2c5ec96cf34c8fa02f47b84516e23e
[0000]  INFO parsed scheme: "dns" source=etcd/grpc
[0000]  INFO ccResolverWrapper: sending update to cc: {​​​​​​​[{​​​​​​​35.185.199.42:31400  <nil> 0 <nil>}​​​​​​​] <nil> <nil>}​​​​​​​ source=etcd/grpc
[0000]  INFO ClientConn switching balancer to "pick_first" source=etcd/grpc
2021-06-13 02:20:06.993141: I tensorflow/core/platform/cpu_feature_guard.cc:142] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
2021-06-13 02:20:07.029095: W tensorflow/python/util/util.cc:348] Sets are not currently considered sequences, but this may change in the future, so consider avoiding using them.
2021-06-13 02:20:07.100361: W tensorflow/core/framework/cpu_allocator_impl.cc:80] Allocation of 93763584 exceeds 10% of free system memory.
2021-06-13 02:20:07.349987: W tensorflow/core/framework/cpu_allocator_impl.cc:80] Allocation of 93763584 exceeds 10% of free system memory.
2021-06-13 02:20:07.382704: W tensorflow/core/framework/cpu_allocator_impl.cc:80] Allocation of 93763584 exceeds 10% of free system memory.
2021-06-13 02:20:08.516193: W tensorflow/core/framework/cpu_allocator_impl.cc:80] Allocation of 93763584 exceeds 10% of free system memory.
2021-06-13 02:20:08.618882: W tensorflow/core/framework/cpu_allocator_impl.cc:80] Allocation of 93763584 exceeds 10% of free system memory.
All model checkpoint layers were used when initializing TFDistilBertForQuestionAnswering.


All the layers of TFDistilBertForQuestionAnswering were initialized from the model checkpoint at distilbert-base-uncased-distilled-squad.
If your task is similar to the task the model of the checkpoint was trained on, you can already use TFDistilBertForQuestionAnswering for predictions without further training.
Downloading Files
Inside the File List Loop
Blob Created
File Downloaded as string
Calling the Question Answer Function
file_read
file_aded to intmd
writing file to location
question_answer completed
calling delete function
delete completed
```


 



