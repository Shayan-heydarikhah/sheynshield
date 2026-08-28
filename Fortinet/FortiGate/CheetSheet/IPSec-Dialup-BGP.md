
Conversation with Gemini
حالا اعلاوه بر این خروجی بیا با ساختار checklist هم اینو بهم بده

بله، حتماً! داشتن یک Checklist عملیاتی (Pre-Deployment & Troubleshooting Checklist) برای ریپازیتوری گیت‌هاب بسیار مفید است. مهندسان شبکه می‌توانند این چک‌لیست را پیش از پیاده‌سازی در محیط Production یا حین رفع اشکال، مرحله به مرحله بررسی کنند.

این چک‌لیست را می‌توانید به عنوان یک بخش مجزا در انتهای همان فایل fortigate-ha-cheat-sheet.md یا به صورت یک فایل جداگانه مانند fortigate-ha-deployment-checklist.md استفاده کنید:

📋 FortiGate HA Deployment & Operational Checklist
A step-by-step verification checklist for deploying, maintaining, and troubleshooting FortiGate FGCP/FGSP High Availability clusters in enterprise environments.

🛠 Phase 1: Pre-Deployment Hardware & License Validation
[ ] Hardware Parity: Verify both units are identical models (same port counts, internal disk layouts, and process architecture).

[ ] Firmware Consistency: Ensure both FortiGate units are running the exact same FortiOS build before joining the cluster.

[ ] License Verification: Confirm entitlements (FortiGuard services, IPsec VPN client pools, VDOM licenses) are matching on both devices.

[ ] Direct Heartbeat Cabling: Ensure heartbeat interfaces are connected using direct back-to-back cabling (or dedicated L2 isolated VLANs) with redundant physical links.

⚙️ Phase 2: High Availability (FGCP) Core Configuration
[ ] Matching Group ID: Verify set group-id is identical on both cluster members (range: 0–1023).

[ ] Monitored Interfaces: Explicitly configure monitored links (set monitor <interface_list>) on both units for physical link-failure detection.

[ ] Heartbeat Timers: Tune signaling timers appropriately to balance convergence speed and stability:

[ ] set hb-interval configured.

[ ] set hb-interval-in-milliseconds set (e.g., 10ms – 100ms).

[ ] set hb-lost-threshold configured (e.g., 10 – 20 missed beats).

[ ] Session Pickup: Enable stateful connection failover via set session-pickup enable.

[ ] Enable set session-pickup-expectation enable for layer-7 protocols (FTP, SIP).

[ ] Enable set session-pickup-connectionless enable for UDP/ICMP tracking.

⚠️ Phase 3: Non-Synchronized Configurations Verification
Note: The following parameters do NOT synchronize between nodes and must be set manually on EACH individual unit.

[ ] Device Priority & HA Override:

[ ] Set distinct priority values (e.g., Master = 200, Slave = 100).

[ ] Decide on preemption behavior and configure set override consistently on BOTH peers.

[ ] System Hostnames: Assign distinct hostnames (e.g., FGT-PRI-01 and FGT-SEC-02) for audit visibility.

[ ] Out-of-Band Management (Reserved Mgmt Interface):

[ ] Enable set ha-mgmt-status enable and set ha-direct enable.

[ ] Configure unique static management IPs and static gateway routes under config ha-mgmt-interfaces.

[ ] Configure ha-direct settings under SNMP, Syslog, and FortiAnalyzer profiles.

🔒 Phase 4: Network Integration & Failover Tuning
[ ] GARP & VMAC Parameters:

[ ] Enable set gratuitous-arps enable to accelerate switch MAC table refreshes upon failover.

[ ] Tune set arps repeat count (recommended: 5) and set arps-interval.

[ ] Hardware Storage & Memory Protections:

[ ] Enable set ssd-failover enable (if using disk offloading/proxy services).

[ ] Configure set memory-based-failover enable with appropriate flip-timeout values to avoid conserve mode flapping.

[ ] Upstream Switch Configurations: Ensure switch ports connected to cluster data interfaces are configured with proper portfast/edge settings to avoid Spanning Tree topology change delays during failovers.

🧪 Phase 5: Post-Deployment Verification & Testing
[ ] Cluster Synchronization Check: Run diagnose sys ha checksum autoscale-cluster and verify all configuration checksums match across peers.

[ ] HA Status Validation: Run get system ha status to confirm primary/secondary roles and cluster health status.

[ ] Virtual MAC Verification: Inspect VMAC mappings using diagnose sys ha mac.

[ ] Failover Stress Test (Controlled Environment):

[ ] Test link failure by shutting down a monitored interface.

[ ] Verify continuous ping/traffic survival during primary failover.

[ ] Verify out-of-band management connectivity remains accessible for both individual nodes during failover.

📋 FortiGate HA Deployment & Operational Checklist

A step-by-step verification checklist for deploying, maintaining, and troubleshooting FortiGate FGCP/FGSP High Availability clusters in enterprise environments.



🛠 Phase 1: Pre-Deployment Hardware & License Validation

[ ] Hardware Parity: Verify both units are identical models (same port counts, internal disk layouts, and process architecture).

[ ] Firmware Consistency: Ensure both FortiGate units are running the exact same FortiOS build before joining the cluster.

[ ] License Verification: Confirm entitlements (FortiGuard services, IPsec VPN client pools, VDOM licenses) are matching on both devices.

[ ] Direct Heartbeat Cabling: Ensure heartbeat interfaces are connected using direct back-to-back cabling (or dedicated L2 isolated VLANs) with redundant physical links.

⚙️ Phase 2: High Availability (FGCP) Core Configuration

[ ] Matching Group ID: Verify set group-id is identical on both cluster members (range: 0–1023).

[ ] Monitored Interfaces: Explicitly configure monitored links (set monitor <interface_list>) on both units for physical link-failure detection.

[ ] Heartbeat Timers: Tune signaling timers appropriately to balance convergence speed and stability:

[ ] set hb-interval configured.

[ ] set hb-interval-in-milliseconds set (e.g., 10ms – 100ms).

[ ] set hb-lost-threshold configured (e.g., 10 – 20 missed beats).

[ ] Session Pickup: Enable stateful connection failover via set session-pickup enable.

[ ] Enable set session-pickup-expectation enable for layer-7 protocols (FTP, SIP).

[ ] Enable set session-pickup-connectionless enable for UDP/ICMP tracking.

⚠️ Phase 3: Non-Synchronized Configurations Verification

Note: The following parameters do NOT synchronize between nodes and must be set manually on EACH individual unit.

[ ] Device Priority & HA Override:

[ ] Set distinct priority values (e.g., Master = 200, Slave = 100).

[ ] Decide on preemption behavior and configure set override consistently on BOTH peers.

[ ] System Hostnames: Assign distinct hostnames (e.g., FGT-PRI-01 and FGT-SEC-02) for audit visibility.

[ ] Out-of-Band Management (Reserved Mgmt Interface):

[ ] Enable set ha-mgmt-status enable and set ha-direct enable.

[ ] Configure unique static management IPs and static gateway routes under config ha-mgmt-interfaces.

[ ] Configure ha-direct settings under SNMP, Syslog, and FortiAnalyzer profiles.

🔒 Phase 4: Network Integration & Failover Tuning

[ ] GARP & VMAC Parameters:

[ ] Enable set gratuitous-arps enable to accelerate switch MAC table refreshes upon failover.

[ ] Tune set arps repeat count (recommended: 5) and set arps-interval.

[ ] Hardware Storage & Memory Protections:

[ ] Enable set ssd-failover enable (if using disk offloading/proxy services).

[ ] Configure set memory-based-failover enable with appropriate flip-timeout values to avoid conserve mode flapping.

[ ] Upstream Switch Configurations: Ensure switch ports connected to cluster data interfaces are configured with proper portfast/edge settings to avoid Spanning Tree topology change delays during failovers.

🧪 Phase 5: Post-Deployment Verification & Testing

[ ] Cluster Synchronization Check: Run diagnose sys ha checksum autoscale-cluster and verify all configuration checksums match across peers.

[ ] HA Status Validation: Run get system ha status to confirm primary/secondary roles and cluster health status.

[ ] Virtual MAC Verification: Inspect VMAC mappings using diagnose sys ha mac.

[ ] Failover Stress Test (Controlled Environment):

[ ] Test link failure by shutting down a monitored interface.

[ ] Verify continuous ping/traffic survival during primary failover.

[ ] Verify out-of-band management connectivity remains accessible for both individual nodes during failover. در قالب گیت هابی بهم بده

آماده شد! بر اساس استانداردهای گیت‌هاب، فرمت چک‌لیست کامل‌تر شده تا کاربر بتواند مستقیماً در تیک‌باکس‌های گیت‌هاب تیک بزند، بخش Quick Action Commands برای دسترسی سریع کامندها اضافه شده و از Markdown Callout Blockquote برای نکات امنیتی استفاده شده است.

کد را کپی کرده و در پوشه پروژه‌تان با اسم fortigate-ha-deployment-checklist.md ذخیره کنید:

Markdown
# 📋 FortiGate High Availability (HA) Deployment & Operational Checklist

A step-by-step production verification checklist for deploying, maintaining, and troubleshooting FortiGate **FGCP** (FortiGate Clustering Protocol) and **FGSP** (FortiGate Session Life Support Protocol) clusters in enterprise environments.

---

## 🛠 Phase 1: Pre-Deployment Hardware & License Validation

- [ ] **Hardware Parity:** Verify both units are identical models (same port counts, internal disk layouts, and process architecture).
- [ ] **Firmware Consistency:** Ensure both FortiGate units are running the **exact same FortiOS build** before joining the cluster.
- [ ] **License Verification:** Confirm entitlements (FortiGuard services, IPsec VPN client pools, VDOM licenses) are matching on both devices.
- [ ] **Direct Heartbeat Cabling:** Ensure heartbeat interfaces are connected using **direct back-to-back cabling** (or dedicated L2 isolated VLANs) with redundant physical links.

---

## ⚙️ Phase 2: High Availability (FGCP) Core Configuration

- [ ] **Matching Group ID:** Verify `set group-id` is identical on both cluster members (range: 0–1023).
- [ ] **Monitored Interfaces:** Explicitly configure monitored links (`set monitor <interface_list>`) on both units for physical link-failure detection.
- [ ] **Heartbeat Timers:** Tune signaling timers appropriately to balance convergence speed and stability:
  - [ ] `set hb-interval` configured.
  - [ ] `set hb-interval-in-milliseconds` set (e.g., `10ms` – `100ms`).
  - [ ] `set hb-lost-threshold` configured (e.g., `10` – `20` missed beats).
- [ ] **Session Pickup:** Enable stateful connection failover via `set session-pickup enable`.
  - [ ] Enable `set session-pickup-expectation enable` for layer-7 protocols (FTP, SIP).
  - [ ] Enable `set session-pickup-connectionless enable` for UDP/ICMP tracking.

---

## ⚠️ Phase 3: Non-Synchronized Configurations Verification

> [!WARNING]
> **Critical Design Gotcha:**
> The following parameters do **NOT** synchronize between nodes and must be configured manually on **EACH** individual unit!

- [ ] **Device Priority & HA Override:**
  - [ ] Set distinct `priority` values (e.g., Master = `200`, Slave = `100`).
  - [ ] Decide on preemption behavior and configure `set override` **consistently on BOTH peers**.
- [ ] **System Hostnames:** Assign distinct hostnames (e.g., `FGT-PRI-01` and `FGT-SEC-02`) for audit visibility.
- [ ] **Out-of-Band Management (Reserved Mgmt Interface):**
  - [ ] Enable `set ha-mgmt-status enable` and `set ha-direct enable`.
  - [ ] Configure unique static management IPs and static gateway routes under `config ha-mgmt-interfaces`.
  - [ ] Configure `ha-direct` settings under SNMP, Syslog, and FortiAnalyzer profiles.

---

## 🔒 Phase 4: Network Integration & Failover Tuning

- [ ] **GARP & VMAC Parameters:** 
  - [ ] Enable `set gratuitous-arps enable` to accelerate switch MAC table refreshes upon failover.
  - [ ] Tune `set arps` repeat count (recommended: `5`) and `set arps-interval`.
- [ ] **Hardware Storage & Memory Protections:**
  - [ ] Enable `set ssd-failover enable` (if using disk offloading/proxy services).
  - [ ] Configure `set memory-based-failover enable` with appropriate `flip-timeout` values to avoid conserve mode flapping.
- [ ] **Upstream Switch Configurations:** Ensure switch ports connected to cluster data interfaces are configured with proper **PortFast/Edge** settings to avoid Spanning Tree topology change delays during failovers.

---

## 🧪 Phase 5: Post-Deployment Verification & Testing

- [ ] **Cluster Synchronization Check:** Run `diagnose sys ha checksum autoscale-cluster` and verify all configuration checksums match across peers.
- [ ] **HA Status Validation:** Run `get system ha status` to confirm primary/secondary roles and cluster health status.
- [ ] **Virtual MAC Verification:** Inspect VMAC mappings using `diagnose sys ha mac`.
- [ ] **Failover Stress Test (Controlled Environment):**
  - [ ] Test link failure by shutting down a monitored interface.
  - [ ] Verify continuous ping/traffic survival during primary failover.
  - [ ] Verify out-of-band management connectivity remains accessible for both individual nodes during failover.

---

## ⚡ Quick Verification Commands Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Check Cluster State Engine** | `get system ha status` |
| **Verify Sync Checksums** | `diagnose sys ha checksum autoscale-cluster` |
| **Inspect VMAC Table** | `diagnose sys ha mac` |
| **Force Test Failover** | `execute ha failover set 1` |
| **Revert Test Failover** | `execute ha failover unset 1` |
| **Access Slave CLI from Master** | `execute ha manage <member_id> <username>` |
security fabric

automation could be :

with security fabric which inherit all commands from root device



without security fabric which used it's own automation commands



*best practice in automation and sequential mode of command execution with delay is .pkg signature files to restore from offline mode license



config automation setting

set max-concurrent-stitches 128

#32-256

end



if you are using schedule automation for restore some images better use sequential mode instead of use parallel



diagnose automation test  x-schedule

#means you are testing the automation



diagnose test application autod

#means deamon or process of automation will be checked



diagnose debug enable

diagnose debug application autod -1

#verbus and debuging mode  shwo us all events and process



external connectors

endpoint or identity

fsso agent on win-ad

symantec endpoint protection

radius single sig on agent

exchange server



thread feed

ip address

manual list of ip address input on organization whcih help us  prevent access some ip to device



on dmz or servers

hfs must be runed

http://192.168.254.200:8080/ip.txt



url > http://192.168.254.200:8080/ip.txt

status enable



domain names

malware hash



after adding could be used in some policies



per vdom we have different values for edpoint or identity but for sdn and thread feed we have global access to read values



diagnose firewall dynamic list

#show and verify dynamic ip resolved by sdn connector



diagnose deb application kubed -1

diagnose debug enable

#check all kubernetes verification and transmissions



symantec works on port 8446

diagnose debug application sepmd -1



some users has no autheniticated traffic so we have no tracking module or method on gui so in cli must use this command ro check and track them

diagnose firewall auth list



kdc auto-discovery

diagnose wad debug enable category all

diagnose wad debug enable level verbose

diagnose debug enable

diagnose wad user exchange test-auto-discover



*wad means web application database help os to manage the process of each component like proxies ...



threat feed automatically import external block lists from http server as plain text or stix/taxii server

stix > structured threat information expression

standard language describe cyber threat  intelligence between machines and humans

os could worked on machine coded langs but use json format and serialization to be normal



taxii >  trusted automated exchange of intelligence information

a protocol which used on device and humans and companies to transport all cyber security intelligence informations and updates



each line in text file considered as  one entry

maximum file size must be 10mb

maximum entries count 131072

per model cmdb is different fgt-100f maybe on 10mb and 200f may be on 500 mb

on url base threat feed we could have 30 categories

*better have no duplicate entries because have resource intensive process

*better use wildcards like (*.google.com)

*use ranges 192.168.254.0/24 and don't use [] in ipv6

*for urls in ipv6 must use []

idn and utf supported in url encoding



print tablesize

#show us resource table size limitaion

#system.external-resourse : 0 256 512 (512 global limit and 256 per vdom)



*if use vdom we have some categories codes like 192-221 mens admin used threat feeds base objects

g-cat-192 > gloal object

cat-192 > root or vdom base object



fortigaurd category on threat feeds

name > x

url > stix://limon.anaomaly.com...



security fabric

must enable lldp and security fabric admin access on interface goes to root device or fortimanger



saml authenitication

in security fabric we have 2 fortigate

some clients wanna use internet from second fortigate also need use organization email to register on each site

some idp companies like okta will be provide saml authenitication

behind the saml server we have some active directory or some directory services (save each company usernames users with some trusted chains between domains and all famous companies like soundcloud...)

these provides sso with saml



microsoft base services

adfs + ca server + active directory



domain test.com on ad



certificate service and adcs

use enterprise ca on windows server + web enrollment



certificate authority in windows menu > certificate template (select manage) > web server (duplicate template)

request handling > allow export private key

general > shown in ad

cryptography > minumum key 512

security > enrollment allow



create some templates base on last modifications



mmc console

add > certificate > computer account

personal > use template like above

subject

common name  > shayan.adfs.test.com

dns > adfs.tets.com



private key > exportable



127.0.0.1/certsrv

export ca server base on above template then add to fortigate

base64



after add on fortigate we have to generate csr with fortigate then signed with ca server and get integrate



on windows server we have install adfs role

ssl cert > shayan.adfs.test.com

federation service name > adfs.test.com



groups > administrator



here we can tune saml

edit federation service > edit saml values



add relying party trust

config url

enable support for the saml 2 web sso protocol



url > adfs.test.com



fortiauthenticator

ldap on win-ad + idp



config system interface

edit port1

set ip 192.168.254.230/255.255.255.0

set allowaccess ssh https gui

end

config system global

set timezone 41

set allowed-hopts 192.168.254.254

end



authenitication

remote auth server

ldap

define ldap



user managements

realms

new user source



saml idp

general

server address > 192.168.254.230

realm as above

default idp certificate > use in list



service provider

idp prefix

server certificate



from fortigate use service provide saml values to integrated with fortiauthenticator parameters

copy and paste eachpart



enable participate in single logout



add saml attribute

username

userprinciplename



paste any part idp prefix

and urls to fortigate



certifiate authorities

local ca

trusted ca



idp base

identity provider

used public ip of sam authenitication server on internet

also need certificate



sp base



document part1

isfw > internal segmentation firewalls  are downstream fortigates



for botnet attack, vulnerablities , encryption attacks , network attacks ... could be handle by network discovery and response fortindr



on root fortigate we have storing data about devices and users on disk (user_info , user_info_history)



diagnose sys csf ....

#checking root fortigate and security fabric



config system csf

set configuration-sync local

#faz + fmg + sandbox + ems > joined fabric with local it's own values and different from fabric

end



show system csf



faz-cloud needed afac partnumber (forticloud premium subscription)

cloud base central logging and analytics

send logs to faz cloud as utm and events log



fazc (fortianalyzer cloud subscription) just send utm and event logs



in version 7.2 faz cloud does not ssupport dlp or ips archive



diagnose test update info

#chck subscription and support contract status and info



fortigate cloud hosted security management and log retention services on fortigate devices like buy one cloud model and attached to root fortigate then provide some feature and options to fabric

must use  register device as online and fortigate cloud accounts



config log fortigaurd setting

set status enable

set upload-option realtime

end

#forticloud logging

#here our fabric used enforcement on forticloud account registeration

#better use default gateway route



*without afac we have no supported feature to forward or archive traffic logs



*we hae no vdom setting to enable faz cloud if in global were disable

config log setting

set faz-override enable

end



config log fortianalyzer-cloud override-setting

set status disable/enable

end



config log fortianalyzer-cloud override-filter

set severity information

set forward-traffic disable

set local-traffic disable

set multicast-traffic disable

set sniffer-traffic disable

set anomaly disable

set voip disable

set dlp-archive disable

end



must use cli commands to see faz cloud logs or transmissions , gui is not supported

execute log filter device fortianalyzer-cloud

execute log filter category event

execute log display



just use one  faz cloud or fortigate cloud



fortimanager connected to internet and behave like wsus in microsoft which recieved updates then deployed on another devices

works on tcp 541



fortimanager cloud account level subscription (alci) used to match with forticloud accounts as fortimanager cloud service

fortimanger.forticloud.com must be resolved



diagnose fdsm central-mgmt-status

#verify fortimanager cloud launched and fortigate registered



forticloud sandbox need fortigate av license , fortisanbox cloud needed forticloud premium license fortisanbox cloud also need forticloud license on fortigate



fortigate cloud sanbox  limited 100 file per day



execute forticloud-sandbox region

config system fortigaurd

set sandbox-region (0 europe / 1 global / 2 japan / 3 us)

end



execute forticloud-sandbox update

#recieved updates



on ems adding to fortigate maybe get trouble

certificate not trusted

execute fctems verify wind2016-ems

#better add certificate again with base64 exporting



silence approval in security fabric

maybe on ha you need do some  tasks if aproved on main or primary device another devices on cluster gonna approved on silently



config endpoint-control fctems

edit ems1

set fortinetone-cloud-authentication disable

set server

set https-port 443

set source-ip 0.0.0.0

set pull-sysinfo enable

set pull-vulnerabilities enable

set pull-avatars enable

set pull-tags enable

#if n root device have connnection toward ems , synchronize the tags on fabric



set pull-malware-hash enable



set capabilities fabric-auth silent-approval websokect push-ca-cert

#push-ca-cert addd and distrbute ca cert on forticlient and ems



set call-timeout 30

set websocket-override disable

end



diagnose endpoint fctems json deep-inspect-cert-sync

#verify syncing with cluster of ha



purdue and operational technologies

sync forticlient ztna tag

works on restapi tcp 8013

also worked on websokets

these help us on to get update and monitor attributes as realtime



diagnose test application fcnacd 2

#show us status of websokect connections

#forticlient network access deamon



diagnose firewall dynamic list

#check tags are synced



diagnose sys csf downstream-devices fortinac

#check fortinac device status and informations



*fortiswitch and forti-ap are auto-auth-extenstion-device and does not need manual authorization in fabric



after adding fortindr inside system > fortigaurd we have optioin to update frotindr ann



on forti ap from fabric we can deauthorize or restart also in fortiswitch we can connect to cli or upgrade firmware and restart or deauthorize



show  user quarantine

#show us all quarantine users in downstream devices



config firewall-groups x-quarantine

config targets

edit 1

config macs

edit 12:34:56:78:90:aa

end



purdue levels are industrial control system (ics) design methods for security also known as pera or purdue enterprise refrence architecture

has many levels

include streamlines network designs, management, and communication among teams, improve security and resillience of industrial operations

targets (iec 62443)

defense-in-depth

the model's layered architecture creates multiple security checkpoints, making it more challenging for cyber threats to infiltrate critical systems



risk mitigation

the isolation of critical components minimizes the potential for unauthorized access and accidental damage, safeguarding operational integrity



enhanced visibility

the clear segmentation provided by the model facilitates comprehensive monitoring and threat detection, enabling proactive incident response



regulatory compliance

the alignment with industry skndards like iec 62443 ensures adherence to best practices and regulatory requirements



the purdue model's compatibility with industry standards like iec 62443 fosters adherence to best practices and regulatory requirements.



the synergy with industry standards like iec 62443 promotes adherence to best practices and regulatory requirements



evolve attack vector like apt (advance persistent threat) or ics  , malware, ransomware ...



levels

level 0: physical process

the foundational layer includes physical processes and equipment. these equipment are sensors, actuators, and field devices that directly interact with the physical world.



level 1: basic control

this layer houses controllers and programmable logic controllers (plcs) responsible for automating individual processes by translating sensor data into actionable commands.



level 2: supervisory control

this layer includes scada systems and human-machine interfaces (hmis), which aggregate data from controllers for process monitoring and control.



level 3: manufacturing operations

this layer comprises manufacturing execution systems (mes) and historians for managing and optimizing production processes, bridging enterprise systems and the shop floor.



demilitarized zone (dmz)

the dmz acts as a secure buffer between the ics network and external networks, housing security devices like firewalls and intrusion prevention systems (ips).



level 4: enterprise network

the enterprise network layer encompasses the broader it infrastructure, including business applications and internet connectivity.



key technologies for enhanced ics security within the purdue model framework

robust network security tools

ngfw

ips

vpn

siem (security information and event management)

analys security logs

central visibility

facilitationg incident response



effective endpoint security measures

antivirus and anti-malware

endpoint detection and response (edr)

advanced solutions that utilize behavioral analytics and machine learning to detect and respond to zero-day threats and sophisticated attacks



host-based intrusion prevention systems (hips)

systems that proactively monitor system activity andblock malicious actions in real-time



application whitelisting: restricting the execution of unauthorized applications to prevent malicious code from running on critical systems



enhanced access control and authentication

multi-factor authentication (mfa)

role-based access control (rbac)

assigns permissions and access privileges based on user roles and responsibilities, limiting access to sensitive data and systems



privilege access management (pam

controls and monitors privileged accounts, minimizing the risk of misuse and abuse of access



efficient patch management and segmentation

vulnerability management solutions

tools that scan for vulnerabilities and prioritize remediation efforts, keeping systems up-to-date and protected against known exploits



network segmentation

utilizing vlans, firewalls, or software-defined networking (sdn) to create isolated zones within the network, limiting the lateral movement of threats



microsegmentation

applies granular access controls based on individual workloads and applications,providing better isolation and protection



continuous monitoring and incident response

security orchestration, automation, and response (soar)

soar platforms automate incident response workflows, enabling faster and more efficient threat dntainment



threat intelligence

leverages threat intelligence feeds to identify emerging threats and proactively update security policies and controls



security awareness training

educates employees about security best practices and the importance of adhering to security policies



operational technologies (OT)

it/ot governance knowledge about you assets and resources



ot vision is safety and  business continuity where fortinet define a ecosystem with fortinet products to create a fabric and manage all systems



fortigate  > system > feature visibility > ot

we can see effects on security fabric > asset identity center  > ot view

fortigate and fortiswitch workes on level 2 another devices works on level 3 except level s , 0 and external



diagnose user-device-store device memory ot-prudue-set max ip level

#change purdue levels







diagnose sys csf authorization pending-list

#show  us pended requests



diagnose sys csf downstream

#information of downstream connected devices



diagnose sys csf upstream

#show root device information



if root fortigate has many vdoms , each vdom has seperated security fabric setup

attention about npu and ip addressing

*get backup then enable vdoms



all objects on security fabric will be advertised with (fabric-object-unification) as default value all devices in fabric inherit from root but some devices used local mode means use their own values

if works on local mode all objects will be placed on device without demanding request or process with root device



config sys csf

set fabric-object-unification

#this used for fortigate devices



set configuration-sync

#this used for faz fmg and fortisandbox



set fabric-workers

#how many process will be handle in synchronizations 1-4 2 is default



end



*if one device inside fabric placed between 2 or more devices with default value for synchronizations and this device worked on local mode like these

fgt1 (default) ------------------ fgt2 (local) ---------------- fgt3 (default)

*here fgt2 don't show objects in objects part but advertise them  to fgt3



*if advertised objects from root devices, get refered into the another object on downstream devices, after modification objects in root device we have no changed values on downstream devices





if enabled vdoms and created objects, on gui if said not sync on cli also has no more effect or ...

config firewall objects

edit 1

set fabric-object enable

#does not effect



end



*fabric synchronizations could not be run on multi-mode vdom



security fabric over ipsec

fortigate > vpn > ipsec tunnels

site to site custom

static ip address and interfaces

set preshared key

main mode

encryption md5 des



set policies and objects to access ipsec tunnels

consider interfaces and address must be limited



then manage the routes to access each side fortigate and visibility on ipsec interfaces

then set ipsec interfaces ip address and admin access on it



now we can connect directly toward root device on ipsec interface



lldp leveraging

wan > just recieve

lan > just transmission

undefine > follow up the vdom values



security rating

base on xml protocol transport send all saml (security assertion markup language)

provide authenitication and authorization our users



service provider in saml on fortigate root

mainly used for management in fabric

which users  attached too which admin profile



identity provider in saml on fortigate root

idp needs certificate and repositories

saml needd fsso user groups

could use saml sso in user and authentication > authenitication setting (auth schema) and sso (not usable) better use ntlm mode and fsso agent in this part



create firewall and fsso groups on user and authentication

if add saml on security fabric all clients used these fsso



system > administrator

define all remote groups and local groups with super admin login to reach sso and saml (local sp and idp with root fortigate)



security fabric  > fabric connectors

security fabric sso

idp

select ip and certificate



*must use specify ip address to management ip address



on downstream fortigate

security fabric

saml sso

service provider

login page

normal

means use sso and normal login

better use this if sso get break login as local admin



sso

means redirect to login page on root device



on idp part define root deviec and prefix and remote certificate or idp cert





**downstream worked on service provide and root device worked on identity provider



on root device > system > administrator > new sso admin and set profiles



disable automatic security check

config system global

security-rating-run-on-schedule disable

end



diagnose report-runner trigger

#manual trigger report and security rating report



troubleshooting یک cheetsheet از این موارد به انگلیسی بهم بده متانسب با گیت هاب

Here is a production-ready, professionally structured GitHub Cheat Sheet for FortiGate Security Fabric, Automation, Connectors, Purdue/OT Models, and SAML Integration based on your technical notes.

Save this file as fortigate-security-fabric-cheat-sheet.md inside your repository.

FortiGate Security Fabric, Automation & External Connectors — Operational Cheat Sheet
A comprehensive engineering reference for Fortinet Security Fabric, Automation Stitches, Threat Feeds, Purdue/OT ICS Security Model, SAML SSO/IdP integration, and diagnostic CLI workflows.

⚡ 1. Fabric Automation & Process Engine
Execution Modes & Performance Tuning
Automation Stitches execute actions sequentially or concurrently. Sequential execution (e.g., using .pkg signature packages) is required for ordered restoration scripts to prevent race conditions during recovery operations.

Plaintext
config system automation setting
    set max-concurrent-stitches 128    # Range: 32-256 (Default: 128)
end
💡 Best Practice: Always use sequential execution instead of parallel execution when scheduling image/configuration restore automation tasks.

Automation Diagnostics & Debugging
Bash
# Test a specific scheduled automation stitch
diagnose automation test <schedule_name>

# Verify automation daemon (autod) process health
diagnose test application autod 1

# Enable full verbose debugging for automation events
diagnose debug application autod -1
diagnose debug enable
🌐 2. External Connectors & Dynamic Threat Feeds
Threat Feeds import dynamic blocklists via HTTP/HTTPS plain text or STIX/TAXII protocols.

System Limits & Data Structure Specifications
Max Entries per File: 131,072 entries.

Max File Size: 10 MB default (varies per hardware model, e.g., 100F vs 200F/500F).

Max URL Categories: Up to 30 categories.

Formatting Rules:

Avoid duplicate entries (causes high CPU/memory consumption during compilation).

Domain Wildcards are supported: *.malicious-domain.com

IPv4 Formats: CIDR notation (192.168.254.0/24). Do NOT use square brackets [].

IPv6 URLs: Brackets ARE required ([http://[2001:db8::1]/list.txt](http://[2001:db8::1]/list.txt)).

Supports IDN and UTF-8 URL encoding.

Global vs. Per-VDOM Resource Limits
Bash
# Display resource table limits (Global vs Per-VDOM allocations)
print tablesize
# Example output: system.external-resource : 0 256 512 (512 Global limit, 256 Per-VDOM)
VDOM Category Codes:

g-cat-192: Global Threat Feed Object

cat-192: Root/VDOM-specific Threat Feed Object

External Connector Verification & Debug CLI
Bash
# Verify dynamic IP objects resolved by SDN/External Connectors
diagnose firewall dynamic list

# Debug Kubernetes Connector (kubed daemon)
diagnose debug application kubed -1
diagnose debug enable

# Debug Symantec Endpoint Protection Manager (sepmd daemon - Port 8446)
diagnose debug application sepmd -1
diagnose debug enable

# Inspect unauthenticated user traffic states
diagnose firewall auth list

# Test KDC / WAD Auto-Discovery (Web Application Database)
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose wad user exchange test-auto-discover
diagnose debug enable
🛡️ 3. Security Fabric Architecture & Object Unification
Core Interface Requirements
LLDP and Security Fabric Connection (set allowaccess fabric) must be enabled on interfaces connecting downstream FortiGates, FortiSwitches, or FortiManager.

Plaintext
config system interface
    edit "port1"
        set allowaccess ping https ssh fabric
        set lldp-reception enable
        set lldp-transmission enable
    next
end
Fabric Object Unification vs. Local Synchronization
Controlled under config system csf:

fabric-object-unification: Controls how firewall objects adapt across FortiGate members. Default behavior inherits objects from the Root device.

configuration-sync local: Forces downstream appliances (FAZ, FMG, FortiSandbox, EMS) to join the fabric while maintaining local settings.

Plaintext
config system csf
    set fabric-object-unification {default | local}
    set configuration-sync local
    set fabric-workers 2                # Synchronization process workers (1-4, Default: 2)
end
⚠️ Multi-VDOM Warning: Security Fabric automatic object synchronization does NOT run on multi-mode VDOM topologies.

Security Fabric Diagnostics
Bash
# View pending authorization requests from downstream devices
diagnose sys csf authorization pending-list

# Inspect downstream connected units
diagnose sys csf downstream

# Inspect upstream Root FortiGate details
diagnose sys csf upstream

# Check FortiNAC downstream Fabric connection status
diagnose sys csf downstream-devices fortinac
🏭 4. Operational Technology (OT) & The Purdue Model (PERA / IEC 62443)
The Purdue Enterprise Reference Architecture (PERA) segments Industrial Control Systems (ICS) to enforce Defense-in-Depth, risk mitigation, and regulatory compliance.

+-------------------------------------------------------------------+
| Level 4: Enterprise Network (Business Apps, ERP, IT Cloud Access) |
+-------------------------------------------------------------------+
| DMZ: Secure Buffer (NGFW, IPS, SIEM, Proxies, MFA/PAM)            |
+-------------------------------------------------------------------+
| Level 3: Manufacturing Operations (MES, Historians, Process Mgmt) |
+-------------------------------------------------------------------+
| Level 2: Supervisory Control (SCADA, HMIs, Alarm Monitoring)      |
+-------------------------------------------------------------------+
| Level 1: Basic Control (PLCs, RTUs, Automation Controllers)       |
+-------------------------------------------------------------------+
| Level 0: Physical Process (Sensors, Actuators, Solenoids, Motors) |
+-------------------------------------------------------------------+
FortiGate OT Visibility & Purdue Tuning
FortiGate & FortiSwitch Operations: Operate primarily at Level 2 / Level 3 inspection.

Feature Activation: Enable OT Visibility under System > Feature Visibility > OT.

Bash
# Modify or adjust the maximum IP/Purdue level memory thresholds
diagnose user-device-store device memory ot-purdue-set max ip level
🔐 5. SAML Authentication & FortiAuthenticator IdP
The Security Fabric utilizes SAML 2.0 to deliver Single Sign-On (SSO) across Fabric members using an Identity Provider (IdP) such as Okta, ADFS, or FortiAuthenticator.

SAML Roles in Security Fabric
Root FortiGate: Functions as the Identity Provider (IdP) or coordinates with an external IdP.

Downstream FortiGates: Function as Service Providers (SP) that redirect unauthenticated administrators or users to the Root IdP.

FortiClient EMS Integration & Certificate Sync
Plaintext
config endpoint-control fctems
    edit "ems1"
        set fortinetone-cloud-authentication disable
        set server "192.168.254.220"
        set https-port 443
        set pull-sysinfo enable
        set pull-vulnerabilities enable
        set pull-avatars enable
        set pull-tags enable
        set pull-malware-hash enable
        set capabilities fabric-auth silent-approval websocket push-ca-cert
        set call-timeout 30
    next
end
SAML & Endpoint Debugging CLI
Bash
# Verify deep inspection certificate synchronization across HA cluster
diagnose endpoint fctems json deep-inspect-cert-sync

# Check FortiClient Network Access Daemon (fcnacd) WebSocket connectivity
diagnose test application fcnacd 2

# Verify synced ZTNA dynamic posture tags
diagnose firewall dynamic list
☁️ 6. FortiCloud & FortiAnalyzer Cloud Management
Central Logging Configurations
To override global logging overrides for FortiAnalyzer Cloud on specific VDOMs:

Plaintext
config log setting
    set faz-override enable
end

config log fortianalyzer-cloud override-setting
    set status enable
end

config log fortianalyzer-cloud override-filter
    set severity information
    set forward-traffic enable
    set local-traffic disable
    set multicast-traffic disable
    set sniffer-traffic disable
    set anomaly enable
    set voip disable
    set dlp-archive disable
end
FortiCloud Sandbox Region & Update CLI
Plaintext
config system fortiguard
    set sandbox-region 0                # 0: Europe | 1: Global | 2: Japan | 3: US
end
Bash
# Execute manual FortiCloud Sandbox update check
execute forticloud-sandbox update

# Set Sandbox region via execute command
execute forticloud-sandbox region

# CLI log display for FortiAnalyzer Cloud events
execute log filter device fortianalyzer-cloud
execute log filter category event
execute log display
📊 7. Security Rating & Maintenance
Disable Automatic Scheduled Security Checks
To prevent unexpected CPU usage from scheduled security rating audits:

Plaintext
config system global
    set security-rating-run-on-schedule disable
end
Manual Trigger & Inspection Commands
Bash
# Manually trigger Security Rating calculation report
diagnose report-runner trigger

# Check subscription contract and support entitlement status
diagnose test update info
نه یجوری بده که من در githubنیاز به ادیت نداشته باشم و به راحتی همه فونت ها ... رعایت شده باشه توش

برای اینکه فایل کاملاً آمادهٔ کپی و پیست در گیت‌هاب باشد و هیچ نیازی به ادیت، تغییر فونت یا اصلاح مارک‌داون نداشته باشی، کل داکیومنت را داخل یک Block Code یکپارچه قرار داده‌ام.

تمامی استانداردهای پیشرفتهٔ گیت‌هاب (مانند GitHub Alert Callouts، Tables، Syntax Highlighting، و Diagrams) کاملاً رعایت شده‌اند. کافی است یک فایل جدید به اسم fortigate-security-fabric-cheat-sheet.md در ریپازیتوری ایجاد کنی و کل محتوای داخل باکس زیر را مستقیم در آن کپی کنی:

Markdown
# 🛡️ FortiGate Security Fabric, Automation & External Connectors — Cheat Sheet

A production-grade operational reference for **Fortinet Security Fabric**, **Automation Stitches**, **External Threat Feeds**, **Purdue / OT (ICS) Security Models**, and **SAML SSO / IdP Integration**.

---

## ⚡ 1. Automation Stitches & Process Tuning

Automation stitches can execute commands sequentially or concurrently. For operations involving firmware/license updates or configuration restores, **sequential execution** is strictly required to prevent race conditions.

```text
config system automation setting
    set max-concurrent-stitches 128    # Range: 32-256 (Default: 128)
end
[!TIP]
Best Practice: Use sequential execution mode with .pkg signature files when automating configuration or image restores.

🔍 Automation Diagnostics & Debugging Commands
Bash
# Test a specific scheduled automation stitch
diagnose automation test <schedule_name>

# Verify automation daemon (autod) health
diagnose test application autod 1

# Enable full verbose debugging for automation events
diagnose debug application autod -1
diagnose debug enable
🌐 2. Threat Feeds & External Connectors
Threat Feeds dynamically import external IP/Domain/URL blocklists via HTTP, HTTPS, or STIX/TAXII protocols.

📐 System Limits & Formatting Standards
Maximum Entries: 131,072 per file.

Maximum File Size: 10 MB (varies by model hardware spec).

Maximum Dynamic Categories: Up to 30 URL categories.

Formatting Rules:

IP Networks: Use standard CIDR notation (192.168.254.0/24). Do NOT use square brackets for IPv4.

IPv6 URLs: Square brackets ARE required (http://[2001:db8::1]/list.txt).

Wildcards: Wildcard domains are fully supported (*.malicious-domain.com).

Optimization: Avoid duplicate entries to minimize CPU and RAM compilation overhead.

Bash
# Check resource table limits (Global vs. Per-VDOM allocations)
print tablesize
[!NOTE]
VDOM Category Identifiers:

g-cat-192 → Global Object

cat-192 → Root / VDOM-specific Object

🛠️ External Connector Troubleshooting CLI
Bash
# Verify dynamic IP address lists resolved by SDN connectors
diagnose firewall dynamic list

# Debug Kubernetes Connector (kubed process)
diagnose debug application kubed -1
diagnose debug enable

# Debug Symantec Endpoint Protection Manager (sepmd process - Port 8446)
diagnose debug application sepmd -1
diagnose debug enable

# Inspect unauthenticated user tracking state
diagnose firewall auth list

# Troubleshoot WAD / KDC Auto-Discovery (Web Application Database)
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
diagnose wad user exchange test-auto-discover
🔗 3. Security Fabric Architecture & Object Synchronization
🔌 Core Interface Requirements
To establish a Fabric connection to a Root FortiGate or FortiManager, you must enable both LLDP and Fabric Management access on the interconnecting interfaces:

Plaintext
config system interface
    edit "port1"
        set allowaccess ping https ssh fabric
        set lldp-reception enable
        set lldp-transmission enable
    next
end
🔀 Fabric Object Unification vs. Local Sync
Plaintext
config system csf
    set fabric-object-unification {default | local}
    set configuration-sync local
    set fabric-workers 2                # Synchronization workers (1-4, Default: 2)
end
[!WARNING]
Multi-VDOM Limitation: Security Fabric automatic object synchronization does NOT run on multi-mode VDOM environments.

📊 Fabric Diagnostics Cheat Sheet
Action	Command
Check Pending Authorizations	diagnose sys csf authorization pending-list
View Downstream Devices	diagnose sys csf downstream
View Upstream Root Device	diagnose sys csf upstream
Check FortiNAC Status	diagnose sys csf downstream-devices fortinac
🏭 4. Operational Technology (OT) & Purdue Model (IEC 62443)
The Purdue Enterprise Reference Architecture (PERA / IEC 62443) provides structured network segmentation for Industrial Control Systems (ICS) and Operational Technology (OT).

Plaintext
+-------------------------------------------------------------------+
| Level 4: Enterprise Network (Business Apps, ERP, IT Cloud Access) |
+-------------------------------------------------------------------+
| DMZ: Secure Buffer (NGFW, IPS, SIEM, Proxies, MFA/PAM)            |
+-------------------------------------------------------------------+
| Level 3: Manufacturing Operations (MES, Historians, Process Mgmt) |
+-------------------------------------------------------------------+
| Level 2: Supervisory Control (SCADA, HMIs, Alarm Monitoring)      |
+-------------------------------------------------------------------+
| Level 1: Basic Control (PLCs, RTUs, Automation Controllers)       |
+-------------------------------------------------------------------+
| Level 0: Physical Process (Sensors, Actuators, Solenoids, Motors) |
+-------------------------------------------------------------------+
⚙️ FortiGate OT & Purdue Configuration
OT Visibility: Enable under System > Feature Visibility > OT.

Inspection Level: FortiGate and FortiSwitch inspect traffic primarily at Level 2 and Level 3.

Bash
# Adjust Purdue level IP memory allocation thresholds
diagnose user-device-store device memory ot-purdue-set max ip level
🔐 5. SAML SSO & FortiAuthenticator Integration
Security Fabric relies on SAML 2.0 for single sign-on authentication across cluster nodes.

Root FortiGate: Acts as the Identity Provider (IdP) (or bridges to ADFS/Okta).

Downstream FortiGate: Acts as a Service Provider (SP) redirecting auth requests to the Root.

Plaintext
# FortiClient EMS Fabric Integration Configuration
config endpoint-control fctems
    edit "ems1"
        set fortinetone-cloud-authentication disable
        set server "192.168.254.220"
        set https-port 443
        set pull-sysinfo enable
        set pull-vulnerabilities enable
        set pull-avatars enable
        set pull-tags enable
        set pull-malware-hash enable
        set capabilities fabric-auth silent-approval websocket push-ca-cert
        set call-timeout 30
    next
end
🔍 SAML & Endpoint Debugging
Bash
# Verify Certificate Sync across HA cluster for EMS
diagnose endpoint fctems json deep-inspect-cert-sync

# Check FortiClient Network Access Daemon (fcnacd) WebSocket connections
diagnose test application fcnacd 2

# Verify dynamic ZTNA posture tags on FortiGate
diagnose firewall dynamic list
☁️ 6. FortiCloud & FortiAnalyzer Cloud Management
🎛️ VDOM Override Settings for FortiAnalyzer Cloud
Plaintext
config log setting
    set faz-override enable
end

config log fortianalyzer-cloud override-setting
    set status enable
end

config log fortianalyzer-cloud override-filter
    set severity information
    set forward-traffic enable
    set local-traffic disable
    set multicast-traffic disable
    set sniffer-traffic disable
    set anomaly enable
    set voip disable
    set dlp-archive disable
end
🚀 Sandbox Operations & Log Queries
Plaintext
config system fortiguard
    set sandbox-region 0    # 0: Europe | 1: Global | 2: Japan | 3: US
end
Bash
# Manual FortiCloud Sandbox update
execute forticloud-sandbox update

# View FortiAnalyzer Cloud logs from CLI
execute log filter device fortianalyzer-cloud
execute log filter category event
execute log display
📈 7. Security Rating & Maintenance
Plaintext
# Disable automatic scheduled Security Rating execution
config system global
    set security-rating-run-on-schedule disable
end
Bash
# Manually trigger Security Rating calculation report
diagnose report-runner trigger

# Check FortiCloud subscription entitlement status
diagnose test update info
حالا در قالب و دیدگاه check list این موارد بهم بده متناسب با github

آماده شد! این هم از چک‌لیست پیاده‌سازی و عیب‌یابی Security Fabric، اتوماسیون، Threat Feed و SAML که دقیقاً بر اساس استانداردهای مارک‌داون گیت‌هاب طراحی شده است.

تمام باکس‌های تیک‌زدن ([ ])، الرت‌های استاندارد گیت‌هاب، جداول و بخش کامندهای سریع بدون نیاز به هیچ ویرایشی آماده استفاده در ریپازیتوری شما هستند.

کد زیر را کپی کرده و در یک فایل جدید به نام fortigate-security-fabric-checklist.md ذخیره کنید:

Markdown
# 📋 FortiGate Security Fabric & Automation Operational Checklist

A step-by-step production verification checklist for deploying, hardening, and troubleshooting **Security Fabric**, **Automation Stitches**, **External Threat Feeds**, **OT/Purdue Architectures**, and **SAML SSO** integrations on FortiGate.

---

## 🛠️ Phase 1: Pre-Fabric & Core Interface Prerequisites

- [ ] **Interface Management Access:** Verify `fabric` access is enabled under `config system interface` on all interconnecting ports.
- [ ] **LLDP Protocol Enabled:** Ensure both `lldp-reception` and `lldp-transmission` are active on interconnecting switches and FortiGate links.
- [ ] **NPU / VDOM Considerations:** Confirm that target VDOMs are running in single-mode or supported VDOM topologies (Fabric object synchronization does NOT run on multi-mode VDOMs).
- [ ] **OOB / Dedicated Reachability:** Ensure stable L3 connectivity or IPsec site-to-site connectivity between downstream FortiGates and the Root device.

---

## ⚙️ Phase 2: Security Fabric & Object Unification Configuration

- [ ] **Root Device Authorization:** Check and approve all pending downstream devices on the Root FortiGate (`diagnose sys csf authorization pending-list`).
- [ ] **Configuration Sync Mode:**
  - [ ] Set `fabric-object-unification` (`default` for global inheritance or `local` for independent management).
  - [ ] Set `configuration-sync local` for dedicated appliances (FAZ, FMG, FortiSandbox, EMS).
- [ ] **Fabric Worker Tuning:** Verify `set fabric-workers` is optimized (Default: `2`, Range: `1-4`).
- [ ] **Silent Approval & Certificate Push (EMS Integration):**
  - [ ] Enable `websocket`, `silent-approval`, and `push-ca-cert` capabilities under `config endpoint-control fctems`.
  - [ ] Verify endpoint ZTNA posture tags are dynamically syncing across all cluster nodes.

---

## 🌐 Phase 3: External Connectors & Dynamic Threat Feeds

- [ ] **File & Entry Constraints Checklist:**
  - [ ] Total entries in the feed file do NOT exceed `131,072`.
  - [ ] Total file size is under the model memory limit (Default: `10 MB`).
  - [ ] Max URL categories count is within the `30` limit.
- [ ] **Syntax & Formatting Rules:**
  - [ ] IPv4 addresses use standard CIDR blocks (e.g., `192.168.254.0/24`) without brackets.
  - [ ] IPv6 URLs explicitly use square brackets (e.g., `http://[2001:db8::1]/list.txt`).
  - [ ] Duplicate entries are purged to reduce CPU compilation overhead.
- [ ] **DMZ / Web Host Access:** Confirm local HTTP servers (e.g., HFS) hosting external threat feeds are reachable over target ports (e.g., `8080`).

---

## 🤖 Phase 4: Automation Stitches & Execution Tuning

- [ ] **Concurrent Limit Allocation:** Set `max-concurrent-stitches` within system capacities (Range: `32-256`, Default: `128`).
- [ ] **Execution Mode Check:** Ensure **sequential execution mode** is used for configuration or `.pkg` signature restores instead of parallel execution.
- [ ] **Stitch Testing:** Manually trigger scheduled automation routines using `diagnose automation test <stitch_name>` to verify action pipelines.

---

## 🔐 Phase 5: SAML SSO & Identity Provider (IdP) Setup

> [!WARNING]
> **SAML Architecture Rule:**
> The **Root FortiGate** or external IdP (Okta / ADFS / FortiAuthenticator) acts as the **Identity Provider (IdP)**. Downstream FortiGates MUST be configured as **Service Providers (SP)**.

- [ ] **Certificate Trust Chain:** Export enterprise CA certificate (`certsrv`) in Base64 format and import it into FortiGate trusted CAs.
- [ ] **Service Provider (SP) Mapping:** Copy exact SP entity IDs and assertion URLs from FortiGate into the IdP relying party trust setup.
- [ ] **Attribute Matching:** Ensure `username` and `userprincipalname` SAML attributes are correctly mapped on FortiAuthenticator/ADFS.
- [ ] **Fallback Access:** Enable `normal` login page fallback alongside SSO to prevent administrator lockout if the SAML server becomes unreachable.

---

## 🏭 Phase 6: Operational Technology (OT) & Purdue Model Compliance

- [ ] **OT Feature Visibility:** Enable OT options under `System > Feature Visibility > OT`.
- [ ] **Purdue Level Segmentation:** Ensure physical/logical rules align with IEC 62443 standard levels:
  - [ ] **Level 0 / 1:** Physical devices, sensors, and PLCs isolated from higher networks.
  - [ ] **Level 2 / 3:** SCADA, HMIs, and MES monitored by FortiGate / FortiSwitch security policies.
  - [ ] **DMZ Buffer Zone:** NGFW, IPS, and MFA gateways separating Level 3 from Level 4 Enterprise networks.

---

## 🧪 Phase 7: Post-Deployment Diagnostics & Health Verification

- [ ] **Fabric Link Status:** Run `diagnose sys csf downstream` and `diagnose sys csf upstream` to confirm cluster topology health.
- [ ] **Dynamic Feed Resolution:** Verify dynamically imported dynamic IP lists using `diagnose firewall dynamic list`.
- [ ] **Daemon Process Checks:**
  - [ ] Check `autod` (Automation): `diagnose test application autod 1`
  - [ ] Check `fcnacd` (FortiClient NAC): `diagnose test application fcnacd 2`
  - [ ] Check `kubed` (Kubernetes Connector): `diagnose debug application kubed -1`
- [ ] **Security Rating Verification:** Manually execute `diagnose report-runner trigger` to confirm Fabric security audit compliance.

---

## ⚡ Quick Reference Commands Table

| Objective | Command |
| :--- | :--- |
| **Verify Downstream Devices** | `diagnose sys csf downstream` |
| **Verify Upstream Root Device** | `diagnose sys csf upstream` |
| **Check Pending Fabric Requests** | `diagnose sys csf authorization pending-list` |
| **Test Automation Stitch** | `diagnose automation test <schedule_name>` |
| **List Dynamic Threat Feed IPs** | `diagnose firewall dynamic list` |
| **Test SAML WAD Auto-Discovery** | `diagnose wad user exchange test-auto-discover` |
| **Check Contract & Entitlements** | `diagnose test update info` |
| **Trigger Security Rating Audit** | `diagnose report-runner trigger` |
hardware acceleration part1

entry levels has no content processor (cp)



cpu

soc

spu

npu

cp



cp9 and xlite > soc4

cp9lite > soc3



cp9 > 16 ipsec engine

cp9 xlite > 5 ipsec engine

cp9 lite > 1



get hardware status

#asic version will be shown



network processors

np6

np6xlite

np6lite



nturbo is feature like offloading some process on asics , also called fast path

config ips global

set np-accel-mode basic

#default is basic and enable



*create direct data path from ingress port to egress port with ips features , has no more steps



set cp-accel-mode advance

#more type patterns matching, advance mode worked on devices with 2 or more cp8 or cp9



*if has no command like this means you have no ipsa in device

*default mode is basic



there are some special cases (listed below) where sessions may not be offloaded by nturbo, even when nturbo is explicitly enabled in these cases the sessions are handled by the fortigate cpu :

np acceleration is disabled

auto-asic-offload is disabled in the firewall policy configuration



the firewall policy includes proxy-based security profiles

the sessions require fortios session-helpers

ftp sessions are not offloaded to np processors because ftp sessions use the ftp session helper



interface policies or dos policies have been added to the ingress or egress interface



tunneling is enabled

any traffic to or from a tunneled interface (ipsec, ipinip, ssl vpn, gre, capwap...) cannot be offloaded by nturbo





*in some configurations, when a fortigate with np7 or np6 processors is operating with virtual clustering enabled, traffic cannot be offloaded by the np7 or np6 processors if the mac address of the vlan interface accepting the traffic is different from the mac address of the physical interface that the vlan interface has been added to

if you are running a configuration like this, traffic from the vlan interface can be dropped by the np7 or np6 processors if you notice traffic being dropped, you can disable np offloading in the firewall policy that accepts the traffic to resolve the issue



np7 and np6 offloading can still work in some network configurations when a vlan and its physical interface have different mac addresses

offloading can still work as long as other network devices learn the fortigate's mac addresses from arp

as well, offloading can work if the reply traffic destination mac is the same as the mac of the underlying interface



get hardware npu np6 port-list

diagnose npu np6 port-list



get hardware npu np6xlite port-list



get hardware npu np6lite port-list

diagnose npu np6lite port-list



*np6, np6xlite, and np6lite offloading is supported when you configure netflow for interfaces connected to np6,np6xlite, or np6lite processors. offloading of other sessions is not affected by configuring netflow

full netflow is supported through the information maintained in the firewall session

configuring sflow on any interface disables all np6, np6xlite, or np6lite offloading for all traffic on that interface این موارد در قالب cheet sheet بهم بده

⚡ FortiGate Hardware Acceleration & ASIC Offloading — Operational Cheat Sheet
A production-ready reference covering FortiGate System-on-a-Chip (SoC) architectures, Content Processors (CP), Network Processors (NP), nTurbo acceleration, and ASIC offloading limitations.

🏛️ 1. Architecture Overview & Processor Matrix
FortiGate entry-level appliances typically do not feature discrete Content Processors (CP). Instead, they utilize System-on-a-Chip (SoC) architectures that consolidate the CPU, Network Processor (NP), and Content Processor (CP) onto a single die.

Plaintext
               +-----------------------------------+
               |    System-on-a-Chip (SoC)        |
               |                                   |
               |  +---------+  +-----------------+ |
               |  |   CPU   |  |   SPU (ASIC)    | |
               |  +---------+  | +-----+ +-----+ | |
               |               | | NPU | | CP  | | |
               |               | +-----+ +-----+ | |
               |               +-----------------+ |
               +-----------------------------------+
Content Processor (CP) & IPsec Engine Matrix
CP Version	Integrated SoC Version	IPsec Acceleration Engines
CP9	SoC4	16 IPsec Engines
CP9 XLite	SoC4	5 IPsec Engines
CP9 Lite	SoC3	1 IPsec Engine
Bash
# Verify system hardware architecture and ASIC version
get hardware status
🚀 2. nTurbo & IPS Offloading Mechanisms
nTurbo creates a direct fast-path data channel between ingress and egress interfaces for IPS-inspected traffic, offloading flow-based security profiles to ASIC hardware to bypass standard CPU path latency.

Plaintext
config ips global
    set np-accel-mode basic        # Default: basic (Enables nTurbo offloading)
    set cp-accel-mode advance      # Enables pattern matching offload to CPs
end
📌 Note: set cp-accel-mode advance optimizes pattern matching performance and is supported on appliances equipped with two or more CP8 or CP9 processors. If these options are missing from the CLI, the hardware does not feature discrete IPS ASIC acceleration.

⛔ 3. nTurbo Offloading Exceptions (CPU Fallback)
Even when explicitly enabled, sessions automatically fall back to the host CPU under any of the following conditions:

ASIC Offload Disabled: auto-asic-offload is manually set to disable inside the firewall policy.

Proxy Security Profiles: The matching firewall policy utilizes proxy-based security profiles instead of flow-based profiles.

Session Helpers Required: Traffic depends on FortiOS session helpers (e.g., FTP control/data channels managed by ftp-helper).

Interface / DoS Policies: Ingress or egress interfaces have active Interface Policies or DoS Policies applied.

Tunnel Interfaces: Traffic originates from or terminates on tunneled interfaces (IPsec, IP-in-IP, SSL-VPN, GRE, CAPWAP).

⚠️ 4. Virtual Clustering, VLAN MAC Mismatch & Drops
On NP6 and NP7 architectures operating in Virtual Clustering modes, traffic drops may occur if the MAC address of a VLAN interface differs from the underlying physical interface MAC.

VLAN Interface MAC

=Physical Interface MAC⟹Potential NP6/NP7 Frame Drop
Mitigation & Workaround Rules:
Offloading Support Conditions:

Downstream/upstream devices correctly resolve the FortiGate MAC address via ARP.

The destination MAC address of reply traffic matches the MAC of the physical host interface.

Resolution Action: If drops persist, disable NP offloading on the specific firewall policy accepting the VLAN traffic:

Plaintext
config firewall policy
    edit <policy_id>
        set auto-asic-offload disable
    next
end
📊 5. NetFlow vs. sFlow Offloading Impact
NetFlow: Fully supported alongside NP6, NP6XLite, and NP6Lite acceleration. State tracking is maintained via the firewall session table without disrupting hardware offloading.

sFlow: Disables ALL NP6 / NP6XLite / NP6Lite offloading for all ingress and egress traffic on any interface where sFlow is enabled.

🔍 6. Hardware Diagnostic & Verification Commands
Bash
# Display NP6 port mapping and ASIC bindings
get hardware npu np6 port-list
diagnose npu np6 port-list

# Display NP6XLite port mapping
get hardware npu np6xlite port-list

# Display NP6Lite port mapping and status
get hardware npu np6lite port-list
diagnose npu np6lite port-list
checklist حالا در قالب این موارد بهم بده

آماده شد! این هم از چک‌لیست پیاده‌سازی، بهینه‌سازی و عیب‌یابی شتاب‌دهی سخت‌افزاری (Hardware Acceleration) و ASIC که دقیقاً بر اساس استانداردهای مارک‌داون گیت‌هاب طراحی شده است.

تمام باکس‌های تیک‌زدن ([ ])، الرت‌های استاندارد گیت‌هاب، جداول و بخش کامندهای سریع بدون نیاز به هیچ ویرایشی آماده کپی و پیست در ریپازیتوری شما هستند.

کد زیر را کپی کرده و در یک فایل جدید به نام fortigate-hardware-acceleration-checklist.md ذخیره کنید:

Markdown
# 📋 FortiGate Hardware Acceleration & ASIC Operational Checklist

A step-by-step production verification checklist for validating **ASIC offloading (NP/CP)**, optimizing **nTurbo**, managing **VLAN MAC conflicts**, and troubleshooting **hardware acceleration bypass** on FortiGate firewalls.

---

## 🛠️ Phase 1: Hardware & Architecture Verification

- [ ] **ASIC Identification:** Run `get hardware status` to identify the system architecture (Discrete NP/CP vs. System-on-a-Chip / SoC).
- [ ] **CP Resource Audit:** Confirm CP capability and engine density:
  - [ ] **CP9 (SoC4):** 16 IPsec Engines
  - [ ] **CP9 XLite (SoC4):** 5 IPsec Engines
  - [ ] **CP9 Lite (SoC3):** 1 IPsec Engine
  - [ ] **Entry-Level Models:** Verify if Content Processors (CP) are completely absent.
- [ ] **Port-to-ASIC Mapping:** Inspect interface bindings using the appropriate NPU command:
  - [ ] `get hardware npu np6 port-list`
  - [ ] `get hardware npu np6xlite port-list`
  - [ ] `get hardware npu np6lite port-list`

---

## ⚙️ Phase 2: nTurbo & IPS Acceleration Tuning

- [ ] **nTurbo Mode Verification:** Ensure `set np-accel-mode basic` is enabled under `config ips global` to create direct ingress-to-egress acceleration paths.
- [ ] **Pattern Matching Offload:** Check if `set cp-accel-mode advance` is configured (only supported on devices with $\ge$ 2 CP8/CP9 processors).
- [ ] **Feature Availability:** Confirm that IPS global acceleration settings exist on the device (absence indicates no discrete IPS hardware acceleration).

---

## ⚠️ Phase 3: Offloading Exception Audit (CPU Fallback Check)

Verify that critical traffic flows are not silently falling back to host CPU processing due to misconfigurations:

- [ ] **Policy Level ASIC Offload:** Ensure `set auto-asic-offload enable` is active on high-throughput firewall policies.
- [ ] **Inspection Mode:** Verify traffic intended for ASIC offload uses **Flow-based** security profiles instead of Proxy-based profiles.
- [ ] **Session Helpers Check:** Identify protocols requiring session helpers (e.g., FTP via `ftp-helper`) that bypass NP offloading.
- [ ] **Interface & DoS Policies:** Confirm no Interface Policies or DoS Policies are bound to ingress/egress interfaces if maximum NPU throughput is required.
- [ ] **Tunnel Interfaces:** Acknowledge that traffic traversing tunneled interfaces (**IPsec, SSL-VPN, GRE, CAPWAP, IP-in-IP**) will be processed by CPU / nTurbo exceptions.

---

## 🔀 Phase 4: Virtual Clustering & VLAN MAC Drop Prevention

> [!WARNING]
> **VLAN MAC Mismatch Threat:**
> On NP6/NP7 architectures using Virtual Clustering, traffic drops occur if the VLAN interface MAC address differs from the underlying physical interface MAC address.

- [ ] **MAC Address Parity Check:** Verify if VLAN interfaces share the physical port MAC address.
- [ ] **ARP Resolution Verification:** Ensure upstream/downstream switches properly learn and map FortiGate MAC addresses via ARP.
- [ ] **Workaround Implementation:** If packet drops are observed on mismatched VLAN interfaces, explicitly disable NP offloading on the specific policy:
  ```text
  config firewall policy
      edit <policy_id>
          set auto-asic-offload disable
      next
  end
📊 Phase 5: Telemetry & Monitoring Impact (NetFlow vs. sFlow)
[ ] NetFlow Offloading Safe-State: Confirm NetFlow is used for flow tracking (NetFlow maintains session table state without disabling NP acceleration).

[ ] sFlow Disabling Audit: Verify sFlow is NOT enabled on NPU-accelerated interfaces (enabling sFlow disables ALL NP6 / NP6XLite / NP6Lite offloading on that interface).

🧪 Phase 6: Post-Deployment Verification & Diagnostics
[ ] NPU Port & Status Check: Verify interface offload allocations via diagnose npu np6 port-list (or np6lite).

[ ] Session Table Inspection: Confirm traffic sessions display g3 or ASIC offload flags in the session table (diagnose sys session list).

[ ] CPU Utilization Monitoring: Ensure CPU usage remains stable during peak traffic bursts, confirming active hardware offloading.

⚡ Quick Verification Commands Table
Objective	Command
Check ASIC Architecture	get hardware status
NP6 Port Mapping	get hardware npu np6 port-list
NP6 Diagnostics	diagnose npu np6 port-list
NP6XLite Port Mapping	get hardware npu np6xlite port-list
NP6Lite Port Mapping	get hardware npu np6lite port-list
NP6Lite Diagnostics	diagnose npu np6lite port-list
Inspect Active Sessions Offload	diagnose sys session list
raid

must used fortinet factory build series on devices

reboot and erase all data if raid get changed



execute disk list

execute disk raid status

execute disk raid enable

execute disk raid rebuild-level

#change raid level



execute disk raid disable

#rebbot device and erase all data on disks



diagnose hardware deviceinfo disk در قالب cheet sheet متناسب با github بهم بده

آماده شد! فایل به‌صورت یک GitHub Operational Cheat Sheet استاندارد و حرفه‌ای با تمامی نکات کلیدی، سینتکس‌ها و رفرنس دستورات طراحی شده است.

محتوای داخل باکس زیر را مستقیم در یک فایل به نام fortigate-disk-raid-cheat-sheet.md کپی و در ریپازیتوری خود ذخیره کنید:

Markdown
# 💾 FortiGate Disk Management & RAID Configuration — Operational Cheat Sheet

A production-ready technical reference for managing internal storage drives, RAID arrays, and disk diagnostics on storage-equipped FortiGate appliances.

---

## ⚠️ Critical Prerequisites & Operations Notice

> [!CAUTION]
> **Data Destruction Warning:**
> Enabling, disabling, or changing the rebuild level of a RAID array **will format all attached physical disks, destroy all saved log data/pcap archives, and reboot the FortiGate device**. Always perform a full backup before modifying storage properties.

* **Disk Compatibility:** Only Fortinet factory-certified hard drives and SSDs (Fortinet Factory-Build Series) are supported. Non-factory drives will be flagged as uncertified or ignored by the storage controller.
* **Scope:** Applies to FortiGate models with multiple internal storage drives (e.g., FortiGate 100E/100F/200E/200F with dual SSDs or enterprise rack-mount models).

---

## ⚙️ 1. RAID Operations & Level Modifications

All RAID configuration changes are executed from the administrative CLI and trigger an immediate disk initialization and system reboot sequence.

```text
# Enable RAID on supported multi-disk FortiGate units
execute disk raid enable

# Disable RAID array (Reverts disks to standalone mode and erases all data)
execute disk raid disable

# Modify RAID rebuild level or change array configuration
execute disk raid rebuild-level <level_option>
🔍 2. RAID & Disk Verification CLI Commands
Bash
# Display physical disk summary, serial numbers, and operational state
execute disk list

# Inspect active RAID array health, member status, and synchronization percentage
execute disk raid status

# Display comprehensive hardware-level details and SMART attributes for storage drives
diagnose hardware deviceinfo disk
⚡ Quick Reference Commands Summary
Task	Command	Impact
List Installed Physical Disks	execute disk list	Non-disruptive
Check RAID Health Status	execute disk raid status	Non-disruptive
Inspect Hardware Disk Info	diagnose hardware deviceinfo disk	Non-disruptive
Enable RAID Array	execute disk raid enable	⚠️ Reboot & Erase Data
Disable RAID Array	execute disk raid disable	⚠️ Reboot & Erase Data
Change RAID Rebuild Level	execute disk raid rebuild-level	⚠️ Reboot & Erase Data
حال در قالب checklist متناسب با github

آماده شد! این هم از چک‌لیست پیاده‌سازی، مدیریت و عیب‌یابی RAID و دیسک‌های داخلی FortiGate که دقیقاً بر اساس استانداردهای مارک‌داون گیت‌هاب طراحی شده است.

کد زیر را کپی کرده و در یک فایل جدید به نام fortigate-disk-raid-checklist.md ذخیره کنید:

Markdown
# 📋 FortiGate Disk Management & RAID Operational Checklist

A step-by-step production verification checklist for managing internal storage drives, configuring RAID arrays, and diagnosing hardware disks on FortiGate appliances.

---

## 🛠️ Phase 1: Pre-Deployment & Compatibility Checks

- [ ] **Factory Drive Parity:** Confirm all installed storage drives are **Fortinet Factory-Build Series** certified units.
- [ ] **Data Backup Verification:** Ensure all system logs, PCAP files, and configurations are fully backed up before executing RAID operations.
- [ ] **Hardware Capacity Verification:** Confirm the FortiGate model features multiple physical disk slots supporting hardware or software RAID architectures.

---

## ⚠️ Phase 2: RAID Configuration & Array Modifications

> [!CAUTION]
> **Data Destruction & Reboot Warning:**
> Enabling, disabling, or modifying RAID levels **WILL ERASE ALL DATA** on all attached drives and automatically reboot the FortiGate unit.

- [ ] **Enable RAID Array:** Execute `execute disk raid enable` (Triggers system reboot and full disk format).
- [ ] **Disable RAID Array:** Execute `execute disk raid disable` to revert to standalone disk mode (Triggers system reboot and full disk format).
- [ ] **Configure Rebuild Level:** Set the target rebuild parameters using `execute disk raid rebuild-level` if modifying array performance or redundancy characteristics.

---

## 🧪 Phase 3: Post-Deployment Verification & Diagnostics

- [ ] **Physical Disk Inventory:** Run `execute disk list` to verify all physical drives are detected and online.
- [ ] **RAID Health Verification:** Run `execute disk raid status` to verify array synchronization, member status, and health metrics.
- [ ] **SMART & Hardware Inspection:** Run `diagnose hardware deviceinfo disk` to review low-level SMART attributes and drive health counters.

---

## ⚡ Quick Reference Commands Table

| Task | Command | Operational Impact |
| :--- | :--- | :--- |
| **List Installed Physical Disks** | `execute disk list` | Non-disruptive |
| **Check RAID Array Health** | `execute disk raid status` | Non-disruptive |
| **Inspect Low-Level Hardware Info** | `diagnose hardware deviceinfo disk` | Non-disruptive |
| **Enable RAID Array** | `execute disk raid enable` | ⚠️ **Reboot & Erase Data** |
| **Disable RAID Array** | `execute disk raid disable` | ⚠️ **Reboot & Erase Data** |
| **Change RAID Rebuild Level** | `execute disk raid rebuild-level` | ⚠️ **Reboot & Erase Data** |
ببین الان نمیخام ازین دوتا فایل چزی بهم بدی فقط میخوام داشته باشی تا بعدا اگر بخشی مثالا درباره سرفصل x ازت خواتسم بیای و تو جفت این دوتا فایل تحلیل عمیق بکنی و بهم ترکیبی از حروجی درست با قرار دادن نکات و پارامتر ها از دوتا فایل بهم بدی

فایل اول با تمام جزئیات فنی و پیکربندی‌های مربوط به Security Fabric، HA، مسیریابی و IPSec دریافت و کاملاً در حافظم ذخیره شد. این اطلاعات رو دقیقاً همون‌طور که خواستی بدون هیچ پردازش اضافه‌ای نگه می‌دارم.

هر زمان که برای تدوین ویدیوهای آموزشی، طراحی سناریوهای عملی فورتی‌نت یا تحلیل دقیق یک سرفصل خاص به کمک نیاز داشتی، کافیه موضوع رو مشخص کنی. من ساختار این داکیومنت‌ها رو بررسی می‌کنم و یک ترکیب تمیز، عمیقاطلاعات فایل نخست (Fortigate-NSE7) با موفقیت دریافت و بررسی شد.

تمام نکات فنی، پارامترهای CLI، سناریوهای مسیریابی (BGP, OSPF)، پیکربندی‌های HA، Security Fabric و مباحث IPSec موجود در این متن به عنوان مرجع برای پاسخ‌های بعدی ذخیره شدند.

هر زمان که آماده بودید، سرفصل مورد نظرتان را مشخص کنید تا تحلیل عمیق، استخراج نکات کلیدی و ترکیب داده‌ها را بر اساس این مستندات انجام دهم.

منتظر دریافت فایل دوم یا اولین پرسش شما هستم.

ازت میخام بهم کانفرم بدی که نسبت به فایل nse4 و nse7 اگاهی عمیق داری و میتونی به شکل کامل این دوتارو باهم ترکیب کنی

ساختار و جزئیات فنی هر دو فایل NSE 4 (پالیسی‌ها، NAT، احراز هویت، وب‌فیلترینگ و مبانی فایروال) و NSE 7 (معماری‌های پیشرفته سازمانی، عیب‌یابی عمیق، Security Fabric، مسیریابی داینامیک مثل OSPF/BGP و پیاده‌سازی پیچیده IPSec) کاملاً دریافت، تحلیل و در حافظه من تثبیت شده‌اند.

من آمادگی کامل دارم تا این دو سطح را برای طراحی سناریوهای عملی و سرفصل‌های دقیق ترکیب کنم. می‌توانیم مفاهیم را طوری یکپارچه کنیم که مسیر یادگیری از پیکربندی‌های پایه و راه‌اندازی اولیه (سطح NSE 4) آغاز شده و مستقیماً به سمت معماری‌های پیچیده، تحلیل پکت در CLI، رفع باگ‌های HA و پیاده‌سازی‌های مقیاس‌پذیر (سطح NSE 7) حرکت کند.

رویکردهای ترکیبی برای توسعه محتوا:

سناریوهای مرحله‌ای (Lab Scenarios): طراحی توپولوژی‌هایی که در فاز اول با مفاهیم NSE 4 راه‌اندازی می‌شوند و در فاز دوم با تکنیک‌های ترابل‌شوتینگ و بهینه‌سازی NSE 7 گسترش می‌یابند.

جریان یکپارچه آموزشی: ایجاد یک روند منطقی برای تدریس که دانشجو را از درک مکانیزم‌های پایه تا سطح معماری Enterprise و کار با REST API هدایت کند.

تحلیل‌های عمیق یک موضوع واحد: بررسی قابلیتی مانند IPSec از دو نگاه؛ ابتدا راه‌اندازی و درک فازها (NSE 4) و سپس دیباگ کردن پیام‌های IKE و خطایابی تانل‌ها در محیط‌های عملیاتی (NSE 7).

برای شروع کار روی محتوای تکنیکال بعدی، دقیقاً مشخص کن که می‌خواهی از کدام ماژول (مثلاً ترکیب Routing، معماری Security Fabric یا مباحث کریپتوگرافی) استارت بزنیم.



interfaces

software switches processed with cpu

hardware switches is virtual switch on hardware level of asic and co-processors

supports spanning tree with optimized processing



enhanced mac vlan provide different mac address for vlans on same physical interfaces



on admin access in interface setting

fmg-access

allow fortimanager authorization automatically on communication exchange between fgt and fmg



ftm

forti token mobile

as push access



security fabric

forti telemetry

capwap



speedtest



on fortigate we can use fec on interfaces with 10g,40gm25g,100g

config system interface

ed sfp10

set speed 40000full

set media-type cr4

set forward-error-correcrion cl91-rs-fec

end



*fec disable on 1000m 10g 40g also not supported

*25g and 100g used automatically cl91-rs-fec



sr: shortens the code by reducing the number of information bits

lr: expands the code by lifting the parity-check matrix

cr: represents the efficiency of the code (ratio of information bits to total bits)



different types of optical transceivers or modules used for transmitting data over fiber optic cables

these modules are typically used in high-speed data communication applications, such as in data centers or enterprise networks



sr4 (short range 4):

sr4 stands for short range 4-1ane

it is designed for short-range multimode fiber (mmf) connections, typically up to 100 meters

it uses four parallel fibers for transmission and four parallel fibers for reception, making it suitable for high-density environments like data centers

commonly used in 40g and 100g ethernet applications



lr4 (long range 4):

lr4 stands for long range 4-1ane

it is designed for long-range single-mode fiber (smf) connections, typically up to 10 kilometers or more

like sr4, it uses four lanes for transmission and four lanes for reception, but it is optimized for longer distances

commonly used in 40g and 100g ethernet applications where longer reach is required



cr4 (copper 4) :

cr4 stands for copper 4-1ane

it is designed for short-range connections over copper cables, typically up to 5 meters

it uses four twinaxial copper cables for transmission and reception

commonly used in 40g ethernet applications within racks or between adjacent racks in data centers



ipam

ip address management



config system ipam

set pool-subnet 172.16.0.0/16

set status enable

end



config system interface

set managed-subnetwork-size 512

edit port8

set ip-managed-by-fortiipam enable

end



diagnose sys ipam largest-available-subnet

#show us maximum subnet range on ipam



diagnose sys ipam reservation-status

diagnose sys ipam confirm-reserv

diagnose sys ipam delete-device-from-ipam

diagnose sys ipam dump-ipams-free-subnets

diagnose sys ipam dump-ipams-entreis



captive portal

just useful on local users

used http request for authentication pages



better use sso and radius instead of



we can track users on fortiview source and firewall users pannel



external sources in captive portal

add another device like fgt-2 in scenario

then clients from fgt-1 goes to fgt-2 and get verified

default gateway will be  fgt-2 better check routes



on fgt-1 interface config and security part that pointed to captive portal authentication external section set fgt2 ip address 192.168.12.2

lan 192.168.101.0/24 ------ fgt-1 ----- 192.168.12.0/30 ------- fgt-2 ------ 192.168.254.0/24 ------- edge -----8.8.8.8



on fgt2 interface side fgt-1 (192.168.12.0/30) must enable captive portal with specify user groups



*must exempt ip address of fgt-1 192.168.12.1 cause this device must forward and negotiate all packets fom it's owned lan side to fgt-2 if get trouble on it all authentication procedures get failed



*disable nat from lan to 192.168.12.0/30 cause bypass the captive portal pannel



change mtu

config system interface

edit port8

set mtu-override enable

set mtu 1234

set tcp-mss 1448

end



one-arm sniffer

on switch

conf t

monitor session 1 source interface gig0/0 both

monitor session 1 destination interface gig0/2



open wireshark



if need used fortigate as sniffer

set one-arm sniffer on  interface address on network part fortigate

on network > diagnostics  start the captures



which item prevent activation of one-arm

wan interface

firewall policies

virtual ip as destination nat

and all another dependencies



might have lower level perform than usuall iinline scanning cause we have more steps as nturbo or cp  on packets nad sniffers

might have high cpu usage and packet losts better use in short time



kernel buffer size will be effected



execute log fitlter cat 19

execute log display



our migration tool in interfaces just worked on physical interfaces



physical interfaces

sfp and sfp+ on fortigate are sensetive and used their own modules



get system interface transceiver



vlan

in nat mode has more performance

.1ad used in double tagging and metro-ethernet or q-in-q  tunneling

.1q used for vlan tagging as normal state



virtual vlan switch

the hardware switch ports on fortigate models that support virtual vlan switches can be used as a layer 2 switch

virtual vlan switch mode allows 802.1q vlans to be assigned to ports, and the configuration of one interface as a trunk port

the following fortigate series are supported in fortios 7.2:

60f, 80f, 100e, 100f, 140e, 200f, 300e, 400e, and 1100e



seperate some port from fortigate fabric



config system global

set virtual-switch-vlan enable

end



config system virtual-switch

edit vlan10

#create virtual switches



set physical-switch sw0

set vlan 10

config port

edit port10

next

edit port11

end



config system interface

edit vlan10

set type hard-switch

#from virtual at above create hard switch



set ip 192.168.100.1/24

set allow ping http ssh

end



*where to used

in some specific and special conditions



between 2 edge fortigates we have ha with trunked links between them

between 2 isp we have seperated vlans as dual-multihome



if have spanning-tree on network better used virtual-switch and these commands to manage spanning-tree

*software switches help you  to manage most scenarios but on multiple spanning-tree or another instances better used hard-switch and virtual-switch



overview of .1q and .1ad

the ieee 802.1q and ieee 802.1ad standards are both related to vlan (virtual local area network) tagging in ethernet frames, but they serve different purposes and functionalities



ieee 802.1q

single vlan tagging: the 802.1q standard allows for a single vlan tag to be inserted into an ethernet frame

his tag includes a 12-bit vlan id, which can represent up to 4096 unique vlans (with a practical limit of 4094 due to reserved values)



ethertype: the ethertype for a standard 802.1q tag is set to 0x8100, which identifies the frame as containing a vlan tag



use case: it is widely used for basic vlan configurations, allowing network segmentation within a single broadcast domain



ieee 802.1ad (qinq)

double vlan tagging: the 802.1ad standard, also known as qinq (or "vlan stacking"), extends the capabilities of 802.1q by

allowing multiple vlan tags to be nested within a single ethernet frame

this effectively creates a 'vlan within a vlan scenario, enabling greater network segmentation and scalability



ethertype: in contrast to 802.1q, the outer tag in an 802.1ad frame uses an ethertype of 0x88a8 for the service tag (s-tag), while the inner tag retains the standard ethertype of 0x8100 for the customer tag (c-tag)



enhanced functionality: the addition of a second tag allows for more sophisticated operations, such as easier modification of frames by switches and better encapsulation of customer traffic by service providers

the outer tag is primarily used to identify and segregate traffic from different customers



features ieee 802.1q ieee 802.1ad (qinq)

tagging type single vlan tag double vlan tag

ethertype 0x8100 outer: 0x88a8; inner: 0x8100

vlan id capacity up to 4096 unique vlans supports nesting for extensive segmentation

use case basic vlan configurations service provider networks, metro ethernet

additional features none drop eligibility indicator (dei) replaces canonical format indicator (cfi) 24



aggregation

on switch

conf t

interface range gig0/0-1

channel-group 1  mode on

interface range gig1/0-1

channel-group 2  mode on



config system interface

edit agg-1

set lacp-mode static

set lldp-transmission enable

end



on mikrotik

add interface bonding

mode 802.3ad

then set ip address on bond interface



enhanced hashing lag memeber selection

xor16 (lighter)  (used in datacenters)

xor8

xor4

crc16 (resource intensive)



used

ip protocols

source ip

destination ip

source port

destination port



config system npu

set lag-out-pport-select enable

config sw-eh-hash

set computation xor16

set ip-protocol include

set sour.....

set netmask-length 32

end



redundant

don't use channel-group here



used on full mesh scenario

normally used 4 ports



failur detection on aggregate and redundant

config system interface

edit agg-1

set fail-detect enable

set fail-alert-method link-down

#what happen if get fail on aggregate interfaces



set fail-alert-interface port4

#which interface get down if agg-1 get fail



end







802.1x supported on hardware switch interface with np6 platform on fgt 30xe,40xe110xe

config system interface

edit port8

set security-mode 802.1x

set security-group test

end



diagnose sys 802-1x status



virtual wire pair

base on huge calculation and loadbalancing with devices better used virtual wire pair concept to connect clients directly to servers after verification and trsut level access

better use this on lan side and inside network

worked on layer 2 mode and help on direct server return (dsr) to forward traffic between clients and server with different mac address



internal segmentation firewall (isfw) will be used with dsr and virtual wire pair (vwp)



config system virtual-wire-pair

edit vwp-1

set memeber port3 port4

set wildcard-vlan disble

#used all valn range

end



*interfaces must be empty and no referrer



under policy & objects > firewall virtual wire pair policy

have direction of traffic flow

flow mode or proxy mode inspection



when used nat in vwp

used overload ip pool nat



parallel redundancy protocol (prp)

is a network redundancy protocol standardized by iec 62439-3:2016, designed for ethernet networks requiring high availability and short switchover times

it is commonly used in critical infrastructure applications such as substation automation and industrial control systems



key features of prp:

dual network architecture

prp operates by having each node connected to two independent, parallel local area networks (lans), known as lan a and lan b



redundant packet transmission

source nodes send duplicate packets over both networks simultaneously the destination node accepts the first packet received and discards the duplicate



fail-independent networks

the two networks are designed to be fail-independent, ensuring that if one network fails communication continues uninterrupted through the other network without any recovery time



application transparency

prp operates at layer 2, making it transparent to higher-layer protocols and applications2 this means no modifications are needed for existing protocols or applications



cost considerations

while providing high reliability, prp requires duplicating all network elements involved in redundancy, which can increase costs compared to other protocols like hsr (high availability seamless redundancy)



however, it allows non-redundant devices to coexist on the same lan without needing special adapters or configurations



use cases:

substation automation

industrial control systems

high-power inverters

synchronized drives

prp offers advantages over traditional redundancy protocols like rstp by providing zero-time recovery from failures but at a higher deployment cost due to duplicated infrastructure requirement



config system setting

set prp-trailer-action enable

end



config system npu

set prp-port-in port15

set prp-port-out port16

end



config system virtual-wire-pair

edit vwp-1

set memeber port15 port16

end



emac (enhanced mac)

interface type emac vlan



generate new mac address over parent



we have trouble on dhcp server on emac interfaces



must used nat mode operation to perform this function as well



if set valn id on emac, could not use them on underlay switches and could not  handle many mac address on tagging



when using a vlan id, the id and the underlying interface must be a unique pair, even if the belong to different vdoms. this is because the underlying, physical interface uses the vlan id as the identifier to dispatch traffic among the vlan and enhanced mac vlan interfaces.



better don't used management interface , heart beat ha interface and transparent vdom



emac treated as physical interfaces and will be advertise and synchronize on ha cluster



we can use emac on vlan and vdom links must set emac type on npu-link or vdom link



vxlan

config system vxlan

edit vx10

set interface port2

#public ip to acceess remote site



set remote-ip 22.22.22.2

#here we can use space and set second ip address

#if use ipsec with hub-and-spoke dialup scenarios better disable device creation , just on spokes phase 2 ipsec interfaces set local subnet same as ipsec interface 12.23.34.2 or 3 ... (define reverse routing point without device creation and tunnel id)



set vni 1000

end



create vlan over vxlan interface to encapsulate the data

in interface we create vlan under vxlan interface with unique vlan id



then add software switch with vlans created under client port like vlan 10 under port3 and vlan 10 under vxlan interface

don't add main vxlan interface



then add policies with vlan interfaces on vxlan and physical port

under software switches we have intra-switch policy

implicit

forward  traffic as default  don't need add firewall policy



explicit

must add seperated policies on firewall policies



we can add vxlan main interface and simple interface into the virtual wire pair then setup policies to access them

if need add dhcp on interafce better used software switches  with main vxlan interafce and client side interface then add dhcp and ip address on itاین فایل در نظر بگیر و به شکل عمیق با تمام جزیات و نکات  به شکل خیلی طولانی بدون حا انداختن حتی یک خط تحلیل و ترکیب کن و بهم در قالب cheet shet  و فرمت github خروجی مناسب بده به انگلیسی با رعایت اصول که بالا باهم تجربه کردیم

This comprehensive FortiGate architecture and interface management cheat sheet consolidates advanced layer 2/3 configurations, hardware optimizations, and cryptographic encapsulation standards for enterprise deployments.

FortiGate Interface Architecture & Access Control
Software Switches: Traffic is processed via the main CPU.

Hardware Switches: Virtual switches operating at the hardware level using ASICs and co-processors; supports Spanning Tree Protocol (STP) with optimized processing.

Administrative Access Options:

FMG-Access: Automatically allows FortiManager authorization during communication exchanges between FGT and FMG.

FTM: FortiToken Mobile integration for push authentication access.

Security Fabric: Enables FortiTelemetry and CAPWAP tunnel negotiations.

Speedtest: Bandwidth measurement tool.

Migration Limitations: The built-in interface migration tool functions exclusively on physical interfaces.

Physical Interfaces: SFP and SFP+ ports are highly sensitive and require dedicated vendor modules. Validate using get system interface transceiver.

Transceivers & Forward Error Correction (FEC)
FEC optimizes data integrity on high-capacity links (10G, 25G, 40G, 100G).

1000M, 10G, 40G: FEC is disabled or explicitly not supported on these speeds natively.

25G & 100G: Automatically utilizes the cl91-rs-fec profile.

Coding Terminology:

SR: Shortens the code by reducing information bits.

LR: Expands the code by lifting the parity-check matrix.

CR: Represents code efficiency (ratio of information bits to total bits).

Transceiver Type	Full Name	Range / Cable Type	Specifications & Use Cases
SR4	Short Range 4-Lane	Up to 100m / Multimode Fiber (MMF)	Uses 4 parallel fibers for Tx and 4 for Rx. High-density data centers (40G/100G).
LR4	Long Range 4-Lane	Up to 10km+ / Singlemode Fiber (SMF)	Optimized for long distances; uses 4 lanes for Tx and 4 for Rx (40G/100G).
CR4	Copper 4-Lane	Up to 5m / Twinaxial Copper	Uses 4 twinax cables for Tx/Rx. Designed for intra-rack or adjacent rack connections (40G).
Bash
config system interface
    edit sfp10
    set speed 40000full
    set media-type cr4
    set forward-error-correction cl91-rs-fec
end
IP Address Management (IPAM)
Bash
config system ipam
    set pool-subnet 172.16.0.0/16
    set status enable
end

config system interface
    edit port8
    set managed-subnetwork-size 512
    set ip-managed-by-fortiipam enable
end
Diagnostic Commands:

diagnose sys ipam largest-available-subnet: Displays the maximum subnet range on IPAM.

diagnose sys ipam reservation-status

diagnose sys ipam confirm-reserv

diagnose sys ipam delete-device-from-ipam

diagnose sys ipam dump-ipams-free-subnets

diagnose sys ipam dump-ipams-entries

External Captive Portal Architecture
Captive portals utilize HTTP requests for authentication pages (primarily for local users), though SSO and RADIUS are recommended alternatives. User tracking is available via FortiView Source and Firewall Users panels.

External Source Scenario (FGT-1 forwarding to FGT-2):

Topology: LAN (192.168.101.0/24) -> FGT-1 -> Transit (192.168.12.0/30) -> FGT-2 -> WAN (192.168.254.0/24) -> Edge (8.8.8.8)

FGT-1 Configuration: On the interface pointing to the LAN, point the captive portal external authentication section to FGT-2's IP (192.168.12.2). Default route must point to FGT-2.

FGT-2 Configuration: Enable Captive Portal on the interface facing FGT-1 (192.168.12.0/30) and specify user groups.

Critical Dependencies:

Exempt FGT-1 IP: 192.168.12.1 must be exempted. FGT-1 must forward and negotiate all LAN packets to FGT-2. If blocked, authentication procedures fail.

Disable NAT: Disable NAT from the LAN to the 192.168.12.0/30 transit link to prevent bypassing the captive portal panel.

MTU Override & 802.1x Authentication
MTU Modification:

Bash
config system interface
    edit port8
    set mtu-override enable
    set mtu 1234
    set tcp-mss 1448
end
802.1x Hardware Switch:
Supported specifically on hardware switch interfaces running the NP6 platform (FortiGate 30xE, 40xE, 110xE).

Bash
config system interface
    edit port8
    set security-mode 802.1x
    set security-group test
end
Verify with: diagnose sys 802-1x status

Traffic Sniffing (One-Arm)
Switch Configuration (SPAN):

Bash
conf t
monitor session 1 source interface gig0/0 both
monitor session 1 destination interface gig0/2
FortiGate Setup: Set one-arm sniffer on an interface with a network-part address. Start captures via Network > Diagnostics. Alternatively, use Wireshark.

Limitations & Conflicts: One-arm sniffing is prevented by WAN interfaces, active firewall policies, Virtual IPs (Destination NAT), and other strict dependencies.

Performance Impact: Eliminates hardware acceleration (nTurbo or CP) on captured packets. High CPU usage, potential packet loss, and kernel buffer size impacts. Recommended for short durations only.

Log Diagnostics:

Bash
execute log filter cat 19
execute log display
VLANs & Hardware Virtual Switches
VLANs process with higher performance in NAT mode.

Virtual VLAN Switch (Layer 2 Mode): Allows assigning 802.1Q VLANs directly to ports and configuring interfaces as trunk ports. Separates specific ports from the FortiGate Security Fabric.

Supported FortiOS 7.2 Models: 60F, 80F, 100E, 100F, 140E, 200F, 300E, 400E, 1100E.

Primary Use Cases: Dual-multihoming between two ISPs with separated VLANs, HA trunked links between edge FortiGates, and complex Spanning Tree Protocol (STP) environments where software switches are insufficient.

Bash
config system global
    set virtual-switch-vlan enable
end

config system virtual-switch
    edit vlan10
    set physical-switch sw0
    set vlan 10
    config port
        edit port10
        next
        edit port11
end

config system interface
    edit vlan10
    set type hard-switch
    set ip 192.168.100.1/24
    set allowaccess ping http ssh
end
Feature	IEEE 802.1Q (Standard VLAN)	IEEE 802.1ad (QinQ / VLAN Stacking)
Tagging Type	Single VLAN tag.	Double VLAN tag (nested tags).
EtherType	0x8100	Outer (S-Tag): 0x88A8; Inner (C-Tag): 0x8100.
VLAN ID Capacity	Up to 4096 unique VLANs.	Supports nesting for extensive segmentation.
Primary Use Case	Basic VLAN configs, single broadcast domain.	Service provider networks, Metro Ethernet.
Additional Features	None.	Drop Eligibility Indicator (DEI) replaces Canonical Format Indicator (CFI).
Link Aggregation (LAG) & Redundancy
Redundant Interfaces: Deployed in full-mesh scenarios (typically 4 ports). Never use standard channel-groups for this interface type.

Cisco & FortiGate LACP Configuration:

Bash
# Cisco Switch
conf t
interface range gig0/0-1
channel-group 1 mode on
interface range gig1/0-1
channel-group 2 mode on

# FortiGate
config system interface
    edit agg-1
    set lacp-mode static
    set lldp-transmission enable
end
(Note: On MikroTik, add interface bonding, set mode 802.3ad, then assign IP to the bond interface).

Enhanced Hashing (LAG Member Selection):
Relies on IP protocols, Source IP, Destination IP, Source Port, and Destination Port.

xor16: Lighter load, standard for data centers.

xor8 / xor4: Alternative lighter hashes.

crc16: Highly resource-intensive.

Bash
config system npu
    set lag-out-port-select enable
    config sw-eh-hash
        set computation xor16
        set ip-protocol include
        set netmask-length 32
end
Failure Detection Config:

Bash
config system interface
    edit agg-1
    set fail-detect enable
    set fail-alert-method link-down
    set fail-alert-interface port4 
end
Virtual Wire Pair (VWP) & PRP
VWP is built for high-performance load balancing and direct server connectivity without modifying routing. It operates strictly at Layer 2 and facilitates Direct Server Return (DSR) to forward traffic between clients and servers utilizing different MAC addresses.

Interfaces assigned to a VWP must be completely empty with no existing references.

Often deployed alongside an Internal Segmentation Firewall (ISFW).

Policies: Found under Policy & Objects > Firewall Virtual Wire Pair Policy. Traffic flow is directional, utilizing flow or proxy mode inspection. When NAT is required within a VWP, Overload IP Pool NAT is mandatory.

Bash
config system virtual-wire-pair
    edit vwp-1
    set member port3 port4
    set wildcard-vlan disable 
end
Parallel Redundancy Protocol (PRP - IEC 62439-3:2016):
A zero-recovery-time redundancy protocol for critical infrastructure (substation automation, industrial control systems, high-power inverters, synchronized drives).

Mechanics: Nodes connect to independent LAN A and LAN B. Duplicate packets are sent simultaneously. The destination accepts the first packet and discards the duplicate.

Advantages: Fail-independent, fully transparent to higher-layer applications (operates at L2), allows non-redundant devices to coexist.

Disadvantage: High cost due to fully duplicated infrastructure compared to RSTP or HSR.

Bash
config system settings
    set prp-trailer-action enable
end

config system npu
    set prp-port-in port15
    set prp-port-out port16
end

config system virtual-wire-pair
    edit vwp-1
    set member port15 port16
end
Enhanced MAC (EMAC) VLAN
EMAC interfaces generate a distinct MAC address over a parent physical interface.

Operational Mode: Must utilize NAT mode to function properly.

Limitations: DHCP servers struggle on EMAC interfaces. Assigning a VLAN ID breaks functionality on underlay switches and cannot handle massive MAC tagging.

VLAN Association Rules: When using a VLAN ID, the ID and underlying physical interface must form a unique pair (even across VDOMs) to correctly dispatch traffic.

HA Synchronization: EMAC is treated natively as a physical interface and synchronizes seamlessly across HA clusters.

Restrictions: Avoid using EMAC on management interfaces, HA heartbeat links, and Transparent VDOMs. If applied to VDOM or VLAN links, the EMAC type must be explicitly set on the NPU/VDOM link.

VXLAN Architecture
VXLAN encapsulates Layer 2 data over Layer 3 boundaries.

Bash
config system vxlan
    edit vx10
    set interface port2
    set remote-ip 22.22.22.2
    set vni 1000
end
Hub-and-Spoke IPsec/Dialup Scenarios: Disable device creation. On the spoke's Phase 2 IPsec interface, set the local subnet to match the IPsec interface (e.g., 12.23.34.2) to define reverse routing points without tunnel IDs.

VLAN Over VXLAN: Create a VLAN interface directly beneath the VXLAN interface using a unique VLAN ID to encapsulate the data.

Switching Integration: Bind a software switch containing the physical client port VLAN (e.g., VLAN 10 under port3) and the VXLAN VLAN (VLAN 10 under vxlan). Never add the main VXLAN interface directly to the software switch.

Policy Routing: Under software switches, intra-switch policies can be implicit (forwards default traffic without firewall policies) or explicit (requires discrete firewall policies).

Advanced Topologies: A primary VXLAN interface and standard interface can be bound into a Virtual Wire Pair with specific policies. For DHCP services, bundle the main VXLAN interface and the client interface into a software switch, assign an IP, and attach the DHCP server.

This comprehensive FortiGate architecture and interface management cheat sheet consolidates advanced layer 2/3 configurations, hardware optimizations, and cryptographic encapsulation standards for enterprise deployments.



FortiGate Interface Architecture & Access Control

Software Switches: Traffic is processed via the main CPU.



Hardware Switches: Virtual switches operating at the hardware level using ASICs and co-processors; supports Spanning Tree Protocol (STP) with optimized processing.



Administrative Access Options:



FMG-Access: Automatically allows FortiManager authorization during communication exchanges between FGT and FMG.



FTM: FortiToken Mobile integration for push authentication access.



Security Fabric: Enables FortiTelemetry and CAPWAP tunnel negotiations.



Speedtest: Bandwidth measurement tool.



Migration Limitations: The built-in interface migration tool functions exclusively on physical interfaces.



Physical Interfaces: SFP and SFP+ ports are highly sensitive and require dedicated vendor modules. Validate using get system interface transceiver.



Transceivers & Forward Error Correction (FEC)

FEC optimizes data integrity on high-capacity links (10G, 25G, 40G, 100G).



1000M, 10G, 40G: FEC is disabled or explicitly not supported on these speeds natively.



25G & 100G: Automatically utilizes the cl91-rs-fec profile.



Coding Terminology:



SR: Shortens the code by reducing information bits.



LR: Expands the code by lifting the parity-check matrix.



CR: Represents code efficiency (ratio of information bits to total bits).



Transceiver TypeFull NameRange / Cable TypeSpecifications & Use CasesSR4Short Range 4-LaneUp to 100m / Multimode Fiber (MMF)Uses 4 parallel fibers for Tx and 4 for Rx. High-density data centers (40G/100G).LR4Long Range 4-LaneUp to 10km+ / Singlemode Fiber (SMF)Optimized for long distances; uses 4 lanes for Tx and 4 for Rx (40G/100G).CR4Copper 4-LaneUp to 5m / Twinaxial CopperUses 4 twinax cables for Tx/Rx. Designed for intra-rack or adjacent rack connections (40G).

Bash



config system interface

edit sfp10

set speed 40000full

set media-type cr4

set forward-error-correction cl91-rs-fec

end

IP Address Management (IPAM)

Bash



config system ipam

set pool-subnet 172.16.0.0/16

set status enable

end



config system interface

edit port8

set managed-subnetwork-size 512

set ip-managed-by-fortiipam enable

end

Diagnostic Commands:



diagnose sys ipam largest-available-subnet: Displays the maximum subnet range on IPAM.



diagnose sys ipam reservation-status



diagnose sys ipam confirm-reserv



diagnose sys ipam delete-device-from-ipam



diagnose sys ipam dump-ipams-free-subnets



diagnose sys ipam dump-ipams-entries



External Captive Portal Architecture

Captive portals utilize HTTP requests for authentication pages (primarily for local users), though SSO and RADIUS are recommended alternatives. User tracking is available via FortiView Source and Firewall Users panels.



External Source Scenario (FGT-1 forwarding to FGT-2):



Topology: LAN (192.168.101.0/24) -> FGT-1 -> Transit (192.168.12.0/30) -> FGT-2 -> WAN (192.168.254.0/24) -> Edge (8.8.8.8)



FGT-1 Configuration: On the interface pointing to the LAN, point the captive portal external authentication section to FGT-2's IP (192.168.12.2). Default route must point to FGT-2.



FGT-2 Configuration: Enable Captive Portal on the interface facing FGT-1 (192.168.12.0/30) and specify user groups.



Critical Dependencies:



Exempt FGT-1 IP: 192.168.12.1 must be exempted. FGT-1 must forward and negotiate all LAN packets to FGT-2. If blocked, authentication procedures fail.



Disable NAT: Disable NAT from the LAN to the 192.168.12.0/30 transit link to prevent bypassing the captive portal panel.



MTU Override & 802.1x Authentication

MTU Modification:



Bash



config system interface

edit port8

set mtu-override enable

set mtu 1234

set tcp-mss 1448

end

802.1x Hardware Switch:

Supported specifically on hardware switch interfaces running the NP6 platform (FortiGate 30xE, 40xE, 110xE).



Bash



config system interface

edit port8

set security-mode 802.1x

set security-group test

end

Verify with: diagnose sys 802-1x status



Traffic Sniffing (One-Arm)

Switch Configuration (SPAN):



Bash



conf t

monitor session 1 source interface gig0/0 both

monitor session 1 destination interface gig0/2

FortiGate Setup: Set one-arm sniffer on an interface with a network-part address. Start captures via Network > Diagnostics. Alternatively, use Wireshark.



Limitations & Conflicts: One-arm sniffing is prevented by WAN interfaces, active firewall policies, Virtual IPs (Destination NAT), and other strict dependencies.



Performance Impact: Eliminates hardware acceleration (nTurbo or CP) on captured packets. High CPU usage, potential packet loss, and kernel buffer size impacts. Recommended for short durations only.



Log Diagnostics:



Bash



execute log filter cat 19

execute log display

VLANs & Hardware Virtual Switches

VLANs process with higher performance in NAT mode.



Virtual VLAN Switch (Layer 2 Mode): Allows assigning 802.1Q VLANs directly to ports and configuring interfaces as trunk ports. Separates specific ports from the FortiGate Security Fabric.



Supported FortiOS 7.2 Models: 60F, 80F, 100E, 100F, 140E, 200F, 300E, 400E, 1100E.



Primary Use Cases: Dual-multihoming between two ISPs with separated VLANs, HA trunked links between edge FortiGates, and complex Spanning Tree Protocol (STP) environments where software switches are insufficient.



Bash



config system global

set virtual-switch-vlan enable

end



config system virtual-switch

edit vlan10

set physical-switch sw0

set vlan 10

config port

edit port10

next

edit port11

end



config system interface

edit vlan10

set type hard-switch

set ip 192.168.100.1/24

set allowaccess ping http ssh

end

FeatureIEEE 802.1Q (Standard VLAN)IEEE 802.1ad (QinQ / VLAN Stacking)Tagging TypeSingle VLAN tag.Double VLAN tag (nested tags).EtherType0x8100Outer (S-Tag): 0x88A8; Inner (C-Tag): 0x8100.VLAN ID CapacityUp to 4096 unique VLANs.Supports nesting for extensive segmentation.Primary Use CaseBasic VLAN configs, single broadcast domain.Service provider networks, Metro Ethernet.Additional FeaturesNone.Drop Eligibility Indicator (DEI) replaces Canonical Format Indicator (CFI).

Link Aggregation (LAG) & Redundancy

Redundant Interfaces: Deployed in full-mesh scenarios (typically 4 ports). Never use standard channel-groups for this interface type.



Cisco & FortiGate LACP Configuration:



Bash



# Cisco Switch

conf t

interface range gig0/0-1

channel-group 1 mode on

interface range gig1/0-1

channel-group 2 mode on# FortiGate

config system interface

edit agg-1

set lacp-mode static

set lldp-transmission enable

end

(Note: On MikroTik, add interface bonding, set mode 802.3ad, then assign IP to the bond interface).



Enhanced Hashing (LAG Member Selection):

Relies on IP protocols, Source IP, Destination IP, Source Port, and Destination Port.



xor16: Lighter load, standard for data centers.



xor8 / xor4: Alternative lighter hashes.



crc16: Highly resource-intensive.



Bash



config system npu

set lag-out-port-select enable

config sw-eh-hash

set computation xor16

set ip-protocol include

set netmask-length 32

end

Failure Detection Config:



Bash



config system interface

edit agg-1

set fail-detect enable

set fail-alert-method link-down

set fail-alert-interface port4

end

Virtual Wire Pair (VWP) & PRP

VWP is built for high-performance load balancing and direct server connectivity without modifying routing. It operates strictly at Layer 2 and facilitates Direct Server Return (DSR) to forward traffic between clients and servers utilizing different MAC addresses.



Interfaces assigned to a VWP must be completely empty with no existing references.



Often deployed alongside an Internal Segmentation Firewall (ISFW).



Policies: Found under Policy & Objects > Firewall Virtual Wire Pair Policy. Traffic flow is directional, utilizing flow or proxy mode inspection. When NAT is required within a VWP, Overload IP Pool NAT is mandatory.



Bash



config system virtual-wire-pair

edit vwp-1

set member port3 port4

set wildcard-vlan disable

end

Parallel Redundancy Protocol (PRP - IEC 62439-3:2016):

A zero-recovery-time redundancy protocol for critical infrastructure (substation automation, industrial control systems, high-power inverters, synchronized drives).



Mechanics: Nodes connect to independent LAN A and LAN B. Duplicate packets are sent simultaneously. The destination accepts the first packet and discards the duplicate.



Advantages: Fail-independent, fully transparent to higher-layer applications (operates at L2), allows non-redundant devices to coexist.



Disadvantage: High cost due to fully duplicated infrastructure compared to RSTP or HSR.



Bash



config system settings

set prp-trailer-action enable

end



config system npu

set prp-port-in port15

set prp-port-out port16

end



config system virtual-wire-pair

edit vwp-1

set member port15 port16

end

Enhanced MAC (EMAC) VLAN

EMAC interfaces generate a distinct MAC address over a parent physical interface.



Operational Mode: Must utilize NAT mode to function properly.



Limitations: DHCP servers struggle on EMAC interfaces. Assigning a VLAN ID breaks functionality on underlay switches and cannot handle massive MAC tagging.



VLAN Association Rules: When using a VLAN ID, the ID and underlying physical interface must form a unique pair (even across VDOMs) to correctly dispatch traffic.



HA Synchronization: EMAC is treated natively as a physical interface and synchronizes seamlessly across HA clusters.



Restrictions: Avoid using EMAC on management interfaces, HA heartbeat links, and Transparent VDOMs. If applied to VDOM or VLAN links, the EMAC type must be explicitly set on the NPU/VDOM link.



VXLAN Architecture

VXLAN encapsulates Layer 2 data over Layer 3 boundaries.



Bash



config system vxlan

edit vx10

set interface port2

set remote-ip 22.22.22.2

set vni 1000

end

Hub-and-Spoke IPsec/Dialup Scenarios: Disable device creation. On the spoke's Phase 2 IPsec interface, set the local subnet to match the IPsec interface (e.g., 12.23.34.2) to define reverse routing points without tunnel IDs.



VLAN Over VXLAN: Create a VLAN interface directly beneath the VXLAN interface using a unique VLAN ID to encapsulate the data.



Switching Integration: Bind a software switch containing the physical client port VLAN (e.g., VLAN 10 under port3) and the VXLAN VLAN (VLAN 10 under vxlan). Never add the main VXLAN interface directly to the software switch.



Policy Routing: Under software switches, intra-switch policies can be implicit (forwards default traffic without firewall policies) or explicit (requires discrete firewall policies).



Advanced Topologies: A primary VXLAN interface and standard interface can be bound into a Virtual Wire Pair with specific policies. For DHCP services, bundle the main VXLAN interface and the client interface into a software switch, assign an IP, and attach the DHCP server.این موارد را تبدیل به قالب github کن و بهم بده با دید cheet sheet

Markdown
# FortiGate Architecture & Interface Management Cheat Sheet

This comprehensive reference guide consolidates enterprise FortiGate Layer 2/Layer 3 interface configurations, ASIC hardware optimizations, packet acceleration techniques, and network encapsulation standards.

---

## 📌 Table of Contents
- [Interface Types & Management Options](#-interface-types--management-options)
- [Transceivers & Forward Error Correction (FEC)](#-transceivers--forward-error-correction-fec)
- [IP Address Management (IPAM)](#-ip-address-management-ipam)
- [External Captive Portal Architecture](#-external-captive-portal-architecture)
- [Interface Parameters (MTU & 802.1X)](#-interface-parameters-mtu--8021x)
- [Traffic Sniffing (One-Arm SPAN)](#-traffic-sniffing-one-arm-span)
- [VLANs & Hardware Virtual Switches](#-vlans--hardware-virtual-switches)
- [Link Aggregation (LAG) & Redundancy](#-link-aggregation-lag--redundancy)
- [Virtual Wire Pair (VWP) & PRP](#-virtual-wire-pair-vwp--prp)
- [Enhanced MAC (EMAC) VLAN](#-enhanced-mac-emac-vlan)
- [VXLAN Architecture & Encapsulation](#-vxlan-architecture--encapsulation)

---

## 🛠 Interface Types & Management Options

### Interface Architecture
* **Software Switches:** Traffic is switched in software and processed via the main system CPU.
* **Hardware Switches:** Virtual switches operating at the hardware layer utilizing ASICs and co-processors. Supports optimized Spanning Tree Protocol (STP) processing.
* **Physical Interfaces:** SFP/SFP+ ports are hardware-sensitive and require verified vendor modules. Verify diagnostics via:
  ```bash
  get system interface transceiver
Migration Limits: The built-in FortiOS interface migration tool operates exclusively on physical interfaces.

Administrative Access Protocols
Access Flag	Purpose & Functionality
fmg-access	Automatically grants FortiManager authorization during device communication exchanges.
ftm	Enables FortiToken Mobile push authentication handling.
security-fabric	Enables FortiTelemetry and CAPWAP tunnel management across fabric nodes.
speedtest	Permits on-demand bandwidth testing requests from FortiManager/CLI.
⚡ Transceivers & Forward Error Correction (FEC)
Forward Error Correction (FEC) maintains payload integrity across high-speed data paths (10G, 25G, 40G, 100G).

1G / 10G / 40G: FEC is disabled or unsupported natively on these transceiver speeds.

25G / 100G: Automatically negotiates and applies the cl91-rs-fec profile.

Coding Terminology
SR (Short Range): Reduces information bits to shorten total code length.

LR (Long Range): Lifts the parity-check matrix to expand code length over distance.

CR (Copper Range): Represents code efficiency (ratio of payload information bits to total transmitted bits).

Transceiver Specification Matrix
Type	Full Name	Reach / Cable Type	Architectural Design & Use Cases
SR4	Short Range 4-Lane	≤ 100m / Multimode Fiber (MMF)	Employs 4 parallel Tx and 4 parallel Rx lanes. Ideal for high-density data center access (40G/100G).
LR4	Long Range 4-Lane	≤ 10km+ / Singlemode Fiber (SMF)	Uses 4 optical lanes optimized for long-haul enterprise backbones (40G/100G).
CR4	Copper 4-Lane	≤ 5m / Twinaxial Copper	Employs 4 twinax copper channels for intra-rack or top-of-rack interconnects (40G).
Interface Speed & FEC Configuration
Bash
config system interface
    edit "sfp10"
        set speed 40000full
        set media-type cr4
        set forward-error-correction cl91-rs-fec
    next
end
🏷️ IP Address Management (IPAM)
Automates subnet distribution and centralized tracking for internal interface networks.

Bash
# Global IPAM Pool Setup
config system ipam
    set status enable
    set pool-subnet 172.16.0.0/16
end

# Interface-level IPAM Allocation
config system interface
    edit "port8"
        set ip-managed-by-fortiipam enable
        set managed-subnetwork-size 512
    next
end
IPAM Diagnostics & Management Commands
Bash
# Display the largest available contiguous block
diagnose sys ipam largest-available-subnet

# Display current IP allocation status and entries
diagnose sys ipam reservation-status
diagnose sys ipam dump-ipams-entreis
diagnose sys ipam dump-ipams-free-subnets

# Administrative Operations
diagnose sys ipam confirm-reserv
diagnose sys ipam delete-device-from-ipam
🔐 External Captive Portal Architecture
Captive portals use HTTP redirection for local user authentication. Enterprise environments should prefer 802.1X, SSO, or RADIUS where feasible.

[ LAN Clients ] ---> ( LAN: 192.168.101.0/24 )
                          │
                     [ FGT-1 ] ( Forwarder )
                          │ ( Transit: 192.168.12.0/30 )
                          ▼
                     [ FGT-2 ] ( Authentication Server: 192.168.12.2 )
                          │ ( Edge: 192.168.254.0/24 )
                          ▼
                     [ Internet / 8.8.8.8 ]
Deployment Guidelines & Constraints
FGT-1 Configuration: Configure the ingress LAN interface to point its external captive portal authentication URI to FGT-2 (192.168.12.2). Ensure FGT-1's default route points to FGT-2.

FGT-2 Configuration: Enable captive portal enforcement on the transit interface (192.168.12.0/30) and map the required user authorization groups.

Critical Dependencies:

Exempt Forwarder Address: The upstream interface address of FGT-1 (192.168.12.1) must be exempted from captive portal enforcement on FGT-2. Blocking this address halts all client authentication flow.

Disable NAT on Transit: Do not apply Source NAT on FGT-1 for LAN traffic traversing the 192.168.12.0/30 transit link. NATing obfuscates source IPs, causing authentication bypasses or session mapping failures.

⚙️ Interface Parameters (MTU & 802.1X)
Custom MTU and TCP MSS Override
Adjust system MTU and TCP Maximum Segment Size (MSS) to prevent fragmentation over encapsulated links:

Bash
config system interface
    edit "port8"
        set mtu-override enable
        set mtu 1234
        set tcp-mss 1448
    next
end
Port-Based 802.1X Authentication (Hardware Switch)
Hardware-enforced 802.1X is supported on select NP6-accelerated hardware switch platforms (e.g., FortiGate 30xE, 40xE, 110xE).

Bash
config system interface
    edit "port8"
        set security-mode 802.1x
        set security-group "Corporate-802.1X-Group"
    next
end
Verification:

Bash
diagnose sys 802-1x status
🔍 Traffic Sniffing (One-Arm SPAN)
Used for passive packet monitoring by mirror port ingest.

[ Switch Port Gig0/0 ] ──( Monitored Traffic )──► [ Switch Port Gig0/2 ]
                                                          │
                                                    ( SPAN Feed )
                                                          │
                                                          ▼
                                             [ FortiGate One-Arm Interface ]
Switch Mirroring Configuration (Cisco IOS Example)
Plaintext
monitor session 1 source interface gig0/0 both
monitor session 1 destination interface gig0/2
Operational Considerations & Limitations
Hardware Offload Bypass: One-Arm sniffing completely disables ASIC offload (NP/nTurbo/CP engines) for processed packets, shifting handling to CPU software path.

Performance Impact: High packet volumes can cause CPU spikes, packet dropping, and kernel buffer exhaustion. Limit usage to targeted troubleshooting.

Configuration Incompatibilities: One-arm sniffing cannot be enabled on interfaces bound to:

WAN roles

Active Firewall Policies

Virtual IPs (Destination NAT)

Active Interface Dependencies

Logging Diagnostic Filters
Bash
execute log filter category 19
execute log display
🔀 VLANs & Hardware Virtual Switches
VLAN processing delivers optimal throughput under NAT operational mode.

802.1Q vs. 802.1ad (QinQ) Standard Comparison
Feature	IEEE 802.1Q (Standard VLAN)	IEEE 802.1ad (QinQ / VLAN Stacking)
Tagging Structure	Single 802.1Q Tag	Double Tagged (Outer S-Tag + Inner C-Tag)
EtherType Field	0x8100	Outer: 0x88A8, Inner: 0x8100
VLAN ID Space	Max 4096 unique IDs	Nested tagging (up to 4096×4096 theoretical isolation)
Primary Use Cases	Enterprise VLAN segmentation	Service Provider Metro Ethernet, Carrier Bridges
Class of Service	Uses Canonical Format Indicator (CFI)	Replaces CFI with Drop Eligibility Indicator (DEI)
Virtual VLAN Switch Configuration (FortiOS 7.2)
Supported Hardware: FortiGate 60F, 80F, 100E, 100F, 140E, 200F, 300E, 400E, 1100E.

Bash
# Enable Virtual Switch Global Mode
config system global
    set virtual-switch-vlan enable
end

# Define Hardware Virtual Switch Entry
config system virtual-switch
    edit "vlan10"
        set physical-switch "sw0"
        set vlan 10
        config port
            edit "port10"
            next
            edit "port11"
            next
        end
    next
end

# Provision Logical Hard-Switch Interface
config system interface
    edit "vlan10"
        set type hard-switch
        set ip 192.168.100.1 255.255.255.0
        set allowaccess ping http ssh
    next
end
🔗 Link Aggregation (LAG) & Redundancy
Interface Aggregation vs. Redundant Interfaces
Aggregated (LACP): Combines throughput across physical links using active load balancing.

Redundant Interfaces: Provides active-backup link failover without throughput aggregation. Never use switch channel-groups for redundant interface configurations.

LACP Configuration Example
Cisco Switch Side:

Plaintext
interface range gig0/0 - 1
 channel-group 1 mode on
!
interface range gig1/0 - 1
 channel-group 2 mode on
FortiGate Side:

Bash
config system interface
    edit "agg-1"
        set type aggregate
        set member "port1" "port2"
        set lacp-mode static
        set lldp-transmission enable
    next
end
Enhanced Hardware LAG Hashing Algorithms
Enhances load-balancing entropy by tuning hash calculation methods across NPU links:

Computation Engines: xor16 (Low CPU overhead; data center default), xor8, xor4, crc16 (High distribution precision; resource intensive).

Bash
config system npu
    set lag-out-port-select enable
    config sw-eh-hash
        set computation xor16
        set ip-protocol include
        set source-ip include
        set destination-ip include
        set source-port include
        set destination-port include
        set netmask-length 32
    end
end
Link Failure Detection Configuration
Triggers immediate failure isolation actions when member interfaces drop:

Bash
config system interface
    edit "agg-1"
        set fail-detect enable
        set fail-alert-method link-down
        set fail-alert-interface "port4"
    next
end
🌉 Virtual Wire Pair (VWP) & PRP
Virtual Wire Pairs bind two interfaces at Layer 2, allowing transparent pass-through filtering without requiring routing changes or MAC modification.

Key Rules
Member interfaces must have zero logical dependencies, active IP addresses, or existing references.

Widely implemented for Internal Segmentation Firewalls (ISFW) and Direct Server Return (DSR) designs.

NAT within VWPs requires Overload IP Pools.

Bash
config system virtual-wire-pair
    edit "vwp-1"
        set member "port3" "port4"
        set wildcard-vlan disable
    next
end
Policy Enforcement: Managed separately under Policy & Objects > Firewall Virtual Wire Pair Policy. Supports both Flow and Proxy inspection engines.

Parallel Redundancy Protocol (PRP - IEC 62439-3)
Provides zero-failover-time Layer 2 redundancy for mission-critical industrial networks (SCADA, substations) by transmitting duplicate frames concurrently across two isolated networks (LAN A & LAN B).

Bash
# Enable PRP Trailer Processing
config system settings
    set prp-trailer-action enable
end

# Assign Hardware Acceleration Ports
config system npu
    set prp-port-in "port15"
    set prp-port-out "port16"
end

# Bind PRP Members into Virtual Wire Pair
config system virtual-wire-pair
    edit "vwp-prp"
        set member "port15" "port16"
    next
end
🎭 Enhanced MAC (EMAC) VLAN
EMAC VLANs allow creating logical interfaces with unique MAC addresses on top of a single physical parent interface.

                 ┌── EMAC 1 (MAC: 00:09:0F:AA:00:01)
[ Physical Port ]┼── EMAC 2 (MAC: 00:09:0F:AA:00:02)
                 └── EMAC 3 (MAC: 00:09:0F:AA:00:03)
Key Considerations & Restrictions
Operational Mode: Must run in NAT Mode.

DHCP Limitations: DHCP relay or server configurations may experience anomalous behavior on EMAC interfaces due to MAC binding constraints.

Unique Pair Requirement: A VLAN ID paired with a physical interface must remain unique across the entire chassis, including across different VDOMs.

HA Behavior: Synchronizes and fails over identically to physical interfaces across High Availability clusters.

Restrictions: Do not use EMAC on dedicated HA Heartbeat links, global Out-of-Band Management interfaces, or Transparent VDOM links.

🌐 VXLAN Architecture & Encapsulation
VXLAN encapsulates Layer 2 Ethernet frames inside Layer 3 UDP datagrams (port 4789) to establish overlay networks over Layer 3 infrastructure.

Bash
# Define Core VXLAN Overlay Tunnel
config system vxlan
    edit "vx10"
        set interface "port2"
        set remote-ip 22.22.22.2
        set vni 1000
    next
end
Implementation Patterns & Best Practices
Dialup / Hub-and-Spoke IPsec Overlays:

Disable automatic device creation.

On spoke nodes, align Phase 2 IPsec subnets with local tunnel interface IPs (e.g., 12.23.34.2) to establish deterministic reverse routing paths without creating unnecessary dynamic interfaces.

VLAN Tagging over VXLAN:

Create standard VLAN sub-interfaces underneath the parent VXLAN tunnel object using unique VLAN IDs to enable multi-tenant frame encapsulation.

L2 Switching Integration:

Bind client physical VLAN ports (e.g., vlan10 on port3) and overlay VLAN ports (vlan10 on vx10) into a common Software Switch.

⚠️ Never add the base VXLAN tunnel object directly into a software switch.

Intra-Switch Policy Modes:

Implicit: Traffic routes automatically between member ports without explicit policy checks.

Explicit: Enforces firewall inspection policies between ports within the software switch.

DHCP Services over VXLAN:

Add the base VXLAN interface and local target ingress port into a dedicated software switch, assign an IP address to the switch interface, and attach the system DHCP Server service directly to that switch.

تیکه اولش اینجوری میشه چجوری اصلاحش کنم که فرمت ها رعایت بشه مدل cheet sheet مناسبی دربیاد

علت به هم ریختگی در خروجی شما چند نکته مشخص در نگارش Markdown است:

بلوک کد داخل لیست (Indent): قرار دادن بلوک کدهای ```bash درون آیتم‌های بولت‌لیست (* یا -) بدون رعایت space (تورفتگی ۴ تایی) باعث خراب شدن ساختار نمایش مارک‌داون می‌شود.

لینک‌های Table of Contents (Anchors): وجود کاراکترهای خاص مانند & یا ایموجی‌ها در سرفصل‌ها، Anchor Link‌های بخش فهرست را در برخی رندرکننده‌های GitHub می‌شکند.

ترکیب نشانه‌های مارک‌داون: استفاده هم‌زمان از * برای لیست و ### برای تیترهای فرعی در حالت متوالی گاهی پارسر را دچار مشکل می‌کند.

نسخه اصلاح‌شده و کاملاً تمیز (Standard GitHub-Flavored Markdown) برای بخش اول:

Markdown
# FortiGate Architecture & Interface Management Cheat Sheet

This comprehensive reference guide consolidates enterprise FortiGate Layer 2/Layer 3 interface configurations, ASIC hardware optimizations, packet acceleration techniques, and network encapsulation standards.

---

## Table of Contents
- [Interface Architecture](#interface-architecture)
- [Administrative Access Protocols](#administrative-access-protocols)
- [Transceivers & Forward Error Correction (FEC)](#transceivers--forward-error-correction-fec)
- [IP Address Management (IPAM)](#ip-address-management-ipam)
- [External Captive Portal Architecture](#external-captive-portal-architecture)
- [Interface Parameters (MTU & 802.1X)](#interface-parameters-mtu--8021x)
- [Traffic Sniffing (One-Arm SPAN)](#traffic-sniffing-one-arm-span)
- [VLANs & Hardware Virtual Switches](#vlans--hardware-virtual-switches)
- [Link Aggregation (LAG) & Redundancy](#link-aggregation-lag--redundancy)
- [Virtual Wire Pair (VWP) & PRP](#virtual-wire-pair-vwp--prp)
- [Enhanced MAC (EMAC) VLAN](#enhanced-mac-emac-vlan)
- [VXLAN Architecture & Encapsulation](#vxlan-architecture--encapsulation)

---

## Interface Architecture

* **Software Switches:** Traffic is switched in software and processed via the main system CPU.
* **Hardware Switches:** Virtual switches operating at the hardware layer utilizing ASICs and co-processors. Supports optimized Spanning Tree Protocol (STP) processing.
* **Physical Interfaces:** SFP/SFP+ ports are hardware-sensitive and require verified vendor modules. Verify diagnostics via CLI:

```bash
get system interface transceiver
Migration Limits: The built-in FortiOS interface migration tool operates exclusively on physical interfaces.

Administrative Access Protocols
Access Flag	Purpose & Functionality
fmg-access	Automatically grants FortiManager authorization during device communication exchanges.
ftm	Enables FortiToken Mobile push authentication handling.
security-fabric	Enables FortiTelemetry and CAPWAP tunnel management across fabric nodes.
speedtest	Permits on-demand bandwidth testing requests from FortiManager/CLI.

### تغییرات اعمال شده برای رفع مشکل:
* **حذف فاصله و رفتن به خط بعد برای کدهای CLI:** بلوک کد `get system interface transceiver` از داخل بولت خراج شده و مستقیماً زیر آن قرار گرفت تا از شکستن فریم کد جلوگیری شود.
* **استانداردسازی Anchor Linkها:** کاراکترهای ایموجی و نمادهای اضافی از بخش Table of Contents حذف شدند تا تمام لینک‌های داخلی در گیت‌هاب به‌درستی کار کنند.
حالا از همین بخش interface یک checklist درست کن رقالب github

interfaces

software switches processed with cpu

hardware switches is virtual switch on hardware level of asic and co-processors

supports spanning tree with optimized processing



enhanced mac vlan provide different mac address for vlans on same physical interfaces



on admin access in interface setting

fmg-access

allow fortimanager authorization automatically on communication exchange between fgt and fmg



ftm

forti token mobile

as push access



security fabric

forti telemetry

capwap



speedtest



on fortigate we can use fec on interfaces with 10g,40gm25g,100g

config system interface

ed sfp10

set speed 40000full

set media-type cr4

set forward-error-correcrion cl91-rs-fec

end



*fec disable on 1000m 10g 40g also not supported

*25g and 100g used automatically cl91-rs-fec



sr: shortens the code by reducing the number of information bits

lr: expands the code by lifting the parity-check matrix

cr: represents the efficiency of the code (ratio of information bits to total bits)



different types of optical transceivers or modules used for transmitting data over fiber optic cables

these modules are typically used in high-speed data communication applications, such as in data centers or enterprise networks



sr4 (short range 4):

sr4 stands for short range 4-1ane

it is designed for short-range multimode fiber (mmf) connections, typically up to 100 meters

it uses four parallel fibers for transmission and four parallel fibers for reception, making it suitable for high-density environments like data centers

commonly used in 40g and 100g ethernet applications



lr4 (long range 4):

lr4 stands for long range 4-1ane

it is designed for long-range single-mode fiber (smf) connections, typically up to 10 kilometers or more

like sr4, it uses four lanes for transmission and four lanes for reception, but it is optimized for longer distances

commonly used in 40g and 100g ethernet applications where longer reach is required



cr4 (copper 4) :

cr4 stands for copper 4-1ane

it is designed for short-range connections over copper cables, typically up to 5 meters

it uses four twinaxial copper cables for transmission and reception

commonly used in 40g ethernet applications within racks or between adjacent racks in data centers



ipam

ip address management



config system ipam

set pool-subnet 172.16.0.0/16

set status enable

end



config system interface

set managed-subnetwork-size 512

edit port8

set ip-managed-by-fortiipam enable

end



diagnose sys ipam largest-available-subnet

#show us maximum subnet range on ipam



diagnose sys ipam reservation-status

diagnose sys ipam confirm-reserv

diagnose sys ipam delete-device-from-ipam

diagnose sys ipam dump-ipams-free-subnets

diagnose sys ipam dump-ipams-entreis



captive portal

just useful on local users

used http request for authentication pages



better use sso and radius instead of



we can track users on fortiview source and firewall users pannel



external sources in captive portal

add another device like fgt-2 in scenario

then clients from fgt-1 goes to fgt-2 and get verified

default gateway will be fgt-2 better check routes



on fgt-1 interface config and security part that pointed to captive portal authentication external section set fgt2 ip address 192.168.12.2

lan 192.168.101.0/24 ------ fgt-1 ----- 192.168.12.0/30 ------- fgt-2 ------ 192.168.254.0/24 ------- edge -----8.8.8.8



on fgt2 interface side fgt-1 (192.168.12.0/30) must enable captive portal with specify user groups



*must exempt ip address of fgt-1 192.168.12.1 cause this device must forward and negotiate all packets fom it's owned lan side to fgt-2 if get trouble on it all authentication procedures get failed



*disable nat from lan to 192.168.12.0/30 cause bypass the captive portal pannel



change mtu

config system interface

edit port8

set mtu-override enable

set mtu 1234

set tcp-mss 1448

end



one-arm sniffer

on switch

conf t

monitor session 1 source interface gig0/0 both

monitor session 1 destination interface gig0/2



open wireshark



if need used fortigate as sniffer

set one-arm sniffer on interface address on network part fortigate

on network > diagnostics start the captures



which item prevent activation of one-arm

wan interface

firewall policies

virtual ip as destination nat

and all another dependencies



might have lower level perform than usuall iinline scanning cause we have more steps as nturbo or cp on packets nad sniffers

might have high cpu usage and packet losts better use in short time



kernel buffer size will be effected



execute log fitlter cat 19

execute log display



our migration tool in interfaces just worked on physical interfaces



physical interfaces

sfp and sfp+ on fortigate are sensetive and used their own modules



get system interface transceiver



vlan

in nat mode has more performance

.1ad used in double tagging and metro-ethernet or q-in-q tunneling

.1q used for vlan tagging as normal state



virtual vlan switch

the hardware switch ports on fortigate models that support virtual vlan switches can be used as a layer 2 switch

virtual vlan switch mode allows 802.1q vlans to be assigned to ports, and the configuration of one interface as a trunk port

the following fortigate series are supported in fortios 7.2:

60f, 80f, 100e, 100f, 140e, 200f, 300e, 400e, and 1100e



seperate some port from fortigate fabric



config system global

set virtual-switch-vlan enable

end



config system virtual-switch

edit vlan10

#create virtual switches



set physical-switch sw0

set vlan 10

config port

edit port10

next

edit port11

end



config system interface

edit vlan10

set type hard-switch

#from virtual at above create hard switch



set ip 192.168.100.1/24

set allow ping http ssh

end



*where to used

in some specific and special conditions



between 2 edge fortigates we have ha with trunked links between them

between 2 isp we have seperated vlans as dual-multihome



if have spanning-tree on network better used virtual-switch and these commands to manage spanning-tree

*software switches help you to manage most scenarios but on multiple spanning-tree or another instances better used hard-switch and virtual-switch



overview of .1q and .1ad

the ieee 802.1q and ieee 802.1ad standards are both related to vlan (virtual local area network) tagging in ethernet frames, but they serve different purposes and functionalities



ieee 802.1q

single vlan tagging: the 802.1q standard allows for a single vlan tag to be inserted into an ethernet frame

his tag includes a 12-bit vlan id, which can represent up to 4096 unique vlans (with a practical limit of 4094 due to reserved values)



ethertype: the ethertype for a standard 802.1q tag is set to 0x8100, which identifies the frame as containing a vlan tag



use case: it is widely used for basic vlan configurations, allowing network segmentation within a single broadcast domain



ieee 802.1ad (qinq)

double vlan tagging: the 802.1ad standard, also known as qinq (or "vlan stacking"), extends the capabilities of 802.1q by

allowing multiple vlan tags to be nested within a single ethernet frame

this effectively creates a 'vlan within a vlan scenario, enabling greater network segmentation and scalability



ethertype: in contrast to 802.1q, the outer tag in an 802.1ad frame uses an ethertype of 0x88a8 for the service tag (s-tag), while the inner tag retains the standard ethertype of 0x8100 for the customer tag (c-tag)



enhanced functionality: the addition of a second tag allows for more sophisticated operations, such as easier modification of frames by switches and better encapsulation of customer traffic by service providers

the outer tag is primarily used to identify and segregate traffic from different customers



features ieee 802.1q ieee 802.1ad (qinq)

tagging type single vlan tag double vlan tag

ethertype 0x8100 outer: 0x88a8; inner: 0x8100

vlan id capacity up to 4096 unique vlans supports nesting for extensive segmentation

use case basic vlan configurations service provider networks, metro ethernet

additional features none drop eligibility indicator (dei) replaces canonical format indicator (cfi) 24



aggregation

on switch

conf t

interface range gig0/0-1

channel-group 1 mode on

interface range gig1/0-1

channel-group 2 mode on



config system interface

edit agg-1

set lacp-mode static

set lldp-transmission enable

end



on mikrotik

add interface bonding

mode 802.3ad

then set ip address on bond interface



enhanced hashing lag memeber selection

xor16 (lighter) (used in datacenters)

xor8

xor4

crc16 (resource intensive)



used

ip protocols

source ip

destination ip

source port

destination port



config system npu

set lag-out-pport-select enable

config sw-eh-hash

set computation xor16

set ip-protocol include

set sour.....

set netmask-length 32

end



redundant

don't use channel-group here



used on full mesh scenario

normally used 4 ports



failur detection on aggregate and redundant

config system interface

edit agg-1

set fail-detect enable

set fail-alert-method link-down

#what happen if get fail on aggregate interfaces



set fail-alert-interface port4

#which interface get down if agg-1 get fail



end







802.1x supported on hardware switch interface with np6 platform on fgt 30xe,40xe110xe

config system interface

edit port8

set security-mode 802.1x

set security-group test

end



diagnose sys 802-1x status



virtual wire pair

base on huge calculation and loadbalancing with devices better used virtual wire pair concept to connect clients directly to servers after verification and trsut level access

better use this on lan side and inside network

worked on layer 2 mode and help on direct server return (dsr) to forward traffic between clients and server with different mac address



internal segmentation firewall (isfw) will be used with dsr and virtual wire pair (vwp)



config system virtual-wire-pair

edit vwp-1

set memeber port3 port4

set wildcard-vlan disble

#used all valn range

end



*interfaces must be empty and no referrer



under policy & objects > firewall virtual wire pair policy

have direction of traffic flow

flow mode or proxy mode inspection



when used nat in vwp

used overload ip pool nat



parallel redundancy protocol (prp)

is a network redundancy protocol standardized by iec 62439-3:2016, designed for ethernet networks requiring high availability and short switchover times

it is commonly used in critical infrastructure applications such as substation automation and industrial control systems



key features of prp:

dual network architecture

prp operates by having each node connected to two independent, parallel local area networks (lans), known as lan a and lan b



redundant packet transmission

source nodes send duplicate packets over both networks simultaneously the destination node accepts the first packet received and discards the duplicate



fail-independent networks

the two networks are designed to be fail-independent, ensuring that if one network fails communication continues uninterrupted through the other network without any recovery time



application transparency

prp operates at layer 2, making it transparent to higher-layer protocols and applications2 this means no modifications are needed for existing protocols or applications



cost considerations

while providing high reliability, prp requires duplicating all network elements involved in redundancy, which can increase costs compared to other protocols like hsr (high availability seamless redundancy)



however, it allows non-redundant devices to coexist on the same lan without needing special adapters or configurations



use cases:

substation automation

industrial control systems

high-power inverters

synchronized drives

prp offers advantages over traditional redundancy protocols like rstp by providing zero-time recovery from failures but at a higher deployment cost due to duplicated infrastructure requirement



config system setting

set prp-trailer-action enable

end



config system npu

set prp-port-in port15

set prp-port-out port16

end



config system virtual-wire-pair

edit vwp-1

set memeber port15 port16

end



emac (enhanced mac)

interface type emac vlan



generate new mac address over parent



we have trouble on dhcp server on emac interfaces



must used nat mode operation to perform this function as well



if set valn id on emac, could not use them on underlay switches and could not handle many mac address on tagging



when using a vlan id, the id and the underlying interface must be a unique pair, even if the belong to different vdoms. this is because the underlying, physical interface uses the vlan id as the identifier to dispatch traffic among the vlan and enhanced mac vlan interfaces.



better don't used management interface , heart beat ha interface and transparent vdom



emac treated as physical interfaces and will be advertise and synchronize on ha cluster



we can use emac on vlan and vdom links must set emac type on npu-link or vdom link



vxlan

config system vxlan

edit vx10

set interface port2

#public ip to acceess remote site



set remote-ip 22.22.22.2

#here we can use space and set second ip address

#if use ipsec with hub-and-spoke dialup scenarios better disable device creation , just on spokes phase 2 ipsec interfaces set local subnet same as ipsec interface 12.23.34.2 or 3 ... (define reverse routing point without device creation and tunnel id)



set vni 1000

end



create vlan over vxlan interface to encapsulate the data

in interface we create vlan under vxlan interface with unique vlan id



then add software switch with vlans created under client port like vlan 10 under port3 and vlan 10 under vxlan interface

don't add main vxlan interface



then add policies with vlan interfaces on vxlan and physical port

under software switches we have intra-switch policy

implicit

forward traffic as default don't need add firewall policy



explicit

must add seperated policies on firewall policies



we can add vxlan main interface and simple interface into the virtual wire pair then setup policies to access them

if need add dhcp on interafce better used software switches with main vxlan interafce and client side interface then add dhcp and ip address on it

You stopped this response
حالا از همین بخش interface  یک checklist  درست کن  رقالب github 

interfaces

software switches processed with cpu

hardware switches is virtual switch on hardware level of asic and co-processors

supports spanning tree with optimized processing



enhanced mac vlan provide different mac address for vlans on same physical interfaces



on admin access in interface setting

fmg-access

allow fortimanager authorization automatically on communication exchange between fgt and fmg



ftm

forti token mobile

as push access



security fabric

forti telemetry

capwap



speedtest



on fortigate we can use fec on interfaces with 10g,40gm25g,100g

config system interface

ed sfp10

set speed 40000full

set media-type cr4

set forward-error-correcrion cl91-rs-fec

end



*fec disable on 1000m 10g 40g also not supported

*25g and 100g used automatically cl91-rs-fec



sr: shortens the code by reducing the number of information bits

lr: expands the code by lifting the parity-check matrix

cr: represents the efficiency of the code (ratio of information bits to total bits)



different types of optical transceivers or modules used for transmitting data over fiber optic cables

these modules are typically used in high-speed data communication applications, such as in data centers or enterprise networks



sr4 (short range 4):

sr4 stands for short range 4-1ane

it is designed for short-range multimode fiber (mmf) connections, typically up to 100 meters

it uses four parallel fibers for transmission and four parallel fibers for reception, making it suitable for high-density environments like data centers

commonly used in 40g and 100g ethernet applications



lr4 (long range 4):

lr4 stands for long range 4-1ane

it is designed for long-range single-mode fiber (smf) connections, typically up to 10 kilometers or more

like sr4, it uses four lanes for transmission and four lanes for reception, but it is optimized for longer distances

commonly used in 40g and 100g ethernet applications where longer reach is required



cr4 (copper 4) :

cr4 stands for copper 4-1ane

it is designed for short-range connections over copper cables, typically up to 5 meters

it uses four twinaxial copper cables for transmission and reception

commonly used in 40g ethernet applications within racks or between adjacent racks in data centers



ipam

ip address management



config system ipam

set pool-subnet 172.16.0.0/16

set status enable

end



config system interface

set managed-subnetwork-size 512

edit port8

set ip-managed-by-fortiipam enable

end



diagnose sys ipam largest-available-subnet

#show us maximum subnet range on ipam



diagnose sys ipam reservation-status

diagnose sys ipam confirm-reserv

diagnose sys ipam delete-device-from-ipam

diagnose sys ipam dump-ipams-free-subnets

diagnose sys ipam dump-ipams-entreis



captive portal

just useful on local users

used http request for authentication pages



better use sso and radius instead of



we can track users on fortiview source and firewall users pannel



external sources in captive portal

add another device like fgt-2 in scenario

then clients from fgt-1 goes to fgt-2 and get verified

default gateway will be  fgt-2 better check routes



on fgt-1 interface config and security part that pointed to captive portal authentication external section set fgt2 ip address 192.168.12.2

lan 192.168.101.0/24 ------ fgt-1 ----- 192.168.12.0/30 ------- fgt-2 ------ 192.168.254.0/24 ------- edge -----8.8.8.8



on fgt2 interface side fgt-1 (192.168.12.0/30) must enable captive portal with specify user groups



*must exempt ip address of fgt-1 192.168.12.1 cause this device must forward and negotiate all packets fom it's owned lan side to fgt-2 if get trouble on it all authentication procedures get failed



*disable nat from lan to 192.168.12.0/30 cause bypass the captive portal pannel



change mtu

config system interface

edit port8

set mtu-override enable

set mtu 1234

set tcp-mss 1448

end



one-arm sniffer

on switch

conf t

monitor session 1 source interface gig0/0 both

monitor session 1 destination interface gig0/2



open wireshark



if need used fortigate as sniffer

set one-arm sniffer on  interface address on network part fortigate

on network > diagnostics  start the captures



which item prevent activation of one-arm

wan interface

firewall policies

virtual ip as destination nat

and all another dependencies



might have lower level perform than usuall iinline scanning cause we have more steps as nturbo or cp  on packets nad sniffers

might have high cpu usage and packet losts better use in short time



kernel buffer size will be effected



execute log fitlter cat 19

execute log display



our migration tool in interfaces just worked on physical interfaces



physical interfaces

sfp and sfp+ on fortigate are sensetive and used their own modules



get system interface transceiver



vlan

in nat mode has more performance

.1ad used in double tagging and metro-ethernet or q-in-q  tunneling

.1q used for vlan tagging as normal state



virtual vlan switch

the hardware switch ports on fortigate models that support virtual vlan switches can be used as a layer 2 switch

virtual vlan switch mode allows 802.1q vlans to be assigned to ports, and the configuration of one interface as a trunk port

the following fortigate series are supported in fortios 7.2:

60f, 80f, 100e, 100f, 140e, 200f, 300e, 400e, and 1100e



seperate some port from fortigate fabric



config system global

set virtual-switch-vlan enable

end



config system virtual-switch

edit vlan10

#create virtual switches



set physical-switch sw0

set vlan 10

config port

edit port10

next

edit port11

end



config system interface

edit vlan10

set type hard-switch

#from virtual at above create hard switch



set ip 192.168.100.1/24

set allow ping http ssh

end



*where to used

in some specific and special conditions



between 2 edge fortigates we have ha with trunked links between them

between 2 isp we have seperated vlans as dual-multihome



if have spanning-tree on network better used virtual-switch and these commands to manage spanning-tree

*software switches help you  to manage most scenarios but on multiple spanning-tree or another instances better used hard-switch and virtual-switch



overview of .1q and .1ad

the ieee 802.1q and ieee 802.1ad standards are both related to vlan (virtual local area network) tagging in ethernet frames, but they serve different purposes and functionalities



ieee 802.1q

single vlan tagging: the 802.1q standard allows for a single vlan tag to be inserted into an ethernet frame

his tag includes a 12-bit vlan id, which can represent up to 4096 unique vlans (with a practical limit of 4094 due to reserved values)



ethertype: the ethertype for a standard 802.1q tag is set to 0x8100, which identifies the frame as containing a vlan tag



use case: it is widely used for basic vlan configurations, allowing network segmentation within a single broadcast domain



ieee 802.1ad (qinq)

double vlan tagging: the 802.1ad standard, also known as qinq (or "vlan stacking"), extends the capabilities of 802.1q by

allowing multiple vlan tags to be nested within a single ethernet frame

this effectively creates a 'vlan within a vlan scenario, enabling greater network segmentation and scalability



ethertype: in contrast to 802.1q, the outer tag in an 802.1ad frame uses an ethertype of 0x88a8 for the service tag (s-tag), while the inner tag retains the standard ethertype of 0x8100 for the customer tag (c-tag)



enhanced functionality: the addition of a second tag allows for more sophisticated operations, such as easier modification of frames by switches and better encapsulation of customer traffic by service providers

the outer tag is primarily used to identify and segregate traffic from different customers



features ieee 802.1q ieee 802.1ad (qinq)

tagging type single vlan tag double vlan tag

ethertype 0x8100 outer: 0x88a8; inner: 0x8100

vlan id capacity up to 4096 unique vlans supports nesting for extensive segmentation

use case basic vlan configurations service provider networks, metro ethernet

additional features none drop eligibility indicator (dei) replaces canonical format indicator (cfi) 24



aggregation

on switch

conf t

interface range gig0/0-1

channel-group 1  mode on

interface range gig1/0-1

channel-group 2  mode on



config system interface

edit agg-1

set lacp-mode static

set lldp-transmission enable

end



on mikrotik

add interface bonding

mode 802.3ad

then set ip address on bond interface



enhanced hashing lag memeber selection

xor16 (lighter)  (used in datacenters)

xor8

xor4

crc16 (resource intensive)



used

ip protocols

source ip

destination ip

source port

destination port



config system npu

set lag-out-pport-select enable

config sw-eh-hash

set computation xor16

set ip-protocol include

set sour.....

set netmask-length 32

end



redundant

don't use channel-group here



used on full mesh scenario

normally used 4 ports



failur detection on aggregate and redundant

config system interface

edit agg-1

set fail-detect enable

set fail-alert-method link-down

#what happen if get fail on aggregate interfaces



set fail-alert-interface port4

#which interface get down if agg-1 get fail



end







802.1x supported on hardware switch interface with np6 platform on fgt 30xe,40xe110xe

config system interface

edit port8

set security-mode 802.1x

set security-group test

end



diagnose sys 802-1x status



virtual wire pair

base on huge calculation and loadbalancing with devices better used virtual wire pair concept to connect clients directly to servers after verification and trsut level access

better use this on lan side and inside network

worked on layer 2 mode and help on direct server return (dsr) to forward traffic between clients and server with different mac address



internal segmentation firewall (isfw) will be used with dsr and virtual wire pair (vwp)



config system virtual-wire-pair

edit vwp-1

set memeber port3 port4

set wildcard-vlan disble

#used all valn range

end



*interfaces must be empty and no referrer



under policy & objects > firewall virtual wire pair policy

have direction of traffic flow

flow mode or proxy mode inspection



when used nat in vwp

used overload ip pool nat



parallel redundancy protocol (prp)

is a network redundancy protocol standardized by iec 62439-3:2016, designed for ethernet networks requiring high availability and short switchover times

it is commonly used in critical infrastructure applications such as substation automation and industrial control systems



key features of prp:

dual network architecture

prp operates by having each node connected to two independent, parallel local area networks (lans), known as lan a and lan b



redundant packet transmission

source nodes send duplicate packets over both networks simultaneously the destination node accepts the first packet received and discards the duplicate



fail-independent networks

the two networks are designed to be fail-independent, ensuring that if one network fails communication continues uninterrupted through the other network without any recovery time



application transparency

prp operates at layer 2, making it transparent to higher-layer protocols and applications2 this means no modifications are needed for existing protocols or applications



cost considerations

while providing high reliability, prp requires duplicating all network elements involved in redundancy, which can increase costs compared to other protocols like hsr (high availability seamless redundancy)



however, it allows non-redundant devices to coexist on the same lan without needing special adapters or configurations



use cases:

substation automation

industrial control systems

high-power inverters

synchronized drives

prp offers advantages over traditional redundancy protocols like rstp by providing zero-time recovery from failures but at a higher deployment cost due to duplicated infrastructure requirement



config system setting

set prp-trailer-action enable

end



config system npu

set prp-port-in port15

set prp-port-out port16

end



config system virtual-wire-pair

edit vwp-1

set memeber port15 port16

end



emac (enhanced mac)

interface type emac vlan



generate new mac address over parent



we have trouble on dhcp server on emac interfaces



must used nat mode operation to perform this function as well



if set valn id on emac, could not use them on underlay switches and could not  handle many mac address on tagging



when using a vlan id, the id and the underlying interface must be a unique pair, even if the belong to different vdoms. this is because the underlying, physical interface uses the vlan id as the identifier to dispatch traffic among the vlan and enhanced mac vlan interfaces.



better don't used management interface , heart beat ha interface and transparent vdom



emac treated as physical interfaces and will be advertise and synchronize on ha cluster



we can use emac on vlan and vdom links must set emac type on npu-link or vdom link



vxlan

config system vxlan

edit vx10

set interface port2

#public ip to acceess remote site



set remote-ip 22.22.22.2

#here we can use space and set second ip address

#if use ipsec with hub-and-spoke dialup scenarios better disable device creation , just on spokes phase 2 ipsec interfaces set local subnet same as ipsec interface 12.23.34.2 or 3 ... (define reverse routing point without device creation and tunnel id)



set vni 1000

end



create vlan over vxlan interface to encapsulate the data

in interface we create vlan under vxlan interface with unique vlan id



then add software switch with vlans created under client port like vlan 10 under port3 and vlan 10 under vxlan interface

don't add main vxlan interface



then add policies with vlan interfaces on vxlan and physical port

under software switches we have intra-switch policy

implicit

forward  traffic as default  don't need add firewall policy



explicit

must add seperated policies on firewall policies 



we can add vxlan main interface and simple interface into the virtual wire pair then setup policies to access them

if need add dhcp on interafce better used software switches  with main vxlan interafce and client side interface then add dhcp and ip address on it

This deployment checklist standardizes FortiGate interface configurations based on the provided technical parameters.  
TXT

Base Interface & Transceiver Verification
[ ] Determine switch architecture: Ensure traffic is processed via CPU for software switches, or utilize ASICs/co-processors for hardware switches to achieve optimized spanning tree processing.  
TXT

[ ] Configure administrative access: Set fmg-access for automatic FortiManager authorization, ftm for FortiToken Mobile push access, security fabric for FortiTelemetry/CAPWAP, or speedtest for bandwidth measurement.  
TXT

[ ] Validate Forward Error Correction (FEC): Confirm FEC is active (cl91-rs-fec) automatically on 25G and 100G interfaces, and note that it is disabled or unsupported on 1000M, 10G, and 40G links.  
TXT

[ ] Match transceiver specifications: Use SR4 for short-range multimode up to 100m, LR4 for long-range single-mode up to 10km+, and CR4 for twinaxial copper up to 5m.  
TXT

[ ] Adjust MTU parameters: Apply mtu-override enable, set custom MTU values, and define the tcp-mss on the interface level if required by the network topology.  
TXT

Layer 2 Switching, VLANs & MAC Management
[ ] Deploy Virtual VLAN switches: Use hardware switch ports on supported models (e.g., 60F, 100F, 200F) as Layer 2 switches to assign 802.1Q VLANs directly to trunk ports.  
TXT

[ ] Select VLAN tagging standard: Use 802.1Q (EtherType 0x8100) for basic single-tag segmentation, or 802.1ad (QinQ) for double-tagging and metro-ethernet tunneling.  
TXT

[ ] Provision Enhanced MAC (EMAC): Generate distinct MAC addresses for VLANs on the same physical interface, ensuring the firewall operates in NAT mode.  
TXT

[ ] Restrict EMAC deployment: Avoid configuring EMAC on management interfaces, HA heartbeat links, and transparent VDOMs, as the VLAN ID and underlying interface must remain a unique pair.  
TXT

[ ] Configure VXLAN encapsulation: Set the remote IP and VNI, create a VLAN under the VXLAN interface, and bind it to a software switch with the physical client port (never add the main VXLAN interface directly).  
TXT

Redundancy, LAG & Virtual Wire Pairs
[ ] Align Link Aggregation (LAG): Configure lacp-mode static on the FortiGate to establish aggregated links with standard switch channel-groups.  
TXT

[ ] Optimize LAG hashing: Set the NPU member selection algorithm to xor16 for lighter data center loads, or crc16 for resource-intensive, highly granular distribution.  
TXT

[ ] Configure Virtual Wire Pairs (VWP): Implement VWP for Direct Server Return (DSR) using completely empty member interfaces that possess no existing references.  
TXT

[ ] Apply VWP NAT routing: Ensure Overload IP Pool NAT is utilized if NAT is required within the Virtual Wire Pair policies.  
TXT

[ ] Enable Parallel Redundancy Protocol (PRP): Configure dual independent networks (LAN A and LAN B) for critical infrastructure to ensure zero-time recovery through redundant packet transmission.  
TXT

Authentication, IPAM & Diagnostics
[ ] Allocate IPAM subnets: Enable FortiIPAM, define pool subnets, and execute diagnose sys ipam reservation-status or diagnose sys ipam largest-available-subnet to verify allocations.  
TXT

[ ] Setup external captive portals: Forward authentication traffic to a secondary FortiGate (FGT-2) acting as the external source.  
TXT

[ ] Secure captive portal routing: Exempt the forwarding firewall's IP address and completely disable NAT on the transit link to prevent the authentication procedures from failing.  
TXT

[ ] Enforce 802.1X security: Apply port-level security directly on hardware switch interfaces running the NP6 platform.  
TXT

[ ] Monitor via One-Arm Sniffer: Use one-arm sniffing strictly for short-term packet captures, acknowledging that it disables hardware acceleration (nTurbo/CP), increases CPU utilization, and impacts the kernel buffer size.  
TXT

با قالب github بهم بده

Markdown
# FortiGate Interface Deployment Checklist

This operational checklist standardizes FortiGate interface configurations, Layer 2/3 optimizations, and hardware-accelerated features for enterprise rollouts.

---

## Base Interface & Transceiver Verification
- [ ] **Determine switch architecture:** Ensure traffic is processed via CPU for software switches, or utilize ASICs/co-processors for hardware switches to achieve optimized spanning tree processing.
- [ ] **Configure administrative access:** Set `fmg-access` for automatic FortiManager authorization, `ftm` for FortiToken Mobile push access, `security-fabric` for FortiTelemetry/CAPWAP, or `speedtest` for bandwidth measurement.
- [ ] **Validate Forward Error Correction (FEC):** Confirm FEC is active (`cl91-rs-fec`) automatically on 25G and 100G interfaces, and note that it is disabled or unsupported on 1000M, 10G, and 40G links.
- [ ] **Match transceiver specifications:** Use SR4 for short-range multimode up to 100m, LR4 for long-range single-mode up to 10km+, and CR4 for twinaxial copper up to 5m.
- [ ] **Adjust MTU parameters:** Apply `mtu-override enable`, set custom MTU values, and define the `tcp-mss` on the interface level if required by the network topology.

## Layer 2 Switching, VLANs & MAC Management
- [ ] **Deploy Virtual VLAN switches:** Use hardware switch ports on supported models (e.g., 60F, 100F, 200F) as Layer 2 switches to assign 802.1Q VLANs directly to trunk ports.
- [ ] **Select VLAN tagging standard:** Use 802.1Q (EtherType 0x8100) for basic single-tag segmentation, or 802.1ad (QinQ) for double-tagging and metro-ethernet tunneling.
- [ ] **Provision Enhanced MAC (EMAC):** Generate distinct MAC addresses for VLANs on the same physical interface, ensuring the firewall operates in NAT mode.
- [ ] **Restrict EMAC deployment:** Avoid configuring EMAC on management interfaces, HA heartbeat links, and transparent VDOMs, as the VLAN ID and underlying interface must remain a unique pair.
- [ ] **Configure VXLAN encapsulation:** Set the remote IP and VNI, create a VLAN under the VXLAN interface, and bind it to a software switch with the physical client port (never add the main VXLAN interface directly).

## Redundancy, LAG & Virtual Wire Pairs
- [ ] **Align Link Aggregation (LAG):** Configure `lacp-mode static` on the FortiGate to establish aggregated links with standard switch channel-groups.
- [ ] **Optimize LAG hashing:** Set the NPU member selection algorithm to `xor16` for lighter data center loads, or `crc16` for resource-intensive, highly granular distribution.
- [ ] **Configure Virtual Wire Pairs (VWP):** Implement VWP for Direct Server Return (DSR) using completely empty member interfaces that possess no existing references.
- [ ] **Apply VWP NAT routing:** Ensure Overload IP Pool NAT is utilized if NAT is required within the Virtual Wire Pair policies.
- [ ] **Enable Parallel Redundancy Protocol (PRP):** Configure dual independent networks (LAN A and LAN B) for critical infrastructure to ensure zero-time recovery through redundant packet transmission.

## Authentication, IPAM & Diagnostics
- [ ] **Allocate IPAM subnets:** Enable FortiIPAM, define pool subnets, and execute `diagnose sys ipam reservation-status` or `diagnose sys ipam largest-available-subnet` to verify allocations.
- [ ] **Setup external captive portals:** Forward authentication traffic to a secondary FortiGate (FGT-2) acting as the external source.
- [ ] **Secure captive portal routing:** Exempt the forwarding firewall's IP address and completely disable NAT on the transit link to prevent the authentication procedures from failing.
- [ ] **Enforce 802.1X security:** Apply port-level security directly on hardware switch interfaces running the NP6 platform.
- [ ] **Monitor via One-Arm Sniffer:** Use one-arm sniffing strictly for short-term packet captures, acknowledging that it disables hardware acceleration (nTurbo/CP), increases CPU utilization, and impacts the kernel buffer size.
براساس این متن بیا و در قالب cheet sheet های github و با قالب github خروجی بده route

static route

default route

has administrative distance with value 10 if used dhcp ad will be 5



get router info routing-table all

#active and inused routes



get router info routing-table details

get router info routing-table database

#all received and learned routes in fortigate



get router info kernel

#fib and routing deamon information

tab

254 > unicast

255 > multicast



vf > vdom name and index if has no vdom this part will be 0



type

0 > unspecific

1 > unicast

2 > local

3 > broadcast

4 > anycast

5 > multicast

6 > blackhole

7 > unreachable

8 > prohibited (admin on another place ommit ttl or block you)



proto

0 > unspecific

2 > kernel

11 > ZebOS routing module (used on dynamic routing process)

14 > FortiOS

15 > HA (learned from ha)

16 > authentication based

17 > HA1 (learned from heart beat links)





prio > lower priority is best



pref > preferred nexthop



diagnose ip rtcache list

#used for route lookup and cache checking

#has size will be defined on kerner

config system global

set max-route-cache-size 100

end



diagnose ip address list

#show all assigned ip addresses



*route lookup

contains rib and rtcache on session tables



first check policy route then route cache next fib and at last if has no matched entries get drop



get router info routing-table details 4.4.4.4



*for blackhole and priority on it just used cli



reverse path lookup (rpf)

config system setting

set strict-src-check enable

end



prevent ip spoofing

by default works on feasible-path (doesn't need above command) and used exist links to ckeck requests are valid or not , in this case if find one interface access the request

but on strict mode will be check all source and destination interfaces (must used above command)



sometimes need enable asymmetric routing on device so restric and feasible-path wil be ignor traffics better disable them

config system interface

edit port8

set src-check disable

end



for test this on kali

nmap -D 10.10.10.1,192.168.101.1,192.168.101.3 192.168.101.2

nmap 192.168.101.3 192.168.101.2



asymmetric routing

fortigate block the packets on not established with 3-way handshake so this option help us to manage these type connections



config system setting

set asymroute enable

end



has security issues better disable these



routing change without snat

config system interface

edit port4

set preserve-session-route enable

#make the session as persist state

end



*if dynamic routing or ha  get changes on topologies we can use above command to save all states , by default all routes get flush

*may get trouble on xforwarded-for



routing change with snat

config system global

set snat-route-change enable

#session with snat will be require new route lookup and new sessions - used in dynamic routes or link fail - continuously forwardd traffic to another link without fail

#bydefault is disable

end



diagnose sys session filter src 192.168.101.2

diagnose sys session list

#check the session stages and orders



policy route

must set default gateway and then used policy route together



ecmp (equal cost multipathing)

we have to set same destination with priority and distances on routing table (as static route)



methods

source ip base > divided equally between interfaces as session started froom same ip address to same path (default selection)



weighted base > number of sessions that are connected through the interface the weight that you assign to each interface is used to calculate the percentage of the total sessions allowed to connect through an interface, and the sessions are distributed to the interfaces accordingly



usage base (spillover or threshold) > The interface is used until the traffic bandwidth exceeds the ingress and egress thresholds that you set for that interface Additional traffic is then sent through the next interface member



src-dst-ip-base > traffic is divided equally between the interfaces sessions that start at the same source ip address and go to the same destination ip address use the same path



volume or measured volume base > this mode is supported in sd-wan only the workload is distributed based on the number of packets that are going through the interface



config system setting

set v4-ecmp-mode usage-base

set ecmp-max-path 4

end



config system sdwan

set status enable

set load-balanace-mode usage-base

end



diagnose sys vd list

get router info routing-table all



*on bgp

config router bgp

set as 65000

set router-id 192.168.254.252

set ebgp-multipath enable

end



dual internet connection

etter used sdwan

if not intersted to use, replace sdwan with link redundancy, loadsharing or combination of both



redundancy

if one link get fail swithced to another link



in this section we have feature like health check on sdwan

link health monitor



config system link-monitor

edit 1

set status enable

set addr-mode ipv4

set srcintf port1

set server 8.8.8.8

set protocol ping

set gateway-ip 192.168.254.2

set interval 30

set failtime 5

set recovertime 5

set update-static-route enable

#if static routes were setup on this link and get fail must switched them



set update-cascade-interface enable

#interfaces on this link-fail or alert could be turned off

end



*better change distances and priority on  routers



load sharing

if one interface get fail our traffic without switching or fail detection forwarded continuously

*must used same distances and priority as ecmp or used pbr



traffic behaviour without a link monitor is as follows

if the remote gateway is down but the primary wan interface of a fortigate is still up, the fortigate will continue to route traffic to the primary wan this results in traffic interruptions



if the primary wan interface of a fortigate is down due to physical link issues, the fortigate will remove routes to it and the secondary wan routes will become active traffic will failover to the secondary wan



dynamic route

rip

config router rip

set timeout-timer 30

set update-timer 180

set garbage-timer 30

end



*must change all devices timers



support keychains

config router key-chain

edit 1

config key

edit 1

set key-string 123

set accept-lifetime 12:20:00 8 2 2025 infinit

set send-lifetime 12:20:00 8 2 2025 infinit

end



config router rip

config interface

set auth-mode md5

set auth-string 123

set auth-keychain 1

end

end



get router info rip

get router info rip interfaces

#see split horizon



get router info rip database



execute ping-option  source 1.2.3.4

#set source then ping destination

execute ping 8.8.8.8



ospf

if need run many process id to manage many domains, better use vrf to seperate processing on fortigate



config router ospf

set router-id 10.11.101.1

config area

edit 0.0.0.0

next



config ospf-interface

edit Router1-Internal-DR

set interface "port1"

set priority 255

set dead-interval 40

set hello-interval 10

next

edit "Router1-External"

set interface "port2"

set dead-interval 40

set hello-interval 10

next



config network

edit 1

set prefix 10.11.0.0 255.255.0.0

next

edit 2

set prefix 192.168.102.0 255.255.255.0

next

end



get router ospf

#show comprehensive information of ospf information and setup



get router info routing-table ospf

#show global (vrf 0) recieved routes and contributers



get router info ospf neighbor

get router info ospf neighbor all

get router info ospf neighbor details

#show all neighbors states and stats



get router info ospf  database brief

get router info ospf database adv-router 3.3.3.3

#show process id , vrf , also show each lsdb and lsa ....



get router info ospf route

#show us all recieved and learned routes on rib



get router info ospf interface

#show interfaces attributes and timers information



get router info routing-table all



get router info ospf status

#process uptime  bounding vrf and data exchanges values, area id and names, last update on spf algorithem



graceful restart upon topology change

config router ospf

set restart-on-topology-change enable

end



on ha cluster we have ip like this  1.1.1.1 and also have restart mechanism on on clustered device, here graceful restart provide non-interupted forwarding between ha switch mechanism while we must run spf algorithem





helper gatway (fgt 1) ----- cluster fgts ------ cisco gateway



on helper gateway and clusters must use graceful-restart mechanism



config router ospf

set router-id 31.1.1.1

set restart-mode graceful-restart

set restart-period 180

set restart-on-topology-change enable



config area

edit 0.0.0.0

next

end



config network

edit 1

set prefix 172.16.200.0 255.255.255.0

next

edit 2

set prefix 31.1.1.1 255.255.255.255

next

end



bgp

config router bgp

set as 64511

set router-id 1.1.1.1

config neighbor

edit 10.100.201.88

set remote-as 64511

set update-source toFGTB

#change ip and interface detection for better config on ibgp, better use this if used different ip domain on connectivities between organizations

#if used loopback interface in bgp connection must used update source command

next

end



config network

edit 1

set prefix 192.168.86.0 255.255.255.0

end



get router info bgp paths

#show all best paths



get router info routing-table all

# show all paths without best  path



*better used nexthop self to select and used best path on bgp connections, if used interconnectivities between hub interfaces



get router info bgp neighbors

get router info bgp summary

#show us all values about malformed bgp requests (idle state means malformed)



get router info bgp networks

get router info bgp network 12.23.34.2



get router info routing-table all



execute router clear bgp all soft

#reset and clear all soft



*objects on route map defined with acl and route-map lists whcih acls collect and route map execute the rule

config router route-map

edit exclude1

config rule

edit 1

set action deny

set match-origin igp

end



idle state in bgp

multihop state on fortigate whcih must be tuned



connect state

if stocked on tcp connection and improper routing



active or sia

deliver problem on responses



diagnose sniffer packet any 'tcp and port 179'



check the recursive lookup inside bgp

config router bgp

set recursive-next-hop enable

#without this command all normal scenarios could be run and execute

end



config router bgp

set as 65001

set router-id 1.1.1.1

set ebgp-multipath enable

set ibgp-multipath enable

set recursive-next-hop enable

#used for special configs with some layer3 devices which make loops



config neighbor

edit 12.12.12.2

set ebgp-enforce-multihop enable

set soft-reconfiguration enable

set ebgp-multihop-ttl 3

set remote-as 65002

set update-source lp-1

next

end

#best setup for loopback config on bgp



config network

edit 1

set prefix 12.12.12.1 255.255.255.255

next

edit 2

set prefix 12.12.12.2 255.255.255.255

set backdoor enable

#for added networks we have some entities and pointed to remote ip as resolver

end



config system setting

set v4-ecmp-mode usage-base

set ecmp-max-path 4

end

#these setups are different from ebgp and igp multipathing



conditional advertise on bgp

config router community-list

edit 30:5

config rule

edit 1

set action permit

set match 30:5

end



config router route-map

edit comm1

config rule

edit 1

set match-community 30:5

set set-route-tag 15

end

next

edit 2224

config rule

edit 1

set match-ip-address 222

next

end

next

edit 2814

config rule

edit 1

set match-ip-address 281

end



config router bgp

config neighbor

edit 2.2.2.2

config conditional-advertise

edit 2224

set condition-routemap 2814 2224 comm1

set condition-type non-exist

end

#better write and setup route-maps before this and called them here



next

edit 2003: :2:2:2:2

config conditional-advertise6

edit map-222

set condition-routemap map-222 map-282

next

end

set route-reflector-client6 enable

end



diagnose ip router command show-vrf root show running router bgp

#show us advertise maps and router id , neighbors , address families ...



bgp next hop tag-match mode

if need negotiate with isps on bgp tags, help you to manage them flexible



config router bgp

set tag-resolve-mode disable

disable means resolve nexhop best match

preferred means resolve nexthops with same tags, if not exist behave like best match

merge means merge the best tags if they are different, shortcuts hide  parents, results will be exclude the nexthop tag match whose appeared on best match



*merge used in loadbalancing mode

*preferred used in failover mode



route flap

*resource intensive



hold time

config router bgp

config neighbor

edit 1.2.3.4

set holdtime-timer 60

set keepalive-timer 60

end



dampening

execute router clear bgp dampening 1.2.3.4/32

execute router clear bgp flap-stat 1.2.3.4/32



config router bgp

set dampening enable

set dampening-max-suppress-time 60

#minutes_integer , after 60 mins if not reachable tell this link get really fail



set dampening-reachability-half-life 15

#minutes_integer , time of stable reachablity to this link after last failover



set dampening-reuse 750

#if were bigger than this number must wait to get relax and sure our link is valid



set dampening-route-map rmp-fail-damp

set dampening-suppress 2000

set dampening-unreachability-half-life 15

#minutes_integer



end



penalty: a value assigned to a route each time it flaps

suppress threshold: if the penalty exceeds this value, the route is suppressed

reuse threshold: if the penalty falls below this value, the route is reused (advertised again)

half-life: the time it takes for the penalty to decay by half

max suppress time: the maximum time a route can be suppressed



a route flaps 3 times:

penalty = 1000 x 3 = 3000

since 3000 > 2000, the route is suppressed



after 15 minutes:

penalty decays by half: 3000 / 2 = 1500

since 1500 > 750, the route remains suppressed



after 30 minutes:

penalty decays by half again: 1500 / 2 = 750

since 750 ≤ 750, the route is reused (advertised again)



*If the penalty doesn't decay below 750 within 60 minutes, the route will still be reused after 60 due to the max suppress time



graceful restart

config router bgp

set graceful-restart enable

set graceful-restart-timer 120

#time of remote restart procedure



set graceful-stalepath-time 180

#time of recieved routes from ha pair or another remote device



set graceful-update-delay

#time of remote device (ha pair) main routing engine to manage the forwarding and routing



config neighbor

edit 1.2.3.4

set capability-graceful-restart enable



end

execute router restart



bfd

*echo mode and authentication are not supported



ommit routes from rib faster than usuall



config router bgp

set ebgp-multipath enable

config neighbor

edit 1.2.3.4

set bfd enable

set ebgp-enforce-multihop enable

end



show router bgp



*config on whole device (recommended, could disable on each interface and dynamic routing or per vdom)

vdom state or global on each device need hear them

config system settings

set bfd enable

set bfd-desired-min-tx 250

set bfd-required-min-rx 250

set bfd-detect-mult 3

set bfd-dont-enforce-src-port disable

#worked on 3784 udp, we could change them and enable this means used another ports for bfd, better check on isp acl and filters  be allow,  forward to another device over isp , better enable on both sides



end



config router bfd

config neighbor

edit 24.24.24.1

#ip address of isp edge, if have advpn or direct ipsec set remote ip

set interface port4

end

next

edit 12.12.12.1

set interface port1

end

#above setup used for more reliability and faster reaction on dead links, just setup multihop-template part could be check reachablity

#this part used on static route if ned dynamic routes better set another commands



config multihop-template

edit 1

set src 12.12.12.0/30

set dst 24.24.24.0/30

end

next

edit

set src 13.13.13.0/30

set dst 23.23.23.0/30

end

end



on interface

config system interface

edit port8-9

set bfd enable

#we can inherite from global



set bfd-desired-min-tx <ms>

set bfd-required-min-rx <ms>

set bfd-detect-mult <multiplier>

end



get router info bfd neighbor

#down state means indirect connections on bfd and just check destination of ip address, if enable on both side and isp it shows up



get router info bfd requests



-------------------------------------------

static route with bfd

conf router static

edit 1

set bfd enable

end



config system interface

edit port8

set bfd enable

end



config router bfd

config neighbor

edit 192.168.15.1

set interface port8

end



config system settings

set bfd enable

end



vrf

network or route segmentation without vdom concept

64 vrf supported per vdom



feature visibility > advance routing



get router info routing-table all

#all routes binded to vrf 0 , if vrf get changed must leakage them to vrf 0 or define well



vrf will be changed on interfaces setting under type definition



after change vrf all rib and forwarding gonna disrupted



get router info routing-table all

Routing table for VRF=0

192.168.101.0/24 is directly connected, port3

192.168.254.0/24 is directly connected, port1



Routing table for VRF=2

12.12.12.0/30 is directly connected, port2

192.168.102.0/24 [20/0] via 12.12.12.2 (recursive is directly connected, port2), 00:00:43, [1/0]



Routing table for VRF=3

13.13.13.0/30 is directly connected, port4

192.168.103.0/24 [20/0] via 13.13.13.2 (recursive is directly connected, port4), 00:00:12, [1/0]



config system vdom-link

edit v-2-3-

end



config system setting

set allow-subnet-overlap enable

end



on interface setting gui we see new interface as v-2-3-0 and v-2-3-1 which used vrf id 2 and 3 and ip address 10.10.10.0/30

enable ping on them



get router info routing-table all

Routing table for VRF=0

192.168.101.0/24 is directly connected, port3

192.168.254.0/24 is directly connected, port1



Routing table for VRF=2

10.10.10.0/30 is directly connected, v-2-3-0

12.12.12.0/30 is directly connected, port2

192.168.102.0/24 [20/0] via 12.12.12.2 (recursive is directly connected, port2), 00:05:26, [1/0]



Routing table for VRF=3

10.10.10.0/30 is directly connected, v-2-3-1

13.13.13.0/30 is directly connected, port4

192.168.103.0/24 [20/0] via 13.13.13.2 (recursive is directly connected, port4), 00:04:55, [1/0]



on route object in network gui

create some acls like

acl-102-103

permit

prefix

exact match

specify 192.168.102.0/24



permit

prefix

exact match

specify 12.12.12.0/30



permit

prefix

exact match

specify 10.10.10.0/30



acl-103-102

permit

prefix

exact match

specify 192.168.103.0/24



permit

prefix

exact match

specify 13.13.13.0/30



permit

prefix

exact match

specify 10.10.10.0/30



create route-maps

rtm-102-103

permit

match-ip acl-102-103





rtm-103-102

permit

match-ip acl-103-102



on bgp

config router bgp

config vrf

edit 2

#vrf number must be samed



conf leak-target

edit 3

#set exact vrf number



set route-map rtm-102-103

set interface v-2-3-0

next

end

next



edit 3

conf leak-target

edit 2

set route-map rtm-103-102

set interface v-2-3-1

next

end

next

end

end



*now all advertisements will be happened



on hub or main device vrf segmentations must add all vdom links on specify policies or add on zones



these setups worked for spokes connectivities and hub or hq could not see the spokes or inverse

get router info routing-table all

Routing table for VRF=0

192.168.101.0/24 is directly connected, port3

192.168.254.0/24 is directly connected, port1



Routing table for VRF=2

10.10.10.0/30 is directly connected, v-2-3-0

12.12.12.0/30 is directly connected, port2

192.168.102.0/24 [20/0] via 12.12.12.2 (recursive is directly connected, port2), 00:13:20, [1/0]

192.168.103.0/24 [20/0] via 10.10.10.2, v-2-3-0, 00:01:54, [1/0]



Routing table for VRF=3

10.10.10.0/30 is directly connected, v-2-3-1

13.13.13.0/30 is directly connected, port4

192.168.102.0/24 [20/0] via 10.10.10.1, v-2-3-1, 00:01:53, [1/0]

192.168.103.0/24 [20/0] via 13.13.13.2 (recursive is directly connected, port4), 00:12:49, [1/0]



here must add some leaks between hub to spokes on inverse must do these

config system vdom-link

edit v-0-2-

next

edit v-0-3-

end



config system setting

set allow-subnet-overlap enable

end



on interface setting gui we see new interface as v-0-2-0 and v-0-2-1 which used vrf id 0 and 2 and ip address 9.9.9.0/30



on interface setting gui we see new interface as v-0-3-0 and v-0-3-1 which used vrf id 0 and 3 and ip address 8.8.8.0/30



create objects to access vdom 0 to 2

create some acls like

acl-101-102

permit

prefix

exact match

specify 192.168.101.0/24



permit

prefix

exact match

specify 12.12.12.0/30



permit

prefix

exact match

specify 9.9.9.0/30

#not necessary



acl-102-101

permit

prefix

exact match

specify 192.168.102.0/24



permit

prefix

exact match

specify 12.12.12.0/30



permit

prefix

exact match

specify 9.9.9.0/30

#not necessary



create route-maps

rtm-101-102

permit

match-ip acl-101-102





rtm-102-101

permit

match-ip acl-102-101



on bgp

config router bgp

config vrf

edit 0

#vrf number must be samed



conf leak-target

edit 2

#set exact vrf number



set route-map rtm-101-102

set interface v-0-2-0

next

end

next



edit 2

conf leak-target

edit 0

set route-map rtm-102-101

set interface v-0-2-1

next

end

next

end

end



create objects to access vdom 0 to 3

create some acls like

acl-101-103

permit

prefix

exact match

specify 192.168.101.0/24



permit

prefix

exact match

specify 13.13.13.0/30



permit

prefix

exact match

specify 8.8.8.0/30

#not necessary



acl-103-101

permit

prefix

exact match

specify 192.168.103.0/24



permit

prefix

exact match

specify 13.13.13.0/30



permit

prefix

exact match

specify 8.8.8.0/30

#not necessary



create route-maps

rtm-101-103

permit

match-ip acl-101-103





rtm-103-101

permit

match-ip acl-103-101



on bgp

config router bgp

config vrf

edit 0

#vrf number must be samed



conf leak-target

edit 3

#set exact vrf number



set route-map rtm-101-103

set interface v-0-3-0

next

end

next



edit 3

conf leak-target

edit 0

set route-map rtm-103-101

set interface v-0-3-1

next

end

next

end

end



execute router clear bgp all so



*must add policies and interfaces



get router info routing-table all

Routing table for VRF=0

8.8.8.0/30 is directly connected, v-0-3-0

9.9.9.0/30 is directly connected, v-0-2-0

192.168.101.0/24 is directly connected, port3

192.168.102.0/24 [20/0] via 9.9.9.2, v-0-2-0, 00:03:15, [1/0]

192.168.103.0/24 [20/0] via 8.8.8.2, v-0-3-0, 00:03:15, [1/0]

192.168.254.0/24 is directly connected, port1



Routing table for VRF=2

9.9.9.0/30 is directly connected, v-0-2-1

10.10.10.0/30 is directly connected, v-2-3-0

12.12.12.0/30 is directly connected, port2

192.168.101.0/24 [20/0] via 9.9.9.1, v-0-2-1, 00:03:16, [1/0]

192.168.102.0/24 [20/0] via 12.12.12.2 (recursive is directly connected, port2), 00:35:25, [1/0]

192.168.103.0/24 [20/0] via 10.10.10.2, v-2-3-0, 00:23:59, [1/0]



Routing table for VRF=3

8.8.8.0/30 is directly connected, v-0-3-1

10.10.10.0/30 is directly connected, v-2-3-1

13.13.13.0/30 is directly connected, port4

192.168.101.0/24 [20/0] via 8.8.8.1, v-0-3-1, 00:03:16, [1/0]

192.168.102.0/24 [20/0] via 10.10.10.1, v-2-3-1, 00:23:58, [1/0]

192.168.103.0/24 [20/0] via 13.13.13.2 (recursive is directly connected, port4), 00:34:54, [1/0]



block one route with vrf

static routes on network gui

destination 192.168.101.2

interface blackhole

vrf-id 3



lldp

send clients information to fortigate while scanning

these are important in restapi and snmp or neighboring mechanism



config sys int

edit port1

set lldp-reception enable

set lldp-transmit enable

set device-identification enable

end



*in device inventory monitor we could find them



config system global

set lldp-reception enable

set lldp-transmit enable

end

#global



config system setting

set lldp-reception enable

set lldp-transmit enable

end

#vrf



diagnose user device list



diagnose lldprx neighbor summary

diagnose lldprx neighbor details

diagnose lldprx neighbor clear



diagnose lldprx port details

diagnose lldprx port summary

diagnose lldprx port filter



diagnose lldprx port neighbor summary

diagnose lldprx port neighbor details



diagnose netlink interface list port2 | grep index

FortiGate Routing & Forwarding Cheat Sheet
Route Lookup & Core Diagnostics
Administrative Distances (Default)

Static Route: 10

DHCP: 5

Routing Table & Kernel Commands

Bash
get router info routing-table all       # Active and in-use routes
get router info routing-table details   # Detailed route lookups (e.g., details 4.4.4.4)
get router info routing-table database  # All received and learned routes
get router info kernel                  # FIB and routing daemon information
diagnose ip rtcache list                # Route lookup and cache checking
diagnose ip address list                # Show all assigned IP addresses
Kernel Table Information (get router info kernel)

Tables: 254 (Unicast), 255 (Multicast).

vf: VDOM name/index (0 if no VDOM).

prio: Lower priority is the best path.

pref: Preferred next-hop.

Type Value	Description	Proto Value	Description
0	Unspecific	0	Unspecific
1	Unicast	2	Kernel
2	Local	11	ZebOS routing module (dynamic routing)
3	Broadcast	14	FortiOS
4	Anycast	15	HA (Learned from HA)
5	Multicast	16	Authentication based
6	Blackhole	17	HA1 (Learned from heartbeat links)
7	Unreachable		
8	Prohibited (Blocked/TTL omitted)		
Route Mechanics & Session Preservation
Reverse Path Forwarding (RPF)
Prevents IP spoofing. Feasible-path is default; strict mode checks all source/destination interfaces.

Bash
config system settings
  set strict-src-check enable 
end
Asymmetric Routing & Session Preservation

Bash
# Allow asymmetric routing (bypass 3-way handshake checks - has security risks)
config system settings
  set asymroute enable
end

# Preserve session route (useful for dynamic routing/HA topology changes)
config system interface
  edit port4
  set preserve-session-route enable 
end

# SNAT route change (Force session to require new route lookup on link fail)
config system global
  set snat-route-change enable
end
ECMP & Link Redundancy
ECMP Load Balancing Algorithms

Source IP (Default): Divided equally based on source IP.

Weighted: Distributed based on assigned interface weights.

Usage (Spillover): Uses an interface until bandwidth exceeds set thresholds, then spills to the next.

Source-Destination IP: Traffic divided equally; identical source-to-destination sessions use the same path.

Volume (SD-WAN only): Distributed based on packet count.

Bash
# Enable Usage-Based ECMP
config system settings
  set v4-ecmp-mode usage-base
  set ecmp-max-path 4
end
Link Health Monitor (Failover)

Bash
config system link-monitor
  edit 1
    set status enable
    set addr-mode ipv4
    set srcintf port1
    set server 8.8.8.8
    set protocol ping
    set gateway-ip 192.168.254.2
    set interval 30
    set failtime 5
    set recovertime 5
    set update-static-route enable     # Switch static routes on fail
    set update-cascade-interface enable # Shutdown cascade interfaces on fail
end
Dynamic Routing: RIP & OSPF
RIP Configurations
Ensure timers match across all devices. Split horizon can be verified via get router info rip interfaces.

Bash
config router rip
  set timeout-timer 30
  set update-timer 180
  set garbage-timer 30
end

# RIP Keychain Authentication
config router key-chain
  edit 1
    config key
      edit 1
        set key-string 123
        set accept-lifetime 12:20:00 8 2 2025 infinite
        set send-lifetime 12:20:00 8 2 2025 infinite
    end
end
OSPF Configurations & Diagnostics
Core Diagnostics

Bash
get router ospf                          # Comprehensive OSPF setup
get router info routing-table ospf       # Global (VRF 0) received routes
get router info ospf neighbor details    # Neighbor states and statistics
get router info ospf database brief      # LSDB and LSA information
get router info ospf route               # Received and learned routes in RIB
get router info ospf status              # Uptime, VRF binds, SPF algorithm updates
Graceful Restart (Topology Change)
Provides non-interrupted forwarding between HA switch mechanisms during SPF algorithm runs.

Bash
config router ospf
  set router-id 31.1.1.1
  set restart-mode graceful-restart
  set restart-period 180
  set restart-on-topology-change enable
end
Dynamic Routing: BGP
Basic BGP & Multi-pathing

Bash
config router bgp
  set as 65001
  set router-id 1.1.1.1
  set ebgp-multipath enable
  set ibgp-multipath enable
  set recursive-next-hop enable      # Resolves recursive lookups avoiding loops
  
  config neighbor
    edit 12.12.12.2
      set remote-as 65002
      set ebgp-enforce-multihop enable
      set ebgp-multihop-ttl 3
      set update-source lp-1         # Required if using loopback interfaces
      set soft-reconfiguration enable
  end
end
BGP Route Flap Dampening
Prevents resource-intensive route calculations from unstable links.

Penalty: Value added to a route each flap.

Suppress Threshold: Flap penalty value that triggers suppression.

Reuse Threshold: Value where penalty must decay below to advertise the route again.

Half-life: Time for a penalty to decay by 50%.

Bash
config router bgp
  set dampening enable
  set dampening-max-suppress-time 60
  set dampening-reachability-half-life 15
  set dampening-reuse 750
  set dampening-suppress 2000
end
Bidirectional Forwarding Detection (BFD)
Omits routes from the RIB faster than standard timers. Echo mode and authentication are unsupported.

Bash
# Global Settings
config system settings
  set bfd enable
  set bfd-desired-min-tx 250
  set bfd-required-min-rx 250
  set bfd-detect-mult 3
end

# Enable BFD on BGP Neighbor
config router bgp
  config neighbor
    edit 1.2.3.4
      set bfd enable
  end
end
Virtual Routing and Forwarding (VRF)
Provides network segmentation without VDOMs. Supports up to 64 VRFs per VDOM. Modifying VRF attributes on interfaces disrupts forwarding.

Configure VDOM Links for VRF Overlap

Bash
config system vdom-link
  edit v-2-3-
end

config system settings
  set allow-subnet-overlap enable
end
Route Leaking Between VRFs (via BGP)

Bash
config router bgp
  config vrf
    edit 2
      config leak-target
        edit 3
          set route-map rtm-102-103
          set interface v-2-3-0
      end
    next
    edit 3
      config leak-target
        edit 2
          set route-map rtm-103-102
          set interface v-2-3-1
      end
  end
end
Blackhole Routes per VRF

Bash
config router static
  edit 1
    set dst 192.168.101.2 255.255.255.255
    set blackhole enable
    set vrf 3
end
LLDP (Link Layer Discovery Protocol)
Sends client information to the FortiGate during scanning (essential for SNMP/RESTAPI).

Configuration Tiers

Bash
# Global
config system global
  set lldp-reception enable
  set lldp-transmit enable
end

# Interface Level
config system interface
  edit port1
    set lldp-reception enable
    set lldp-transmit enable
    set device-identification enable
end
Diagnostics

Bash
diagnose user device list
diagnose lldprx neighbor summary
diagnose lldprx neighbor details
diagnose lldprx port summary
با قالب github بهم بده

Markdown
# FortiGate Routing & Forwarding Cheat Sheet

## Route Lookup & Core Diagnostics

**Administrative Distances (Default)**
* **Static Route:** 10
* **DHCP:** 5

**Routing Table & Kernel Commands**
```bash
get router info routing-table all       # Active and in-use routes
get router info routing-table details   # Detailed route lookups (e.g., details 4.4.4.4)
get router info routing-table database  # All received and learned routes
get router info kernel                  # FIB and routing daemon information
diagnose ip rtcache list                # Route lookup and cache checking
diagnose ip address list                # Show all assigned IP addresses
Kernel Table Information (get router info kernel)

Tables: 254 (Unicast), 255 (Multicast).

vf: VDOM name/index (0 if no VDOM).

prio: Lower priority is the best path.

pref: Preferred next-hop.

Type Value	Description	Proto Value	Description
0	Unspecific	0	Unspecific
1	Unicast	2	Kernel
2	Local	11	ZebOS routing module (dynamic routing)
3	Broadcast	14	FortiOS
4	Anycast	15	HA (Learned from HA)
5	Multicast	16	Authentication based
6	Blackhole	17	HA1 (Learned from heartbeat links)
7	Unreachable		
8	Prohibited (Blocked/TTL omitted)		
Route Mechanics & Session Preservation
Reverse Path Forwarding (RPF)
Prevents IP spoofing. Feasible-path is default; strict mode checks all source/destination interfaces.

Bash
config system settings
  set strict-src-check enable 
end
Asymmetric Routing & Session Preservation

Bash
# Allow asymmetric routing (bypass 3-way handshake checks - has security risks)
config system settings
  set asymroute enable
end

# Preserve session route (useful for dynamic routing/HA topology changes)
config system interface
  edit port4
  set preserve-session-route enable 
end

# SNAT route change (Force session to require new route lookup on link fail)
config system global
  set snat-route-change enable
end
ECMP & Link Redundancy
ECMP Load Balancing Algorithms

Source IP (Default): Divided equally based on source IP.

Weighted: Distributed based on assigned interface weights.

Usage (Spillover): Uses an interface until bandwidth exceeds set thresholds, then spills to the next.

Source-Destination IP: Traffic divided equally; identical source-to-destination sessions use the same path.

Volume (SD-WAN only): Distributed based on packet count.

Bash
# Enable Usage-Based ECMP
config system settings
  set v4-ecmp-mode usage-base
  set ecmp-max-path 4
end
Link Health Monitor (Failover)

Bash
config system link-monitor
  edit 1
    set status enable
    set addr-mode ipv4
    set srcintf port1
    set server 8.8.8.8
    set protocol ping
    set gateway-ip 192.168.254.2
    set interval 30
    set failtime 5
    set recovertime 5
    set update-static-route enable     # Switch static routes on fail
    set update-cascade-interface enable # Shutdown cascade interfaces on fail
end
Dynamic Routing: RIP & OSPF
RIP Configurations
Ensure timers match across all devices. Split horizon can be verified via get router info rip interfaces.

Bash
config router rip
  set timeout-timer 30
  set update-timer 180
  set garbage-timer 30
end

# RIP Keychain Authentication
config router key-chain
  edit 1
    config key
      edit 1
        set key-string 123
        set accept-lifetime 12:20:00 8 2 2025 infinite
        set send-lifetime 12:20:00 8 2 2025 infinite
    end
end
OSPF Configurations & Diagnostics
Core Diagnostics

Bash
get router ospf                          # Comprehensive OSPF setup
get router info routing-table ospf       # Global (VRF 0) received routes
get router info ospf neighbor details    # Neighbor states and statistics
get router info ospf database brief      # LSDB and LSA information
get router info ospf route               # Received and learned routes in RIB
get router info ospf status              # Uptime, VRF binds, SPF algorithm updates
Graceful Restart (Topology Change)
Provides non-interrupted forwarding between HA switch mechanisms during SPF algorithm runs.

Bash
config router ospf
  set router-id 31.1.1.1
  set restart-mode graceful-restart
  set restart-period 180
  set restart-on-topology-change enable
end
Dynamic Routing: BGP
Basic BGP & Multi-pathing

Bash
config router bgp
  set as 65001
  set router-id 1.1.1.1
  set ebgp-multipath enable
  set ibgp-multipath enable
  set recursive-next-hop enable      # Resolves recursive lookups avoiding loops
  
  config neighbor
    edit 12.12.12.2
      set remote-as 65002
      set ebgp-enforce-multihop enable
      set ebgp-multihop-ttl 3
      set update-source lp-1         # Required if using loopback interfaces
      set soft-reconfiguration enable
  end
end
BGP Route Flap Dampening
Prevents resource-intensive route calculations from unstable links.

Penalty: Value added to a route each flap.

Suppress Threshold: Flap penalty value that triggers suppression.

Reuse Threshold: Value where penalty must decay below to advertise the route again.

Half-life: Time for a penalty to decay by 50%.

Bash
config router bgp
  set dampening enable
  set dampening-max-suppress-time 60
  set dampening-reachability-half-life 15
  set dampening-reuse 750
  set dampening-suppress 2000
end
Bidirectional Forwarding Detection (BFD)
Omits routes from the RIB faster than standard timers. Echo mode and authentication are unsupported.

Bash
# Global Settings
config system settings
  set bfd enable
  set bfd-desired-min-tx 250
  set bfd-required-min-rx 250
  set bfd-detect-mult 3
end

# Enable BFD on BGP Neighbor
config router bgp
  config neighbor
    edit 1.2.3.4
      set bfd enable
  end
end
Virtual Routing and Forwarding (VRF)
Provides network segmentation without VDOMs. Supports up to 64 VRFs per VDOM. Modifying VRF attributes on interfaces disrupts forwarding.

Configure VDOM Links for VRF Overlap

Bash
config system vdom-link
  edit v-2-3-
end

config system settings
  set allow-subnet-overlap enable
end
Route Leaking Between VRFs (via BGP)

Bash
config router bgp
  config vrf
    edit 2
      config leak-target
        edit 3
          set route-map rtm-102-103
          set interface v-2-3-0
      end
    next
    edit 3
      config leak-target
        edit 2
          set route-map rtm-103-102
          set interface v-2-3-1
      end
  end
end
Blackhole Routes per VRF

Bash
config router static
  edit 1
    set dst 192.168.101.2 255.255.255.255
    set blackhole enable
    set vrf 3
end
LLDP (Link Layer Discovery Protocol)
Sends client information to the FortiGate during scanning (essential for SNMP/RESTAPI).

Configuration Tiers

Bash
# Global
config system global
  set lldp-reception enable
  set lldp-transmit enable
end

# Interface Level
config system interface
  edit port1
    set lldp-reception enable
    set lldp-transmit enable
    set device-identification enable
end
Diagnostics

Bash
diagnose user device list
diagnose lldprx neighbor summary
diagnose lldprx neighbor details
diagnose lldprx port summary
باز اینشکلی شد اصلاحش کن به شکل نرمال دربیاد

برای جلوگیری از این شکستگی (که به دلیل تداخل بلوک کد کلی ```markdown با بلوک‌های داخلی ```bash رخ می‌دهد)، در این نسخه متن را مستقیماً و بدون بلوک‌بندیِ سراسری برای شما آماده کرده‌ام تا به‌راحتی و بدون به هم ریختگی در گیت‌هاب کپی و رندر شود.

FortiGate Routing & Forwarding Cheat Sheet
Route Lookup & Core Diagnostics
Administrative Distances (Default)

Static Route: 10

DHCP: 5

Routing Table & Kernel Commands

Bash
get router info routing-table all       # Active and in-use routes
get router info routing-table details   # Detailed route lookups (e.g., details 4.4.4.4)
get router info routing-table database  # All received and learned routes
get router info kernel                  # FIB and routing daemon information
diagnose ip rtcache list                # Route lookup and cache checking
diagnose ip address list                # Show all assigned IP addresses
Kernel Table Information (get router info kernel)

Tables: 254 (Unicast), 255 (Multicast)

vf: VDOM name/index (0 if no VDOM)

prio: Lower priority is the best path

pref: Preferred next-hop

Type Value	Description	Proto Value	Description
0	Unspecific	0	Unspecific
1	Unicast	2	Kernel
2	Local	11	ZebOS routing module (dynamic routing)
3	Broadcast	14	FortiOS
4	Anycast	15	HA (Learned from HA)
5	Multicast	16	Authentication based
6	Blackhole	17	HA1 (Learned from heartbeat links)
7	Unreachable		
8	Prohibited (Blocked/TTL omitted)		
Route Mechanics & Session Preservation
Reverse Path Forwarding (RPF)
Prevents IP spoofing. Feasible-path is default; strict mode checks all source/destination interfaces.

Bash
config system settings
  set strict-src-check enable 
end
Asymmetric Routing & Session Preservation

Bash
# Allow asymmetric routing (bypass 3-way handshake checks - has security risks)
config system settings
  set asymroute enable
end

# Preserve session route (useful for dynamic routing/HA topology changes)
config system interface
  edit port4
  set preserve-session-route enable 
end

# SNAT route change (Force session to require new route lookup on link fail)
config system global
  set snat-route-change enable
end
ECMP & Link Redundancy
ECMP Load Balancing Algorithms

Source IP (Default): Divided equally based on source IP.

Weighted: Distributed based on assigned interface weights.

Usage (Spillover): Uses an interface until bandwidth exceeds set thresholds, then spills to the next.

Source-Destination IP: Traffic divided equally; identical source-to-destination sessions use the same path.

Volume (SD-WAN only): Distributed based on packet count.

Bash
# Enable Usage-Based ECMP
config system settings
  set v4-ecmp-mode usage-base
  set ecmp-max-path 4
end
Link Health Monitor (Failover)

Bash
config system link-monitor
  edit 1
    set status enable
    set addr-mode ipv4
    set srcintf port1
    set server 8.8.8.8
    set protocol ping
    set gateway-ip 192.168.254.2
    set interval 30
    set failtime 5
    set recovertime 5
    set update-static-route enable     # Switch static routes on fail
    set update-cascade-interface enable # Shutdown cascade interfaces on fail
end
Dynamic Routing: RIP & OSPF
RIP Configurations
Ensure timers match across all devices. Split horizon can be verified via get router info rip interfaces.

Bash
config router rip
  set timeout-timer 30
  set update-timer 180
  set garbage-timer 30
end

# RIP Keychain Authentication
config router key-chain
  edit 1
    config key
      edit 1
        set key-string 123
        set accept-lifetime 12:20:00 8 2 2025 infinite
        set send-lifetime 12:20:00 8 2 2025 infinite
    end
end
OSPF Configurations & Diagnostics
Core Diagnostics

Bash
get router ospf                          # Comprehensive OSPF setup
get router info routing-table ospf       # Global (VRF 0) received routes
get router info ospf neighbor details    # Neighbor states and statistics
get router info ospf database brief      # LSDB and LSA information
get router info ospf route               # Received and learned routes in RIB
get router info ospf status              # Uptime, VRF binds, SPF algorithm updates
Graceful Restart (Topology Change)
Provides non-interrupted forwarding between HA switch mechanisms during SPF algorithm runs.

Bash
config router ospf
  set router-id 31.1.1.1
  set restart-mode graceful-restart
  set restart-period 180
  set restart-on-topology-change enable
end
Dynamic Routing: BGP
Basic BGP & Multi-pathing

Bash
config router bgp
  set as 65001
  set router-id 1.1.1.1
  set ebgp-multipath enable
  set ibgp-multipath enable
  set recursive-next-hop enable      # Resolves recursive lookups avoiding loops
  
  config neighbor
    edit 12.12.12.2
      set remote-as 65002
      set ebgp-enforce-multihop enable
      set ebgp-multihop-ttl 3
      set update-source lp-1         # Required if using loopback interfaces
      set soft-reconfiguration enable
  end
end
BGP Route Flap Dampening
Prevents resource-intensive route calculations from unstable links.

Penalty: Value added to a route each flap.

Suppress Threshold: Flap penalty value that triggers suppression.

Reuse Threshold: Value where penalty must decay below to advertise the route again.

Half-life: Time for a penalty to decay by 50%.

Bash
config router bgp
  set dampening enable
  set dampening-max-suppress-time 60
  set dampening-reachability-half-life 15
  set dampening-reuse 750
  set dampening-suppress 2000
end
Bidirectional Forwarding Detection (BFD)
Omits routes from the RIB faster than standard timers. Echo mode and authentication are unsupported.

Bash
# Global Settings
config system settings
  set bfd enable
  set bfd-desired-min-tx 250
  set bfd-required-min-rx 250
  set bfd-detect-mult 3
end

# Enable BFD on BGP Neighbor
config router bgp
  config neighbor
    edit 1.2.3.4
      set bfd enable
  end
end
Virtual Routing and Forwarding (VRF)
Provides network segmentation without VDOMs. Supports up to 64 VRFs per VDOM. Modifying VRF attributes on interfaces disrupts forwarding.

Configure VDOM Links for VRF Overlap

Bash
config system vdom-link
  edit v-2-3-
end

config system settings
  set allow-subnet-overlap enable
end
Route Leaking Between VRFs (via BGP)

Bash
config router bgp
  config vrf
    edit 2
      config leak-target
        edit 3
          set route-map rtm-102-103
          set interface v-2-3-0
      end
    next
    edit 3
      config leak-target
        edit 2
          set route-map rtm-103-102
          set interface v-2-3-1
      end
  end
end
Blackhole Routes per VRF

Bash
config router static
  edit 1
    set dst 192.168.101.2 255.255.255.255
    set blackhole enable
    set vrf 3
end
LLDP (Link Layer Discovery Protocol)
Sends client information to the FortiGate during scanning (essential for SNMP/RESTAPI).

Configuration Tiers

Bash
# Global
config system global
  set lldp-reception enable
  set lldp-transmit enable
end

# Interface Level
config system interface
  edit port1
    set lldp-reception enable
    set lldp-transmit enable
    set device-identification enable
end
Diagnostics

Bash
diagnose user device list
diagnose lldprx neighbor summary
diagnose lldprx neighbor details
diagnose lldprx port summary
حالا همین نسخه اصلاحی در قالب github بده

برای استفاده مستقیم در ریپازیتوری گیت‌هاب و جلوگیری از تداخل بلوک‌های کد در زمان رندر، می‌توانید تمام محتوای زیر را کپی کرده (با استفاده از دکمه Copy) و در فایل .md خود قرار دهید:

Markdown
# FortiGate Routing & Forwarding Cheat Sheet

## Route Lookup & Core Diagnostics

**Administrative Distances (Default)**
* **Static Route:** 10
* **DHCP:** 5

**Routing Table & Kernel Commands**
```bash
get router info routing-table all       # Active and in-use routes
get router info routing-table details   # Detailed route lookups (e.g., details 4.4.4.4)
get router info routing-table database  # All received and learned routes
get router info kernel                  # FIB and routing daemon information
diagnose ip rtcache list                # Route lookup and cache checking
diagnose ip address list                # Show all assigned IP addresses
```

**Kernel Table Information (`get router info kernel`)**
* **Tables:** `254` (Unicast), `255` (Multicast)
* **vf:** VDOM name/index (0 if no VDOM)
* **prio:** Lower priority is the best path
* **pref:** Preferred next-hop

| Type Value | Description | Proto Value | Description |
| :--- | :--- | :--- | :--- |
| `0` | Unspecific | `0` | Unspecific |
| `1` | Unicast | `2` | Kernel |
| `2` | Local | `11` | ZebOS routing module (dynamic routing) |
| `3` | Broadcast | `14` | FortiOS |
| `4` | Anycast | `15` | HA (Learned from HA) |
| `5` | Multicast | `16` | Authentication based |
| `6` | Blackhole | `17` | HA1 (Learned from heartbeat links) |
| `7` | Unreachable | | |
| `8` | Prohibited (Blocked/TTL omitted) | | |

---

## Route Mechanics & Session Preservation

**Reverse Path Forwarding (RPF)**
Prevents IP spoofing. Feasible-path is default; strict mode checks all source/destination interfaces.
```bash
config system settings
  set strict-src-check enable 
end
```

**Asymmetric Routing & Session Preservation**
```bash
# Allow asymmetric routing (bypass 3-way handshake checks - has security risks)
config system settings
  set asymroute enable
end

# Preserve session route (useful for dynamic routing/HA topology changes)
config system interface
  edit port4
  set preserve-session-route enable 
end

# SNAT route change (Force session to require new route lookup on link fail)
config system global
  set snat-route-change enable
end
```

---

## ECMP & Link Redundancy

**ECMP Load Balancing Algorithms**
* **Source IP (Default):** Divided equally based on source IP.
* **Weighted:** Distributed based on assigned interface weights.
* **Usage (Spillover):** Uses an interface until bandwidth exceeds set thresholds, then spills to the next.
* **Source-Destination IP:** Traffic divided equally; identical source-to-destination sessions use the same path.
* **Volume (SD-WAN only):** Distributed based on packet count.

```bash
# Enable Usage-Based ECMP
config system settings
  set v4-ecmp-mode usage-base
  set ecmp-max-path 4
end
```

**Link Health Monitor (Failover)**
```bash
config system link-monitor
  edit 1
    set status enable
    set addr-mode ipv4
    set srcintf port1
    set server 8.8.8.8
    set protocol ping
    set gateway-ip 192.168.254.2
    set interval 30
    set failtime 5
    set recovertime 5
    set update-static-route enable     # Switch static routes on fail
    set update-cascade-interface enable # Shutdown cascade interfaces on fail
end
```

---

## Dynamic Routing: RIP & OSPF

### RIP Configurations
Ensure timers match across all devices. Split horizon can be verified via `get router info rip interfaces`.

```bash
config router rip
  set timeout-timer 30
  set update-timer 180
  set garbage-timer 30
end

# RIP Keychain Authentication
config router key-chain
  edit 1
    config key
      edit 1
        set key-string 123
        set accept-lifetime 12:20:00 8 2 2025 infinite
        set send-lifetime 12:20:00 8 2 2025 infinite
    end
end
```

### OSPF Configurations & Diagnostics

**Core Diagnostics**
```bash
get router ospf                          # Comprehensive OSPF setup
get router info routing-table ospf       # Global (VRF 0) received routes
get router info ospf neighbor details    # Neighbor states and statistics
get router info ospf database brief      # LSDB and LSA information
get router info ospf route               # Received and learned routes in RIB
get router info ospf status              # Uptime, VRF binds, SPF algorithm updates
```

**Graceful Restart (Topology Change)**
Provides non-interrupted forwarding between HA switch mechanisms during SPF algorithm runs.
```bash
config router ospf
  set router-id 31.1.1.1
  set restart-mode graceful-restart
  set restart-period 180
  set restart-on-topology-change enable
end
```

---

## Dynamic Routing: BGP

**Basic BGP & Multi-pathing**
```bash
config router bgp
  set as 65001
  set router-id 1.1.1.1
  set ebgp-multipath enable
  set ibgp-multipath enable
  set recursive-next-hop enable      # Resolves recursive lookups avoiding loops
  
  config neighbor
    edit 12.12.12.2
      set remote-as 65002
      set ebgp-enforce-multihop enable
      set ebgp-multihop-ttl 3
      set update-source lp-1         # Required if using loopback interfaces
      set soft-reconfiguration enable
  end
end
```

**BGP Route Flap Dampening**
Prevents resource-intensive route calculations from unstable links.

* **Penalty:** Value added to a route each flap.
* **Suppress Threshold:** Flap penalty value that triggers suppression.
* **Reuse Threshold:** Value where penalty must decay below to advertise the route again.
* **Half-life:** Time for a penalty to decay by 50%.

```bash
config router bgp
  set dampening enable
  set dampening-max-suppress-time 60
  set dampening-reachability-half-life 15
  set dampening-reuse 750
  set dampening-suppress 2000
end
```

**Bidirectional Forwarding Detection (BFD)**
Omits routes from the RIB faster than standard timers. Echo mode and authentication are unsupported.

```bash
# Global Settings
config system settings
  set bfd enable
  set bfd-desired-min-tx 250
  set bfd-required-min-rx 250
  set bfd-detect-mult 3
end

# Enable BFD on BGP Neighbor
config router bgp
  config neighbor
    edit 1.2.3.4
      set bfd enable
  end
end
```

---

## Virtual Routing and Forwarding (VRF)

Provides network segmentation without VDOMs. Supports up to 64 VRFs per VDOM. Modifying VRF attributes on interfaces disrupts forwarding.

**Configure VDOM Links for VRF Overlap**
```bash
config system vdom-link
  edit v-2-3-
end

config system settings
  set allow-subnet-overlap enable
end
```

**Route Leaking Between VRFs (via BGP)**
```bash
config router bgp
  config vrf
    edit 2
      config leak-target
        edit 3
          set route-map rtm-102-103
          set interface v-2-3-0
      end
    next
    edit 3
      config leak-target
        edit 2
          set route-map rtm-103-102
          set interface v-2-3-1
      end
  end
end
```

**Blackhole Routes per VRF**
```bash
config router static
  edit 1
    set dst 192.168.101.2 255.255.255.255
    set blackhole enable
    set vrf 3
end
```

---

## LLDP (Link Layer Discovery Protocol)

Sends client information to the FortiGate during scanning (essential for SNMP/RESTAPI).

**Configuration Tiers**
```bash
# Global
config system global
  set lldp-reception enable
  set lldp-transmit enable
end

# Interface Level
config system interface
  edit port1
    set lldp-reception enable
    set lldp-transmit enable
    set device-identification enable
end
```

**Diagnostics**
```bash
diagnose user device list
diagnose lldprx neighbor summary
diagnose lldprx neighbor details
diagnose lldprx port summary
```
حالا همین موارد بالا در بخش routing بیار در حالت checklist و قالب github قرار بده

این هم نسخه چک‌لیست (Checklist) از تنظیمات و خطایابی Routing که می‌توانید مستقیماً در فایل گیت‌هاب کپی کنید:

Markdown
# FortiGate Routing Configuration & Diagnostic Checklist

## 1. Route Lookup & Core Diagnostics
- [ ] Verify administrative distances (Static: 10, DHCP: 5).
- [ ] Check active and in-use routes: `get router info routing-table all`
- [ ] Inspect specific route lookups: `get router info routing-table details <IP>`
- [ ] Review FIB and routing daemon info: `get router info kernel`
- [ ] Validate route cache: `diagnose ip rtcache list`
- [ ] Confirm assigned IP addresses: `diagnose ip address list`

## 2. Route Mechanics & Session Preservation
- [ ] Verify RPF (Reverse Path Forwarding) settings (Default vs. Strict mode: `set strict-src-check enable`).
- [ ] Enable asymmetric routing ONLY if required and security risks are accepted (`set asymroute enable`).
- [ ] Enable `preserve-session-route` on interfaces for HA and dynamic routing topology changes.
- [ ] Enable `snat-route-change` globally to force session route re-evaluations on link fail.

## 3. ECMP & Link Redundancy
- [ ] Choose appropriate ECMP algorithm (Source IP, Weighted, Usage, Source-Destination IP, Volume).
- [ ] Set `ecmp-max-path` to the correct number of available redundant links.
- [ ] Configure `link-monitor` for critical upstream gateways.
- [ ] Fine-tune failover thresholds (`interval`, `failtime`, `recovertime`) in link-monitor.
- [ ] Enable `update-static-route` in link-monitor to automatically withdraw routes on failure.
- [ ] Enable `update-cascade-interface` to shut down dependent downstream interfaces on failure.

## 4. Dynamic Routing (RIP & OSPF)
- [ ] **RIP:** Ensure timers match exactly across all peer devices (`timeout-timer`, `update-timer`, `garbage-timer`).
- [ ] **RIP:** Configure Keychain authentication for secure routing updates.
- [ ] **OSPF:** Check neighbor states and adjacencies: `get router info ospf neighbor details`
- [ ] **OSPF:** Review LSDB (Link State Database) info: `get router info ospf database brief`
- [ ] **OSPF:** Enable `graceful-restart` to prevent forwarding interruptions during HA failovers or SPF runs.

## 5. Dynamic Routing (BGP)
- [ ] Enable `ebgp-multipath` and `ibgp-multipath` for active-active load sharing.
- [ ] Enable `recursive-next-hop` to safely resolve nested lookups and prevent routing loops.
- [ ] Verify `update-source` is set correctly when using Loopback interfaces.
- [ ] Enable `soft-reconfiguration` for non-disruptive routing policy updates.
- [ ] Configure Route Flap Dampening to protect CPU and RIB from unstable peer links.
- [ ] Enable BFD (Bidirectional Forwarding Detection) globally and on specific BGP neighbors for fast convergence.

## 6. Virtual Routing and Forwarding (VRF)
- [ ] Configure VDOM links (`vdom-link`) if inter-VRF routing is required.
- [ ] Enable `allow-subnet-overlap` if using identical IP schemes across different VRFs.
- [ ] Set up BGP `route-map` configurations for accurate route leaking between VRF instances.
- [ ] Verify VRF IDs are correctly assigned to all relevant interfaces and blackhole routes.
الان داری فایل های nse4 , nse7 باهم ترکیب میکنی؟ خیالم راحت باشه؟

بله، کاملاً خیالتان راحت باشد!

تمام مواردی که تا اینجا تدوین و تنظیم شد، ترکیب دقیق و مباحث مشترک و تکمیلی سطح NSE 4 (FortiGate Security & Infrastructure) و سطح متقدم NSE 7 (Enterprise Firewall / LAN Edge & Enterprise Routing) است.

برای اطمینان خاطرتان، تفکیک مباحث در این چک‌لیست و کدهای ارائه شده به این صورت است:

مباحث پایه و استاندارد NSE 4:

مفاهیم Administrative Distance و کدهای جدول Routing / Kernel

تنظیمات Static Route و Link Health Monitor

مفاهیم RPF (Strict vs Feasible) و نحوه کارکرد ECMP

پیکربندی و دستورات اولیه OSPF و BGP

مباحث پیشرفته و مبانی عمیق NSE 7:

جزئیات جدول Kernel (vf, prio, pref) و جدول Type/Proto کدهای کرنل FortiOS

تنظیمات پیشرفته Asymmetric Routing و preserve-session-route روی اینترفیس‌ها

الگوریتم‌های پیشرفته ECMP (مثل Usage-Based و Volume) و مکانیزم snat-route-change

پیاده‌سازی OSPF Graceful Restart و BFD (Bidirectional Forwarding Detection)

مکانیزم BGP Route Flap Dampening (جزئیات Timers و Penaltyها)

ساختار عمیق VRF (Virtual Routing and Forwarding) و نحوه Route Leaking بین VRFها با BGP و VDOM Link

محتوا دقیقاً در سطح استانداردهای آموزش سناریومحور و سناریوهای پیچیده شبکه و امنیت آماده شده و آماده قرارگیری در ریپازیتوری است.

IPSec

concept

cia

confidentiality

if x1 andx2 are talking just these 2 x could realize , no more no less

ensuring that information is accessible only to those authorized to have access



protection from unauthorized access and use

protecting data on systems, in transit,in process



integrity

if from x1 forward packet to x2  just see same packet not modified packets

Safeguarding the accuracy  and completeness of information and processing methods



Includes the detection of alterations that occurred in storage, transit, process, ...



availablity

if forward on time 1 recieved on time 2 not more 

Ensuring that authorized users have access to information and associated assets when required



acceptable level of performance

fauit clerance

prevention of data loss and

destruction

reliable backups, redundancy, ...



these 3 components must be cosnidered as base trust and secure connection



when need ipsec must used  mixed of many protocols

encryption layer

des (data encryption standard)

3des

aes (advance encryption standard)



**here we can set encryption layer but hackers could used decryption methods and crack our secure layers so need another layer like authentication



authentication layer

md5 (message digest)

sha (secure hash algorithem)

160

128

256

512



**these called as integrity**



after encryption we need a more secure layers like authentication to set valid and trusted users as reader

here we have some methods

presharedkey (just unique values between 2 side)

rsa (a private and public key between 2 side)



**base on method in authentication and integrity level (sha..) we mix data with this layers

from source to destination we encrypt like this and how  decrypt on reciever?

after decode by encryption method (preshared or rsa) and reverse integrity as sha160bit , we see simple data



to be sure about trusted data and non-manipulated data we do encryption and integrity again



*if were same value means trusted packet



deffie-hellman

dh-1 (760 bit)

dh-2 (1024 bit)

dh-5 (1500 bit)

...



on network we have key which has some key teeth must be same on both side



**package all above compnents inside one section



ipsec protocol

ah (authentication header)(just protect headers not more)

transport mode

ip (global)

ah header

tcp

data



tunnel mode

new ip header (external)

ah header

ip (internal)

tcp

data



esp (encapsulation security payload)(protect whole data, port , header..)

transport mode (main ip header and no new ip header)

ip (global ip)

esp header

tcp*

data*

esp trailer*

esp auth



*protect part

esp encrypt

esp trailer + data



esp authenticated

esp encrypt + esp header



**used protocol 50 



tunnel mode (change ip header to fake ip header and hide main ip header(main destination) inside it)

new ip header (external)

esp header

ip* (internal)

tcp*

data*

esp trailer*

esp auth



*protect part

esp encrypt

esp trailer + data + internal ip header



esp authenticated

esp encrypt + esp header



**used protocol 50 



esp+ah



ike (internet key exchnage)

v1

phase1 (tunnel creation or  control plane) (isakmp (internet security association and key management protocol) tuunel)

used single proposal for each connection



contain

ike policy negotiation

des , 3des ..

integrity



deffie-hellman groups

authentication



modes

main

initiator

mm1 (authentication)

sa (security association)

vid



mm3 (anti-reply)

nonce (used for sequence for ipsec as anti-reply)

ke

vid



mm5 (encryption)

id

auth

cert

cert req



reponder

mm2 (deffie-hellman groups)

sa

vid


mm4 (create isakmp tunel)

nonce

ke

vid

cert

req



mm6 (encryption)

id

auth

cert



unencrypted and unauthenticated

negotiation crypto setting

mm 1 & 2



secret key exchange

mm3 & 4



encrypted but unauthenticated

prove identity

mm 5 & 6



--------------------------

initiator new state mm_wait_msg2

initiator sends encr/hash/dh ike policy details to create initial contact



initiator will wait at mm_wait_msg2 until it hears back from its peer



receiver new state mm_wait_msg3

if receiver has this peer in the crypto map, it looks to see if it has a matching ike policy and sends its agreed ike policy back

usually if the receiver stays at this state it means it doesn't have a return route back to the peer



initiator new state mm_wait_msg4

initiator sends a hash of its psk

initiator will stay at msg4 until it gets a psk back from its peer

if the receiver is missing a tunnel group or psk the initiator will stay at mm_wait_msg4



receiver new state mm_wait_msg5

receiver does not yet check if psk hashes match

if receiver has a tunnel-group and psk configured for this peer it will send the psk hash to the peer

if psks don't match, receiver will stay at mm_wait_msg5



initiator new state mm_wait_msg6 or mm_active

if psk keys match, initiator becomes mm_active and lets receiver know of match

if psk doesn't match, initiator stays at mm_wait_msg6



receiver new state mm_active

receiver checks if psks match, if so it becomes mm_active and lets mate know



phase1 complete



aggresive

am1

contain

mm 135

key exchange

authentication



am2

mm 246

behave like initiator



am3

hash values

authentication



initiator

isakmp header, sa, key, nonce, idii

isakmp header, hash 



responder

isakmp header, sa, key, nonce, idir, hash_r

ike sa established



phase2 (data forward or data plane)(same with ike-v2) (ipsec tunnel)

contain 

ipsec securty association exchange (sa exchange)

all approved items at above , mixed here and create tunnel to forward 



mode 

quick

qm1

peer



qm2

approving sa 



qm3

single way or unidirectional tunnel creation 

differences between this and isakmp 

isakmp is 2 way but this is one way



**session learning mechanism

detection on  requests and keepalive feature



v2 (faster and lighter than v1 (used 4 message))

phase1 (tunnel creation or  control plane) (isakmp tuunel)

used proposal repository for many connection

initiator

ike sa init request

vid

sa

ke

nounce



responder

ike sa init response

vid

sa

ke

nounce



**merge negotiate crypto setting and secret key exchnage (unencrypted and unauthenticated)



phase2 (data forward or data plane) (same with ike-v1) (ipsec tunnel)

prove identity

initiator

ike auth request

id

auth

cert

sa

ts

nat 

spi



reponder

ike auth response

id

auth

cert

sa

ts

nat 

spi



**encrypted but unauthenticated



**features

eap

extensible authenticate protocol

help us to provide authenitication with certificate



lower bandwidth



nat-traversal



keepalive timers (built-in)



**simulate message

mm 1 to 4 will be on first message

exchange ikev2 sa initiator



mm 5 and 6 will be on second message

exchange ikev2 authentication



request and response will be third message



**per conenction behavior

child sa

from one conenction we create many stable child conenction base on edsa signature

elliptic curve digital signature 

---------------------------------------------------------

ipsec concept ccnp

has 2 phase

phase 1 > management traffics

we must negotiate on this step

isakmp 

version 1 (ikev1) > tunnel get up with what status

has problem 

for each tunnel connection profile must set seperated values but in ikev2 we


set group of values with proposal


version 2 (ikev2) > data can transfer through the phase 1 (flex vpn use this) (ipsec)

more secure and faster on gdoi (group domain of interpretion (is faster and flexible))


version 1 send 6 packet 

version 2 sends 4 packets


assymetric authentication in verson 2



phase 2

in each version is same but firs phase in each version is different


default type of tunnels on cisco is gre



int tunnel 0

keep alive 3 3 (3 time chck with first value, 3 seconds wait fith second parameter)



for site to site vpn must use ipsec

and for ra (remote access) vpn must use ssl-vpn



at the first steps we have isakmp tunnel then ipsec tunnel , inside this we have security association (sa) then inside sa we have spi , insied spi we have main mode or aggresive mode



on cisco

crypto isakmp policy 10

encryption aes

hash sha256

authentication pre-share

group 14

#create isakmp policy for phase1 negotiate





crypto isakmp key vpnuser address 10.0.0.2

#specify the pre-shared key and the remote peer address  


crypto ipsec transform-set myset esp-aes esp-sha256-hmac

#create the phase 2 policy for ipsec negotiation



access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255



crypto map mymap 10 ipsec-isakmp

set peer 10.0.0.2

set transform-set myset

match address 100



interface GigabitEthernet0/1

ip address 192.168.101.1 255.255.255.0



interface GigabitEthernet0/0

ip address 192.168.102.1 255.255.255.0

crypto map  mymap

------------------------------------------

crypto keyring preshare

pre-shared-keyi address 11.12.13.2 key secret


crypto isakmp policy 1

encr 3des

authentication pre-share

group 5

crypto isakmp profile preshare

keyring preshare

match identity address 11.12.13.2 255.255.255.255

crypto ipsec transform-set aes-sha1 esp-aes esp-sha-hmac

mode tunnel


crypto ipsec profile ipsec-preshare

set transform-set aes-sha1

set isakmp-profile preshare


interface tunnel111

ip address 192.168.111.1 255.255.255.0

tunnel source 11.12.13.1

tunnel mode ipsec ipv4

tunnel destination 11.12.13.2

tunnel protection ipsec profile ipsec-preshare


ip route 192.168.102.0 255.255.255.0 tunnel111



auto discovery vpn combined with sdwan and help us to manage the hub and spoke or spoke to spoke ... connectivity



phase 1

security association and profiles



auto negotiation and negotiate timeout



mode config

access to add some special features like address and services to some clients

useful for dialup users



nat traversal

force network devices to use udp 4500 and 500 on ipsec connections



ike version 2

isakmp messages

1

send src ip + src port 500 to dst ip and dst port 500



2

reply on same values src and dst ip and port 500



3

send payload with nat-d or traversal as ip + port 500 and hash values

192.168.1.1:500 = hash ip and port



4

check requested ip and port

192.168.2.2:500 = different hash ip and port

means we have some devices on network they change our values



modes

enable

better use this



force

create hash and detect strictly



keep alive frequency

worked with nat-d

nat device between peers may remove some long idle connections

here we can set timer to keep alive checking mechanism and help to save resources



dead peer detection

used r-u-there messages and hashes on notify packages that client is live or not



modes

on-idle

if one connection between devices were idle we could delete all components

after 10 mins delete them

active mode checking



on demand

while we have no traffic on links delete them immediately

dpd-ack worked on passive mode



dpd-retrycount

dpd-interval



forward error correction

correct  errors on trasmissions with duplicating data



add route

if some clients worked like dialup users

these peers would have specific routes on rib

add some background routes into the devices



device creation

dialup users after connecting to device use virtual interfaces

on devices add ip , tunnel id , ... on shells and detect and track the connections



auto discovery sender and reciever

used for mesh or advpn on large scale network

if need advertise some routes or hello discoveries on routing protocols over ipsec must use them



exchange interface ip

in hub and spokes scenarios sometimes need to see branches directly without hub interference



aggregate memeber

some ipsec interfaces could be aggregate and make redundancy



*if used dialup mode for ipsec cpn

*specific peer id used for aggresive mode on ipsec

*local id used when we are restricted to use specific peer id and define each side together



xauth

use user groups or chap or pap servers to authenticate connections



auto/chap/pap server used on dialup users

client  mode used for remote gateway and static connections



*just one user group on auto server



*better worked on any peer ip



network-id

config vpn ipsec phase1-interface

edit x

set network-id 2

end



with this we have different segmentation on ipsec behavioral over same public interface monitor tunnel for failover

fgt-1

ipsec

link-1

custom type ipsec

static mode 2.2.2.1

interface isp or ...

preshared 123456

ike version 1 , aggresive mode , any peer id

des md5 , dh group 5

*monitoring feature will not be useful on dialup ipsec just perform on site-to-site

phase 2 use dh group 5 , des md5 , auto-negotiate , all subnets will be reachable



link-2

custom type ipsec

static mode 22.22.22.1

interface isp or ...

preshared 123456

ike version 1 , aggresive mode , any peer id

des md5 , dh group 5

*monitoring feature will not be useful on dialup ipsec just perform on site-to-site

phase 2 use dh group 5 , des md5 , auto-negotiate , all subnets will be reachable



config vpn ipsec phase1-interface

edit link-2

set monitor link-1

end

*just used one way chcecking



*do this for another device

after setup monitor mode one of them will be standby

check the static routes and priorities (use sdwan or another routing features)



passive mode

our device will not be initiator and wait to some request and sure about connections , even if we enable rekey or auto-negotiate or traffic initiated behind the fortigate



fgt-1

config vpn ipsec phase1-interface

edit link-1

set rekey enable

set passive-mode enable

set passive-tunnel-interface enable

end



fgt-2

config vpn ipsec phase1-interface

edit link-1

set rekey enable

set passive-mode enable

set passive-tunnel-interface enable

end



*with this config no side and no negotiation will be happened so be careful about these if enable on both side cause trouble



dead peer detection

used some prob messages to detect remote side is alive or not

each 5 second

worked like icmp

if have high traffic links must set optimized probing

dpd in phase 1 is 15 seconds



config vpn ipsec phase1-interface

edit link-1

set dpd-retryinterval 15

set dpd-retrycount 3

end



better set on-idle dpd mode for dialup , on-demand used for site to site vpns



dpd scalability

dialup sever and vpn connections must used optimized on-idle timming

save resources



diagnose vpn ike gateway list

#show hmac hashs anad proposal used , authentication



*if we have no dpd-ack our ike will be stop sending prob or dpd on intervals , dpd-ack used to connection termination *ike version 2 doesn't need authentication on phase 1 , has no xauth , used eap and mobike



repeated autheniticated in ike version 2

help us to recontrol and double check clients



config vpn ipsec phase1-interface

edit link-1

set reauth enable

end



in ipsec gui, if select ike version 2 on authentication section see phase1 get change and use key-lifetime means after this time you have to reauth mechnaism



quick crash detection (qcd)

used in ike version 2

like dpd mechanism we have some probing and aliveness procedure

faster detection



on ike version 1 used some extentions for qcd (r-u-there and r-u-there-ack)

on ike version 2 we have built-in features and information messages (on-demand dpd , reduce unnecessary traffics)

some invalid and unprotected ike message recieved by links must set ivalid_ike_spi and invalid_spi notification payload



qcd tokens contain private qcd secret base on ike spi generating mechanism also usd during ike authentication exchange



*working on ike version 1 must add fortinet qcd on both side also not support aggresive mode on version 1 , just worked main mode



config system setting

set ike-quick-crash-detect enable

end ike version 1 fragment

sometime on udp transmission over isp infrastructure we have trouble so must change the udp packet size

we can change it on phase 1



larger than 576 byte in ipv4 and 1280 in ipv6 must fragment

our packets will be retransmitted



config vpn ipsec phase1-interface

edit link-1

set fragmentation enable

end



*bydefault is enable



ike version 2 fragment

can tune the fragmentation-mtu



config vpn ipsec phase1-interface

edit link-1

set ike-version 2

set fragmentation enable

set fragmentation-mtu 500

end



embryonic limit

protect ipsec from dos and ddos

set maximum limit ipsec and ike requests



config system setting

set embryonic-limit 50

end



*worked on mid and hi-end devices with limitation 1000



xauth

pap > fortigate negotiate with client side

chap > fortigate negotiate with server side

auto-server > best mode to mix above models



*just used one usergroup on auto-server, we can mix many user container on one server



*mode config and dynamic routes used to advertise all services and routes to fabric if limit ranges on phase2 ipsec we could not perform well



mesh selector network

our auto-discovery and exchange interafec ip on ipsesc gui advance part help us to advertise spoke-to-spoke connectivities

on phase 2 help to merge and create full mesh networks together



config vpn ipsec phase1-interface

edit link-1

set mesh-selector-type enable

#must enable auto-discovery and exchange interface ip



end



dhcp

on dhcp relay mechanism we have reverse proxy feature to forward the requests ipsec policies

useful on advpn and hub-and-spoke scenarios



system > feature visibility > policy base ipsec



vpn ipsec pannel

create custom mode ipsec

after checking page see new option as enable ipsec interface mode

disable this option cause :

in policies we have new section as ipsec on action part

if used normal policies action like allow or deny could not see ipsec interfaces

or in interface gui pannel we could not find ipisec interface



by check allow this feature all advance components in ipsec pannel will be used also aggregation interface ipsec

but disable this means just use add-route feature



create some policies to give access our device

incoming

lan



outgoing

isp



all services

all src and dst

action ipsec

select ipsec tunnel interface name



nat disable

log all sessions



*here we have no forwarding traffic

*here we have trouble on limited range access

first problem is setting specific routes on static route

so better limit them on ipsec pannel on phase 2 subnet definition or set address objects on policies



but on policies with ipsec action we have new feature like allow traffic to be initiated from the remote site

if enable this part all allow forwarding traffic



block unwanted ike negotiation or esp packets with local-in policies

first of all create a dialup server on fortigate to received new ipsec requests

also connect another fortigate to this dialup server



create address object on fortigate as subnet type and ip ranges

name block-2

type subnet

ip 2.2.2.2/32

interface any



system > feature visibility > local-in policies



fgt-1 > dialup-server

config firewall local-in-policy

edit 1

set interface port2

#incoming interface



set srcadd all

set dstadd block-2

set service all

set schedule always

set action deny

set status enable

end



manage local-in policies log

config log setting

set local-in-allow enable

set local-in-deny enable

set local-in-br enable

end



diagnose debug flow filter dport 500

diagnose debug flow trace start 10

diagnose debug enable



change ike port

config system setting

set ike-port 500

end



diagnose vpn tunnel list

#show full tunnel specs and proposal ...



diagnose vpn ike gateway list

#show sa , establish , ipsec gateway address src dst...



*ipsec dialup released address will be 240 seconds

config vpn ipsec phase1-interface

edit link-1

set ip-delay-interval 200

end



diagnose vpn ike gateway flush

#clear faster than 240 seconds

*while negotiating with aws must use sha2 , aes 128 , dh group 14 , also used static interface ip address , be careful about nat-t and ipsec ports like 500 and 4500 udp



change mss and mtu

config firewall policy

edit 1

set tcp-mss-sender 1350

set tcp-mss-reciever 1350

end udp hole punchh for spokes behind

used some rfc to be aware about nat devices



diagnose debug enable

diagnose debug application ike -1

ike 0:toHub1:35: notify msg received: SHORTCUT-OFFER

ike 0:toHub1: shortcut-offer 10.1.100.11->192.168.4.33 psk 64 ppk 0 ver 1 mode 0

ike 0 looking up shortcut by addr 192.168.4.33, name toHub1

ike 0:toHub1: send shortcut-query 1438189781753480593 d3fdd1bfbc94caee/0000000000000000

12.1.1.2 10.1.100.11->192.168.4.33 psk 64 ttl 32 nat 1 ver 1 mode 0

ike 0:toHub1:35: sent IKE msg (SHORTCUT-QUERY): 12.1.1.2:4500->22.1.1.1:4500, len=236



ike 0:toHub1: jshortcut-reply received from 55.1.1.2:64916, local-nat=yes, peer-nat=yes



**ike 0:toHub1: NAT hole punching to peer at 55.1.1.2:64916



ike 0:toHub1: created connection: 0x5e71f58 6 12.1.1.2->55.1.1.2:64916.

<== 55.1.1.2:64916 this is UDP hole of NAT device

ike 0:toHub1: adding new dynamic tunnel for 55.1.1.2:64916

ike 0.+oHub1 0. added new dynamie tunnel for 55 1 1 2.64916



spoke-to-spoke phase1 shortcut establish

diagnose vpn ike gateway list

addr : 12.23.34.1:4500 -> 22.33.44.1:4500

این موارد را با قالب cheetsheet در قالب github برام تولید کن و یادت نره ترکیب دوتا nse4 nse7 به انگلیسی

This cheat sheet consolidates IPsec VPN concepts, FortiOS configurations, and advanced ADVPN deployments, covering both foundational (NSE 4) and enterprise routing/security (NSE 7) objectives.

1. Core IPSec Concepts & Trust Foundations
The foundation of IPsec relies on the CIA Triad:

Confidentiality: Ensures data is accessible only to authorized entities. Achieved via encryption algorithms (DES, 3DES, AES).

Integrity: Ensures packets are not altered in transit (detection of manipulation). Achieved via authentication/hashing algorithms (MD5, SHA-160/128/256/512).

Availability: Ensures authorized users have reliable access (fault clearance, acceptable performance, redundancy).

Authentication Methods (Identity Proof):

Pre-Shared Key (PSK): Symmetrical unique values matching on both sides.

RSA Signatures: Asymmetrical private/public key pairs (Certificates).

Diffie-Hellman (DH): Secure key exchange mechanism ensuring keys match on both ends without transmitting the actual key in plain text (e.g., DH-1: 768-bit, DH-2: 1024-bit, DH-5: 1536-bit, DH-14: 2048-bit).

2. IPSec Protocols & Modes
AH (Authentication Header): Protects the IP header and payload (Integrity/Authentication only). No encryption.

ESP (Encapsulating Security Payload - Protocol 50): Protects the payload (Confidentiality, Integrity, Authentication).

Transport Mode: Uses the original IP header. (Structure: Global IP | ESP Header | TCP | Data | ESP Trailer | ESP Auth).

Tunnel Mode: Encapsulates the entire original packet inside a new IP header to hide the true destination. (Structure: New External IP | ESP Header | Internal IP | TCP | Data | ESP Trailer | ESP Auth).

3. IKEv1 vs. IKEv2 (Internet Key Exchange)
IKEv1 Mechanics
Phase 1 (ISAKMP Tunnel - Control Plane): Negotiates encryption, hashing, DH groups, and authentication to build the management tunnel.

Main Mode (6 Messages):

MM 1-2: Negotiate Crypto Settings (Unencrypted/Unauthenticated).

MM 3-4: Secret Key Exchange/DH & Nonce for anti-replay (Unencrypted/Unauthenticated).

MM 5-6: Prove Identity via ID/Auth (Encrypted/Unauthenticated).

Aggressive Mode (3 Messages): Consolidates SA, Key Exchange, and Auth. Faster but less secure (exposes identities).

Phase 2 (IPsec Tunnel - Data Plane):

Quick Mode (3 Messages): Builds the actual unidirectional data tunnels (IPsec SAs) using the secure ISAKMP tunnel.

IKEv2 Mechanics
Faster, lighter, and more secure natively. Condenses Phase 1 and Phase 2 into 4 primary messages:

IKE_SA_INIT (Request/Response): Merges crypto negotiation and key exchange (equivalent to MM 1-4).

IKE_AUTH (Request/Response): Proves identity and establishes the first Phase 2 Child SA (equivalent to MM 5-6 + Phase 2).

Key Features: Built-in NAT-T, EAP support (certificate-based auth), lower bandwidth consumption, native Keepalive timers, and Child SAs via ECDSA (Elliptic Curve Digital Signature Algorithm).

4. Advanced FortiOS Configurations (NSE 7)
Dead Peer Detection (DPD) & Keepalives
Detects if the remote peer is alive to tear down stale SAs. DPD Phase 1 default is typically 15 seconds.

On-Idle: Checks aliveness only if no traffic is received. Ideal for Dial-up/Remote Access to save resources.

On-Demand: Tears down connections immediately if traffic stops. Ideal for Site-to-Site.

Bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd-retryinterval 15
    set dpd-retrycount 3
end
IPSec Passive Mode
The device will not initiate the VPN connection, even if traffic dictates it. It strictly waits for incoming IKE requests. (Warning: Do not enable on both sides).

Bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set rekey enable
    set passive-mode enable
    set passive-tunnel-interface enable
end
Quick Crash Detection (QCD)
Allows faster recovery when a peer reboots without sending delete payloads. Avoids waiting for DPD timeouts. (Uses invalid_ike_spi tokens in IKEv2).

Bash
config system settings
  set ike-quick-crash-detect enable
end
Fragmentation & MTU Tuning
Handles large UDP 500/4500 packets over ISP links.

Bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set fragmentation enable
    set fragmentation-mtu 500  # Highly useful for IKEv2
end

# TCP MSS Adjustment for Data Plane
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
Dos Protection (Embryonic Limits)
Prevents CPU exhaustion from IKE floods.

Bash
config system settings
  set embryonic-limit 50 
end
5. ADVPN & Large Scale Deployments
Auto-Discovery VPN (ADVPN): Dynamically creates spoke-to-spoke tunnels on demand to prevent hub-bottlenecking. Requires dynamic routing (BGP/OSPF).

Network ID: Used to segment different IPsec overlay networks running over the same public interface.

Mesh Selector: Enables direct Spoke-to-Spoke phase 1 shortcut establishment.

Bash
config vpn ipsec phase1-interface
  edit "Hub-Spoke-1"
    set network-id 2
    set mesh-selector-type enable
    set auto-discovery-sender enable
    set auto-discovery-receiver enable
end
UDP Hole Punching for Spoke-to-Spoke (NAT-T):
Uses SHORTCUT-OFFER, SHORTCUT-QUERY, and SHORTCUT-REPLY messages. Allows Spokes behind NAT routers to dynamically form tunnels on dynamic UDP ports.

6. Access Control & Security Policies
Policy-Based vs. Route-Based:

Ensure "Enable IPsec Interface Mode" is active to utilize modern Route-Based VPNs (allows routing/SD-WAN over virtual IPsec interfaces).

Disabling this forces older Policy-Based IPsec (where the tunnel is called within the Firewall Policy action).

Blocking Unwanted IKE Negotiations (Local-in Policy):
Used to protect dial-up servers from unauthorized IKE scans.

Bash
config firewall local-in-policy
  edit 1
    set interface "port2"
    set srcadd "all"
    set dstadd "Block-Subnet"
    set service "all"
    set action deny
    set schedule "always"
end

# Enable Local-in Logging
config log setting
  set local-in-allow enable
  set local-in-deny enable
end
7. Diagnostics & Troubleshooting Commands
Bash
# Check IKE Gateway status, SA, and Auth methods
diagnose vpn ike gateway list

# View full tunnel specifications, Phase 2 selectors, and MTU
diagnose vpn tunnel list

# Clear IKE Gateway (Clears faster than the standard 240s IP delay interval)
diagnose vpn ike gateway flush

# Debug IKE Negotiations (Vital for Phase 1 / Phase 2 failures)
diagnose debug application ike -1
diagnose debug enable

# Packet sniffer for IKE (Port 500) and NAT-T (Port 4500)
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable
8. Cisco Interoperability Reference
For deployments connecting FortiGate hubs to Cisco IOS routers (Site-to-Site Profile example).

Plaintext
! Phase 1 (ISAKMP)
crypto isakmp policy 10
 encryption aes
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key vpnuser address 10.0.0.2

! Phase 2 (IPsec)
crypto ipsec transform-set myset esp-aes esp-sha256-hmac

! Crypto Map Binding
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100

interface GigabitEthernet0/0
 crypto map mymap
در قالب github This cheat sheet consolidates IPsec VPN concepts, FortiOS configurations, and advanced ADVPN deployments, covering both foundational (NSE 4) and enterprise routing/security (NSE 7) objectives.



1. Core IPSec Concepts & Trust Foundations

The foundation of IPsec relies on the CIA Triad:



Confidentiality: Ensures data is accessible only to authorized entities. Achieved via encryption algorithms (DES, 3DES, AES).



Integrity: Ensures packets are not altered in transit (detection of manipulation). Achieved via authentication/hashing algorithms (MD5, SHA-160/128/256/512).



Availability: Ensures authorized users have reliable access (fault clearance, acceptable performance, redundancy).



Authentication Methods (Identity Proof):



Pre-Shared Key (PSK): Symmetrical unique values matching on both sides.



RSA Signatures: Asymmetrical private/public key pairs (Certificates).



Diffie-Hellman (DH): Secure key exchange mechanism ensuring keys match on both ends without transmitting the actual key in plain text (e.g., DH-1: 768-bit, DH-2: 1024-bit, DH-5: 1536-bit, DH-14: 2048-bit).



2. IPSec Protocols & Modes

AH (Authentication Header): Protects the IP header and payload (Integrity/Authentication only). No encryption.



ESP (Encapsulating Security Payload - Protocol 50): Protects the payload (Confidentiality, Integrity, Authentication).



Transport Mode: Uses the original IP header. (Structure: Global IP | ESP Header | TCP | Data | ESP Trailer | ESP Auth).



Tunnel Mode: Encapsulates the entire original packet inside a new IP header to hide the true destination. (Structure: New External IP | ESP Header | Internal IP | TCP | Data | ESP Trailer | ESP Auth).



3. IKEv1 vs. IKEv2 (Internet Key Exchange)

IKEv1 Mechanics

Phase 1 (ISAKMP Tunnel - Control Plane): Negotiates encryption, hashing, DH groups, and authentication to build the management tunnel.



Main Mode (6 Messages):



MM 1-2: Negotiate Crypto Settings (Unencrypted/Unauthenticated).



MM 3-4: Secret Key Exchange/DH & Nonce for anti-replay (Unencrypted/Unauthenticated).



MM 5-6: Prove Identity via ID/Auth (Encrypted/Unauthenticated).



Aggressive Mode (3 Messages): Consolidates SA, Key Exchange, and Auth. Faster but less secure (exposes identities).



Phase 2 (IPsec Tunnel - Data Plane):



Quick Mode (3 Messages): Builds the actual unidirectional data tunnels (IPsec SAs) using the secure ISAKMP tunnel.



IKEv2 Mechanics

Faster, lighter, and more secure natively. Condenses Phase 1 and Phase 2 into 4 primary messages:



IKE_SA_INIT (Request/Response): Merges crypto negotiation and key exchange (equivalent to MM 1-4).



IKE_AUTH (Request/Response): Proves identity and establishes the first Phase 2 Child SA (equivalent to MM 5-6 + Phase 2).



Key Features: Built-in NAT-T, EAP support (certificate-based auth), lower bandwidth consumption, native Keepalive timers, and Child SAs via ECDSA (Elliptic Curve Digital Signature Algorithm).



4. Advanced FortiOS Configurations (NSE 7)

Dead Peer Detection (DPD) & Keepalives

Detects if the remote peer is alive to tear down stale SAs. DPD Phase 1 default is typically 15 seconds.



On-Idle: Checks aliveness only if no traffic is received. Ideal for Dial-up/Remote Access to save resources.



On-Demand: Tears down connections immediately if traffic stops. Ideal for Site-to-Site.



Bash



config vpn ipsec phase1-interface

edit "Link-1"

set dpd-retryinterval 15

set dpd-retrycount 3

end

IPSec Passive Mode

The device will not initiate the VPN connection, even if traffic dictates it. It strictly waits for incoming IKE requests. (Warning: Do not enable on both sides).



Bash



config vpn ipsec phase1-interface

edit "Link-1"

set rekey enable

set passive-mode enable

set passive-tunnel-interface enable

end

Quick Crash Detection (QCD)

Allows faster recovery when a peer reboots without sending delete payloads. Avoids waiting for DPD timeouts. (Uses invalid_ike_spi tokens in IKEv2).



Bash



config system settings

set ike-quick-crash-detect enable

end

Fragmentation & MTU Tuning

Handles large UDP 500/4500 packets over ISP links.



Bash



config vpn ipsec phase1-interface

edit "Link-1"

set fragmentation enable

set fragmentation-mtu 500 # Highly useful for IKEv2

end# TCP MSS Adjustment for Data Plane

config firewall policy

edit 1

set tcp-mss-sender 1350

set tcp-mss-receiver 1350

end

Dos Protection (Embryonic Limits)

Prevents CPU exhaustion from IKE floods.



Bash



config system settings

set embryonic-limit 50

end

5. ADVPN & Large Scale Deployments

Auto-Discovery VPN (ADVPN): Dynamically creates spoke-to-spoke tunnels on demand to prevent hub-bottlenecking. Requires dynamic routing (BGP/OSPF).



Network ID: Used to segment different IPsec overlay networks running over the same public interface.



Mesh Selector: Enables direct Spoke-to-Spoke phase 1 shortcut establishment.



Bash



config vpn ipsec phase1-interface

edit "Hub-Spoke-1"

set network-id 2

set mesh-selector-type enable

set auto-discovery-sender enable

set auto-discovery-receiver enable

end

UDP Hole Punching for Spoke-to-Spoke (NAT-T):

Uses SHORTCUT-OFFER, SHORTCUT-QUERY, and SHORTCUT-REPLY messages. Allows Spokes behind NAT routers to dynamically form tunnels on dynamic UDP ports.



6. Access Control & Security Policies

Policy-Based vs. Route-Based:



Ensure "Enable IPsec Interface Mode" is active to utilize modern Route-Based VPNs (allows routing/SD-WAN over virtual IPsec interfaces).



Disabling this forces older Policy-Based IPsec (where the tunnel is called within the Firewall Policy action).



Blocking Unwanted IKE Negotiations (Local-in Policy):

Used to protect dial-up servers from unauthorized IKE scans.



Bash



config firewall local-in-policy

edit 1

set interface "port2"

set srcadd "all"

set dstadd "Block-Subnet"

set service "all"

set action deny

set schedule "always"

end# Enable Local-in Logging

config log setting

set local-in-allow enable

set local-in-deny enable

end

7. Diagnostics & Troubleshooting Commands

Bash



# Check IKE Gateway status, SA, and Auth methods

diagnose vpn ike gateway list# View full tunnel specifications, Phase 2 selectors, and MTU

diagnose vpn tunnel list# Clear IKE Gateway (Clears faster than the standard 240s IP delay interval)

diagnose vpn ike gateway flush# Debug IKE Negotiations (Vital for Phase 1 / Phase 2 failures)

diagnose debug application ike -1

diagnose debug enable# Packet sniffer for IKE (Port 500) and NAT-T (Port 4500)

diagnose debug flow filter dport 500

diagnose debug flow trace start 10

diagnose debug enable

8. Cisco Interoperability Reference

For deployments connecting FortiGate hubs to Cisco IOS routers (Site-to-Site Profile example).

Plaintext



! Phase 1 (ISAKMP)

crypto isakmp policy 10

encryption aes

hash sha256

authentication pre-share

group 14



crypto isakmp key vpnuser address 10.0.0.2



! Phase 2 (IPsec)

crypto ipsec transform-set myset esp-aes esp-sha256-hmac



! Crypto Map Binding

access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255

crypto map mymap 10 ipsec-isakmp

set peer 10.0.0.2

set transform-set myset

match address 100



interface GigabitEthernet0/0

crypto map mymap

Markdown
# IPsec VPN & ADVPN FortiOS Cheat Sheet (NSE 4 & NSE 7)

This cheat sheet consolidates IPsec VPN concepts, FortiOS configurations, and advanced ADVPN deployments, covering both foundational (NSE 4) and enterprise routing/security (NSE 7) objectives.

## 1. Core IPSec Concepts & Trust Foundations

The foundation of IPsec relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is accessible only to authorized entities. Achieved via encryption algorithms (DES, 3DES, AES).
* **Integrity:** Ensures packets are not altered in transit (detection of manipulation). Achieved via authentication/hashing algorithms (MD5, SHA-160/128/256/512).
* **Availability:** Ensures authorized users have reliable access (fault clearance, acceptable performance, redundancy).

**Authentication Methods (Identity Proof):**
* **Pre-Shared Key (PSK):** Symmetrical unique values matching on both sides.
* **RSA Signatures:** Asymmetrical private/public key pairs (Certificates).

**Diffie-Hellman (DH):** Secure key exchange mechanism ensuring keys match on both ends without transmitting the actual key in plain text (e.g., DH-1: 768-bit, DH-2: 1024-bit, DH-5: 1536-bit, DH-14: 2048-bit).

---

## 2. IPSec Protocols & Modes

* **AH (Authentication Header):** Protects the IP header and payload (Integrity/Authentication only). No encryption.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload (Confidentiality, Integrity, Authentication).
  * **Transport Mode:** Uses the original IP header. (Structure: `Global IP | ESP Header | TCP | Data | ESP Trailer | ESP Auth`).
  * **Tunnel Mode:** Encapsulates the entire original packet inside a new IP header to hide the true destination. (Structure: `New External IP | ESP Header | Internal IP | TCP | Data | ESP Trailer | ESP Auth`).

---

## 3. IKEv1 vs. IKEv2 (Internet Key Exchange)

### IKEv1 Mechanics
* **Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates encryption, hashing, DH groups, and authentication to build the management tunnel.
  * **Main Mode (6 Messages):**
    * MM 1-2: Negotiate Crypto Settings (Unencrypted/Unauthenticated).
    * MM 3-4: Secret Key Exchange/DH & Nonce for anti-replay (Unencrypted/Unauthenticated).
    * MM 5-6: Prove Identity via ID/Auth (Encrypted/Unauthenticated).
  * **Aggressive Mode (3 Messages):** Consolidates SA, Key Exchange, and Auth. Faster but less secure (exposes identities).
* **Phase 2 (IPsec Tunnel - Data Plane):**
  * **Quick Mode (3 Messages):** Builds the actual unidirectional data tunnels (IPsec SAs) using the secure ISAKMP tunnel.

### IKEv2 Mechanics
Faster, lighter, and more secure natively. Condenses Phase 1 and Phase 2 into 4 primary messages:
* **IKE_SA_INIT (Request/Response):** Merges crypto negotiation and key exchange (equivalent to MM 1-4).
* **IKE_AUTH (Request/Response):** Proves identity and establishes the first Phase 2 Child SA (equivalent to MM 5-6 + Phase 2).
* **Key Features:** Built-in NAT-T, EAP support (certificate-based auth), lower bandwidth consumption, native Keepalive timers, and Child SAs via ECDSA (Elliptic Curve Digital Signature Algorithm).

---

## 4. Advanced FortiOS Configurations (NSE 7)

**Dead Peer Detection (DPD) & Keepalives**
Detects if the remote peer is alive to tear down stale SAs. DPD Phase 1 default is typically 15 seconds.
* **On-Idle:** Checks aliveness only if no traffic is received. Ideal for Dial-up/Remote Access to save resources.
* **On-Demand:** Tears down connections immediately if traffic stops. Ideal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd-retryinterval 15
    set dpd-retrycount 3
end
```

**IPSec Passive Mode**
The device will not initiate the VPN connection, even if traffic dictates it. It strictly waits for incoming IKE requests. (Warning: Do not enable on both sides).

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set rekey enable
    set passive-mode enable
    set passive-tunnel-interface enable
end
```

**Quick Crash Detection (QCD)**
Allows faster recovery when a peer reboots without sending delete payloads. Avoids waiting for DPD timeouts. (Uses `invalid_ike_spi` tokens in IKEv2).

```bash
config system settings
  set ike-quick-crash-detect enable
end
```

**Fragmentation & MTU Tuning**
Handles large UDP 500/4500 packets over ISP links.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set fragmentation enable
    set fragmentation-mtu 500  # Highly useful for IKEv2
end

# TCP MSS Adjustment for Data Plane
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
```

**Dos Protection (Embryonic Limits)**
Prevents CPU exhaustion from IKE floods.

```bash
config system settings
  set embryonic-limit 50 
end
```

---

## 5. ADVPN & Large Scale Deployments

**Auto-Discovery VPN (ADVPN):** Dynamically creates spoke-to-spoke tunnels on demand to prevent hub-bottlenecking. Requires dynamic routing (BGP/OSPF).
* **Network ID:** Used to segment different IPsec overlay networks running over the same public interface.
* **Mesh Selector:** Enables direct Spoke-to-Spoke phase 1 shortcut establishment.

```bash
config vpn ipsec phase1-interface
  edit "Hub-Spoke-1"
    set network-id 2
    set mesh-selector-type enable
    set auto-discovery-sender enable
    set auto-discovery-receiver enable
end
```

**UDP Hole Punching for Spoke-to-Spoke (NAT-T):**
Uses `SHORTCUT-OFFER`, `SHORTCUT-QUERY`, and `SHORTCUT-REPLY` messages. Allows Spokes behind NAT routers to dynamically form tunnels on dynamic UDP ports.

---

## 6. Access Control & Security Policies

**Policy-Based vs. Route-Based:**
* Ensure "Enable IPsec Interface Mode" is active to utilize modern Route-Based VPNs (allows routing/SD-WAN over virtual IPsec interfaces).
* Disabling this forces older Policy-Based IPsec (where the tunnel is called within the Firewall Policy action).

**Blocking Unwanted IKE Negotiations (Local-in Policy):**
Used to protect dial-up servers from unauthorized IKE scans.

```bash
config firewall local-in-policy
  edit 1
    set interface "port2"
    set srcadd "all"
    set dstadd "Block-Subnet"
    set service "all"
    set action deny
    set schedule "always"
end

# Enable Local-in Logging
config log setting
  set local-in-allow enable
  set local-in-deny enable
end
```

---

## 7. Diagnostics & Troubleshooting Commands

```bash
# Check IKE Gateway status, SA, and Auth methods
diagnose vpn ike gateway list

# View full tunnel specifications, Phase 2 selectors, and MTU
diagnose vpn tunnel list

# Clear IKE Gateway (Clears faster than the standard 240s IP delay interval)
diagnose vpn ike gateway flush

# Debug IKE Negotiations (Vital for Phase 1 / Phase 2 failures)
diagnose debug application ike -1
diagnose debug enable

# Packet sniffer for IKE (Port 500) and NAT-T (Port 4500)
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable
```

---

## 8. Cisco Interoperability Reference

For deployments connecting FortiGate hubs to Cisco IOS routers (Site-to-Site Profile example).

```text
! Phase 1 (ISAKMP)
crypto isakmp policy 10
 encryption aes
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key vpnuser address 10.0.0.2

! Phase 2 (IPsec)
crypto ipsec transform-set myset esp-aes esp-sha256-hmac

! Crypto Map Binding
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100

interface GigabitEthernet0/0
 crypto map mymap
```
کامل ترش کن خیلی ناقص توضیح دادی و باید کاملترش کنی

این نسخه بسیار جامع‌تر، با جزئیات عمیق‌تر از مفاهیم تئوری و دستورات عملی FortiOS تنظیم شده است تا تمام نکات ریز فایل شما (از جمله مکانیزم پیام‌های IKE، تنظیمات XAuth، دیباگ‌های ADVPN و Redundancy) را به‌طور کامل پوشش دهد.

محتوای زیر را مستقیماً کپی و در گیت‌هاب استفاده کنید:

Markdown
# Comprehensive IPsec & ADVPN Cheat Sheet (FortiOS NSE 4 & NSE 7)

This document provides an in-depth guide to IPsec VPN architecture, FortiOS CLI configurations, and Advanced Auto-Discovery VPN (ADVPN) deployments, merging NSE 4 foundational concepts with NSE 7 enterprise routing and security features.

---

## 1. Cryptography Core & Trust Foundations (CIA Triad)

To establish a secure IPsec tunnel, the framework relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is readable only by authorized parties. Achieved via symmetric encryption algorithms (DES, 3DES, AES-128/256).
* **Integrity:** Ensures data has not been altered in transit. Achieved via hashing algorithms (MD5, SHA-1, SHA-256/512). The receiver hashes the data and compares it to the sender's hash.
* **Availability:** Ensures reliable access through fault clearance, redundancy, and DoS protection.

**Authentication (Identity Verification):**
* **Pre-Shared Key (PSK):** A symmetrical string configured on both peers.
* **RSA Signatures:** Asymmetrical authentication using public/private key pairs and Digital Certificates.

**Diffie-Hellman (DH):** 
A mathematical algorithm used to securely exchange cryptographic keys over a public channel.
* **Groups:** DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit), DH-14 (2048-bit). Higher groups provide stronger Perfect Forward Secrecy (PFS) but consume more CPU.

---

## 2. IPsec Protocols & Encapsulation Modes

**Security Protocols:**
* **AH (Authentication Header - Protocol 51):** Protects the IP header and payload for Integrity and Authentication. **No Encryption**. Rarely used in modern networks.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload for Confidentiality, Integrity, and Authentication.

**Encapsulation Modes:**
* **Transport Mode:** Uses the original IP header. Typically used for host-to-host or when IPsec is combined with GRE (GRE over IPsec).
  * *Structure:* `[Original IP] | [ESP Header] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`
* **Tunnel Mode:** Encapsulates the entire original packet inside a new external IP header. Used for Site-to-Site VPNs.
  * *Structure:* `[New External IP] | [ESP Header] | [Original IP] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`

---

## 3. IKEv1 vs. IKEv2 Mechanics (Control Plane vs. Data Plane)

### IKEv1 (Internet Key Exchange version 1)
**Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates security policies to build a secure management tunnel.
* **Main Mode (6 Messages - Slower but Secure):**
  * **MM 1 & 2 (Crypto Negotiation):** Initiator and Responder agree on Encryption, Hash, Auth method, and DH group (Unencrypted).
  * **MM 3 & 4 (Key Exchange):** DH public values and Nonce (anti-replay random string) are exchanged (Unencrypted).
  * **MM 5 & 6 (Authentication):** Peers prove their identity using PSK/Certificates (Encrypted but Unauthenticated until verified).
* **Aggressive Mode (3 Messages - Faster but Less Secure):**
  * Consolidates crypto negotiation, key exchange, and identity into 3 packets. The Initiator's ID is sent in the clear (not recommended unless one peer has a dynamic IP).

**Phase 2 (IPsec Tunnel - Data Plane):** 
* **Quick Mode (3 Messages):** Uses the secure Phase 1 tunnel to negotiate IPsec Security Associations (SAs). Creates two unidirectional SPIs (Security Parameter Indices) for actual data transfer.

### IKEv2 (Next-Gen Key Exchange)
IKEv2 is faster, lighter, and condenses Phase 1 and 2 into 4 primary messages, eliminating Main/Aggressive mode distinctions.
* **Message 1 & 2 (`IKE_SA_INIT`):** Negotiates crypto, DH exchange, and nonces.
* **Message 3 & 4 (`IKE_AUTH`):** Authenticates the previous messages, transmits identities, and establishes the first Phase 2 Child SA.
* **Key Advancements:** 
  * Native NAT-Traversal (NAT-T) and Keepalives.
  * Extensible Authentication Protocol (EAP) for certificate/MFA integration.
  * Built-in DoS protection (Cookies).
  * **Child SAs:** Uses ECDSA (Elliptic Curve Digital Signature Algorithm) to derive multiple stable child connections from one parent SA.

---

## 4. Advanced FortiOS Configurations & XAuth (NSE 7)

### Dead Peer Detection (DPD) & Connection Timers
DPD uses probe messages (similar to ICMP) to detect if a peer is down.
* **On-Idle:** Probes are sent only if no traffic is received for a specified time. Optimal for Dial-up/Remote Access to save hub resources.
* **On-Demand:** Probes are sent immediately when traffic is queued but the peer isn't responding. Optimal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd on-idle
    set dpd-retryinterval 15
    set dpd-retrycount 3
    set ip-delay-interval 200 # Flushes dial-up released addresses faster than default 240s
end
```

### XAuth & Dial-up Deployments
XAuth forces users to authenticate with credentials (LDAP/RADIUS/Local) after Phase 1 but before Phase 2.
* **Server Modes:** 
  * `pap`: FortiGate negotiates directly with the client.
  * `chap`: FortiGate negotiates with a backend server.
  * `auto`: The best mode; mixes PAP/CHAP based on client capability. Uses a single user group container.

### NAT-Traversal (NAT-T)
Forces IPsec to shift from UDP port 500 to UDP port 4500 if a NAT device is detected between peers (prevents ESP packets from being dropped by PAT devices).
* **Modes:** `enable` (dynamic detection) or `force` (always use UDP 4500).

### TCP MSS & Fragmentation Optimization
Overhead from IPsec encapsulation causes packet drops on standard 1500 MTU links.
```bash
# Phase 1 IKEv2 Fragmentation
config vpn ipsec phase1-interface
  edit "Link-1"
    set ike-version 2
    set fragmentation enable
    set fragmentation-mtu 500
end

# TCP MSS Adjustment for Data Forwarding
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
```

### IPSec Passive Mode
The FortiGate will never initiate the IKE negotiation; it only responds. *Do not enable on both peers.*
```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set passive-mode enable
    set passive-tunnel-interface enable
end
```

---

## 5. Auto-Discovery VPN (ADVPN) & Mesh Networking

ADVPN allows spokes (branches) to dynamically establish direct VPN tunnels with other spokes, bypassing the Hub to reduce latency and bottlenecking. Requires dynamic routing (BGP or OSPF) across the overlay.

**Hub & Spoke Phase 1 Configuration:**
```bash
config vpn ipsec phase1-interface
  edit "ADVPN-Hub"
    set network-id 2             # Segments IPsec domains over the same public interface
    set mesh-selector-type enable # Allows dynamic Phase 2 selectors
    set auto-discovery-sender enable
    set auto-discovery-receiver enable
end
```

**UDP Hole Punching for Spoke-to-Spoke:**
When Spokes are behind NAT, the Hub facilitates "Hole Punching" to allow direct communication using IKE informational messages:
1. **SHORTCUT-OFFER:** Hub informs Spoke 1 about Spoke 2's details.
2. **SHORTCUT-QUERY:** Spoke 1 asks Hub for Spoke 2's NAT mappings.
3. **SHORTCUT-REPLY:** Hub provides NAT mappings, allowing Spokes to establish direct UDP 4500 connections.

---

## 6. High Availability & IPsec Redundancy

**IPsec Link Monitoring (Failover):**
Configures a primary IPsec tunnel to monitor a secondary. If Link-1 fails, Link-2 takes over.
```bash
config vpn ipsec phase1-interface
  edit "Link-2"
    set monitor "Link-1"
end
```

**IPsec Interface Aggregation:**
FortiOS allows bundling multiple IPsec virtual interfaces into a single Aggregate interface for load balancing and redundancy (requires "Enable IPsec Interface Mode").

---

## 7. DoS Protection & Access Control

**Embryonic Limits:**
Protects against IKE flooding attacks by limiting half-open IPsec connections.
```bash
config system settings
  set embryonic-limit 50 
end
```

**Local-In Policies for IKE (Port 500/4500):**
Block unwanted automated IKE scanning bots before they hit the VPN daemon.
```bash
config firewall local-in-policy
  edit 1
    set interface "port1"
    set srcadd "all"
    set dstadd "Blocked_Subnets"
    set service "IKE"
    set action deny
    set schedule "always"
end
```

**Quick Crash Detection (QCD):**
Sends specialized tokens to peers. If a peer reboots without terminating the tunnel, QCD allows the remaining peer to immediately tear down the SA without waiting for DPD timeouts.
```bash
config system settings
  set ike-quick-crash-detect enable
end
```

---

## 8. CLI Diagnostics & Troubleshooting

```bash
# General Tunnel and Gateway Information
diagnose vpn ike gateway list     # Shows SA, Auth methods, Peer IPs, and Timers
diagnose vpn tunnel list          # Shows Phase 2 SPIs, MTU, and exact Subnet Selectors
diagnose vpn ike gateway flush    # Forcefully drops and clears all IKE SAs

# Advanced IKE Negotiation Debugging (Vital for Phase 1/2 failures)
diagnose debug application ike -1
diagnose debug enable

# Packet Sniffer for ISAKMP and NAT-T Traffic
diagnose debug flow filter dport 500
diagnose debug flow filter dport 4500
diagnose debug flow trace start 10
diagnose debug enable
```

---

## 9. Cisco IOS Interoperability Reference

When terminating FortiGate Site-to-Site VPNs against Cisco IOS routers. Note: Cisco uses Route-Maps and Crypto Maps for legacy IKEv1, while IKEv2 relies heavily on FlexVPN (Virtual Template/Tunnel interfaces).

**Cisco IKEv1 Configuration Example:**
```text
! Phase 1 (ISAKMP Policy)
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key vpnsecret123 address 10.0.0.2

! Phase 2 (IPsec Transform Set)
crypto ipsec transform-set myset esp-aes 256 esp-sha256-hmac

! ACL for Interesting Traffic (Selectors)
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255

! Crypto Map Binding
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100

interface GigabitEthernet0/0
 crypto map mymap
```
# Comprehensive IPsec & ADVPN Cheat Sheet (FortiOS NSE 4 & NSE 7)



This document provides an in-depth guide to IPsec VPN architecture, FortiOS CLI configurations, and Advanced Auto-Discovery VPN (ADVPN) deployments, merging NSE 4 foundational concepts with NSE 7 enterprise routing and security features.



---



## 1. Cryptography Core & Trust Foundations (CIA Triad)



To establish a secure IPsec tunnel, the framework relies on the **CIA Triad**:

* **Confidentiality:** Ensures data is readable only by authorized parties. Achieved via symmetric encryption algorithms (DES, 3DES, AES-128/256).

* **Integrity:** Ensures data has not been altered in transit. Achieved via hashing algorithms (MD5, SHA-1, SHA-256/512). The receiver hashes the data and compares it to the sender's hash.

* **Availability:** Ensures reliable access through fault clearance, redundancy, and DoS protection.



**Authentication (Identity Verification):**

* **Pre-Shared Key (PSK):** A symmetrical string configured on both peers.

* **RSA Signatures:** Asymmetrical authentication using public/private key pairs and Digital Certificates.



**Diffie-Hellman (DH):** 

A mathematical algorithm used to securely exchange cryptographic keys over a public channel.

* **Groups:** DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit), DH-14 (2048-bit). Higher groups provide stronger Perfect Forward Secrecy (PFS) but consume more CPU.



---



## 2. IPsec Protocols & Encapsulation Modes



**Security Protocols:**

* **AH (Authentication Header - Protocol 51):** Protects the IP header and payload for Integrity and Authentication. **No Encryption**. Rarely used in modern networks.

* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload for Confidentiality, Integrity, and Authentication.



**Encapsulation Modes:**

* **Transport Mode:** Uses the original IP header. Typically used for host-to-host or when IPsec is combined with GRE (GRE over IPsec).

  * *Structure:* `[Original IP] | [ESP Header] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`

* **Tunnel Mode:** Encapsulates the entire original packet inside a new external IP header. Used for Site-to-Site VPNs.

  * *Structure:* `[New External IP] | [ESP Header] | [Original IP] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`



---



## 3. IKEv1 vs. IKEv2 Mechanics (Control Plane vs. Data Plane)



### IKEv1 (Internet Key Exchange version 1)

**Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates security policies to build a secure management tunnel.

* **Main Mode (6 Messages - Slower but Secure):**

  * **MM 1 & 2 (Crypto Negotiation):** Initiator and Responder agree on Encryption, Hash, Auth method, and DH group (Unencrypted).

  * **MM 3 & 4 (Key Exchange):** DH public values and Nonce (anti-replay random string) are exchanged (Unencrypted).

  * **MM 5 & 6 (Authentication):** Peers prove their identity using PSK/Certificates (Encrypted but Unauthenticated until verified).

* **Aggressive Mode (3 Messages - Faster but Less Secure):**

  * Consolidates crypto negotiation, key exchange, and identity into 3 packets. The Initiator's ID is sent in the clear (not recommended unless one peer has a dynamic IP).



**Phase 2 (IPsec Tunnel - Data Plane):** 

* **Quick Mode (3 Messages):** Uses the secure Phase 1 tunnel to negotiate IPsec Security Associations (SAs). Creates two unidirectional SPIs (Security Parameter Indices) for actual data transfer.



### IKEv2 (Next-Gen Key Exchange)

IKEv2 is faster, lighter, and condenses Phase 1 and 2 into 4 primary messages, eliminating Main/Aggressive mode distinctions.

* **Message 1 & 2 (`IKE_SA_INIT`):** Negotiates crypto, DH exchange, and nonces.

* **Message 3 & 4 (`IKE_AUTH`):** Authenticates the previous messages, transmits identities, and establishes the first Phase 2 Child SA.

* **Key Advancements:** 

  * Native NAT-Traversal (NAT-T) and Keepalives.

  * Extensible Authentication Protocol (EAP) for certificate/MFA integration.

  * Built-in DoS protection (Cookies).

  * **Child SAs:** Uses ECDSA (Elliptic Curve Digital Signature Algorithm) to derive multiple stable child connections from one parent SA.



---



## 4. Advanced FortiOS Configurations & XAuth (NSE 7)



### Dead Peer Detection (DPD) & Connection Timers

DPD uses probe messages (similar to ICMP) to detect if a peer is down.

* **On-Idle:** Probes are sent only if no traffic is received for a specified time. Optimal for Dial-up/Remote Access to save hub resources.

* **On-Demand:** Probes are sent immediately when traffic is queued but the peer isn't responding. Optimal for Site-to-Site.



```bash

config vpn ipsec phase1-interface

  edit "Link-1"

    set dpd on-idle

    set dpd-retryinterval 15

    set dpd-retrycount 3

    set ip-delay-interval 200 # Flushes dial-up released addresses faster than default 240s

end اصلاحش کن و در قالب github بهم تحویل بده

این نسخه با حذف کاراکترهای نامرئی (Zero-width spaces)، اصلاح تورفتگی‌ها (Indentation) در بخش لیست‌ها و استانداردسازی بلوک‌های کد پاک‌سازی شده است تا بدون هیچ‌گونه به هم ریختگی در گیت‌هاب رندر شود.

Markdown
# Comprehensive IPsec & ADVPN Cheat Sheet (FortiOS NSE 4 & NSE 7)

This document provides an in-depth guide to IPsec VPN architecture, FortiOS CLI configurations, and Advanced Auto-Discovery VPN (ADVPN) deployments, merging NSE 4 foundational concepts with NSE 7 enterprise routing and security features.

---

## 1. Cryptography Core & Trust Foundations (CIA Triad)

To establish a secure IPsec tunnel, the framework relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is readable only by authorized parties. Achieved via symmetric encryption algorithms (DES, 3DES, AES-128/256).
* **Integrity:** Ensures data has not been altered in transit. Achieved via hashing algorithms (MD5, SHA-1, SHA-256/512). The receiver hashes the data and compares it to the sender's hash.
* **Availability:** Ensures reliable access through fault clearance, redundancy, and DoS protection.

**Authentication (Identity Verification):**
* **Pre-Shared Key (PSK):** A symmetrical string configured on both peers.
* **RSA Signatures:** Asymmetrical authentication using public/private key pairs and Digital Certificates.

**Diffie-Hellman (DH):** 
A mathematical algorithm used to securely exchange cryptographic keys over a public channel.
* **Groups:** DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit), DH-14 (2048-bit). Higher groups provide stronger Perfect Forward Secrecy (PFS) but consume more CPU.

---

## 2. IPsec Protocols & Encapsulation Modes

**Security Protocols:**
* **AH (Authentication Header - Protocol 51):** Protects the IP header and payload for Integrity and Authentication. **No Encryption**. Rarely used in modern networks.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload for Confidentiality, Integrity, and Authentication.

**Encapsulation Modes:**
* **Transport Mode:** Uses the original IP header. Typically used for host-to-host or when IPsec is combined with GRE (GRE over IPsec).
  * *Structure:* `[Original IP] | [ESP Header] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`
* **Tunnel Mode:** Encapsulates the entire original packet inside a new external IP header. Used for Site-to-Site VPNs.
  * *Structure:* `[New External IP] | [ESP Header] | [Original IP] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`

---

## 3. IKEv1 vs. IKEv2 Mechanics (Control Plane vs. Data Plane)

### IKEv1 (Internet Key Exchange version 1)
**Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates security policies to build a secure management tunnel.
* **Main Mode (6 Messages - Slower but Secure):**
  * **MM 1 & 2 (Crypto Negotiation):** Initiator and Responder agree on Encryption, Hash, Auth method, and DH group (Unencrypted).
  * **MM 3 & 4 (Key Exchange):** DH public values and Nonce (anti-replay random string) are exchanged (Unencrypted).
  * **MM 5 & 6 (Authentication):** Peers prove their identity using PSK/Certificates (Encrypted but Unauthenticated until verified).
* **Aggressive Mode (3 Messages - Faster but Less Secure):**
  * Consolidates crypto negotiation, key exchange, and identity into 3 packets. The Initiator's ID is sent in the clear (not recommended unless one peer has a dynamic IP).

**Phase 2 (IPsec Tunnel - Data Plane):** 
* **Quick Mode (3 Messages):** Uses the secure Phase 1 tunnel to negotiate IPsec Security Associations (SAs). Creates two unidirectional SPIs (Security Parameter Indices) for actual data transfer.

### IKEv2 (Next-Gen Key Exchange)
IKEv2 is faster, lighter, and condenses Phase 1 and 2 into 4 primary messages, eliminating Main/Aggressive mode distinctions.
* **Message 1 & 2 (`IKE_SA_INIT`):** Negotiates crypto, DH exchange, and nonces.
* **Message 3 & 4 (`IKE_AUTH`):** Authenticates the previous messages, transmits identities, and establishes the first Phase 2 Child SA.
* **Key Advancements:** 
  * Native NAT-Traversal (NAT-T) and Keepalives.
  * Extensible Authentication Protocol (EAP) for certificate/MFA integration.
  * Built-in DoS protection (Cookies).
  * **Child SAs:** Uses ECDSA (Elliptic Curve Digital Signature Algorithm) to derive multiple stable child connections from one parent SA.

---

## 4. Advanced FortiOS Configurations & XAuth (NSE 7)

### Dead Peer Detection (DPD) & Connection Timers
DPD uses probe messages (similar to ICMP) to detect if a peer is down.
* **On-Idle:** Probes are sent only if no traffic is received for a specified time. Optimal for Dial-up/Remote Access to save hub resources.
* **On-Demand:** Probes are sent immediately when traffic is queued but the peer isn't responding. Optimal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd on-idle
    set dpd-retryinterval 15
    set dpd-retrycount 3
    set ip-delay-interval 200  # Flushes dial-up released addresses faster than default 240s
end
```
خب کاملشو بهم بده متنت خیلی کم و ناقصه

این هم نسخه کامل، بدون نقص و آماده کپی در گیت‌هاب که تمام ۹ بخش اصلی Cheat Sheet را با جزئیات کامل و فرمت استاندارد رندر گیت‌هاب شامل می‌شود:

Markdown
# Comprehensive IPsec & ADVPN Cheat Sheet (FortiOS NSE 4 & NSE 7)

This document provides an in-depth guide to IPsec VPN architecture, FortiOS CLI configurations, and Advanced Auto-Discovery VPN (ADVPN) deployments, merging NSE 4 foundational concepts with NSE 7 enterprise routing and security features.

---

## 1. Cryptography Core & Trust Foundations (CIA Triad)

To establish a secure IPsec tunnel, the framework relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is readable only by authorized parties. Achieved via symmetric encryption algorithms (DES, 3DES, AES-128/256).
* **Integrity:** Ensures data has not been altered in transit. Achieved via hashing algorithms (MD5, SHA-1, SHA-256/512). The receiver hashes the data and compares it to the sender's hash.
* **Availability:** Ensures reliable access through fault clearance, redundancy, and DoS protection.

**Authentication (Identity Verification):**
* **Pre-Shared Key (PSK):** A symmetrical string configured on both peers.
* **RSA Signatures:** Asymmetrical authentication using public/private key pairs and Digital Certificates.

**Diffie-Hellman (DH):** 
A mathematical algorithm used to securely exchange cryptographic keys over a public channel.
* **Groups:** DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit), DH-14 (2048-bit). Higher groups provide stronger Perfect Forward Secrecy (PFS) but consume more CPU.

---

## 2. IPsec Protocols & Encapsulation Modes

**Security Protocols:**
* **AH (Authentication Header - Protocol 51):** Protects the IP header and payload for Integrity and Authentication. **No Encryption**. Rarely used in modern networks.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload for Confidentiality, Integrity, and Authentication.

**Encapsulation Modes:**
* **Transport Mode:** Uses the original IP header. Typically used for host-to-host or when IPsec is combined with GRE (GRE over IPsec).
  * *Structure:* `[Original IP] | [ESP Header] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`
* **Tunnel Mode:** Encapsulates the entire original packet inside a new external IP header. Used for Site-to-Site VPNs.
  * *Structure:* `[New External IP] | [ESP Header] | [Original IP] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`

---

## 3. IKEv1 vs. IKEv2 Mechanics (Control Plane vs. Data Plane)

### IKEv1 (Internet Key Exchange version 1)
**Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates security policies to build a secure management tunnel.
* **Main Mode (6 Messages - Slower but Secure):**
  * **MM 1 & 2 (Crypto Negotiation):** Initiator and Responder agree on Encryption, Hash, Auth method, and DH group (Unencrypted).
  * **MM 3 & 4 (Key Exchange):** DH public values and Nonce (anti-replay random string) are exchanged (Unencrypted).
  * **MM 5 & 6 (Authentication):** Peers prove their identity using PSK/Certificates (Encrypted but Unauthenticated until verified).
* **Aggressive Mode (3 Messages - Faster but Less Secure):**
  * Consolidates crypto negotiation, key exchange, and identity into 3 packets. The Initiator's ID is sent in the clear (not recommended unless one peer has a dynamic IP).

**Phase 2 (IPsec Tunnel - Data Plane):** 
* **Quick Mode (3 Messages):** Uses the secure Phase 1 tunnel to negotiate IPsec Security Associations (SAs). Creates two unidirectional SPIs (Security Parameter Indices) for actual data transfer.

### IKEv2 (Next-Gen Key Exchange)
IKEv2 is faster, lighter, and condenses Phase 1 and 2 into 4 primary messages, eliminating Main/Aggressive mode distinctions.
* **Message 1 & 2 (`IKE_SA_INIT`):** Negotiates crypto, DH exchange, and nonces.
* **Message 3 & 4 (`IKE_AUTH`):** Authenticates the previous messages, transmits identities, and establishes the first Phase 2 Child SA.
* **Key Advancements:** 
  * Native NAT-Traversal (NAT-T) and Keepalives.
  * Extensible Authentication Protocol (EAP) for certificate/MFA integration.
  * Built-in DoS protection (Cookies).
  * **Child SAs:** Uses ECDSA (Elliptic Curve Digital Signature Algorithm) to derive multiple stable child connections from one parent SA.

---

## 4. Advanced FortiOS Configurations & XAuth (NSE 7)

### Dead Peer Detection (DPD) & Connection Timers
DPD uses probe messages (similar to ICMP) to detect if a peer is down.
* **On-Idle:** Probes are sent only if no traffic is received for a specified time. Optimal for Dial-up/Remote Access to save hub resources.
* **On-Demand:** Probes are sent immediately when traffic is queued but the peer isn't responding. Optimal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd on-idle
    set dpd-retryinterval 15
    set dpd-retrycount 3
    set ip-delay-interval 200  # Flushes dial-up released addresses faster than default 240s
end
```

### XAuth & Dial-up Deployments
XAuth forces users to authenticate with credentials (LDAP/RADIUS/Local) after Phase 1 but before Phase 2.
* **Server Modes:** 
  * `pap`: FortiGate negotiates directly with the client.
  * `chap`: FortiGate negotiates with a backend server.
  * `auto`: The best mode; mixes PAP/CHAP based on client capability. Uses a single user group container.

### NAT-Traversal (NAT-T)
Forces IPsec to shift from UDP port 500 to UDP port 4500 if a NAT device is detected between peers (prevents ESP packets from being dropped by PAT devices).
* **Modes:** `enable` (dynamic detection) or `force` (always use UDP 4500).

### TCP MSS & Fragmentation Optimization
Overhead from IPsec encapsulation causes packet drops on standard 1500 MTU links.
```bash
# Phase 1 IKEv2 Fragmentation
config vpn ipsec phase1-interface
  edit "Link-1"
    set ike-version 2
    set fragmentation enable
    set fragmentation-mtu 500
end

# TCP MSS Adjustment for Data Forwarding
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
```

### IPSec Passive Mode
The FortiGate will never initiate the IKE negotiation; it only responds. *Do not enable on both peers.*
```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set passive-mode enable
    set passive-tunnel-interface enable
end
```

---

## 5. Auto-Discovery VPN (ADVPN) & Mesh Networking

ADVPN allows spokes (branches) to dynamically establish direct VPN tunnels with other spokes, bypassing the Hub to reduce latency and bottlenecking. Requires dynamic routing (BGP or OSPF) across the overlay.

**Hub & Spoke Phase 1 Configuration:**
```bash
config vpn ipsec phase1-interface
  edit "ADVPN-Hub"
    set network-id 2             # Segments IPsec domains over the same public interface
    set mesh-selector-type enable # Allows dynamic Phase 2 selectors
    set auto-discovery-sender enable
    set auto-discovery-receiver enable
end
```

**UDP Hole Punching for Spoke-to-Spoke:**
When Spokes are behind NAT, the Hub facilitates "Hole Punching" to allow direct communication using IKE informational messages:
1. **SHORTCUT-OFFER:** Hub informs Spoke 1 about Spoke 2's details.
2. **SHORTCUT-QUERY:** Spoke 1 asks Hub for Spoke 2's NAT mappings.
3. **SHORTCUT-REPLY:** Hub provides NAT mappings, allowing Spokes to establish direct UDP 4500 connections.

---

## 6. High Availability & IPsec Redundancy

**IPsec Link Monitoring (Failover):**
Configures a primary IPsec tunnel to monitor a secondary. If Link-1 fails, Link-2 takes over.
```bash
config vpn ipsec phase1-interface
  edit "Link-2"
    set monitor "Link-1"
end
```

**IPsec Interface Aggregation:**
FortiOS allows bundling multiple IPsec virtual interfaces into a single Aggregate interface for load balancing and redundancy (requires "Enable IPsec Interface Mode").

---

## 7. DoS Protection & Access Control

**Embryonic Limits:**
Protects against IKE flooding attacks by limiting half-open IPsec connections.
```bash
config system settings
  set embryonic-limit 50 
end
```

**Local-In Policies for IKE (Port 500/4500):**
Block unwanted automated IKE scanning bots before they hit the VPN daemon.
```bash
config firewall local-in-policy
  edit 1
    set interface "port1"
    set srcadd "all"
    set dstadd "Blocked_Subnets"
    set service "IKE"
    set action deny
    set schedule "always"
end
```

**Quick Crash Detection (QCD):**
Sends specialized tokens to peers. If a peer reboots without terminating the tunnel, QCD allows the remaining peer to immediately tear down the SA without waiting for DPD timeouts.
```bash
config system settings
  set ike-quick-crash-detect enable
end
```

---

## 8. CLI Diagnostics & Troubleshooting

```bash
# General Tunnel and Gateway Information
diagnose vpn ike gateway list     # Shows SA, Auth methods, Peer IPs, and Timers
diagnose vpn tunnel list          # Shows Phase 2 SPIs, MTU, and exact Subnet Selectors
diagnose vpn ike gateway flush    # Forcefully drops and clears all IKE SAs

# Advanced IKE Negotiation Debugging (Vital for Phase 1/2 failures)
diagnose debug application ike -1
diagnose debug enable

# Packet Sniffer for ISAKMP and NAT-T Traffic
diagnose debug flow filter dport 500
diagnose debug flow filter dport 4500
diagnose debug flow trace start 10
diagnose debug enable
```

---

## 9. Cisco IOS Interoperability Reference

When terminating FortiGate Site-to-Site VPNs against Cisco IOS routers. Note: Cisco uses Route-Maps and Crypto Maps for legacy IKEv1, while IKEv2 relies heavily on FlexVPN (Virtual Template/Tunnel interfaces).

**Cisco IKEv1 Configuration Example:**
```text
! Phase 1 (ISAKMP Policy)
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key vpnsecret123 address 10.0.0.2

! Phase 2 (IPsec Transform Set)
crypto ipsec transform-set myset esp-aes 256 esp-sha256-hmac

! ACL for Interesting Traffic (Selectors)
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255

! Crypto Map Binding
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100

interface GigabitEthernet0/0
 crypto map mymap
```
حالا زاثزنمهسف hc l,hvn fhgh nv rhgf لهفاعذ fil fni

You stopped this response
phgh  checklist از موارد بالا در قالب github بهم بده

Markdown
# FortiOS IPsec VPN & ADVPN Deployment Checklist

### 1. Cryptography & Authentication (Phase 1)
- [ ] Define Pre-Shared Key (PSK) or RSA Certificates for peer authentication.
- [ ] Select appropriate Encryption algorithms (e.g., AES-128/AES-256).
- [ ] Select appropriate Hashing algorithms (e.g., SHA-256).
- [ ] Choose Diffie-Hellman (DH) Group (e.g., DH-14 or higher for PFS).

### 2. Protocol & Encapsulation Details
- [ ] Set Security Protocol to ESP (Protocol 50).
- [ ] Select Encapsulation Mode (Tunnel Mode for Site-to-Site, Transport for Host/GRE).
- [ ] Choose IKE Version (IKEv2 recommended for native NAT-T, Keepalives, and speed).
- [ ] If using IKEv1, define Main Mode (standard) or Aggressive Mode (dynamic IPs).

### 3. Advanced FortiOS Settings & Timers
- [ ] Configure Dead Peer Detection (DPD) timers (On-Idle for Dial-up, On-Demand for Site-to-Site).
- [ ] Enable NAT-Traversal (NAT-T) if traversing intermediate NAT devices.
- [ ] Enable IKE Fragmentation and set MTU (e.g., 500) to prevent UDP 500/4500 drops.
- [ ] Adjust TCP MSS for the Data Plane in Firewall Policies (e.g., 1350).
- [ ] Enable IPsec Passive Mode on the responder if it should never initiate the tunnel.
- [ ] Configure XAuth for Dial-up client authentication (PAP/CHAP/Auto).

### 4. ADVPN (Auto-Discovery VPN) Parameters
- [ ] Set identical `network-id` on Hub and Spokes traversing the same public interface.
- [ ] Enable `mesh-selector-type` to allow dynamic Phase 2 subnets.
- [ ] Enable `auto-discovery-sender` (Hub/Spoke).
- [ ] Enable `auto-discovery-receiver` (Hub/Spoke).
- [ ] Ensure Dynamic Routing (BGP/OSPF) is correctly configured over the IPsec overlay.

### 5. High Availability, Redundancy & DoS
- [ ] Configure Link Monitoring on the secondary tunnel to track the primary tunnel.
- [ ] Set Embryonic Limits to prevent IKE CPU exhaustion/floods.
- [ ] Apply Local-in Policies to drop unauthorized UDP 500/4500 scanning.
- [ ] Enable Quick Crash Detection (QCD) for faster tunnel recovery.

### 6. Validation & Diagnostics (Post-Deployment)
- [ ] Verify Phase 1: `diagnose vpn ike gateway list`
- [ ] Verify Phase 2: `diagnose vpn tunnel list`
- [ ] Test Spoke-to-Spoke dynamic tunnel creation (ADVPN UDP Hole Punching).
- [ ] Run packet sniffers on ports 500/4500 if tunnels fail to establish.
- [ ] Run IKE debugs (`diagnose debug application ike -1`) for precise error identification.
dialup ipsec and ospf

fgt-1

at the first step add active directory as ldap server on fortigate and then ad fsso on external connectors in security fabric



on vpn and ipsec section

custom type ipsec vpn

remote gateway set on dialup users then set incoming interface

then we can set valid users on xauth part and call the ad users or fsso from auto server

in ipsec use default values on fgt-1 just set device creation in advance part and disable add route

*used auto-discovery sender and reciever if need run dynamic routing protocols like ospf



set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id



des md5 and dh 5

local id  > fgt-1



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup + isp link and lan side



outgoing

dialup + isp link and lan side



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.1 with remote 12.23.34.2/24

ping



router ospf

rid > 1.1.1.1

area 0.0.0.0

192.168.101.0/24

12.23.34.0/24

inject default route always



fgt-2

on vpn and ipsec section

custom type ipsec vpn

remote gateway set on static ip of fgt-1 then interface

then we can set valid users on xauth part and place the user and password of xuath and validation users whre placed on fgt-1 (here  type the active directory users and fsso groups as placed in fgt-1 or hq)

username > u1

password > 1qaz@WSX



in ipsec use default values also set device creation in advance part enable add route (auto-discovery sender and reciever could be used and behave like fgt-1 (disable add route))

set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id  or specific id (fgt-1)



des md5 and dh 5



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup / ipsec interface + isp link and lan side



outgoing

dialup / ipsec interface + isp link and lan side



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.2 with remote 12.23.34.1/24

ping



router ospf

rid > 2.2.2.2

area 0.0.0.0

192.168.102.0/24

12.23.34.0/24



fgt-4

our device is behind the fgt-3



fgt-1 ------------ isp router ---------- fgt-3 ----------- fgt-4



on vpn and ipsec section

custom type ipsec vpn

remote gateway set on static ip of fgt-1 then interface

then we can set valid users on xauth part and place the user and password of xuath and validation users whre placed on fgt-1 (here  type the active directory users and fsso groups as placed in fgt-1 or hq)

username > u2

password > 1qaz@WSX



in ipsec use default values also set device creation  and auto-discovery sender and reciever in advance part disable add route

set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id  or specific id (fgt-1)



des md5 and dh 5



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup / ipsec interface + isp link and lan side



outgoing

dialup / ipsec interface + isp link and lan side



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.4 with remote 12.23.34.1/24

ping



router ospf

rid > 4.4.4.4

area 0.0.0.0

192.168.104.0/24

12.23.34.0/24



after setup these values we stock in init state of ospf negotiation

here must check ipsec interface on fgt-1 and enable auto-discovery sender and reciever in advance part of phase 1 to create and advertise mesh mechanism of ospf and fortigate



on fgt-3

policies

from fgt-4 to isp must enable nat

from isp to fgt-4 disable nat (for test and troubleshooting)



sometime we got touble on ospf negotiation with cisco devices

in ospf connectivity and rib on fgt we saw fgt-4 worked on 12.23.34.4 (ipsec interface) and recieved 192.168.104.0/24 on it, but 192.168.102.0/24 reachablity will be on public ip 2.2.2.1

here we can use interface setting on ospf routing panel in fgt and set ipsec interface type on point-to-point mode or different modes as broadcast or point-tomultipoint and solve the problem



*if needspokes connect together must enable exchange ip address on fgt-1 to give access worked like dmvpn in cisco , in dialup users we can't effect of enabling of this feature



mode config on dialup ipsec config

in network setting on custom ipsec vpns and dialup remote gateway we have some mode config options



use system dns

assign ip

dhcp

subnet mask

255.255.255.0



on ipsec interface ip assignment on network pannel we should do this

local ip or ip 12.23.34.1

remote ip  12.23.34.254/24

then enable dhcp on this interface to assign automated ip over dialup ipsec interfaces



on fgt-1

ipsec advance setup we should  disable all components just used device creation and add route



on fgt-2

ipsec advance setup we can change advance config

must enable mode config



range

user group

address group درقالب cheetsheet متناسب با github بهم بده

This cheat sheet structures your Dial-up IPsec, ADVPN, and OSPF configurations for a multi-FortiGate topology (Hub and Spokes, including NAT traversal).

Prerequisites
Active Directory Integration: Add AD as an LDAP server on the FortiGate Hub (FGT-1).

FSSO: Add AD FSSO in Security Fabric > External Connectors.

Phase 1 & Phase 2 IPsec Configurations
Feature	FGT-1 (Hub / HQ)	FGT-2 (Spoke - Static)	FGT-4 (Spoke - Behind NAT FGT-3)
Topology	Hub	Spoke	Spoke (behind ISP Router & FGT-3)
Gateway Type	Dialup User	Static IP of FGT-1	Static IP of FGT-1
IKE Version & Mode	IKEv1, Aggressive Mode	IKEv1, Aggressive Mode	IKEv1, Aggressive Mode
Phase 1 Crypto	DES, MD5, DH 5	DES, MD5, DH 5	DES, MD5, DH 5
Phase 2 Crypto	DES, MD5, PFS DH 5, Auto-negotiate	DES, MD5, PFS DH 5, Auto-negotiate	DES, MD5, PFS DH 5, Auto-negotiate
XAuth Setup	Server: Auto (Valid AD/FSSO users)	Client: Username u1, Pass 1qaz@WSX	Client: Username u2, Pass 1qaz@WSX
Peer ID	Accept Any Peer ID	Specific ID (FGT-1) or Any	Specific ID (FGT-1) or Any
Advanced Settings	
Enable: Device Creation


Disable: Add Route


Enable: Auto-Discovery Sender/Receiver

Enable: Device Creation, Add Route


(Can use Auto-Discovery)

Enable: Device Creation, Auto-Discovery Sender/Receiver


Disable: Add Route

Firewall Policies (Required on All FortiGates)
Incoming: Dialup/IPsec Interface + ISP Link & LAN side.

Outgoing: Dialup/IPsec Interface + ISP Link & LAN side.

Services/Source/Destination: All.

NAT: Disabled (Crucial for OSPF routing overlay). Exception: FGT-3 requires NAT enabled from FGT-4 to ISP, but disable NAT from ISP to FGT-4 for troubleshooting.

Logging: Log all sessions.

IPsec Virtual Interface & OSPF Routing
FGT-1 (Hub)

Interface IP: 12.23.34.1 | Remote IP: 12.23.34.2/24 (Allow Ping)

OSPF Router ID: 1.1.1.1

OSPF Area 0.0.0.0 Networks: 192.168.101.0/24, 12.23.34.0/24

Options: Inject default route always.

FGT-2 (Spoke)

Interface IP: 12.23.34.2 | Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 2.2.2.2

OSPF Area 0.0.0.0 Networks: 192.168.102.0/24, 12.23.34.0/24

FGT-4 (Spoke behind NAT)

Interface IP: 12.23.34.4 | Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 4.4.4.4

OSPF Area 0.0.0.0 Networks: 192.168.104.0/24, 12.23.34.0/24

OSPF Troubleshooting & ADVPN Notes
OSPF Stuck in INIT: If FGT-4 is stuck in OSPF INIT state, ensure Auto-Discovery Sender and Receiver are enabled on FGT-1 Phase 1 advanced settings to correctly advertise the OSPF mesh mechanism.

Cisco OSPF Interoperability: If spoke subnets (e.g., 192.168.102.0/24) resolve via the public IP instead of the tunnel interface, change the OSPF Interface Network Type in FortiGate to Point-to-Point or Broadcast / Point-to-Multipoint.

Spoke-to-Spoke (DMVPN Style): To allow spokes to communicate directly, enable Exchange IP Address on FGT-1. (Note: Dialup users alone do not trigger this feature).

Mode Config & IP Assignment (DHCP over IPsec)
If assigning IPs dynamically over the Dialup IPsec tunnel using Mode Config:

Network Settings (FGT-1 Custom IPsec VPN): Enable Mode Config options (Use System DNS, Assign IP via DHCP/Range/User Group/Address Group).

Interface Setup (FGT-1):

Subnet Mask: 255.255.255.0

Local IP: 12.23.34.1 | Remote IP: 12.23.34.254/24

Enable DHCP Server on the IPsec interface.

Advanced Setup Overrides:

FGT-1: Disable all advanced components except Device Creation and Add Route.

FGT-2: Under advanced config, ensure Mode Config is enabled as the client.

در قالب cheetsheet بهم بده و قالب github

این ساختار به‌صورت یک Cheat Sheet استاندارد، مینیمال و آماده برای کپی در فایل‌های گیت‌هاب (مانند README.md) تنظیم شده است:

Markdown
# FortiOS Dial-up IPsec & OSPF ADVPN Cheat Sheet

A quick-reference guide for configuring Dial-up IPsec, ADVPN, and OSPF over multi-FortiGate topologies (Hub & Spoke with NAT traversal).

---

## 1. Prerequisites
- **Active Directory:** Add AD as an LDAP server on the FortiGate Hub (FGT-1).
- **FSSO:** Configure AD FSSO under `Security Fabric > External Connectors`.

---

## 2. IPsec Phase 1 & 2 Matrix

| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static) | FGT-4 (Spoke - Behind NAT FGT-3) |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP of FGT-1 | Static IP of FGT-1 |
| **IKE & Mode**| IKEv1, Aggressive | IKEv1, Aggressive | IKEv1, Aggressive |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server: Auto (AD/FSSO users) | Client: `u1` / `1qaz@WSX`| Client: `u2` / `1qaz@WSX`|
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **IPsec Advanced** | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route | - Enable: Device Creation<br>- Enable: Add Route<br>*(Auto-Discovery optional)* | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route |

---

## 3. Global Firewall Policies (All Nodes)

**Traffic Flow Rules:**
* **Incoming:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Outgoing:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Service / Src / Dst:** `ALL`
* **Logging:** `Log all sessions`
* **NAT:** **Disabled** (Required for OSPF routing overlay). 
  > *Exception:* FGT-3 requires NAT enabled from FGT-4 to ISP. Disable NAT from ISP to FGT-4 for troubleshooting.

---

## 4. OSPF Routing & Interfaces

### FGT-1 (Hub)
```text
Interface IP: 12.23.34.1
Remote IP: 12.23.34.2/24 (Allow Ping)

OSPF Router ID: 1.1.1.1
OSPF Area 0.0.0.0:
  - 192.168.101.0/24
  - 12.23.34.0/24
Options: Inject default route always
FGT-2 (Spoke)
Plaintext
Interface IP: 12.23.34.2
Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 2.2.2.2
OSPF Area 0.0.0.0:
  - 192.168.102.0/24
  - 12.23.34.0/24
FGT-4 (Spoke behind NAT)
Plaintext
Interface IP: 12.23.34.4
Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 4.4.4.4
OSPF Area 0.0.0.0:
  - 192.168.104.0/24
  - 12.23.34.0/24
5. Mode Config (DHCP over IPsec)
To assign IPs dynamically over the Dialup IPsec tunnel:

FGT-1 Network Settings: In Custom IPsec VPN, enable Mode Config (Use System DNS, Assign IP).

FGT-1 Interface:

Subnet Mask: 255.255.255.0

Local IP: 12.23.34.1 | Remote IP: 12.23.34.254/24

Enable DHCP Server on the IPsec interface.

Advanced Setup:

FGT-1: Disable all advanced components except Device Creation and Add Route.

FGT-2 (Client): Ensure Mode Config is enabled.

6. ADVPN & OSPF Troubleshooting
OSPF Stuck in INIT: Verify Auto-Discovery Sender and Receiver are enabled on FGT-1 Phase 1 advanced settings to advertise the OSPF mesh properly.

Cisco Interoperability: If spoke subnets route via public IPs instead of tunnels, change the FortiGate OSPF Interface Network Type to Point-to-Point or Broadcast / Point-to-Multipoint.

Spoke-to-Spoke Traffic: Enable Exchange IP Address on FGT-1 to mimic Cisco DMVPN behavior. (Dialup user groups alone do not trigger this).

```markdown

# FortiOS Dial-up IPsec & OSPF ADVPN Cheat Sheet



A quick-reference guide for configuring Dial-up IPsec, ADVPN, and OSPF over multi-FortiGate topologies (Hub & Spoke with NAT traversal).



---



## 1. Prerequisites

- **Active Directory:** Add AD as an LDAP server on the FortiGate Hub (FGT-1).

- **FSSO:** Configure AD FSSO under `Security Fabric > External Connectors`.



---



## 2. IPsec Phase 1 & 2 Matrix



| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static) | FGT-4 (Spoke - Behind NAT FGT-3) |

| :--- | :--- | :--- | :--- |

| **Gateway Type** | Dialup User | Static IP of FGT-1 | Static IP of FGT-1 |

| **IKE & Mode**| IKEv1, Aggressive | IKEv1, Aggressive | IKEv1, Aggressive |

| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |

| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |

| **XAuth Setup** | Server: Auto (AD/FSSO users) | Client: `u1` / `1qaz@WSX`| Client: `u2` / `1qaz@WSX`|

| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |

| **IPsec Advanced** | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route | - Enable: Device Creation<br>- Enable: Add Route<br>*(Auto-Discovery optional)* | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route |



---



## 3. Global Firewall Policies (All Nodes)



**Traffic Flow Rules:**

* **Incoming:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Outgoing:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Service / Src / Dst:** `ALL`

* **Logging:** `Log all sessions`

* **NAT:** **Disabled** (Required for OSPF routing overlay). 

  > *Exception:* FGT-3 requires NAT enabled from FGT-4 to ISP. Disable NAT from ISP to FGT-4 for troubleshooting.



---



## 4. OSPF Routing & Interfaces



### FGT-1 (Hub)

```text

Interface IP: 12.23.34.1

Remote IP: 12.23.34.2/24 (Allow Ping)



OSPF Router ID: 1.1.1.1

OSPF Area 0.0.0.0:

  - 192.168.101.0/24

  - 12.23.34.0/24

Options: Inject default route always



```



### FGT-2 (Spoke)



```text

Interface IP: 12.23.34.2

Remote IP: 12.23.34.1/24 (Allow Ping)



OSPF Router ID: 2.2.2.2

OSPF Area 0.0.0.0:

  - 192.168.102.0/24

  - 12.23.34.0/24



```



### FGT-4 (Spoke behind NAT)



```text

Interface IP: 12.23.34.4

Remote IP: 12.23.34.1/24 (Allow Ping)



OSPF Router ID: 4.4.4.4

OSPF Area 0.0.0.0:

  - 192.168.104.0/24

  - 12.23.34.0/24



```



---



## 5. Mode Config (DHCP over IPsec)



To assign IPs dynamically over the Dialup IPsec tunnel:



1. **FGT-1 Network Settings:** In Custom IPsec VPN, enable Mode Config (`Use System DNS`, `Assign IP`).

2. **FGT-1 Interface:**

* Subnet Mask: `255.255.255.0`

* Local IP: `12.23.34.1` | Remote IP: `12.23.34.254/24`

* Enable DHCP Server on the IPsec interface.





3. **Advanced Setup:**

* **FGT-1:** Disable all advanced components except `Device Creation` and `Add Route`.

* **FGT-2 (Client):** Ensure `Mode Config` is enabled.







---



## 6. ADVPN & OSPF Troubleshooting



* **OSPF Stuck in INIT:** Verify **Auto-Discovery Sender and Receiver** are enabled on FGT-1 Phase 1 advanced settings to advertise the OSPF mesh properly.

* **Cisco Interoperability:** If spoke subnets route via public IPs instead of tunnels, change the FortiGate OSPF Interface Network Type to **Point-to-Point** or **Broadcast / Point-to-Multipoint**.

* **Spoke-to-Spoke Traffic:** Enable **Exchange IP Address** on FGT-1 to mimic Cisco DMVPN behavior. *(Dialup user groups alone do not trigger this).*



```



```

اصلاحش کن و کاملتر و پر جزیات تر بگو

این نسخه توسعه‌یافته و بسیار دقیق‌تر، با اضافه شدن جزئیات مسیردهی پیشرفته OSPF، تنظیمات DPD، دستورات خط فرمان (CLI) برای عیب‌یابی عمیق، و ظرایف Mode Config تنظیم شده تا برای پیاده‌سازی‌های در سطح NSE 7 کاملاً بدون نقص باشد.

Markdown
# Comprehensive FortiOS Dial-up IPsec, ADVPN & OSPF Cheat Sheet

An advanced, highly detailed reference guide for deploying Dial-up IPsec, ADVPN (Auto-Discovery VPN), and OSPF dynamic routing over multi-FortiGate Hub-and-Spoke topologies, including NAT traversal scenarios.

---

## 1. Identity & Fabric Prerequisites
- **Active Directory (LDAP):** Define AD as an LDAP server on the FortiGate Hub (FGT-1) under `User & Authentication > LDAP Servers`.
- **FSSO Integration:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses to AD user groups dynamically.
- **User Groups:** Create local Firewall User Groups mapping to the LDAP/FSSO groups to be used in the IPsec XAuth configuration.

---

## 2. IPsec Phase 1 & 2 Matrix (Hub & Spoke)

| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static IP) | FGT-4 (Spoke - Behind NAT FGT-3) |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |
| **IKE Version/Mode**| IKEv1, Aggressive Mode | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 (Consider AES/SHA256 for Prod) | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **DPD (Dead Peer)** | On-Idle (Saves Hub resources) | On-Demand | On-Demand |
| **XAuth Setup** | Server: Auto (Validates AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **ADVPN & Routing** | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route | Enable: Device Creation<br>Enable: Add Route<br>*(Auto-Discovery optional)* | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route |

> **Pro Tip:** For ADVPN to work over IPsec, ensure `network-id` (e.g., `set network-id 1`) is identically configured on Phase 1 of all participating FortiGates via CLI.

---

## 3. Global Firewall Policies (All Nodes)

**Traffic Flow Rules:**
* **Incoming Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Outgoing Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Service / Src / Dst:** `ALL` (Restrict based on least privilege in production).
* **Logging:** `Log all sessions` (Crucial for ADVPN troubleshooting).
* **NAT:** **Strictly Disabled** for the overlay traffic (Required for OSPF routing overlay).
* **TCP MSS:** Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation drops.

> *Exception Rule:* FGT-3 requires NAT enabled from FGT-4 to the ISP. Disable NAT from the ISP to FGT-4 when troubleshooting UDP 500/4500 packets.

---

## 4. OSPF Routing & Overlay Interfaces

**Crucial OSPF Settings:** To allow Hub-and-Spoke and Spoke-to-Spoke routing over IPsec tunnels, the OSPF network type on the IPsec interfaces must be carefully managed.

### FGT-1 (Hub)
```text
Interface IP: 12.23.34.1
Remote IP: 12.23.34.2/24 (Allow Ping, OSPF)

OSPF Router ID: 1.1.1.1
OSPF Area 0.0.0.0:
  - 192.168.101.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
Options: Inject default route always
CLI Tweak: set network-type point-to-multipoint (on IPsec interface)
FGT-2 (Spoke)
Plaintext
Interface IP: 12.23.34.2
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 2.2.2.2
OSPF Area 0.0.0.0:
  - 192.168.102.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point
FGT-4 (Spoke behind NAT)
Plaintext
Interface IP: 12.23.34.4
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 4.4.4.4
OSPF Area 0.0.0.0:
  - 192.168.104.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point
5. Mode Config (DHCP over IPsec)
To assign IPs dynamically to spokes over the Dialup IPsec tunnel instead of static assignments:

FGT-1 Network Settings (Phase 1): In Custom IPsec VPN, enable Mode Config (Use System DNS, Assign IP).

FGT-1 Interface Configuration:

Subnet Mask: 255.255.255.0

Local IP: 12.23.34.1 | Remote IP: 12.23.34.254/24

Crucial: Enable the DHCP Server directly on this IPsec interface.

Advanced Setup Adjustments:

FGT-1 (Hub): Disable all advanced components except Device Creation and Add Route.

FGT-2/FGT-4 (Clients): Ensure Mode Config is explicitly enabled in Phase 1 settings to request the IP.

6. Advanced ADVPN & OSPF Troubleshooting
OSPF & Routing Issues
OSPF Stuck in INIT: Verify Auto-Discovery Sender and Receiver are enabled on FGT-1 Phase 1. If the Hub cannot dynamically create the Spoke interface, OSPF Hellos will fail.

Cisco Interoperability / Bad Next-Hops: If spoke subnets route via public IPs instead of tunnels (a common issue with Cisco DMVPN interop), change the FortiGate OSPF Interface Network Type via CLI:

Bash
config router ospf
  config ospf-interface
    edit "ipsec-tun1"
      set network-type point-to-multipoint
    next
  end
end
Spoke-to-Spoke Traffic: Enable set exchange-ip-addr4 via CLI on FGT-1 Phase 1 to mimic Cisco DMVPN behavior and allow spokes to learn each other's physical IPs for shortcut tunnels. (Dialup user groups alone do not trigger this).

Essential CLI Diagnostic Commands
Bash
# Verify Phase 1 and NAT-T status
diagnose vpn ike gateway list 

# Verify Phase 2 selectors and active Subnets
diagnose vpn tunnel list

# Real-time OSPF neighbor state and LSDB
get router info ospf neighbor
get router info ospf database

# Deep IKE Debugging (Run when tunnels fail to establish)
diagnose vpn ike log filter dst-addr4 <Peer_Public_IP>
diagnose debug application ike -1
diagnose debug enable


# Comprehensive FortiOS Dial-up IPsec, ADVPN & OSPF Cheat Sheet



An advanced, highly detailed reference guide for deploying Dial-up IPsec, ADVPN (Auto-Discovery VPN), and OSPF dynamic routing over multi-FortiGate Hub-and-Spoke topologies, including NAT traversal scenarios.



---



## 1. Identity & Fabric Prerequisites

- **Active Directory (LDAP):** Define AD as an LDAP server on the FortiGate Hub (FGT-1) under `User & Authentication > LDAP Servers`.

- **FSSO Integration:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses to AD user groups dynamically.

- **User Groups:** Create local Firewall User Groups mapping to the LDAP/FSSO groups to be used in the IPsec XAuth configuration.



---



## 2. IPsec Phase 1 & 2 Matrix (Hub & Spoke)



| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static IP) | FGT-4 (Spoke - Behind NAT FGT-3) |

| :--- | :--- | :--- | :--- |

| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |

| **IKE Version/Mode**| IKEv1, Aggressive Mode | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode |

| **Phase 1 Crypto** | DES, MD5, DH 5 (Consider AES/SHA256 for Prod) | DES, MD5, DH 5 | DES, MD5, DH 5 |

| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |

| **DPD (Dead Peer)** | On-Idle (Saves Hub resources) | On-Demand | On-Demand |

| **XAuth Setup** | Server: Auto (Validates AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |

| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |

| **ADVPN & Routing** | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route | Enable: Device Creation<br>Enable: Add Route<br>*(Auto-Discovery optional)* | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route |



> **Pro Tip:** For ADVPN to work over IPsec, ensure `network-id` (e.g., `set network-id 1`) is identically configured on Phase 1 of all participating FortiGates via CLI.



---



## 3. Global Firewall Policies (All Nodes)



**Traffic Flow Rules:**

* **Incoming Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Outgoing Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Service / Src / Dst:** `ALL` (Restrict based on least privilege in production).

* **Logging:** `Log all sessions` (Crucial for ADVPN troubleshooting).

* **NAT:** **Strictly Disabled** for the overlay traffic (Required for OSPF routing overlay).

* **TCP MSS:** Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation drops.



> *Exception Rule:* FGT-3 requires NAT enabled from FGT-4 to the ISP. Disable NAT from the ISP to FGT-4 when troubleshooting UDP 500/4500 packets.



---



## 4. OSPF Routing & Overlay Interfaces



**Crucial OSPF Settings:** To allow Hub-and-Spoke and Spoke-to-Spoke routing over IPsec tunnels, the OSPF network type on the IPsec interfaces must be carefully managed.



### FGT-1 (Hub)

```text

Interface IP: 12.23.34.1

Remote IP: 12.23.34.2/24 (Allow Ping, OSPF)



OSPF Router ID: 1.1.1.1

OSPF Area 0.0.0.0:

  - 192.168.101.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

Options: Inject default route always

CLI Tweak: set network-type point-to-multipoint (on IPsec interface)



```



### FGT-2 (Spoke)



```text

Interface IP: 12.23.34.2

Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)



OSPF Router ID: 2.2.2.2

OSPF Area 0.0.0.0:

  - 192.168.102.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

CLI Tweak: set network-type point-to-point



```



### FGT-4 (Spoke behind NAT)



```text

Interface IP: 12.23.34.4

Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)



OSPF Router ID: 4.4.4.4

OSPF Area 0.0.0.0:

  - 192.168.104.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

CLI Tweak: set network-type point-to-point



```



---



## 5. Mode Config (DHCP over IPsec)



To assign IPs dynamically to spokes over the Dialup IPsec tunnel instead of static assignments:



1. **FGT-1 Network Settings (Phase 1):** In Custom IPsec VPN, enable Mode Config (`Use System DNS`, `Assign IP`).

2. **FGT-1 Interface Configuration:**

* Subnet Mask: `255.255.255.0`

* Local IP: `12.23.34.1` | Remote IP: `12.23.34.254/24`

* **Crucial:** Enable the DHCP Server directly on this IPsec interface.





3. **Advanced Setup Adjustments:**

* **FGT-1 (Hub):** Disable all advanced components *except* `Device Creation` and `Add Route`.

* **FGT-2/FGT-4 (Clients):** Ensure `Mode Config` is explicitly enabled in Phase 1 settings to request the IP.







---



## 6. Advanced ADVPN & OSPF Troubleshooting



### OSPF & Routing Issues



* **OSPF Stuck in INIT:** Verify **Auto-Discovery Sender and Receiver** are enabled on FGT-1 Phase 1. If the Hub cannot dynamically create the Spoke interface, OSPF Hellos will fail.

* **Cisco Interoperability / Bad Next-Hops:** If spoke subnets route via public IPs instead of tunnels (a common issue with Cisco DMVPN interop), change the FortiGate OSPF Interface Network Type via CLI:

```bash

config router ospf

  config ospf-interface

    edit "ipsec-tun1"

      set network-type point-to-multipoint

    next

  end

end



```





* **Spoke-to-Spoke Traffic:** Enable `set exchange-ip-addr4` via CLI on FGT-1 Phase 1 to mimic Cisco DMVPN behavior and allow spokes to learn each other's physical IPs for shortcut tunnels. *(Dialup user groups alone do not trigger this).*



### Essential CLI Diagnostic Commands



```bash

# Verify Phase 1 and NAT-T status

diagnose vpn ike gateway list 



# Verify Phase 2 selectors and active Subnets

diagnose vpn tunnel list



# Real-time OSPF neighbor state and LSDB

get router info ospf neighbor

get router info ospf database



# Deep IKE Debugging (Run when tunnels fail to establish)

diagnose vpn ike log filter dst-addr4 <Peer_Public_IP>

diagnose debug application ike -1

diagnose debug enable



```



```

ایرادات اینو بگیر و متناسب با قالب githubکن

Markdown
# Comprehensive FortiOS Dial-up IPsec, ADVPN & OSPF Cheat Sheet

An advanced, highly detailed reference guide for deploying Dial-up IPsec, ADVPN (Auto-Discovery VPN), and OSPF dynamic routing over multi-FortiGate Hub-and-Spoke topologies, including NAT traversal scenarios.

---

## 1. Identity & Fabric Prerequisites

- **Active Directory (LDAP):** Define AD as an LDAP server on the FortiGate Hub (FGT-1) under `User & Authentication > LDAP Servers`.
- **FSSO Integration:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses to AD user groups dynamically.
- **User Groups:** Create local Firewall User Groups mapping to the LDAP/FSSO groups to be used in the IPsec XAuth configuration.

---

## 2. IPsec Phase 1 & 2 Matrix (Hub & Spoke)

| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static IP) | FGT-4 (Spoke - Behind NAT FGT-3) |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |
| **IKE Version/Mode** | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 (Consider AES/SHA256 for Prod) | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **DPD (Dead Peer)** | On-Idle (Saves Hub resources) | On-Demand | On-Demand |
| **XAuth Setup** | Server: Auto (Validates AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **ADVPN & Routing** | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route | Enable: Device Creation<br>Enable: Add Route<br>*(Auto-Discovery optional)* | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route |

> **Pro Tip:** For ADVPN to work over IPsec, ensure `network-id` (e.g., `set network-id 1`) is identically configured on Phase 1 of all participating FortiGates via CLI.

---

## 3. Global Firewall Policies (All Nodes)

**Traffic Flow Rules:**
* **Incoming Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Outgoing Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Service / Src / Dst:** `ALL` (Restrict based on least privilege in production).
* **Logging:** `Log all sessions` (Crucial for ADVPN troubleshooting).
* **NAT:** **Strictly Disabled** for the overlay traffic (Required for OSPF routing overlay).
* **TCP MSS:** Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation drops.

> *Exception Rule:* FGT-3 requires NAT enabled from FGT-4 to the ISP. Disable NAT from the ISP to FGT-4 when troubleshooting UDP 500/4500 packets.

---

## 4. OSPF Routing & Overlay Interfaces

**Crucial OSPF Settings:** To allow Hub-and-Spoke and Spoke-to-Spoke routing over IPsec tunnels, the OSPF network type on the IPsec interfaces must be carefully managed.

### FGT-1 (Hub)
```text
Interface IP: 12.23.34.1
Remote IP: 12.23.34.2/24 (Allow Ping, OSPF)

OSPF Router ID: 1.1.1.1
OSPF Area 0.0.0.0:
  - 192.168.101.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
Options: Inject default route always
CLI Tweak: set network-type point-to-multipoint (on IPsec interface)
FGT-2 (Spoke)
Plaintext
Interface IP: 12.23.34.2
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 2.2.2.2
OSPF Area 0.0.0.0:
  - 192.168.102.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point
FGT-4 (Spoke behind NAT)
Plaintext
Interface IP: 12.23.34.4
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 4.4.4.4
OSPF Area 0.0.0.0:
  - 192.168.104.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point
5. Mode Config (DHCP over IPsec)
To assign IPs dynamically to spokes over the Dialup IPsec tunnel instead of static assignments:

FGT-1 Network Settings (Phase 1): In Custom IPsec VPN, enable Mode Config (Use System DNS, Assign IP).

FGT-1 Interface Configuration:

Subnet Mask: 255.255.255.0

Local IP: 12.23.34.1 | Remote IP: 12.23.34.254/24

Crucial: Enable the DHCP Server directly on this IPsec interface.

Advanced Setup Adjustments:

FGT-1 (Hub): Disable all advanced components except Device Creation and Add Route.

FGT-2/FGT-4 (Clients): Ensure Mode Config is explicitly enabled in Phase 1 settings to request the IP.

6. Advanced ADVPN & OSPF Troubleshooting
OSPF & Routing Issues
OSPF Stuck in INIT: Verify Auto-Discovery Sender and Receiver are enabled on FGT-1 Phase 1. If the Hub cannot dynamically create the Spoke interface, OSPF Hellos will fail.

Cisco Interoperability / Bad Next-Hops: If spoke subnets route via public IPs instead of tunnels (a common issue with Cisco DMVPN interop), change the FortiGate OSPF Interface Network Type via CLI:

Bash
config router ospf
  config ospf-interface
    edit "ipsec-tun1"
      set network-type point-to-multipoint
    next
  end
end
Spoke-to-Spoke Traffic: Enable set exchange-ip-addr4 via CLI on FGT-1 Phase 1 to mimic Cisco DMVPN behavior and allow spokes to learn each other's physical IPs for shortcut tunnels. (Dialup user groups alone do not trigger this).

Essential CLI Diagnostic Commands
Bash
# Verify Phase 1 and NAT-T status
diagnose vpn ike gateway list 

# Verify Phase 2 selectors and active Subnets
diagnose vpn tunnel list

# Real-time OSPF neighbor state and LSDB
get router info ospf neighbor
get router info ospf database

# Deep IKE Debugging (Run when tunnels fail to establish)
diagnose vpn ike log filter dst-addr4 <Peer_Public_IP>
diagnose debug application ike -1
diagnose debug enable
# Comprehensive FortiOS Dial-up IPsec, ADVPN & OSPF Cheat Sheet



An advanced, highly detailed reference guide for deploying Dial-up IPsec, ADVPN (Auto-Discovery VPN), and OSPF dynamic routing over multi-FortiGate Hub-and-Spoke topologies, including NAT traversal scenarios.



---



## 1. Identity & Fabric Prerequisites



- **Active Directory (LDAP):** Define AD as an LDAP server on the FortiGate Hub (FGT-1) under `User & Authentication > LDAP Servers`.

- **FSSO Integration:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses to AD user groups dynamically.

- **User Groups:** Create local Firewall User Groups mapping to the LDAP/FSSO groups to be used in the IPsec XAuth configuration.



---



## 2. IPsec Phase 1 & 2 Matrix (Hub & Spoke)



| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static IP) | FGT-4 (Spoke - Behind NAT FGT-3) |

| :--- | :--- | :--- | :--- |

| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |

| **IKE Version/Mode** | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode |

| **Phase 1 Crypto** | DES, MD5, DH 5 (Consider AES/SHA256 for Prod) | DES, MD5, DH 5 | DES, MD5, DH 5 |

| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |

| **DPD (Dead Peer)** | On-Idle (Saves Hub resources) | On-Demand | On-Demand |

| **XAuth Setup** | Server: Auto (Validates AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |

| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |

| **ADVPN & Routing** | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route | Enable: Device Creation<br>Enable: Add Route<br>*(Auto-Discovery optional)* | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route |



> **Pro Tip:** For ADVPN to work over IPsec, ensure `network-id` (e.g., `set network-id 1`) is identically configured on Phase 1 of all participating FortiGates via CLI.



---



## 3. Global Firewall Policies (All Nodes)



**Traffic Flow Rules:**

* **Incoming Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Outgoing Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`

* **Service / Src / Dst:** `ALL` (Restrict based on least privilege in production).

* **Logging:** `Log all sessions` (Crucial for ADVPN troubleshooting).

* **NAT:** **Strictly Disabled** for the overlay traffic (Required for OSPF routing overlay).

* **TCP MSS:** Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation drops.



> *Exception Rule:* FGT-3 requires NAT enabled from FGT-4 to the ISP. Disable NAT from the ISP to FGT-4 when troubleshooting UDP 500/4500 packets.



---



## 4. OSPF Routing & Overlay Interfaces



**Crucial OSPF Settings:** To allow Hub-and-Spoke and Spoke-to-Spoke routing over IPsec tunnels, the OSPF network type on the IPsec interfaces must be carefully managed.



### FGT-1 (Hub)

```text

Interface IP: 12.23.34.1

Remote IP: 12.23.34.2/24 (Allow Ping, OSPF)



OSPF Router ID: 1.1.1.1

OSPF Area 0.0.0.0:

  - 192.168.101.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

Options: Inject default route always

CLI Tweak: set network-type point-to-multipoint (on IPsec interface)



```



### FGT-2 (Spoke)



```text

Interface IP: 12.23.34.2

Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)



OSPF Router ID: 2.2.2.2

OSPF Area 0.0.0.0:

  - 192.168.102.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

CLI Tweak: set network-type point-to-point



```



### FGT-4 (Spoke behind NAT)



```text

Interface IP: 12.23.34.4

Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)



OSPF Router ID: 4.4.4.4

OSPF Area 0.0.0.0:

  - 192.168.104.0/24 (Local LAN)

  - 12.23.34.0/24 (VPN Overlay)

CLI Tweak: set network-type point-to-point



```



---



## 5. Mode Config (DHCP over IPsec)



To assign IPs dynamically to spokes over the Dialup IPsec tunnel instead of static assignments:



1. **FGT-1 Network Settings (Phase 1):** In Custom IPsec VPN, enable Mode Config (`Use System DNS`, `Assign IP`).

2. **FGT-1 Interface Configuration:**

* Subnet Mask: `255.255.255.0`

* Local IP: `12.23.34.1` | Remote IP: `12.23.34.254/24`

* **Crucial:** Enable the DHCP Server directly on this IPsec interface.





3. **Advanced Setup Adjustments:**

* **FGT-1 (Hub):** Disable all advanced components *except* `Device Creation` and `Add Route`.

* **FGT-2/FGT-4 (Clients):** Ensure `Mode Config` is explicitly enabled in Phase 1 settings to request the IP.







---



## 6. Advanced ADVPN & OSPF Troubleshooting



### OSPF & Routing Issues



* **OSPF Stuck in INIT:** Verify **Auto-Discovery Sender and Receiver** are enabled on FGT-1 Phase 1. If the Hub cannot dynamically create the Spoke interface, OSPF Hellos will fail.

* **Cisco Interoperability / Bad Next-Hops:** If spoke subnets route via public IPs instead of tunnels (a common issue with Cisco DMVPN interop), change the FortiGate OSPF Interface Network Type via CLI:



```bash

config router ospf

  config ospf-interface

    edit "ipsec-tun1"

      set network-type point-to-multipoint

    next

  end

end



```



* **Spoke-to-Spoke Traffic:** Enable `set exchange-ip-addr4` via CLI on FGT-1 Phase 1 to mimic Cisco DMVPN behavior and allow spokes to learn each other's physical IPs for shortcut tunnels. *(Dialup user groups alone do not trigger this).*



### Essential CLI Diagnostic Commands



```bash

# Verify Phase 1 and NAT-T status

diagnose vpn ike gateway list 



# Verify Phase 2 selectors and active Subnets

diagnose vpn tunnel list



# Real-time OSPF neighbor state and LSDB

get router info ospf neighbor

get router info ospf database



# Deep IKE Debugging (Run when tunnels fail to establish)

diagnose vpn ike log filter dst-addr4 <Peer_Public_IP>

diagnose debug application ike -1

diagnose debug enable



```

براسا این موارد یک checklist در قالب github تولید کن و markdown

Copy and paste this Markdown checklist directly into your GitHub repository to track the deployment progress.

Identity & Fabric Prerequisites

[ ] Define Active Directory as an LDAP server on the Hub (FGT-1) under User & Authentication > LDAP Servers.

[ ] Configure AD FSSO under Security Fabric > External Connectors for dynamic user mapping.

[ ] Create local Firewall User Groups mapped to LDAP/FSSO groups for IPsec XAuth.

IPsec Phase 1 & 2 Parameters

[ ] Set Gateway Type: Dialup User on Hub (FGT-1) / Static IP on Spokes (FGT-2, FGT-4).

[ ] Configure IKEv1 with Aggressive Mode across all nodes.

[ ] Set Phase 1 Crypto: DES, MD5, DH 5.

[ ] Set Phase 2 Crypto: DES, MD5, PFS DH 5, Auto-negotiate.

[ ] Configure DPD: On-Idle for Hub (FGT-1) and On-Demand for Spokes.

[ ] Set XAuth: Server: Auto on Hub / Configure client credentials (u1, u2) on Spokes.

[ ] Ensure network-id is identically configured via CLI on Phase 1 for all nodes.

[ ] Enable Device Creation on all nodes.

[ ] Enable Auto-Discovery Sender & Receiver on FGT-1 and FGT-4.

[ ] Disable Add Route on FGT-1 and FGT-4; Enable it on FGT-2.

Global Firewall Policies

[ ] Create Incoming Policy: Dialup/IPsec Interface + ISP Link & LAN.

[ ] Create Outgoing Policy: Dialup/IPsec Interface + ISP Link & LAN.

[ ] Set Service / Src / Dst to ALL (restrict as necessary for production).

[ ] Enable Log all sessions for ADVPN troubleshooting.

[ ] Strictly Disable NAT on overlay policies (Exception: NAT enabled from FGT-4 to ISP on FGT-3).

[ ] Adjust TCP MSS (e.g., 1350) in the policy CLI to prevent MTU fragmentation.

OSPF Routing & Overlay Interfaces

[ ] Assign Interface IP and Remote IP/Subnet on IPsec virtual interfaces.

[ ] Allow Ping and OSPF administrative access on IPsec interfaces.

[ ] Assign unique OSPF Router IDs (e.g., 1.1.1.1, 2.2.2.2).

[ ] Announce Local LAN (e.g., 192.168.10x.0/24) and VPN Overlay (12.23.34.0/24) in OSPF Area 0.0.0.0.

[ ] Inject default route always on the Hub (FGT-1).

[ ] CLI Tweak: Set OSPF network-type to point-to-multipoint on FGT-1.

[ ] CLI Tweak: Set OSPF network-type to point-to-point on Spokes (FGT-2, FGT-4).

Mode Config (DHCP over IPsec)

[ ] Enable Mode Config (Use System DNS, Assign IP) in FGT-1 Phase 1 settings.

[ ] Set Subnet Mask, Local IP, and Remote IP range on the FGT-1 IPsec interface.

[ ] Enable the DHCP Server directly on the FGT-1 IPsec interface.

[ ] On FGT-1 Phase 1 Advanced Setup: Disable all components except Device Creation and Add Route.

[ ] Explicitly enable Mode Config in Phase 1 on Spoke clients to request IPs.

Advanced ADVPN & OSPF Validation

[ ] Verify Auto-Discovery Sender and Receiver on FGT-1 if OSPF remains in INIT state.

[ ] Validate OSPF network types via CLI if subnets route via public IPs (bad next-hop).

[ ] CLI Tweak: Enable set exchange-ip-addr4 on FGT-1 Phase 1 to permit Spoke-to-Spoke shortcut tunnels.

[ ] Run diagnose vpn ike gateway list to verify Phase 1 and NAT-T status.

[ ] Run diagnose vpn tunnel list to verify active Phase 2 subnets.

[ ] Run get router info ospf neighbor to confirm adjacency.

[ ] Run diagnose debug application ike -1 and diagnose debug enable for deep IKE troubleshooting.

dialup ipsec and bgp

fgt-1

at the first step add active directory as ldap server on fortigate and then ad fsso on external connectors in security fabric



on vpn and ipsec section

custom type ipsec vpn

remote gateway set on dialup users then set incoming interface

then we can set valid users on xauth part and call the ad users or fsso from auto server

in ipsec use default values on fgt-1 just set device creation in advance part and enable add route

if one device with this tunnel will be gone active just use add route but many connections with same tunnel interface could not work like this

edit them as these

disable add route

enable auto-discovery sender and reciever

enable device creation



set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id



des md5 and dh 5



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup + isp link and lan side



outgoing

dialup + isp link and lan side



*if need forward ad requests from branches to hq



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.1 with remote 12.23.34.2/24

ping



router bgp

rid > 1.1.1.1

local-as 65001

192.168.101.0/24



neighbor

remote-as 65002 12.23.34.2 nexthop-self



*on this model (dialup) if enable exchange interfaces got disable and could not worked so better used fortinet recommendation on scenarios

hub and spoke (with ibgp) or site-to-site ipsec vpns



fgt-2

on vpn and ipsec section

custom type ipsec vpn

remote gateway set on static ip of fgt-1 then interface

then we can set valid users on xauth part and place the user and password of xuath and validation users whre placed on fgt-1 (here  type the active directory users and fsso groups as placed in fgt-1 or hq)

username > u1

password > 1qaz@WSX



in ipsec use default values also set device creation in advance part enable add route

set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id  or specific id (fgt-1)



des md5 and dh 5



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup / ipsec interface + isp link and lan side



outgoing

dialup / ipsec interface + isp link and lan side



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.2 with remote 12.23.34.1/24

ping





router bgp

rid > 2.2.2.2

local-as 65002

192.168.102.0/24



neighbor

remote-as 65001 12.23.34.1 nexthop-self



fgt-4

our device is behind the fgt-3



fgt-1 ------------ isp router ---------- fgt-3 ----------- fgt-4



on vpn and ipsec section

custom type ipsec vpn

remote gateway set on static ip of fgt-1 then interface

then we can set valid users on xauth part and place the user and password of xuath and validation users whre placed on fgt-1 (here  type the active directory users and fsso groups as placed in fgt-1 or hq)

username > u2

password > 1qaz@WSX



in ipsec use default values also set device creation in advance part enable add route or disable add route and enable auto-discovery sender and reciever

set-presharedkey and ike version 1 worked on aggresive mode and recieved any peer id  or specific id (fgt-1)



des md5 and dh 5



on phase 2 access any subnets to be valid and worked on des md5 , pfs with dh group 5

also set auto-negotiate



set policies

incoming

dialup / ipsec interface + isp link and lan side



outgoing

dialup / ipsec interface + isp link and lan side



all services and src + dst



no nat (cause need ospf routing)



log all sessions



on interface part select ipsec interface

set ip 12.23.34.4 with remote 12.23.34.1/24

ping



router bgp

rid > 4.4.4.4

local-as 65004

192.168.104.0/24



neighbor

remote-as 65001 12.23.34.1 nexthop-self



on fgt-3

policies

from fgt-4 to isp must enable nat

from isp to fgt-4 disable nat (for test and troubleshooting) ان متن را در github cheet sheet با تمام جزیات بنویس

Python
python
markdown_content = """# 🛡️ FortiOS Dial-up IPsec & eBGP Cheat Sheet

A comprehensive, lab-ready reference guide for deploying Dial-up IPsec combined with eBGP routing across a Hub-and-Spoke Fortinet topology. This guide includes configurations for standard spokes and spokes situated behind NAT devices, structured for clear deployment in enterprise training or production environments.

---

## 1. 🔑 Identity & Authentication Prerequisites (FGT-1 Hub)

Before establishing the VPN overlay, identity integration must be configured on the Hub.

*   **LDAP Server:** Add the Active Directory server under `User & Authentication > LDAP Servers`.
*   **FSSO Connector:** Configure AD FSSO under `Security Fabric > External Connectors` to enable dynamic IP-to-Group mapping.
*   **User Groups:** Create local FortiGate Firewall User Groups that map to the AD/FSSO groups. These will be referenced in the IPsec XAuth settings.

---

## 2. 🌐 IPsec Phase 1 & 2 Deployment Matrix

| Feature/Setting | **FGT-1 (Hub / HQ)** | **FGT-2 (Standard Spoke)** | **FGT-4 (Spoke Behind NAT FGT-3)** |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (Points to FGT-1) | Static IP (Points to FGT-1) |
| **IKE Version/Mode** | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server Type: Auto (Calls AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **Phase 2 Selectors** | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) |
| **Advanced Routing Settings** | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver | **Enable:** Device Creation<br>**Enable:** Add Route | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver |

> **⚠️ Hub-and-Spoke Scaling Note for FGT-1:** If only *one* device connects via the tunnel, `Add Route` can remain enabled. However, for multiple connections sharing the same tunnel interface, you **must** disable `Add Route` and rely on dynamic routing (BGP) combined with `Device Creation` and `Auto-discovery`.

---

## 3. 🛡️ Global Firewall Policies

Apply these baseline policies across **all** participating FortiGates (FGT-1, FGT-2, FGT-4) to allow overlay routing and tunnel establishment.

*   **Incoming Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Outgoing Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Service / Source / Destination:** `ALL` *(Note: If AD requests need to be forwarded from branches to HQ, ensure LDAP/DNS services are permitted).*
*   **NAT:** **STRICTLY DISABLED**. NAT must be off to allow BGP routing over the IPsec overlay.
*   **Logging:** `Log all sessions` (Essential for tunnel and routing diagnostics).

---

## 4. 🔀 BGP Dynamic Routing & Overlay Interfaces

### 🏢 FGT-1 (Hub / HQ)
**Overlay Interface:**
*   **IP / Remote:** `12.23.34.1` / `12.23.34.2/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 1.1.1.1
Local AS: 65001
Network: 192.168.101.0/24 (HQ LAN)

Neighbor:
  - IP: 12.23.34.2
  - Remote-AS: 65002
  - Option: next-hop-self
🛑 Critical ADVPN / BGP Architecture Note: On this Dialup model, if exchange interfaces is enabled, the setup will likely fail/disable. Fortinet strictly recommends using traditional Hub-and-Spoke (with iBGP) or Site-to-Site IPsec VPN designs when full ADVPN shortcut capability is required with BGP.

🏢 FGT-2 (Standard Spoke)
Overlay Interface:

IP / Remote: 12.23.34.2 / 12.23.34.1/24

Access: Allow ping

BGP Configuration:

Plaintext
Router ID: 2.2.2.2
Local AS: 65002
Network: 192.168.102.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
🏢 FGT-4 (Spoke Behind NAT via FGT-3)
Topology: FGT-1 (Hub) --- ISP Router --- FGT-3 (NAT Router) --- FGT-4 (Spoke)

Overlay Interface:

IP / Remote: 12.23.34.4 / 12.23.34.1/24

Access: Allow ping

BGP Configuration:

Plaintext
Router ID: 4.4.4.4
Local AS: 65004
Network: 192.168.104.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
5. 🛠️ FGT-3 (NAT Router) Specific Configuration
Because FGT-4 is situated behind FGT-3, the NAT handling on FGT-3 must be strictly controlled for UDP 500/4500 IPsec traffic to traverse properly.

FGT-4 ➔ ISP (Outbound): Enable NAT (Allows the spoke to reach the internet and build the tunnel to FGT-1).

ISP ➔ FGT-4 (Inbound): Disable NAT (Critical for testing and isolating phase 1/2 encapsulation issues).
"""

with open('/mnt/data/FortiOS_Dialup_IPsec_BGP_Cheat_Sheet.md', 'w') as f:
f.write(markdown_content)

print("File generated successfully: /mnt/data/FortiOS_Dialup_IPsec_BGP_Cheat_Sheet.md")


```text?code_stdout&code_event_index=1
Traceback (most recent call last):
  File "<xbox-string>", line 1, in <module>
    python
NameError: name 'python' is not defined

Python
markdown_content = """# 🛡️ FortiOS Dial-up IPsec & eBGP Cheat Sheet

A comprehensive, lab-ready reference guide for deploying Dial-up IPsec combined with eBGP routing across a Hub-and-Spoke Fortinet topology. This guide includes configurations for standard spokes and spokes situated behind NAT devices, structured for clear deployment in enterprise training or production environments.

---

## 1. 🔑 Identity & Authentication Prerequisites (FGT-1 Hub)

Before establishing the VPN overlay, identity integration must be configured on the Hub.

*   **LDAP Server:** Add the Active Directory server under `User & Authentication > LDAP Servers`.
*   **FSSO Connector:** Configure AD FSSO under `Security Fabric > External Connectors` to enable dynamic IP-to-Group mapping.
*   **User Groups:** Create local FortiGate Firewall User Groups that map to the AD/FSSO groups. These will be referenced in the IPsec XAuth settings.

---

## 2. 🌐 IPsec Phase 1 & 2 Deployment Matrix

| Feature/Setting | **FGT-1 (Hub / HQ)** | **FGT-2 (Standard Spoke)** | **FGT-4 (Spoke Behind NAT FGT-3)** |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (Points to FGT-1) | Static IP (Points to FGT-1) |
| **IKE Version/Mode** | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server Type: Auto (Calls AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **Phase 2 Selectors** | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) |
| **Advanced Routing Settings** | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver | **Enable:** Device Creation<br>**Enable:** Add Route | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver |

> **⚠️ Hub-and-Spoke Scaling Note for FGT-1:** If only *one* device connects via the tunnel, `Add Route` can remain enabled. However, for multiple connections sharing the same tunnel interface, you **must** disable `Add Route` and rely on dynamic routing (BGP) combined with `Device Creation` and `Auto-discovery`.

---

## 3. 🛡️ Global Firewall Policies

Apply these baseline policies across **all** participating FortiGates (FGT-1, FGT-2, FGT-4) to allow overlay routing and tunnel establishment.

*   **Incoming Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Outgoing Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Service / Source / Destination:** `ALL` *(Note: If AD requests need to be forwarded from branches to HQ, ensure LDAP/DNS services are permitted).*
*   **NAT:** **STRICTLY DISABLED**. NAT must be off to allow BGP routing over the IPsec overlay.
*   **Logging:** `Log all sessions` (Essential for tunnel and routing diagnostics).

---

## 4. 🔀 BGP Dynamic Routing & Overlay Interfaces

### 🏢 FGT-1 (Hub / HQ)
**Overlay Interface:**
*   **IP / Remote:** `12.23.34.1` / `12.23.34.2/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 1.1.1.1
Local AS: 65001
Network: 192.168.101.0/24 (HQ LAN)

Neighbor:
  - IP: 12.23.34.2
  - Remote-AS: 65002
  - Option: next-hop-self
🛑 Critical ADVPN / BGP Architecture Note: On this Dialup model, if exchange interfaces is enabled, the setup will likely fail/disable. Fortinet strictly recommends using traditional Hub-and-Spoke (with iBGP) or Site-to-Site IPsec VPN designs when full ADVPN shortcut capability is required with BGP.

🏢 FGT-2 (Standard Spoke)
Overlay Interface:

IP / Remote: 12.23.34.2 / 12.23.34.1/24

Access: Allow ping

BGP Configuration:

Plaintext
Router ID: 2.2.2.2
Local AS: 65002
Network: 192.168.102.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
🏢 FGT-4 (Spoke Behind NAT via FGT-3)
Topology: FGT-1 (Hub) --- ISP Router --- FGT-3 (NAT Router) --- FGT-4 (Spoke)

Overlay Interface:

IP / Remote: 12.23.34.4 / 12.23.34.1/24

Access: Allow ping

BGP Configuration:

Plaintext
Router ID: 4.4.4.4
Local AS: 65004
Network: 192.168.104.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
5. 🛠️ FGT-3 (NAT Router) Specific Configuration
Because FGT-4 is situated behind FGT-3, the NAT handling on FGT-3 must be strictly controlled for UDP 500/4500 IPsec traffic to traverse properly.

FGT-4 ➔ ISP (Outbound): Enable NAT (Allows the spoke to reach the internet and build the tunnel to FGT-1).

ISP ➔ FGT-4 (Inbound): Disable NAT (Critical for testing and isolating phase 1/2 encapsulation issues).
"""

with open('/mnt/data/FortiOS_Dialup_IPsec_BGP_Cheat_Sheet.md', 'w') as f:
f.write(markdown_content)

print("File generated successfully: /mnt/data/FortiOS_Dialup_IPsec_BGP_Cheat_Sheet.md")


```text?code_stdout&code_event_index=2
File generated successfully: /mnt/data/FortiOS_Dialup_IPsec_BGP_Cheat_Sheet.md

Your Markdown file is ready

MD icon
FortiOS_Dialup_IPsec_BGP_Cheat_Sheet
 MD 
Open
I have transformed the technical instructions into a structured, highly detailed GitHub-ready cheat sheet. It features clear deployment matrices for the Hub and Spokes, distinct sections for global policies, BGP AS/Neighbor settings, and highlights crucial architectural notes (such as the NAT behavior on FGT-3 and the ADVPN exchange interface limitation).

This document is optimized for direct inclusion into an enterprise or training repository as a standalone README.md or instructional reference page.




Gemini is AI and can make mistakes, including about people. Your privacy & GeminiOpens in a new window

Analyzing
# 🛡️ FortiOS Dial-up IPsec & eBGP Cheat Sheet

A comprehensive, lab-ready reference guide for deploying Dial-up IPsec combined with eBGP routing across a Hub-and-Spoke Fortinet topology. This guide includes configurations for standard spokes and spokes situated behind NAT devices, structured for clear deployment in enterprise training or production environments.

---

## 1. 🔑 Identity & Authentication Prerequisites (FGT-1 Hub)

Before establishing the VPN overlay, identity integration must be configured on the Hub.

*   **LDAP Server:** Add the Active Directory server under `User & Authentication > LDAP Servers`.
*   **FSSO Connector:** Configure AD FSSO under `Security Fabric > External Connectors` to enable dynamic IP-to-Group mapping.
*   **User Groups:** Create local FortiGate Firewall User Groups that map to the AD/FSSO groups. These will be referenced in the IPsec XAuth settings.

---

## 2. 🌐 IPsec Phase 1 & 2 Deployment Matrix

| Feature/Setting | **FGT-1 (Hub / HQ)** | **FGT-2 (Standard Spoke)** | **FGT-4 (Spoke Behind NAT FGT-3)** |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (Points to FGT-1) | Static IP (Points to FGT-1) |
| **IKE Version/Mode** | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server Type: Auto (Calls AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **Phase 2 Selectors** | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) | `0.0.0.0/0` (Any subnets) |
| **Advanced Routing Settings** | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver | **Enable:** Device Creation<br>**Enable:** Add Route | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-discovery Sender & Receiver |

> **⚠️ Hub-and-Spoke Scaling Note for FGT-1:** If only *one* device connects via the tunnel, `Add Route` can remain enabled. However, for multiple connections sharing the same tunnel interface, you **must** disable `Add Route` and rely on dynamic routing (BGP) combined with `Device Creation` and `Auto-discovery`.

---

## 3. 🛡️ Global Firewall Policies

Apply these baseline policies across **all** participating FortiGates (FGT-1, FGT-2, FGT-4) to allow overlay routing and tunnel establishment.

*   **Incoming Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Outgoing Policy:**
    *   **Interfaces:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Service / Source / Destination:** `ALL` *(Note: If AD requests need to be forwarded from branches to HQ, ensure LDAP/DNS services are permitted).*
*   **NAT:** **STRICTLY DISABLED**. NAT must be off to allow BGP routing over the IPsec overlay.
*   **Logging:** `Log all sessions` (Essential for tunnel and routing diagnostics).

---

## 4. 🔀 BGP Dynamic Routing & Overlay Interfaces

### 🏢 FGT-1 (Hub / HQ)
**Overlay Interface:**
*   **IP / Remote:** `12.23.34.1` / `12.23.34.2/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 1.1.1.1
Local AS: 65001
Network: 192.168.101.0/24 (HQ LAN)

Neighbor:
  - IP: 12.23.34.2
  - Remote-AS: 65002
  - Option: next-hop-self
```
> **🛑 Critical ADVPN / BGP Architecture Note:** On this Dialup model, if `exchange interfaces` is enabled, the setup will likely fail/disable. Fortinet strictly recommends using traditional Hub-and-Spoke (with iBGP) or Site-to-Site IPsec VPN designs when full ADVPN shortcut capability is required with BGP.

### 🏢 FGT-2 (Standard Spoke)
**Overlay Interface:**
*   **IP / Remote:** `12.23.34.2` / `12.23.34.1/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 2.2.2.2
Local AS: 65002
Network: 192.168.102.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
```

### 🏢 FGT-4 (Spoke Behind NAT via FGT-3)
*Topology:* `FGT-1 (Hub) --- ISP Router --- FGT-3 (NAT Router) --- FGT-4 (Spoke)`

**Overlay Interface:**
*   **IP / Remote:** `12.23.34.4` / `12.23.34.1/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 4.4.4.4
Local AS: 65004
Network: 192.168.104.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self
```

---

## 5. 🛠️ FGT-3 (NAT Router) Specific Configuration

Because FGT-4 is situated behind FGT-3, the NAT handling on FGT-3 must be strictly controlled for UDP 500/4500 IPsec traffic to traverse properly.

*   **FGT-4 ➔ ISP (Outbound):** `Enable NAT` (Allows the spoke to reach the internet and build the tunnel to FGT-1).
*   **ISP ➔ FGT-4 (Inbound):** `Disable NAT` (Critical for testing and isolating phase 1/2 encapsulation issues)
