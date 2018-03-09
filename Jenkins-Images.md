_This document is incomplete and there is an open issue to fix it. See https://github.com/fabric8io/fabric8-build-team/issues/25_

There are 3 kinds of images;

1. Tenant Jenkins Master image
1. The master image on [jenkins.cd.test.fabric8io](https://jenkins.cd.test.fabric8.io/)(Our CI/CD Server)
1. Slave nodes spawned on Tenant and test.cd.fabric8.io

TODO: Document how each one is built and deployed. Point to the source as much as we can.

## 1. Tenant Jenkins Master image

#### 1. The Base image jenkins-openshift-base
* Source code repository - https://github.com/fabric8-jenkins/jenkins-openshift-base, forked from [openshift/jenkins][1] This repository contains Dockerfiles for Jenkins Docker images intended for
 use with OpenShift v3
* Docker hub - https://hub.docker.com/r/fabric8/jenkins-openshift-base/tags/, you could relate master commit SHA as the tags.
#### 2. The Jenkins image used by OSIO openshift-jenkins-s2i-config 
* Source code repository - https://github.com/fabric8io/openshift-jenkins-s2i-config, is uses the above [base image][2] on top of it using S2i strategy builds the OSIO specific image repository 
* Docker hub - https://hub.docker.com/r/fabric8/jenkins-openshift/tags/, you could relate master commit SHA as the tags 
#### 3. Tenant Jenkins fabric8-tenant-jenkins 
* Source code repository - https://github.com/fabric8-services/fabric8-tenant-jenkins/,this is code for user's tenant jenkins on OSIO. which contains Jenkins and content repository apps. The configs related to both of them lies in this repository. For Jenkins specifically it is here https://github.com/fabric8-services/fabric8-tenant-jenkins/tree/master/apps/jenkins/src/main/fabric8.
#### 4. Users Tenant fabric8-tenant
 * Source code repository - https://github.com/fabric8-services/fabric8-tenant/ , The one file which we should be interested in https://github.com/fabric8-services/fabric8-tenant/blob/master/JENKINS_VERSION, this shows Jenkins version deployed on Production.

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

Using a `container {}` block in a Jenkinsfile delegates the tasks to several
other containers from the Master images. For example, [here][container block
example] using the `container('docker')` uses a separate image. The names and
images are correlated with the [podTemplate][podTemplate] function from the
[kubernetes-plugin][kubernetes-plugin]. For example, we accociate the name
`maven` with the image `fabric8/maven-builder:vd81dedb`
[here][containerTemplate]

The most common ones we use are,

| Container     | Source                                          | Tools installed             | Notes |
|---------------|-------------------------------------------------|-----------------------------|-------|
| Maven builder | [fabric8io-images/maven-builder][maven-builder] | Maven 3.5.2, Open JDK 1.8.0 |       |
|               |                                                 |                             |       |
|               |                                                 |                             |       |

[1]: https://github.com/openshift/jenkins
[2 Jenkinsfile]: https://github.com/fabric8-jenkins/jenkins-openshift-base/blob/master/Jenkinsfile
[2]: https://github.com/fabric8-jenkins/jenkins-openshift-base
[3]: https://github.com/bparees/openshift-jenkins-s2i-config
[4]: https://github.com/fabric8io/openshift-jenkins-s2i-config
[container block example]: https://github.com/jaseemabid/maven-builder/blob/079478cde9e859455aba1574cb71aeb4889201ba/Jenkinsfile#L31
[containerTemplate]: https://github.com/fabric8io/fabric8-pipeline-library/blob/master/vars/mavenTemplate.groovy#L36
[kubernetes-plugin]: https://github.com/jenkinsci/kubernetes-plugin
[maven-builder]: https://github.com/fabric8io-images/maven-builder
[podTemplate]: https://github.com/jenkinsci/kubernetes-plugin/blob/master/src/main/java/org/csanchez/jenkins/plugins/kubernetes/PodTemplate.java
