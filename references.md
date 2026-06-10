# References

External documentation, downloads, and reference material used to build and document this lab.

## Tools and Downloads

| Tool | Version | Download / Home |
|------|---------|-----------------|
| Vultr Cloud | Current platform | [vultr.com](https://www.vultr.com/) |
| Windows Server 2022 Standard | Eval / Vultr-licensed image | [microsoft.com/evalcenter/windows-server-2022](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) |
| Ubuntu Server | 22.04 LTS | [ubuntu.com/download/server](https://ubuntu.com/download/server) |
| Splunk Enterprise | 9.x (free trial) | [splunk.com/en_us/download/splunk-enterprise.html](https://www.splunk.com/en_us/download/splunk-enterprise.html) |
| Splunk Universal Forwarder | 9.x | [splunk.com/en_us/download/universal-forwarder.html](https://www.splunk.com/en_us/download/universal-forwarder.html) |
| Splunk Add-on for Microsoft Windows | Latest | [splunkbase.splunk.com/app/742](https://splunkbase.splunk.com/app/742) |
| Shuffle SOAR | Cloud-hosted | [shuffler.io](https://shuffler.io/) |
| Discord | N/A | [discord.com](https://discord.com/) |
| `ufw` (Uncomplicated Firewall) | 0.36 | [help.ubuntu.com/community/UFW](https://help.ubuntu.com/community/UFW) |

## Windows Event ID References

| Event ID | Description | Where to look it up |
|----------|-------------|---------------------|
| **4624** | An account was successfully logged on | [Ultimate Windows Security 4624](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624) |
| 4625 | An account failed to log on | [Ultimate Windows Security 4625](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625) |
| 4634 | An account was logged off | [Ultimate Windows Security 4634](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4634) |
| 4647 | User initiated logoff | [Ultimate Windows Security 4647](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4647) |
| 4672 | Special privileges assigned to new logon | [Ultimate Windows Security 4672](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4672) |
| 4720 | A user account was created | [Ultimate Windows Security 4720](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4720) |
| 4725 | A user account was disabled | [Ultimate Windows Security 4725](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4725) |

### Logon Type quick reference

| Type | Meaning |
|------|---------|
| 2 | Interactive (console) |
| 3 | Network (SMB, named pipe) |
| 4 | Batch |
| 5 | Service |
| **7** | **Unlock / reconnected RDP** |
| 8 | NetworkCleartext |
| 9 | NewCredentials (RunAs) |
| **10** | **RemoteInteractive (RDP)** |
| 11 | CachedInteractive |

Microsoft reference: [Audit Logon Events — Logon Type values](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)

## Vultr Documentation

- [Deploy a Server — Vultr Docs](https://docs.vultr.com/how-to-deploy-a-new-cloud-server)
- [Virtual Private Cloud (VPC) overview](https://docs.vultr.com/about-vpc-2-0)
- [Firewall Groups](https://docs.vultr.com/vultr-firewalls)
- [Manage Windows VMs over RDP](https://docs.vultr.com/how-to-access-and-control-a-vultr-cloud-server-via-rdp)

## Splunk Documentation

- [Splunk Enterprise — Install on Linux](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonLinux)
- [Splunk Universal Forwarder — Install on Windows](https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/InstallaWindowsuniversalforwarderfromaninstaller)
- [`inputs.conf` reference](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf)
- [Configure receivers — Splunk Web](https://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Enableareceiver)
- [Scheduled alerts and cron expressions](https://docs.splunk.com/Documentation/Splunk/latest/Alert/Definescheduledalerts)
- [Webhook alert action](https://docs.splunk.com/Documentation/Splunk/latest/Alert/Webhooks)
- [Splunk Add-on for Microsoft Windows — docs](https://docs.splunk.com/Documentation/AddOns/released/MSWindows/About)

## Shuffle SOAR Documentation

- [Shuffle — Getting Started](https://shuffler.io/docs/about)
- [Workflow building blocks](https://shuffler.io/docs/workflows)
- [Webhook triggers](https://shuffler.io/docs/triggers#webhook)
- [User input trigger](https://shuffler.io/docs/triggers#user_input)
- [Active Directory app reference](https://shuffler.io/apps/active_directory)
- [Self-hosting Shuffle workers](https://shuffler.io/docs/configuration#self_hosting)

## Discord Webhooks

- [Intro to Webhooks (Discord Support)](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)
- [Executing a Webhook — Discord Developer Docs](https://discord.com/developers/docs/resources/webhook#execute-webhook)

## Active Directory References

- [Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Install Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [`userAccountControl` flags reference](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties)
- [LDAP overview for AD](https://learn.microsoft.com/en-us/windows/win32/ad/ldap)

## YouTube Series This Lab Was Built Against

- *Cybersecurity Project — Active Directory 2.0* by MyDFIR — five-part series (Intro + Parts 2–5). The transcripts that drove the build sequence in this repo are preserved under `snips/*.txt`.

## MITRE ATT&CK Techniques Demonstrated

| ID | Name | Where it shows up in this lab |
|----|------|-------------------------------|
| [T1078](https://attack.mitre.org/techniques/T1078/) | Valid Accounts | The whole detection — an attacker using legitimate credentials (`JSmith`) to log on. |
| [T1078.002](https://attack.mitre.org/techniques/T1078/002/) | Valid Accounts: Domain Accounts | Specifically a domain account (`MyDFIR\JSmith`), not a local one. |
| [T1021](https://attack.mitre.org/techniques/T1021/) | Remote Services | The category the RDP technique sits under. |
| [T1021.001](https://attack.mitre.org/techniques/T1021/001/) | Remote Services: Remote Desktop Protocol | The exact protocol detected (Logon Type 10 over TCP/3389). |
| [T1531](https://attack.mitre.org/techniques/T1531/) | Account Access Removal | The SOAR response — disabling the account is the defensive analogue. |
