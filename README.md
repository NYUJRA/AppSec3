# Homework 3: Deployment Gone Wrong

## Introduction

You work for a company which develops a credit card processing application.
Now that the web application is fixed and ready, your company wants it
deployed in a scalable, reliable, and secure manner. To do this, your
company hired Shoddycorp's Cut-Rate Contracting to containerize your
application, then deploy it in a way that ensures availability and security.
What they delivered, falls quite short of the mark.

What Shoddycorp's Cut-Rate Contracting provided was a deployment
that *almost* works. They containerized the application, the database, and an
Nginx reverse proxy all in Docker. They then created Kubernetes yaml files to
run these containers in a Kubernetes cluster, and configured them to talk to
each other as needed. 

However, upon further inspection we can see that they didn't quite do things
right. They attempted to do Django migrations and database seeding using methods
that don't really work, they only create one replica of each pod, and there are
passwords floating around all over the place. In addition, your company must
comply with various cybersecurity standards and frameworks and must attest that
the application is secure against a set of security benchmarks. It seems that 
the contractor may have failed to meet all the regulatory requirements. All-in-all,
it's a mess.

It looks like the job to fix this falls to you. Luckily Adrian Abdala (AA) 
has read through the files already and pointed out some of the things that
are going wrong, and provided a list of things for you to fix. Before you can
work on that, though, let's get your environment set up.

Just a disclaimer, in case it needs to be said again: 
Like with all Shoddycorp's Cut-Rate Contracting deliverables, this is not code
you would like to mimic in any way.

## Frequently Asked Questions

Kubernetes is a fairly complicated beast. To help you get oriented, we've created a [Frequently Asked Questions](FAQ.md) document that should help with common questions. As, always, please make use of office hours and ask questions by email when you run into trouble!

## Part 0: Setting up Your Environment
### 1) Synchronize Your Repository and Acquire the Lab Material
---
Log into GitHub within any web browser and create an empty, **private** repository named ``<NetID>-appsec3``.
```
cd ~
git clone https://github.com/NYUJRA/AppSec3.git AppSec3
cd AppSec3
git remote remove origin
git init
git remote add origin https://<YourGitHubHandle>:<YourPersonalAccessToken>@github.com/<YourGitHubHandle>/<NetID>-appsec3.git
git push -u origin main
```


You should now have a local working directory in ``~/AppSec3`` that is configured to use your remote GitHub repository at ``https://github.com/<YourGitHubHandle>/<NetID>-appsec3`` as a version control system.
```
```

This assignment requires Docker, minikube, and kubectl. These are all installed on your
NYU-AppSec VM for the class. There is an install script included in this repository for
reference only. If you decide to perform this assignment outside your VM, you will be
responsible for troubleshooting any issues yourself. It should be stated that kubernetes can be confusing, so it is critical that students take the time with the commands and read the kubernetes documentation in order to troubleshoot issues.
```
bash nyu-appsec-a3-ubuntu20043lts-setup.sh
```
To build Docker images from the Dockerfiles, run the following commands. 
```
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```
If the builds complete without errors, you should be able to check the images in your local Docker image repository with the command
```
docker images
```
Once you have these images in your Docker image repository, you can deploy a kubernetes configuration that will use these images.

```

kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
```
To check the status of your kubernetes deployment run the following commands. 

```

kubectl get pods
kubectl get service

```

When you are ready to begin the project, please create a repository 
on GitHub for your third assignment. Like before, be sure to make 
the repository **private**.

### Part 0.1: Rundown of Files

This repository has a lot of files. The following are files you will likely be
modifying throughout this assignment.

* GiftcardSite/GiftcardSite/settings.py
* GiftcardSite/LegacySite/views.py
* GiftcardSite/k8/django-deploy.yaml
* db/Dockerfile
* db/setup.sql
* db/k8/db-deployment.yaml

In addition, you will likely need to make new files to work with Prometheus, as
described in Part 3.

### Part 0.2: Getting it to Work 

Once you have installed the necessary software, you are ready to run the whole thing
using minikube. First, start minikube.

```
minikube start
```

You will also need to set things up so that docker will use minikube, by running:

```
eval $(minikube docker-env)
```

Next,  we need to build the Dockerfiles Kubernetes will use to create the
cluster. This can be done using the following lines, assuming you are in the
root directory of the repository.

```
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```

Then use kubectl to create the pods and services needed for our project. Again,
these commands assume you are in the root directory of the repository.

```
kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
```
Verify that the pods and services were created correctly.

```
kubectl get pods
kubectl get service
```

