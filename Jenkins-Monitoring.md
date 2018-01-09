# Jenkins monitoring

On behalf of the build team, we would like to monitor the health of Jenkins instances.

These include

1. (A) The ad-hoc instance at jenkins.cd.test.fabric8.io
1. (B) The slave nodes it brings up
1. (C) Tenant images

## Metrics that we _really_ care about

1. Simple uptime and readiness percentage

   Since builds queuing up is one of the worst issues right now, we should
   hopefully see a big difference in the mertrics. Its up (high liveness) but is
   not really building anything (low readiness) 1.

## Zabbix

_I'm a Zabbix newbie; so I might be really wrong here. Please feel free to
correct anything that is potentially wrong. Thanks to @konarde for a quick heads
up and most of this info_

1. Zabbix is part of the existing infrastructure and the single point of truth
   already for a bunch of services. There is value in using this and not using
   new tools.

2. Works for node level monitoring. There is not much work required to get a
   basic uptime metric etc. I might be able to write some code to send queuing
   time periodically to Zabbix with a simple job that is run in a cron or
   something. I expect it to handle liveness metrics just fine out of the box.

3. 3 kind of probes

  1. Active agent: Install a daemon which sends info to a Zabbix master.
    1. How much do we have to tweak our GKE firewall config to make this work?

  2. Passive agent: Setup a web server on the node which gets queried by zabbix master.
    1. Is this connection encrypted?

  3. Web probe: Make zabbix query a URL (if provided by Jenkins) repeatedly and
     report on that.
     1. Does Jenkins provide any such metrics?

## Stuff to do:

  1. How different is monitoring A B and C?
  2. Pradeepto considers (A) to be a priority now since it blocks other people.
  2. Apparently Prometheus is built into openshift. Should we use it instead of
     zabbix?

## References

1. https://www.datadoghq.com/blog/monitor-jenkins-datadog/
