SDWAN
**************************************
general(1):
	cisco + viptela  > sdwan features

	if need depployement should use server
	64 gig ram and many cpu cores needed to launch it
		vmanage
		vsmart
		vbond
		vedge
		cedge

	best versions for labratoar > 20.4.1 & 19.3.0

	router cisco 7604

	data plane / forwarding plane
		share all control informations 
		fib (forwarded information base) is harware table
			routerd packet

	control plane
		routing packets
			punt means processs that transfer data plane on rib and create rib table

	all rib and fib goes to ip module or comes from

	c7604
		control plane module
			switch fabric module
				i/o module

	cause we are using single device and old models so doesn't need use orchestration and addressing
	each plane has own processor 

	we have higher cpu usage if list alot of routes in rib

	we have order in process and packets
		rib and routing
		then routed packets like fib

		punt and parking punt
**************************************
SDWAN Mechanism :
	between heach device use standard connectivity 
	inside the devices we have some connectivity might be not standard

	if we have too many broadcast in network may see lots of load
	in sdwan we make seprated concept and contents like data plane and control plane 
	our contents is not centralized also is not in same chasis

	instead of switch fabric module we can use isp and lte 4g or mpls

	in each location we have edge
	this edge recieve dta like traditional model and forward it to end user 
	in sdwan we exclude the control plane and put on another location

	edge devices just use i\o modules that data plane devices > edge(cedge or vedge) and control plane > central controler like vsmart

	all control plane contents are on vsmart or controler
	vsmart knows all path and network so calculate our routing process then put it on vedge or cedge
	whith this method we can decrease load of cpu in vedge or cedge

	cause we use isp mpls 4g ... in sdwan and replace these in devices with witch fabric may see lost less
	higher sla can solve this

	ini sdwan we have many edges and smart controlers on cloud so need define and control definitions our edge to controlers
	must use vbond to make orchestrations plane
	like server connect smart controlers to edges
	first make connectivity between our smart and edge 
	after confirmations for routing information basically on edge vbond disconnect connection with edge

	management plane 
		vmanage is our total dashboard to manage all components
		transfer all commands and operations vsmart
		we have security concept here
**************************************
provisioning :
	ztp (zero trust provisioning)
	pnp (plug and play)

	vanalitic > network analyzer in vmanage

	all components like vsmart ... are virtual except vedge or cedge is virtual and physical

	control plane use dtls and tls
		we have persistence connection on 
			vbond 
			vsmart
			vmange

			*use cert white list

		we have temporal connection on
			vbond
			vedge

			*use dtls

			*after definition our vsmart and vmange to edges purge our connection
			
			here we can connect our edges to control plane so we can use dtls or tls for permanent connection

	data plane use ipsec

	bgp > omp is overlay content that be on dtls and tls for routing protocol
		overlay manage protocol

	when we encapsulate our data on ipsec base connections to branch could use gre but better wait for it and replace

	control plane bring up > main parts like manage smart and bond

	in edges how find vbond?

	in first steps we have 3 mode to implement :
		cisco cloud (recommended)
		onpermis
		private cloude

		isr4000 can be useful for cisco cloud because we can set dhcp and ip then with ztp and pnp we get configs (namedd wan edge)

		when take order on cisco , our vbond get config and tell us which one is our bound
		can use it in ztp and pnp
		means vbond for vbond
		need just simple authentication

		if get trouble in connectivity between our smart and bond to edge must reset all configs?
		no all components get download on edges
		if use invalid configs might break connections

		after these our omp starts process of routing and edges with ipsec transfer all data

		*vmanage could be connect with api and get config

		edges in connectivity with smarts use dtls and tls session also use omp on ipsec tunnel

	instead of omp we can use another protocols to safe transfer our routes 
	edges learned all routes with omp (like reflector)

	*in ipsec we have many parameters based on smart and bond get transfer to edges

	vmanage and smart send destinations of eah router or edges and set ipsec on it
	example > smart detect with bfd (bidirectional forwarding detection) mpls link is not stable so change it to 4g these chanages comes on ipsec

	*bfd has request and reply time base on this time can detect link quality
		hardware accuracy and dataplane also cpu load

	edges send these attributes to smart then smart detect load and bandwidth then change it

	omp works inside the box (inside connections we have omp to advertise routes)
**************************************
TLOC(3)
	transport location

	we have nexthop concept in sdwan with this

	with rt we can seperate them
	per vrf / per vlan / per interface

	each customer has own site id
	each edge can use many ways for communication like mpls lte ....

	each edge get unique with system ip also could be not routeable but must be unique

	each link and communication ways has own circuite
	each circuite means unique tloc

	our uniqueness will be like this > x (system ip + color + encapsulation) + tloc

	tlocc is same our ip interface to seperation nexthops

	sometime we have nextop with 2 links so must setup our outgoing path

	wan edge1 > wanedge1-systemip1
		|--- circuite --- > mpls (tloc1)
		|--- circuite --- > isp (tloc2)

	wan edge2 > wanedge2-systemip2
		|--- circuite --- > lte (tloc3)
		|--- circuite --- > isp2 (tloc4)

	why tloc is so complicated?
		for nexthop we have 2 elemnt
			x
				1 : system ip and wan edge identity
					*with system ip we can define which wan edge must be used

				2 : color like internet ....
					* on mpls in private mode connect many links
					*if use internet must change behavior so change our policies and use ipsec

					tlocs must use specific values on wanedge connectivities  like need nat? or need ipsec?
					colors can help us 
					seperate them into the private and public
					bsically on these we can findout need nat or not 

					private need directly connections
						metro ethernet
						mpls
						internet
						private x
						private y

					public need nat modes
						3g
						biz-internet
						lte
						public internet
						green
						bronze
						customer-1

				3: encapsulation like gre or ipsec .....

			ip

		*based on type and nbar values can detect type of traffic then with vsmart change link and manage them


	we have segmentation on sdwan
		vpn is like vrf without rt 
		used vpid instead of rt

		vpn 0 means transport side (reserved)
			sdwan side 
			use ipsec and tls/dtls

			tloc and connections

		vpn 512 means out of bound management (reserved)
			not routeable

		vpn x means lan side
**************************************
IPSec on SDWAN(4)
	with different values on casual
		outer ip header
		udp header
		ipsec header
		segmentation header (vpn and vrf on sdwan)
		client ip header
		ipsec trailer

		with segmentation header we can make full mesh topologies and vpns
		each wanedge has security mechanism 
		access level for wan links and wan facing
		limit access for modifications on unauthorized smart vmanage or other edges
		all traffics are block except these on ingress ssh, netconf, omp, ospf, bgp, stun (session traversal utilities for nat)
		on egress dhcp, dns, icmp

		bfd (bidirectional forwarding detection)
			one of the must useful features with link state analysis
			transfer on 2 links between 2 router
			detect transfer rate and change link to make more effective connection
			implement inside fib and hardware modules

			if get connection loss on ipsec
			send echo back and latency  + loss

			after see bfd and nbar attributes we can change and use another links

			echo packets just transmit attributes about link has no process

			on sdwan we use for ipsec and wanedge interconnectivity
**************************************
WAN Edeg or vedge and vbond (OnBoarding)
	automatic (usually see this on reality)
		ztp
		pnp

	bootstrap
	manual

	when wan edge turn on must connect to dns and specific cisco servers
	after connection to pnp server our vedge get authentication and authorization
	based on serial number

	wan edges has 2 types
		viptela or cisco edge ussed ztp (use udp 12346)
		ios xe sdwan used pnp (use https)

	bootstrap mode use a flash memory include vbond codes (servers address)

	our wanedge
		virtual machine
		physical

		each type has viptela and cisco (ios xe sdwan) modes

		physical
			ios xe sdwan
				isr 1k series
				isr 4k series
				asr 1k server
				encs
				csp

			viptela
				100 series
				1k series
				2k series
				5k series


		vm
			cisco 1k v csr
			viptela vedge clow
			cisocisr-v

		*isr > integrated services router
		*asr > advance service router
		*csr > cloud service router
		*encs > enterprise network compute system
		*csp > cloud service platform

		our throughput + tunnel count + interfaces type is selection criteria

		if we have ios xe devices just need update our ios to ios xe sdwan

		dia (direct internet access)
			on traditional topologies we had hub&spoke model
			on hub&spoke model we needed more time to communication
			more cost
			on sdwan we have self-made security check
			we make direct access users our services or many services 
*************************************
SDWAN componentso
	vmanage
		network management system (nms)
		our dashboard to manage all systems

		provisioning means all first and initializing devices config

		policy creation
		software manager
		troubleshooting (visualize our tarnsmissions for users)

		better use cluster and odd count
		handle k wanedge

		support radius, tacacs+, saml

		api
			restconf
			netconf

		roll base access controll

		multitenant and single tenant

	vsmart
		control plane

		routing
			reachability with omp 
			update path with omp
			works like route reflector
			select best path
			transfer omp on dtls secure tunnel
			calculate all path then distribute them

		policy
			control
				generate topologies

			data
				acl and qos applicaion aware

		security
			ipsec between edges 
				phase 1 
					sa exchange 
					isakmp

				phase2 
					ipsec exchange 
					ipsec

		just on vsmart we can see all routes and best point of decision making also policy creation

		scaleable
		up to connections
		we can add 20 vsmart on each sdwan topology

		with policies we can define mesh connections and connectivity or be like hub and spoke model just be reachable from dc

		with netconf write policies on vsmart
		then omp and vsmart transfer update path and policies to wanedge

		on edges if each one connect directly get trouble
		on ipsec connection must send ipsec informations to vsmart then upload on edges

		better use 2 vsmart and same config on  different physical places 
		our policies get load share if get lost one of them reload balance happen
		*must make full mesh on vsmarts
		all informations are synced, in tarnsmissions we don't send database

		on edges if get connection lost with vsmart, all leasted informations are valid for 12 hours

		omp is a protocol for advertisement and management

	vbond
		orchestratin plane
		most important component
		first authentication and parameter connections (glue)
		with ztp and pnp connect our edges to vbond with vbond id 
		connect edges to vbond on dtls

		*dtls > datagram transport layer security

		our dtlsl will be on all transport of edges (mpls, isp1, tdlte ....)

		after vbond connections we can recieve and see vmange and vsmart attributes on edges

		then disconnection happen on vbond and edges

		if use many vbond must connect our edges to each vbond

			vbond 1.2.3.4 (here just use one ip address if need define another source must use dns and a-record)
			vbond x.vbond.com

			on special scenarios we have nat traversal must use stun

			wedge(192.168.1.10)----firewall(1.2.3.4)----internet----(stun 5.6.7.8)vbond
											nat public

			here vbond tell wedge you are behinde nat
			wedge send request to vbond, doesn't know is behinde nat

			vbond use dtls and nat-t and stun to reply on destination ip of wedge
			but edge is private ip
			so vbond realize that headers and packet of wedge and notify wedge you are on nat

			here wedge on advertise itself to vsmart can states with which ip to be seen

			it's okey our vsamrt or vmanage were behinde the nat 

			vbond must find and detect direct acess or nat model

	multitenancy
		when we use one isp and need share that have 3 modes
			dedicate
				use one seperated vmanage vbond and vsmart for each users

			vpn tenancy
				use set of controllers for each users
				use vpn and vrf on seprated model

				no common implement
				has no segmenetation

			enterprise tenancy
				mixed models of above
				use one vbond for all users
				vmanage can be pertenant but be one controller
				use many vsmart like dedicate

	sdwan depployement
		cisco cloud ops
			edges must be reachable on internet auto configuration with cisco

		msp ops (onpermis)
		enterprise it
**************************************
control plane and data plane(6)
	vsmart controller use omp to advertise attributes

	vsamrt can set best path selection adn manage key exchange on security association for ipsec

	omp works higher level than bgp
	more important than bgp and route reflector

	all connections were start with dtls and tls then use omp, for data plane use ipsec

	dtls (datagram transport layer security)/ tls (transport layer security)
	dtls use udp 12346

	has reorder

	ssl cert on dtls\tls used for connectivity after authentication
	all connections are zero trust so must use trusted root ca for every thing
	aprove all requests on organization ca

	vmanage and vsamrt use multicore (0-7)
	first and basic port and core
		core0 > port udp 12346 for dtls
		core1 > port udp 12446
		core2 > port udp 12546
		core3 > port udp 12646
		core4 > port udp 12746
		core5 > port udp 12846
		core6 > port udp 12946
		core7 > port udp 13046

		must manage our firewall on these ports and address till getconfig

		first config be on core0 after process distribute another ports and cores

	omp is main routing protocol on sdwan
	between vsmart vmanage vbond and edge use omp
	between edges we didn't use omp

	might see underlay with many different values
	but on overlay
		if use dtls for connections  our omp will run on vsmart and ddatacenter links

		omp
			router or vroutes
				address behind each router or edges

			tloc
				systemip + color + encapsulation
				rx + mpls + ipsec

			services route


	system ip on our scenarios are like
		1.0.1.1
			site.wedge

	with omp must advertise our routes to vsmart and dtls

	service side or lan --------- edge ---------- transport side or wan

	with omp
		tloc 
			bgp nexthop

			here means systemip + color + encapsulation

		origin
			we need to know records are advertised from which network location
			if be dynamic route must ser metric and use in best path selection

		originator
			which system ip has this route 
			if were x must do some thing

		prefences
			priority of routes or routing protocols
			if recieve range from some networks can redirect to another path
			higher is better

		site-id
			on best path selection can be applicable
			detect our routes comes from site x
			loop protection

		tag
			is optional attributes
			our redistribution is not included

		vpn
			segmentation on vrf

		services (service insertion)
			on datacenter we have a equipment called service and our topology is hub&spoke 
			if our service be like waf or firewall or ... need redirect all requests to datacenter and hub then send to spokes so in vsmart set this method and has own policy

	show omp route
		status cir (meeans chosen ,  install on rib , resolve the nexthop)

	tloc route (recursive lookup in traditional routing )
		for each interface or specific circuite we have tloc
		means you have path to reach device

		private tloc is not necessarily be private ip without nat
		some tlocs are public and manage with stun on vbound can determine them are behinde nat (first ip is public second ip is private)

		if we wanna simulate our concept for tloc, can point to gre
		gre contains 2 ip part (inside and outer header)
		here tlocs use public and private to detect our tlocs are using nat or not 
		if see same ip on public and private part means request comes from public ip
		if were different means request is behind nat

	on tlocs we have default full mesh connection for reliablity and stabilty
	site 1
		tloc mpls
		tloc isp

	site 2
		tloc mpls
		tloc isp

	our conenctions are 
		site 1 mpls > site 2 mpls
		site 1 mpls > site 2 isp
		site 1 isp > site 2 mpls
		site 1 isp > site 2 isp

		create 4 tunnels but use 1 of them

		better create connection with same colors
			site 1 mpls > site 2 mpls
			site 1 isp > site 2 isp

		we have restriction option to deploye this

	must use same encapsulation on each site (gre or ipsec)

	we have local weight with values that higher means better for each tloc

	on wedge if recieve route
		sh omp route 
			status c red (chosen, redistribute)

			means we have local
			on peer advertise reaching some ranges are available from me
			on vsamrt detect this way and change originators to this hop

		sh ip route vpn1
		sh temp tloc
		sh bfd session
**************************************
services routes(7)
	our hub&spoke with firewalls
	apply this services to maange connections 

	datacenter or hub tell hey vsmart i have specific service must advertise on omp to others
	is not same with tloc route and omp route

	first define our firewall location then set how works or how acess to analyz

	templates
		feature (services are here)
		devices

	must lable traffics to forward traffics and redirect them to firewall or services
	after recieve data from branch and bridge them to datacenter tloc, must lable 1006 for policies and services check get applied

	on vsamrt set this and define wedge topology structure is hub&spoke

	our services like firewall must be reachabe layer 3 or directly on layer 2
	if use layer3 be carefull use gre or ipsec for direct connections

	service route
		vpn-id

	service type
		firewall (fw) - svcid-1
		ids
		...

	lable
		service changing 

	generator -id
		same system ip

	tloc
	path-id
		each omp has one path

	*if see 2 wedges for one range what happen on best path selection?
		like route reflector  on traditional routing concepts behave
		
		*vsmart select best path by 
			active bfd session

			reachability like valid tloc and valid omp route

			localy source omp route (connected resources has higher priority)	

			higher omp preferences

			lower administrative distance (one ip range  has 2 way, use lower ad)

			higher tloc prefences (when check this on brnach or wedge has 2 links like isp and mpls)

			prefer origin and lowest origin metric (our ad)
				connected
				static
				eigrp intra area
				eigrp inter network
				ospf intra area
				ospf inter area
				ospf external
				eigrp external
				igp

			highest system ip
			highest tloc private address

		*we can use 16 ecmp links but make 4 active ecmp links

		sh run omp
			show us what is the advertisement

		*sdwan advertise all ospf interarea and intra area , statics and connected

		lan---------------branch-----------wan
		dynamic		| 					|
		routing 	|-- redistribute ---| omp
		protocols 	| 					|

		omp send origin types and origin sub types
			origin 		|		sub origin
			-------------------------------
			bgp 		|		external and internal
			ospf 		|		inter area , intra area , external
			eigrp 		| 		external , internal

	loop protection is an automatic option to solve problems
	wedges on organization unit advertise our routes to decrease latency and administrative distance
	each range can advertise also write black hole for them 
	if wedge 1 recieve wedge 2 networks and range advertise can detect down ward or down with flag then drop them
		ospf use down ward for loop protection
		bgp use site of origin for loop protection (soo)
			other word opm site-id in sdwan

		*in sdwan if see omp site-id on another wedge and same value detect is internal advertise on another edge

	must set same values on site-id

	extended community

	on ios xe sdwan we have omp-agent in eigrp, works like down wa' in ospf 
	on redistribute would be applicable
	if reach 0 another sdwan edge get down adn increase ad like 252 so omp adn another ad of edges are 250 
	here switch between them
