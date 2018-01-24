# Github OAuth based authentication and Authorisation

[Jenkins CD][jenkins-cd] uses the [github oauth plugin][gh-oauth] to authenticate and authorise users as mentioned in [task #14 of fabric8-build][issue-14]. The plugin is configured to use  Matrix Authorisation strategy which provides a means to configure ACL.

The current settings
- allows anonymous (unauthenticated) read only access to all jobs
- allows all members of [`fabric8io`][fabric8io] org to start any job
- allows members of [`ci-ops`][ci-ops] team to terminate any job
- allows members of [`ci-admins`][ci-admins] team to administrate Jenkins 

[gh-oauth]: https://wiki.jenkins.io/display/JENKINS/GitHub+OAuth+Plugin
[jenkins-cd]: https://jenkins.cd.test.fabric8.io
[fabric8io]: https://github.com/orgs/fabric8io/people
[ci-ops]: https://github.com/orgs/fabric8io/teams/ci-ops/members
[ci-admins]: https://github.com/orgs/fabric8io/teams/ci-admins/members
[issue-14]: https://github.com/fabric8io/fabric8-build-team/issues/14
