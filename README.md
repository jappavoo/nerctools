# nerctools

This repository is for tools that make using nerc openstack resources a little easier for my groups use case.  In particular we want our vm's to be ondemand --- shut down when they are "idle" and automatically started when we a user attempts to ssh to them.

These tools assume you have the openstack command line client installed
- https://nerc-project.github.io/nerc-docs/openstack/advanced-openstack-topics/openstack-cli/openstack-CLI/#install-the-openstack-command-line-clients
- this repo also requires that you have a copy of the application credentials for our project ... ask your nerc project admin for a copy -- you will need to place it in the config directory as `default_cred.rc`
- your project admin will want to setup the idleculler on your servers
- the main tool is `openstackssh` -- see below


The project admin of a new vm needs to install the tools, setup the
configuration and install the periodic cronjob.  See below for details.


## `openstackssh`

This is the main script run by the users of a openshift vm to access it in an ondemand way.

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

## `config`

You can configure and influence the behaviour of the tools by placing appropriately name files in this directory.
The following describes each of the files, what they influence and what put in them.

### `config/default_cred.sh`

The application credential file that was obtained from the openstack project administrator.  This contains secrets and
should be secured appropriately.  Do not put on public repos or in places where its contents can be read by other users.

### `config/idleusers`

Used on the vm to specify  the list of users used to detect idle.  Processes owned by members of this list
will determine if the vm is considered idle.  The file should should be a space or newline seperated list of
user names.

### `config/idlegroup`

Used on the vm to specify a list of users via UNIX group name used to detect idle.
Processes owned by users who have this group as secondary group will be used
will determine if the vm is considered idle.  The file should contain the name of a single
UNIX group.  We use a group called `data` for all our users that we expect to be doing work
on the machine. You can use both `idleusers` and `idlegroup`.


### `config/idlesec`

Used on the vm to override the default number of seconds of finding no processes of the specified user group
before the system will be considered in the idle state. The contents should contain a single integer number of seconds.
Eg.

```
$ echo $(( 60 * 15 )) > config/idlesec
```
Will set the idle second threshold to 15 minutes of not detecting any processes in the users in the idlegroup.
When this time is exceeded the action in `config/onidle` will be taken.

### `config/maxlifetimesec`

Overides the default time in seconds that the vm will be allowed to stay alive for.  Note this is independent of idleness.
The file should contain a single integer value in seconds. Eg.
```
$ echo $((60 * 60 * 8)) > config/maxlifetimesec
```
Will set the maximum time the system is allowed to live for to 8 hours.  When this time expires the action specified
in `config/onlifetimeexeed` will be taken.

### config/warninglifetimesec

Overides the default time in seconds that will trigger a warning action to be taken if the vm is on.  Note this is independent of idleness.
The file should contain a single integer value in seconds. Eg.
```
$ echo $((60 * 60 * 8)) > config/warninglifetimesec
```
Will set the maximum time the system is allowed to live for to 8 hours.  When this time expires the action specified
in `config/onlifetimewarning` will be taken.


### `config/onidle`

On the vm it specifies a command to execute on when the system is determined to be idle.
Eg.
```
echo "sudo shutdown now" > config/onidle
```

### config/onlifetimewarning

On the vm it specifies a command to execute on when the system has be alive for this period.
Eg.
```
echo "touch /tmp/lifetimewarning" > config/onlifetimewarning
```

### config/onlifetimeexceed
On the vm it specifies a command to execute on when the system has be alive for this period.
Eg.
```
echo "sudo shutdown now" > config/onlifetimeexceed
```

### config/idleculler

Used on the vm to how often to periodically to run the checklifetime and the isidle (and onidle) scripts via a cronjob.
This is a value in minutes.  It is used by the `installidleculler` script that needs to be run on the vm by your
project admin.


## isidle

Run on the vm by the idle culler job to determine if the system is idle.  This is safe to run by hand but
generally you should not need to.

## checklifetime

Run on the vm by the idle culler job to determine if the system's life time has exceed the warning or max lifetimes.
Not recommended to be run by hand but you can if you don't mind the actions to be taken if lifetime thresholds have
been exceed. 

## installidleculler

Run on the vm to setup to execute the checklifetime and the isidle (and onidle) scripts via a cronjob.

## isup

Utility script that uses the same logic as openstackssh to check if the vm.


## Notes

Here are somen openstack commands that are useful
- list running :  `openstack server list
- status : `openstack server show -c status JA-CC` or more concisely `openstack server show -c status -f value JA-CC`