**************************************
data plane operation(8)
	best transport is isp
	another transports like mpls ... used for another targets

	for ipsec must create key chaining then phase 2 start

	each connections and transport can be in ipsec negotiation

	n ^ 2 > 100 > 10000

	segmentation on sdwan can works with dmvpn
	on mpls
		ip(20byte), udp(8byte), esp(36byte), vpn(4byte and vrf), data

	transports and tloc color
		public
			3g 
			biz-internet
			public-internet
			lte
			blue
			bronze
			customer1
			gold
			green
			red
			silver

		private
			metroethernet
			mpls
			private1

	on vbond
		learn another branch to recieve tlocs and advertise their tlocs
		on normal condition make full mesh connection for ipsec
		if set restriction can limit ipsec connections
		usaully we use modified mode for connection and no default ipsec connection

		sh omp tlocs
			restrict > 1
				means our same interface types must connect to each other

			some situations like wedge with mpls and private tlocs might get trouble if have one connection with another edge just have one link like mpls so what should we do

			cause activate our restrict mode for tunnels couldn't tunnel between wedge 2 mpls tloc and wedge 1 private tloc

			if use biz internet instead of another types get trouble 

			must use tunnel group
				same like restriction

				can make 2 ^ 32 tunnel group

		for nat method better use full cone nat (one nat / static nat)
			if were single private ip address use translations
			can use dnat concepts for it
			in sdwan our ports are important

			fix port and state could be more faster adn useful for publishing our sdwan topologies

			symetric nat (dynamic pat)
				too many hosts use many ip address for internet
				or we have one ip address and use many port for this on xlate table to track connections

				private ip : random port x
				public ip : random port x (65535)

			address restriction cone nat
				mix model of above nat systems

			port restriction cone nat
				static nat from outside to inside

			first of all make connection (establish) from inside to outside then can access like dnat from out side to inside

			these access could be like all valid ip to structure
			on restrict mode can limit access on subnets

			must replay on same port and ip (sensetive on port)
			in situations of nat must reply just on same establish port and any packet must become from this port

			in sdwan we can limit same color nat connectivity

			wedges connect to vbond and vbond send wedge to vsmart 
			vsmart advertise all wedges to each other
			must connect to symetric nat from specific port and address connected to vbond
			nat table contain these edge ip and port address connected to vbond must negotiate on these not more
			if edges recieve another request from another port or ip address drop or block them

			here must use full cone nat for make connections

	netowrk segmentation
		like vrf

		transport (vpn 0 wan edges connect to outside network)
		management (vpn 512)
		service (vpn x customers  must be same on 2 side datacenter and wedge)

		vpn 0 contain ipsec and transport

		for vrf and seperation customers traffic must use label
		also use tloc route define lables to vsmart and detect customers location
		all transport interfaces must be on vpn0

		each wedge has own local label

		wedges for each transport can use different encryption key on ipsec connection
			aes 256 bit

		between wedge and vsamrt we have omp and dtls
		vsmart can advertise keys and tlocs
		each 24 hours update keys

		sdwan ipsec 
			authentication
				2048 bit key and esp + aeh

			encryption
				aes 254 bit

			intigirty
				antireplay enable
				gcm (galios counter mode) and aes 256

			use symetric keys for data
			our key exchange are asymetric because here we have vsmart

			vsmart transfer all keys data and wedges must negotiate each other

			wedge 1
				transport 1 key 1
				transport 2 key 2

			another type
				each wedge make a key for each transport (pair wise)
				also each wedge has seperated key

				wedge 1 key 1
					transport 1 key 1
					transport 2 key 2

				*better disable our pair wise to faster connection and decrease cpu load

			better use bfd 
				path mtu discovery and padding is advantages

				per minutes check the link status 
**************************************
wan edge list(9)
	on vmanage and wedges we must define our systems (edges) with list
	like license

	recieve from cisco

	cisco must knwon our locations and smart account attachments also count of wedges

	in real world or labratoar

	on cisco account > personal > company detail
		access management > smart account > request smart account > organization

		then login and goes to network > plug and play > add software devices and controllers

		controller profile
		devices > software devices
			use vedge cloud
			csr 1000 v
			cedge
**************************************
sdwan labratoar design and implement(10)
	onpermis
						cpu(core) 			ram(g)
		vmanage  		4 					24
		vbond 			2 					2	
		vsmart 			2 					2
		vedge 			2 					2
		cedge 			2 					4

		images version
			20.3 and 19.3

		vbond and vedge use same image

		viptela-x(vmanage/vsmart/vedge)-19.3.0-genericx86-64.ova
		viptela-x(vmanage/vsmart/vedge)-19.3.0-genericx86-64.serial.qcow2

		rename them to virtioa.qcow2

		need 100gig disk space for vmange
			/opt/qemu/bin/qemu-img create -f qcow2 virtiob.qcow2 100g

			cd /opt/unitlab/addons/qemu/vtmgmt19.3.0
			mv viptela-manager-19.3.0-genericx86-64.qcow2 virtioa.qcow2

			/opt/unitlab/wrappers/unl-wrapper -a fixpermission

		must set name of organization unit to detect them in the world

		we need routers and switch also need ca server on microsoft windows server (ca authentication)
			on windows server
				run active directory and domain controller then ca role

				domain controller on : "shayan.sdwan"

				on role installation better use active directory certificate service (ca and web enrollment)(4th option)

				click on warning alert then click on enterprise mode of ca and web enrollment

				inside active directory users adn computers > list users 
					cretae a admin level and admin group access user

				now take a certificate
					http://localhost/certsrv

		commands on vedge or viptela is different from cisco
			for config must assign organization unit name
			system ip must be unique
			vbond address
			site-id be unique
			ip vpn 0 is necessary also vpn 512
			ntp server

		vmange config
			conf ter
			system 
			#(equal global mode on cisco traditional)

			clock timezone asia/tehran
			system-ip 1.1.1.1
			organization-name "shayan.sdwan"
			vbond 192.168.1.3
			ntp server 162.159.200.1
			vpn 0 
			ip route 0.0.0.0/0 192.168.1.1 
			#(point to svi in topologies)
			#cause we need work on vpn must use this

			interface eth 0
			ip address 192.168.1.2  255.255.255.0
			no sh

			tunnel-interface 512
			#(point to vpn512)(can use seperated default gateway for each vpn)
			#if need use another vpn except vpn0 must use no tunnel-interface
			no tunnel-interface

			#on ios xe sdwan we need to commit our cnfigs so use these :
			show config
			#see configs before commit

			commit
			show runing
			#see after commit

		controllers username password is admin/admin
		must see system ready on cli then start config

		here we can login on vmange
			https://192.168.1.2:8443
			admin
			admin

			define certificate authority
				vmanage > administration > setting
					organization name > fill like smart account attributes

					vbond > 192.168.1.3
					controller certificate authorize > enterprise root certificate

					must define certificate on ca and windows server http://localhost/certsrv
						download
							base64
								ca certificate
									root

					insert on vmanage and inject all files instead of pem
						organization + secondary organization unit + organizor unit + domian name + set csr properties (must be enable)

							organization part > vIPtelaInc and name of organization unit on smart account

					here ca can authorize and authenticate others
					must set csr (certificate signing request)
						vmanage > config > certificate
						vmanage > devices

							from vmanage to ca can accomplishment generate csr  
							on ca we need request certificate adn advance ca cert to check our csr 
							after checking and generate base64 cert must upload them on vmanage
								vmanage > config > certificate

									install all ca server exported on vmange

			vbond cli
				config ter 
				system 
				host-name vbond
				site-id 1
				system-ip 1.1.1.3
				organization-name "shayan.sdwan"
				vbond 192.168.1.3 local
				#means vbond is itself

				vpn 0 
				int gig0/0
				no sh
				ip address 192.168.1.3 255.255.255.0
				no tunnel-interface
				#our tunnels between vmanage and vbond get start if do it

				ip route 0.0.0.0/0 192.168.1.1
				commit 

			must add devices on vmanage
				vmanage > config > devices
					add controller 
						vbond
						192.168.1.3
						admin
						admin
						generate csr

				must set certificate and csr to detect and authorize device
					vmanage > config > certificate 
						vbond > generate csr
							use this csr to request ca from ca server and export certificate
							base64 ...

					vmanage > config > certificate
						while we are onvbond must install our certificate

			on vmange must use these commands after see vbond
				interface eth 0
				tunnel-interface
				allow-service all
				commit

				*also do for vbond
			on vmanage dashboard must see vbond is up
			here we use bootstrap and doesn't hae certificate, so get trouble on authentication 
			so reason of disable our dtls and tunnel-interface is it

			we must do this process for vsmart

			*on ca server must set data sync service
				https://192.168.1.2/dataservice/system/devices/sync/rootcertchain

		vedges can be pnp or ztp initial config
		serialfile.viptela on smart account cisco has attributes to add devices on vmanage
		has list of organization unit devices
		must placed on ca server adn import on vmanage

		vmanage > config > devices > wan edge list
			validate & upload our serialfile.viptela

			*after these vmanage advertise this list details to vbond, vsmart

		vedge cli
			config ter
			system
			site-id 2
			system-ip 1.1.1.5
			organization-name"shayan.sdwan"
			vbond 192.168.1.3
			vpn 0 
			int gig0/0/0
			ip address 10.0.0.2 255.255.255.252
			ip route 0.0.0.0/0 10.0.0.1
			no tunnel-interface
			no sh
			commit

		on vmange add devices
			vmanage > config > devices 
				wan edge list
					request bootstrap config

					attributes of wedges recieved by cisco and give us adn file
						cloud initializing > kvm > eveng
						encoded string > esxi > vmware

			here our vmanage has cers but vedge doesn't have so must define root ca on vedge
				soloution 1 is ca server upload certs with ftp
					winscp > 10.0.0.2 and admin admin

					vedge cli
						request root-cert-chain install /home/admin/root.cert
						show certificate root-cert-chain

				now can detect certificates
				here vedge reqeust vmanage to retrieve all attributes related to chassis and .. vedge or itself parameters from serialfile.viptela 

					vedeg cli
						request vedge-cloud activate chassis-number 10 token 10
						
						#token is otp
						# now our serialfile is reachable and readablefrom vmanage
							vmanage > config > devices > wedge

						show controller connections
						vpn 0 
						int gig0/0/0
						no sh
						tunnel-interface
						allow-service all
						encapsulation ipsec
						commit

						#here with tunnel-interface we access run dtls and ... for first time

		csr1000v
			en
			config-transaction
			#need time

			system
			site-id 3
			system-ip 1.1.1.6
			organization-name "shayan.sdwan"
			vbond 192.168.1.3
			commit
			host-name cedge
			user admin secret admin priviledge 15
			int gig 0/1
			no sh
			ip address 10.0.0.6 255.255.255.252
			int tun 0
			ip unnumber gig0/1
			tun mode sdwan
			tun source gig0/1
			sdwan
			int gig0/0
			#outside interface

			tunnel-interface
			allow-service all
			encapsulation ipsec
			exit
			ip route 0.0.0.0 0.0.0.0 10.0.0.5
			commit

			must add ca root on cedge
			can use scp from ca server
			better add root cert on vmanage then use scp to recieve certs from vmanage

			on ca server we run winscp and conenct to vmanage ip adress
				/home/admin
				upload our root certificate

				then on cedge 
					copy scp://admin@192.168.1.2 bootflash: root.cert
					sh bootflash:
					requests platform software sdwan root-cert-chain install bootflash:root.cert

					for vedge
						vshell
						ls
						request platform software sdwan vedge-cloud active chassis-number x token y

						#use serialfile on vmanage > config > devices > wan edge list

			vmanage > monitor > network
				select devices
					relative (bfd)
**************************************
onboarding(11)
	vedge connections and execution need some prerequisites
	connections between vmanage vbond and vsmart has dtls media

	after authentication vedge by vbond  all attributes from vmanage and vsmart get uploaded on vedges
	then cnnection between vbond adn vedges get cut off
	next vmange and vedge negotiate on dtls
	vmanage upload setting and configurations on vedge
	this uploadign will be on netconf
	on configuration uploading procedure we need tokens 

	here we have vsmart ip address and load omp on vedge to recieve the routes 
	between	vedge and vsamrt we have dtls
	on negotiation vsmart and vedge we have tloc advertise and route advertise if have no policy limitation or filtering all tlocs get link

	next step is ipsec tunnel

	how find vbond by wanedge
		vedge
			zero touch provisioning (ztp)
				ztp.viptela.com

				managed by cisco
				like pnp
				recieved queries
				like sas

		cedge (ios xe sdwan)
			plug and play (pnp) (automatic model just need internet)
			bootstrap (usb falsh)				

			pnp model
				dhcp and dns resolve path and vbond values on https
					devicehelper.cisco.com

					we can resolve values with this on device by default
					also these are factory default configs

		all types has manual mdoe with cli config

	*with software.cisco.com can detect vbonds and parameters 
		serialfiles on smart acccount and chassis serial numbers can be unique identifier for cisco vbonds on internet

		vmanage get sync with software.cisco.com and pull configs on devices
			modified with serial numbers and tempelates
