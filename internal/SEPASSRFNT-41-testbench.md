# Continuous Integration ([SEPASSRFNT-41](https://jira.open-groupe.com/browse/SEPASSRFNT-41))

## Automated Build


## On-Target test automation

### Hardware
How to setup the testbench:
<TODO>

### Linux Test Project (LTP)
The test harness on the target is based on [LTP](https://github.com/linux-test-project/ltp).

The complete test suite can be run with:
```
cd /opt/ltp
./runltp
```

Then the summary of results can be read from the `results` directory.

In addition to the built-in LTP tests, we have specific tests for:

#### Network
The two network interfaces can be tested simultaneously with:
```
RHOST_1=... RHOST_2=... /opt/ltp/testscripts/network.sh -B
```
Where RHOST_1 and RHOST_2 are environment variables which should contain IP adresses of 2 hosts on 2 different sub-networks, running iperf3 server (with `iperf3 -s`).

#### Serial connections
Read [Configuration for quad-uarts chips](SEPASSRFNT-64-uarts.md) for details about the rs485-test program.

#### Logging

#### Real Time Clock

#### USB

#### sdcard

#### GPIOs

#### Memory

#### Flash

#### EEPROM

[Back](toc.md)
