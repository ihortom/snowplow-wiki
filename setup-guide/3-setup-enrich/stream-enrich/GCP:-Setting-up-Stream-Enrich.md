## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud PubSub](#pubsub)
- 3. [Setting up Stream Enrich](#se)
- 4. [Running Stream Enrich](#running)
    * 4a. [locally (useful for testing)](#running-locally)
    * 4b. [on a GCP instance](#running-instance)
   
<a name="intro">

### 1. Introduction

Stream Enrich can now use Google Cloud PubSub as its source and sink. PubSub is a distributed
message queue, implemented on Google's infrastructure. Publisher applications publish to topics,
whilst subscriber applications listen to subscriptions, which can be set as pull or push, and with
different acknowledgment policies. For more on PubSub go to:
<https://cloud.google.com/pubsub/docs/concepts>

<a name="pubsub">

### 2. Enable and Setup Google Cloud PubSub

- To use PubSub, first we need to enable it. 
- Go to https://console.cloud.google.com/apis/api/pubsub.googleapis.com/overview
    * Make sure your project is selected (on the navbar, to the left of the search bar)
    * Click Enable

[[images/gcloud/gcloud-enable-pubsub.png]]

- You'll then have to create the topics to which Stream Enrich publishes and subscribes:
    * Click on the hamburger, on the top left corner
    * Scroll down until you find "PubSub", under "Big Data"

    [[images/gcloud/gcloud-pubsub-sidebar.png]]

    * Create two topics: these will be the good and bad enriched topics.

    [[images/gcloud/gcloud-pubsub-topics.png]]

<a name="se">

### 3. Configure Stream Enrich

To set up Stream Enrich, fill the apropriate fields of the configuration file
You can find and example in the repository: [config.hocon.sample](https://github.com/snowplow/snowplow/blob/master/3-enrich/stream-enrich/examples/config.hocon.sample)

You will also need an Iglu resolver, and optional enrichments. These files can be stored both on the
same machine where Stream Enrich will run or in Cloud Datastore, under any entity with a "json"
property. To add, for example, the Iglu resolver, go to
<https://console.cloud.google.com/datastore/entities/query?project=YOUR-PROJECT-ID>, click "Create
Entity", fill in its Kind (we use "resolver") and introduce its name manually, or take note of the
auto-generated one after you click "Create".

[[images/gcloud/iglu-resolver.png]]

[[images/gcloud/iglu-resolver2.png]]

Then, when running the project, pass the resolver parameter as:
`--resolver datastore:yourProjectId/resolver/resolver_name_or_id`.

You can do the same with enrichments by passing the prefix of the enrichment names:
`--enrichments datastore:yourProjectId/enrichment/enrichment-`. This supposes that you have a kind
name `enrichment` and all your enrichment entities have a name starting with `enrichment-`

<a name="running" >

### 4. Running Stream Enrich

- Download Stream Enrich from [Bintray](https://bintray.com/snowplow/snowplow-generic/snowplow-stream-enrich/).
- To run Stream Enrich , you'll need a config file as the one above.
- You'll also want to authenticate the machine where the it will run by doing:

``` 
$ gcloud auth login 
$ gcloud application-default auth login
```

NOTE: If you're running Stream Enrich on a Compute Instance, you don't need to authenticate with the
above commands, you just need to set the appropriate permissions for your service accounts
(automatically authenticated in Compute Instances), so that they're allowed to use PubSub.

<a name="running-locally" >

#### 4a. locally (useful for testing)

To run the collector locally, assuming you have the above files in place, simply run:

```
$ java -jar snowplow-stream-enrich-google-pubsub-*version*.jar --config config.hocon --resolver file:path/to/iglu_resolver.json --enrichments file:path/to/enrichments
```

<a name="running-instance" >

#### 4b. on a GCP instance

To run the collector on a GCP instance, you'll first need to spin one up.
There are two ways to do so:

##### 4b-1. via dashboard

- Go to the [GCP dashboard](https://console.cloud.google.com/home/dashboard), and once again, make
sure your project is selected.
- Click the hamburger on the top left corner, and select Compute Engine, under Compute
- Enable billing if you haven't (if you haven't enabled billing, at this point the only option
you'll see is a button to do so)

[[images/gcloud/gcloud-instance-nobilling.png]]

- Click "Create instance" and pick the apropriate settings for your case, making sure of, at least the following:
    * Under _Access scopes_, select "Set access for each API" and enable "Cloud PubSub"
    
[[images/gcloud/gcloud-instance-create1.png]]

[[images/gcloud/gcloud-instance-create2.png]]

##### 4b-2. via command line

- Make sure you have authenticated as described above
- Here's an example command of an instance spin up: (check the [gcloud reference](https://developers.google.com/cloud/sdk/gcloud/reference/compute/?hl=en_US) for more info)

```
$ gcloud compute --project "example-project-156611" instances create "instance-2" \
                 --zone "us-central1-c" \
                 --machine-type "n1-standard-1" \
                 --subnet "default" \
                 --maintenance-policy "MIGRATE" \
                 --scopes 189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/pubsub",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/servicecontrol",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/service.management.readonly",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/logging.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/monitoring.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/trace.append",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/devstorage.read_only" \
                 --tags "collector" \
                 --image "/ubuntu-os-cloud/ubuntu-1604-xenial-v20170113" \
                 --boot-disk-size "10" \
                 --boot-disk-type "pd-standard" \
                 --boot-disk-device-name "instance-2"
```

To place the above mentioned files in the instance (config file and the stream enrich JAR), you can:

- For the jar, you'll `wget` it from Bintray into the instance directly;
- For the config file, store it using GCP Storage and then download it into the instance. To store
the config file:
    - Click the hamburger on the top left corner and find Storage, under _Storage_
    - Create a bucket

    [[/images/gcloud/gcloud-storage1.png]]

    - Then click "Upload Files" and upload your configuration file


Once you have your config file in place, ssh into your instance:

```
$ gcloud compute ssh your-instance-name --zone your-instance-zone
```

And then run:

```
$ sudo apt-get update
$ sudo apt-get -y install default-jre
$ sudo apt-get -y install unzip
$ wget https://dl.bintray.com/snowplow/snowplow-generic/snowplow_stream_enrich_google_pubsub_<VERSION>.zip
$ gsutil cp gs://<YOUR-BUCKET-NAME/<YOUR-CONFIG-FILE-NAME> .
$ unzip snowplow_stream_enrich_google_pubsub_<VERSION>.zip
$ java -jar snowplow-stream-enrich-google-pusub-<VERSION>.jar --config <YOUR-CONFIG-FILE-NAME> --resolver datastore:resolver/iglu --enrichments datastore:enrichment/enrichment
```

<a name="running-cluster" >

#### 4c. a load balanced auto-scaling GCP cluster

To run an auto-scaling cluster of Stream Enrich boxes, you'll need to follow the following steps:

- Create an instance template
- Create an auto managed instance group

##### Creating an instance template

First you'll have to store your config file in some place that your instances can download from,
like Google Cloud Storage. We suggest you store it in a GCP Storage bucket, as described above.

###### via Google Cloud Console

- Click the hamburger on the top left corner and find "Compute Engine", under _Compute_
- Go to "Instance templates" on the sidebar. Click "Create instance template"
- Choose the appropriate settings for your case. Do (at least) the following:
    * Under _Access scopes_, select "Set access for each API" and enable "Cloud PubSub"
    * Click _Management, disk, networking, SSH keys_
    * Under _Management_ "Startup script" add the following script (changing the relevant fields for
your case):

```bash
#! /bin/bash
sudo apt-get update
sudo apt-get -y install default-jre
sudo apt-get -y install unzip
archive=snowplow_stream_enrich_google_pubsub_<VERSION>.zip
wget https://dl.bintray.com/snowplow/snowplow-generic/$archive
gsutil cp gs://<YOUR-BUCKET-NAME/<YOUR-CONFIG-FILE-NAME> .
unzip $archive
$ java -jar snowplow-stream-enrich-google-pusub-<VERSION>.jar --config <YOUR-CONFIG-FILE-NAME> --resolver datastore:resolver/iglu --enrichments datastore:enrichment/enrichment
```

- Click "Create"

###### via the command-line

Here's the command-line equivalent for the options selected by performing the steps above:

```
$ gcloud compute --project "example-project-156611" instance-templates create "se-instance-template" \
                 --machine-type "n1-standard-1" \
                 --network "default" \
                 --maintenance-policy "MIGRATE" \
                 --scopes 189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/pubsub",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/servicecontrol",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/service.management.readonly",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/logging.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/monitoring.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/trace.append",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/devstorage.read_only" \
                 --image "/ubuntu-os-cloud/ubuntu-1604-xenial-v20170113" \
                 --boot-disk-size "10" \
                 --boot-disk-type "pd-standard" \
                 --boot-disk-device-name "se-instance-template" \
                 --metadata "startup-script=<THE-STARTUP-SCRIPT-AS-DESCRIBED-ABOVE>"
```

##### Create an auto managed instance group

###### via Google Cloud Console

- On the side bar, click "Instance groups"
- Click "Create instance group"
- Fill in with the appropriate values. We named our instance group "enrichers".
- Under _Instance template_ pick the instance template you created previously
- Set _Autoscaling_ to "On". By default the Autoscale is based on CPU usage and set with default
settings.
- Click "Create"

###### via command-line

Here's the command-line equivalent for te options selected by performing the steps above:

```
$ gcloud compute --project "example-project-156611" instance-groups managed create "enrichers" \
                 --zone "us-central1-c" \
                 --base-instance-name "enrichers" \
                 --template "se-instance-template" \
                 --size "1"

$ gcloud compute --project "example-project-156611" instance-groups managed set-autoscaling "enrichers" \
                 --zone "us-central1-c" \
                 --cool-down-period "60" \
                 --max-num-replicas "10" \
                 --min-num-replicas "1" \
                 --target-cpu-utilization "0.6"
```



