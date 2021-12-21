# Logging with syslog

## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_development_methods/SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-6](https://jira.open-groupe.com/browse/SEPASSRFNT-6) | MG70 (18/12/21) | feature complete, must be integrated in CI for qualification |

## Feature Description

NOTE: For further design details, please also refer to [SEPASSRFNT-6](https://jira.open-groupe.com/browse/SEPASSRFNT-6)

The system is configured to be a syslog *client*, pushing logs to a syslog *server*.

## On the target
The target runs the rsyslog service which configuration is in `/etc/rsyslog.conf`. This file contains the address of the server, and it's set from a Yocto recipe override in `meta-sisgateway/recipes-extended/rsyslog/rsyslog_%.bbappend`

By default, the service uses TCP on port 514.

If you modify the config file on the target, don't forget to restart the service with `systemctl restart rsyslog`.

## On a server
You can configure any host as a syslog server:
- install rsyslog
- edit the config file in /etc/rsyslog.conf
- add to the configuration:
```
module(load="imtcp") # needs to be done just once
input(type="imtcp" port="514")
$template RemoteLogs,"/var/log/remote.log"
*.* ?RemoteLogs
& ~
```

## Check connection
On the target, you can read journals locally, first it will show failures to connect to the server:
```
am64xx-evm rsyslogd[634]: cannot connect to 192.168.0.1:514: Network is unreachable [v8.2002.0 try https://www.rsyslog.com/e/2027 ]
```

then, when the syslog client succeeds in connecting to the server, you'll be able to check remote logging.

On the target, type:
```bash
# echo "Logs test" | systemd-cat -t testing -p emerg
```
and then you'll get on the server, in the `/var/log/remote.log` file:
```
Jul 16 07:25:22 am64xx-evm testing[839]: Logs test
```

[Back](toc.md)
