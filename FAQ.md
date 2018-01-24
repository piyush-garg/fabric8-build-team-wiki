# I get the error Jenkins doesn’t have label 'whatever'

To the best of our knowledge, this is an issue with the
[kubernetes-plugin][k8s-plugin] we are using to spawn pods on demand. Slaves are
not reconnecting back to the master after a reboot and [we are investigating
this issue][issue-label]. If you have any info to provide, please comment there.

You can just restart the build and _hope to_ have the thing magically work. It
usually works during the second attempt.

If you don't see a restart option, try logging into Jenkins with your github
account. If that still doesn't work, contact someone from the build team. We are
working on the [authentication and authorization][issue-auth] spec right now.

[k8s-plugin]: https://github.com/jenkinsci/kubernetes-plugin
[issue-label]: https://github.com/fabric8io/fabric8-build-team/issues/17
[issue-auth]: https://github.com/fabric8io/fabric8-build-team/issues/14
