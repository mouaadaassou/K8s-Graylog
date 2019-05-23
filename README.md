# Graylog with Kubernetes and GKE
<br/><br/>

## Project Goal :

This project aims to demonstrate how to deploy The Graylog Stack - Graylog v3, Elasticsearch v6 along with MongoDB v3 - using Kubernetes, 
and how to collect data from different data sources using inputs, and use the stream & outputs.

## What is Graylog :
Graylog is a leading centralized log management solution built to open standards for capturing, storing, and enabling real-time analysis
of terabytes of machine.

Graylog is very flexible in such a way that support multiple inputs (data sources : by this this i mean how Graylog nodes will accept data) 
we can mention 
- GELF TCP.
- GELF Kafka. 
- AWS Logs. 

as well as Output (how can Graylog nodes forward messages) - we can mention :  
- GELF Output.
- STDOUT.


You can route incoming messages into streams by applying rules against them. Messages matching the rules of a stream are routed into it.
A message can also be routed into multiple streams.

## Scenario
In this article, we gonna create a kubernetes cron job which will be used as data source for Graylog, this data source
will send message to the Graylog pod every 2 seconds. then we gonna create a stream to hold these message.

the advantage of this approach is that you can collect data from multiple data source and each one get its stream - each message stream -
so for example a stream for data that comes from AWS EC2 instances, or you Application - which we will discover in a another article.

#### Pre-requisites:
* GKE clsuter, Google gives you an account with 300$ for free.
* or use can use minikube, for that you have to adjust the elasticsearch manifest, so that you shouldn't use
affinity so that the elasticsearch service can start.
<br/>
<br/>

## setting up the  project on your cluster
#### cloning the project
you can use the my project from github [Repository](https://github.com/mouaadaassou/K8s-Graylog) :
```
    git clone https://github.com/mouaadaassou/K8s-Graylog.git
```

#### Explaining the Graylog Stack Deployments:

First things first we need to explain why Graylog uses both Elasticsearch and MongoDB. 
* Graylog uses MongoDB to store your configuration data, not your log data. Only metadata is stored, such as user information or stream configuration

* Graylog uses Elasticsearch to store the logged data, as we know Elasticsearch is powerful search engine . it is recommend to use a dedicated Elasticsearch cluster for your Graylog setup. 


#### Explaining the Corn Job
to simulate a data source that sends some data to be logged to Graylog, we created a Kubernetes Core Job
that will be running every 2 seconds. and it uses curl to post the message to Graylog.
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: curl-cron-job
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: curl-job
            image: alpine:3.9.4
            args:
            - /bin/sh
            - -c
            - apk add curl -y; while true; do curl -XPOST http://graylog3:12201/gelf -p0 -d '{"short_message":"Hello there", "host":"alpine-k8s.org", "facility":"test", "_foo":"bar"}';sleep 1s; done
          restartPolicy: OnFailure
```

#### Configuring Graylog deployment
you have to customize the GRAYLOG_HTTP_EXTERNAL_URI value so that it points to your local or remote host

```
  - name: GRAYLOG_HTTP_EXTERNAL_URI
    value: #your_remote_or_localhost_ip
```

you can also change the default password to login to the Graylog web interface, to do so you have to run the following 
command into your terminal

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1

```

this command will ask you to enter your password, then copy past the generated hashed password to the environment variable :

```
  - name: GRAYLOG_ROOT_PASSWORD_SHA2
    value: generated_hashed_password_here
```

you can check the Graylog config file [graylog.conf](https://github.com/Graylog2/graylog2-server/blob/master/misc/graylog.conf) for more details.

#### Deploying th Graylog stack :
in order to deploy the stack, you create a kubernetes deployment as follow :
```
kubectl create -f es-deploy.yaml
kubectl create -f mongo-deploy.yaml
kubectl create -f graylog-deploy.yaml
``` 
 
 you can check the deployment using the following command :
```
 kubectl get deploy
```
 
 also you can check the pods created by this deployments :
```
 kubectl get pods
```

#### Creating the Corn Job
you can create the Kubernetes corn job using the following command
```
kubectl create -f cornJob.yaml
``` 

#### Creating a Graylog Input - GELF HTTP input
after creating the necessary deployments you have to logging to Graylog web interface, after that go to 
System -> Input

  
## Q/A
### Why Elasticsearch version 6 and not version 7 ?
Graylog v3 support only Elasticsearch version 6 - as a majot version. so you cannot use Elasticsearch 7 - at least at the time of writing, 
but you can check the Grayog Documentation for further information.

### how to generate an admin password:
you can generate a hashed password using the following command, but first you have to install sha256sum : 

```
echo -n "Enter password" && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

