Things that annoy me.

1. Lack of Docker build support

Just cannot move any of the OSIO components into OSIO without this. Build is
building crap even build cannot use. The irony of a build tool that cannot use
itself is beyond me.

2. Lack of code ownership boundary

The default Jenkins file that gets imported into the project contains enough
nuances to get it wrong. Over time a user is going to change it bit by bit to
the point that when a build fails, there wont be a way to say if that is a
script failure or a system failure without looking into the logs. This is not a
sane way to build a service.

Need to be able to wrap a big try catch around the code that is provided by the
user such that

  a. The user maintains only their code, not this weird mix
  b. We have more confidence and easier debug process

3. Simple things should be simple.

I had to use this crap for a while to truly appreciate the beauty of platforms
like Travis or Circle CI.

4. Need a small footprint and API

All the API that is allowed inside the Jenkinsfile should be committed and
documented. This weird mix of fabric8 pipeline library, 130+ plugins and the
things that Jenkins itself provides make it a painful thing to maintain.

This leaves us in this silly clueless state with questions like

1. [Where does the 'container' function come from?](https://github.com/fabric8io/fabric8-build-team/issues/20) :confused:
2. [Why does this build fail?](https://github.com/fabric8io/fabric8-build-team/issues/26)
3. [Jenkins master reboot and slave reconnect issue](https://github.com/fabric8io/fabric8-build-team/issues/17)
