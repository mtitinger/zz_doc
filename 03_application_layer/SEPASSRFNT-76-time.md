# Time-keeping

## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_development_methods/SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-99](https://jira.open-groupe.com/browse/SEPASSRFNT-99) | MG80 (18/12/21) | development complete, qualified, not further test harness or special software configuration. |

## Feature Description


The system uses NTP to sync time with external servers, through systemd-timesyncd daemon.

One can get information with the commands:
```bash
# systemctl status timesyncd
# timedatectl
```

Note that timezones information is *not* installed, so `timedatectl list-timezones` shows nothing.

Locally, time is kept with an RTC backup from a coin cell. Hardware information can be read from the `/sys/class/rtc/rtc0` directory.

To save system time into the RTC manually:
```bash
# hwclock --systohc --utc
```

[Back](toc.md)
