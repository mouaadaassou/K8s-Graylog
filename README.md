# K8s-Graylog

## Project Goal :

This project aims to demonstrate how to deploy The Graylog Stack - Graylog v3, Elasticsearch v6 along with MongoDB v3 - with Kubernetes.

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
A message can also be routed into multiple streams

## How to use this project
you can easily use this project just by cloning it, and customize the manifest files for K8s.
but it is mandatory to set the GRAYLOG_HTTP_EXTERNAL_URI variable to your ip host

```
  - name: GRAYLOG_HTTP_EXTERNAL_URI
    value: #your_remote_or_localhost_ip
```


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
