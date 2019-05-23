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

## 
## Q/A
### Why Elasticsearch version 6 and not version 7 ?
Graylog v3 support only Elasticsearch version 6 - as a majot version. so you cannot use Elasticsearch 7 - at least at the time of writing, 
but you can check the Grayog Documentation for further information.

### how to generate an admin password:
this project uses a admin - admin as credentials, but you can override the passowrd using the following command :

```
echo -n "tour_password_here" && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```
and copy the result of this command in the graylog-deploy.yaml
```
- name: GRAYLOG_ROOT_PASSWORD_SHA2
  value: #############################
```
