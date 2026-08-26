ISIS (intermediate system to intermediate system) :
	isis used in first day of networking platforms and isp
	isis base topology was osi model
	devices has one global and unique address to communicate with each other
	in tcpip each interface use diffrent ip and address and connection-less mechanism + range + addressing in tcp we do not care about the destination
	but in osi and isis we use clnp (connection less network protocol) + x.25 (connection oriented)

	x.25 in tcpip converted to wan layer2 and mp wan switching (before these technologies used framerelay and atm)

	in isis we have host(end system (es)) and router(intermediate system (is))
	between es and is we use es-is protocol that works like arp

	in old models we must install some tese protocols on systems to make connection between them

	routed protocol was clnp and routing protocol was isis

	isis has version :
		rfc 10589
		dual isis > newest version of isis (named integarted isis or i-isis)
		in 2008 use dual tack mode (ipv6 + ipv4)

	in isis each interface use another term circute was alternate name
	circutes use seprated and diffrent mac address but use same ip (ip means id or nsap address and clnp address)

	nsap (network service access port)

	in isis for autonomous system (as) use domain
		domain types :
			subdomain
			inter-domain
			intra-domain
	
			for inter-domain routing use idrp (inter domain routing protocol) or iso-igrp

	each router in isis use rid like ospf and detected by nsap address known as identifier
	
	nsap > identifier : idp (initial domain part) (outside the organization) + dsp (domain specific part) (inside the organization)
	
	idp contain : afi (authority format identifier) + idi (initial domain identifier)
		set format and codes
	
	dsp contain : hodsp (high order bit) + system-id (define system in organization) + sel (network selector)(a code use inside organization or application works on clnp)
	
	---idp---  +  ----------dsp----------
	afi + idi  +  hodsp + system-id + sel
	
	routing domain > idp (afi+idi)
	some times we don't attend to idi
	
	example : 80.0001.6666.6666.6666.80
		
		idp 				dsp
		----	--------------------------------
		80  	0001 	6666.6666.6666 		80
		afi 	hodsp		system-id		sel
	
	afi can be like country code in phone numbers
	
	for private mode use 49 in afi and idp
		49.0001.4444.4444.4444.80
	
	in present day most use 00 for nsel or sel part of nspa beacuase we don't have any clnp application 
	
	nsap without nsel or sel get new format like net (network entry titel)
		49.0001.4444.4444.4444.00
	
	conf t
	router isis
	net 49.0001.4444.4444.4444.00
	
	isis use ad 115 for routing 
	
	connection modes and domains :
			es to es in same domain > intra-link > level 0
			intra area 		> level 1
			inter area 		> level 2
			inter domain 	> level 3
	
								    domain 1
		-----------------------------------------------------------						
				 area 1 							area 2
		--------------------------		  	   --------------------
		es1---sw1---is1-1---is1-2-------is-----is2-1---is2-2---es2 
			|		  	  | 			|			 |	
			es0		     es3 			|			es4
										|
										|
										|
										|
									domian 2
		-----------------------------------------------------------						
				 area 1 							area 2
		--------------------------		  	   --------------------
		es5---sw1---is1-1---is1-2-------is-----is2-1---is2-2---es8 
			|		  	  | 						 |	
			es6		     es7 						es9

		------area 1---             ---area2-------
		A---R1----- 					-----R4---C
					| R3-----R7-----R6 |
		B---R2----- 					-----R5---D
		
		link state and spf algorythem
		
		in osi use clnp and nsap address
			r1 : 49.0001.1111.1111.1111.00
			r2 : 49.0001.2222.2222.2222.00
			r3 : 49.0001.3333.3333.3333.00
		
			r4 : 49.0002.4444.4444.4444.00
			r5 : 49.0002.5555.5555.5555.00
			r6 : 49.0002.6666.6666.6666.00
		
			routers in isis use modes :
				level 1 : 
					1 - 2 - 4 -5 
		
				level 1-2 (abr in ospf) :
					3 - 6
		
				level 2 only (asbr in ospf) : 
					7
		
		level 1 :
			create self area map
				r1 ls :
					network a connected to :
						49.0001.1111.1111.1111.00 (r1 nsap)
						49.0001.3333.3333.3333.00 (r3 nsap)
		
				r2 ls :
					network b connected to :
						49.0001.2222.2222.2222.00 (r2 nsap)
						49.0001.3333.3333.3333.00 (r3 nsap)
		
				r3 ls :
					network e and f are addresses connect to r1 and r2 :
						49.0001.1111.1111.1111.00 (r1 nsap)
						49.0001.2222.2222.2222.00 (r2 nsap
						49.0001.3333.3333.3333.00 (r3 nsap)
		
			in ospf each interface on router could be join area means router could join many area but in isis we have single area joining
			area in isis has type
			area borders set with links 
			r3 use attached bit (att) to define this router is level 1-2 (abr)
			attached bit define to level 1 routers i'm default gateway
			in level 1-2 or r3 and r6 we have all network map 
			we can start area number from any number
		
			isis use silent mode for logs
	
	router isis
	net 49.0001.3333.3333.3333.00
	log-adjencency-change all
	is-type level 1-2 (default is this)	
	interface ether 0/0
	ip router isis
	isis circute-type level 1-2 
		if seet level in level 1 or level 2 only don't need set this because detect automatically
		if set level 1-2 and our interface connect to router that is level 1 must define circute-type on level 1
	
	router isis 5 
		5 is process id like ospf
	
	r1 and r2 use same config in leveling :
		r1 : 
			router isis
			net 49.0001.1111.1111.1111.00
			is-tpe level 1
			log-adjencency-change all
			int fas 0/0
			ip router isis
			isis circute-type level 1
			
		r2 : 
			router isis
			net 49.0001.2222.2222.2222.00
			is-tpe level 1
			log-adjencency-change all
			int fas 0/0
			ip router isis
			isis circute-type level 1
	
	r3 : 
		router isis
		net 49.0001.3333.3333.3333.00
		is-tpe level 1-2
		log-adjencency-change all
		int fas 0/0
		ip router isis
		isis circute-type level 1-2
	
	
	r7 : 
		router isis
		net 49.0000.7777.7777.7777.00
		is-tpe level-2 only
		log-adjencency-change all
		int fas 0/0
		ip router isis
		isis circute-type level-2 only	
	
	for loopback interfaces must set passive interface to make hlaf neighboring
		router isis
		passive interface loop 0
	
	sh router isis
	metric of isis is 10 per interface

	neighboring and database :
		sh isis neighbor details
		sh isis database
		sh ip route isis
		sh isis hostname

		in isis :
			hello interval 	> 10 seconds
			hold interval	> 3 x hello

			if we didn't catch good hello neighboring will be reject

		in isis we have broadcast and multipoint mode for links
			hello > 3 seconds
			hold  > 10 seconds

		we have dis (designated is)

	circute-id
		bordering in isis use circute-id
		we have random	circute-id
		in ehternet we have format :
			system id + identifier number

	in tcpip 
		interface use mac 

	in osi
		interface use snap (subnetwork point of attachment)

	in each multiaccess network we have dis (designated is)
		in dis mode we send hello each 3 seconds and wait 10 second other use 20 seconds

	sh isis topology level-1
	sh isis topology level-2

	sh isis database level-1
	sh isis database level-1 details
		show values in router r2 and r3

	sh isis database level-2
		in this show we see 5 router on level 2 instead of 3 router

	sh isis database level-2 details
		show connected networks on level 1 and level 2

		*in area we have topology and database but in inter-area or out side of area we just have database*

	if change ip in network don't need run spf algorythem but our interface change must run spf beacuase isis works on circutes

	sh isis database
		lsp checksum
		lsp sequence

	sh isis database details
		network layer protocol id (nlpid) > transfer wich type of data
		0xcc > ipv4
		network address + hostnames
		use area id

	if see * in isis shows means this roouter use this parameters

	in level 1 routers if > sh ip router isis
		can't see default route

	hello :
		isis hello (iih)
		snap (subnetwork point of attachment)
			layer 2 
			layer 3

		isis send data like train wagon

		isis hello header
			information
				hello
				data

		code lentgh value (clv)
		type lentgh value (tlv)
			means if see capability of features can be use if not use normal mode

			tlv 132 > ip address

			for neighboring :
				tlv 1 > area address
				length > 4 byte
				value > 49.0001

		states transfer with tlv
			like authentication > tlv 10

		padding tlv > 1500 byte must be use for mtu and neighboring
			with padding make correction in neighboring mechanism
			length 255 type 8

		level 1 hello :
			hello packet + link state pdu + complete sequence number pdu (csnp (like dbd in ospf))
			partial sequence number pdu (psnp (like lsu and lsr and lsack in ospf))

			in level 1 must be same area and use same mtu and same level to make neighboring state
			if set authentication must pass
			use unique system-id

		level 2 :
			we have same state for neighboring but area id is not important

	do clear isis *

	authentication :
		clear text
			use for hello (iih)

		hash md5 
			jus use for lsp and csnp and pspnp
			no iih

		int fast 0/0
		isi authentication mode cisco123 level-1
		isi authentication mode md5 level-2

		key-chain r1key
		key1
		key-string cisco

		isis authentication key-chain r1key

		if be same our string can use isis

		isis authentication send-only (just send authentication not get ack because use many loads)

		debug isis adjencency-packet
		debug isis authentication information
	
		sh clns interface fast 0/0
		sh clns is-neghibor
	
		lsp authentication :
			lsp 
			csnp
			psnp
			
			active for each process
			
			old mode :
				router isis
				area-password 123 (this is level 1)
				domain-password 123 (this is level 2)
			
			neighborship is ok but can't send lsp because lsp authentication didn't pass
			
			new version :
				router isis
				authentication mdoe md5 level-2
				authentication key-chain r1key level-2
				sh key-chain

			domain-password x authentication cnp snp

			isis authentication send-only
			isis authentication validate (make ack for password)(better use this after send only and change all configs then set to this)

	metric :
		default metric
		delay metric
		expense metric
		error metric 

		lowest is better
		in spf use default mmetric 
			for passive interface use 0
			for non-passive use 10

		for serial 1.55mbps and ethernet 100mbps have loadbalance in isis beacuase use value 10 in metric so :
			interface serial 0/0/0
			isis metric 6476

			but can't use above 63 in default way
			type of metrics in isis
				wide  > 20 bit
				narrow > 6 bit

			router isis
			metric-style wide
			metric-style transition > when we have load in network can works on wide and narrow then change and transfer to wide

			sh isis database details
				isextended means wide metric

			if need set automatic worst metric in serial must :
				int serial 0/0/0
				isis metric maximum

					if enter manualy > maximum value be 16777214
					in this command  > get 16777215

	lsp concept :
		with nsap we can make name resoloution mechanism
		in edge router we use same values in level 1 but in level 1-2 we have attached bit > 1
		means one hand is in level 2 only network
		like totaly sub area or totaly nssa in ospf
		level 2 database contain 
			49.0001.1111.1111.1111.00
			1.1.1.1 metric=0
			2.2.2.2 metric=10
			3.3.3.3 metric=10
			192.168.12.0/24 metric=10
			192.168.13.0/24 metric=10
			192.168.14.0/24 metric=10

			49.0001.1111.1111.1111.00 > if get resoloution > R1.00-00

		we have psudonode id 
			in point to point network we have single lsp for each router
			but in multiaccess and ethernet we have 2 lsp for one router
			another routers send one lsp
			reason of this additional lsp is designated is or dis
			multiaccess links :
				one lsp for original lsp
				one lsp for psudonode 

				format view : R4.01-00

		we have fragmentation mechanism in lsp transmission and applicated in large scale 

		for internet access must use default route from asbr or abr so must advertise it

		router isis
		default infrmation originate

		this command add some things on abr or asbr router like r5 and r4 > 0.0.0.0 metric=0

		with this default route advertisement even with no access to the Internet it will receive all the packets so better use this method :

		ip prefix-list a permit 192.168.49.0/24 (better set isp edge ip)
		route-map aa per 10
		match ip address prefix-list a

		router isis
		default infrmation originate route-map aa

		in this condition if r4  and r5 has access to isp can address all clients and routers to 0.0.0.0

		another default route advertisement :
			router isis
			redistribute statics (redistribute static routes to network)

		flags on lsp :
			p bit
				partition bit like virtual link in ospf
			ol bit
				ue when router have too much load
				if set on 1 don't be computed in spf algorythem
				if our router be only way to acccess some routes and ol bit were 1 must by pass this value and forward traffics
			attached bit
				use for level 2 only

		we have sequence number like ospf
			each lsp has sequence
			0x00000000 (csnp)(like dbd in ospf)
			
			first lsp > 0x00000001 
	
			psnp (partial snp)(can use for requesting new updates or data)
			ls ack
			ls request

		in isis we have age like ospf
		after each hop increase like ospf
		in ospf each 30 minutes update all lsa overally in isis we have 15 minutes
		affter this sequence number growth and all routers will be forced to coordinate and update

		lsp hold time is 20 minutes (after this time purge from rib)
		this is diffrent from hello in isis
		after lsp goes to second and step 0 after 60 secnd routers in cisco clear from rib cause is synchronization (means zero age life time and in cisco use 20 minutes)

	ecmp :
		router isis
		maximum-path 1-32(4 is default)
		sh ip protocols

		with same metric can make load balance

		link states in area can't be filter or summary
		in level 1-2 can summary and aggregate then send it to level 2 only
		router isis
		summary-address 192.168.1.0 255.255.255.0 (with this send to level 2)
		summary-address 192.168.1.0 255.255.255.0 level 1
		summary-address 192.168.1.0 255.255.255.0 level 2

		now without computing interface metric on summary can be use and use lowest metric to advertise
		after summarization we have null 0 in rib
		if loop backs get trouble we can add null 0 on it

		can advertise summary with diffrent metric
			router isis
			summary-address 192.168.1.0 255.255.255.0 metric 50

		router abr or level 1-2 is summarizing is or device

		sh isis database

	route leakage :
		if happen suboptimal routing we can by pass some default routes or load balance
		same time advertisement attached-bit we can leak some routes and address
		from level 2 only we can't leak or send summary to level 1
		maybe use longest way to access some address so make suboptimal routing

		access-list 100 permit ip host 1.2.3.4 host 255.255.255.255
		router isis
		redistribute isis ipv4 level-2 into level-1 distribute-list 100

		ip prefix-list a per 4.4.4.4/32
		route-map aa per 10
		match ip address prefix-list a
		router isis
		redistribute isis ipv4 level-2 into level-1 route-map aa

		we can send some level-2 address in summary mode in level 1 router
		summary-address 1.2.3.0 255.255.255.0 level-1

	advance isis config :
		router isis
		advertise passive-only 
			advertise passive interfaces only and if recieve address just use special address and ips that access from special way and paths

		sh isis database levl-1 r1.00-00

		in multiaccess network for some networks see one router cconnect to a is and connect to another attached router visibel from 2 path and with dis we can manage advertising routes like directly connection and send without an intermediary operation

		psudonode use metric 0 for virtualization mode
		send a lsp to some routers :
			is r5.00-00 metric 0
			is r6.00-00 metric 0
			is r7.00-00 metric 0

			psudonode is manager for isis and make multiaccess network to point-to-point mode

			always dis and isis use ethernet and multiaccess + ptp 

		to bypass dis :
			interface fa1/0
			isis network point-to-point

			router isis
			shutdown (restart)

	hello and dead :
		dis help to multiaccess make synchronization in rapid mode
		if see higher sequence number in non-dis router our dis advertise this updates and flood it
		dis each 10 seconds advertise csnp sequence
		dis in isis is not competitive and unlike ospf in isis we have layer 2 selection , multicast mac address :
			level 1 > 0180.c200.0014
			level 2 > 0180.c200.0015

		dis selection :
			1-higher priority in interface priority
				in ptp like serial interfaces we don't use dis
				if be broadcast (can set 0-127 default is 64 bigger is better)
					int fas0/0
					isis priority 100 level-1 
					isis priority 100 level-2

			2-higher snpa (subnetwork point attach)
				higher mac
				mtu in tcpip

			3-framerelay and atm
				neighbor use same snpa
				higher sid

		in isis if see better mac can change dis unlike ospf dr use fix state and can't be preemtive except change it manual

		hello timers :
			dis
				send > 3 sec 
				hold > 10 sec
			non-dis
				send > 10 sec
				hold > 30 sec

			int fast 0/0
			isis hello-interval 5
			isis hello-multiplier 3 (means 3*5)

			if set some timers for non-dis, our dis use 1/3 of timing 
				isis hello-int minimal (send faster than normal state)
				isis hello-multiplier 4

		in hello and neighboring use hello padding if :
			no isis hello padding point-to-point

			get disable so make hello packets with padding of 0 value
			better set mtu in manual mode and 2000 byte
			if use padding cheeck the path mtu be comfortable
			mtu use for neighboring

			int fast 0/0
			no isis hello padding
				works on interface mtu 
				if our mtu was correct set this command 
				but routers make padding in hello so not recommended to apply 
				first steps make padding to use 1500 byte if were 1500 mtu or default value in inerface don't make padding in hello mechanism
				try many times many diffrent values of mtu
				step 1 > 1800 byte
				step 2 > 1500 byte 
				....

			no isis hello padding always
				don't use padding mechanism
				don't use consume in vain our bandwidth

	3waay handshake :
		if some routers could not reply requests we had half neighboship so must use 3way handshake

		states:
			down 
				without hello
				first step
			initial
				after send first hello goes to this state
				sysid + adjencency state tlv
				router number 2 must reply 
					initial router number one and packet type (sysid + adjencency state tlv)
			up

		circute-id > sh clns ehter 0/0
			local circute
			0x100

			in first day use 1 byte or 2*4bit > 1st hex value + 2nd hex value

			this is hello with 3way hand shake in cisco
			in ietf use 32 bit fo 3 wya hand shake hello and interface count

			interface fast 0/0
			isis three-way-handshake cisco
			isis three-way-handshake ietf

			why cisco send 0x100? in transfer use 0?
				in broadcast and point-to-point mode starts from 0
				so use point-to-point from 256 to 0 and convert it to hoted decimal > 0x100 > 256 in cisco mode
				inside router make this

			in fiber optic links we use smart mechanism in ietf unlike cisco 
			ietf use interface id (2^32) + hello
			cisco just use hello

			cuase is same system id maybe get confilict

			correct neighborship mechanism and attributes :
				local neighbor system id 
				circute-id extended
				neighbor interface id extended

		in dis each 10 sec make synchronization for csnp psnp
		interface fast 0/0
		isis csnp-interval 10

	IPV6:
		spf run in seperated and database > prc (partial route calculation)
		spf process > ipv4 + ipv6 + topology (single topology \ multi topology)

		single mode use ipv6 in all paths
		must config wide metric
		also active ipv6 unicast routing

		ipv6 router isis

		network layer protocol id (nlpid) :
			ipv4 > 0xcc
			ipv6 > 0x8e

		router isis
		address-family ipv6
		multi-topology transit (link ipv4 and ipv6)

		better set on transit then set on ipv6

		in dh isis database details
			topology 
				ipv4 0x0
				ipv6 0x4002

			we have spf for each topology

			use more resources but have best managemnet

			sh process cpu sort
