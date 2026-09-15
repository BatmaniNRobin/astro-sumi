---
title: "Gitlab Journey to Kubernetes"
description: "Converting omnibus Gitlab installation to Kubernetes"
pubDate: 2022-12-27
updatedDate: 2022-12-27 # optional
tags: ["gitlab", "kubernetes", "omnibus", "tech", "minio", "helm"]
draft: false # drafts are visible in dev, dropped from builds
---

# Getting Started

Hey all, If you’ve landed here, then you and your team are discussing the benefits of migrating from an on-premise package-based Linux installation of GitLab to managing GitLab on Kubernetes. This post is about the ideation and reasons for doing so. Below we have two videos, the first, is by a previous manager discussing the pros and cons of each deployment, followed by a presentation given by a colleague and myself discussing the how-to and the many trials and tribulations throughout the process.

### Advantages & Disadvantages

https://youtu.be/kdTFAAAot7A

A brief synopsis of what Frank discussed is that using GitLab’s ‘omnibus’ installation that gets deployed to a Linux server allows a quick and dirty way to get up and running for small organizations to achieve a production experience. However, running GitLab in containers, namely on Kubernetes or in the cloud, brings both high availability, and scalability allowing the deployment to be a lot more flexible, easier to upgrade and may even prove to be easier to manage.

### Process Overview

https://youtu.be/7Pm8kUGsIBo

A synopsis for this is a deep dive into the actual process, the trials and tribulations faced, as well as the solutions found to these problems in particular. Hopefully, by illuminating the pitfalls we faced, we can help avoid completely or smooth out your transition from Omnibus to Kubernetes or cloud and motivate you to take the plunge as we did and reap all of the rewards! 💰

### Follow-Up

For a follow-up to this article, I have done a walkthrough of setting up this environment which can be found at Gitlab Omnibus to K8s Migration. There will be some differences between our environments based on company infrastructure as well as different configuration items but those will be outlined in the article. 

If you enjoyed this, I hope to see you in the walkthrough and wish you the best of luck tackling this! and in the meantime while it gets discussed and you’re feeling generous, feel free to buy me an ice cream. 😄

Mohammad Malik is a SWE learning languages, piano, guitar, climbing, and traveling