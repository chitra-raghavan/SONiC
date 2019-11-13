# sFlow Test Plan
## Overview
The purpose is to test the functionality of the sFlow monitoring system on the SONIC switch DUT . The test assumes that the host bridge has been preconfigured according to the t0 topology.
### Scope
The test is targeting a running SONIC system with fully functioning configuration. Rather, it is testing the functionality of sFlow on the SONIC system by verifying that sFlow is monitoring traffic flow data correctly.
## Test Structure
### Setup Configuration
The test will run on the t0 testbed:

![](../raw/gh-pages/images/sflow/testbed-t0.png)

* Ethernet0, Ethernet4, Ethernet8, and Ethernet12 of dut are used for traffic.
* sFlow collectors are configured to be reachable from Ethernet16 and Ethernet20.
* Collection is implemented using the sflowtool. Counter sampling output and flow sampling output are directed to a text file. The test script parses the text file and validates the data according to the polling/sampling rate configured and the interfaces enabled.

### Test Cases
#### Test Case #1
##### Test objective
Verify that the SFLOW_COLLECTOR configuration additions and configuration deletions are processed by hsflowd.

| # | Steps | Expected Result |
|---|---|---|
| 1. | 1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Add a single collector, "config sflow collector add <_collector name_> <_ip_>". 3. Enable sFlow globally, "config sflow enable". | The configurations should be reflected in “show sflow” and "show sflow interface". Samples should be received by the collector . |
| 2. | 1. Remove the collector, "config sflow collector del <_collector name_>". 2. Add it back, "config sflow collector add <_collector name_> <_ip_>". | The configurations should be reflected in “show sflow” and "show sflow interface". Samples should be received by the collector . |
| 3. |1. Remove the collector, "config sflow collector del <_collector name_>". | The configurations should be reflected in “show sflow”  and "show sflow interface". Samples should not be sent to the removed collector. |
| 4. | 1. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". | The configurations should be reflected in “show sflow” and "show sflow interface". Samples should be received by both collectors. |
| 5. | 1. Remove the second configured collector "config sflow collector del <_collector name_>". | Samples should continue to be received by the first collector. |
| 6. | 1. Add back the second collector, "config sflow collector add <_collector name_> <_ip_>". 2. Remove the first configured collector, "config sflow collector del <_collector name_>". | Samples should continue to be received by the second collector. |
| 7. | 1. Attempt to add 2 more collectors for a total of 3, "config sflow collector add <_collector name_> <_ip_>". | An error message should be returned stating that only two collectors are supported. |

#### Test Case #2
##### Test objective
Verify that it is possible to change the counter polling interval using the SFLOW table

| # | Steps | Expected Result |
|---|---|---|
| 1. |1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Enable sFlow globally, "config sflow enable". 3. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". | Both collectors should be receiving samples at the default counter polling rate of 20 seconds. |
| 2. | 1. Configure the counter polling interval to 0 seconds, "config sflow polling-interval <_time interval_>". | Counter polling should be disabled and both collectors should continue receiving samples. |
| 3. |1. Configure the counter polling interval to 60 seconds, "config sflow polling-interval <_time interval_>". | Both collectors should be receiving samples at a counter polling rate of 60 seconds. |

#### Test Case #3
##### Test objective
Verify that it is possible to change the agent-id using the SFLOW table

| # | Steps | Expected Result |
|---|---|---|
| 1. | 1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Enable sFlow globally, "config sflow enable". 3. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". | Both collectors should be receiving samples with the default agent-id selected by hsflowd. |
| 2. | 1. Add an agent-id, "config sflow agent-id add <_interface name_>". | The configurations should be reflected in “show sflow” and both collectors should receive samples with the agent-id configured. |
| 3. | 1. Remove the agent-id, "config sflow agent-id del". | Both collectors should be receiving samples with the default agent-id selected by hsflowd. |
| 4. | 1. Add an agent-id, "config sflow agent-id add <_interface name_>". | Both collectors should receive samples with the agent-id configured. |

#### Test Case #4
##### Test objective
Verify that interfaces can be enabled/disabled using additions/deletions in SFLOW_SESSION table

| # | Steps | Expected Result |
|---|---|---|
| 1. | 1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Enable sFlow globally, "config sflow enable". 3. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". | The samples received by both collectors should reflect the changes. |
| 2. | 1. Disable 2 interfaces, "config sflow interface disable <_interface name_>". | The samples received by both collectors should reflect the changes. |

#### Test Case #5
##### Test objective
Verify that it is possible to change the sampling rate per interface using SFLOW_SESSION interface sample rate field

| # | Steps | Expected Result |
|---|---|---|
| 1. | 1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". 3. Configure an interface sampling rate of 256, "config sflow interface sample-rate <_rate_>". | The configurations should be reflected in “show sflow” and "show sflow interface" |
| 2. | 1. Enable sFlow globally, "config sflow enable". 2. Change the sampling rate for the interfaces to 512, "config sflow interface sample-rate <_rate_>". | The samples received by both collectors should reflect the changes. |
| 3. | 1. Restore the sampling rate of the previously configured interfaces, "config sflow interface sample-rate <_rate_>". | The samples received by both collectors should reflect the changes. |

#### Test Case #6
##### Test objective
Verify that with config saved in the config_db.json, restarting the unit should result in sFlow coming up with saved configuration.

| # | Steps | Expected Result |
|---|---|---|
| 1. |1. Enable 4 interfaces, "config sflow interface enable <_interface name_>". 2. Enable sFlow globally, "config sflow enable". 3. Save the configuration and then reboot. | The configurations should be reflected in “show sflow” and "show sflow interfaces". |
| 2. | 1. Disable sFlow globally, "config sflow disable". Save the configuration and then reboot. | The configurations should be reflected in “show sflow” and "show sflow interfaces". |
| 3. | 1. Enable sFlow, "config sflow enable". 2. Add two collectors, "config sflow collector add <_collector name_> <_ip_>". 3. Configure the counter polling interval, "config sflow polling-interval <_time interval_>". 4. Configure the interfaces, "config sflow interface enable <_interface name_>". 5. Save the configuration and then reboot. | The configurations should be reflected in “show sflow” "show sflow interfaces". |

