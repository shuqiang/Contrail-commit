# commit for opencontrail

## Fix tsn agent crash in FlowStatsManager::SetProfileData

Issue
-----
The agent pointer in FlowStatsManager is pointing to different memory location, causing agent crash.
In TSN mode, agent does not create FlowStatsManager/FlowStatsCollector.

Fix
---
In TSN mode, we don't register FlowStatscb.

https://review.opencontrail.org/c/Juniper/contrail-controller/+/53082
