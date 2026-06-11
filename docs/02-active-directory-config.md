# Part 2 — Active Directory Configuration

## Objective

This phase turns the bare Windows Server 2022 host into a working Active Directory environment. The AD DS role is installed, the server is promoted to the first domain controller of a new forest (`MyDFIR.local`), a test user is created, and the second Windows VM is joined to the domain. From this point on the lab has a directory of identities that subsequent phases can detect against and respond to — which is the entire reason a SOC cares about AD in the first place.

## Prerequisites

- Both Windows VMs deployed, in the same VPC, on `10.22.96.0/20`
- RDP reachable from the analyst workstation to both VMs
- Static IP configuration applied to the VPC NIC on both VMs (Part 1)
- Administrator password for each VM (visible in the Vultr panel)

## Steps

### Installing the AD DS role

RDP into `mydfir-ad-dc01` as `Administrator`. In Server Manager:

1. **Manage → Add Roles and Features → Next** through the wizard
2. Select **Active Directory Domain Services** when prompted, accept the *Add Features* dialog
3. Continue with defaults, click **Install**

The post-install banner reads *Configuration required. Installation succeeded on mydfir-ad-dc01.*

Server Manager kicks off the role install — the screenshots below show each wizard pane from launch through completion.

![Server Manager — Add Roles and Features wizard opened](../screenshots/02-active-directory/Domain_Controller_Setup_1.png)
*Server Manager → Add Roles and Features launched on `mydfir-ad-dc01`*

![Selecting Active Directory Domain Services in the role list](../screenshots/02-active-directory/Domain_Controller_Setup_2.png)
*Active Directory Domain Services role selected; required management features auto-added*

![AD DS install reaching the completion screen](../screenshots/02-active-directory/Domain_Controller_Setup_3.png)
*Installation progress complete — server is now eligible for promotion to a domain controller*

![Selecting AD DS in the role wizard](../screenshots/02-active-directory/activedirec.png)
![AD DS install complete](../screenshots/02-active-directory/creating_activedirec.png)

### Promoting the server to a domain controller

The yellow warning flag at the top of Server Manager links to **Promote this server to a domain controller**. In the deployment wizard:

- **Add a new forest** (no existing forest yet)
- Root domain name: `MyDFIR.local`
- DSRM password: a non-trivial password (the default policy blocks weak ones — `password` was rejected with *verification of safe mode password failed*)
- Click through the remaining defaults and **Install**

The server signs out automatically and reboots. After it comes back up, search "Active" from the Start menu — **Active Directory Users and Computers** is now present, which confirms the promotion succeeded.

Each pane of the deployment wizard, in order:

![Post-install yellow warning flag linking to the promotion wizard](../screenshots/02-active-directory/Domain_Controller_Promotion_1.png)
*Server Manager's post-install warning — *Promote this server to a domain controller**

![Deployment Configuration — "Add a new forest" selected](../screenshots/02-active-directory/Domain_Controller_Promotion_2.png)
*New forest selected — no existing AD infrastructure in the lab*

![Root domain name set to MyDFIR.local](../screenshots/02-active-directory/Domain_Controller_Promotion_3.png)
*Root domain name `MyDFIR.local` entered*

![Prerequisites check passing before install](../screenshots/02-active-directory/Domain_Controller_Promotion_4.png)
*Prerequisites check passed — install can proceed*

![Promotion complete, server about to restart](../screenshots/02-active-directory/Domain_Controller_Promotion_5.png)
*Promotion complete — server signs out and reboots into its new role as the DC*

![Promotion in progress](../screenshots/02-active-directory/applying_ad.png)
![Active Directory Users and Computers available on the DC](../screenshots/02-active-directory/AD_success.png)

After reboot the **Active Directory Users and Computers** console exposes the new `MyDFIR.local` forest:

![ADUC opened on the DC showing MyDFIR.local](../screenshots/02-active-directory/AD_Users_Computers_Open.png)
*Active Directory Users and Computers — `MyDFIR.local` tree expanded, ready for user creation*

### Creating the Bob Smith test user

In ADUC, expanded the `MyDFIR.local` forest → right-clicked **Users → New → User**:

- First name: `Bob`
- Last name: `Smith`
- Full name: `Bob Smith`
- User logon name: `BSmith`
- Password: `Winter2026!`
- Unchecked **User must change password at next logon**
- Checked **Password never expires** (lab convenience — not a setting that belongs in production)

Each step of the new-user dialog, in order:

![Right-clicking Users → New → User](../screenshots/02-active-directory/AD_User_Creation_1.png)
*Right-click context menu in ADUC — `New → User`*

![New User form populated with Bob Smith / BSmith](../screenshots/02-active-directory/AD_User_Creation_2.png)
*New user form filled out with `Bob Smith` and logon name `BSmith`*

![Setting the initial password for BSmith](../screenshots/02-active-directory/AD_User_Creation_3.png)
*Initial password set to `Winter2026!`; "Password never expires" enabled for lab convenience*

![BSmith visible in the Users container](../screenshots/02-active-directory/AD_User_Creation_4.png)
*New `Bob Smith` account visible in the Users OU of MyDFIR.local*

![Bob Smith account created](../screenshots/02-active-directory/newuser.png)

### Joining the test machine to the domain

RDP into the test workstation as the local `Administrator`. **This PC → Properties → Rename this PC (Advanced) → Change**, then:

- Member of → **Domain** → `MyDFIR`
- Authenticate as `Administrator` with the DC's admin password

The first attempt failed with *the specified domain either does not exist or cannot be contacted* — a DNS resolution failure. The test machine's preferred DNS server was blank, so it had no way to resolve `MyDFIR.local`. Fix:

