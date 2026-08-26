ACS
cpu 1 sock core
ram 4gb
hdd 120 gb
redhat 

in installation of acs select option 1 to fully install
if select option 3 we can reset password 

configs like cisco 
for crackinf acs must select power on frameware with another linux
flxm.jar
5.8.tar
in ( dev/dm5 > cscoaccs > mgmt >apache > line) replace flxm.jar
for another file 5.8.tar must goes to acs and type dir disk to find the location of update files
dev/sda2 > place the 5.8.tar

now we can license the acs in web mode
https:// address of acs

if not found :
in acs :
	acs start managment 
	show application status acs

user of acs in web mode
user : accsadmin
pass : default

upgrade :
	make repository in acs for upgrade

	dir disk
	conf t
	repository repo 
	url disk
	acs path install name-of-package repository repo

in web mode :
	system admin 
		software repository

ASA

in gns3 we can upload files of asa like template 
asa need flash space in cmd we can type and find gns3 path :
cd qemu2.2 > qemu_img.exe create flash 512m

then select the flash sapce in gns3 path 
goes to hdd for it.

uncheck used legacy networking mode

if we cant launch asa must change value of binary asa to x86-64

conff t
int g 0
ip add 192.168.1.1 255.255.255.0
nameif inside 

nameif makke ecurity level to have accessability 0-100 for networks and users and services
inside is 100
outside is 0

show asdm image

asdm is a appication to access asa easier 

show flash 
copy tftp: flash:

asdm image flash:asdm7.2.1.bin
http server enable 
http 0 0 inside

in web mode:
https://192.168.200.1 (web bas + java)

in senario:
asa  :
like asdm launching 

username admin password 123abc privilege 15
clear config user admin 
aaa authentication  http console local (for use username and password in web)
aaa acces > http /adsm must be enabled

security level define direction of data transfering 
if use value of 50 can access ower than of the values but for highers cant

here we have statefull term works like syn-synack and makes transmissions easier between diffrent securitylevel
by default ue http and https

for exeptions :
asdm > config > firewall > service policy rules 

if in firewall we didnt see these logs must set them to ue catalog of ports :
conf t
class-mao inspectoni-default
match default-inspection-traffic
policy-map type inspecton dns preset-dns-map
parameters 
message-len max 512
policy-map global-policy
class inspection-default
inspection dns preset-dns-map
inspect http
inspect ftp
inspect sip
inspect smtp
exit
service-policy global-policy global

fixup protocol icmp
easier way to insert protocols

for ssh to asa:
crypto key gen rsa general  modul 1024
ssh 10.11.0.0 255.255.255.0 inside

for connectivity between dmz and out we can use rules :
asdm > config > firewall > access rule 
	have some parameters that ordered by security level

	add access rule
	port outside 
	source any
	destination dmz
		we can add object of one thing in rules (add network object type hot ip 10.22.0.10)
	service http

in asa command mode:
object network webserver
host 10.22.0.10
access-list outside_access_in line 1 extended permit tcp any object webserver eq http
access-group outside_access_in in interface outside

syn flooding
hacker hack 3way handshake 
in backtrack > ifconfig ether0 5.5.5.9 netmask 255.255.255.0 
				route add default gw 5.5.5.254 ethe 0

				hping -I ether 0 -u 5.5.5.10 -s 10.22.0.10 -p 800 -i u1000

in asa :
show connection count
show connection 
show connection details

asdm > config > firewall > sevice policy rule
	add > global 
			use class-default
			in connection tab  > set max embryonic connection 10

in asav v 9.3.1
we can use vm and launch asdm config like before but add this to asa
ssl encryption rc4-md5 aes 128-sha1

for unicast rpf (reverse path forwarding)
asdm > config > firewall > advance > anti spoofing 
	double click on untrusted port

nat :
asdm > config > firewall > nat rules
add > network object nat rule
type network
range ip > 192.168.1.0 255.255.255.0

nat types are :
static 
* dynamic pat * 
dynamic 

show xlate (live nat in asa)

botnet filtering :
asdm > config > firewall > botnet traffic filter
download database and use dns for it


high availability :
we have actiive and standby mode in asa ha
in ha we have one port for data synchronization and one for checking availability
in primary if make some thing we can see side effects on secondary
if secondary get active make garp package to define switch new output port
asa1 :
conf t
int g 0/0
no sh
nameif inside 
ip add 192.168.0.1 255.255.255.0 standby 192.168.0.2
ex
int g 0/1
no sh 
nameif inside
ip address 192.168.1.1 255.255.2555.0 standby 192.168.1.2
exit
int g 0/2-0/3
no sh
exit
failover lan unit primary
failover lan interface config-lan g0/2
failover interface ip config-lan 192.168.2.1 255.255.255.0 standby 192.168.2.2
failover link state-lan g0/3
failover interface ip state-lan 192.168.3.1 255.255.255.0 standby 192.168.3.2
failover key shayan
failover replication http
failover

we must set these on asa2

show failover
failover active

asa1 :
asdm > wizard > ha > active standby
	ip asa2
	in asav we have problem in cpu 

security context :
asa by default is one view or single part 
with this option we can splite asa to 2 or 3 part 
but we cant use dynamic routing 
asa cant be vpn servver 
have shared interface 

show mode
show activation-key
conf t
mode multiple

context on asa :
	1 : system (single mode + multiple)
			controlll other contexts
			for ping problem must command this for mac changing :
				mac-address auto
	2 : admin  (multiple)
		flash:/admin.cfg
	3 : others

for changing between contexts :
changeto contexts admin
changeto system

if we code no shut in one context shows it in self context

class shayan
limit resource ssh
limit resource tft
limit resource all
limit resource telnet
limit resource xlate

context shayan
member shayan

set some ports for context :
allocate interface g 0/1 i1 (i1 > is aliases)

config-url flash:/shayan.cfg

convert shayan to admin :
admin-context shayan

changeto context shayan
conf t
int i1
no sh
ip add 192.168.1.0 255.255.255.0
nameif inside

asdm launch steps

redo these steps for admin context
launch another asdm together 

asdm > config > device setup > interface
