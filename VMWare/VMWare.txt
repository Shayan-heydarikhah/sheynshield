VCP

in kernel of os we have some rings :
1 > ring 3 application 
	2 > 1+2 > drivers (1 use for cpu rama ...) (2 use for other prepherals)
		3 > 1 > ring 0 kernel

each application hase pid in tasks in real world os comes with pids.(hypervisor)

hypervisors share hardwares to each vm and manage usages (if attached hdd reset and format all) :
types :
	1 > hyper-v , citrix xen , esxi
		works directly with hardwares

	2 > kvm , vmwareworkstaion , virtualbox
		works on os host or intermediate cant expand type1 in this mode

customized image > a company like hp sell a hardware then download esxi from vmware site and install hp drivers in source , upload it in net (hp image)(digiboy)
offline bundle > update without bootable device 

boot modes :
	1 > mbr > esxi > less than 2 tb hdd
	2 > gpt > upper win 8.1 > higher than 2 tb hdd

after esxi installation we connect to esxi with web 

advance setting > registery windows
	swaploation >  change hdd in boot
	autostart > start automatically

add ip range of front view network

benefit of ipssec is syncablity of many devices
kerio vpn has speedy connection 

ntp setting > 172.16.15.1,192.168.1.1

pci > toggle passthrough  then reboot it and attached to vm (applicated on voip , nvidia grid) > sharing of resources

in services must active ssh

acceptance level > 
	1 > parent 
			accept packages with vm company and other companies
	2 > vmware cert	
			hard way > must get certs from vm and install 
	3 > vmware accept
			certs get free and install
	4 > community
			every thig get install its better test on desktop not in server

ovf > file is config files + disk
ova > is single file

register on esxi... > if lost a vm this option looks inside esxi and find vm

compatilability > its better be on latest version 

numa > none uniform memory access
	core per sockets (effects on multithreads task)
	count of sockets in vm must be same as count of cpu on moherboard

cpu ha add >   less stabillity
if  hot add checket and attached new cpu we can use it like new more socket but cant use their cores

reserve > reserve cpu for one person and ommit delay in tasking
share > higher number has much priority
limit

core sharing >
	to 8 machines give 8 cores or devide it  or use all cores by priority sences.
	1/64
	8*64 > 512
	4*64 > 256
	2*64 > 128
	.
	.
	.
	.

expand hd assisted  virtualization to guest os 
	nested virtualization

enable cpu performance counter 
	some apps need counter
		use much loads in system

in ram when we set reservation that our hdd get fully and we dont have swap and must be locked 
in less memoy capacity we have slowness working

reserve  all guest .... > when hdd has little capacity must locked ram and dedicate value of ram to force deactivation of swap.

hdd is vmdk format
thin provision > 10 gig set for vm but didnt attached to vm how many need in time can get ( 10 mb writing > 10 mb formatted then write) fast provision

thick provision (lazily zerod) > 10 gig attached to vm dedicate ( 10 mb writing > 10 mb format , write with 0 then write) more effective than thin provision but slowly at creating (block by block)

thick provision (eagerly zerod) > 10 gig attached to vm dedicate (feel all blocks with 0 then format again hdd for bad sectors) reliable.clusterng and fault telorance.

in work space must set thick
in thin mode mabe we ommit vm but in hdd we have effects and didnt delete

gpu must set in 8 mb for full view

customize setting > vmoptioin > vm remote console option >
	lock..... 
		if some body logged in and logged in another system , locked first session and close it

	max ... > 1 persion can additional logging in another system


its better opened by vmrc

its better set ppowr manager to standby .

force bios setup > admin can goes to bios in next restart.
failed boot recovery > until find boot file reset (network booting)


some times we need a nic (physical network interface card > uplink (nic on server)) converting for transfering a lot of data
solution of this task is vswitch (virtual 2960 sw)

esxi > networking
	vm-kernels > virtaul nic in esxi can get ip and mac
	virtaul machine port group > vlanning and vlan making
		esxi > networking > portgroup
		if set 4095 means trunk

	in vswitch must set outgoing interface or nic to vmnic get trunk (by deafualt is trunk)
	in hardware switch must set trunk port

	on hosts must click edit setting and set network adaptor
	some times we need nested virtualization or have firewall in our network and have authentication 
	solution of this part is trunking port and make vmnic behinde that host

	switches in vmware:
		1 > standard
		2 > distribute
			enterprise + 
		3 > thirdparty

	security in portgroup (just for trunk must allowed these items): 
		1 > promiscouse
			recieve a packet that isnt for vm
		2 > mac change
		3 > forged transmission (like port security in cisco)
			transferd packets had another source

	traffic shaping > perport assinging a value 
			peak time
			avg time
			burst time

