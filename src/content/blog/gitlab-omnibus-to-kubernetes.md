---
title: "Gitlab Omnibus to Kubernetes Transition"
description: "Converting omnibus Gitlab installation to Kubernetes"
pubDate: 2021-08-05
updatedDate: 2022-12-27 # optional
tags: ["gitlab", "kubernetes", "omnibus", "tech", "minio", "helm"]
draft: false # drafts are visible in dev, dropped from builds
---

# Preface

If you’ve read my Journey to k8s post, you’re probably wondering how to go about actually making the migration. It’s possible based on your current environment that things may vary however the general steps should all be the same. 

This guide is meant to serve as a work along article. Following these steps, minus configuring your own specific instance’s settings, should serve to help you stand up a working instance of your GitLab instance on Kubernetes and make a successful migration off of an on-premise Linux installation.

*NOTE:* In order to follow along with this guide, you must already have an understanding of Kubernetes, helm charts, as well as your current, in-place omnibus Gitlab installation environment. It also assumes you have a working Kubernetes cluster with persistent volumes setup, in this guide’s case using NFS.

# Getting Started

In this tutorial, we’ll be migrating from a currently existing on-premise Linux package-based GitLab installation to a helm chart deployment of GitLab on Kubernetes. As mentioned, we’ll need an already existing Kubernetes cluster with persistent volumes, with added bonuses such as, an ingress controller, Cert-Manager, and logging and metrics tools such as OpenTelemetry, Prometheus, and Grafana.

The reason for this migration is to utilize the inherent benefits of Kubernetes, allowing automatic scaling of resources as needed, as well as allowing for very easy to do rolling updates by simply upgrading the helm chart, completely minimizing any downtime and ensuring your infrastructure is as secure as possible and as up to date as possible. Not to mention all of the cool tools GitLab adds all the time 👀.

## Overview

### Prerequisites

There are a few prerequisites to the migration. 

1. Having a working and functional omnibus GitLab installation with no down services, this can be confirmed using `gitlab-ctl status`. 
2. To verify the integrity of your Git repositories prior to the migration.
3. A Helm charts based deployment on your Kubernetes cluster, running the same GitLab version as the package-based installation is required.
4. You need to set up the object storage that the Helm chart based deployment will use. For production use, we recommend you use an external object storage and have the login credentials to access it ready. If you are using the built-in MinIO service, read the docs on how to grab the login credentials from it.

### High-Level Steps

The migration can be broken down into a few high-level steps, all of which can be found here.

1. Migrate any existing files (uploads, artifacts, LFS objects) from the package-based installation to object storage
2. Create a backup tarball and exclude the already migrated uploads
3. Restore from the package-based installation to the Helm chart, starting with the secrets.
4. Restart all pods to make sure changes are applied
5. Visit the Helm-based deployment and confirm projects, groups, users, issues etc. that existed in the package-based installation are restored. Also, verify if the uploaded files (avatars, files uploaded to issues, etc.) are loaded fine.

# Walkthrough

This is a prerequisite and a necessary step in proceeding, so we’ll include this.

1. Set up some kind of external object storage, such as AWS S3, Azure’s blob storage, or GitLab’s default, MinIO, an S3 emulation. For this guide, we’ll be using MinIO. It is possible to set up MinIO within your cluster, as outlined below.
    1. download and configure chart for MinIO
    2. deploy with`kubectl apply -f minio-dev.yaml`, should create a namespace and deploy a MinIO pod
    3. ensure functionality: 
        1. **`kubectl get pods -n minio-dev`**
        2. The output should resemble the following:
            1. **`NAME    READY   STATUS    RESTARTS   AGE
            minio   1/1     Running   0          77s`**
        3. You can also use the following commands to retrieve detailed information on the pod status:
        4. **`kubectl describe pod/minio -n minio-dev
        
        kubectl logs pod/minio -n minio-dev`**
    4. Use the **`kubectl port-forward`** command to temporarily forward traffic from the MinIO pod to the local machine:
        1. **`kubectl port-forward pod/minio 9000 9090 -n minio-dev`**
    5. access your instance via browser at `localhost:9090`
        1. login with default admin credentials 
        2. ***MAKE A NOTE TO CHANGE FOR PRODUCTION*** `minioadmin:minioadmin`
        
        !Image showcasing MinIO GUI in the browser. Additionally, showing various settings that can be done from here.
        
        Image showcasing MinIO GUI in the browser. Additionally, showing various settings that can be done from here.
        
    6. Now you should be all set up, make sure to explore it a bit and setup some metrics logging as well as any IAM policies and other users as needed here via the GUI
