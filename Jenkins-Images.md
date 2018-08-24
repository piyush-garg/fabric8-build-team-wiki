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
* Source code repository - https://github.com/fabric8io/openshift-jenkins-s2i-config, is uses the above [base image][2] on top of it using S2i strategy builds the OSIO specific image repository. we mention the above base image https://github.com/fabric8io/openshift-jenkins-s2i-config/blob/master/Jenkinsfile#L7 while building the s2i image.
* Docker hub - https://hub.docker.com/r/fabric8/jenkins-openshift/tags/, you could relate master commit SHA as the tags 
#### 3. Tenant Jenkins fabric8-tenant-jenkins 
* Source code repository - https://github.com/fabric8-services/fabric8-tenant-jenkins/, this is code for user's tenant jenkins on OSIO. which contains Jenkins and content repository apps. The configs related to both of them lies in this repository. For Jenkins specifically it is here https://github.com/fabric8-services/fabric8-tenant-jenkins/tree/master/apps/jenkins/src/main/fabric8.
#### 4. Users Tenant fabric8-tenant
 * Source code repository - https://github.com/fabric8-services/fabric8-tenant/ , The one file which we should be interested in https://github.com/fabric8-services/fabric8-tenant/blob/master/JENKINS_VERSION, this shows Jenkins version deployed on Production.
#### 5. Deployment and release story:
As I explained above about all the images they are related when for releasing the final tenant jenkins image.
Below is the workflow how it works:

For every repository above there is a Jenkisfile which is designed to deploy for [CI](https://github.com/fabric8io/openshift-jenkins-s2i-config/blob/master/Jenkinsfile#L15)(when a PR is raised) and [CD](https://github.com/fabric8io/openshift-jenkins-s2i-config/blob/master/Jenkinsfile#L49)(when a PR is merged to master) related changes.

`jenkins-openshift-base (base image) -> openshift-jenkins-s2i-config(Jenkins image) -> fabric8-tenant-jenkins (tenant Jenkins) ->  fabric8-tenant (user tenant )`

* when there is a PR made to base image and after merging(manual merge) to master the [CD pipline](https://jenkins.cd.test.fabric8.io/job/fabric8-jenkins/job/jenkins-openshift-base/job/master/) raises PR to openshift-jenkins-s2i-config by updating the Jenkins image, for eg: https://github.com/fabric8io/openshift-jenkins-s2i-config/pull/141. 
* At this point the [CI pipeline](https://jenkins.cd.test.fabric8.io/job/fabric8io-auto-rel/job/openshift-jenkins-s2i-config/view/change-requests/) of openshift-jenkins-s2i-config is triggered and it run some basic [godog test](https://github.com/fabric8-jenkins/godog-jenkins), you can check that in the console output and it comments on your PR `@fabric8cd snapshot Jenkins image is available. docker pull fabric8/jenkins-openshift:SNAPSHOT-PR-141-5` and also comments `Snapshot Jenkins is deployed and running HERE` the second comment is only done if the user who raised the PR is a collaborator of the fabric8io org and this deploys this image on GKE tiger cluster where you have the running Jenkins with this image. You can do some testing there check the jenkins logs here https://<server>/log/all if there are compilation failures for plugins etc. And you can also test by modifying the Jenkins tenant DC(Deployment Config) on OSIO. you can kick off a quickstart to check if everything is good after updating the tenant DC.
* You need to merge (manual merge) the PR raised to openshift-jenkins-s2i-config and then it triggers the [CD pipeline](https://jenkins.cd.test.fabric8.io/job/fabric8io-auto-rel/job/openshift-jenkins-s2i-config/job/master/) for openshift-jenkins-s2i-config which raises a PR to fabric8-tenant-jenkins for eg  like this https://github.com/fabric8-services/fabric8-tenant-jenkins/pull/66 
* You need to merge (manual merge) the PR raised to fabric8-tenant-jenkins and that triggers a [CD pipeline](https://jenkins.cd.test.fabric8.io/job/fabric8-services/job/fabric8-tenant-jenkins/job/master/) and raises a PR to fabric8-tenant for eg like https://github.com/fabric8-services/fabric8-tenant/pull/561 
* This PR is auto merged and it triggers a [CD pipeline](https://jenkins.cd.test.fabric8.io/job/fabric8-services/job/fabric8-tenant/job/master/) 
After this the build team raises an [issue](https://github.com/openshiftio/openshift.io/issues/2566) for platform team to do a cluster wide tenant update so that the newly released Jenkins Version is merged.


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
We can find the slave images here 
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
| Maven builder | [fabric8io-images/maven-builder][maven-builder] | Maven 3.3.9, Open JDK 1.8.0 | OSIO  |
|               |                                                 |                             |       |
|               |                                                 |                             |       |

Note:
Once there is a change made in the maven builder image and merged to master you need to update the [maven template](https://github.com/fabric8io/fabric8-pipeline-library/blob/master/vars/mavenTemplate.groovy#L12 
) with [image](https://hub.docker.com/r/fabric8/maven-builder/tags/) pushed by your change. 

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
