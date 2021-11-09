# Continuous Integration ([SEPASSRFNT-41](https://jira.open-groupe.com/browse/SEPASSRFNT-41))

## Automated Build


## On-Target test automation

### Hardware
How to setup the testbench:
<TODO>

### Linux Test Project (LTP)
The test harness on the target is based on [LTP](https://github.com/linux-test-project/ltp).

#### Running the whole test suite
The complete test suite can be run with:
```
cd /opt/ltp
./runltp
```

Then the summary of results can be read from the `results` directory.

#### Running tlgate BSP tests
You can run all the tests specific to tlgate with the following "scenario":
```bash
cd /opt/ltp
RHOST_1=... RHOST_2=... ./runltp -f bsp.tlgate
```

In addition to the built-in LTP tests, we have specific tests for the following features.

*RHOST_1 and RHOST_2 variables are required for the network stress test. Read the instructions below to setup the 2 required hosts.*

#### Network
The two network interfaces can be tested simultaneously (manually) with:
```
RHOST_1=... RHOST_2=... /opt/ltp/testscripts/network.sh -B
```
**Where RHOST_1 and RHOST_2 are environment variables which should contain IP adresses of 2 hosts on 2 different sub-networks, running iperf3 server (with `iperf3 -s`).**

#### Serial connections
```
cd /opt/ltp
./runltp -f bsp.uarts
```

Also read [Configuration for quad-uarts chips](SEPASSRFNT-64-uarts.md) for details about the rs485-test and serial-loops program.

#### Logging
```
cd /opt/ltp
./runltp -f bsp.logging
```

To check connection with a remote server, read [SEPASSRFNT-6-syslog.md](SEPASSRFNT-6-syslog.md).

#### Real Time Clock
```
cd /opt/ltp
./runltp -f bsp.rtc
```

This runs 2 tests: one for the hardware RTC and one for the timesyncd service.

#### USB

#### GPIOs
```
cd /opt/ltp
./runltp -f bsp.gpio
```

#### Memory

#### Flash

#### EEPROM
```
cd /opt/ltp
./runltp -f bsp.eeprom
```

[Back](toc.md)