inherite from vswitch
link discovery > cdp (cisco) , lldp (other vendors)(link layer discovery portocols)
	send > listen 
	recive > advertise
	both
	none > disable

mtu > mximum transmission unit

some times we need many uplink for this task we have nic teaming
	load-balancing > standard >  route base on ip hash src and des
	vmxnet 3 > 10 gig
	network fail over detection > 
		becan link
			ping outgoing port 
		link state
			links light
	notify switch
		from their mac table advertise their mac between links without broad casting

vmkernels are connects to port group and cant give theme ip but can make vlans 
vmk-0 > management 
	cant change ip 
tcpip stack > default route + dns  +....
after define vlan in vmk we can set type of vmk (management or vmotioin or ...)

storage >  at first time attaching device format in vmfs form
	diskmgmt.msc

storage controler > raid controler

storage type:
	das (direct attache storage)
		cheap - designed to one os handeling 
	nas (network attache storage)
		expensive - usable by many oses
	san (storage area network)
		large scale 
		computers -------------------------------------------------------------------------- storage area
					need hba to connections to san
					hba is switch or card 
														fiber cannel (has itself protocol stack + tcpip)
														san switch
														storage processor (jbod (transfer managing of hdd to os) we can use many jbods)
														refs

	lff (huge size of hdd in pc)
	sff (small size of hdd in laptop and externals)

cluster gatten
hyper-converged:
	get all storages in network together and make them in one logical unit (lun)
in running san with fiberchannel must config all settings  in storage side
in iscasi mode must config in esxi side
	1 > san and vmk must be in smae vlan 
	2 > that vmk must goes out from one nic 
		nic teaming doesnt support iscasi , solution is make 2 different vmkernel and give them vlan and ip
		must create vlan port for each iscasi and vmkernel  in nic teaming then load balance it then disable other nics and enable just one link for working 

	in windows serrver must roll up fs and iscasi target server (for running san in network)
	set ip in nic in win servers but unchecked services for iscasi
	lun must be in one target not 2
	targets can be in many disks
	target must be create for each consomrs
	target means share ;)
	in disk pool we hhave raid that include vdisk is 1tb of lun
	in target must define useres or consumers

config of iscasi in vm 
	1 > vmkernel 
	2 > portgroup ( edit > nic team disable)
	3 >  storage > adaptor > software iscasi (enable)	
		network port binding  > set outgoing vmk 
		iscasi port 3260

	after these must define hard to esxi in datastore

multipathing with round roubin algorythem has effective performance
for useablity of two link in iscasi mode must goes to :
networkiing > vswitch > add uplink

vmkernel nic > add vmkernel > ip and vlan set

for each vmkernel have port group

installation of vcenter 
	1 linux (vcs)
	2 wiindows (vin)

	2type of install :
		1 appliance
		2 in windows

	embeded > one dc (less than 10 services)
	external > many dc (more than 10 services)(distribute style)

	has pass for linux kernel of esxi and  vcsa 
	after that instalation of vcenter get start

	vcenter server appiance  (vcsa) :
		stage 1 : copy vms
		stage 2 : copy services

		has more than 50 services

	installatio of vcenter is like dc if install  in one place others can access it
	in installatio must make a record in dns server with ip fqdn
	if install on windows must join domain that windows

	sso config (single sign on)
	before these updates for appliances we dont have this option 
	for relating between winservver and esxi must define admins with different values (not bee same in ad and vcenter)
	when show us join to domain its dident get embeded
	this sso on vcenter aggrigated with  psc (platform servvice controller)
		if logged in to one vcenter automatically logged to others

	flex > need flash palyer
	license >
		1 > hosts
		2 > vcenter

	vcenter > admin > config > identity source (tell device where is users and how can read them)
	ad windows integrity  (v center joined to domain)
	ad over lldp (connect without joining)
		dc=rayancollege,dc=com
		dc >
			any 
			specific
				ldap://192.168.1.1:389

	first make data center 

	DCUI (Direct Console User Interface)
	locked down hold (any connections with root user from vspheer client get locked and just logable with vcenter in ui mode)
		1 > normal (every one has task with vcenter block  pass through from esxi)(if get crash must goes to console)
		2 > stricket (just need reinstall esxi os)(must enable ssh)

	some times we need change server for vm and dont want change storage
	must set same or shared storage for task and just change ram and cpu of vm or serrver or host
	vm kernel for this task is vmotion mode 
	must be similar vmk in many machiines or hosts
	
	change compute migration 
	all port gorups must be same

	clustering 
		must use shared storage
		
		drs > loadbalance
		dpm > power manager 
		
		failor and response 
			enable host monitoring 
				netwrks and connected cable to switch get monitor (if get down vmotioin and migration take machine off then transfer it)(gw ping is the way of solution)
		
		default vm start priority > in restart of vm which one is before others.
		
		response for host isolation (if vm hasnt network )(vmware tools must be installed)
			shutdown and rstart vm (shut machine and restart in another host)
		
		datacenter pdl > 
			if the way of host to san get down what should we do (better be disable)
		
		datacenter apd (per vm) > delay 3min ime out 140 sec
			restart conservetive 
				wait a minut for up or starting vm
			restart aggresive
				wake up vm instansly

		vm dependency restart condotion (0 is in moment)
			if resources get down what should do
			resource allocation (decide from their resources)
			guest heartbeat hardware detect(turn on os then calculate then start other vm)

		if our vms works near 100% msut run admission control and set maximum  percentage 
		define host failover capacity by :
			1 > slot policy
				1 > cover all 
				2 > fixed
			2 > dedicate host
				Proactive HA is a new solutioin for detectiing problems in vms

			heart beat data store > if vms network get down from which data store location load data
			advance otion is regedit for each machine 


