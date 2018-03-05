Things that annoy me.

### 1. Lack of Docker build support

Just cannot move any of the OSIO components into OSIO without this. Build is
building a CI even build cannot use. This product must be dogfood if we need any
quality out of this.

### 2. Iteration times are ridiculously high 

Changes in a plugin or something goes through a series of Jenkins builds, each
one potentially taking a very long time. The repository fetish is severely
hurting development.

### 3. Simple things should be simple.

I had to use this for a while to truly appreciate the beauty of platforms like
Travis or Circle CI. We need a better ramping up story so that someone can start
using this service without learning all of Jenkins, pipeline library and all the
random plugins.

### 4. Lack of code ownership boundary

The default Jenkins file that gets imported into the project contains enough
nuances to get it wrong. Over time a user is going to change it bit by bit to
the point that when a build fails, there wont be a way to say if that is a
script failure or a system failure without looking into the logs. This is not a
sane way to build a service.

Need to be able to wrap a big try catch around the code that is provided by the
user such that

  1. The user maintains only their code, not this weird mix
  2. We have more confidence and easier debug process

### 5. Need a small footprint and API

All the API that is allowed inside the Jenkinsfile should be committed and
documented. This weird mix of fabric8 pipeline library, 130+ plugins and the
things that Jenkins itself provides make it a painful thing to maintain.

This leaves us in this silly clueless state with questions like

1. [Where does the 'container' function come from?](https://github.com/fabric8io/fabric8-build-team/issues/20) :confused:
2. [Why does this build fail?](https://github.com/fabric8io/fabric8-build-team/issues/26)
3. [Jenkins master reboot and slave reconnect issue](https://github.com/fabric8io/fabric8-build-team/issues/17)

### 6. The [fabric8io/openshift-jenkins-s2i-config](https://github.com/fabric8io/openshift-jenkins-s2i-config) is mediocre beyond words.

1. Kitchen sink, outdated fork
2. The documentation is the worst. Even basics like how to run the image is missing.
   https://github.com/fabric8io/openshift-jenkins-s2i-config/issues/142
3. The build process of a base image and a second s2i build makes the whole
   process frustratingly slow and painful. Just because redhat built s2i doesn't
   mean we should use it.

### 7. Fabric8 pipeline library.

I'm fundamentally unable to like f-p-library which is honestly a glorified
kitchen sink of a whole of crap and nothing much more. It reimplements a lot of
tooling I've learned and liked poorly in groovy. Yet another layer of
indirection, complexity, code ownership, increased build times, things to learn
for love of Jenkins?

For example see
https://github.com/fabric8io/fabric8-pipeline-library/blob/master/vars/goMake.groovy.
That's 49 lines of groovy poorly implementing docker run make. I'd rather write

```Dockerfile
FROM golang
RUN make
```
instead of all this mess.

I really really wish I could throw away a lot of this. But most stuff doesn't
make sense to me anymore and there are probably a lot of people using it
directly from master.

### 8. Automate the test.cd.fabri8.io deployment story

What we have now is the equivalent of the /var/jenkins folder backup. This
should probably be powered by something like Ansible.

### 9. Fabric8 maven plugin

Its an overkill for anything we are trying to do. You should not need a 37K LOC
Java app with 400MB+ deps to generate a bunch of yaml files.

### 10. Installer

There should be one obvious and sane way to install OSIO everywhere - local and
various prod/testing environments. It would be nice to not invent more software
here and just use Ansible.

The various repositories with tons of yaml and the intricate process to bring
them all together really needs to be reconsidered.

### 11. Same base image for everyone?

I'm not so sure about this like one.

Why should all our users run their builds on the same base jenkins image? For
example, when the UI team wanted g++ on their build server, they had no way to
do it without installing it per build making things super slow or interfering
with the build image of everyone else.

### 12. Our own CD story needs to improve 

Updating an image is such a manual process. The irony. We have the worst CI/CD
story for a CI/CD platform.

### 13. Tests 

In the meanwhile an unreliable CI/CD is worse than not having one.
