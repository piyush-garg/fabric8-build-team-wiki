** Multi cluster support
   Copy at https://github.com/fabric8io/fabric8-build-team/issues/8#issuecomment-355946351

1. Plan for scaling OSIO into multiple openshift clusters. Now we only have
   us-starter-east, but there will be few more in the future to scale.

2. Effectively a user level shading. OSIO itself is going to run in one cluster.

3. Profile, identity and cluster information is in auth service. It is the
   single source of truth.

4. An Openshift Service account token is the permission to create a namespace
   etc on behalf of some other user. Auth stores the service token for all
   clusters.

   Q1. Is the service token per user or applicable for the entire cluster?
   Q2. What is the damage/impact of a security breach?

5. Tenant service needs the openshift service token for the user's cluster to
   create the apps and namespace on their behalf and it gets it from the auth
   service.

6. The Platform Proxy abstracts the cluster specific info from all the apps that
   talk to OSIO. For example, the Quick start, pipeline etc and it automagically
   picks the right openshift to talk to behind the scene.

   Q1. Where is the code/docs/design/scope of this proxy?
   Q2. Single point of failure/scale?

Notes for the build team:

1. What about Jenkins idler? Does this have to be multi cluster aware? How much
   work is required here?

2. Same as 1, but for fabric8-maven-plugin

3. Find the point Jenkins deploys with fabric8 maven plugin and see if it can be
   made local cluster aware.