automatioin level > 1 - 2 mins take along to migrate in new host
	manual (all manual)
	partial automatically (making + power on (auto) + migration (manual))
	full automatically (all auto)

	where should launch vm > dsr decide

	conservetive > lowe 
	aggresive > high  (balance ram and cpu)(vps little vms and alot)

	dont migrate vms just for usage value migrate on count of vms

	vm roles > we can set some roles deny has higher priority 
	vm to vm 
	host to vm 
	this to that 
	all of them 
	seperate of them

cloning in vcenter
	install vmwaretools
	customize os
	sysprep

	menu > privacy and policies and profiles
		customization specification

		convert to template

update esxi > 
	esxcli software vib update -d /vmfs/values/lun0/esxi....

port allocation > 
	1 > elastic
		until this host is up work correctly if get down switch to another one
	2 > fixed
		just for monitor

vcsa : 5480 port number

restart network management > esx clie
	release and renew in cmd
	clear arp cache

advance tshoot in shell :
	alt + f2
	alt + f1

	for using linux envirment must goes to bash-shell :
		sevice-controll -status

boot storm > in many vms turnning on maybe have slowness

ilo > integrated light out
ipmi portocol

for upgrade must goes too boot menue and try esc many times insert cdrom
easier way is upload update file in system and upgarde it with boot menue
quick boot >
	in vers 6 - 7 and upper we can just reboot esxi

over commit > 
	assign more resources from origin  resource on board

sata > cdrom
scsi > hdd ssd

paravirtual >
	is like bridge on mother board and get servers cpu cool down because this option collect commands then send to servers cpu

for logging must set vlan in stage 1 installation between esxi and our network
if we logged in and see error from our user pass:
	service.sh restart

if dns side was ok must set dns and a record in our windows ;
	c:\windows\system32\drivers\etc\host
		insert ip and host name 

transfer all vmkernels and vms and adaptors to new vswitch :
	create new vswitch
	add their own ports
	then select (migrate vmkernels network adaptor to the selected switch)
	put them in same vlan 
	go to networkiing tab and select datacenter then distribute switch (upper is better)
	config > topolgy
		add hosts
	in transfering data between standard switch  >  distribute switch must not disconnect with vcenter

	roll backs :
	1 > physical adaptors
	2 > vm kernel adaptors
	3 > migrate vms networking

	its better do it step by step 
	for avoiding of confilicts must do step 1 + 2 together

	multicast address are destination not a source

	assign ports
	static
		always goes with that port
	dynamic (expire)
		port binding change ports at retart 
	epherenal
		in runtime works like dynamic wen done delete all ports

	trunk vlan range - private vlan - i/o traffic shaping
	monitoring + netflow - acl - cos - dscp - shutdown command
	packet marking (higher is better)

	lacp > link aggregation contorl protocol
		beeter be passive

	we can migrate our data in lacp but must set it in standby mode (suspend others must be diactive)
	monitoring shows details of network 
	netflow show type of traffics

	ip collector (9596 port)

	if forget password :
		install new esxi
		preserve data store
		run live linux
		open password shadow
		and clear hash of admin ppart
