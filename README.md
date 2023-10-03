# nerctools

These tools assume you have the openstack command line client installed
- https://nerc-project.github.io/nerc-docs/openstack/advanced-openstack-topics/openstack-cli/openstack-CLI/#install-the-openstack-command-line-clients
- this repo requires that you have a copy of the application credentials for our project ... ask jappavoo@bu.edu for a copy


Repository to make using nerc resources easier

- list running :  `openstack server list
- status : `openstack server show -c status JA-CC` or more concisely `openstack server show -c status -f value JA-CC`


## nercssh

A wrapper around `openstack server ssh <server> -l user` that tests to see if the server is up if not it will try and bring it up.

ssh config to use nercssh transparently (following assumes nerc server name is JA-CC and user on the server is jappavoo)
```
Host JA-CC
     User jappavoo
          ProxyCommand /Users/jappavoo/Work/nerctools/nercssh %n -- -l %r -W 127.0.0.1:%p
```