**************************************
how make tempelates
	cli config might contain error or faults
	not scalable
	has no revison number (version controll or roll back)

	here we need tempelates(12)
		vmanage > config templates (push on devices with netconf)
			template devices
				devices
					container

				features
					bgp
					multicast
					stp
					...

				cli

			attached devices
				device template
					feature template
						wizard (automated)

					device template
						cli template
							manual and command

		vmanage > config > devices
			on mode column we see cli and say change these

		vmanage > config > template
			first use feature then set devices

			add template
				undefined (first define devices	type (vedge cloud))

				then set part of config
					system(like global)
						site-id

					we can deploye 
						default (named after these (D))
						global (all configs get changes)(named after these (G))
							all templates with this parameter and type get effect same value

						device specific (depend our policies and site id selection)(named afetr these (DS))
							each joining request recieved must check the site id then deploye on it

				for timezone 
					timezone > global > asia/tehran
						*without checking for depployement on which site id apply on all configs

			vmanage > config > template > devices
				from feature templates
					device model
						vedge cloud

					name
						vedge devices

					basic info
						system 
							factory
								vedge system
									add profile name defined previous part

					transport & management vpn
					secure vpn
					additional tempelates

			vmanage > config > template
				features
					devices
						vpn
							vpn 0
							name > transport vpn

						ipv4 route
							prefix 0.0.0.0/0
							nexthop > device specific > nexthops will be asked from admin after finishing 

			vmanage > config > template > devices
				from features
				devices model > vedge cloud
				name > x
					*basic info > transport & management vpn

					vpn > vpn0 > template-shayan

			here make vpn interfcae template for topology
				vmanage > config > tempelates
					devices template
						click on 3 line helper then select attach template

						on next step must fill the devices specific

						we can review our changes with config review and config diffrences (green parts are added red means ommited)

				*must config vpn 512 (out of band management) to maek correct connection

				just need create vpn interface 512 with no config

				after apply these we cann't write or change configs with cli on wedges
	**************************************
	vsmart after recieving routes and networks from vedges on dtls, advertise it to all wedges on omp(13)
		we need define vpn 0,512,1 for each edge and their vpn interfaces

		vmanage > config templates
			features
				vedges cloud
					system basic config
						name > vedge-system (default)
						site-id , system-ip (device specific)
						timezone > asia/tehran (global)

						maximum omp session (if set 0 means omp get block)

					vpn
						basic config
							name > vpn 0 , transport vpn
							vpn > 0
							ipv4 route
								prefix 0.0.0.0/0 (g)
								nexthop (ds)

					vpn
						basic config
							name > vpn 512 , out-of-band-management vpn
							vpn > 512

							name > vpn 512 , out-of-band-management vpn interface
							vpn > 512

						*must create vpn 512 and vpn 1 for vedge with these values
						*need redistribute connected routes to omp on vpn 1 configs

					vpn interface
						name > vpn 0-gig (g)
						interface > gig 0/0/0 (g)
						ipv4 > static (ds)
						tunnel-interface > on (g)
						color > biz-internet (g)
						allow-service > all (g)

					vpn interface
						name > vpn 512-ether (g)
						interface > ether 0/1 (g)
						ipv4 > dynamic (g)

						*for vpn512 must use ipv4 on dynamic mode and interface on eth1
						*for vpn1 must use interafce g 0/0/1 and ipv4 static on device specific mode deployement

				after runing these we have ipsec tunnels and bfd with 3 targets 
					hello and hello back time
					jitter
					application aware

					vmanage > config > template 
						features
							vedge-cloud
								bfd
									name > vedge-bfd (G)
									appliation aware status check
										multiplier
										pull intervall

								*now we cann't turn off our connections so need define colors

								color
									name > bfd-tunnels (G)
									biz-internet (G)
									hello > 500 (g)
									multiplier > 6 (g)
									path mtu discovery > on (d)

					now bind them on vedge from vmanage
						vmanage > config > template
							devices
								from features
									basic
										system > vedge-system
										bfd

							now attach them and fill the DF parts put we can add them with ⇡⇣ load from excell file values

								on vedge cli
									sh ip route (full mesh)

								vmanage > monitor > network
	**************************************
	template on controllers (14) 
		we can config from gui and easier way
		better use cli config on vbond

		vmanage > config > devices 
			controllers
				transfer all of them to cli mode

				just vmanage are conencted to gui and cli

		vmanage > config > template
			features
				vmanage
					system 
						basic info
							system-ip (ds)
							site-id (ds)
							host-name (ds)
							organization name (d) > cann't change because all components will use it

					vpn
						basic info
							vpn 0
								name > vpn0 (g)

						ipv4
							static route
								prefix
									0.0.0.0/0 (g)

								nexthop 
									10.0.0.1 (g)

					vpn interfaces
						basic informations
							shut > no (g)
							name > vpn0-interfaec (g)

						ip
							10.0.0.2/24

						tunnel interface > on (g)
						color > biz-internet (g)
						allow-service > all (g)

		vmanage > config > template
			devices
				vmanage
					basic
						system > my-temp-name
						transport & management vpn 
							vpn0 and vpn0-interface
							vpn512 and vpn512-interfaces

		we can use these values for vsmart
			vmanage > config > template > features > vsmart

			*if were not about vmange couldn't set policies adn unmanaged

			just need add nexthops on them 
	**************************************
	on cedge we doesn't need add vpn512(15)
		vmanage > config > template
			features
				csr1kv
					cedge system 
						system-ip (ds)
						site-id (ds)
						host-name (ds)

						tiemzone > asia/tehran (g)

					vpn
						0
							name > transport-vpn(g)
							vpn > 0 (g)

						ipv4
							prefix
								0.0.0.0/0

							nexthop (1) (ds)
							nexthop (2) (ds)
						---------------------------------
						512
							name > out-of-band-management
							vpn > 512 (g)
						---------------------------------
						1
							name > lan/service (g)
							vpn > 1 (g)

						omp
							connected (g)

					vpn interfaces
						vpn0-int-g1
							basic info
								shut down > no (g)
								interfaces > GigabitEthernet1 (g) (be carefull must be same name of interface)
								ipv4 > static (ds)
								tunnel interface > on (g)
								allow-service > all (g)
								color > biz-internet (g)
						---------------------------------									
						vpn0-int-g2
							basic info
								shut down > no (g)
								interfaces > GigabitEthernet2 (g) (be carefull must be same name of interface)
								ipv4 > static (ds)
								tunnel interface > on (g)
								allow-service > all (g)
								color > mpls (g)
						---------------------------------
						vpn1-interface
							basic info
								shut down > no (g)
								interfaces > GigabitEthernet3 (g)
								ipv4 > static (ds)

			devices
				csr1kv
					system > cedge-system
					transport & management
						vpn0 + vpn0-int-g1
						vpn0 + vpn0-int-g2

					services vpn
						vpn1 + vpn1-interface

			then select attach to devices
			fill the blanks
			apply on cedges with slower mechanism

			here we can set restric mode to make connection on same interface type
				vmanage > config > template 
					features
						vpn0-int-g1
						vpn0-int-g2

						can change values on tunnel interface and set restric > on (g)

			vmanage > network > realtime > omp recieve routes
			on vmanage each place see (-) means we have same value

			on cedge we have omp ad on 251
	**************************************
	vrrp(16)
		vmanage > config > template
			features
				vedge-cloud & csr1kv
					system info
						site-id (ds)
						system-ip (ds)					
						host-name (ds)

						baudrate > 96000 (g)

					vpn
						basic info
							0
								vpn > 0 (g)
								name > transport (g)

							ipv4
								prefix
									0.0.0.0/0 (g)

								nexthop (ds)

							--------------------------
							1
								vpn > 1 (g)
								name > service (g)

							omp
								advertise connected (g)
							-----------------------------------
							512
								vpn > 512 (g)
								name > out-of-band-management (g)

					vpn interfaces
						vpn0-int-g1
							shut down > no (g)
							interface > GigabitEthernet1 (g)
							ip address > static (ds)
							tunnel interface > on (g)
							color > biz-internet (g)
							allow-service > all (G)
						-----------------------------------
						vpn1-interface
							shut down > no (g)
							interface > gig0/1 (G)
							ip address > static (DS)
							vrrp
								new
									groupid > 1
									priority > (ds)
									timer > 2 seconds (g)
									track omp > yes (g) (emans tracking on hsrp (change device if get trouble))
									ip > vip (ds) (on manual must add 10.0.0.1)
						-----------------------------------
						vpn512-interface
							shut down > no (g)
							interface > ether 0/1 (G)
							ip address > dynamic (g)

			devices
				from features
					vedge-cloud or csr1kv
						system > vedge-system
						vpn
							vpn0 + vpn0-int-g1
							vpn512 + vpn512-interface
							vpn1 + vpn1-interface

				attached to 
				must set rapid spanning tree on switches adn edge portsmust be known
					on switch cli	
						spaning-tree portfast edge default

		on vsmart recieve all routes from master and backup router (special tlocs)
		dependes on egress and ingress tunnel to reach the another wedge

		on egress we see vrrp effect and config effect no on ingress
		vsmart make full mesh connection on vrrp
		cause see and recieve same site id on ipsec and bfd session didn't advertise again on them

		on vsmart just use dtls and tls

		edge
			sh vrrp
			sh bfd sessions
			sh ip route
				s means selected
				f means fib

				*here can load balance them
			cedge
				show sdwan runing config

		vmanage > maintenance > device reboot
			after reboot see our devices are reacha bel and ping is stable
	**************************************
	we have too many transport on sdwan must be carefull to not be confused(17)
		on lab
			switch cloud dc
				ip routing
				int ether 0/0
				no switchport
				no sh
				ip address 100.0.0.10 255.255.255.0

				int ether 0/1
				no sh
				no switchport
				ip address 10.0.0.1 255.255.255.0

				show control local-properties

		be careful turn on tunnel interface
			vmanage > config > template 
				features
					vmange
						system
							basic info
								site-id (ds)
								system-ip (ds)
								host-name (ds)

								timezone > asia/tehran (g)

						vpn
							0
								vpn > 0
								name > transport

							ipv4
								prefix
									0.0.0.0 (g)

								nexthop
									10.0.0.1 (g)
							-----------------------
							512
								vpn > 512 (g)
								name > out-of-band-management (g)

						vpn interface
							vpn0-int-gig1
								shut down > no (g)
								interface > GigabitEthernet1 (g)
								ip address > static (ds)(10.0.0.3/24)
								tunnel interface > on (g)
								allow service > all (g)
								color > biz-internet (g)
								restrict > on (g)

								*we have same config on vsmart
								*use vpn512 on wedges adn apply restriction

				devices
					from features
						vmanage
							system profile and vpn profiles (vpn + vpn interfaces)

				attach on devices

		for vedges
			vmanage > configs > template
				features
					vedge-cloud
						system 	
							site-id (ds)
							system-ip (ds)
							host-name (ds)

							timezone > asia/tehran (g)

						vpn
							0
								vpn > 0 (g)
								name > transport (g)

							ipv4
								prefix 
									0.0.0.0/0 (g)

								nexthop
									100.0.0.100 (g)
 								#casue we have mpls link and need ipsec on one link use thi model config could placed with  another ip like 101.0.0.101
 							----------------------------
 							512
 								vpn > 512 (g)
 								name > out-of-band-management (g)
 							----------------------------
 							1
 								vpn > 1 (g)
 								name > services (g)

 							omp 
 								advertise connected + statics (g)

 							ipv4
 								static (ds)

						vpn interfaces
							vpn0-int-g1-intertnet
								shut down > no (g)
								interface > GigabitEthernet1 (g)
								ip address > static (ds)
								tunnel interface > on (g)
								color > biz-internet (g)
								restrict > on (g)
								allow service > all (g)
							-------------------------------
							vpn0-int-g2-mpls
								shut down > no (g)
								interface > GigabitEthernet2 (g)
								ip address > static (ds)
								tunnel interface > on (g)
								color > mpls (g)
								restrict > on (g)
								allow service > all (g)
							-------------------------------
							vpn512-interface
								shut down > no (g)
								interface > FastEthernet1 (g)
								ip address > dynamic (g)
							-------------------------------
							vpn1-interface
								shut down > no (g)
								interface > GigabitEthernet3 (g)
								ip address > static (ds)

				devices
					from features
						add our profiles

				attach devices
		-----------------------------
		some vedges on some scenarios has different vlaues adn configs on lesson 17 we have these
			router 1 on branch 1
				ip route 0.0.0.0 0.0.0.0 10.0.0.1

			vmanage > config > tempelates
				features
					vedge-cloud
						vpn
							vpn > 1 (g)
							name > services (g)

						omp
							advertise static and connected (g)

						ipv4
							static (ds)
							nexthop (ds)

				during deploye we must fill some parts
			------------------------------------------
			on cedges we have ospf on lan side
				routers
					router ospf1
					router-id 4.4.4.4
					network 10.0.0.14 0.0.0.0 area 0 
					network 4.4.4.4 0.0.0.0 area 0

				vmanage > configs > template
					features
						csr1kv
							ospf
								router-id > (ds)
								redistribute > omp (g)
								area number > 0 (g)
								area type > normal (g)
								interface > GigabitEthernet3 (g)
								default informations originate > on (d)

					devices
						from features
							add our profiles

					attach devices

					*for cedge better ommit vpn1 static routes (cause we add redistribution)
			------------------------------------------
			on edge we have bgp
				site 1 means as 1

				router
					ip route 0.0.0.0 0.0.0.0 10.0.0.1

					router bgp 1
					neighbor 10.0.0.1 remote-as 1
					network 1.1.1.1 mask 255.255.255.255

				vmanage > config > template
					features
						vedge-cloud
							bgp
								shut down > yes (g)
								as number > 1 (g)
								propagate as-path > no (g)(we have site communication better turn it off)
								redistribute > omp (g)
								neighbor
									address (ds)
									remote-as > 1 (g)
									address family > ipv4 (g)

							*must set bgp redistribute on vpn1
							*ommit static route on vpn1 cedges (cause we add redistribution)

					devices
						add profiles defined at above
			------------------------------------------
			eigrp on cedge
				vmanage > config > tempelates
					features	
						csr1kv
							eigrp
								as id > 1 (g)
								interface > GigabitEthernet3 (g)
								shut down > no (g)
								redistribute > omp (g)
								network 
									prefix (ds)


							vpn
								1
									vpn > 1 (g)
									name > services (g)
									omp > advertise eigrp (g)

					devices
						from features
							csr1kv

						appply all profiles

				router
					router eigrp 1
					network 10.0.0.13 0.0.0.0
					network 4.4.4.4 0.0.0.0
			------------------------------------------
	**************************************
	network segmentation on tempelates(18)
		we use same ip address on topologies to create vrf

		vmanage > config > template 
			features
				vmanage-&-vsmart
					system
						site-id (ds)
						system-ip (ds)
						host-name (ds)

						tiemzone > asia/tehran (g)

					vpn
						0
							vpn > 0 (g)
							name >  transport (g)

						ipv4
							prefix
								0.0.0.0/0 (g)

							nexthop
								10.0.0.1 (g)
						---------------------------
						512
							vpn > 512 (g)
							name > out-of-band-management (g)

					vpn interfaces
						vpn0-int-g0
							shut > no (g)
							interface > GigabitEthernet0 (g)
							ip address > static > 10.0.0.3/24 (cause we have vrf and same ip on many devices)
							tunnel interface > on (g)
							color > biz-internet (g)
							allow service > all (g)
						---------------------------
						vpn512-interface
							shut > no (g)
							interface > FastEthernet1 (g)
							ip address > dynamic (g)

			devices
				from features
					vmanage-&-vsmart
						system profile
						transport and management profile
							vpn 0 + vpn0-interafce	
							vpn512 + vpn512-interface

			we msut create another profile for vsmart with diffrent ip addressing
				here doesn't need vpn512
				 for vsmart

				 vmanage > config > template
				 	features
				 		vedge-cloud
				 			system
				 				site-id (ds)
				 				system-ip (ds)
				 				host-name (ds)

				 				timezone > asia/tehran (g)
				 				baudrate > 9600 (g)

				 			vpn
				 				0
				 					vpn > 0 (g)
				 					name > transport (g)

				 				ipv4 
				 					prefix > 0.0.0.0/0(g)
				 					nexthop > 100.0.0.100 (g)
				 				------------------------------
				 				512	
				 					vpn > 512 (g)
				 					name > out-of-band-management (g)
				 				------------------------------
				 				1
				 					vpn > 1 (g)
				 					name > services (g)

				 				omp
				 					advertise > static + connected (g)

				 				ipv4 route
				 					prefix > 1.1.1.1/32 (g)
				 					nexthop > 10.1.0.2 (g) (cause we have vrf and same network)
				 				------------------------------
				 				2
				 					vpn > 2 (g)
				 					omp > advertise ospf + connected (g)
				 				------------------------------
				 				3
				 					vpn > 3 (g)
				 					omp > advertise bgp and connected

				 			vpn interfaces
				 				vpn0-int-g0-internet
				 					shut > no (g)
				 					interface > GigabitEthernet1 (g)
				 					ip address > static 100.0.0.1/24 (g) (cause we have vrf)
				 					#we have same thing on cedge but another ip address
				 						100.0.0.100
				 						nexthop 100.0.0.2

				 					color > biz-internet (g) 
				 					restrict > on (g)
				 					tunnel interface > on (g)
				 					allow-service > all (g)
				 				------------------------------
				 				vpn512-interface
				 					shut > no (g)
				 					interface > FastEthernet1 (g)
				 					ip address > dynamic (g)
				 				------------------------------
				 				vpn1-int-g3
				 					shut > no (g)
				 					interface > GigabitEthernet3 (g)
				 					ip address > static 10.1.0.1/30 (g)(cause we have vrf and same values)
				 				------------------------------
				 				vpn2-int-g2
				 					shut > no (g)
				 					interfaces > GigabitEthernet2 (g)
				 					ip address > static 10.1.0.1/30 (g)
				 				------------------------------
				 				vpn3-int-g3
				 					shut > no (g)
				 					interfaces > GigabitEthernet3 (g)
				 					ip address > static 10.1.0.1/30 (g)

				 			ospf (need for vpn2)
				 				router id > 1.0.0.5 (g)
				 				redistribute > omp (g)
				 				area number > 0 (g)
				 				interface > GigabitEthernet2 (g)

				 			bgp (vpn3)
				 				shut > no (g)
				 				as number > 1 (g)
				 				propagate aspath > off (g)
				 				redistribute > omp (g)
				 				neighbor	
				 					address > 10.1.0.2 (g)
				 					address family > on + ipv4 unicast (g)

				 	devices
				 		from features
				 			vedge-cloud	
				 				system
				 					vpn0 +vpn0-int-g0-internet
				 					vpn512 + vpn512-interface
				 					vpn1 + vpn1-int-g3
				 					vpn2 + vpn2-int-g2 + ospf
				 					vpn3 + vpn3-int-g3 + bgp

				 	attach devices

				 	*use these for cedge but instead of bgp set eigrp

				on vmanage > monitor > network > realtime > bfd session
					see ipsec tunnel and segmentation
**************************************
sdwan policies(19)
	we can define topology adn policy 
	hub and spoke
	branch to branch to datacenter

	we have many links like mpls and internet
	mpl has qos and privacy benefits
	internet is cheaper soloution

	some policies are control and data plane base

	internet base networking is our reponses from sdwan

	on tloc and omp tarnsmission and create full mesh topologies could be modified and converted to hub&spoke topologies

	sdwan policies
		localize
			at least effects on one device

		centralize 
			our full fabric effected

			control (effects on omp and vpn)
				vpn memebership
					topology policy
					can limit vpn access on sdwan and modify our topologies

					some guest use dia (direct internet access) on different path and bypass organization lan

				control
					same condition on tloc transmission

			data (traffic policy)
				traffic data
					like acl deployement on devices 

				applicaion aware routing
					some applications need minimum bandwidth here can set these apps and bandwidth

				cflowd
					special type wiht destination definition then export traffic to make them analyze
