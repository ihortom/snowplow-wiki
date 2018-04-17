[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 1: setup a Collector](Setting-up-a-Collector) » [[GCP: Setting up the Scala Stream Collector]]

## Contents

- 1. [Introduction](#intro)
- 2. [Enable and Setup Google Cloud PubSub](#pubsub)
- 3. [Setting up the Scala Stream Collector](#ssc)
- 4. [Running the Collector](#running)
    * 4a. [locally (useful for testing)](#running-locally)
    * 4b. [on a GCP instance](#running-instance)
    * 4c. [on a load balanced auto-scaling GCP cluster](#running-cluster)

<a name="intro">

### 1. Introduction

The Scala Stream Collector can publish raw events to Google Cloud PubSub as its sink. PubSub is a
distributed Message Queue, implemented on Google's infrastructure. Publisher applications publish to
topics, whilst subscriber applications listen to subscriptions, which can be set as pull or push,
and with different acknowledgment policies. For more on PubSub go to:
<https://cloud.google.com/pubsub/docs/concepts>.

<a name="pubsub">

### 2. Enable and Setup Google Cloud PubSub

- Go to <https://console.cloud.google.com/apis/api/pubsub.googleapis.com/overview>
    * Make sure your project is selected (on the navbar, to the left of the search bar)
    * Click Enable

![gcloud-enable-pubsub](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-enable-pubsub.png)

- You'll then have to create the topics to which the Scala Stream Collector publishes:
  * Click on the hamburger, on the top left corner
  * Scroll down until you find it, under "Big Data"
  ![gcloud-pubsub-sidebar](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-pubsub-sidebar.png)
  * Create two topics: these will be the good and bad raw topics.
  ![gcloud-pubsub-sidebar](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-pubsub-topics.png)

<a name="ssc">

### 3. Configuring the Scala Stream Collector

To set up the Scala Stream Collector, fill the apropriate fields of the configuration file.
You can find an example in the repository:
[config.hocon.sample](https://github.com/snowplow/snowplow/blob/master/2-collectors/scala-stream-collector/examples/config.hocon.sample).

<a name="running" >

### 4. Running the Collector

- Download the Scala Stream Collector from [Bintray](https://bintray.com/snowplow/snowplow-generic/snowplow-scala-stream-collector/).
- To run the collector, you'll need a config file as the one above.
- You'll also want to authenticate the machine where the collector will run by doing:

```
$ gcloud auth login
$ gcloud auth application-default login
```

NOTE: If you're running the collector in a Compute Instance, you don't need to authenticate, you
just need to set the appropriate permissions for your service accounts (automatically authenticated
in Compute Instances), so that they're allowed to use PubSub.

<a name="running-locally" >

#### 4a. locally (useful for testing)

To run the collector locally, assuming you are authenticated as well as have the above configuration
file in place, simply run:

```
$ java -jar snowplow-stream-collector-google-pubsub-*version*.jar --config config.hocon
```

<a name="running-instance" >

#### 4b. on a GCP instance

To run the collector on a GCP instance, you'll first need to spin one up.
There are two ways to do so:

##### 4b-1. via the dashboard

- Go to the [GCP dashboard](https://console.cloud.google.com/home/dashboard), and once again, make sure your project is selected.
- Click the hamburger on the top left corner, and select Compute Engine, under Compute
- Enable billing if you haven't (if you haven't enabled billing, at this point the only option you'll see is a button to do so)

![gcloud-instance-nobilling](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-instance-nobilling.png)

- Click "Create instance" and pick the apropriate settings for your case, making sure of, at least
the following:
    * Under _Access scopes_, select "Set access for each API" and enable "Cloud PubSub"
    * Under _Firewall_, select "Allow HTTP traffic"
    * *Optional* Click _Management, disk, networking, SSH keys_
        - Under _Networking_, add a Tag, such as "collector". (This is needed to add a tagged
Firewall rule, explained below)

![gcloud-instance-create1](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-instance-create1.png)

![gcloud-instance-create2](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-instance-create2.png)

- Click the hamburger on the top left corner, and click on "VPC Network", under _Networking_
- On the sidebar, click on "Firewall rules"
- Click "Create Firewall Rule"
- Name your rule
- Under _Source filter_ pick "Allow from any source"
- Under _Protocols and ports_ add "tcp:8080"
    * Note that 8080 is the port assigned to the collector in the configuration file. If you choose
another port here, make sure you change the config file
- Under _Target tags_ add the Tag with which you labeled your instance (here `collector`)
- Click "Create"

![gcloud-firewall](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-firewall.png)

##### 4b-2. via the command line

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

$ gcloud compute --project "example-project-156611" firewall-rules create "collectors-allow-tcp8080" \
                 --allow tcp:8080 \
                 --network "default" \
                 --source-ranges "0.0.0.0/0" \
                 --target-tags "collector"
```

To place the above mentioned files in the instance (config file and the collector jar), you can:

- For the jar, you'll `wget` it from Bintray into the instance directly;
- For the config file, store it using GCP Storage and then download it into the instance. To store
the config file:
    - Click the hamburger on the top left corner and find Storage, under _Storage_
    - Create a bucket
![gcloud-storage1](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-storage1.png)
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
$ wget https://dl.bintray.com/snowplow/snowplow-generic/snowplow_scala_stream_collector_google_pubsub_<VERSION>.zip
$ gsutil cp gs://<YOUR-BUCKET-NAME/<YOUR-CONFIG-FILE-NAME> .
$ unzip snowplow_scala_stream_collector_google_pubsub_<VERSION>.zip
$ java -jar snowplow-stream-collector-google-pubsub-<VERSION>.jar --config <YOUR-CONFIG-FILE-NAME>
```

<a name="running-cluster" >

#### 4c. a load balanced auto-scaling GCP cluster

To run a load balanced auto-scaling cluster, you'll need to follow the following steps:

- Create an instance template
- Create an auto managed instance group
- Create a load balancer

##### Creating an instance template

First you'll have to store your config file in some place that your instances can download from,
like Google Cloud Storage. We suggest you store it in a GCP Storage bucket, as described above.

###### via Google Cloud Console

- Click the hamburger on the top left corner and find "Compute Engine", under _Compute_
- Go to "Instance templates" on the sidebar. Click "Create instance template"
- Choose the appropriate settings for your case. Do (at least) the following:
    - Under _Access scopes_, select "Set access for each API" and enable "Cloud PubSub"
    - Under _Firewall_, select "Allow HTTP traffic"
    - Click _Management, disk, networking, SSH keys_
![gcloud-instance-template1](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-instance-template1.png)
    - Under _Networking_, add a tag, such as "collector". (This is needed to add a Firewall rule)
    - Under _Management_ "Startup script" add the following script (changing the relevant fields for
your case):

```bash
#! /bin/bash
sudo apt-get update
sudo apt-get -y install default-jre
sudo apt-get -y install unzip
archive=snowplow_scala_stream_collector_google_pubsub_<VERSION>.zip
wget https://dl.bintray.com/snowplow/snowplow-generic/$archive
gsutil cp gs://<YOUR-BUCKET-NAME/<YOUR-CONFIG-FILE-NAME> .
unzip $archive
java -jar snowplow-stream-collector-google-pubsub-<VERSION>.jar --config <YOUR-CONFIG-FILE-NAME> &
```

- Click "Create"
- Add a Firewall rule as described above (if you haven't already)

###### via the command-line

Here's the command-line equivalent for the options selected by performing the steps above:

```
$ gcloud compute --project "example-project-156611" instance-templates create "ssc-instance-template" \
                 --machine-type "n1-standard-1" \
                 --network "default" \
                 --maintenance-policy "MIGRATE" \
                 --scopes 189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/pubsub",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/servicecontrol",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/service.management.readonly",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/logging.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/monitoring.write",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/trace.append",189687079473-compute@developer.gserviceaccount.com="https://www.googleapis.com/auth/devstorage.read_only" \
                 --tags "collector" \
                 --image "/ubuntu-os-cloud/ubuntu-1604-xenial-v20170113" \
                 --boot-disk-size "10" \
                 --boot-disk-type "pd-standard" \
                 --boot-disk-device-name "ssc-instance-template" \
                 --metadata "startup-script=<THE-STARTUP-SCRIPT-AS-DESCRIBED-ABOVE>"
```

##### Create an auto managed instance group

###### via Google Cloud Console

- On the side bar, click "Instance groups"
- Click "Create instance group"
- Fill in with the appropriate values. We named our instance group "collectors".
- Under _Instance template_ pick the instance template you created previously
- Set _Autoscaling_ to "On". By default the Autoscale is based on CPU usage and set with default
settings. We'll leave them as they are for now.
- Under _Health Check_, pick "Create health check"
    * Name your health check
    * Under _Port_ add 8080 or the port you configured above
    * Under _Request path_ add "/health"
    * Click "Save and Continue"
- Click "Create"

![gcloud-group-create1](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-group-create1.png)

![gcloud-group-create2](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-group-create2.png)

###### via command-line

Here's the command-line equivalent for te options selected by performing the steps above:

```
$ gcloud compute --project "example-project-156611" instance-groups managed create "collectors" \
                 --zone "us-central1-c" \
                 --base-instance-name "collectors" \
                 --template "ssc-instance-template" \
                 --size "1"

$ gcloud compute --project "example-project-156611" instance-groups managed set-autoscaling "enrichers" \
                 --zone "us-central1-c" \
                 --cool-down-period "60" \
                 --max-num-replicas "10" \
                 --min-num-replicas "1" \
                 --target-cpu-utilization "0.6"
```


##### Configure the load balancer

- Click the hamburger on the top left corner, and find "Network services" under _Networking_
- On the side bar, click "Load Balancing"
- Click "Create load balancer"
- Select "HTTP load balancing" and click "Start configuration"

![gcloud-load-balancer1](https://github.com/snowplow/snowplow/wiki/images/gcloud/gcloud-load-balancer1.png)

- Under _Backend configuration_:
    * Click "Create a backend service"
    * Pick an appropriate name
    * Pick the instance group we created above
    * Pick the port number we configured earlier
    * Input a maximum number of requests per second per instance
    * You can input "Maximum CPU utilization" and "Capacity" at your discretion
    * Under _Health check_ pick the health check you created previously

- Under _Host and path rules_: you can just make sure that the backend service selected is the one
we just created

- Under _Frontend configuration_:
    * Leave _IP_ as "Ephemeral" and leave _Port_ to 80

- Click "Review and finalize" to check everything is OK.

- Click "Create"

- You'll be able to check out this load balancer IP and port by clicking on it

- You can then make sure this load balancer is used by your instance group by going back to
"Instance groups"

