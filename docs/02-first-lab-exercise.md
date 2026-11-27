# First lab exercise: Sysmon to Wazuh

I built a two-VM security monitoring lab in VMware Workstation. A Windows 11 VM runs Sysmon and the Wazuh agent, and an Ubuntu Server VM runs the Wazuh manager and dashboard. The VMs exchange telemetry over the private ADR-Lab network.

## Verify collection

I launched Notepad, found the Sysmon process creation event in Windows, and matched its process GUID in Wazuh's raw archive.

## Test detection

I created rule 100101 to alert when a Sysmon process creation event contains a unique lab marker. The first run exposed a mismatch between the test marker and the rule. After correcting it, I verified the alert in the manager log and found the matching result in Threat Hunting.

## Respond

I checked the endpoint, rule ID, and command line against my test. The alert matched expected lab activity, so no containment was needed. I saved evidence of both event delivery and the resulting alert.