**************************************
centralized policy
	convert full mesh topologies to hub&spokes
	some times we can save money and create network on hub&spoke model

	after wedges advertise tlocs and omp to vsmart get meaning we have default vpn
	vsmart can change and advertise again our omp adn vpn default

	here if change or filter our smart advertisement, instead of transfer all ltocs for branches just advertise datacenter tloc, with these methods every one must negotiate with other branches on datacenter

	first of all must create our lists(20)
		vmanage > config > policy > centrall
			site
				site name > dc
				address > 1
				-----------------
				site name > branches
				address > 2,3,4

			then use use topology or vpn memebership to limit the connections
			here we use topology
				set on custom (righ up we have menu) and  hub&spoke
					default action > block or reject (like routers acl deny any)
					------------------
					name > x
						sequence type
							tloc
								add sequence rule
									site
										datacenter
											action > accept (just this tloc get advertise)

							route
								add sequence rule
									site
										datacenter
											action > accept (just route and ip address of datacenter get advertise)

				then use addpolicy adn named our policies like hub&spoke-policy1-v1

				*our vsamrt must be on vmanage mode to recieve policies

				on topology must select new site list
					inbound and outbound direction
						out means advertise to branch
						in means recieving by datacenter

					at the end of process must select activate option on 3 pointer and commit them
	**************************************
	like above scenarios but on branch to branch to datacenter(21)
		traffic will be like proxy behavior

		we need semmerarization or tloc list (routes summary recieved by dc then all nodes known paths)

			vmanage > config > policy > centrall
				site
					name > datacenter
					site id > 1
					----------------
					name > branch
					site id > 2,3,4

				vpn
					name > datacenter-vpn
					vpn > 1
					
				topology
					hub&spoke
						hub > spoke
						spokes > branch
						vpn list > vpn1

				then active them

				*on vpn1 we set advertise ment connected and static 
					also  we set ipv4 route
						prefix > 0.0.0.0/0
						nexthop > ip datacenter
						null 0 > on
						*mark as optional

					*if deactivate them could earase from topology
				here our wedges are recieving our datacenter tloc as default gateway
				sh ip route vrf 1

				see all path are black hoel cause set null0

			we can change and replace dc tlocs on vsmart
				vmanage > config > policy > centrall
					tloc
						tloc list
							ip
							color
							encapsulation
							prefences

					topology
						sequence role
							route
								site
									datacenter
									------------
									branch > accept

									tloc > dc

							tloc
								type site
									dc

					*replace tloc branch with dc and make dc bridge for all connections
	**************************************
	traffic engineering and tloc prefences(22)
		on these topologies we have different site-id
		so set restriction on ipsec 

		recieve network ranges and advertise it to vsmart on this model we see 4 path

		vmanage > monitor > network > realtime > omp recieve tloc

		we can see load balance
			cir (chosen on lower metric , tloc installation , resolved)

		migh detect asymetric transmission cause we have mpls and internet link also use many links to negotiate
		must make primary and secondary wedges

		we can set this methods on
			tloc preferences	
				base on template
					vmanage > config > template
						features
							vedge-cloud
								must create vpn0 and interfaces for transport and vpn0
								aslo need vpn1 for clients adn vpn 512 for oob-mgmt

								on vpn0 and interfaces inside tunnel part we see allow-service ad advertise adn preferences
								 better set prefences on (ds)

								 weight parameter is the load distribute 
				------------------------
				base on central policy
					vmanage > config > policy
						site
							name > x
							site id > 2
							-----------
							name > y
							site id > 3


						custom tloc and route
							default action > pass
							---------
							tloc
								sequence role
									originator
										1.0.2.1
										----------
										1.0.2.2 
											preferences (300) 
											action > accept

						we set dc recieved attributes must be same at above values
						we recieve omp records but no routes

						set inbound dc adn site lists
	**************************************
	prefer regional datacenter(23)
		we have 2 dc in scenarios
		and they are redundant

		always use nearest location for internet access

		in this topology we nee hub&spoke model also need nearest internet access

		on spkes we set static routes adn redirect their omp on local networks
		here we have loadbalance 

		vmanage > monitor > troubleshoot (can simulate traffic)

		vmanage > config > policy > centrall
			site
				tehran-dc
				site id > 50
				priority > 1
				----------------
				rash-dc
				site id 51
				priority > 3

			*priority 2 means don't care

			prefix list
				incoming
					ipv4
						8.8.8.8/32
						8.8.8.9/32


			topology
				custom
					tehran-dc-topology
						default action > reject

						sequence role
							tloc
								sequence role
									site
										tehran-dc
											action > accept
										-------------------
										rasht-dc
											action > accept
							-------------
							route
								sequence role
									site
										tehran-dc
											prefix list > internet
											action > accept
											preferences > 200
										---------------
										rasht-dc												
											prefix list > internet
											action > accept
											preferences > 100
					---------------------------
					rasht-dc-topology
						default action > reject

						sequence role
							tloc
								sequence role
									site
										tehran-dc
											action > accept
										-------------------
										rasht-dc
											action > accept
							-------------
							route
								sequence role
									site
										tehran-dc
											prefix list > internet
											action > accept
											preferences > 100
										---------------
										rasht-dc												
											prefix list > internet
											action > accept
											preferences > 200

			apply on site list out direction for vsmart
			also our vsmart must be on vmanage mode

			if need some sites get full mesh and hub&spoke
				vmanage > config > policy > centrall
					topology
						custom
							tehran-rasht-dc
								default action > reject
								--------------------------
								sequence
									tloc
										site
											tehran-dc
											action > accept
											----------
											rasht-dc
											action > accept
									--------------
									route
										sequence
											site list
												tehran
												prefix > internet
												action > accept
												--------
												rasht
												prefix > internet
												action > accept
	**************************************
	regional mesh (24)
		we mix all scenarios in this model
		every body use own city dc
		for internet can use primary and secondary edge 
		if get trouble change wedges

		our branch dc edges can see each other directly
		if need conenction between branch dc must negotiate on dc (hub&spoke model)

		we have internet and mpls link
		for internet link
			we have 2 dc
				ahvaz-dc (primary for their city and secondary for rasht)
					each dc has own city and branch

					dezfol
					abadan
					shoshtar
					
				rasht-dc (primary for own city and secondary for ahvaz)
					lahijan
					anzali
					astara

		for mpls we have same concept on internet (primary and secondary)

		tlocs between each city and provins mudt be replaces with their datacenter provins
		guilans cities must replace the ahvaz cities with thier ahvaz datacenter tloc

		vmanage > config > policy > centrall
			site
				rasht
				site id > 200
				-------------
				ahvaz 
				site id > 201
			---------
			site list
				rasht-cities
					1,2,3
				-------------
				ahvaz-cities
					4,5,6
			---------
			tloc 
				ahvaz-tlocs
					1.0.201.1
					color > mpls
					encapsulation > ipsec
				-----------
				guilans-tloc
					1.0.200.1
					color > mpls
					encapsulation > ipsec
			---------
			topology
				custom
					guilan-topology
						default action > reject
						----------------------
						sequence 
							tloc
								site
									rash-dc
										action > accept
										preferences > 200
									----------
									ahavaz-dc
										action > accept
										preferences > 100 (make ahvaz dc adn internet secondary)
									----------
									guilan
										action > accept (we ommit the priority)
							--------
							route
								sequence
									site
										rasht
											action> accept
										---------
										ahvaz
											action > accept
										---------
										guilan
											action > accept
											site list > rasht-cities
										---------
										khozestan
											action > accept
											tloc > ahvaz-tlocs (here replace tlocs of ahvaz cities with ahvaz datacenter provin)
					---------------
					ahvaz-topology
						default action > reject
						----------------------
						sequence 
							tloc
								site
									rash-dc
										action > accept
										preferences > 100 (make ahvaz dc adn internet secondary)
									----------
									ahavaz-dc
										action > accept
										preferences > 200 
									----------
									ahvaz
										action > accept (we ommit the priority)
							--------
							route
								sequence
									site
										rasht
											action> accept
										---------
										ahvaz
											action > accept
										---------
										khozestan
											action > accept
											site list > ahvaz-cities
										---------
										guilan
											action > accept
											tloc > rasht-tlocs (here replace tlocs of rasht cities with rasht datacenter provin)

		if need negotiate with another provin must change tloc and replace cities site id and tlocs with edge of provin tloc then goes to another provins
		must deploye on outgoing direction
	**************************************
	service insertion(25)
		more security is our obsessed
		so apply firewall service in datacenter
		must redirect traffic to firewall

		first check data on datacenters firewall then goes to branches

		like b2b through datacenter
		here we have optional redirections to firewall

		from vpn0 on vedges asked to define services on vsmart

		vmanage > config > policy > centrall
			site
				dc
				site id > 1
				--------------
				branches
				site id 2,3,4
			----------
			tloc
				dctloc
					1.0.1.1
					color > mpls
					encapsulation > ipsec
			----------
			topology 
				sequence
					tloc
						site > dc
						action > accept
					---------
					route
						site > dc
						action > accept
						---------------
						(*)
						site > branch
						action > accept
						tloc > dc

			apply on outgoing direction over site brancehs

		vmanage > monitor > omp recieve route (lables)
			with this can advertise our services on dc to others

		sh omp service

		on dc edge has firewall need negotiate with vsmart
		dc must use vpn1 and advertise firewall service on it
		vpn1 service type > firewall
		if need make gre or tunnel must set interface on config also need one transport ip

		(*)site
			branch
			action > accept
			services > firewall

			*update	our policies and avoid change tloc to dc, must say use the services
			last lable is not valid use new features
	**************************************
	isolation guest (26)
		with vpn policy
		we need limit guest	access to their city sites and network resources
		also can negotiate with other cities guests not more (no resource visibility from other branches)

		guests > full mesh acces but no internet
		corporations > full mesh access + internet + b2b through dc

		define vedges like
			vedge-1
				net-a-vpn-1
				ipsec
				biz-internet
				1.0.1.1
			---------
			vedge-2
				net-a-vpn-2
				ipsec
				biz-internet
				1.0.2.1

		vmanage > config > policy > centrall
			site
				dc
				site id > 1
				-------------
				branches
				site id > 2,3,4
			---------
			tloc
				dc-tloc
				1.0.1.1
				encapsulation > ipsec
				color > biz-internet
				preferences > 200
			---------
			topology
				custom
					default action > reject
					------------
					sequence
						tloc
							site
								dc
								action > accept
						---------
						route
							site
								dc
								action > accept
								----------------
								branch
								action > accept
								tloc > dc
			---------
			vpn
				on vpn1 must define each services we need to get publsh and allow

		after deployement we set vpn memebershiping
			vpn memebership > branches = corporate-vpn

		apply on outgoing direction to branches
	**************************************
	per segment topology(27)
		we have full mesh toplogy
		but need different topology for each vpn and vrf
		just advertise route and tloc datacenter
		(hub&spoke created)

		if need b2b through dc
			tloc & route dc + branches route (replace tloc with dc)

		vmanage > config > policy > centrall
			site
				dc
				site id > 1
				----------
				branch
				site id > 2,3,4
			--------
			tloc
				dc
					1.0.1.1
					color > biz-internet
					encapsulation > ipsec
			--------
			vpn
				vpn2
				number > 2
			--------
			topology
				custom
					sequence
						tloc
							dc
							action > accept
						-------
						route
							dc
								accept
							-----------
							branch
								vpn2
								action > accept
								tlocs > dc

			apply on outgoing direction to branches
	**************************************
	extranet & shared services (28)
		some services shared with our partners with limited scope access
		vpn1 must see vpn1 and vpn2 is same
		also vpn 100 can connect on isolated model with each vpn1 and vpn2
		must write porlicy on vsmart adn say export and convert them 

		datacenter on vpn100 >>>> in >>>>> vsmart (export\convert) >>>> vpn1 and vpn2

		vsmart (convert vpn1 adn vpn2) >>>>> out >>>>> datacenter on vpn100

		vmanage > config > policy > centrall
			site
				dc 
				site id > 1
				----------
				branches
				site id > 2,3,4
			--------------------
			vpn
				dc
				vpn > 100
				------------
				brancehs
					vpn > 1,2
			--------------------
			topology
				custom
					dc-to-branch-vpn
						default action > reject
						-----------------------
						sequence
							tloc
								dc
								action > accept
								-------------
								branch
								action > accept
							------
							route
								dc
								action >  accept
								export to > 1,2 or branch site list
								----------------
								branch
								action > accept
					----------------------------------
					branch-to-dc-vpn
						default action > reject
						-----------------------
						sequence
							tloc
								branch
								action > accept
							------
							route
								branch
								action >  accept
								export to > dc or extranet or 100
							

				apply on inbound and profile dc-to-branch-vpn
				apply on outgoing and profile branch-to-dc-vpn
				to vsmart
	**************************************
	dia for guest users (29)
		vpn1 is b2b through dc
		vpn2 is limited internet access no ou lan access

		must set nat on transport and outside interfaces
		then set vpn2

		now all links are full mesh

		vpn1 doesn't need nat so use direct access

		vmanage > config > template
			features
				create vpn0 and interface transport for them
				also set nat > on (g)
				block icmp > on (g)
				responding > on (g)
				--------------------------
				*must leak our static default route for vpn2 on vpn1 and nat concept

				vpn2
				set static ip 0.0.0.0/0 also set vpn > on (g)
				means we leaked routes on vpn0

				on vedge
					sh ip route vpn2 (say you have nat for default route)

				here we have googel ping for vpn2

		vmanage > monitor > network > vedge2 > realtime > ip nat

		here we need change toplogy and limit 
			vmanage > config > policy > centrall
				site
					dc > 1
					-------
					branch > 2,3,4
				-------
				tloc
					dc
					1.0.1.1
					color > biz-internet
					ipsec
				-------
				vpn
					corporations > 1
					---------
					guest >2
				-------
				data prefix
					ipv4
						10.0.0.0/8
						172.16.0.0/16
						192.168.0.0/16
				-------
				topology
					custom
						default action > reject
						------------------
						sequence 
							tloc
								site
									dc > accept
							--------
							route
								site
									dc > accept
									-----------
									branch > accept
									vpn1
									tloc > dc

				on vpm memeber ship must set vpn1 or corporations be accept adn site is branch
				on traffic policy we set traffic data
					traffic data (if our traffic has x destination ip set these values)
						sequence
							application aware
							qos
							...
							custom (is best)
								sequence (retrieve whole packet)
 									destination data prefix list (list created last part)
 									drop
 									counter (guest drop counter)
 									-------------------------
 									y
 										action > accept
 										nexthop / nat vpn (disable fallback)
 										counter

 				apply our
 					traffic policy 
 						from site branch and service on guest vpn 

 						to

 						destination adn transport

 					control policy
 						outgoing on branch

 				our traffic control deployed on vedge
 					sh policy from-vsmart
 					sh policy data-policy-filter
 	**************************************
 	application base trafic engineering (30)
 		internet
 		mpls

 		are our youtube access path also has primary and secondary priority

 		also just use internet for facebook

 		show policy service-path vpn1 int gig0/0 source-ip 10.1.0.1 des-ip 10.2.0.1 protocol 1 app youtube all

 		data policy set outgoing path

 		vmanage > config > policy > centrall
 			site
 				branch1 > 1
 				--------------
 				branch2 > 2
 			---------
 			tloc
 				branch2> 1.0.2.1
 				biz-internet
 				ipsec
 			---------
 			vpn
 				vpn1 > 1
 			---------
 			application
 				list
 					youtube
 					facebook
 			---------
 			topology
 			---------
 			traffic data 
 				rule
 					custom
 						x
 							application
 								youtube > accept
 								local tloc 
 									color > biz internet
 									ipsec

 									*this attribute can switch the link on local vedge
 									*except this works normally

 						y
 							application
 								facebook > accept
 								tloc 
 									color > biz internet
 									ipsec

 									*this attribute is fixed and diffrent from local tloc
 									*except this our traffic get drop


 			apply  traffic datat on 
 				service > branch1 > vpn1

 		vmanage > tools > optional > reset interfaces
 	**************************************
 	packet loss(31)
 		tcp\ip and tcp use to make warranty of sla and ...
 		use application aware routing used for multiple links 
 		single link monitor	
	 		fec > forwarded error correction
	 		packet duplication 

	 	fec is parity bit transfer on links help fast recovery
	 	resource intensive

	 	sh runnel statistics fec des-ip 172.16.1.2 

	 	in 3 second loss one packet and 100 msecond were our jitter can recover one packet

	 	[p1,p2,p3,p4] > (fec parity and block base on xor) generate p5
	 	p5 get loss > not important
	 	if p2 get loss could recover on p5 and parity

	 	25%  occur link bandwidth casue parity packet is the same with largest packet in network
	 	we can accept this for voice not always

	 	if get 2 packet loss couldn't recover them

	 	fec mode
	 		always
	 		adaptive (dyanmic)(if loss 2%  get active)

	 		vmanage > config > policy > centrall
	 			site
	 				branch1 > 1
	 				-----------
	 				branch2 > 2
	 			---------------
	 			vpn
	 				vpn1 > 1
	 			---------------
	 			traffic data policy
	 				sequence
	 					custom
	 						x
	 							destination ip prefix > 0.0.0.0
	 							action > accept
	 							loss correction
	 								adaptive
	 								always (in iran use this)
	 								packet duplication

	 			apply on branch1 and branch2 vpn1
	 			from services adn traffic data policy name

	 			if have too many loss is not applicable

	 			packet deduplication
	 				between 3 edge of ou we have many path 
	 				sen each packet 2 times

	 				packets [1,2,3] > 
	 					mpls > 3,4 >>>>> 1,2
	 					internet  > 1,2 >>>>> 3,4

	 					vmanage > monitor > realtime >  tunnel deduplication

	 					sh tunnel statistics packet-deduplication
	**************************************
	application aware routing (32)
		sdwans the best advantage over traditional networks 
		need more payment
		check realtime link status
		our main criteria is jitter ....

		our checking is active active mode
		used bfd attributes then transfer to vsmart

		bfd adn bfd sessions are useful for ipsec tunnels
		echo and echo back

		on scenario we have wedge with 3 nexthop and default gateway

		we set restric on mpls to be dedicated link and private 
		use lte and internet links without restrict (make full mesh on these)

		path quality monitoring
			during certian periods time check bfd counts
			send and recieves will be checked

			example > each 10 seconds based on hello packets measure our link quality 600 seconds are normal

			must consider buckets on intervals
			4 buckets in 10 seconds
			our average time is real behavior

			application routing poll interval
			application routing multiplier

			migh detect slower than normal state sidepends on link status and maybe behave asymetric

			vmanage > config > policy > centrall
				site
					branch1 > 1
					------------
					branch2 > 2
				------
				vpn
					vpn1
				------
				application
					x
						youtube
				------
				sla class
					sla-a
					set priority fo r traffic monitor

					latency > 50 msec
					loss > %
					jitter > 100 msec
				------
				traffic data
					application aware
						sequence
							application family list
								application > x
								sla class > sla-a

			if recieve application x data must check sla state adn provide link with sla on omp and best paths

			apply on branches on outgoing direction and vpn1
			set application aware routing on service (lan) to transport (wan) 
			opposite of traffic data works on ingress or egress path

			inside application aware policy we have prefered color means 
				if set value like use mpls > mustbe with these sla if were not change it to another link 
				if be blank means you make load balance on them

				none of them hasn't our sla policy, use mpls  or set backup prefered color might help us to use backups 

				restric model in this method means drop traffic and don't use any thing 

		we can see rtt (round trip time) on bfd packets 
			bfd hello > 1000 msec or 1 second

		we have multiplier value means you didn't recieve many packets must drop connection
			we can cache 7 packets
			or 7 seconds

		vmanage > monitor > realtime > bfd history

		vmanage > config > template
			features
				default
					bfd
						we cann't cahnge values

		vmanage > config > template
			features
				bfd-vedge-cloud
					bfd
						color
							mpls
								100 (hello)
								5 (multiplier) 
								(g)
							-----------
							internet
								200 (hello)
								5 (multiplier) 
								(g)
							-----------
							lte
								1000 (hello)
								7 (multiplier) 
								(g)

			we add these bfd values on vpn0 inside basic info
			set false positive intervals means you detect link is up by mistake
