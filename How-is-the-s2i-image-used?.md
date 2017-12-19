- Assume a plugin (Eg. https://github.com/jenkinsci/kubernetes-plugin) gets
  updated and a new Jenkins release is required.

- The changes can be submitted as a PR to this repo or the [base image][base]

- NOTE: Anything that is a PR is a CI pipeline, and a  build on master is CD.

- The [CI Job][Jenkinsfile] of s2i-config kicks in for the PR.

- Builds a snapshot docker image `fabric8/jenkins-openshift:SNAPSHOT-$PR_NUMBER-$BUILD_NUMBER` with s2i, from the base
  image `fabric8/jenkins-openshift-base` and pushes to dockerhub using a tag mapping to the PR and CI build number.

- Notifies with a comment on the PR so that this image can be manually verified
  elsewhere.

- CI pipeline deploys the snapshot image to the Tiger test cluster and runs the automated BDD tests https://github.com/fabric8-jenkins/godog-jenkins updating the PR status with CI success / fail result.

- Accepting the PR is a manual process.

- The CD job from the same Jenkins file kicks in once the PR is merged to
  master.

- Same build process, but we build an image tagged latest.
  `fabric8/jenkins-openshift:latest` and a released version e.g. `fabric8/jenkins-openshift:v1.2.3`

- The build sends a PR to
  [fabric8-services/fabric8-tenant-jenkins](https://github.com/fabric8-services/fabric8-tenant-jenkins)
  and updates the jenkins version.

  NOTE: https://github.com/fabric8-services/fabric8-tenant-jenkins is a bit
  complex. I was expecting to see a single SHA1 hash there but there is a whole
  lot more there. Looks like the yaml artifacts from the fabric8 maven plugin is
  tracked as well. Not sure.

- Same drill all over again. The [CI Job](https://github.com/fabric8-services/fabric8-tenant-jenkins/blob/master/Jenkinsfile)
  of fabric8-services/fabric8-tenant-jenkins kicks in and deploys a SNAPSHOT tenant yaml to https://nexus.cd.test.fabric8.io/ which is then used in a link on a PR comment to customise a users OSIO tenant.

- Adds a PR with an comment to update the tenant Jenkins. An OSIO user can go to
  the link and update his tenant Jenkins image. So cool!

- Manual merge required

- Updates the Jenkins version in [Fabric8 jenkins
  platform][f8-jenkins-platform].

  NOTE: This is not used by OSIO. It is part of the upstream
  community.
  [Code](https://github.com/fabric8-services/fabric8-tenant-jenkins/blob/master/release.groovy#L35)

- Send a PR to [Update fabric8 tenant jenkins](https://github.com/fabric8-services/fabric8-tenant-jenkins/blob/master/release.groovy#L46),
  such that the Jenkins image gets updated for all users.

- The processes after this point is handled by the service delivery team.

[base]: https://github.com/fabric8-jenkins/jenkins-openshift-base
[f8-jenkins-platform]: https://github.com/fabric8-jenkins/fabric8-jenkins-platform
[Jenkinsfile]: https://github.com/fabric8io/openshift-jenkins-s2i-config/blob/master/Jenkinsfile