2. Next up is downloading the GitLab helm chart via helm by navigating to GitLab
    1. add the helm repo: `helm repo add charts.gitlab.io`
        
        !YAML snippet of ingress from GitLab chart
        
        YAML snippet of ingress from GitLab chart
        
    2. configure the values.yaml to consist of your infrastructure’s settings or planned settings.
        1. The docs are very helpful here and each property explanation can be found here. Ensure that if you’re not upgrading to the newest GitLab version to change the branch accordingly. 
        2. Some common items to update are the domain name under hosts, the ingress to provide SSL termination and/or if traefik is used instead of nginx, as shown above. Furthermore, if other GitLab services such as praefect, and gitaly, or GitLab pages are needed.
        3. Make sure, if behind a company proxy that if you have a private container registry, to add the images specified in the chart to your registry and to pull those images from your private registry
        
        !YAML snippet for GitLab chart, showcasing an example MinIO object storage.
        
        YAML snippet for GitLab chart, showcasing an example MinIO object storage.
        
    3. deploy the GitLab instance to your Kubernetes cluster within the gitlab namespace
        1. `helm install gitlab ./gitlab -f values.yaml`
    4. ensure functionality: 
        1. **`kubectl get pods -n gitlab`**
        2. The output should resemble the following:
            1. **`NAME    READY   STATUS    RESTARTS   AGE
            gitlab   1/1     Running   0          77s`**
        3. You can also use the following commands to retrieve detailed information on the pod status:
        4. **`kubectl describe pod/<gitlab-pods> -n gitlab
        
        kubectl logs pod/<gitlab-pods> -n gitlab`**
3. Next will be following the steps outlined in the preface, starting with migrating files to your object storage
    1. *Note:* During this step, it’s best to have a code freeze as any changes done to repos during or after this is done will not be included as part of the migration and would require the same changes to be done again in the new instance or to have the steps proceeding to be done again.
    2. Modify `/etc/gitlab/gitlab.rb` file for currently existing omnibus install and configure object storage for: Uploads Artifacts LFS Packages This **must** be the same object storage service that the Helm charts based deployment is connected to (MinIO in our case). The snippet above is a good example of all the information needed for this step.
    3. Run reconfigure to apply the changes:`sudo gitlab-ctl reconfigure`
    4. Migrate existing artifacts to object storage:`sudo gitlab-rake gitlab:artifacts:migrate`
    5. Migrate existing LFS objects to object storage:`sudo gitlab-rake gitlab:lfs:migrate`
    6. Migrate existing Packages to object storage:`gitlab-rake gitlab:packages:migrate`
    7. Migrate existing uploads to object storage:`sudo gitlab-rake gitlab:uploads:migrate:all`
    8. See documentation. Visit the package-based GitLab instance and make sure the uploads are available. For example check if user, group and project avatars are rendered fine, if images and other files added to issues load correctly, etc. In previous testing, we needed to do this multiple times. Lots of troubleshooting was done here.
    9. Make sure to test the connection to MinIO and that the folders and repos are all set up accordingly. Some of which consists of doing some directory traversal in the configuration for proper repo structure and permissions.
4. Create a backup tarball and exclude the already migrated uploads
    1. `sudo gitlab-rake gitlab:backup:create SKIP=artifacts,lfs,packages,uploads`
        1. The backup file will be stored under `/var/opt/gitlab/backups`, unless you explicitly changed it.
5. Restore from the package-based installation to the Helm chart, starting with the secrets. You will need to migrate the values of `/etc/gitlab/gitlab-secrets.json` to the YAML file that will be used by Helm.
6. Restart all pods to make sure changes are applied:
    
    `kubectl delete pods -lrelease=<helm release name>`
    
7. Visit the Helm-based deployment and confirm projects, groups, users, issues etc. that existed in the package-based installation are restored. Also, verify if the uploaded files (avatars, files uploaded to issues, etc.) are loaded fine.
8. And there you go, your previous omnibus install on a Linux server is now deployed on Kubernetes using helm and MinIO object storage! 😄
9. Additionally, what is not addressed in this guide is setting up GitLab runners to run your GitLab jobs and pipelines. it is a separate task but can also be deployed to the Kubernetes cluster. Stay tuned for a guide on that. Heads up, it’s MUCH easier. If you already have them deployed you can just go ahead and grab their registration token, which can be found in your GitLab instance) and add that token to your helm deployment.

Remember, this is going to be a long and slow process; making sure everything transfers, that your object storage is fully functional, your new instance is up to date and functional, and ensuring you have all of the storage restraints met will take time. You will have to repeat some steps multiple times and you’ll probably run into many issues that I did not, regardless you’ll be able to do it and best of luck because a Kubernetes deployment is much easier to handle!

If you enjoyed this, please feel free to buy me an ice cream :)

Mohammad Malik is a SWE learning languages, piano, guitar, climbing, and traveling