**************************************
localize policy(33)
	we deploye our policies on one wedge no total fabrid configuration
	effect just one branch

	on sdwan version 18 use zone base firewall
	on version 19 get better

	localize policy
		traditional
			route policy (control)
				filter some routes on advertisement

				has more details over centralize model and better

			qos (data)
			access controll list (acl)(data)

		security
			firewall
			url filter
			dns security
			ips
			advance malware protection (amp)

	control policy localize(34)
		localize route policy is same with route map

		vmanage > config > template 
			features	
				vedge-cloud
					bgp
						shut > no (g)
						router-id > (ds)
						as-numbner > (ds) / 1 (g)
						propagate as-path > on (g)
						redistribute > omp (g)

						neighbor
							address family > on > ipv4 (g)
							address > (ds) (cause our variables has same name must change them so set mark as optional)
							route policy > out (g)
							policy name > (ds)
							remote-as > 1 (g)

			now set on devices template adn run bgp on another router

		router
			tuer bgp 1
			max-path 3 
			#for ebgp

			max-path ibgp 2
			#for ibgp

		on sdwan bgp we hvae some different values like local preference on ios is 100 but in sdwan is 50
		higher is better 
		set vedge1 local preference > 200
		vedeg2 local preference > 100

		vmanage > config > policy > localize policy
			route policy
				sequence
					all routes
					action accept
					local preference > 200

			*define another policy then set local preference on 100

			then apply on some devices no all fabric devices

			vmanage > config > template > device template
				additional tempelates > policy

					cause we have 2 policy route should define another parent policy and be usable
					so set this parent
						vmanage > config > policy > localize > route policy
							name 2 route policy on parent policy then will be useful on templates

		our policies are uploaded on devices
	**************************************
	data policy localize(35)
		same above scenarios 

		on branch site we set ip loop back 2.2.2.2/32 injected into the bgp
		set limit access on acl to reaching  this site

		vmanage > config > policy > localize
			access control list policy
				acl sequence
					default action > accept
					----------------------------
					destination data prefix
						2.2.2.2/32
						action > drop

			then import our route policy and named it make package on policies

			*on localize policy mode we cann't set more than one policy
			now import to devices
				vmanage > config > template > devices template > vedge > additional tempelates > policy

				just deploye one policy
				here upload on all fabric devices must load on one device and it doesn't applied
					on vpn1 and gig0/1 of vedge1 must set this
						vmanage > config > template > features > vedge1-vpn1-g1
							ingress acl ipv4
								turn on object (ds)

				doesn't give access to make optional attributes must create new template
				just access and deploye one global policy
				setting acls doesn't exist has no effect on devices but in sdwan opposit ios must create another object for route policy couldn't set random name 

				show policy access-list-counters

		here applied qos policy
			create qos map
			map some class forwarding and classifications to hardware queue

			bandwidth percent
			buffer percent
			schedule
			drop

			qos map on interface
				vmanage > config > policy > localize policy
					acl > ipv4
						sequence
							dscp > 46 (just use decimal format)
							action > accept
							calss > defined before (real-time)
							-------------------------------
							destination data prefix
							2.2.2.2/32
							action > accept
							class > bulk

							disable variables didn't  get troubles
							-------------------------------
							q2
							class > other
							action > accept

				vmanage > config > template > devices template > vedge > additional tempelates > policy

				vmanage > config > policy > localize policy > forward class > qos map
					just has one q0
					depend on our classess defined on last part (bulk,real-time....),cerate queues

					q1
						bulk
						bandwidth > 50%
						buffer > 50%
						schedule > wrr
						drop > random early detection
					----------
					q2
						others
						bandwidth > 30%
						buffer > 30%
						schedule > wrr
						drop > tail drop
					----------
					q0
						default queue
						use burst and llq

					now set on interface
						vmanage > config > template > devices > vedge1-vpn0-g1

							vmanage > config > template > features > vedge1-vpn0-g1
								acl
									qos map
										itemname (g)

								*must applied the clod qos and cloud qos reserved
									vmanage > config > policy > localize > forward class/qos
										import last policy

										define parent policy then enable our netflow cloud qos contain cloud qos and cloud qos service 

										*now we can set on devices template and features template

										sh policy qos-map-info
										sh policy qos-schedule-info
**************************************
security
	security localize(36)
		application aware enterprise firewall (zbfw)
		amp
		ips ids
		url and dns filter security
		cloud security
		vmanage authentication and authorization
		web layer security
		iaas \ saas

		wanedges has these attributes and free

		cloud security
			whole fabric recieve traffic then check it
			then give access have flow

		integrated security
			all security components were inside datacenter

		regional hub
			vnf is tools for security stack and shared point 

		ios xe sdwan use nbar and perfect checking

		segment aware

		application aware enterprise firewall (zbfw)
 		is one of the powerfull stetfull inspection model firiewall

 		vmanage > config > policy > centrall
 			site
 				branch1 > 1 
 				---------
 				brach2 > 2
 			----------------
 			topology
 				custom
 					sequence
 						tloc
 							site
 								b1
 								action > accept
 								----------------
 								b2
 								action > accept
 						----------
 						route
 							vpn list
 								1
 									action > accept
 									export to vpn2
 								-------
 								2
 									action > accept
 									export to vpn2

 			apply on inboud direction and branches site

 		ssh -l admin -v2 -vrf 1 192.168.2.2

 		vmanage > config > security 
 			custom
 				firewall policy
 					zone pair (equal vpn)

 					*must set source to destination flow checking
 						vmanage > config > security > custom option > zone
 							zone a > vpn11

 					default action 
 						drop
 						inspect (use for return access/permission option)
 						pass
 					-----------------
 						protocol 
 							ssh 
 								action > inspect
 							-----------
 							telnet
 								action > drop

 				just use next option till apply them

 		vmanage > config > template > devices > cedge > additional tempelates
 			security policy

 		vmanage > monitor > network > security monitor / realtime > policy zone pair

 		sh tech-support
 	**************************************
 	IDS & IPS (37)
 		base on signatures can detect attacks in basic version

 		modular os
 			application service container
 				some services could be injected to os also capable run some application 

 				ios xe sdwan 
 					first install modules then other features get install like security	

 			kvm (kernel virtual machine)(more portable)
 			lxc (linux virtual container)(higher performance)(like ios xe sdwan)
 				use direct hardware resources

 		sdwan ips is snort engine and depends on update
 		cisco talos and manual updates
 		daily get update

 		after recieve update must restart our snort engine

 		our packet are fail open means if restart engine access packets to transmit don't drop them
 		opposit fail to open we have fail to close means drop packets if get restart

 		vmanage > config > security > ips
 			x
 			signature set
 				balance (cvs 9 and higher)
 					black list
 					exploit kit
 					malware cnc
 					sql injection

 				connectivity (cvs 10 and higher)

 				security (cvs 8 and higher)
 					app detection
 					black list
 					exploit kit
 					malware cnc
 					sql injection

 		sdwan has many images must be update
 			vmanage > maintenance > software repository > virtual machines > vmanage
 				upload images

 		first ips event use these policies and machines

 		inspector
 			detection > ids > log and alert

 			prevention > ips > log and block

 		signature white list 
 		client level

 		target vpn

 		fail mode option (clos is recommended)

 		vmanage > config > template > devices > additional tempelates > security policy (works on cedges)

 		vmanage > setting > smart account credential
 			recive update images on dynamic way
 	**************************************
 	url filter(38)
 		snort checks the payload on http and https packet

 		has 82 categories

 		use white and black lists
 		reputation score (-10 to +10)

 		need uploaded and updated images

 		vmanage > config > security >
 			custom
 				url filter
 					x
 						web categories > blcoj (cats)
 						web reputation
 							trust worthy
 							low risk
 							high risk (clos high risks)
 							suspicious
 							moderate

 							advance 
 								white list
 								black list

 						replacement message
 						alerts
 							black list
 							white list
 							reputation

 						redirect block
 							content body
 							default headers

 					apply policy
 						syslog
 						fail mode (close)
 						target vpn

 		vmanage > config > template > devices > additional tempelates > security policy

 		vmanage > monitor > network > security > url filter
 	**************************************
 	amp (39)
 		protect our network from spyware ....
 		cisco amp clod (north america, asia, europe)

 		upload images to devices

 		vmanage > config > security > amp
 			ampx
 				amp cloud region
 					eu (europe)
 					nam (north america)
 					apjc (asia)(select this)

 				match 
 					all vpn
 					custom vpn

 				alert log level
 					critical
 					warning

 				file analyze
 					tag cloud region
 						nam
 						eu

 					file type
 					alert log level
 					syslog adn failur mode

 					*must add api
 			applied on devices template
 			vmanage > config > template > devices > additional tempelates > security policy

 			vmanage > monitor > network > security > amp		
 	**************************************
 	dns web layer security (40)
 		cisco umbrella cloud
 			a cloud for security services
 			has portal access us define adn report newest Vulnerabilities
 			like live black list

 			our dns request queries goes to wedge
 			if set umbrella, first check the request then intecpt them
 			check on database
 			our responses were from umbrella
 			if were good send ip if were bad block them base on scores

 			show us ip block page

 			if were so-so  make proxy on ip and send ip (make man in the middle)

 			end user has connectivity with umbrella like ssl and https (dnscrypt or edns (extention)(change size of dns queries))(replace requests)

 			buy sdwan , api , umbrella license

 			vmanage > config > security 
 				list
 					domain list > b.com

 				umbrella
 					cisco token (custom options)

 				dns
 					x
 					match 
 						all vpn or custom vpn
 					----------------------------
 					dns server
 						umbrella (outside iran use this)
 						----------
 						custom
 					---------------
 					dnscrypt
 					---------
 					local domain bypass
 						add our ou domain to bypass process from umbrella (trusted)

 			vmanage > config > template > devices > additional tempelates > security policy
 	**************************************
 	cloud security(41)
 		cloud deliverd firewall
 			till here were integrated model and local firewall deployement
 			on wedges we have overhad and much load on forwarding and cpu usage
 			no higher throughput

 			here can use cloud for most security service
 			use ipsecor gre for connection
 			connect our security center service to edges
 			or use regional hub (shared point) 

 			cisco edges use ipsec
 			vedeg use ipsec and gre
 	**************************************
 	vmanage hardening (42)
 		local authentication role base access controll (rbac)

 		user group 
 			netadmin (full)
 			operator (readonly)
 			basic

 		vmanage > administration > manager user
 			user definition

 		vmanage > administration > vpn segment and vpn group (users access which vpn)

 		vmanage > config > template > features > vmanage
 			aaa
 				authentication order
 				fallback
 				radius & tacacs+

 				vmanage > config > template > devices
 					basic info
 						aaa  on vmanage

 		sso
 			vmanage > administration > setting > identity provider
 				enable and upload

 				saml v2 and meta data smust be uploaded
 	**************************************
 	Policy Administration, Activation, and Enforcement – Order of Operations
 		vmanage send traffic to 
 			vsmart
 				yaml base and netconf

 				enrollment and vpn memebership
 				application aware routing policy

 			wedge
 				yaml base and netconf

 				localize policy
 				security policy

 		vsmart to wedge
 			omp

 				application aware routing
 				data policy

 		packet frowarding steps
 			ip destination lookup
 			----------------------------
 			local ingress policy
 				policing
 				admission controll
 				classification and marking
 			----------------------------
 			centralize application routing policy
 			sla base path selection
 			----------------------------
 			centralize data policy
 				policing
 				classification and marking
 				path selection
 				forward error correction
 				services
 			----------------------------
 			routing and forwarding
 				topology
 				driven forwarding
 			----------------------------
 			security policy
 				firewall
 				ips
 				url filter
 				amp
 			----------------------------
 			local egress policy
 				remaining
 				acl
 				policing
 			----------------------------
 			queueing and schedule
 				llq
 				shape
 				wr
 				congestion avoidance
 			----------------------------
