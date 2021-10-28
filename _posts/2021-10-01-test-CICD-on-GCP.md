---
title: How to test CI/CD on GCP image
date: 2021-10-01 14:10:00 +0800
categories: [DevOps, CI/CD]
tags: [ci/cd]     # TAG names should always be lowercase
---

## Steps

1. Run image `google/cloud-sdk`. In order to run docker command inside the container, we bind the host's Docker socket instead of using dind.

```bash
$ docker run -v /var/run/docker.sock:/var/run/docker.sock -it google/cloud-sdk
```

2. Then, you may run your CI/CD script in the container.

- Authorizing access to GCP

```bash
$ echo "$GCLOUD_SERVICE_KEY" > ~/gcloud-service-key.json
$ gcloud auth activate-service-account --key-file ~/gcloud-service-key.json
$ gcloud auth configure-docker -q
```
