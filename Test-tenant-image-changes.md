
This page is a WIP for https://github.com/fabric8io/fabric8-build-team/issues/47

We can manual test this though.

1.    Update oc binary just like KB mentioned.
1.    The PR will get a comment from a bot with image version.
1.    Use that image version to build a s2i image
1.    Bot will comment a link to use that tenant image in osio
1.    Update your tenant image and run one or 2 quickstart builds
1.    If that goes well, share the logs and raise a PR.