**************************************
tloc extention
	dedicate interface (44)
		high availablity on branches
		if one link get cut off there is no assurance to redirect traffic
		better connect wedges tlocs and transports to each other physically

		indirect connection on tlocs and special types
		works unidirectional and bidirectional

		each tloc extention use color and specify tunnel

		isp make ebgp on wedges and outside routers
		wedge2 connect to internet and isp with accessability from wedge1 and dtloc extentions

		here our wedges can set default gateway on another wedge not isp

		must create a seperated tempelates
			vmanage > config > template 
				features
					vedge1-1
						system
							site-id (ds)
							system-ip (ds)
							host-name(ds)

							baudrate 9600(g)
						-----------
						vpn
							0
								ipv4
									prefix > 0.0.0.0 (g)
									nexthop >100.0.0.100 (g) (for vedge1-2 use 101.0.0.100)
							----------------------
							512
							----------------------
							1
								omp advertise connected (g)
						-----------
						vpn interface
							vpn0-int-g0/0
								shut > no (g)
								interface > gig0/0 (g)
								ip address > static 100.0.0.1/24 (g)
								color > biz-internet (g)
								tunnel interface > on (g)
								allow service > all (g)
								restrict > on (g)
							--------------
							vpn512-ethernet
								shut > no (g)
								interface > fastether0/0 (g)
								ip address > dynamic (g)
							--------------
							vpn1-int-g0/3
								shut > no (g)
								interface > gig0/3 (g)
								ip address > static 172.16.0.1/30 (g)
						-----------
						ospf
							router id > 1.0.1.1 (g)
							redistribute > omp (g)
							area > 0 and gig0/3 (lan) (g)
					-----------------------------
					vedge1-2
						system
							site-id (ds)
							system-ip (ds)
							host-name(ds)

							baudrate 9600(g)
						-----------
						vpn
							0
								ipv4
									prefix > 0.0.0.0 (g)
									nexthop >101.0.0.100 (g)
							----------------------
							512
							----------------------
							1
								omp advertise connected (g)
						-----------
						vpn interface
							vpn0-int-g0/0
								shut > no (g)
								interface > gig0/0 (g)
								ip address > static 101.0.0.1/24 (g)
								color > mpls (g)
								tunnel interface > on (g)
								allow service > all (g)
								restrict > on (g)
							--------------
							vpn512-ethernet
								shut > no (g)
								interface > fastether0/0 (g)
								ip address > dynamic (g)
							--------------
							vpn1-int-g0/3
								shut > no (g)
								interface > gig0/3 (g)
								ip address > static 172.16.0.4/30 (g)
						-----------
						ospf
							router id > 1.0.1.2 (g)
							redistribute > omp (g)
							area > 0 and gig0/3 (lan) (g)
				----------------------
				devices
					bind all profiles ....

		same config for site2 with 2 wedge  (vedge1-2 and vedge1-1)
		here we need extend our tlocs
			vmanage > config > template
				features
					vedge1-2
						vpn interfaces
							shut > no (g)
							interface > add interface connected to redundant mode and directly to wedges2 (used for tloc extention) 
							ip static > 172.18.0.1/30
							advance > tloc extention (gig 0/0) (isp or transport interface name)

					*on another interface must bring up tunnel

						vpn interfaces (on vpn0)(config like direct connection)
							shut > no (g)
							interface > gig0/2 (used for tloc extention) 
							ip static > 172.18.0.2/30
							color > biz-internet (g)
							restrict > on (g)
							tunnel interface > on (g)

					*use these for vedge1-1 on mpls and diffrent ip addressing
			here we have duplicated default gateway last part on vpn0 our gateway was 100.0.0.100 and 101.0.0.100 but here for tloc extention must set another and diffrent gateway 172.17.0.0/30
			and 172.18.0.0/30

			cause we use private ip must negotiate on bgp to make advertise 

				vedge1-1
					vpn0-bgp
						bgp
							shut > no (g)
							as > 1 (g)
							neighbor
								100.0.0.100 (g) (for vedge1-2 use 101.0.0.101)
								remote-as > 1 (g)

							network
								172.18.0.0/30 (g) (for vedge1-2 use 172.17.0.0/30)

			then add bgp on device attachment and use their own interfaces
	**************************************
	subinterface (45)
		configs are on ios xe

		on vpn0
			vmanage > config > template
				features
					csr1kv
						vpn
							0
								static route
									prefix > 0.0.0.0
									nexthop > 100.0.0.100 and 172.19.0.2
						--------------
						vpn interfaces
							vpn0-int-g0/1-internet
								interface > g0/1 (g)
								shut > no (g)
								ip static > 100.0.0.2/24
								tunnel interface > on (g)
								color > biz-internet (g)
								allow service > all (g)
								restrict > on (g)
							-------------------
							vpn0-int-g0/2
								shut > no (g)
								interface > g0/2 (g)(our subinterface added)
								advance > ip mtu > 1504 (g) 
							-------------------
							vpn0-int-g0/2.1-mpls
								shut > no (g)
								interface > g0/2.1 (detect our subinterface)
								ip > static > 172.19.0.1/30		
								tunnel interface > on (g)
								color > mpls (g)
								allow service > all (g)
								restrict > on (g)		
								-------------------
							vpn0-int-g0/3
								shut > no (g)
								interface > g0/3 (g)(our subinterface added)
								advance > ip mtu > 1504 (g) 
							-------------------
							vpn0-int-g0/3.1-mpls
								shut > no (g)
								interface > g0/3.1 (detect our subinterface)
								ip > static > 172.19.0.1/30		
								color > mpls (g)
								allow service > all (g)
								restrict > on (g)	
								advance > tloc extention (gig 0/1) (isp or transport interface name)	
						--------------
						apply on bgp ospf with as 1 and neighbor 100.0.0.100
				-------------
				devices

				apply on devices

				use these for another wedge
				on vedge and vpn 0 couldn't have best perform  must use another vpn like 1 2 ... better use cedge
	**************************************
	tunnel group (46)
		some scenarios we have one tloc and use one of them to negotiate with transports
		each tloc has ow color and interface 
		if wannna use 2 interface on one tloc mens aggregate them or redundant them what should we do?
		eahc color were used once

		tunnel group designed for each group shown with same number our interfaces will be insert (color doesn't matter here)

		vmanage > config > templates
			features
				vedge-cloud
					vpn
						0
						ipv4 prefix > 0.0.0.0
						nexthop > internet , mpls , private (select name of interfaces and must add all colors here)
						--------------
						512
					---------------
					vpn interface
						vpn0-int-g0/1-internet
							int g0/1 (ds)
							shu > no (g)
							tunnel interface > on (g)
							color > biz-internet (g)
							allow service > all (g)
						--------------------------
						vpn0-int-g0/2-mpls
							int g0/2 (ds)
							shu > no (g)
							tunnel interface > on (g)
							color > mpls (g)
							allow service > all (g)
						--------------------------
						vpn0-int-g0/3-private
							int g0/3 (ds)
							shu > no (g)
							tunnel interface > on (g)
							color > private (g)
							allow service > all (g)
						---------------------------
						vpn0-int-g0/0-internet
							tunnel section
								group > 1 (here we create aggregate mode with number 1 and same values for interfaces were 1)
						---------------------------
						vpn512-interface
							int > ether (g)
							shut > no (g)
							ip > dynaimc (g)

						*ommit restrict mode cause we have group tunneling
**************************************
cloud on ramp
	saas(47)
		each application were on cloud instead of implementing datacenter
		what should we do > works like hub&spoke or direct connection to applications or ...

		set direct internet access (dia)
		our control will be bold

		our prerequisites
			performance
			reliable connection
			flexible
			scalable

		cloud on ramp divided on 
			management or visibility
			performance

		on traditional way use mpls and vpls to make faster connection each branch usually have specific link to dc
		here on sdwan access with dia

		quality of experience (qoe)
			ranks are
				10-8 means green and good
				8-5 means yellow and so-so
				0-5 means red and bad

				measure applications and their servers status also check the link state to present really good experience

				use probe here to measure them

				per application anlysis is the best perform

		mix status check with applications then detect egress path if need applicaion reliable connection must use transport x if need other applicaions must use link y

		saas dia methods
			direct internet access
			gateway site 
				a sharesingle point each client reached then access the internet

		if recieve same qoe or not qualified qoe use normal route and default gateway

		send http request on intervals then set scores per applicaion and per link
		delay and jitter detect by probes

		inside bfd our clinets collect all information about times and delays between application and gateway (wedge) they didn't use the second model (setting gateway for inappropriate qoe)

		http ping > each 1 second advertise

		buckets > 10 seconds must sent and waited for replies

		4 subuckets were needed 

		4*30 means 120 seconds or 2 minutes must checks and wait for it

		average of all these subbuckets determine delay and loss

		avg (vqoe(loss)+vqoe(latency)) > vqoe

		vqoe(loss) > (baseline loss / actual loss)*100
		vqoe(latency) > (baseline latency / actual loss)*100
		
		actual loss means loss percentage over 12 minutes

		with deep packet inspection detect applications
		each sdwan version has their specify database
		each wedge traffic flow were inspect and control to rach saas application

		first checking interval score all application then check dia links state

		if our wedge has many links to reach the internet
			first model access with internet link and directly
			second model access datacenter with mpls link adn datacenter has internet access

			here our deep packet inspection or dpi distribute packets and detect link quality fro saas applications

			first packets were initial flow to check links if were stable and suitable use them if were not after analyzing change the links

			cache table > flowtable info

			in dpi we need symetric transmission 
			symetric detection will be on dns queries and saas return many ip address so we set score on sdwan for them 
			after this we compare all scores select the best of them

			dns server on vpn 0 

			here dpi behave lke a proxy and checks client dns queries

			after checking redirect flows

		dia  versions on vedge 16.3 and cedge 17.2.1 also we have gateway site on vedge with version 17.1

		must set default gaeway to reach the internet

		better set nat and dns servers on vpn0

		vmanage >  administration  >setting > cloud on ramp fpr saas

		on vmanage dashboard select cloud on ramp 

		vmanage > config > cloud on ramp > manage cor saas
			applicayion
			vpn
			dia site > attached > vedeg1 (who connect directly)
				also set interface on it

			gateway site > attach  > vedge2
	**************************************
	iaas(48)
		instead of managing and installing hardware and infrastructures inside lan , use cloud

		sdwan version 20 performs on azure and aws

		virtual private cloud (vpc)	
			some containers have many informations about systems on the cloud

			between vpcs have connections and self generated data can be transmit 
			they are not transite area or forwarder

			from branches to app and vpc we have simple ipsec
			cause we doesn't have sdwan feature create a ipsec connection to use bfd and manage them on our onpermis sdwan

			standard ipsec were terminated on cloud cause without sdwan works (without bfd and cloud on ramp)

			apply our cloud account inside the vmanage

			aws say vpc and azure say vnet

			on vpc gateways use 2 wedge on same account but in seperated mode also make redundant for connection

			between vpc virtual gateway and wedge gateway cloud must make bgp connection (not necessary)(inside aws)

		works with default route and make bgp connection also redistribute all omp between branches and aws cloud edge

		if were single (segmentation or vpn) be easy and redistribute 
		if use many vpns must seperated them for each subnets

		we need token license to add wedge clouds on sdwan

		has some tempelates better use cisco configs recommends

		identity and management aslo access key used for credential on vmanage connect to aws base on api call 

		vmanage > config > cloud on ramp > iaas
			error on adding wedge with template

			transit vpc page
				region
				wedge version (ios)
				size of transite wedge (aws resources reservation)
				maz host vpc per device pair

				then add 2 wedge in this page

				on advance part vpc cidr (default is 10.0.0.0/16) 
				then discover host vpc findd and map them to sdwan

				vmanage > config > cloud on ramp for iaas
					click on aws and select mapping
						host vpc
							unmapped host vpc 
								discover host vpc
								credential

				on list map vpc get deployed on some specific thing
	**************************************
	colocation(49)
		network optimisation on sdwan
		wan optimise
		loadbalance
		all features on one point like aggregation point

		backhauling means first dc checks the decide what to do (centralize fashion or access)

		with backhauling user experience get trouble
		this method is not so applicable

		distributed access give internet to clients without reaching the datacenter
		this model cause attacks and security issues

		many of them couldn't provide and consider our security and levels
		can set mpls and internet links for direct communication but have challenges

		regional access > cisco sdwan cloud on ramp colocation > optimize access
			better quality for users

			instead of hub&spoke create colocation point that is middle point access for services and security

			this colocation point recieved all users traffic then forward them to dc or another places

			colocation point using the most higher bandwidth to internet and clouds
			also provide security 

			service insertion and inteligent routing determine destination for better timing also qoe get modified

			this colocation point has seperation mechanism on resources and users

		scalable arch (csp (cisco cloud service platform))
		security
		performance agility (fast config)
		flexible (vnf(virtual network function))(has some containers)
		cost saving
		malware protection
		security policies
		decrease dos attacks 

		sdwan can detect these colocations and routing faster on them
		with dpi (deep packet inspection) handel users traffic redirect to colocation or another places
		must define our services with service chain

		wedges must negotiate with vsmart and transfer these services to them
		vsmart after see requested services  can change wedges nexthop to nearest colocation

		our classified traffics on dpi transfer to colocation
		inside colocation with service chain get redirect and process

		after reaching traffic on colocation must check mechanisms on cisco colocation cloud on ramp

		this method doesn't need cisco sdwan but has advantages on aggregation

		normal traffics can be redirect to colocation
		sites and brnaches communications could be on 

		all vnf are virtual
		cisco 9k catalyst 9500 su1 xe 16.9.1 and above
		csp 5444 (powerfull)
		44core
		4.8 tb hdd
		8*10gig nic
		192 gig ram
		zero touch provisioning

		policy
			central (vsmart)
			data (wedge)

		per vpn is different

		when forwarde or redirect traffic to colocation can use special vlan or vnf

		how apply colocation
			first of all need bandwidth
			physical location must be nearest
			our colocation has best access to clouds (direct acces aws(direct come), azure (express route))
			has good storage power supply

			use site to site vpn reaching the region of cloud every where

		on branches we have 2 link on wedges one of them connected to colocation another connection is direct on cloud
		when need saas must use scores and qoe with probe on links
		usaully connect on colocation but some times we have direct access like before 

		csp in colocation manage automatic reboots and active passive or redundant , failovers

		must use many csp and same service chain

		service chain design based on traffic source and destination and many vnf  

		better use cisco 9k catalyst in vmanage
		add them with smart account
		like wan edge list

		vmanage > config > devices
		vmanage > config > cloud on ramp for colocation
			we can use cluster and provisioning

			name
			site id > 10
			location > tehran
			setting
				ntp
				syslog

			credential
				user pass for cluster
				admin
				admin

			resource pool
				ip range
				management
				dtls tunnel ip > system ip on sdwan (when a vnf need joining to device)
				service chain vlan pool (each services has ip)(1100-1200)
				vnf management pool ip (inside csp we have management part)
				vnf data plane ip pool (ip pools on vnf connection inside vmanage use this)
				management subnet gateway
				management mask
				switch pnp ip (our switch must config on another part this is useful for switches management, normally used vnf ip pool range management colocation config manager)
					9500 could be in sdwan with this setting

			after activation need one hour
			must reboot them (find pnp), recieve ip from dhcp with optioni 43
				option 43
					ascii 
						5A;B2;K4;I10.114.11.40;J9191

			port 9191
			then insert vnf (img > .qcow2)

			vmanage > maintenance > software repository > virtual image
				custom images

		must fetch traffic to devices

		vmanage > configs > policy > centrall
			application > google
			traffic role > traffic data
				nat
					x
					sequence type
						service chain andcustom

						sequence role
							application > google app
							action >  accept
							services > firewall
								vpn1,ipsec,biz-internet,tloc=1.1.1.1 (colocation tloc)

			apply on devices like before
			set on device template

			vmanage > config > cloud on ramp for colocation
				cluster
				service group
					services chain and config vnf
						add
							bandwidth
							vlan
							monitoring
							service chain 
								custom

			vmanage > monitor > network > colocate cluster
**************************************
datacenter design
	direct transport side(50)
		we have bgp from lan to core 
		access to controllers
		on firewalls we have nat

		vmanage > config > template 
			features
				vedge-cloud
					system
						system-ip , site-id , hostname (ds)
						baudrate > 9600 (g)

					vpn
						0
						route prefix 0.0.0.0
						nexthop
							10.0.1.21
							10.0.1.25
						---------------
						1
						ospf
							router-id 1.0.1.1
							redist > omp
							area 0 > g0/2-g0/3
						---------------
						512
					---------------------------
					vpn interfaces
						vpn0-int-g0/0-internet
							int > g0/0 (g)
							shut > no (g)
							ip  > 10.0.1.25/30
							restrict > on (g)
							color > biz-internet (g)
							tunnel interface > on (g)
							allow service > all (g)
						-------------------------------
						vpn0-int-g0/1-mpls
							int > g0/1 (g)
							shut > no (g)
							ip  > 10.0.1.21/30
							restrict > on (g)
							color > mpls (g)
							tunnel interface > on (g)
							allow service > all (g)
						-------------------------------
						vpn1-int-g0/0-internet(core side interface)
							shut > no (g)
							core side interface
							ip > 10.0.1.5/30 (g)
						-------------------------------
						vpn512-interface
							shut > no (g)
							ip > dynamic (g)
							interface > ether 1 (g)
					---------------------------

			-----------
			devices

			use device role
	**************************************
	indirect transport side(51)
		here we have same project like direct transport side but use dot1q
		necessarily not connect to edge 

		vlan 10 > till router
		valn 20 > service side
		vlan 30 > router and switch core connection

		between router 1 and sw has ospf
		vlan 30, loop0, are in are 0

		on cedges must define subintefaces
			vpn 0 and vpn0-int-g0/2.20
				interface > gig0/2.20
				tunnel interface > on (g)
			----------------------------
			vpn 1 and vpn1-int-g0/2.10
				has ospf with omp advertise and connected for vpn1
				has no tunnel interfaces

			also has vpn 512
	**************************************
	loop back tloc unbind mode - standard(52)
		on transport side we have 3 mpls link lan side is ok
		on each interface must use one color and each color must be on one interface
		problem solve is loopback interafce or unbind

		loop0 (has mpls color) on 3 interface with same color

		bind mode each loopback interface attached to one interface
		special target on designs

		some situations need redirect traffic to gig0/0 not more
		our interface will be used for single target 

		vmanage > config > template
			features	
				vedge-cloud
					system
						site-id, system-ip , hostname (ds)
						baudrate > 9600 (g)
					---------------------------------
					vpn
						0
						route prefix >  0.0.0.0/0 (g) 
						nexthop > (ds)

						*must set default route for each underlay
						---
						1
						---
						2
						---
						512
					---------------------------------
					vpn interfaces
						vpn0-int-bind-g0/123-internet
							shut > no (g)
							interface > loop0 (g)
							ip static > (ds)
							restrict > on
							tunnel interface > yes (g)
							color > biz-internet
							allow service > all (g)
						---------------------------------
						vpn0-int-gig0/1-bind-loop0
							shut > no (g)
							interface > gig0/1 (g)
							ip static > (ds)

							*we just turn it on and set tunnels on parent interface (loop0) not here
						---------------------------------
						vpn0-int-gig0/2-bind-loop0
							shut > no (g)
							interface > gig0/2 (g)
							ip static > (ds)
						---------------------------------
						vpn0-int-gig0/3-bind-loop0
							shut > no (g)
							interface > gig0/3 (g)
							ip static > (ds)
			-----------
			devices

			our application aware routing is not applicable

			per tunnel has bfd and determine on echos to qoe adn link quality checks
	**************************************
	loop back tloc design - bind mode (53)
		each loop back interface bind to physical interface 

		vmanage > config > template
			features
				vedge-cloud
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					----------------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g)
						nexthop > 100.0.0.1 , 101.0.0.1 , 102.0.0.1 (g)
						-----------------
						1
						-----------------
						2
						-----------------
						512
					----------------------------
					vpn interfaces
						vpn0-int-gig0/0-internet
							shut > no (g)
							interface > gig 0/0 (g)
							ip static > (ds)
						------------------------
						vpn0-int-gig0/1-lte
							shut > no (g)
							interface > gig 0/1 (g)
							ip static > (ds)
						-------------------------
						vpn0-int-gig0/2-mpls
							shut > no (g)
							interface > gig 0/2 (g)
							ip static > (ds)
						--------------------------
						vpn0-int-bind-g0/012-x
							shut > no (g)
							interface > loop0 (g)
							ip static > (ds)
							tunnel interface > yes (g)
							restrict > on (g)
							allow service > all (g)

							advance > tunnel > bind loop tunnel (g)
					----------------------------
					bgp
						shut > no (g)
						as number > 1 (g)
						network prefix > (ds)
						neighbor > 100.0.0.100 (g)
						remote-as > 100 (g)

			if couldn't connect to lte must roll back them till 5 minutes
	**************************************
	service side connectivity (54)
		must integrate some features on mixinig sdwan and traditional

		service side connectivity
			usually set bgp connection between datacenter and sdwan with mixinig target
			better decrease redistribute method 
			also better make connection between wedges and ce routers (customer edge)
