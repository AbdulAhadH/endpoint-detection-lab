# Deployment Log

This document captures the deployment process

## 1. Wazuh server install

Installed Wazuh manager, indexer, and dashboard as a single-node deployment on an
Ubuntu VM.

## 2. Agent deployment (Windows)

Generated an install command from the dashboard's **Deploy new agent** wizard and
installed the Wazuh agent on the first Windows VM. Getting it fully connected involved
working through a few issues along the way: the server address field needed to be
typed manually rather than pasted, the Windows service's actual internal name turned
out to be different from what's shown in the service list, and the agent initially
failed to start due to a leftover placeholder server address in its config file from
a prior install attempt. Once resolved, the agent came up as Pending in the dashboard
and flipped to Active within about a minute.

A second agent was deployed the same way and briefly showed as disconnected in the
dashboard, even though the manager itself reported it as healthy and actively
receiving data.

## 3. Sysmon → Wazuh integration

After installing Sysmon and configuring Wazuh to read its event log channel, an initial
search for the expected data came back empty even though Sysmon was clearly generating
events. Tracking it down came down to understanding exactly which field in a Wazuh
alert actually holds the Windows event channel name versus which field just holds a
generic placeholder value. Once searching on the correct field, the full pipeline —
Sysmon → agent → manager → rule engine → indexer → dashboard — was confirmed working
end-to-end, including a real alert automatically generated and mapped to a MITRE ATT&CK
technique from ordinary command activity.

## 4. Checking agent health going forward

Between the two incidents above, a few checks turned out to be the ones that actually matter day-to-day: on the manager, agent_control -i <id> shows the true live status and last keep-alive time, which is more reliable than the dashboard alone. On the Windows side, confirming the WazuhSvc service is actually running (not just installed) catches the most common failure mode, and the agent's own ossec.log almost always explains why a service that starts immediately stops again. When something looks disconnected but the manager disagrees, checking the manager's own indexer-sync status is worth doing before assuming the agent itself is the problem.

## Key takeaways
Reinstalling an agent over an existing install can silently preserve a broken config file. Worth checking the config directly rather than assuming a reinstall fixed everything.
A service's internal name and its display name in Windows aren't always the same thing, which can make a service look unavailable when it's really just being called by the wrong name.
The dashboard's agent status and the manager's own live state can disagree temporarily. It's best to check both rather than trusting one source alone.
When expected data doesn't show up in a search, it's worth confirming the exact field name being used rather than assuming the obvious one is correct.
