# Jenkins monitoring

On behalf of the build team, we would like to monitor the health of fabric8 Jenkins instance [http://jenkins.cd.test.fabric8.io/] .

These include

  A. The ad-hoc instance at jenkins.cd.test.fabric8.io
  B. The slave nodes it brings up
  C. Nodes used for OSIO itself

## Zabbix

__(Thanks to @konarde for a quick heads up and most of this info)__

__I'm a complete newbie; so I might be really wrong here. Please feel free to
correct the stuff.__

  1. Zabbix is part of the existing infrastructure and the single point of truth
     already for a bunch of services. There is value in using this and not using
     new tools.

  2. Works for node level monitoring. There is not much work required to get a
     basic uptime metric etc.

  3. 3 kind of probes

     1. Active agent: Install a daemon which sends info to a Zabbix master.
        1. How much do we have to tweak our firewall config to make this work?

     2. Passive agent: Setup a web server on the node which gets queried by zabbix master.
        1. Is this connection encrypted?

     3. Web probe: Make zabbix query a URL (if provided by Jenkins) repeatedly
        and report on that.

## Stuff to do:

  1. How different is monitoring A B and C?
  2. Apparently Prometheus is built into openshift. Should we use it instead of
     zabbix?