**************************************
branch design
	single sdwan edge - l2 (site2)(55)
		complete ce replacement single wedge
			disconnect all nodes with old router the replace with wedge and trasfer to sdwan

			best practices are on integrated model
			mix traditional and sdwan technologies
			after mixing can trasfer to new topology

			better works on dual stack method

		cedge has different config

		vmanage > config > template
			features
				csr1kv
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					--------------------------------
					vpn
						0
						route prefix > 0.0.0.0/0
						nexthop > 100.0.0.100 , 101.0.0.100
						--------------
						1
						advertise omp > connected (g)
					--------------------------------
					vpn interfaces
						vpn0-int-g0/1-internet
							shut > no (g)
							color > biz-internet
							tunnel interface > yes (g)
							allow service > all (g)
							restrict > on (g)
							interface > g0/1
							ip static > 100.0.0.2/24 (g)
						--------------
						vpn0-int-g0/2-mpls
							shut > no (g)
							color > biz-internet
							tunnel interface > yes (g)
							allow service > all (g)
							restrict > on (g)
							interface > g0/2
							ip static > 101.0.0.2/24 (g)

						*we need subinterface on cedges and inside vpn0 not vpn1
						---------------
						vpn1-int-g0/3
							shut > no (g)
							interface > g0/3 (g)
							advance > mtu > 1504 (g)(can change subinterface mtu on lower value)
						----------------
						vpn1-subint-g0/3.10
							shut > no (g)
							interface > g0/3.10 (g)
							ip static > 172.16.10.1/24 (g)
						----------------
						vpn1-subint-g0/3.20
							shut > no (g)
							interface > g0/3.20 (g)
							ip static > 172.17.10.1/24 (g)

						*doesn't need tunnel interfaces
			---------------
			devices

			on vpn 0 with 3 interface recieve data then on vpn1 (service vpn) and sub interfaces handle them
			for subinterface better use cedge
	**************************************
	single sdwan edge - l3 (site3) (56)
		ce router completly get replace
		between core switch on site3 and wedge we have ospf

		vmanage > config > template
			features
				vedge-cloud
					system 
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					----------------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g)
						nexthop > 100.0.0.100 , 101.0.0.100 (g)
						---------------------
						1
						advertise omp > connected
						---------------------
						512
					-----------------------------
					vpn interfaces
						vpn0-int-g0/0-internet
							shut > no (g)
							interface > g0/0
							ip static > 100.0.0.2/24 (g)
							tunnel interface > on (g)
							color > biz-internet (g)
							restrict > on (g)
							allow service > all (g)
						-----------------------
						vpn0-int-g0/1-mpls
							shut > no (g)
							interface > g0/1
							ip static > 101.0.0.2/24 (g)
							tunnel interface > on (g)
							color > mpls(g)
							restrict > on (g)
							allow service > all (g)
						-----------------------
						vpn1-int-g0/2
							shut > no (g)
							interface > g0/2
							ip static > 172.16.1.1/30 (g)
						-----------------------
						vpn512-ethernet
							shut > no (g)
							interface > g0/3
							ip dynamic >  (g)
					----------------------------
					ospf
						router id > 1.0.3.1 (g)
						redistribute > omp
						area > 0 int g0/2 (g)
			------------
			devices
	**************************************
	dual sdwan edge - l2 branch + vrrp and tclo extention (site1) (57)
		cause we have vedge use one vlan

		vmanage > config > template
			features
				vedge1
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g) 
						nexthop > 101.0.0.100 , 103.0.0.2 (g)

						*cause we have vrrp and use tloc extention for redundant connectivity must set these additional address
						------------------
						1
						advertise omp > connected (g)
					------------------
					vpn interfaces
						vpn0-int-g0/0-mpls
							shut > no (g)
							interface > g0/0 (g)
							ip static > 101.0.0.1/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > mpls
						------------------
						vpn0-int-g0/2-vrrp-internet
							shut > no (g)
							interface > g0/2 (g)
							ip static > 103.0.0.1/30 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > biz-internet
						------------------
						vpn0-int-g0/1-vrrp-tloc
							shut > no (g)
							interface > g0/1 (g)
							ip static > 102.0.0.1/30 (g)

							advance > tloc extention > gig0/0 (g)
						------------------
						vpn1-int-g0/3-vrrp
							shut > no (g)
							interface > gig0/3 (g)
							ip static > 172.16.2.1/24 (g)
							vrrp
								group > 1 (g)
								priority > 200 (g)
								track > yes (g)
								vip > 172.16.2.100 (g)
					------------------
					bgp
						vpn0-bgp
						as > 1 (g)
						network > 102.0.0.0/30 (on tloc extention ip and need to visibility)
						neighbor > 101.0.0.100 remote as 1 (must neighbor with mpls core device)
				--------------------
				vedge2
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g) 
						nexthop > 101.0.0.100 , 102.0.0.1 (g)

						*cause we have vrrp and use tloc extention for redundant connectivity must set these additional address
						------------------
						1
						advertise omp > connected (g)
					------------------
					vpn interfaces
						vpn0-int-g0/0-internet
							shut > no (g)
							interface > g0/0 (g)
							ip static > 100.0.0.2/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > biz-internet
						------------------
						vpn0-int-g0/1-vrrp-mpls
							shut > no (g)
							interface > g0/1 (g)
							ip static > 102.0.0.2/30 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > mpls
							------------------
							vpn0-int-g0/2-vrrp-tloc
								shut > no (g)
								interface > g0/2 (g)
								ip static > 103.0.0.2/30 (g)

								advance > tloc extention > gig0/0 (g)
							-----------------------
							vpn1-int-g0/3-vrrp
								shut > no (g)
								interface > gig0/3 (g)
								ip static > 172.16.3.1/24 (g)
								vrrp
									group > 1 (g)
									priority > 150 (g)
									track > yes (g)
									vip > 172.16.2.100 (g)
					------------------
					bgp
						vpn0-bgp
						as > 1 (g)
						network > 103.0.0.0/30 (on tloc extention ip and need to visibility)
						neighbor > 100.0.0.100 remote as 1 (must neighbor with mpls core device)
			-------------------
			devies 

		on vpn0 and transport and management must set devices template with bgp adn tloc extention on core switch til make bgp conenction
	**************************************
	dual sdwan edge l3 branch – private tloc extension and nat (site4) (58)
		between devices we use 172.16.xy.0/24
		each device has ospf on loopback with siwtch number 

		on vedge 4 we have bgp to advertise ip private (it's no matter on vedge 5)
		between	vedge 4 and 5 
			192.168.45.0/24 and 192.168.54.0/24

		vmanage > config > templates
			features
				vedge4
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g) 
						nexthop > 101.0.0.100 , 192.168.54.5/24 (g)

						*cause front side has nat
						------------------
						1
					------------------
					vpn interfaces
						vpn0-int-g0/0-mpls
							shut > no (g)
							interface > g0/0 (g)
							ip static > 101.0.0.4/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > mpls
						------------------
						vpn0-int-g0/2-internet
							shut > no (g)
							interface > g0/2 (g)
							ip static > 192.168.54.4/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > biz-internet

							cause we use nat on tloc extentions in vedge 5 use tunnel interface
						------------------
						vpn0-int-g0/1-tloc
							shut > no (g)
							interface > g0/1 (g)
							ip static > 192.168.45.4/24 (g)

							advance > tloc extention > gig0/0 (g)
					------------------
						vpn1-int-g0/3
							shut > no (g)
							interface > gig0/3 (g)
							ip static > (ds)
					------------------
					bgp
						vpn0-bgp
						as > 1 (g)
						network > 192.168.45.0/24 (on tloc extention ip and need to visibility)
						neighbor > 101.0.0.100 remote as 1 (must neighbor with mpls core device)
					-----------------
					ospf
						route id > (ds)
						redistribute > omp (g)
						area > 0 , int g0/3 , int gig0/4 (g)
				--------------------
				vedge5
					system
						site-id , system-ip , hostname (ds)
						baudrate > 9600 (g)
					------------------
					vpn
						0
						route prefix > 0.0.0.0/0 (g) 
						nexthop > 100.0.0.100 , 192.168.45.4/24 (g)

						*cause nat
						------------------
						1
					------------------
					vpn interfaces
						vpn0-int-g0/0-internet
							shut > no (g)
							interface > g0/0 (g)
							ip static > 100.0.0.4/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > biz-internet
							nat > enable (g)
							refresh > out (g)

							*cause we have private ip and no routing protocol must use nat 
						------------------
						vpn0-int-g0/1-mpls
							shut > no (g)
							interface > g0/1 (g)
							ip static > 192.168.45.5/24 (g)
							tunnel interface > on (g)
							restrict > on (g)
							allow service > all (g)
							color > mpls
						------------------
						vpn0-int-g0/2--tloc
							shut > no (g)
							interface > g0/2 (g)
							ip static > 192.168.54.5/24 (g)

							advance > tloc extention > gig0/0 (g)
						-----------------------
						vpn1-int-g0/3
							shut > no (g)
							interface > gig0/3 (g)
							ip static > (ds)
					------------------
					ospf
						route id > (ds)
						redistribute > omp (g)
						area > 0 , int g0/3 , int gig0/4 (g)
			---------------
			devices

			*on site 1 we had 2 links with tlocs and public ip address so on bgp negotiate just advertise the networks here need nat
			tloc make redundant interface 
			in scenario 1 we set public ip between 2 wedges based o this method tx\rx were fine
			in the last scenario we had private ip between 2 wedges (vedge4 and 5)
			private ip address are not routeable on internet so need nat
			on tloc extentions we use one link and ip range like 192.168.45.0/24 could be transfer on mpls
			here we need run tunnel interface on vedge 5 gig0/1 and ip 192.168.45.5 to negotiate in mpls links
			if request from  192.168.54.4 and vedge 4 with ip private wanna go internet could not be routeable
			we don't set tunnels interface on private ip
			here use nat
			why say these? cause bgp just run on vedge 4 and must be on it not vedge 5 to advertise our private range
	**************************************
	overlay only – overlay and backup – full overlay underlay(60)
		physical connectivity on traditional network design
		newest connectivity contain a overlay on traditional equipments
		with this fabric set ipsec connection to another devices
		on legacy mode devices we use ipsec anad redirect traffic to datacenter
		but our sdwan works on it way methods

		over with underlay backup
			sites on legacy mode and updated to wedge 
			first works on overlay and master vrrp if get problem works on legacy router and bgp must set aas-path-prepend

			cost default route
				on lan side routing protocols must set higher value on wedges (ibgp)
				between legacy router and wedges must have link and filtered roles

			full overlay and underlay integrated
				special targets on underlay then goes to overlay if needed some services
					wedge > overlay
					underlay > legacies

				also loop back and tloc design bind mode can help us	
**************************************
provisioning sdwan controller in private cloud(61)
	version 20.4.1

	vbound > define devices and authentication
	vmanage > control all parameter
	vsmart > routing, policy, key exchange

	on first initializein cli, disable our tunnel interfaces then on vmanage define again and enable it

	new way to config them
		out of band management
			useful in real world
			on another interfaces we start configuration then another interface start tunnel

			for oob (out of band management) doesn't need things
				vbond 
					show centrall connections 
					#before vbond configuration this command might help us vbond = vedges

					show orchestration 
					show control local-properties
					#root ca chain status
					#cisco and symantec not our organization unit

		oob config steps
			depployement fro vmange , vsamrt , vbond
			bootstrap and config for vmange controller(set and restart to see system ready)
				wan edges list = smart account and organization name
				system-ip
				ip vpn512 (must config this at the first steps)
				ip vpn0 (transport)
				site id
				vbond address

				conf t
				system
				site-id 100
				system-ip 1.0.0.3
				organization-name "shayan.sdwan"
				ntp server 1.2.3.4
				vbond 10.0.0.2
				commit

				vpn 512
				interface ether 0/1
				ip address 10.0.0.3 255.255.255.0
				no sh
				commit

				show centrall local-properties

				vpn512
				ip route 192.168.1.0 255.255.255.0 10.0.0.1 
				commit

				sh ip route vpn512

			sw
				hostname switch
				int ether 0/1 
				no switchport
				ip address 192.168.1.50 255.255.255.0

				int vlan 1
				no sh
				ip address 10.0.0.1 255.255.255.0

			vmanage > administration > setting 
				organization unit
				vbond
				central certificate authority
					enterprise root cert (always use this)
						on vmanage
							show certificate instead
							show certificate root-ca-chain

					manual (mix of symantec and cisco account)
						symantec (real world)
						cisco (cisco cloud)

				*vbond for certificate deployement must use vpn0 and vmanage
				here we set vpn512 for it
				better set vpn0
				cause we use tunnel on vpn0

			vmanage
				vpn0
				interface ether 0/1
				ip address 100.0.0.6 255.255.255.252
				no sh
				ip route 0.0.0.0 0.0.0.0 100.0.0.5 (internet router or isp)
				commit

				vbond 100.0.0.10

			on vmanage panel must set and define controllers with ip public
			on vpn0 disable tunnel interface 
			in vbond must set ipsec tunnels to negotiate with vedge (just add command)
			
			activate tunnel interface on vmanage
			vmanage > white list > vbond
				serial numbers
				certificate
				organization unit

				2 white list
					wedge list
					controller list

					serial number on controllers
					organization unit name

			vmanage > administration  > setting
				smart account credential
				pnp connect symantec

				csone case 
					buy tag
					on cisco must approve us or on symantec					

				normal mode cert
					doesn't need define root cer and csone case
