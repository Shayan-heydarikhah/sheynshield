MPLS
####################################################################################################################################################################################
MPLS L3VPN :
	multiprotocol labe switching
	is newest type of traffic forwarding 
	label use resources in minimal mode

	fundumental :
		sh ip route 1.1.1.1 255.255.255.255
		sh ip cef 1.1.1.1 255.255.255.255 

		in mpls use cef mechanism to forwad traffics 
		high rate forwarding

		forwarding procedure
			process switch
				were checked each line and routes in rib with software mode 
			
			fast switching
				in first steps and newest request use process switch mechanism 
				then make cache list of requests 
				for next requests use cache and process faster than before
				if get ommit recalculate them
				works on demand mode if have traffic request had cache

			cisco express forward 
				use asiic chips with high rate processing
				application specific ic > fib (forwrding information base)
				a hardware in routing
				fib use beneficial part of rib
					network + mask + egress port + next hop

				ingress traffic match fasster than before with these parameters

		control plane 
			ios > routing table > routing protocol > static route > connected interfaces

		in fib we have sorted items and use adjencency table
		in ip forwarding use cef and fib
		in mpls just use cef (must enable and by default is enable)

		mpls ip (by default is enable on cisco)
		interface fast 0/0/
		mpls ip (active on interface)

		sh mpls interfaces

		after these commands say ldp is up in logs

		routing table contain > fib + control plane
		in rib use :
			ep(egress port)
			nh(nexthop)

			r1							r2 							r3
		net 192.168.3.0/24 		net 192.168.3.0/24			net 192.168.3.0/24
		nh 192.168.12.2/24 		nh 192.168.23.3/24 			nh 192.168.3.0/24 (connected)
		ep fast 0/0 			ep fast 0/0 				ep fast 0/0 or loop0

		mpls create a new tabel on rib (controll plane) with lib term
		lib (lable information base) means a repository of labels
		lib generate a number on specific label range for each rib records with random mechanism
		use this generated labels for mpls

		0-(2^20-1) > range of mpls
		starts from 16 to above number
		1-16 used for special tasks

		in mpls and lib use :
			ep(egress port)
			nh(nexthop)
			lib(label for mpls)

			r1								r2 								r3
		net a > 192.168.3.0/24 		net a >  192.168.3.0/24			net a > 192.168.3.0/24
		nh 192.168.12.2/24 			nh 192.168.23.3/24 				nh 192.168.3.0/24 (connected)
		ep fast 0/0 				ep fast 0/0 					ep fast 0/0 or loop0
		lib 100						lib 200 						lib 300 to net 

		each router advertise selected label to side routers
		each router use random label for each netwrk and subnet on their rib
		after these steps advertise each label to another like before

		r1 in controll plane use r2 rid with label 200
			r1
				rib
					net a > 192.168.3.0/24
					net b > 192.168.1.0/24
					192.168.12.1/24 local
				lib
					net a > 200
					net b > 100
					192.168.12.1 > 100 (local)

		in comparission with fib on labels we have some modifications :
			r1
				net a > fast 0/0 > 192.168.12.2 > 200

			r2
				net a > fast 0/0 > 192.168.23.3 > 300

		label distribution protocol (ldp) > advertise labels on network mpls bases
		lfib (label forwarding information base)
			in fib we have nwe item like this
			with this make labels

			r1
				inbound lable 		outbound label 		nexthop 			egress port 
				100					200					192.168.12.2 		fast 0/0

			r2
				inbound lable 		outbound label 		nexthop 			egress port 
				200					300					192.168.23.3 		fast 0/0

		if recieve ip first of all checks with fib and forward it in normal procedure
		if recieve ip + label checks with lfib then mpls and labels procedure get start
		in r1 checks label and destination then goes to lib and lfib
		r1 use push methood to inject records at first time
		r2 use swap method on next times

		push in first days > impuse
		pop in first days > dispose

		lib is on rib and use lable advertisement with ldp
		then lfib and fib set direction and forwarding path

		php > penaltimate hop popping
			in r3 we have connected interfacce and destination of all net a requests
			so must pop our label and forward it on ip base model then transfer it to users
			inject null label on connected networks and destination requests
			r3 set values and transfer it to r2 and say after this don't use lfib and lib
			just use fib to forward data and requests faster than before

			 in default mechanism this actions happened

			 in one left to alst router we don't use label in lfib for some networks
			 use label number 3 to define pop also use for implicit numm or imp-null

		how set lable range for router
			mpls label range 100 199 (use in global)

			sh mpls ldp binding 192.168.3.0/24

		in mpls use lsr term for router that forward labels (label switch router)
		ler (label edge router)
		edge lsr > incoming router for mpls in isp (like pe (provider edge))

		lsr is like rid with recieve and transmit labels task

		sh ip cef 192.168.3.0 255.255.255.0
		sh mpls forwarding-table 192.168.3.0 255.255.255.0
			show us inbound label > outbound labe > egress port > newtwork + prefix + tunnel id

		if doesn't have label pount (shoot) to fib

		shim header > is like layer in 2.5

		mpls header
			32 bit
				label > 20 bit
					use 0-(2^20-1)
					ecept 0-16

				exp > 3 bit
					exprimental
						000(0) , 001(1) , 010(2) , ... 7
						use for qos in mpls te and mpls qos

				s (bos) > 1 bit
					bottom of stack
					some of the mpls packets contain 2 or 3 or many labels with this can define last item
					if set value on 1 means 

				ttl > 8 bit

		in 1980 cisco invent mpls tag
		switching invent in 2000 in associate with ietf then change name to label switching

		cisco used tag distribution protocol (get depricated)

		lsp means label switch path

		in mpls we use some modes like :
			cell
				atm
				48byte data
				5byte header
				53byte mpls label instead of vpi used

			frame
				all another components of networks be on this 

		mpls ttl beahviour
			in src to dst path we use label swithcing path (lsp)
			all packets use fec or forwarding equivalence class means behave like normal and same
			advertise all labeles to all nodes with fec for each interface
			this method calld per platform label space
			works in frame mode mpls
			on router just have a one label and use it to negotiate
			use one diffrent label for each diffrent network

			per interface label space works in cell mode mpls
				use one diffrent label for each interface also has many repositories for each of them
				more secure but resource intensive
				set diffrent fec for each label

		in mpls use ldpid instead of rid
		use by lsr
		ldpid > label space
		if already set must use force command to change it
		if doesn't define which interface must be use in default set :
			biggest loopback ip address with up status
			or use biggest ip address on another interface type with up sattus

			mpls ldp router-id loop0 force

		routers send ldp + ldpid in negotiations

		lib on r1
			net a
			label 100 local
			ldp id 1.1.1.1:0

		lib on r2
			net a
			label 200 local
			ldp id 2.2.2.2:0

		1.1.1.1:0 or 2.2.2.2:0 > :0 means per platform label space (define like remote lsr)

			sh mpls forwarding-table
			sh mpls ldp binding

		here we have problem on null 0 and php mechanism also have  trouble in 3.3.3.3:0 and nexthop ip address 192.168.23.3
			in this situation we have mechanism in ldp 
			ldp contain interfaces and ip of rid interfaces

		ttl in ipv6 and mpls are hop limit base

		ttl in ip and ttl in mpls have diffrences > in mpls we reduce ttl value and in php mechanism change mpls ttl to ip ttl
		when arrived to destination we pushed labels that mean have a label or add more labels on it

		ldp :
			label distribution protocol
			in mpls we have isis or ospf 
			in ospf we can set mpls activation

				ip cef
				mpls ip
	
				sh mpls interface

				router ospf 1 
				mpls ldp autoconfig area 0
	
				interface fast 0/0
				no mpls ldp igp autoconfig
	
				sh mpls interface fas 0/0 details (which of them is active shown here, details means how get actie)
				sh mpls ldp neighbor details 
				sh mpls ldp discovery details (hello)
	
				interface fas 0/0
				mpls ip
				router ospf 1
				mpls ldp autoconfig area 0

			label distribution protocol
				ip over mpls
					tdp
					ldp
				bgp (mpls l3vpn)(best choice)
				rsvp (mpls te)(resource reservation protocol)(needs isis or ospf)

			ospf and isis just transmit these labels no manage to generate or managing interferences

			between 2 router or many routers advertise all labels
			befor ldp advertisement must create lsr neighborship (udp+tcp 646) and diffrent with multicast in igp neighboring
			224.0.02(ldp) > discover and learn or advertise

			has some parameters in negotiations but here in ldp just use transport ip
			one ip like above works between 2 routers and make tcp session on them and ake random session like random session connection between r1 and r2

			use udp for neighbor discovery
			use tcp to transfer data
			ip used for session is rid r mpls rid (ldp rid)

				sh ip int fast 0/0
				mpl ldp router-id 1.2.3.4

			from bigger rid make connection to smaller rid (connection order)
			ospf 224.0.0.5
			mpls 224.0.0.2

				sh tcp brief

			all transport ip address must be reachable or injected in ospf

				sh mpls ldp discovery
					 (on which interface with which attribute the are negotiate and transferring hello)
				sh mpls ldp neighbor (bounded lsr address +ldp id + tcp connectors and discovery)

			ldp find lsr in network and make ldp mapping 
			make connection and sen keepalive message
			also use notification to manage
			phases :
				session establishment
				session discovery

			order of ldp
				udp hello ldp
				tcp session establishment
				initial message (negotiate and ack)
				keepalive
				address labeling request

				or
				hello from bigger ip to 224.0.0.2:646
				transport ip address
				tcp (syn + syn-ack + ack)
				tcp establish
				ldp neighbor (initialization message)(hold time > 180 (defualt) 1/3 of hold is hello)
				ldp session
				advertise label (mapping ldp message)
				keepalive(each 60 seconds)
				notification 
					error and fatal also they are too important
					simple discovery
						touble in ldp version also tlv problem
						pdu got destroy

					if use same vendor for mpls see fewer troubles

					clear mpls ldp neighbor 1.2.3.4

			keepalive is most importand point of neighboring
			in initial message we have tlv
				timer value
				ldp mtheods

			in hold time comparission we sshould use lower value

			ldp methods :
				frame mode
					unsolicited down stream (ud)
					means yo always access to lables and networks
					here we have smart mechanism also use more resources cause we have replication path to fast convergence
					in mpls l3vpn must use ud model

				cell mode
					down stream on demand (dod)
					all lsr must request the ldp
					advertise main label and path to other

				sh mpls ldp parameters
					fetures + basic + autoconfig +tcp md5 rollover
					ldp v1 	keepalive 	discovery hello (5 and 15)
					ldp initial 
					max backoff (routers try neighborship in teratel 15/120 seconds)

				mpls ldp backoff 20 200 (20 second wait)

		authentication
			we have roll over in mpls without discontinuation of the neighborship
				sh mpls ldp neighbor details

			in default we have not actie authentication
			we have not required option to disconnect neighbors

				mpls ldp pass required

			if have neighbor get disconnected

				access list 1 permit 1.1.1.1
				mpls ldp pass required for 1 (acl number)

				* each router use 1.1.1.1 must use authentication

				mpls ldp neighbor 1.1.1.1 pass 123

				sh mpls ldp neighbor details (which	method used)

			if set on required our session get restart

			use per neighbor

				access-list 1 permit 1.1.1.1
				access-list 1 permit 2.2.2.2

				mpls ldp pass option 1 for 1 pass123
				mpls ldp pass option 2 for 2 pass456

			on another routers can use these ways :
				mlps ldp neighbor 2.2.2.2 pass pass123
					or
				access-list 1 permit 2.2.2.2
				mpls ldp pass option 1 for1 pass456

			mpls ldp pass fallback 123 (if can't use other set this , don't disconnect neighborship but after restarting need password)

			mpls ldp pass fallback option 1 for 1 (offer this way)

			sh mpls ldp neighbor details (stale state say you are on last tcp session)

			loss less ldp authentication > loss less md5 session authentication

			in change password we have stale state need restrat + clear mpls to make authentication 

			rollover
				change time for changing the next password is a fixed time and then it changes
					mpls ldp pass rollover duration 1 (minute)

			in authentication we use neighbor authentication and fallback

				mpls ldp logging pass config rate limit 1-60
				mpls ldp logging pass rollover rate limit 1-60

				tdp doesn't have authentication

		must disable mpls on client side interfaces
		all ospf routers
			mpls ip
			ip cef
			
			router ospf 1
			mpls ldp autoconfig
			int range fas 0/0-0/1
			ip ospf 1 are 0

			int serial0/0/0(client side)
			no mpls ldp igp autoconfig

		in default we have ldp method on mpls but
			int fas 0/0
			mpls label protocol ldp

		if nee advertise all loopback interfaces on mpls vpn
			access-list 1 permit 1.1.1.1
			access-list 1 permit 2.2.2.2

			no mpls ldp advertise-label (don't advertise label just select and learn)

			sh mpls ldp binding

			mpls ldp advertise-label for 1 (each network in acl 1 can be advertise and learn)

		if need effects our learning to other router :
			access-list 2 permit 1.1.1.1
			mpls ldp advertise-label for 1 to 2 (send label to acl number 2)

		if need set which range learn or accept :
			access-list 1 permit 5.5.5.5
			mpls ldp neighbor 2.2.2.2  label accept 1 (acl number)

		we can make connection between 2 seperated router with mpls like virtual link in ospf
			in this method we use target session and udp is not aplicated

			mpls ldp neighbor 5.5.5.5 targeted (sttaiic neighborship)

			in this model instead of default timers we have :
				10 second and hold infinit or 90 seconds

			access-list 1 permit 1.1.1.1
			mpls ldp discovery targeted-hello accept from 1(whith this can set starter could be another router altho can set recieve from who)

		ttl propagation :
			we can hide our labels and infrastructure from users
			just see pe ip and another pe 

			no mpls ip propagation-ttl (if we trace must see path also if propagation-ttl for inter isp routers our topology get fuck)
			no mpls ip propagation-ttl frward (which packets you sent)
			mpls ip propagation-ttl (on pe)

		ldp session protection
			use targeted session
			if had triangle topology and use ospf and mpls
			one link get ommit then return to circute our ldp get trouble and need more time to recover so we should use targeted session and session protection to make this recovery fast

			to make fast recovery must use static neighborship
				access-list 1 permit 2.2.2.2
				mpls ldp session protection for 1 duration 1

			now use saved and buffered lables in cache and make neighborship faster than before
			should use this method for each router in triangle topology

				mpls ldp discovery targeted-hello accept

				sh mpls ldp neighbor details
					if be 84600 second oon unreachable mode take in protection

				sh mpls ldp binding
					till 84600 sseconds save labels of unreachable router

		mpls ldp discovery hello-interval 3
		mpls ldp discovery hold-interval 9

			works on lower value

		mpls ldp discovery target-hello interval 5
		mpls ldp discovery target-hold interval 15

		mpls ldp hold-time 150
		mpls ldp backoff 15 200

		mpls ldp discovery transport-address loop0 (change rid in tranmission)

		active ssession protection on all routers that have many ways to each other

		frame mode use liberal label retuntion (llr)
			all labels save in lib for each network
			kind of maintenance for the next hop change time, don't use the next next hop

		cell mode use conservation label retuntion (clr)
			just nexthop labels

		also in frame mode we have indepent model or label switch path (select one and advertise)
		in cell mode we have orderded or control mode recieve requests then make label

	layer 3 vpn :
		we have peer to peer vpn in mpls laye3 vpn
		use for spiliting contents and routes

		overlay vpn
			layer 1
				still using this model
				leased lines
					e1 , e3 , sonet , sdh , tdm
				all bandwidth use by customer
				expensive and not useful for mesh and many branch
				telco > telecomunication company
				must provide some special infrastructure

			layer 2 
				x25 , atm , framerelay , virtual circute
				with one link connected to each branch in spilit mode
				no optimized routing
				is faster and cheaper than layer 1
				maybe have suboptimal routing

			layer 3
				gre
				diffrent routing from isp
				has administration overhead

			point to poiint 
				mpls vpn alyer3

		overlay means i'v just connected not more you must make routing and ....
		in point to point have routing between ce and pe 
		in scenarios we must use isis or ospf to advertise all rid and...
			int loop 0
			ip address 192.168.254.1 255.255.255.255

			int fast 0/0
			no sh
			ip address 192.168.12.1 255.255.255.252

			router ospf 1
			router-id 1.1.1.1
			network 192.168.254.1 255.255.255.255 area 0
			network 192.168.12.1 255.255.255.255 area 0

			*do it for all ip routers

			int serial  0/0/0
			no sh
			ip address 192.168.14.1 255.255.255.0

			ip route 192.168.6.0 255.255.255.0 192.168.16.6

		if use same range for customers must use vrf for them
			ip vrf blue (or customer-a)
			int serial 0/0/0
			ip vrf forwarding blue
			ip address 192.168.16.1 255.255.255.0

			ip vrf green
			int serial 0/0/1
			ip vrf forwarding green
			ip address 192.168.16.1 255.255.255.0

			ip route vrf blue 192.168.6.0 255.255.255.0 192.168.16.6
			ip route vrf green 192.168.6.0 255.255.255.0 192.168.16.6

		must define all vrf on each pe routers

			sh ip vrf
			sh ip vrf interface
			sh ip route vrf blue

		between pe edge must have routing to determine all vrf and tags
		igp in this situation is not so useful so bgp comes instead
		bgp can transfer large volume of routing and requests
		must create ibgp on loop back interfaces and pe routers
			router bgp 1
			neighbor 192.168.254.5 remote-as 1
			neighbor 192.168.254.5 update-source loop0

			address-family vpnv4
			neighbor 192.168.254.5 active (define pe router to use vrf tags in bgp)

			address-family ipv4 vrf blue
			redist static
			address-family ipv4 vrf green
			redist static
				
			sh bgp ipv4 unicast summary

		route distinguisher :
			ipv4 + vpnv4 addressing must be like this
				rd 	  : ipv4
				64bit : 32bit
				vpnv4 96bit

				every time send address in blue vrf we send vpnv4 
				spilite vrf

				vrf blue : rd 1:1
				vrf green : rd 1:2
				vrf orange : rd 1:3
					1 is as number
					:x means customer id or number

			ip vrf blue
			rd 1:1
			ip vrf green
			rd 1:2

		must enable vpnv4 transmission on bgp
			sh bgp vpnv4 unicast all summary
			sh bgp vpnv4 unicast rd 1:1

		in this state we injected into the bgp but can't send
		rd just spilite addresses no relate to redirection of vrf
			ip vrf blue
			route-target 1:100 both

				route target can be diffrent in vrf when need hub&spoke or spoketospoke topologies must change import and export route target values

				by default use both model in route-target

		pe routers send rt and rd on bgp community in extended mode
		on another pe we can use rd and rt tags to make something
			router bgp 1
			neighbor 192.168.254.5 send-community extended

		on ce must set :
			ip route 0.0.0.0 0.0.0.0 192.168.16.1 (define default gateways)

		in this state r1 in example is pe router if forward it to another pe like r5 and need to negotiate but here we have problem after bgp connection betwen r1 and r5 that is all routers in isp can't realize vrf and couldn't use vrf cocnfiguration on all isp routers so must use mpls labels to advertise our vrf subnets like a normal packet
			mpls label range 100 199

			router ospf 1
			mpls ldp autoconfig

			*now make ldp neighboring on isp routers here can advertise networks with labels

		all another isp routers doesn't know how can reach the customers network
		so use mpls and labels to forward traffics on eachother
		then bgp connection on pe routers recieve the networks on mpls forwarding

			in destination ip routing if don't use bgp and ... must use ibgp for each router in isp also use this ibgp for each user or customer vrf now just add label on pe routers and hide details of data 

		on r5 just recieve some traffics and doesn't know which blongs to which vrf
			
		with rd and rt on bgp packets into the mpls packets can realize them

		1- on router we have some vrf on pe
		2- between pe we have bgp connection
		3- to spiliting between vrf must use vpnv4 > vpnv4= route-distinguisher + ipv4
			vpnv4 > asnumber organization + customer branch number
			outter label of bgp
			
			bgp label (vpnv4)

		4- inside vpnv4 and bgp need to direct vrf connectivity
			use route-target
			which vrf must reciee which vpnv4 traffic

		r1 and r5 are pe routers

		r1 > send bgp update to r5 and r5 use these attributes to forward traffics
			vpnv4 label for vrf blue-r1 on r1 > 1-1 (rd > asnumber-customerid) : 	192.168.9.0/24 (ipv4)
			route-target label > 1:100
			vrf forward to vrf blue-r5

			vpnv4 label for vrf green-r1 on r1 > 1-2 (rd > asnumber-customerid) : 	192.168.9.0/24 (ipv4)
			route-target label > 1:200
			vrf forward to vrf green-r5

		r5
			sh bgp vpnv4 unicast rd 1:1 192.168.9.0 255.255.255.0

			with mpls label just fast forward traffics without processing on each router in main part of isp

		bottom label use here to detect which of the labels must be used on pe to forwarding vrf
		top label removed on php

		vpn lable is fix label on bgp and starts from 16
		nexthoplabel is swapable label applicated on php

			sh mpls forwarding-table 192.168.254.5
				[v] > vpnv4 label

			sh ip cef vrf blue 192.168.9.0 255.255.255.0
				say have 2 label one of them is lib (205 nexthop) and another bgp label (16 vrf)

		if trace from clients all path will be visible, must use ttl propagation 
			no mpls ip propagation-ttl forward
			mpls ldp router-id loop0 force

		for security items better use mpls accepted labels jsut between pe routers not main parts  must set on all routers and main part just forward r5 and r1 labels and use their own labels local mode:
			no mpls ldp advertise-label

			access-list 1 permit 192.168.254.5
			access-list 1 permit 192.168.254.1

			mpls ldp advertise-label for 1

			on pe :
				sh mpls ldp binding
					just show 254.5 and local labels not more

		in route distinguisher we use 64 bits for ipv4 uniquenest on vpnv4 and vrf
									
			type 0 (2byte) = as (2byte) + value (4byte)	
				2byte:4byte
				0 to 65535 : 0 to (2^32-1)

			type 1 (2byte) = ip (4byte) + value (2byte)
				4byte:2byte
				a.b.c.d : 0 to 65535

			type 2  = as (4byte) + value (2byte)
				0 to 65535 : 0 to (2^32-1)

			clear bgp vpnv4 unicast 1.2.3.4 out

		multiprotoocol nlri
			mpreach nlri > vrf
			mpunreach nlri

		each router with mpls cocnfiguration even if didn't recieve any record from bgp, label get generatd cause bgp make label mechanism

		in customers connectivity must use redistribution static and connected to make it possible

		in privilege mode must use :
			vrf context blue
			ping 192.168.16.6

		for route-target in vrf altho same users must bee diffrent values

			ip vrf cus1-1
			rd 1:1
			route-target  1:100 both

			ip vrf cus1-2
			rd 1:3
			route-target  1:100 both

			ip vrf cus2-1
			rd 1:2
			route-target  1:200 both

			router bgp 1
			neighbor pe peer-group
			neighbor pe remote-as 1
			neighbor pe update-source loop0
			neighbor 192.168.254.2 peer-group pe

			address-family vpnv4
			neighbor 192.168.254.2 active

			address-family ipv4 vrf cus1-1 
			redistribution connected
			redist static
				*for customers 1-1 we have static route whole network

			sh ip route vrf cus1-1
			sh ip route vrf cus1-1 bgp
			sh bgp vpnv4 all summary

			ip cef
			mpls ip
			mpls label range 100.199
			mpls ldp router id loop0 force
			no mpls	ip propagation-ttl forward

			access-list 1 permit 192.168.254.0 255.255.255.0

			no mpls ldp advertise-label
			mpls ldp advertise-label for 1

			router ospf 1
			mpls ldp autoconfig

			int fas 0/0
			no sh
			ip vrf cu1-1
			ip address 192.168.15.1 255.255.255.0

			ip route vrf cus1-1 192.168.254.5 255.255.255.255 192.168.15.5

		now every users sets on same rt can see eachother
		on ce must use default route to isp and pe

		vpnv4 generate base on rd and rt information

		rt filter if enable on some routers our vpnv4 just get forward didn't process them in route reflector we break rt filter roll must publish rt
			sh bgp vpnv4 rd 1:1 
			sh gp vpnv4 unicast rd 1:1 neighbor 192.168.254.2 advertise-routes
			sh bgp vpnv4 nicast all
			sh mpls forwarding-table (outgoing aggregation lookup)
				means deliver to destination and make it nolabel
				whole packet get no label and use ip address in simple mode

		mpls vpn ipv4 and ip6 with static route on pe-ce + route reflector :
			same configuration like above 

			interface range fast 0/0-1/3
			ip ospf 1 area 0
			ip ospf network point-to-point (disable dr and bdr make neighborship faster than before)

			int loop 0
			ip ospf 1 area 0

			vrf definition cus1-1
			rd 1:1
			address-family ipv4
			route-target   1:100 both
			address-family ipv6
			route-target   1:100 both

			router bgp 1 
			neighbor 192.168.254.2 remote-as 1
			neighbor 192.168.254.2 update-source loop0

			address-family vpnv4
			neighbor 192.168.254.2 active

			address-family ipv4 vrf cus1-1
			redist static
			redist connected

			interface fas0/0
			ip vrf forwarding cus1-1


			have rr in this scenario
				router bgp 1
				neighbor rr peer-group
				neighbor rr remote-as 1 
				neighbor rr update-source loop0
				neighbor 192.168.254.1 peer-group rr

				address-family vpnv4
				neighbor 192.168.254.3 active
				neighbor 192.168.254.1 active
				neighbor 192.168.254.4 active

				neighbor rr route-reflector-client (make rr on vpnv4)

			if need disable bgp ipv4 on mpls scenario should :
				router bgp 1
				no bgp default ipv4-unicast

			all other configs are same like before

		mpls vpn ipv4 and ip6 with static route on pe-ce + route reflector group :
			if doesn't have same ip address on pe can use same rd
			we have same config just :
				router bgp 1
				bgp cluster-id 1
				no bgp default ipv4-unicast

				neighbor rr peer-group
				neighbor rr remote-as 1
				neighbor rr update-source loop0
				neighbor rr route-reflector-client
				neighbor 192.168.254.3 peer-group rr
				neighbor 192.168.254.1 peer-group rr

				neighbor 192.168.254.4 remote-as 1
				neighbor 192.168.254.4 update-source loop0 (r4 is another rr)

				address-family vpnv4
				neighbor 192.168.254.3 active
				neighbor 192.168.254.1 active
				neighbor 192.168.254.4 active

				* we have same config on both route reflectors but between them must use simple neighborship not reflection accessablity

			on r1 pe
				vrf define cus1-1
				rd 1:1
				address-family ipv4
				route-target   1:1 both

				vrf define cus1-2
				rd 1:2
				address-family ipv4
				route-target   1:2 both

				int fast 0/0
				ip vrf forwarding cus1-1
				no sh 
				ip address 192.168.16.1 255.255.255.0

			on ce must have default route to pe

			here we have duplicate advertise of route reflector
				on rr
					router bgp 1
					address-family vpnv4
					bgp rr-group 1

					ip excommunity-list 1 permit rt 1:1
					ip excommunity-list 1 permit rt 1:2

					do clear bgp vpnv4 unicast * soft

		mpls vpn with pe and ce with ripv2 :
			in small office or home users we have ripv2
			adsl

			router ospf1
			router-id 1.1.1.1
			mpls ldp autoconfig

			interface fast 0/0 
			ip ospf 1 area 0
			ip ospf netwo point-to-point

			int loop0
			ip ospf 1 area 0

			ip cef
			mpls ip

			mpls label range 100 199
			mpls ldp router-id loop0 force
			no mpls ldp advertise-label
			mpls ldp advertise-label for 1
			no mpls	ip propagation-ttl forward

			access-list 1 permit 192.168.254.0 255.255.255.0

			router bgp 1
			neighbor 192.168.254.5 remote-as 1
			neighbor 192.168.254.5 update-source loop0

			address-family vpnv4
			neighbor 192.168.254.5 active

			address-family ipv4 vrf	cu1-1
			redist rip
				*every thing recieve from rip on this vrf inject to ibgp then forward it to r5

				we must do same thing in upside down
					r5
						router rip
						vers 2
						no auto-summar
						address-family ipv4 vrf cu1-1
						redist bgp 1 metric 5 (maximum count of hops)
						redist bgp 1 metric transparent (don't need set)

			vrf define cu1-1
			rd 1:1
			address-family ipv4
			route-target 1:12 export
			route-target 1:11 import

			int serial 0/0
			vrf forward cus1-1
			no sh
			ip address 192.168.11.1 255.255.255.0

			router rip
			vers 2
			no auto-summar
			address-family ipv4 vrf cu1-1
			network 192.168.1.0
			network 192.168.6.0

			*for every users must be enable

		hub and spoke model mpls :
			rt is the most important hing here
			router ospf 1
			router-id 192.168.254.1
			mpls ldp autoconfig

			int fas 0/0
			ip ospf 1 area 0
			ip ospf network point-to-point

			int loop 0
			ip ospf 1 area 0

			ip cef
			mpls ip
			mpls label range 100 199
			mpls ldp router-id loop0 force
			no mpls ip propagation-ttl forward
			no mpls ldp advertise-label
			mpls ldp advertise-label for 1

			access-list 1 permi 192.168.254.0 255.255.255.0

			router bgp 1
			no bgp default ipv4-unicast
			neighbor 192.168.254.2 remote-as 1
			neighbor 192.168.254.2 update-source loop0

			address-family vpnv4
			neighbor 192.16.254.2 active

			*r2 is rr

			here we have route reflector must set config

			r2 :
				router bgp 1
				no bgp default ipv4-unicast
				bgp cluster-id 1

				neighbor rr peer-group
				neighbor rr remote-as 1
				neighbor rr update-source loop0
				neighbor 192.168.254.1 peer-group rr
				neighbor 192.168.254......

				address-family vpnv4
				neighbor 192.168.254.1 active
				neighbor 192.168.254...... active
				neighbor rr route-reflector-client

				vrf define cus2-1
				rd 2:1
				address-family ipv4
				route-target 1:22
				route-target 1:23
				route-target import 1:21

				int fas 0/1
				vrf forward cus2-1
				no sh
				ip address 192.168.212.2 255.255.255.0

			rt export means send to which router
				from center send to which
				in hubandspoke must efine all branches
			rt import means recieve from who
				from which one send to center 
				in huband spoke must define main incoming source

			in this scenario we have no branach access to each other just hub access

		mpls and eigrp between pe and ce :
			use rd and rt in same range
			on ce
				router eigrp 2
				network 192.168.16.6 0.0.0.0
				network 192.168.6.0 0.0.0.0

			on pe
				router eigrp 1 
				address-family ipv4 vrf cus1-1
				auto-system 1
				network 192.168.6.1 0.0.0.0
				redist bgp 1 metric 1500 10 255 1 1500

				or

				router eigrp
				address-family ipv4 unicast vrf cus1-1
				auto-system 2
				network 192.168.0.0 0.0.0.0
				topology base 
				redist bgp 1 metric 1500 10 255 1 1500

				sh eigrp address-family ipv4 vrf cus1-1 neighbors
				sh ip cef vrf cus1-1 192.168.9.0 255.255.255.0

		bgp	extended community for eigrp
			from ce with eigrp connect to pe and from pe to another pe we use ibgp
			in redist we lost some attributes
			in another branch say we have foreign routes and get fuck
			extended community happen in automatic mode
			find here
				sh bgp vpnv4 unicast rd 1:1 1.2.3.4

				0x8800 > flag tag (general route info)
				0x8801 > as + delay (route metric info + as)
				0x8802 > reliability + hop + bw (route metric info)
				0x8803 > reserve + load + mtu (route metric info)
				0x8804 > remote-as + remote-id (route information for external)
				0x8805 > remote protocol + remote metric (route information for external)

		cost community bgp on eigrp :
			route condition
			we have same config on ospf mpls and bgp or ibgp 

			router bgp 1
			bgp best-path cost-community ignor

			in some scenarios we have backup links and this link has fewer bw than main connected link to isp
			between pe and ce use eigrp

			sh ip eigrp vrf w topology
				say what type of protocols like ipv4 or vpnv4 get source with network or  etc

			customer learn network of branch b like lan-b on 2way like isp and backuplink then choice the fastest link like isp based on metric
			means ue mpbpg and ebgo is the best path
			in bgp on scan time import might have changes on route attributes
			in these changes might have suboptimal routing 
			this suboptimal routing say branch has access eachother on backuplinks and ignor the isp main path
			here must use cost community
				64bit and extended

			sh bgp vpnv4 unicast all

				we can use pre best path
				point of insertion
					128 (use before weight attribute)
					129 (afteer igp metric to nexthop for loaad balancing)

					ad value cost on eigrp
						128 internal
						129 external

						sh bgp vpnv4 unicst rd 1:1 6.6.6.6

				if recieve cost-community attribute inside packets check them before weigh value and attribute or check them after igp metric to nexthop in bgp

			work automatically

		eigrp soo (site of origin) :
			if our branchs lan become unreachable our eigrp start query and reply
			on isp link and backup link our query and reply will be sent
			pe routers on isp edge use withdrawn roll and ommit routes and advertise it on mpls and bgp also another pe recieve branch accessablity from backdoor response on eigrp query reply
			if another isp pe said no path to route means our problem is solved
			and get ommit from all route tables
				order of action
					mp bgp update
					query

			if answer on bgp low speed condition
				order of action
					query
					mp bgp update

					customers before recieve withdrawn roll on bgp use query and didn't check bgp state so get fuck
					we get in loop

			eigrp use hop count by default like 100 if reach 100 traffic droped

			int serial 0/0/0 (isp link from ce)
			router eigrp 1
			metric mamx-hop 100 (default can set lower value in caution)

			*must use same value
			now use soo in bgp udpates to prevent loop

			in backdoor scenario we have more cpu usage if forget use soo

			if isp ink of one branch get down on ce we have no any way to reach another branch so must set soo on backdoorlink

			route-map soo per 10
			set excommunity soo 1:100 (must be same on each branch side)

			int fast 0/1
			ip vrf site-map soo

			int serial 0/0/0
			ip vrf site-map soo

			sh bgp vpnv4 unicast rd 1:1 1.2.3.4 255.255.255.0

		mpls vpn pe and ce witth ospfv2
			same config like before
			isp pe and igp content :
				router ospf 1
				mpls ldp autoconfig
				router-id 1.1.1.1

				int fas 0/0
				ip osfp 1 area 0
				ip ospf network point-to-point

				int loop0
				ip ospf 1 area 0

				ip cef
				mpls ip
				mpls label range 100 199
				no mpls ldp advertise-label
				mpls ldp advertise-label for 1
				no mpls ip propagation-ttl forward
				mpls ldp router-id loop0 force

				access-list 1 permit 192.168.254.0 255.255.255.0

				router bgp 1
				neighbor 192.168.254.5 remote-as 1
				neighbor 192.168.254.5 update-source loop0

				address-family vpnv4
				neighbor 192.168.254.5 activate

				address-family ipv4 vrf cus1-1
				redist ospf1

				vrf define cus1-1
				rd 1:1
				address-family ipv4
				route-target both 1:1

				int serial 0/0/0
				ip vrf forward cus1-1
				no sh
				ip address 192.168.6.1 255.255.255.0

			here we can't create address-family for ospfv2 unlike ospfv3 and eigrp (fro this must create another pid for each customer(resource intensive))
				conf t
				router ospf 2 vrf cus1-1
				router-id 1.1.1.1
				network 192.168.1.1 0.0.0.0
				redist bgp 1 subnets

				conf t
				router ospf 2 vrf cus1-1 (device get fuck for each vrf pid processing)
				router-id 1.1.1.1
				network 192.168.1.1 0.0.0.0 (set same range for branch connectivity)
				redist bgp 1 subnets

			on ce
				router ospf 12 (no problem)
				network 192.168.1.7 0.0.0.0 area 0
				network 192.168.7.7 0.0.0.0 area 0

				sh ip route ospf
					show network like o ia means another network is reachable
					o and oia is same and has diffrences on internal attributes

				sh ip ospf
					tell us works in super backbone ospf area (mpls)

			in ospf could send extended community like eigrp
				on ce routers
					int loop 0
					ip address 6.6.6.6 255.255.255.255
					route-map a permit 10
					match interface loop 0

					router ospf 1
					redist connected route-map a subnets

						now ospf inject address 6.6.6.6 to ospf but didn't inject to bgp
						cause it's external

				pe router isp
					router bgp 1
					address-family ipv4 unicast vrf cus1-1
					redist ospf 2 match externaal internal / nssa-external (keep record type)

					sh bgp vpnv4 unicast rd 1:1 192.168.6.0 255.255.255.0
					sh bgp vpnv4 unicast rd 1:1 6.6.6.6 255.255.255.0

				now have distribution with diffrent bgp values

				rt 0.0.0.0:2:0 > internal
				rt 0.0.0.0:5:1 > external

		maximum count of routing protocol process could launch on each router is 32

		3 state of country works on abr and mpls wanna get link together
			on abr routers
				router ospf 1 
				mpls ldp autoconfig
				router-id 192.168.254.1

				interface fas 0/0
				ip ospf 1 area 0
				ip ospf network point-to-point

				int loop 0
				ip ospf 1 area 0

				ipcef
				mpls ip
				mpls ldp label range 100 199
				mpls ldp router-id loop0 force
				no mpls ip propagation-ttl forward
				no mpls ldp advertise-label
				mpls ldp advertise-label for 1

				access-list 1 permit 192.168.254.0 255.255.255.0

				router bpg 1
				neighbor 192.168.254.4 remote-as 1
				neighbor 192.168.254.4 update-source loop0
				no bgp default ipv4-unicast
				address-family vpnv4
				neighbor 192.168.254.4 active

			on r4 or isp core
				same config with abr router

				router bgp 1 
				bgp cluster-id 1
				no bgp default ipv4-unicast
				neighbor rr peer-group
				neighbor rr remote-as 1
				neighbor rr update-source loop0
				neighbor 192.168.254.1 peer-group rr
				neighbor 192.168.254.2 peer-group rr
				neighbor 192.168.254.3 peer-group rr

				address-family vpnv4
				neighbor 192.168.254.1 active
				neighbor rr route-reflector-client

				vrf define cus1-1
				rd 1:1
				address-family ipv4
				route-target both 1:1

				int serial 0/0
				ip vrf forwarding cus1-1
				no sh
				ip address 192.168.15.1 255.255.255.0

				*for each customer in many state of country must use same ospf pid if use diffrent means we have external

				router ospf 2 vrf cus1-1
				router-id 1.1.1.1
				network 192.168.15.1 0.0.0.0
				redist bgp 1 subnets

				router bgp 1
				address-family ipv4 vrf cus1-1
				redist ospf 2 match eternal internal

				*ospf redistribute all connected automatic

				on ce and branch
					router ospf 1
					router-id 5.5.5.5
					network 192.168.15.5 0.0.0.0 area 0

				if has enable ip cef and mpls doesn't work must check ios , ios must be advance enterprise no advance security



		if had 2 area 0 an superbackbone (in triangel topology must us ospf between r123)
			r2  and r3 are same and between r2 and r3 have bgp:
				router ospf 1
				router-id 192.168.254.2
				
				int fas 0/0
				ip osp 1 area 0
				ip ospf network point-to-point
	
				int loop0 
				ip ospf 1 area 0
	
				router bgp 1
				neighbor 192.168.254.3 remote-as 1
				neighbor 192.168.254.3 update-source loop0
	
				address-family vpnv4
				neighbor 192.168.254.3 active
	
				int loop 1
				ip vrf forwarding cus1-1
				ip address 2.2.2.2 255.255.255.255
	
				vrf defin cus1-1
				rd 1:1 
				address-family ipv4
				route-target both 1:1

			for customers
				router ospf 2 vrf cus1-1
				network 2.2.2.2 0.0.0.0 areaa 0
				router-id 2.2.2.2
				redistribution bgp 1 subnets

				router bgp 1
				address-family ipv4 vrf cus1-1
				redist ospf 2

				sh ip ospf database (lsa 3 transmission on bgp)
					must send area 0 to area 0 with term o in ospf and reachable shamlink

		ospf down hit
			automatic task with lsa 3
			in some topologies we have many pe and ce that use one user to have redundancy on connections
			might have loop
			cause injection between bgp and ospf
			if inject from ospf to mpbgp we can see downbit in packets (loop prevention)

			sh ip ospf database summary 5.5.5.5



		ospf tag field
			loop 0 on r1 (pe router)
			if get redistribute to bgp on vpnv4 and ospf > shows oe2
			if use backdoor mechanism in bgp and ospff f branches 
			maybe have have loop in ospf and bgpp if use tag loop prevention procedure automatic generated

			sh ip ospf database
				lsa 5
					tag(use isp as)
					32bit
					1101000000000000(16bit) + as-number(16bit) 

			on backdoor router
				router ospf 2
				router-id 7.7.7.2
				network 192.168.57.7 area 0
				redist ospf 2 subnets
				
				router ospf 3
				router-id 7.7.7.3
				network 192.168.76.7 area 0
				redist ospf 3 subnets

			on pe router
				int loop 0
				vrf define cus1-1
				no sh 
				ip address 1.1.1.1 255.255.255.255

				route-map loop1 permit 10
				match int loop0

				router bgp 1
				address-family ipv4 vrf cus1-1
				redist opsf 2 mathc internal external
				redist connected route-map loop1

				*downtag add on interface and routing tag get ommit

				router ospf 2
				downtag 1 (on pe)


		bgp excommunity for ospf
			route type 
			downid or pid (findout internal or external)
				for internal we have 0.0.0.0:2:0
				for external we have 0.0.0.0:5:1

			option
				area number > 0.0.0.0
				downid > type of route (lsa for type 2 and 5)
				metric type 
					if set 0 means type 1 used
					if 1 set means type 2 is used
				ospf rid

		sham link
			between pe and ce if use ospfv2 is usefull
			lsa type 3 injecte to bgp and mpls with oia
			o > oia

			like virtual link
			should say a tunneling between r2 r3 and transmmit lsa
			in this state oia replace with o
			shamlink use more cost for them
			better use loop for shamlink

			on pe
				int loop1
				no sh
				ip vrf forwarding cus1-1
				ip address 2.2.2.2 255.255.255.255

				router bgp 1
				address-family ipv4 vrf cus1-1
				network 2.2.2.2 mask 255.255.255.255

				router ospf 2
				area 0 shamlink 2.2.2.2 3.3.3.3 cost 10 (fewer than backdoor cost)
					better same on 2side

				sh ip ospf shamlink
					dna (no hello time)

			on ce and links to backdoor
				int fas 0/0
				bandwidth 1000 (change to lower value of bw)

					if lsp get break our backdoor used

				sh ip ospf 2 database
				sh ip ospf 2 router 2.2.2.2

		mpls vpn pe ce with bgp
			if same customer in many branch use same as and make our network like transit network all clients bgp packets get block and drop
			doesn't need redistribution
			just need as-override in vrf
			rt must be on both

			pe
				router bgp 1
				address-family ipv4 vrf cus1-1
				neighbor 192.168.16.6 as-override

		allow as
			like as overide
			doesn't exist on ip in real world just works on vpnv4
			in allow as we can use threshold of repeated asnumber

			ce
				router bgp 2
				neighbor 192.168.24.2 allow-as 1 (if see your as givee access)

		multipath
			for eibgp and eebgp we can use multipath
			better use as override
			better set no bgp default ipv4-unicast on pe to inside router of isp not to internet and external networ (foreign)

			some topologies migh have dual link single home and isp detect internal path is better than direct path our traffics use more as path
			reason is rr in isp set this policy
			rid in this situation is important

			core router of isp
				int fas 0/0
				ip ospf cost 20 (set this from p to pe)

			on pe
				rouer bgp 1
				address-family ipv4 vrf cus1-1
				maximumpath ebgp 2

		multipath import path
			in pe routers use multipathing but in show we see best path
			pe
				router bgp 1
				address-family ipv4 vrf cus1-1
				max path ibgp 2
				importpath selection all (now add all path )
				import path limit 2
					*bestpath > many ways and best of them in compare
					*multipath > just path that activated multipath 

		bgp site origin
			like eigrp
			internal network must learn from igp not from isp
			between pe and ce ue bgp
			if recieve update from site don't return them to origin site
			on pe
				router bgp 1
				address-family ipv4 vrf	cus1-1
				neighbor 192.168.15.5 soo 1:100
				neighbor 192.168.36.6 soo 1:100 (independent from rt and rd)

				sh bgp vpnv4 unicast rd 1:1 5.5.5.5
				clear bgp ipv4 unicast 192.168.36.3 out

		advance mpls vpn > iverlapping von mpls
			confilict on vpn
			till here were seperated from each other but here rt controll is mportant
			pe config for clients :
				r6
					rd 1:11
					rte 1:12 1:13
					rti 1:11
				r7
					rd 1:21
					rte 1:22 1:23
					rti 1:21
				r8
					rd 1:13
					rte 1:11 1:12 1:23
					rti 1:13
				r9
					rd 1:12
					rte 1:12 1:11
					rti 1:12
				r10
					rd 1:22
					rte 1:21 1:23
					rti 1:22
				r11
					rd 1:23
					rte 1:22 1:21 1:13
					rti 1:23

			ping 1.2.3.4 -c 1

		central service
			central branch of banks connected to all another brnaches like bmi tejarat ....
			pe and ce connect to each other with bgp
			we have same config on pe and ce
			clients rd config on pe
				r6
					rd 1:11
					rti 1:11
					rte 1:12 1:13
				r7
					rd 1:21
					rti 1:21
					rte 1:22 1:23
				r8
					rd 1:13
					rti 1:13
					rte 1:11 1:12 1:31
				r9
					rd 1:12
					rti 1:12
					rte 1:13 1:11
				r10
					rd 1:22
					rti 1:22
					rte 1:21 1:23
				r11
					rd 1:23
					rti 1:23
					rte 1:22 1:21 1:31

				in this scenario must use as-override
				also have bgp with rr and pe
				between rr or pe and ce must use normal bgp on ipv4 not vpnv4

		managed ce router services
			except isp need to connect on ce routers
			we configure and monitor ces
			usually use pe routers for tem
			in rt config must be carefull and set them on same import rt and export rt
			to some reasons must use rt on pe ce and p router of isp

			each router has loop0 with same number of self router
			for each loop00 on ce need inject them excommunity then add new rt
			with igp bgp injection for loop0 will be happen

			on pe
				access-list 2 permit 6.6.6.6
				access-list 3 permit 7.7.7.7

				route-map cus1-1 permit 10
				match ip address 2
				set excommunity rt 1:4 additive (add or prepend)
				route-map cus1-1 perm 20

				vrf define cus1-1
				address-family ipv4
				export map cus1-1 (every one use this rf add excommunityy with rt 1:4)

				here change rt and address 6.6.6.6 get changed

				for nms better use rt-export 1:11
				better use rt import for all customers for nms access 1:14

		internet access
			till here we were sp or service provide now need internet for each sp
			get isp

			should use policy for rt and make each site visibility
			for internet access just make one simple bgp connection on pe and p router of isp
				for nat mechanism

			customers use pa or pi addresss model for internet access

			on ce
				access-list 100 deny ip any 192.168.0.0 255.255.0.0 (better use another private ranges for mpls)
				access-list 100 permit any any

				for site to site connectivity must use nat just for internet not more

				ip nat pool cus 1-1 100.0.0.1 100.0.0.2 network 255.255.255.252
				ip nat inside source lis 100 poll cus1-1 overload

				int fas 0/0 
				ip nat inside
				int serial 0/0/0
				ip nat outside

				ip route 0.0.0.0 0.0.0.0 192.168.16.1

			now need leackage for route  understanding that rr connected to ict and need do this on pe
				ip route vrf cus1-1 0.0.0.0 0.0.0.0 192.168.254.3 global (mean i have an ip in global vrf or global routing table must use it to rah 192.168.254.3)
				*do for each vrf
				ip route 100.0.0.0 224.0.0.0 siral 0/0/0

				router bgp 1
				address-family ipv4 
				redist static

		vrf aware nat
			isp develop all nat on edge of isp 
			our isp public ip is on p router and another ict edge
			on p router of isp we have nat
			with vrf aware nat can advertise default orute

			we rome from pe thee items
				bgp redist
				static routes
				and nat

			pe router :
				router bgp 1 
				address-family ipv4 vrf cus1-1
				neighbor 192.168.16.6 default-originate

			p router (just route has no vrf for customers)
				int fas 0/0
				ip nat inside
				int serial 0/0/0
				ip nat outside

				ip nat pool sp 100.0.0.1 100.0.0.2 network 255.255.255.252

				vrf define internet
				rd 1:4
				address-family ipv4 
				route-target export 1:4 1:12 (inject on pe a default route for vrf)
				route-target import 1:14

				ip route vrf internet 0.0.0.0 0.0.0.0 192.168.123.12 global (ict ip address)

				router bgp 1
				address-family ipv4 vrf internet
				network 0.0.0.0
				redist static

				access-list 100 permit ip 192.168.0.0 255.255.0.0 any
				ip access-list 100 
				5 deny tcp any any eq 179
				6 deny tcp any eg 179 any 
					*or bgp doesn't go behind nat

				ip nat inside source list 100 pool sp  vrf internet overload
					*every body hasa rt of internet vrf use vpnv4 and internt accessablity

				clear ip nat translation *

				sh bgp vpnv4 unicast rd1:1 0.0.0.0 0.0.0.0

				ip route 100.0.0.0 224.0.0.0 null 0
				router bgp 1
				address-family ipv4
				redist statics (ict route could find address 100)

			*we have default route on pe

		shared services
			isp ussualy use this method
			vrf of internet use special rt in import
			on pe
				r6
					rd and rt import 1:11
					rt export 1:12 1:51 (51 use for internet vrf)
					
				r7
					rd and rt import 1:21
					rt export 1:22
					
				r8
					rd and rt import 1:31
					rt export 1:51 
					
				r9
					rd and rt import 1:12
					rt export 1:51 1:11

				r10
					rd and rt import 1:22
					rt export 1:21

				r11
					rd and rt import 1:41
					rt export 1:51
					
				r12
					rd and rt import 1:51
					rt export 1:12 1:11 1:41 1:31

		inter as option a
			same ospf and mpls	
			p router	
				int serial
				ip vrf forwarding cus1-1
				ip address 192.16.8123.3 255.255.255.0
	
				router bgp 1 
				address-family ipv4 vrf cus1-1
				neighbor 192.16.123.3 remote-as 64515

			ce
				we set nat on pool with 100.0.0.0/3 and make our branch visible on acl
				better use default route on ce to pe

			pe
				ip route vrf cus1-1 100.0.0.1 255.255.255.255 192.168.16.6

				on bgp every vrf we use vpnv4 so set redistribution on eigrp and rip also bgp (redist static)

				rd and rt must be carefully set to detect internet

			in multiprotocol interconvetion like eigrp to rip must set some metric attributes
				pe 
					router rip
					no redist bgp 1 metric transport
					address-family ipv4 vrf cus1-1
					redist bgp metric 5

					clear bgp ipv4 unicast * soft
		
			*recieve internet with one rt

				in 2 state of country we have mpls connection with 2 smae isp in diffrent state must connect together
				between isps make mpls connection
				between pa and ce has bgp
				on pe routers of isp1 use subinterface , set vrf and dot1q tag for each customer

				same config ospf mpls bgp and vrf
				use asoverride on pe side

				same rt and rd on pe

				on pe
					int fas 0/0
					vrf forwarding cus1-1
					encapsulation dot1q 1 (for each vrf and customer set diffrent value)
					ip address 192.168.24.2 255.255.255.0

					vrf definition
					rd 1:1
					address-family ipv4
					route-target both 1:1

					router bgp 1
					neighbor 192.168.254.3 remote-as 1
					address-family vpnv4
					neighbor 192.168.254.3 aactive

				*must enable ospf on internal isp interfaces not more

					address-family ipv4 vrf cus1-1
					neighbor 192.168.24.2 remove-as 2

				on each interface that has dot1q and roas must set shape qos and policy
				this method has overhead
				in qos deployement is suitable but type a is hard for administration

		inter as option b
			nexthop self option
				same config on ospf mpls bgp and between pe and ce has bgp

				in option a we were use ipv4 bgp between pe side of each isp branchs
				pe routers ommit vrf and dot1q

				each router doesn't have vrf and rt must have rtfiter but for pe router must deny rt filter
					pe
						router bgp 1
						no bgp default route-target filter
						sh bgp vpnv4 unicast all
						clear bgp vpnv4 unicast * in

						*with this transmit all rt to another pe

					here must use net hop self cause 2 as number is working
					on pe to p or pe routers
						router bgo 1
						address-family vpnv4
						neighbor 192.168.254.3 nexthop-self
						neighbor 192.168.254.5 nexthop-self

						mpls bgp forwarding (casue we don't use ipv4 in method)(in transmission from customers we have trouble if be on ip cause label are used)

				qos is not easily applied but tag can be used for qos

			redistribution option
				same scenario
				need as override

				ommit nexthop self on pe routers to p or pe routers
				must make default option on route-target filter in bgp
				must enable vpnv4 on pe routers

				on pe
					ip prefix-list a permit 192.168.24.0/24

					route-map a permit 10
					match ip address prefic-lis a

					router bgp 1
					address-family ipv4 
					redist connected route-map a1 subnets

					*better use in ospf change bgp to ospf

					router ospf 1
					redist connected route-map a1 subnets

					in pe router and spf isp redist all 2 pe in states of country
					here we have problem on mpls labeling from 192.168.254.0/24 on 192.168.24.0/24
					here doesn't advertisement for this range and ldp label in acl on mpls for bgp must take labels

					no next-hop-self and mpls label created trouble
					in one before the last router we have poping mechanism and detect tag x could findout this tag meaning
					when we have bgp for vpnv4 between 2 pe wanna used for redistribution and visible on rib like cnnnected ip
					better use single /32 ip for nexthop self and labels

					access-list 1 permit 192.168.24.4/32
					access-list 1 permit 192.168.254.0/24

					ip prefix-list a permit 192.168.24.4/32

					here msut disable pop on one before the last router and forward it to pe router with label
					on pe we have connected range with label range and use null for forwarding 
					on one before the last hop we have pop mechanism

			not connected
				we have multiple link between each pe router of isp in 2 state of country
				must enable our bgp vpnv4 connection on lopback
				if nexthops set on 192.168.254.4 ccould not find them

				default rt filter
				no nexthop
				same mpls bgp ospf
				no vrf no roas

				between pe and p router use vpnv4

				pe
					router bgp 1
					neihgbor 192.168.254.4 remote-as 2
					neighbor 192.168.254.4 update-source loop 0
					neighbor 192.168.254.4 ebgp-multihop 3
					address-family vpnv4
					neihgbor 192.168.254.4 active

					*here advertise and learn vpn on rt

					here 192.168.254.4 is not reachable
					cause doesn't have satatic route

					ip route 192.16.254.4 255.255.255.255 192.168.24.4
					ip route 192.168.254.4 255.255.255.255 192.168.42.4
						*use 2 redundant interface

					router ospf 1
					redist sattic subnets

					*now isp network get reachable on lable without nexthop self command
					here we have proble > mpls bgp forwarding is disable and doesn't active automatic
						int range fas 0/0-1/1
						mpls bgp forwarding

					still no label cause pe use static route on it and one before the last router use php mechanism so make no lable

						mpls static binding ipv4 192.168.254.4 255.255.255.255 output 192.168.24.4 explicit-null
						mpls static binding ipv4 192.168.254.4 255.255.255.255 output 192.168.42.4 explicit-null
							*here if recieve a packet for this destination make null lable

						mpls static binding ipv4 192.168.254.4 255.255.255.255 input

					enable on mpls routers in isp (all nodes)

					qos is not suitable

		inter as option d or ad
			benefits of option a and b
				qos by vrf on option a
				flexible and automatic on option b

				we use one bgp session on pe routers in  2 state and transfer all vrf

				pe routers use bgp for ce
				internal
					r6
						rd and rt import 1:11
						rt export 1:12
					
					r7
						rd and rt import 1:21
						rt export 1:22
					
					r8
						rd and rt import 1:31
						rt export 1:32

				external on pe
					r6
						rd and rt import 1:12
						rt export 1:11
					r7
						rd and rt import 1:22
						rt export 1:21
					r8
						rd and rt import 1:32
						rt export 1:31

				another pe extrenal
					r9
						rd and rt import 1:13
						rt export 1:14
					r10
						rd and rt import 1:23
						rt export 1:24
					r11
						rd and rt import 1:33
						rt export 1:34

				another pe in state extrenal
						rd and rt import 1:14
						rt export 1:13
					r10
						rd and rt import 1:24
						rt export 1:23
					r11
						rd and rt import 1:34
						rt export 1:33

				on pe in state define vrf for each customer
				use dot1q and roas
				rt filter is default
				we have neighbor bgp on rr
				disable nexthop self on pe routers in state to direct of pe an p routers

				we use one bgp connection on pe in2 states
				fir connect o ipv4 then works on vpnv4

				on each router vrf in pe must set 192.168.24.2 255.255.255.0 (on roas interface ip)

				router bgp 1
				neighbor 192.168.24.4 remote-as 2 (connect on ip ten send vpn for them)
				address-family vpnv4
				neihgbor 192.168.254.4 active

					*here must inject from vrf to vpn
					neighborship get break for roas reason
					on dedicated interface didn't set an ip but out global interface on vrf
					must be default type 
					didn't make 3 bgp connection like type b 
					native vlan used for global vrf or vlan1 and use sub interface on them

				*between pe on another state and pe for customers should have bgp connection without nethop self

				must use native vlan to transmit all tags

				on pe in another state
					int fast 0/0.100 (to ict)
					encapsulation dot1q 100 native
					ip address 192.168.24.2 255.255.255.0

						*vlan 100 transmit all traffics without tags (mpls bgp forwarding get enable by deault)

				on each vrf in pe routers on states must set inter as hybrid (ios feature)
					vrf definition cus1-1
					rd 1:1
					address-family ipv4
					route-target export 1:11
					route-target export 1:12
					inter as hybrid

				router bgp 1
				address-family vpnv4
				neighbor 192.168.24.2 inter-as hybrid

					*here inject vrf on ip to vpn converting get transfer
					on one bgp session transfer all vpnv4 for all vrf
					here rt values for eport and import is diffrent no one can't see each other
					better set and change rt values to make visibility

					sh ip vrf details (show us inter as hybrid)

					if use this method our packets contain many lsp
					all ios doesn't support inter as hybrid
					we have overhead to config on each vrf


		inter as option c
			works better than other options
			after option ab option c invented 
			in option c between rr and 2 isp make bgp connection for vpn 
			use one lsp for it
			and in special mechnaism use bgp session and make ping betwen pe on another states
			between pe and e use bgp
			
			same config like before

			between pe and p router of isp we don't use vonv4 and ipv4 bgp connection
			p router must have full neighborship with pe router o customer side
			p router must be rr

			on in pe routers to customer side must make a vrf and bp connection

			r6
				rd and dt import > 1:11
				rt export > 1:12
			r7
				rd and dt import > 1:21
				rt export > 1:22
			r8
				rd and dt import > 1:31
				rt export > 1:32
			r9
				rd and dt import > 1:12
				rt export > 1:11
			r10
				rd and dt import > 1:22
				rt export > 1:21
			r11
				rd and dt import > 1:32
				rt export > 1:31

			in this state must make a simple bgp connection between 2 state of isp on country (find each address from ospf fro rr connectivity (just ipv4 bgp not vpnv4))
				pe
					router bgp 1
					neighbor 192.168.24.4 rmeote-as 2
					(doesn't need update source ... also doesn't have vrf)
					network 192.168.254.3 mask 255.255.255.0

					*do this for another state pe router and define the range

					ip prefix-list a permit 192.168.254.5/32
					route-map aa permit 10
					match ip address prefic-lis a
					router ospf 1 
					redist bgp 1 route-map aa subnets

					*rt filter on default mode

				on p router isp (rr)
					router bgp 1 
					neighbor 192.168.254.5 remote-as 2
					neighbor 192.168.254.5 ebgp-multihop 5
					neighbor 192.168.254.5 update-source loop 0
					address-family vpnv4
					neighbor 192.168.254.5 active

				here pe router of state edge could not understand labels of p router and pe router to customer sides must set special mechanism for bgp ipv4 not dot1q
				on pe routers of 2 state country
					mpls bgp forwarding
					router bgp 1
					neighbor 192.168.24.4 send-label


		carrier supporting carrier
			csc (in security is most important thing)
			2 isp that connected to sites with ict mak connectivity

			is like option c

			between pe customer side and ict router must see loop backs
			use vpnv4 on bgp
			on isp internal links and loop 0 of each isp router must set ospf

			rid of ospf must be on loop 0 or 192.168.254.x/32

			same mpls for all nodes

			between ce and pe use bgp and vrf

			pe of isct and pe of isp must use simple bgp
				on isp pe
					router bgp 100 
					no bgp default ipv4-unicast
					neighbor 192.168.49.7 remote-as 200
					address-family ipv4 
					neighbor 192.168.49.4 active
					network 192.168.254.12 mask 255.255.255.255 (define pe router os customer part)

			here must link all another isp pe router on ict devices
			if betwee 2 pe of ict make seperated connection our labels forward get delete and doesn't create
			so must add isp links to vrf on ict pe routers to make visibility

			casue mesh bgp topology in ict 

			on pe router of ict (for another isct pe use diffrent values)
				vrf define isp100
				rd 1:13
				address-family ipv4
				route-target both 200:1

				router bgp 200
				address-family ipv4 vrf isp100
				neighbor 192.168.49.9 remote-as 100
				neighbor 192.168.49.9 as-override

				router bgp 200
				neighbor 192.168.254.4 remote-as 200
				neighbor 192.168.254.4 update-source loopp0

				address-family vpnv4
				neighbor 192.168.254.4 active

				*here use ipv4 and vpnv4 for gbp on ict pe routers (must use mpls)

			better use same rd on 100:1 isp pe routers to customer side 

			on isp pe to ict side
				ip prefix-list a permit 192.168.254.12/32
				route-map aa permi 10
				match ip address prefic-lis a
				router ospf 100
				redist bgp 1 route-map aa subnets

				*this router define routers of another isp to their network

			on pe customer side of isp
				router bgp 100
				neighbor 192.168.254.12 remote-as
				neighbor 192.168.254.12 update-source loop0
				address-family vpnv4
				neighbor 192.168.254.12 active

				on these pe customer side of isps we have as override and vrf

				here customers wanna access to branch must use label with pe nexthop like 192.168.254.5

				on ict pe to isp pe
					router bgp 200
					address-family ipv4 vrf	isp100
					neighbor 192.168.18.8 send-label

				on isp pe to ict
					router bgp 100
					address-family ipv4
					neighbor 192.168.18.1 send-label

				under port of isp msut set mpls bgp forward

				here use many abels to advertise them
