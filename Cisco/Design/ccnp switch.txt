ccnp switch :
common :
	ip domain-lookup 
	ip name-server 0.0.0.0
	
	telnet on switch :
		hostname switch-1
		interface vlan 1
		no shutdown
		ip address 192.168.1.2 255.255.255.0
		exit
		services password-encryption
		username shayan password 123 
		enable password 123
		line vty 0 4 
		login local 
		loggin synchronous 
		no exec-timeout

	ssh on switch:
		enable
		configure terminal
		interface vlan 1
		no shutdown
		ip address 192.168.1.2 255.255.255.0
		hostname switch-1
		ip default-gateway 192.168.1.1 (use for ssh on vlans)
		ip domain-name test.com
		crypto key generate rsa module 1024  
		ip ssh version 2 
		username shayan secret 123 
		enable secret 123
		line vty 0 4
		login local 
		loggin synchronous 
		no exec-timeout
		transport input ssh
		transport output none

	it's better use 2960 (8-ports) for camera and dvr
	catalyst 3550 > cheap, layer 2, routing, no ipv6, no stackwise, no private vlan
	catalyst 3560 > layer 2 & 3, private vlan, qos, no stackwise
	catalyst 3750 > is best choise support all options at above

	if we don't have voip service, doesn't matter have poe support on our switches

	network segmentation > vlan give this option and decrease our broadcasts

	we have 3 layers design in lan side :
		1 > core layer switches and devices
			the best switches in world must be used in this layer
			it's better don't write acls in this layer
			qos (shapping)
			redundancy
			fhrp

		2 > distribute layer switches
			aggregation of many switches
			can handle layer 3 and routing (mls)
			security and policy base acl3
			qos
			support sfp and scaleability + redundancy

		3 > access or leaf node layer switches
			this layer must use poe and high port density, scaleable to higher layer wit uplinks
			high available (stack), qos and ablity to convergence network services (data, voice, video)

		we have compressed layer core and deistribute layr can be compressed and tell collapsed-core

	lom > list of material

	what happend, when a packet or traffic comes to switch?
		in layer 2 > cam table + tcm table 
			cam table : show mac-address-table
				in real world when we write this command see some mac address, these are built-in
			layer 2 switches have ingress queue (incoming flow)(recieve) and egress queue (outgoing flow)(transfer)
			cam table includes : vlan + egress port to second client + mac second client 

			tcam table is optional setting and policies that checked after cam table flow, these settings are like qos and acls

		show sdm prefer (show us capacity of mac table)

		show mac address-table (cam table)
		show mac address-table count (show us capacity of mac table in eve emulator)

		show mac-address-table (for 4500 and 6500 series)
		show platform tcam utili

	what is differences between routing in mls and router?
		in mls we have topology bae concept for routing
			contains :
				fib (orward information base)
				adjuncy table
				rewrite engine
			cef (cisco express forward):
				is a mechanism that handle asic chips and routing in mls if get disable our cpu usage goes over 80%, bydefault is enable and must be enable, in 6500 seriess don't give access to disable it.

					without cef we have longer process like this :
						routing table > arp table > mac table (cam table)
							each time need to route or find mac of client must check aand pass these process

					now see process of cef procedure:
						in cef mode in additional to have cam table use tcam and fib
						fib table :
							hardware lookup
							sorted records
							host records
							resolved records

							with fib our cef and asics process faster

							ip address > nex-hop ip address > nex-hop mac address > egress port

					some times works with cef and some times not, why?
						for first time for chaching and learning the way must run without cef then works with caches and cef
						exactlly works without cef to complete arp table and mac table

					after stable state our packets firs goes to fib then make hash for each host and match traffics with them
						show adjency detailes (sho hash of hosts)
		some times we have confilict and can't match packets with cef nd fib in term we say glean effect:
			show ip cef adjuncy glean
		
		some times we need change our ports from management to access mode (convert mac counts) 
			sdm prefer router (mls)
			sdm prefer access (normal layer 2 switch)
			exit
			reload
				must reload that to make changes
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
for more than 5 min saving mac :
	has second parameter
	mac address-table aggregation-time 10 
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
for define specific mac to device :
	must use ios 12.x and above
	mac address-table static .... vlan .... interface ...
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
STP > spanning tree protocol
	in router we have ttl for no loop data
	in switches we have spanning-tree for that
	by default is enable on each switch

	modes or versions :
		802.1 D :
			common > cst or stp > supports by all devices (slow)
			per-vlan spanning > pvst > cisco propriatary (higher speed (per vlan execute a tree))
	
		802.1 W :
			rapid > rstp
			per-vlan rapid > pvrst > cisco	propriatary (very fast (each vlan has one tree and has more loads on device))

		802.1 S :
			multiple > mstp 

	root bridge :
		bridge id :
			lower mac + priority that is default 32768 + vlan number

		dp or designated ports are selected ports that forward traffics to root switch
		all root switch must be dp 
		each switch must select best path to root bridge

		bridge is term from past
		
		show spanning-tree

		each port has cost
			fast is 19
			1 gig is 4
			10gig is 2

		port priority :
			port number 
			has more effects and priority on little interfaces
			must config in front view

		port number :
			if connect to fast then one port connect to gig
			the stp see port fast 25 instead of gig

	for change bridge id or root bridge must set these :
		spanning-tree vlan 11 priority (x4096)
	if give lower than 32768 (the default value) can win

		values make root port selection :	
			lower interface cost
			bridge id
			port priority
			port number

	we have direct and indirect changes :
		direct 
			15 lsn + 15 lrn > 30 seconds
		indirect
			15 lsn + 15 lrn + (10 * 2 wait for root switch and don't accept second switch bpdus for masters mode) > 50 seconds

		learn > getting mac address table
		listen > get bpdu in each 2 seconds

	in rapdid mode of spanning tree :
		each device send bpdu in normal mode just master sent that
		instead of 10*2 waiting we wait 3*2
		and we don't have lsn

	we can set faster way to load users in spanning tree :
		interface range fast 0/0-0/10
		spanning-tree portfast

			debug spannig-tree events (topology change notice (tcn) sends to root if our client goes down, each state advertise with tcn)

			in global mode for access layer switches (not higher) not roots we should use fast upping link (direct changes) :
				spannig-tree uplinkfast
					after do this our changes are tese :
					1 >	bridge priority get 49152
							why? because speed and loopffree option, to prevent get root
					2 >	cost for port get increment over 3000
					3 >	didn't check 30 seconds on direct changes and goes to forward mode

					these settings are enable by default in rapid mode
					we learn these to change standard spannig-tree and behave like rapid
			
			for indirect changes we wait lsn and lrn (30 second), if a link get down our switches get start to advertise bpdu and who is root so for prevent this confilict we set these commands to stay root our root bridge:
				spannig-tree backonefast
					it's better set on all switches
					if set these commands the middle switch chech priority and mac address to see wich one is master
			
			for client side (access layer) fast links :
				spannig-tree portfast edge
					if set on global, automaticaly detect access ports and make them to fast mode
	
	for use client side command its better (access - portfast - no channel group) : 
		switchport host
	
	see inconsistetnt ports :
		show spannig-tree inconsistetntports

	port fast disable lsn and lrn must be carefully because mabe make loops
	
	must block bpdu packets in client side ports :
		interface range fast 0/0-0/10
		spanning-tree bpdugaurd enable

	spanning-tree bpdugaurd filter (don't send stp)

	root gaurd :
	some time we set a switch on force mode to be root but also another switch is root, to force our policies:
		in root switch and their interfaces we must set these :
			int range gig 0/0-2
			spannig-tree gaurd root
				even if find better priority and detailes in lan with bpdugaurd packets didn't give a way

	at the end of configure spanning-tree :
		spanning-tree mode rapid

	insignificant alarm is client down state

	for load balance mode in switches must set these commands on mls 
	before that must define all vlans with vtp in devices then 
	set some vlan to switche 1 and switche 2 for load balance :
		switch 1 :
			spanning-tree vlan 1,10 root primary
			spanning-tree vla 20,30 root secondary

		switch 2 :
			spanning-tree vlan 20,30 root primary
			spanning-tree vla 1,10 root secondary

	in spanning-tree game if all interfaces be fasst ethernet our gig ether nets seems fast

	udld :
		in fiber optic cables we have one way send and one way receive, maybe on receive way we have problems it's better use these suggestions because spannigtree game sence loop for prevention of this must:
			normal > use syslog
			aggressive > use error disable (it's better)
				global 		 	: udld aggressive
				interface 		: interface gig 0/0
									udld aggressive
						
		some times we need to block spannigtree (it's different with bpdugaurd) :
			spannig-tree portfast bpdufilter default 
				this use light load on device and unusauly deployement

	til here we use rapid spannigtree this version use portfast and wait 3*2 seconds, all switches sends bpdu packets, have discard and lrn and forward state on ports but have one problem:
		use per vlan rapid state means each vlan has a seperated spannigtree process and for alot of count of vlans we have much load on device
			soloution is mst

	mst :
	for sync our mst we must have these same on each device :
		instance-to-VLAN mapping table 
		configuration name 
		configuration revision number 

		by default all vlans are in mst instance 0 the says that internal spannig tree (ist)
		we can create 15 mst	
		we can use mst with normal spannigtree or rpvst.
		
		(must be same at area):
			name test 
			revision 1
			instance 1 vlan 100-200
		
		show pending 
		show current 
		exit
	
	vtp v3 + mst :
		on root :	
			for combining vtp and mst must set mst and vlans server primary:
			(priviledge)	
					vtp primary vlan force
					vtp primary mst force
			vtp mode server vlan
			vtp mode server mst
			
			vtp domain test.com
			vtp version 3
	
		creat vlans.
			vlan 100-300
			exit
	
		set mst to root :
			spannig-tree mode mst
			spannig-tree mst configuration
			name test
			revision 1
			instance 1 vlan 100-200
			instance 2 vlan 200-300
			show pending 
			show current 
			exit
	
		do sh spannig-tree mst config
	
		on others :
			vtp mode client mst
			vtp mode client vlan
			spannig-tree mst 1 root primary
			spannig-tree mst 2 root secondary
			spannig-tree mode mst
			spannig-tree mst configuration
			name test
			revision 1
			instance 1 vlan 100-200
			instance 2 vlan 200-300
			show pending 
			show current 
			exit
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
vtp :
	vlan trunking protocol
	get each 60 seconds update use trigger update option
	define all vlans with one config on one device to other devices(switches).
	vtp is cisco exclusive
	gvrp is standard term
	must set trunk on all linked device
	by default is on in all devices and on server mode
	has 4 modes
		v1,2,3 > server (define to others can have one primary in versioin 3 and many secondary)
		v1,2,3 > client (just learn not more)
		v1,2,3 > transpparent (just transfer the data not more no action no learn)
		v3     > off 

			it's better run version 3 on all devices, cause is hash authentication 

			on server mode we can learn new vlans from clients
			each 300 seconds server advertise vlans
			we have a type of message that clients send to server
			in version 3 we can make 4097 vlan
			vtp join message > is a message from access layer says this person joined to vlan x
			vtp server has 2 mode :
				primary > we don't have one more and can do every thing
				secondary >  in normal mode
			version 3 has mst and hidden password 
			version 3 can work with version 2 but has confilict with version 1
			in version 3 we have mode off, in interface and in global
			version 3 has remote span

	why after making vlas the device don't say i make that vlan ?
		because we make client mode

	must set vtps to client mode in access-layer in network

	in vtp we have revision number
	this number define changes and synchronization unit

	if use geographical design we can use vtp prunning option to use less broadcasts in device and make lower load.

	for erase vlan configs of vlans must behave like this :
		erase startup-config
		reload
		dir flash:/
		delete flash:clan.dat

	for making vtp must set these commands on switch:
		vlan 10,20,30
		exit (must exit from vtp and vlan to create them)
		feature vtp
		vtp domain test.ir
		vtp version 3
		vtp password shayan (visible in show vtp password)
	
		for password in vtp we can set modes:
			Hidden–Password is not saved as clear text in vlan.data file. Instead, a hexadecimal secret key generated from the password is saved. This is displayed as the output of the show vtp password
				vtp password shayan hidden
			Secret–Use this keyword to directly configure the 32-character hexadecimalsecret key. System administrators can distribute this secret key instead of the clear text password.
				vtp password shayan secret

	by default vtp mode in ios 15 on server mode
	when we set password to vtp v1 and v2 we can see that in running config or #show vtp stats in clear text mode

	but in v3 we can set hash mode or secure mode for passwords.
	and it doesn't need this :
		service pasword-encryption (make our passwords hash level 7)
	
	in vtp we have powerfull command that filter our broadcasts in geographical topology if our vlans in some switches ommited, vtp detect that:
		vtp prunning 
		switchport trunkpruning vlan
			configure the list of vlans allowed to be pruned from the trunk for explanations about using the add, except, none, and remove keywords, Separate nonconsecutive vlan ids with a comma and no spaces; use a hyphen to designate a range of ids valid IDs are 2 to 1001 extended-range vlans (1006-4094) cannot be pruned vlans that are pruning-ineligible receive flooded traffic the default list of vlans allowed to be pruned contains VLANs 2 to 1001.

	some times we add switch in our lan, it used in server mode in another lan, to force our policies to make our switche primary must set this:
		vtp primary force
			if set just (vtp primary) and make vlan didn't access us to create vlan beacuse (vtp primary) means in normaal mode secondary not primary must set (vtp primary force)

	show vtp devices
	show vtp status
	show vtp password
	show vtp counters
	show vtp interfces
	show interface fast 0/1 switchport (displays the vtp pruning eligibility of the trunk port the default is that all the VLANs from 2 to 1001 are pruning eligible)

	for disable and suspend some vlan :
		vlan 10
		state suspend

	when we set trunk mode to dynamic desirable it takes isl type for transfor data
	isl is cisco exclusive and dot1q is standard version of trunking protocol that use native vlan and 4 byte in headers
	isl have more steps for packets analyze in real word have seperated device to use it eacuase use a frame of client packets and 30 byte in header, doesn't support native vlan

	we should set trunk for using 802.1q mode to use less mtu part and bypass baby giant:
		switchport trunk encapsulation dot1q
		switchport mode trunk

	what is baby giant?
		in accounting of mtu we set in first days value on 1518 and in difinition of encapsulations we say isl use +30 byte and dot1q use +4 bytes for packets and mtu
		cisco for isl use hardware soloution and dot1quse logical way and change mtu size

		by default encapsulation of vtp is desirable that means isl is enable and must change it to dot1q
	
	switchport modes and mixing up :

		dynamic auto
			if no one comes to negotiate makes mode to access
		dynamic desireable
			dtp > dynamic trunkig protocol > negotiate to get trunk
		trunk 
			negotiate nd allowes many vlans get transfer(dtp)
		access
			didn't have negotiation option and allows one vlan transfering

	VLAN concept :
		in ingressing of data use 802.3
		in egressing of data use dot1q  
		vlan isolate our network in layer 2
		with ip we can have isolation in layer 3 but get broadcasts
		vlan means virtual lan
		by default we use native vlan number 1

		vlan 1 is native and can change it to other vlan :
			interface range fast 0/1-02
			switchport trunk native vlan 100
	
		vlans number are like this :
			1 - 1024 >>> standard
			1 - 4095 >>> extended
			1002 - 1005 >>> frddi
	
			feddi use tocken ring technology
	
			interface fast 0/1
			switchport access vlan 10
			switchport mode access

	why some times we can't see interfaces in show vlan brief ?
		maybe interfaces get trunk 
		maybe our created vlan get delete

	show vlan brief
	show interface trunk

	it's better use no negotiation mode in trunk ports to disallow attackers
		interface fast 0/0
		switchport no-negotiation
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
metro ethernet :
	some times we need connect our routers in same organization in many cities, must buy metro ethernet service from cti, must make subinterface and tag packets, cti receive our packets on layer 2 and tag again on packets 
	they say that is q in q 
		interface gig0/0.1
		switchport mode dot1q-tunnel
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
vlan and mac acl:
	Access-Lists :
		access-list job is packet matching and execute a action on it , no firewalling or ips or idc or ....
			we have 2 type blocking :
				impelicete deny :
					deny any in end of each acls
	
				expelicit deny :
					write deny in sequences
	
			acls type:
				standard
					if block traffic in point a traffic transfer to a and block after a point
	
				extended
					we can add ports and .. it's flexible (name base)
	
			Protocol				Range
	
			Standard 				1–99 	and 1300–1999
	
			Extended 				100–199 and 2000–2699
			
			Ethernet-type code		200–299
			
			Ethernet-address		700–799
	
			must apply our acls on input direction on each interfaces (must be nearlly to destination for extended mode)
	
			access-list 1 deny host 192.168.1.100 (deny all traffics of this host)
	
			for assignment policies to router and directions of flow :
				interface serial 0/0/0
				ip access-group 1 out
	
				number 1 is acl number and out is drection of flow
	
			access-list 1 permit  192.168.1.0 0.0.0.255
			aaccess-list 1 deny host 192.168.1.100
	
			with this acl our host 1.100 get traffics because in line 1 we access to range of the host and it's get conflict.
			in acls we use wilde cad masks
	
			how insert between sequence numbers:
				ip access-list standard 10
				5 deny host 192.168.1.100
				11 permit 192.168.1.0 0.0.0.255
	
			some times we need bllock pin on devices but ned allow other ports be visible:
				access-list 100 permit tcp any 192.168.1.0 0.0.0.255 
	
				with thi line we say all tcp protocols will be visible from outside or internet on range inside 192.168.1.0/24
				but must set icmp deny for bank (dos attack)
				altho have impelicete deny in end of acl lines
	
				access-list 100 deny icmp any 192.168.1.0 0.0.0.255
	
			here we want show some port allowing and play with them :
				access-list 100 deny tcp host 192.168.1.11 host 192.168.1.12 eq 20,21 (just open port 20,21 if 1.11 wants connect to 1.12 )
	
	
			impelicete  deny is : access-list 100 deny ip any any
	
			for tcp connections need reply and must set established connections (syn-synack)
	
			ip access-list is much flexible for insert between some lines and editable
	
			we can access for one ip to access ssh to device
				run ssh on device
				access just one connection to line vty
	
				access-list 1 permit host 192.168.1.10
				line vty 0
				login local
				access-class 1 in (sequence number)
				or
				ip access-class vty-one-conn in (namebase)
	
			acls contains and analyz every thing except self produced data by device.
	
			we need summary some networks and subnets to add some ports :
				network : 10.1.1.100 > subnet : 255.255.255.252
				network : 10.1.1.101 > subnet : 255.255.255.252
				
				new subnets : 255.255.255.254
	
				10.1.1.100 /31 > subnet 255.255.255.254 > wild-card-mask : 0.0.0.1
	
				ip access-list extended 100
					permit tcp host 10.1.1.101 10.1.1.100 0.0.0.1 eq 80
					permit tcp host 10.1.1.101 10.1.1.100 0.0.0.1 eq 443
	
					deny ip  10.1.2.0 0.0.0.255 10.1.1.0 0.0.0.255
					permit ip  10.1.2.0 0.0.0.255 any
	
					permit udp host 8.8.8.8 eq 53 10.1.2.0 0.0.0.255 (permit port 53 in acl)
					
					deny ip any 10.1.2.0 0.0.0.255 log
	
			reload 10 (after device fully load , wait 10 minute then apply these configs)

	vlan acl :
		access-list 110 permit host 192.168.10.10 host 192.168.10.11
		access-list 110 permit host 192.168.10.11 host 192.168.10.10
		valn access-map blk10-11 10
		match ip add 110
		action drope
		exit
		valn access-map blk10-11 20
		action forward
		exit
		vlan filterblk10-11 vlan=list 1
	
	mac acl :
		mac access-list extended blk10-11 permit host (mac) host (mac)
		vlan access-map blk10-11 10
		match mac address blk10-11
		action drope 
		exit
		vlan access-map blk10-11 20
		action forward
		exit
		vlan filter blk10-11 vlan-list 1
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
private vlan :
	in datacenters or hosting providers we need isolation and need more security for entered users, behind of each switchport or switches we have many various services so must protect them
	we have two part in private vlan primary (router side (promiscuous)) and secondary (server or client side (host))
		secondary has two part :
			isolated 	(can't see even eachother)
			community 	(can see eachother)
	
	for run this option we must set vtp to transparent mode.
		vtp mode transparent 
		vlan 10
			private-vlan community
			exit
		vlan 20 
			private-van isolated
			exit
	
		vlan 200
			private-vlan primary
			private-vlan association 10,20
			exit
	
	set client side mode :
		interface range gig0/0-1
			switchport mode private-vlan host
			switchport private-vlan host-associat 200 10
		interface range gig0/2-3
			switchport mode private-vlan host
			switchport private-vlan host-associat 200 20
	
	set router side mode :
		interface gig0/0
			switchport mode private-vlan promiscuous
			switchport private-vlan mapping 200 10,20
	
	do show vlan private-vlan 
	
	we can run it on svi :
		interface vlan 100
		private-vlan mapping 200 10,20
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
etherchannel layer 3 load balance mode set :
	port-channel load-balance src-des-port
	do show etherchannel port-channel
	
	cisco 		: pagp (auto , desirable)
		maximum use 8 ports together
		by default is silent mode
		at the end of command must set no silent
	standard 	: lacp (active , passive)
		maximum use 16 pports together and make 8 port active and 8 ports standby
		decision maker (lower priority, default 32768)
		lower mac
	manual 		: on

	we have cost for ether channel
			fast is 19
			1 gig is 4
			10gig is 3
	
	interface range gig0/0-3
	shutdown (it's better turn off ports before ether channel)
	no switchport
	channel-group 1 mode on
	no shutdown
	
	do show ehterchannel summary
	
	we can set ip add to etherchannel layer 3 :
	interface portchannel 1
	ip add 192.168.1.1 /24

	in etherchannel if run spannigtree we can't find block port
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
redundancy in supervisor :
	in cisco we can ue 9 switches to stack them with 0/5 or 9 meter stack cable
	has master and memebers each config on master take effects on members 
	if master get fail between members get election with these states:
		priority
		better hardware
		configs
		uptime
		lower mac address

	modes :
		vss
			 4500 & 6500 series
			 active - standby

		rpr (router processor redundancy)
			if our sup 1 of switch 1 get fail the time of sup 2 upping is 1min for 4500 and 2-4 min for 6500

		rpr+ (router processor redundancy plus)
			just works on 6500 series and take 30 seconds to wake up

		sso (statefull switch over)
			6500 series > 0-3 seconds
			4500 series > 1 seconds

		nsf with sso
			instansly wake up no delay

	redundancy 
	mode sso

	do show redundancy state
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
redundancy modes for routers and switches:
	ether channel
	spanning-tree (hsrp)
	hsrp in routers
	stackwize

	layer 3 redundancy and layer 2 redundancy

	fhrp > first hop redundancy protocols (standard)
	hsrp > hot standby router protocol (cisco exclusive) (2 device)
	vrrp > virtual router redundancy protocol
	glbp > gateway load balancing protocol (cisco exclusive) (4 or more devices)

	for automatic failover must use hsrp:
		at maximum we can high availablity 2 device together
		we have vip (virtual ip address)
		has 3 mode (active - standby - listen)
		has 2 version (1 - 2)
			version 1 > 255 group
			version 2 > 4096 group
		has 1 real gateway

	between 2 router or switches wich one get active in hsrp?
		priority (higher is better)
			default priority is 100 and has range between 0-255
		biger ip address
		time of config
			first router or switche if get ready, be boss

			if config one of them it takes ownerships and if get down another one give ownership and nevere give back
	
	we have states in hsrp :
		disable
		init
		listen
		speak
		standy
		active

		interface gig 0/0 standby 10 ip address 192.168.1.1

		show standby brief

		standby numbers can be 0-4096		

		each 3 seconds sends hello packets together
		if 10 seconds wait and don't get hello packets wil get active in version 1 we have sconds unit but in version 2 we have mili seconds unit
		if change timer in one device must change timer values on another devices in hsrp game

		in vrrp and glnp we set on one device then learned to another devices

		we can change these times 
			standby 10 timers 1 3 

		two part of last section in mac in hsrp define standby group number

		in version 1 we have many  group, 2 pow 8 = 255 groups
		here we have virtual mac : 0100.5e00.0002 (all routers)
					 virtual ip : 224.0.0.2 (multicast all routers)
					 port : 1985 udp

				2 section of last part in mac address will be used for address and group number in hsrp
					  0000.0c07.ac0a
					  0a (hexa dec 10) is the group id in hsrp version 1

		in version 2 we have more security and more groups with these options :
			destination is not all routers
			hsrp ip address > 224.0.0.102 (hsrp)
			hsrp mac address > 0100.5e00.0066 (hsrp)
			support ipv6
			we can set timers in mseconds
			we can use 2 pow 12 groups in version 2
			3 section of last part in mac address will be used for address and group number in hsrp
			0000.0c07.a00a
			00a (hexa dec 10) is the group id in hsrp version 2

			standby version 2

		we can set authentication in hsrp:
			normal authentication:
					standby 20 authentication cisco123
			md5 authentication:
				standby 20 authentication md5 key-chain test

			set key-chain to hsrp :
				key chain test 
				key 1
				send life time == acceptlife-time
				key-string cisco
				exit
				key 2
				send life time == acceptlife-time
				key-string ocsic
				exit
				interface gig0/0
				standby 20 authentication md5 key-chain test

		we must set primary and secondary devices in hsrp :
			standby 10 priority 101

		if one device get down or fail we use our spares and if get back in circute we must give back active state to it
		this job will be able with preemt:
			interface gig0/0
			standby 10 preemt

		it's better have delay in fully load of device and preemting.
			interface g0/0
			standby 10 preemt delay 10

		in vrrp bydefault is active this option (preemt)

		in switch must set portfast on routers side and disable spanning-tree:
			spanning-tree portfast network

		for load-balance in hsrp :
			vlaning and priority and roas must be used
			mls can be used in hsrp and vrrp 
			for glbp must be 6500 serier switches
			must trunk all links on router side in switches

	vrrp :
		virtual router redundancy protocol
		has priority range between 1-254 default is 100
		priority 255 is vip (virtual ip) and ip address of dedicated interface
		hello interval is 1 second
		by default use preemt command
		doesn't have track
		if enable vrrp on cisco devices we can aggregate vrrp and track option
		we can run it on 4500 and 6500 (sup 2, sup 720)

		here we set a diffrent ip address on dedicated interface and vip of vrrp (use priority between 1-254):
			interface gig0/0
			no sh
			ip add 192.168.1.2 255.255.255.0
			vrrp 1 ip 192.168.1.1
		
		show vrrp brief (it shows priority 0-254)
		

		here we set a same ip address on dedicated interface and vip of vrrp (use priority 255)(we just can do this in vrrp no in hsrp):
			interface gig0/0
			ip add 192.168.1.1 255.255.255.0
			vrrp 1 ip 192.168.1.1
		
		show vrrp brief (it shows priority 255)
	
	sla (service lavel agreement) and track :
		ip sla 1تجمیع
		icmp-echo 100.100.100.2 source interface gig0/0
		frequency 5
		exi
		ip sla schedule 1 start-time now life forever
		track 1 ip sla 1
		exi
		do sh ip sla statics
		do sh track

	track :
		track 1 interface gig 0/1 ip routing
		exit
		interface gig0/0
		standby 20 track 1 decrement 10
		standby 20 preemt delay reload 5
		standby 20 priority 105

	glbp :
		gateway load balancing protocol 
		cisco exclusive 
		4 or more devices
			4 used together and if used more the others goes to backup
		this option eliminate all restrictions on hsrp and vrrp
		support on 6500 (sup 2, sup 720)
		has priority range between 0-255 default is 100
		bigger ip is important and get avg (tie breaker)
		if enable preemt the boss get active and determine
		we have one virtual ip and many virtual mac
		the boss or master task is distribute vmacs between clients
		hello times interval are 3 seconds
		avg is active virtual gateway (boss + avf (if all devices get fail can work in avf mode))
		av active virtual forwarder
		if set timers on avg, advertise it to another devices
			in avg :
				glbp 1 timers 6 msec hell 10 msec hold 
		we set priority only on avg and backups
		arp reply to clients handle by avg
		preemt is not enable
		if our avg without preemt get fail our topology get fail
		works with track

		avf : 0007.64xx.xxyy
			xx.xx > number of glbp and group number, avf number
			yy > the mac number or router number, 8bit avf

		glbp load-balance methods :
			round robin 	(default)(one by one)
			host-depends 	(when goes one way just use it)
			weighted 		(set priority or weight to devices)
				by default is 100

		what is redirect=10 minute and timeout=4 hours in glbp?
				if one avf get fail and was on host-depends mode, says to biggest ip or backup that you are avf (numebr x) also you are avf (number x), use 2 vmacs together till 10 minutes after 10 minute distribute new assignment of macs and gateways with timeout option till 4 hours to clients
					timeout use hour unit
					redirect use minute unit

		set glbp to devices :
			sw1 and other switches :
				inetrface vlan 1
				no sh
				ip add 192.168.1.2 255.255.255.0
				glbp 1 ip 192.168.1.1
				
				interface vlan 1
				glbp 1 load-balance round-robin
				glbp 1 load-balance host-depends
				glbp 1 load-balance weighted

		show glbp brief	(show brief all state of glbp related devices)
		show glbp (detailes in glbp)

	set sla and track on glbp :
		avg sw1 :
			glbp 1 weightining 150 lower 140 upper 145
				with track we say our value or weight be on 150 on default value if get one link fail -5 assign to weightining and if get 2 link fail -10 assign to weightining automatic, our avg stay in avg mode and don't change style to avf, and others in avf mode be internet gateway
				we must set threshold
					(lower 140 upper 145) if take lower than 141 you are not avf
			
			interface gig0/2
			no shutdown
			no switchport
			ip add 200.200.200.1 255.255.255.0

			interface gig0/1 
			no shutdown
			no switchport
			ip add 100.100.100.1 255.255.255.0
			
			ip sla 1
			icmp-echo 100.100.100.2 source-interface gig0/1
			frequency 5
			exit
			ip sla 2
			icmp-echo 200.200.200.2 source-interface gig0/2
			frequency 5
			exit
			
			ip sla schedu 1 start-time now	life forever
			ip sla schedu 2 start-time now	life forever
			
			track 1 ip sla 1 
			delay down 5 up 5 (use delay)
			exit
			track 2 ip sla 2 
			delay down 5 up 5
			exit
			
			interface vlan 1
			glbp weightining track 1 decrement 5 (if in track we sense failing and decrement 5 level of weightining)
			glbp weightining track 2 decrement 5

		it's better set our sla on google.com (8.8.8.8) 
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
port security:
		protect our mac address table in switch

		interface fast 0/1
		switchport mode access
		switchport access vlan 10
		switchport port-security (enable the option)
		switchport port-security violation (if get blank=shut the port if get protected=don't get log and block if get restrict=get log + block interface)
		switchport port-security mac-sticky (cache macs like static mac not 5min role)
		switchport port-security maximum 3 (maximum macs behinde this port that can transfer are 3 not more if get 4th mac make this port block)

		switchport port-security aging time 5 (by default is 0 minuts infinit time changeable if shut and no shut port)
		switchport port-security aging type inactivity (absolute (if send or if be idle write mac), inactivity (if sends traffic write and cache mac, if not cleared after 5 min or minuts))

			if law violated, goes to error disable mode
				error disable :
					errdisable recovery cause psecurity-violation
					errdisable recovery interval 30
					show errdisable recovery

		clear port-security all
		clear mac addresss-table dynamic
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
AAA :
	authentication	(who want access)
	accounting	(monitor the traffics till specific hour)	
	authorization (access levels)

	this is a concept not a protocol 
	use some protocol inside it like tacacs+ and radius

	radius :
		standard verson of aaa protocol
		just encrypt passwords

	tacacs+ :
		cisco exclusive
		encrypt all data (pass + username + commands on telnet )

	aaa new-model
	tacacs server host 192.168.1.1 key cisco
	aaa authentication logging default group tacacs+ local

	default : is list of users and passwords 
	group : means have many servers
	local : means if our aaa server get down or fail the authentication can be transfer to local user and passwords databse on devices

	username shayan secret 123
	enable secret 123

	line vty 0
	login authentication mytelnet

	aaa new-model
	tacacs server host 192.168.1.1 key cisco
	aaa authentication logging mytelnet group tacacs+ local

	mytelnet : can define in line vty

	in ccna security we have enable view for users on cisco devices and can define access levels

	ACS :
		user : acsadmin
		password: P@ssw0rd

		we should set ip address on router in same range acs

		http server enable
		http 0.0.0.0 0.0.0.0 mgmt
		username admin pass 123 previledge 15
		ssh encryption aes128-sha1 rc4-md5

		802.1x > whit 802.1x, when plug the cable to client, bypass all protocols except eapol, we known this protocol as dynamic vlan, works on radius, must enable on windows
			eapol is a protocol goes to aaa server and take our authentication from server then take dhcp options

			we should goes to services.msc and (wired auto config), set on automatic, start
			make dot1x enable on windows
			ncpa.cpl > ether0 > properties > authentication (header) > uncheck 2 check boxs on bottom page (remember and fail) > in setting uncheck verify... > in setting set mschap v2 and click on configure button cheched automatic option > go back to authentication (header) and goes to additional > set specify option on user authentication

		show application status acs (if show us all rtasks running state we can use it)
		application start acs
		ip name-server 192.168.168.16 (active directory ip add, ntp, dns)
		ntp server 192.168.168.16

		authentication port :1645
		accounting port : 1646
		
		ACS server :
			Creat devices 				Network Resources > Network Devices and AAA Clients
			Creat users 				Users and Identity Stores > Internal Identity Stores > Users
			Enable PEAP Session 		System Administration > Configuration > Global System Options > PEAP Settings
			Download Certificates   	System Administration > Configuration > Local Server Certificates > Local Certificates
	
		ACS client :
			sw1 :
				aaa new-model
				radius server acs
				address ipv4 192.168.244.144 auth-port 1645 acct-port 1646(acs ip address)
				key cisco123
				exit
				aaa authentication dot1x default group radius (maeasured by acs and dot1x)
				dot1x system-auth-control (all interfaces goes to dot1x)
				interface GigabitEthernet 0/1
				switchport host
				authentication port-control auto (authentication checked with dot1x)
				dot1x pae authenticator (authentication checked with dot1x)
				interface gig 0/0
				authentication host-mode multi-host (behinde of interface we have many users)
				authentication violation restrict (if unauthorize user comes what sould we do)
				authentication event fail action authorize vlan 200 (if can not recognize take member to vlan 200)
				authentication event no-response action authorize vlan 300 (old nic doesn't support dot1x so must use this command to set unauthorize user vlan)
				authentication timer restart 30 ()
				dot1x timeout ?	

			in priviledge :
				test aaa group radius gholi 123456 new-code (check connections)
				show authentication sessions
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
dhcp :
	dhcp has 255 option but one of them is ip assignment, assign setting of layer 3,4,5 from tcp 
	has 4 steps :
		dora :
			discovery
			offer
			request
			acknowledge

	all steps are broadcast
	has some pools 
	works on port 68 udp client and 67 tcp server side
	in offering prt of dhcp we must make sure that do'nt have duplicate ip address so  systems check their arp tables if didn't exist get ip if not behave another way
	in mange switches must define trusted dhcp server port and untrusted ports

	define dhcp pools in cisco switches or routers :
		ip dhcp pool mypool
		network 192.168.1.0 255.255.255.0
		default-router 192.168.1.1
		ip dhcp exclude address 192.168.1.1 192.168.1.10

		show ip dhcp binding

		ip helper-address 192.168.1.1 (define dhcp server in router and interface + sub-interfaces)

	dhcp relay agent :
		the route don't send broadcast packets to another networks or subnets (all dhcp packets are broadcast)
		soloution of this problem is dhcp relay agent and definantion this option to router
		this option convert broadcast packets to multicastthen access them to cross away from router
		this is option 82 means relay agent

	DHCP snooping :
		trusted port : 		have permission send and recive dhcp packets 
		untrusted ports :	have'nt permission send and recive dhcp packets 
		
		in dhcp must define trusted ports for prevention other users infiltrate :
			first define all ports as untrusted port 
				ip dhcp snooping 
				ip dhcp snooping vlan 2
			then define trusted port :
				interface gig0/0
				ip dhcp snooping trust

			sw1 :
				dhcp snooping
				ip dhcp snooping vlan 1-10
				interface g0/0 (routers our dhcp servers side) 
				ip dhcp snooping trust
				interface gig0/1-3 (other ports)
				ip dhcp snooping limit rate 5 (client just can request 5 time for dhcp option)
				
				we have problem here becuase some new clients can't take ip address from dhcp server (option 82) :
				(global) no ip dhcp snooping inoformation option 

			the last way is port-security

		show ip dhcp snooping database
		
		which mac take wich ip address :
			show ip dhcp binding 
	
		we can set rate limit on devices that get dhcp server :
			ip dhcp snooping limit rate 5
	
		dhcp starving and man in the middle attack are dhcp attacks :
			dynamic arp inspection (before all of them we must set dhcp snooping beacuse works with database of dhcp-snooping)(when arp was designing security doesn't matter) : 
				ip arp inspection vlan 1-100
				interface gig0/0
				ip arp inspection trust
					
		with this option just users get ip from dhcp can access our network for manual ip setting :
			arp access-list arpacl
			permit ip host 192.168.1.100 mac host -------
			exit
			ip arp inspection filter arpacl vlan 1 static
			
		ip source gaurd (before all of them we must set dhcp snooping) : 
			some times set ip of bmi.ir then ping 30000 clients and the replys goes to bmi.ir and make dos attack for prevention must:
				interfce range gig 0/0-3
				ip verify source port-security (port-security option checks ip and mac together)
				
			manual mode ip source gaurd :
			ip source binding ------ vlan vlan-id 1 interface gig0/0
				
			show ip verify source interface gig0/0
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
power over ethernet (poe):
	has many modes:
		cisco inline power  	7w
		standard 802.3af 		15.4w
		standard 802.3at 		30w (poe+)
		cisco universal poe 	60w
			cisco exclusive
			used for access points

	we have poe attack

	interface gig0/0
	power inline auto (by cdp)
		send a little voltage with cdp to detect device and reply on cdp the value 
	power inline never (off)

	show power inline 
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
span :
	local span :
		monitor session 1 source interface gig0/0 both
		monitor session 1 destination interface gig 0/2
	
	remote span (must set trunk and remove transmissions vlan in vtp prunning) : 
		sw 1 :
			vlan 200
			remote-span
			monitor session 1 source gig0/1
			monitor session 1 destination remote vlan 200
		
		sw 2 :
			vlan 200
			remote-span
			monitor session 1 source remote vlan 200
			monitor session 1 destination gig0/1
		
		switchport trunk prunning vlan remove 200
	
	erspan (just for use on 6500 series and also on gre tunnel) :
		sender :
			monitor session 1 type erspan-source
			source interface gig0/0
			no sh
			destination
			erspan-id 101
			ip add 10.10.10.1
			origin ip add 172.16.0.1
		
		reciver :
			monitor session 2 type erspan-destination
			destination interface gig0/0
			no sh
			source
			erspan-id 101
			ip add 10.10.10.1
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
storm braodcast control :
	interface gig0/0
	storm-control broadcast level 70 - 80
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
ntp :
	ntp types :
	server
	peer (sync with erver in interval timers)
	client
	
	show ntp status 
	show ntp association
	ntp server 192.168.10.10
	ntp master 192.168.10.1
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
switches config :
	en
	conf t
	no cdp enabel
	vtp pruning
	hostname sw4
	username shayan sec 123
	enab sec 123
	ip domain-name test.com
	ip default-gateway 192.168.10.1
	spa mo rapid
	vtp mo clie
	
	int range fast 0/0-25
	shutdown
	
	int fas 0/1
	no shutdown
	ip dhcp snooping limit rate 5
	storm-controll broadcast level 80
	sw acc vl 10
	sw mod acc
	sw po 
	sw po vio re
	sw po mac sti
	sw po max 3
	sw noego
	spa bpdu-gaurd ena
	no cdp ena
	span port
	
	int rang fas 0/2-3
	no shutdown
	ip dhcp snooping limit rate 5
	storm-controll broadcast level 80
	sw acc vl 30
	sw mod acc
	sw port
	sw po vio re
	sw po mac sti
	sw po max 3
	sw noego
	spa bpdu-gaurd ena
	no cdp ena
	exi
	
	int gig 0/2
	sw mode tr
	ip dhcp snooping trust
	exit
	
	ip dhcp snooping 
	ip dhcp snooping vlan 1-10
	no ip dhcp snooping information option
	
	monitor session 1 source fas 0/1-3 both
	monitor session 1 des fas 0/24 
	vlan 69
	int gig 0/1
	sw mo tr
	monitor session 1 source fas 0/1 both
	monitor session 1 des remote vlan 69 reflector-port gig 0/1
	vlan 69
	int gig 0/1
	sw mo tr
	monitor session 1 source remote vlan 69
	monitor session 1 des int fas 0/1
	
	int vlan 10
	no sh
	ip add 192.168.10.4 255.255.255.0
	
	int vlan 1
	no sh
	ip add 192.168.1.10 255.255.255.0
	exit
	
	line vty 0 4
	login loc 
	tr in ss
	tr out no
	no exe
	exit
	ip ssh ver 2
	
	aaa new-model
	tacacs-server 192.168.1.100 key sheyn2142
	aaa authenticat login default group tacacs+ local
	aaa authenticat login mytelnet group tacacs+ local
	
	line vty 0
	login authen mytelnet
	
	archive
	tftp://192.168.4.100/$h
	writememory
	time-periodic
	 
	logging host 192.168.1.100
	service timestamps log datetime msec 
	 
	crypto key gen rsa mod 1024
	do wr
	
	!!!!!!!!!! rommon commands when device has no ios !!!!!!!!!!
					ip-address=192.168.1.1
					ip-subnet-mask=255.255.255.0
					default-gateway=192.168.1.100
					tftp-server= 192.168.40.100
					tftp-file=iosname
					tftp dnld=yes
	!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
mls config :
	en
	conf t
	no cdp enabel
	vtp pruning
	hostname mlscore1
	ip domain-name test.com
	username shayan sec 123
	enab sec 123
	ip default-gateway 192.168.20.1
	span mod rap
	vtp mo ser
	vtp domain test.com
	vlan 10
	vlan 20
	vlan 30
	
	spa vlan1,10 root primary
	span vlan20 root secondary
	
	int vlan 10
	no sh 
	ip add 192.168.10.2 255.255.255.0
	ip helper-add 192.168.30.30
	standby 10 ip  192.168.10.1
	standby 10 pree
	standby 10 timer 1 3
	standby 10 pri 101
	
	int vlan 20
	no sh
	ip add 192.168.20.2 255.255.255.0
	ip helper-add 192.168.30.30
	standby 20 ip  192.168.20.1
	standby 20 pree
	standby 20 timer 1 3
	standby 20 pri 100
	
	int vlan 30
	no sh 
	ip add 192.168.30.2 255.255.255.0
	ip helper-add 192.168.30.30
	standby 30 ip  192.168.30.1
	standby 30 pree
	standby 30 timer 1 3
	standby 30 pri 100
	
	int ran gig1/0/1-4
	sw tr en do 
	sw mo tr
	sw no neg
	channel-gr 1 mode au
	int port-chan 1
	sw mo tr
	do sh etherchan sum
	exit
	
	monitor session 1 source fas 0/1-3 both
	monitor session 1 des fas 0/24 
	
	line vty 0 4
	login loc
	tr in ss
	tr ou no
	no exe
	ip ssh ver 2
	
	aaa new-model
	tacacs-server 192.168.1.100 key sheyn2142
	aaa authenticat login default group tacacs+ local
	aaa authenticat login mytelnet group tacacs+ local
	
	line vty 0
	login authen mytelnet
	
	archive
	tftp://192.168.4.100/$h
	writememory
	time-periodic
	
	logging host 192.168.1.100
	service timestamps log datetime msec 
	
	crypt key gen rsa mod 1 024
	do wr
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