There should be three pod entries:

* One that starts with assignment3-django-deploy
* One that starts with mysql-container
* One that starts with proxy

They should each have status RUNNING after approximately a minute.

There should also be four service entries:

* One called kubernetes
* One called assignment3-django-service
* One called mysql-service
* One called proxy-service

To see if you can connect to the site, run the following command:

```
minikube service proxy-service
```

This should open your browser to the deployed site. You should be able to view
the first page of the site, and navigate around. If this worked, you are ready
to move on to the next part.

## Part 1: Securing Secrets.

Unfortunately there are many values that are supposed to be secret floating
around in the source code and in the yaml files. Typically we do not want this.
Secret values should be protected so that we can move the source code to GitHub
and put the docker images on Dockerhub and not compromise any secrets. In
addition to keeping secrets secret, this method also allows for changing secrets
more easily.

For this part, your job will be to find some the places in which secrets are
used and replace them with a more secure way of doing secrets. Specifically, you
should look into Kubernetes secrets, how they work, and how they can be used
with both kubernetes yaml files and how they may be accessed via Python (hint:
they end up as environment variables).

For this portion of the assignment, you should submit:

1. All kubernetes yaml files modified to use secrets
2. All changes necessary to the Web application (limited to views.py and
   settings.py as mentioned above) needed to use the passed secrets.
3. A file, called secrets.txt, which demonstrates how you added the secrets.
   This must include all commands used, etc.

Finally, rebuild your Docker container for the Django application, and then
update your pods using the kubectl apply commands specified earlier.

When you are finished with this section, please mark your part 1 
submission by tagging the desired commit with the tag "part_1_complete"


* Each running service gets a DNS name that corresponds to the service name. So to refer to the proxy running on port 8080, you would use `proxy-service:8080`.

## Grading

Total points: 100

Part 1 is worth 40 points:

* 20 points for the yaml files that use Kubernetes secrets.
* 10 points for the changes to the Django code.
* 10 points for the writeup.

Part 2 is worth 30 points:

* 10 points for the kubernetes jobs
* 5 points for modified and/or new Dockerfiles
* 5 points for the code to seed the database
* 10 points for the writeup.

Part 3 is worth 30 points:

* 5 points for removing dangerous monitoring
* 5 points for expanding monitoring
* 10 points for all yaml files for Prometheus
* 10 points for the writeup.

## What to Submit

On NYU Classes, submit a link to your GitHub repository. The repository
should be **private**, and you should add the instructor/TA's GitHub
account as a contributor to give them access for grading.

For this section, your instructors are:
* John Ryan Allen, GitHub ID `NYUJRA`.
* Abhijit Chitnis, GitHub ID `achitnis007`.

For this section, your TAs are:
* Jess Ayala, GitHub ID `jayala-29`.
* Geetha D, GitHub ID `dgeeth9595`.
* Harsh Patel, GitHub ID `harshsorra`.

The repository should contain:

* Part 1
  * Your yaml files using Kubernetes secrets.
  * All files you changed from the GiftcardSite/ directory.
  * A writeup called secrets.txt.
  * A commit with the above mentioned files tagged as part_1_complete.
* Part 2
  * Yaml files that create the Kubernetes jobs.
  * Modified and/or new Dockerfiles.
  * All code you wrote to seed the database.
  * A writeup called jobs.txt.
  * A commit with these files and code tagged as part_2_complete.
* Part 3
  * A modified GiftcardSite/LegacySite/views.py file.
  * Your yaml files for running Prometheus.
  * A writeup called Prometheus.txt.
  * A commit with these files and code tagged as part_3_complete.

## Concluding Remarks

With the changes you made in this assignment, your company is a lot closer to a
decent deployment solution. However, even with the changes, there are a lot of
things that are still lacking.

One of the benefits of using Kubernetes is the ability to create replicas that
are load balanced to avoid overwhelming one instance of the application. The
same can be done with other micro-services such as the database, though this
would require database syncing across the difference database instances. These
solutions do not currently exist in this version of the assignment.

For more experience working with cloud security and deployment, consider taking
this one step further and replicating these micro-services. Attempt to load
balance over many replicas, and syncing databases. Try using Prometheus to
gather more metrics from all of your different micro-services. Try adding logging
and other useful tools.

Though these attempts will not be graded, and should not be submitted as part of
the assignment, they should help you learn a lot about how using cloud
deployment helps you preserve the availability of your service (and the
micro-services that comprise it) and how good monitoring and logging can help you
spot errors in the application before they become serious issues.