**************************************
labratory provisioning and initial configuration and tloc extension (62)
	in this scenario we have 2 vbond must define a-record on dns server

	vmanage cli
		config terminal
		system
		system-ip 1.1.1.6
		site-id 1
		organization-name "shayan.sdwan"
		clock timezone asia/tehran
		vbond 1.1.1.3
		vpn0
		host vbond ip 10.1.1.2 10.1.1.3
		#create ip and a-record on router

		ip route 0.0.0.0 0.0.0.0 10.1.1.1

		interface ether 0
		no tunnel-interface
		ip address 10.1.1.6 255.255.255.0
		no shutdown
		commit

	vbond-1 cli
		config terminal
		system
		system-ip 1.1.1.2
		site-id 1
		organization-name "shayan.sdwan"
		clock timezone asia/tehran
		vbond 1.1.1.3 local
		vpn0
		host vbond ip 10.1.1.2 10.1.1.3
		#create ip and a-record on router

		ip route 0.0.0.0 0.0.0.0 10.1.1.1

		interface ether 0
		no tunnel-interface
		ip address 10.1.1.62 255.255.255.0
		no shutdown
		commit
		#after config be carefull turning on tunnel interface, color (biz-internet) , allow service (all), encapsulation (ipsec)

		#vbond-2 use these configs with another ip address on interface

	vsmart cli
		config terminal
		system
		system-ip 1.1.1.5
		site-id 1
		organization-name "shayan.sdwan"
		clock timezone asia/tehran
		vbond 1.1.1.3
		vpn0
		host vbond ip 10.1.1.2 10.1.1.3
		#create ip and a-record on router

		ip route 0.0.0.0 0.0.0.0 10.1.1.1

		interface ether 0
		no tunnel-interface
		ip address 10.1.1.5 255.255.255.0
		no shutdown
		commit

	dc1-vedge1 cli
		config terminal
		system
		system-ip 10.0.10.1
		site-id 10
		organization-name "shayan.sdwan"
		clock timezone asia/tehran
		vbond 1.1.1.3
		vpn0
		host vbond ip 10.1.1.2 10.1.1.3

		ip route 0.0.0.0 0.0.0.0 2.0.0.1
		ip route 0.0.0.0 0.0.0.0 3.0.0.1

		interface gig0/0
		ip address 2.0.10.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color biz-internet
		encapsulation ipsec

		interface gig0/1
		ip address 3.0.10.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color mpls restrict
		encapsulation ipsec

		commit

		#on dc1 has vedge2 set these values on vedge2 but change ip address

	br1-vedge1 cli
		config terminal
		system
		system-ip 10.0.101.1
		site-id 101
		organization-name "shayan.sdwan"
		clock timezone asia/tehran
		vbond 1.1.1.3
		vpn0
		host vbond ip 10.1.1.2 10.1.1.3

		ip route 0.0.0.0 0.0.0.0 3.0.0.1

		interface gig0/1
		ip address 3.0.101.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color mpls restrict
		encapsulation ipsec

		commit

		#between br1-vedge1 and br1-vedge2 have tloc extention
		#same config on br1-vedge2 with diffrent ip addressing and no restrict config for tlocs

	br2-vedge1
		here we have 3 interface just use restrict on mpls 
		all configs are same like above but diffrent ip addressing
		set default gateways

	br3-cedge1
		config-transaction 
		system
		system-ip 10.0.103.1
		organization-name "shayan.sdwan"
		site-id 103
		vbond 1.1.1.3
		
		host vbond ip 10.1.1.2 10.1.1.3

		ip route 0.0.0.0 0.0.0.0 2.0.0.1
		ip route 0.0.0.0 0.0.0.0 3.0.0.1
		ip route 0.0.0.0 0.0.0.0 4.0.0.1

		interface gig0/1
		ip address 2.0.103.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color biz-internet
		encapsulation ipsec

		interface gig0/2
		ip address 3.0.103.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color mpls restrict
		encapsulation ipsec

		interface gig0/3
		ip address 4.0.103.1 255.255.0.0
		no shutdown
		tunnel-interface
		allow-service all
		color lte
		encapsulation ipsec

		int tun0
		ip unnumber gig0/1
		tunnel source gig0/1
		tunnel mode sdwan

		int tun1
		ip unnumber gig0/2
		tunnel source gig0/2
		tunnel mode sdwan

		int tun2
		ip unnumber gig0/3
		tunnel source gig0/3
		tunnel mode sdwan

		username admin priviledge 15 secret 123

		commit

		#on core switch we have controller interconnectivity base on default route, of course we have colocation connectivity with those default routes 
		#inside colocation we have static route to 10.1.1.0/24

	vmanage > config > devices > wan edge list
		generate bootstrap config
			vedge

	on windows server we add ca role then run winscp on each vedge to add certificate and ca
	on cedges must use scp command copy and request mechanism happen
	after config must check bfd session will be 22 
	if use tlco extention see 27 bfd session

	bfd count
		mpls to mpls > 7 (before tloc were 6)
		lte to lte > 3 (were 2)
		internet to internet > 7 (were 6)
		lte to internet > 3 (were 2)
		internet to lte > 7 (were 6)

	between br1-vedge1 and br1-vedge2 have 3 interfaces 
		br1-vedge1 > internet and lte
		br1-vedge2 >mpls

	172.17.0.0/30 (1 or 2 used on br1-vedge1 and br1-vedge2)
	172.17.0.4/30 (5 and 6 used on br1-vedge1 and br1-vedge2)
	172.17.0.8/30 (9 and 10 used on br1-vedge1 and br1-vedge2)

	br1-vedge1	
		vmanage > config > template
			features
				br1-vedge1
					system
						site-id > 101 (g)
						system-ip > (ds)
						host-name > (ds)

						baudrate > 9600 (g)
						timezone > asia/tehran (g)
					----------------
					 vpn
					 	0
					 	dns > hostname > vbond > 10.1.1.3 , 10.1.1.2
					 	route
					 		prefix > 0.0.0.0/0
					 		nexthop > 3.0.0.1 (mpls) , 172.17.0.2/30 (tloc) , 172..17 0.6/30 (tloc)
					 	------------------
					 	512
					----------------
					vpn interfaces
						vpn0-int-g0/1-internet
							shut> no (g)
							interface > gig0/1 (g)
							ip static > 172.17.0.1/30
							tunnel interface > on (g)
							color > biz-internet (g)
							allow service > all (g)

							*no restrict 
						---------------------------
						vpn0-int-g0/2-mpls
							shut> no (g)
							interface > gig0/2 (g)
							ip static > 3.0.101.1/16
							tunnel interface > on (g)
							color > mpls (g)
							allow service > all (g)
							restrict > on (g)
						---------------------------
						vpn0-int-g0/3-lte
							shut> no (g)
							interface > gig0/3 (g)
							ip static > 172.17.0.5/30
							tunnel interface > on (g)
							color > lte (g)
							allow service > all (g)

							*no restrict 
						---------------------------
						vpn0-tloc-g0/4-g0/1
							shut > no (g)
							interface > gig 0/4 (g)
							ip static > 172.17.0.9/30 (g)
							advance > tloc extention > g0/1 (g)
						---------------------------
						vpn512-ether
							shut > no (g)
							interface > ether1 (g)
							ip dynamic > (g)						
					-----------------
					bgp
						shutdown > on (g)
						as > 101 (g)
						network > 172.17.0.8/30 (g)
						neighbor > 3.0.0.1 remote as 100 (g)

						#on vpn0

						#on colocation route
							router bgp 100
							neighbor 3.0.101.1 remote-as 101
							neighbor 2.0.1.1 remote-as 1
							neighbor 2.0.101.2 remote-as 1

						#on core-switch connect to 172.17.0.8/30 (br1-vedge1)
				------------------
				br1-vedge2
					system
						site-id > 101 (g)
						system-ip > (ds)
						host-name > (ds)

						baudrate > 9600 (g)
						timezone > asia/tehran (g)
					----------------
					 vpn
					 	0
					 	dns > hostname > vbond > 10.1.1.3 , 10.1.1.2
					 	route
					 		prefix > 0.0.0.0/0
					 		nexthop > 3.0.0.1 (mpls) , 4.0.0.1 (lte) , 172..17 0.6/30 (tloc)
					 	------------------
					 	512
					----------------
					vpn interfaces
						vpn0-int-g0/1-internet
							shut> no (g)
							interface > gig0/1 (g)
							ip static > 2.0.101.2/16 (g)
							tunnel interface > on (g)
							color > biz-internet (g)
							allow service > all (g)

							*no restrict 
						---------------------------
						vpn0-int-g0/2-mpls
							shut> no (g)
							interface > gig0/2 (g)
							ip static > 172.17.0.10/30 (g)
							tunnel interface > on (g)
							color > mpls (g)
							allow service > all (g)
							restrict > on (g)
						---------------------------
						vpn0-int-g0/3-lte
							shut> no (g)
							interface > gig0/3 (g)
							ip static > 4.0.101.2/16 (g)
							tunnel interface > on (g)
							color > lte (g)
							allow service > all (g)

							*no restrict 
						---------------------------
						vpn0-tloc-g0/3-g0/2
							shut > no (g)
							interface > gig 0/3 (g)
							ip static > 172.17.0.6/30 (g)
							advance > tloc extention > g0/2 (g)

							*tloc extention on lte interface
						---------------------------
						vpn0-tloc-g0/4-g0/1
							shut > no (g)
							interface > gig 0/3 (g)
							ip static > 172.17.0.2/30 (g)
							advance > tloc extention > g0/2 (g)

							*tloc extention on biz-internet interface
						---------------------------
						vpn512-ether
							shut > no (g)
							interface > ether1 (g)
							ip dynamic > (g)						
					-----------------
					bgp
						shutdown > on (g)
						as > 101 (g)
						network > 172.17.0.0/30, 172.17.0.4/30 (g)
						neighbor > 2.0.0.1 remote as 1 (g)

						#on vpn0

						#on colocation route
							router bgp 100
							neighbor 4.0.101.2 remote-as 101
							neighbor 2.0.1.1 remote-as 1
							neighbor 2.0.101.2 remote-as 1

						#on core-switch connect to 172.17.0.0/30  and 172.17.0.4/30 (br1-vedge2)
			---------------
			devices
**************************************
labratory service vpn configuration, routing protocol, copy and change tempelates (63)
	datacenter side config
		*datacenter doesn't have bgp
		on lan side we have ospf and vpn1
		must define vpn512 on all vedge1

		vmanage > config > template
			features
				vedge
					system 
						site-id > 10 (g)
						system-ip > (ds)
						host-name > (ds)
						timezone > asia/tehran (g)
						baudrate > 9600 (g)
					------------
					vpn
						0
							route prefix > 0.0.0.0/0
							nexthop > 2.0.0.1 (internet) , 3.0.0.1 (mpls) (g)
							dns > hostname > vbond > 10.1.1.2 , 10.1.1.3 (g)
						--------------
						512
						--------------
						1
							advertise omp > connected (g)
					------------
					vpn interfaces
						vpn0-int-g0/1-internet
							shut > no (g)
							interface > gig 0/1 (g)
							ip static > 2.0.10.1/16 (g)
							tunnel interface > on (g)
							color > biz-internet (g)
							allow service > all (g)

							*no restrict
						--------------------------
						vpn0-int-g0/2-mpls
							shut > no (g)
							interface > gig 0/2 (g)
							ip static > 3.0.10.1/16 (g)
							tunnel interface > on (g)
							color > mpls(g)
							allow service > all (g)
							restrict > on (g)
						--------------------------
						vpn1-int-g0/3
							shut > no (g)
							interface > gig 0/3 (g)
							ip static > 10.1.10.2/30 (g)
						--------------------------
						vpn512-ether
							shut > no (g)
							interface > ether 0/1 (g)
							ip dynamic > (g)
					------------
					ospf
						route-id > 10.0.10.1 (like system-ip)
						area > 0 > int gig0/1
						redistribute > omp
			-----------------------
			devices

			*use these for dc2 with diffrent values on ip address (just need copy template and rename then customize them) 
**************************************
labratory use case1 isolating remote branches from each other (64)	
	in this condition we have 2 decision
		1-filter all attributes

		2- just advertise dc route and dc tloc

	vmanage > config > policy > centrall
		site
			dc > 10,20
			branch > 100 to 200
		----------------
		topology and vpn memebership
			custom
				x
					default action > reject
					------------------------
					sequence
						tloc
							site dc
							action > accept
						--------------
						route
							site > dc
							action > accept

		appply policy 
			topology
				hub&spoke
					add sites and  direction outbound
**************************************
labratory use case2 enabling branch-to-branch communication through data centers (65)
	cause we have 2 datacenter behave like anycats(nearest is beeter for responses)

	vmanage > config > template 
		features
			dc1-vedge1
				vpn
					1
						route prefix > 0.0.0.0/0 (g)
						nexthop > null 0 (discard route) (g)

						advertise omp > static (useful jus on sending)(g)

						*here we inject our omp fromo dc1 to network
	-------------------------------------------
	vmanage > config > policy > centrall
		*each vedge on dc1 has 2 tlocs  must be defined with their colors and tlocs 
		*used for nexthop for branches

		topology
			hub&spoke
				custom
					x
						sequence
							tlco
								site > dc
								action > accept
							----------				
							route
								site > dc
								action > accept
								------------
								site > branch
								action > accept
								replace tloc > dc
**************************************
labratory use case3 traffic engineering at sites with multiple routers (66)
	tloc preferences

		show omp tlocs
			find preferences

	apply policies to vsmart
		inbound (more general)
			recieve updates 
			commit policies then continue working (scope on whole fabric)

		outbound (more details)
			select best path then continue
			scope on specific wedge

	originator in sdwan are devices that geneate packets (system-ip)

	vmanage > config > policy > centrall
		topology
			custom
				x
					default action > accept (cause we set inbound direction)(on inboud direction policy must use accept for default action)
					-------------------------
					tloc
						originator
							10.0.10.1
							action > accept
							preferences > 500
							-------------
							10.0.10.2
							action > accept
							preferences > 400
							-------------
							10.0.20.1
							action > accept
							preferences > 500
							-------------
							10.0.20.2
							action > accept
							preferences > 400

		apply on inboud direction for dc 
		on allow service in vpn ether and advance section set prefences weight
		default behave on tempelates and custom behave on policy
**************************************
labratory use case4 preferring regional data centers for internet access (67)
	better conect our users branch to datacenter and hub then access internet

	vmanage > config > template
		features
			dc1-vedge1
				vpn
					1
						route prefix > 0.0.0.0/0 (g)
						nexthop > null 0 (g)

						advertise omp > static

					*instead 8 default routes sends 2
					by default use 4 maximum path in rib and with vsmart distribute them
					vsmart send tloc 500
			-------------
			dc1-vedge2
				vpn
					1
						route prefix > 0.0.0.0/0 (g)
						nexthop > null 0 (g)

						advertise omp > static
			-------------
			vsmart
				omp
					number of path advertise per prefix > 8

					*here advertise 8 
					cause we have different values on preferences tloc doesn't need use this
						vedge > omp
							ecmp limit > 8

	our branches might use any datacenter so limit them
		we set b2b through datacenter but didn't say which datacenter

		vmanage > config > policy > centrall
			site
				dc-uroupe > 2
				-----------------
				dc-america > 1
				-----------------
				br-uroupe > 101 , 102
				-----------------
				br-america > 103
			----------------------
			prefix
				default route > 0.0.0.0/0
			----------------------
			topology
				custom
					x
						default action > reject
						-----------
						sequence
							tloc
								site > dc (total values)
								action > action
							---------------
							route
								site > dc-america
								prefix > 0.0.0.0/0
								action > accept
								preferences > 100 (omp preferences
								----------------------------
								site > dc-america , dc-uroupe
								action > accept

								*without preferences
								----------------------------
								br-america
								action > accept
								tloc > dc
									better set tlocs on seperated mode and parts
									cause we have 2 tlocs on each vedge better set general values

								replace with dc-america

			apply on outbound and branch sites
**************************************
labratory use case5 regional mesh networks (68)
	on sending traffic from paris to berlin has problem must use minimum hop counts to reaching applications
	must has direct connections except datacenter connections

	dc tloc and branch tlocs on same country wedges get advertised on own wedges

	vmanage > config > policy > centrall
		topology
			custom
				europe-branche
					tloc
						dc
						accept
						------
						europe-brnaches
						accept
					-------------
					route
						europe-dc
						default route >  0.0.0.0/0
						preferences > 100
						accept
						------------
						dc
						accept
						----------
						europe-branche
						accept
						-------
						america
						tloc > dc tloc or amerca tloc
						accept
				-----------------------
				amerca-branches
					tloc
						dc
						accept
						------
						amrecia
						accept
					-------------
					route
						america
						default route >  0.0.0.0/0
						preferences > 100
						accept
						------------
						dc
						accept
						----------
						america
						accept
						-------
						europe
						tloc > dc tloc or europe-dc tloc
						accept


	apply on outbound direction and america + europe-branches  site
	bfd session were 12 now get 17 (casue use mpls , lte and internet links)
**************************************
labratory – use case6- enforcing security perimeters with service insertion (69)
	our services must be local on datacenter and use gre tunenl

	show omp services 
	show omp services family ipv4 service firewall

	our branches onlast part used america dc tlocs and negotiate with europe (dc or branch)
	here instead of using america tlocs must use firewall services

	before use dc tlocs of europe and america but here use firewall routes and services

	make a copy from policies created and inject our services
	apply on europe and america sites also outbound direction

	sh omp tlocs
**************************************
labratory use case7 isolating guest users from the corporate wan (70)
	on datacenter add our guests vpn
	better disable our redional mesh and firewall services
**************************************
labratory use case 8- creating different network topologies per segment (71)
	if need management on exported tlocs must use inbound direction policy deployement

	route leaking

	vmanage > config > policy > centrall
		custom
			x
				sequence
					route
						default action > accept
						---------------------
						vpn > 101
						accept
						export to > vpn service (101 , 102)
						omp tag > 101
						-------------------
						vpn > 102
						accept
						export to > service vpn (101 , 102)
						omp tag > 102
					
		apply on inboud direction and branch sites
**************************************
labratory use case 9 creating extranets and access to shared services (72)
	for guest dia must use nat to define routes also use in other vpns need negotiate on vpn 0
**************************************
labratory use case 10 direct internet access for guest users (73)
	show policy service-path vpn20 int loop1 source-ip 10.3.102.10 destination-ip 8.8.8.8 protocol 1 all
	#service-path used for simulate traffics

	show sdwan policy data-policy-filter

	on nat vpn inside the data policy we have fallback mechanism
	fallback help us if lost one link make nat and connection to another interface for dia
	*fallback used all sdwan fabric as backup

	on application aware routing can help us on destination ip

	with tloc seting or taging on egress port can manage egress application outgoing interface

	like policy base routing
	tloc list means our nexthop (if get loss didn't switch to another)
	local tloc means our tloc or our policy (get loss change interface and switched)

	show interface description
**************************************
labratory use case 13 protecting corporate users with a cloud-delivered firewall (76)
	csico unbrella and secure internet gateway (sig)

	users data checks on services insertion then goes out

	centralize data policy and remote conenction (ipsec)
	checks traffic didn't change routes
	ipsec tunnel between unbrella adn wedge
	need unbrella account

	on vpn0
		show run von 0 int ipsec
		pvn0
		interface ipsec 1
		ip address 10.255.255.253 255.255.255.252
		tunnel source int gig0/0
		tunnel destination 146.112.82.8
		ike
		version 2
		rekey 28800
		cipher-suite aes 256-cbc-sha1
		group 14
		authen-type
		pre-shared
		pre-shared sec 123
		local-id sdwan
		remote-id 146.112.82.8
		ipsec
		rekey
		3600
		replay-window 518
		cipher-suite aes 256-gcm
		perfect-forward-recovery none
		no shutdown

		sh interface ipsec 1

		vpn1
		service idp interfcae ipsec 1
		int gig0/3
		ip address 10.1.102.1 255.255.255.252
		no shutdown

		ip route 10.1.102.64 255.255.255.192 10.1.102.2

	vmanage > config > template
		features
			vedge
				vpn interfaces
					ipsec 

		then apply data policy
		direct deploye on wedge
		first insert our acceptable policies then rejected
			default action > accept

		outbound direction policy and manipulate
		opposite control policy deployed on vsmart then on wedges we use wedge first