1. **Change adapter options → Ethernet instance 02 → Properties → IPv4 → Preferred DNS server: `10.22.96.4`** (the DC's VPC IP)
2. Retry the domain join

This time it succeeded with *Welcome to the MyDFIR domain*. The machine prompted for a reboot.

The full domain-join sequence on the test machine, in order:

![System Properties on the test machine](../screenshots/02-active-directory/Domain_Join_1.png)
*System Properties → Computer Name → Change…*

![Computer Name/Domain Changes dialog with MyDFIR entered](../screenshots/02-active-directory/Domain_Join_2.png)
*Member of → Domain → `MyDFIR` entered*

![Domain join success popup](../screenshots/02-active-directory/Domain_Join_3.png)
*Welcome to the MyDFIR domain — restart prompt*

![Logon screen — "Sign in to: MYDFIR"](../screenshots/02-active-directory/Domain_Join_4.png)
*Test machine's logon screen now shows *Sign in to: MYDFIR* — trust established*

![DNS-related domain join error](../screenshots/02-active-directory/active_Direc_err.png)
![Domain join succeeded after pointing DNS at the DC](../screenshots/02-active-directory/activedirec_authen.png)
![Test machine reporting MyDFIR as its domain](../screenshots/02-active-directory/same_domaincontroller.png)

> **Troubleshooting — Domain Not Found Error:** The initial domain join
> attempt returned: *"The specified domain either does not exist or cannot
> be contacted."* This occurred because the target machine had no DNS
> server configured on its VPC network adapter. Without DNS pointing to
> the domain controller, the machine could not resolve MyDFIR.local to
> an IP address. The fix was straightforward — the preferred DNS server
> on Ethernet Adapter 2 was set to the domain controller's private VPC
> IP (10.22.96.4). The domain join succeeded immediately after this change.

### Configuring RDP access for the domain user

After the reboot, the test machine's logon screen now shows *Sign in to: MYDFIR*, confirming the domain trust. Logging on as `MyDFIR\BSmith` from the console worked immediately.

The interesting failure happened on RDP from the analyst workstation: `BSmith` / `Winter2026!` returned *the connection was denied because the user account is not authorized for remote login*. Standard domain users are not in the local **Remote Desktop Users** group by default.

Fix, from the test machine's console as `Administrator`:

1. Search `remote` → **Allow remote connections to this computer**
2. **Show settings → Select Users → Add → `BSmith` → Check Names → OK**
3. Retry RDP as `MyDFIR\BSmith`

The second RDP attempt landed straight on Bob's desktop.

The RDP enablement, BSmith authorization, and successful login, in order:

<!-- Firewall_RDP_Rule — the matching Vultr firewall RDP rule lives under screenshots/01-environment-setup/firewallgroup.png (set during VM provisioning). Not duplicated here. -->

![Allow Remote Desktop turned on for the test machine](../screenshots/02-active-directory/Remote_Desktop_Settings.png)
*System Properties → Remote → "Allow remote connections to this computer" enabled*

![Adding BSmith to the Remote Desktop Users group](../screenshots/02-active-directory/Add_BSmith_RDP.png)
*BSmith added via Select Users → Check Names → OK*

![RDP credentials dialog with MyDFIR\BSmith](../screenshots/02-active-directory/Bob_Smith_RDP_Login_1.png)
*Remote Desktop Connection — credentials entered as `MyDFIR\BSmith` (domain-qualified)*

![Successful desktop session as Bob Smith](../screenshots/02-active-directory/Bob_Smith_RDP_Login_2.png)
*Successful RDP — Bob Smith's session on the domain-joined test machine*

![Adding BSmith to the Remote Desktop Users group](../screenshots/02-active-directory/allow_remotecon.png)
![BSmith logged in over RDP to the test machine](../screenshots/02-active-directory/AD_login.png)

## Verification

This phase is complete when:

- `dsa.msc` (Active Directory Users and Computers) is present on the DC and the `MyDFIR.local` forest is visible
- The `BSmith` user exists under `Users` in the DC's directory tree
- The test machine's logon screen shows *Sign in to: MYDFIR*
- `BSmith` can RDP into the test machine from the analyst workstation as `MyDFIR\BSmith`
- An ADUC search for `Bob Smith` returns the user

## Troubleshooting

**Domain join fails with *the specified domain either does not exist or cannot be contacted*.** The joining machine can't resolve the domain name. On the joining machine, set the preferred DNS server on the VPC NIC to the DC's VPC IP (`10.22.96.4`) and try again.

**`The connection was denied because the user account is not authorized for remote login`.** The user is a domain user, not a member of the local Remote Desktop Users group on the target. Add the user via **System Properties → Remote → Select Users** on the target machine.

**`Logon attempt failed` even with the correct password.** The username didn't include the domain prefix and Windows resolved it as a local account. Use `MyDFIR\BSmith` instead of bare `BSmith`.

**DSRM password rejected with *verification of safe mode password failed*.** Windows applied default complexity requirements (length, mixed case, etc.). Pick something stronger than `password`.

## Key Concepts

- **Domain controller** — a server that hosts Active Directory Domain Services and answers authentication, authorization, and directory queries for the domain. The DC is the source of truth for who exists and what they can do.
- **Forest, domain, OU** — the forest is the top-level security boundary; a domain is an administrative partition inside it; OUs are containers for policy targeting. This lab has one forest, one domain, default OUs.
- **DNS and AD are inseparable** — domain join, group policy, and the Splunk Universal Forwarder all locate the DC via DNS service records (`_ldap._tcp.dc._msdcs.MyDFIR.local`). If DNS is wrong, AD looks broken.
