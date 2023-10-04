# nerctools

This repository is for tools that make using nerc openstack resources a little easier for my groups use case.  In particular we want our vm's to be ondemand --- shut down when they are "idle" and automatically started when we a user attempts to ssh to them.

These tools assume you have the openstack command line client installed
- https://nerc-project.github.io/nerc-docs/openstack/advanced-openstack-topics/openstack-cli/openstack-CLI/#install-the-openstack-command-line-clients
- this repo also requires that you have a copy of the application credentials for our project ... ask your nerc project admin for a copy -- you will need to place it in the config directory as `default_cred.rc`
- your project admin will want to setup the idleculler on your servers
- the main tool is `openstackssh` -- see below





## `openstackssh`

A wrapper around `openstack server ssh <server> -l user` that tests to see if the server is up if not it will try and bring it up.

ssh config to use openstackssh transparently.  The following example  assumes:
   - nerc server name is `JA-CC`
   - user on the server is `jappavoo`
   - and that the tools are locally installed in `~/Work/nerctools/openstackssh`

```
Host JA-CC
     User jappavoo
#     ProxyCommand openstack server ssh JA-CC -- -l jappavoo -W 127.0.0.1:22
#     ProxyCommand /Users/jappavoo/Work/nerctools/nercssh JA-CC -l jappavoo -W 127.0.0.1:22
     ProxyCommand ~/Work/nerctools/openstackssh %n -l %r -W 127.0.0.1:%p

Host JA-CC-jup
     User jappavoo
     HostName JA-CC
     LocalForward 18888 localhost:18888
     RequestTTY yes
     RemoteCommand jupyter-lab --port 18888
     ProxyCommand ~/Work/nerctools/openstackssh JA-CC -l %r -W 127.0.0.1:%p
```

## config
### config/default_cred.sh
### config/defaultidlesec
### config/onidle
### config/idleculler
### confug/defaultserver

## isidle

## installidleculler

## isup




## Notes

Here are somen openstack commands that are useful
- list running :  `openstack server list
- status : `openstack server show -c status JA-CC` or more concisely `openstack server show -c status -f value JA-CC`

