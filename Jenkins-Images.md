_This document is incomplete and there is an open issue to fix it. See https://github.com/fabric8io/fabric8-build-team/issues/25_

There are 3 kinds of images;

1. Tenant images
1. The master image on test.cd.fabric8.io
1. Slave nodes spawned on test.cd.fabric8.io

TODO: Document how each one is built and deployed. Point to the source as much as we can.

## 1. Master image deployed by fabric8-tenant-jenkins

### Repositories and Fork history

#### 1. Canonical upstream is [openshift/jenkins][1]

> This repository contains Dockerfiles for Jenkins Docker images intended for
> use with OpenShift v3

#### 2. [fabric8-jenkins/jenkins-openshift-base][2] forked from [openshift/jenkins][1]

The build team added the [Jenkinsfile][2 Jenkinsfile] to the image which mainly

  1. Adds CI/CD to the images on test.cd.fabric8.io infrastructure
  2. Builds the image using s2i strategy using image fabric8/s2i-builder:0.0.3
  3. Tags and pushes image `fabric8/jenkins-openshift-base`
  4. if master; tags and pushes `fabric8/jenkins-slave-base-centos7` as well.
  5. Sends a PR to [fabric8io/openshift-jenkins-s2i-config][4] to use the new
     image. This is called `updateDownstreamRepos`

NOTE: As of writing this, this fork is 30 commits ahead, 207 commits behind
openshift:master. I guess we need to rebase our changes on top of the current
upstream.

#### 3. [fabric8io/openshift-jenkins-s2i-config][4] forked from [bparees/openshift-jenkins-s2i-config][3]

> The Jenkins image for OISO Tenants and https://jenkins.cd.test.fabric8.io.

The upstream seems to have stopped development so the fork is justified.

# Slave images 

Using a `container {}` block in a Jenkinsfile delegates the tasks to several other containers from the Master images. TODO: Link some examples. 

The most common ones we use are, 

|Container| Source |Tools installed | Notes |
|---------|--------|----------------|-------|
| Maven builder | | | | 





[1]: https://github.com/openshift/jenkins
[2]: https://github.com/fabric8-jenkins/jenkins-openshift-base
[2 Jenkinsfile]: https://github.com/fabric8-jenkins/jenkins-openshift-base/blob/master/Jenkinsfile
[3]: https://github.com/bparees/openshift-jenkins-s2i-config
[4]: https://github.com/fabric8io/openshift-jenkins-s2i-config
