vxlan evpn
***************************
general speaking
	new network technologies
	useful on sd-access (dnac offers lisp and vxlan evpn) and data centers (sdn aci)

	stands for virtual extensible lan

	layer 2 over layer 3 tunneling

	next generation of vpls, mplsvpn

	ethernet will be replace and convert to vxlan evpn

	on wan we have sdwan and overlay ipsec

	advnatages over traditional networks
		on traditional we have 3 limitation
			vlan starvation
				use 4000 vlan numbers
				we need more these numbers on azure aws google....

			mac table depletion
				core switched need learn and store mac address
				on large scale get trouble

			flooding
				get trouble on bum traffics (broadcast, unicast, multicast)

		*also spanning-tree issues
		we use fast mdoe for convergence

		multi tenancy help us to provide many layer2 and layer3 networks

		on vpls we have problem on multihoming here don't have this

		tunnel base mechanism to forward traffics over tunnels

	framing
		classic
			dmac
			smac
			802.1q
				12 bit 
				4096 vlan

			etype
			payload
			crc

		vxlan header
			dmac
			smac
			802.1q
			tpid = tag protocol identifier
				16 bit

			tci = tag control information
				pcp = priority code point
					3 bit

				cfi = canonical format indicator
					1 bit

				vid = vlan identifier
					12 bit

			etype
			data
			crc
			------------------
			outer mac
			ip
			udp
			vxlan (like 802.1q on classic but get replaced)
				flags
				res
				vni (24 bit)
					vxlan network identifier
					hardware support
					like vlan id

				res

			dmac (like classic headers)
			smac (like classic headers)
			etype (like classic headers)
			payload (like classic headers)
			crc (new)

			*on core network infrastructure base on layer 3 and make tunnel on layer 2 frames 
				no problems on mac learning
				no loop
				no spanning-tree
				has fast convergence with bfd
				no vlan limitation
				no flooding

				16m segments 
			--------------------
			50 (54) bytes of overhead
			underlay
				outer mac header (14 byte)
					dest mac address
						next hop mac address
						48 bit

					src mac address
						src vtep mac adddress
						48 bit

					vlan type
						0x8100
						16 bit

					vlan id
						16 bit

					tag
						16 bit

					ether type
						0x0800
						16 bit

					*4 byte optional 

				outer ip header (20 byte)
					ip header misc data
						72 bit

					protoccl
						0x11 (udp)
						8 bit

					header checksum
						16 bit

					scurce ip (src vtep address)
						32 bit

					dest ip (dest vtep address)
						32 bit

				udp header (8 byte)
					source port 
						hash of the inner l2/l3/l4 headers of the original frame
						enables entropy for ecmp load balancing in the network

						16 bit

					vxlan port
						udp 4789
						16 bit

					udp length
						16 bit

					checksum 
						0x0000

						16 bit

			overlay
				vxlan header (8 byte)
					vxlan flags rrrr|rrr
						8 bit

					reserved
						24 bit

					vni
						24 bit

					reserved
						8 bit
			
				original layer-2 frame

	scheme
		vm-a--------|									|---------vm-d
					|	 								|
		vm-b--------|--- sw1 ------------------ sw2 --- |
					|	ip1						ip2		|
		vm-c--------|									|---------vm-e

		loaction 										location
			vm-a > virtual ether 1 							vm-d > virtual ether 1
			vm-b > virtual ether 2 							vm-e > virtual ether 2
			vm-c > virtual ether 3
															vm-a > ip1
			vm-d > ip2 										vm-b > ip1
			vm-e > ip2 										vm-c > ip1

		vxlan  (vni)											vxlan
			vm-a > 5000										vm-a > 5000
			vm-b > 5000 									vm-b > 5000
			vm-c > 6000										vm-c > 6000

			vm-d > 5000										vm-d > 5000
			vm-e > 6000										vm-e > 6000
	
		*ip address on switches which connects to virtual machines called vtep (virtual tunnel endpoint)
		*between switches we have multipoint tunnels

	vxlan vs vxlan evpn
		learning mechanism is different and how create tables
		vxlan learning will be on data plane
			on forwarding learn them

		evpn will be on control plane
			bgp protocol help learn mac and ip and vni
			over layer 3 fabric work like multicast 
			before running anything we must set bgp connection and define each switch or vtep to fabric so worked without flood or broadcast


						arp and simple vxlan	
			vm-1 	|  																	| 	vm-2
			vxlan 	|---- vswitch-1 (vtep-1) ------------------ vswitch-2 (vtep-2) ----	| 	vxlan
			5000 	|																	| 	5000
			--------------------------------					---------------------------------
			host-1 												host-2

			at the first steps our switches doesn't know each other so try to discover with arp
			broadcasting with arp

			on host-1 discovery values
				src mac > vx 1.1
				src ip > vtep-1
				dest ip > unknown (if were multicast it's okey if were not on multicast must broadcast them to all switches)
				dest mac > ffff broadcast

				*after this broadcasting our vni and mac address will be broadcast to vtep-2 and arp-reply will be like classical ethernet
				*in this condition aftre arp and mac learning just create tunnels and no dependencies on control plane cause use simple vxlan mechanism
				*evpn is different so use bgp and mpbgp for learning and don't need flooding (learned mac and ip of vteps before on simple vxlan)(evpn is next level of simple vxlan)

			mp-bgp (multiprotocol bgp)
				extention to bgp

				use this instead of simple bgp to advertise everything 

				vpn address-family
					allows different types of address families 
						vpnv4
						vpnv6
						l2vpn evpn
						mvpn

					information transported across single bgp peering

				*ebgp supported without bgp route reflector

			multitenancy problems
				might have many same ip or mac addresses
				must set route distinguisher (rd) for each vrf
				our ip adn mac address works with rd
				on vxlan we use automatic rd
					type 1 format
						4 byte ip address (router id)
						4 byte value (vrf id)

				advertise mac must be imported to which vlan or ip advetise must be on which vrf
				solved with route targets (rt)
				combination of vni (vlan number) + as-number
					8 byte route target (2 * 4 bute)

				vrf context vrf-a
				vni 50000
				rd auto
				address-family ipv4 unicast
				route-target both auto
				route-target both auto evpn
				address-family ipv6 unicast
				route-target both auto
				route-target both auto evpn

				*cust-a > vlan 10 over vrf a >> on vxlan we have vni 1000 
				*cust-b > vlan 10 over vrf b >> on vxlan we have vni 2000
					on another side will be translate to another attributes

			control plane on mpbgp
			data plane might be mpls or pbb (prorvider backbone bridges) or nvo (network virtualization overlay)

			network virtualization overlay
				is tunnel for data centers fabric encapsulations
				provide layer 2 and layer 3 overlay over simple	ip network

			important route type must be advertised on evpn nlri (network reachability information)
				 route type 1 - ethernet auto-discovery (a-d) route

				 route type 2 - mac/ip advertisement route*
				 	provides end host reachability

				 	rd (1 octet)
				 	esi (10 octets)
				 	ethernet tag id (4 octets)
				 	mac address length (1 octet)
				 	mac address (6 octets)
				 	ip address length (1 octet)
				 	ip address (0, 4, or 16 octets)
				 	mpls label1 (3 octets)
				 	mpls labe12 (0 or 3 octets)

				 	the following fields are part of the evpn prefix in the nlri
				 		ethernet tag id (zeroed out)
				 		
				 		mac address length (/48), mac address
				 		
				 		ip address length (/32, /128), ip address [optional]

				 		additional route attributes
				 			ethernet segment identifier (esi) (zeroed out)
				 			mpls label1 (l2vni)*
				 			mpls label2 (l3vni)*

				 	vtep v1 advertises host "a" reachability information
				 	 	mac and l2vni [mandatory]
				 	 	ip and l3vni [optional]
				 			depending on arp
				 	
				 	additional route attributes advertised
				 		mpls label1 (l2vni)
				 	 	mpls label2 (l3vni)
				 	 	extended communities

				 	 |route type 	|mac, ip 		|l2vni ("vlan") |l3vni ("vrf") 		|nh 		|encap 		|seq
				 	 |2 			|mac_a, ip_a 	|30000 			|50000 				|ip_v1 		|8:vxlan 	|0

				 	 v2# show bgp 12vpn evpn 192.168.1.73

				 	 bgp routing table infermation for vrf defalt, address/family l2ven evpn

				 	 route distinguisher: 10.0.0 4:38868

				 	 bgp routing table entry for [2]: [0] : [0] : [48] : [0050.56a3.c2bb] : [32] : [192.168.1.73]/272, version 4
				 	 !route type: 2 - mac/ip | ethernet segment | identifier | ethernet tag | identifier | mac address | length | mac address | ip address length | ip address
				 	
				 	 paths: (1 available, best #1)
				 	
				 	 flags: (0x000202) on xmit-list, is not in 12rib/evpn, is locked

				 	 advertised path-id 1
				 	
				 	 path type: internal, path is valid, is best path, no labeled nexthop
				 	
				 	 as-path: none, path sourced internal to as
				 	
				 	 10.0.0.1 (metric 3) from 10.0.0.111 (10.0.0.111)
				 	
				 	 origin igp, med not syt, localpref 100, weight 0
				 	
				 	 received labet30001 50001
				 	 !30001 is l2vni and 50001 is l3vni
				 	
				 	 extcommunity: rt: 65501:30001 rt: 65501:50001 encap:8 router mac: 5087.89d4.5495
				 	 !remote vtep ip address | route target: l2vni (vlan) | route target: l3vni (vrf) | overlay encapsulation: 8 - vxlan | router mac of remote vtep
				 	
				 	 originator: 10.0.0.1 czuster list: 10.0.0.111
				 
				 route type 3 - inclusive multicast ethernet tag route* (used where we have no multicast enable)
				 	assistance to ingress replication
				 		ingress replication / head-end replication is used for multi-destination traffic (broadcast, unknown unicast, multicast)

				 	the following fields are part of the evpn prefix in the nlri
				 		ip address length
				 		originating router's ip address

				 	rd (1 octet)
				 	esi (10 octets)
				 	ethernet tag id (4 octets)
				 	ip address length (1 octet)
				 	originating router's ip address (4 or 16 octets)

				 route type 4 - ethernet segment route
				 
				 route type 5 - ip prefix route* (silent hosts without advertise mac use this and advertise all subnet)
				 	rd (8 octet)
				 	esi (10 octets)
				 	ethernet tag id (4 octets)
				 	ip prefix length (1 octet)
				 	ip prefix. (4 or 16 octets)
				 	gw ip address (4 or 16 octets)
				 	mpls label (3 octets)

				 	decouples ip prefix from mac (rt-2) and provides flexible advertisement of ipv4 and ipv6 prefixes with variable
				 	length

				 	the following fields are part of the evpn prefix in the nlri
				 		ip prefix length (0-32 bits for ipv4 or 0-128 bits for ipv6)
				 		ip prefix (ipv4 or ipv6)
				 		gw ip address
				 		mpls label (l3vni)

				 	v2# show bgp 12vpn evpn 192.168.2.0

				 	bgp routing table infermation for vrf default, address/family l2vpn evpn
				 	
				 	route distinguisher: 10.0.9.8:3
				 	
				 	bgp routing table entry for [5] : [0] : [0] : [24] : [192 68. 2. 0] : [0. 0.0.0]/224, version 3
				 	!route type: 5 - ip prefix | ethernet segment | identifier | ethernet tag | identifier |  ip prefix length | ip prefix | gw ip address
				 	
				 	paths: (1 available, best #1) *
				 	
				 	flags: (0x000002) on xmit-list, is not in 12rib/evpn, is locked

				 	advertised path-id 1
				 	
				 	path type: internal, path is valid, is best path, no labeled nexthop
				 	
				 	as-path: none, path sourced internal to as
				 	
				 	10.0.0.1 (metric 3) from 10.0.0.111 (10.0.0.111)
				 	
				 	origin incomplete, med 0, localpref 100, weight 0
				 	
				 	received laber 50001
				 	!l3vni
				 	
				 	extcommunity: rt: 65501:50001 encap:8 router mac: 5087.89d4.5495
				 	!remote vtep ip address | route target: l3vni (vlan) | overlay encapsulation: 8 - vxlan | router mac of remote vtep
				 	
				 	originator: 10.0.0.1 ciaster list: k0.0.0.111

			 leaf-1
			 	show bgp l2vpn evpn
			 	!show us route-type 2

			 	show bgp l2vpn evpn 172.16.141.10 
			 	!show us mac 
				-------------------------------------------------
				host attaches
				 	host "a" attaches to edge device (vtep)

				
				 *vteps receive respective reachability information and installs them related to route-policy into rib/fib

				 *if multiple vtep announce same ip prefix,equal cos multipath (ecmp) will apply

				 *vtep v1 advertises local subnet through redistribution of "direct" (connected) routes 
				 	ip prefix, ip prefix length, and l3vni

				 *additional route attributes advertised
				 	mpls label (l3vni)
				 	extended communities

	leaf and spine architecture
		access aggregation core model
			3 layer or tier design and north -south traffic flow

			medium scale up

			limited multitenancy

			just use vss or vpc or stacking some switches (maximum support 2 switches)

		leaf and spine model
			east-west traffic flow

			high scale out

			mobility

			huge cloning data

			multitenancy 

			leaf is core and spine is access 

			links between leaf and spine we have layer 3 and layer 2

			has no connection between leafs except vpc

			has no connection between spines except vpc

			advnatages
				capacity and spaces will be solved with adding some spine

				if need server ports just add new leaf
***************************
distributed anycast gateway
	gateways are on leaf switches (closest to end point) and use same ip and mac address 
	distributed routing and disaggregate state achieved by scale out
	gateway is always active no redundancy protocol, hello exchange 
	doesn't need redundancy
	distributed state - smaller arp tables only local attached end-points (servers)
	has faster access to tor (top of rack) so help us manage arp table on smaller scope

	anycast means assigned address on many palces which reachable for end point as nearest
	nearest means routing protocol distance like bgp attributes
	in ospf metric is our anycast parameter	

	routing between different subnet (vni)
	briging within same subnet (vni)

	inter-vxlan routing at access layer (leaf)

	config
		vlan 43
		vn-segment 30001
		
		vlan 55
		vn-segment 30002
		!vlan to vni mapping

		fabric forwarding anycast-gateway-mac 0302.0032.0002
		!svi using fabric forwarding

		interface vlan 43
		no shutdown
		vrf member vrf-a
		ip address 11.11.11.1/24 tag 21921
		fabric forwarding mode anycast-gateway

		interface vlan 55
		no shutdown
		vrf member vrf-a
		ip address 98.98.98.1/24 tag 21921
		fabric forwarding mode anycast-gateway
		!distributed ip anycast gateway svi
***************************
data plane and unicast forwarding
	intra-subnet forwarding (brdging)
		same vni

		before anything must run bgp to learn vni and mac and ip address

												route reflector
		vm-1 --- vtep-1 (switch-1) ----------------- rr -------------------- vtep-2 (switch-2) --- vm-2
		-------------------------- 											 ---------------------------
		mac a ip a l2vni 3000 l3vni 5000 nh local 							 mac b ip b 	l2vni 3000 l3vni 5000 nh local

		mac b ip b l2vni 3000 l3vni 5000 nh ip-vtep-2						 mac a ip a 	l2vni 3000 l3vni 5000 nh ip-vtep-1

		we have alll mapping on bgp over leaf 1 so check the destination and detect the vni so transfer the main traffic over udp tunnels
			source ip is vtep 1 and destination ip is vtep 2 for tunnel

			source mac is vtep 1 and destination mac is spine which connect leaf 1 and leaf 2 to eachother

			this vni 3000 is main detection parameter for layer 2 or brdging mechanism

	inter-subnet forwarding (routing)
		different vni or different subnets

												route reflector
		vm-1 --- vtep-1 (switch-1) ----------------- rr -------------------- vtep-2 (switch-2) --- vm-2
		-------------------------- 											 ---------------------------
		mac a ip a l2vni 3000 l3vni 5000 nh local 							 mac b ip b 	l2vni 3001 l3vni 5000 nh local

		mac b ip b l2vni 3001 l3vni 5000 nh ip-vtep-2						 mac a ip a 	l2vni 3000 l3vni 5000 nh ip-vtep-1

		before everything must run bgp and know all parameters

		here vxlan detect layer 2 vni is different
		so check layer 3 vni and sense they are same

		after tunneling we can see tunnels are over l3vni

		per vrf we have just one vni (which is scoped) and each vlan over each vrf has different l2 vni

		when traffic reached destination on same l3vni and different l2vni findout these are on different subnets 

		asymmetric routing with consistent configuration 
			like traditional networks and not useful
			like intervlan routing
			need consistent config

			post routed traffic will leverage destination layer 2 segment (l2vni), same as for bridged traffic
	
		symmetric routing with scoped configuration (usually use this)
			doesn't need define all l2vni on all leafes
			inconsistent config

		*recommended use one l3vni pervrf and intervlan routing works on l3vni, has no dependencies on routing or brdgingat the any conditions use l3vni

		*consistent config
			every vni every where

		*inconsistent config
			scope and optimal scale
***************************
dual home support
	customers buy 2 vpls links can use just one of those if bought 2 vxlan link could use 2 of them
	we aggregate 2 leaf with vpc on nexus switches

	*normally leaf switches doesn't connect each other traditional methods

	vpc concept , forwarding, orphan hosts

	from servers see one switch 
	between 2 leaf we have peer-link like etherchannel layer 2 so must forward all vlans bydefault
	we have vtep addresses which define and advertise hosts behinde the vteps 
	here need set secondary ip address for anycast ip that introduce by pe to fabric
	on peer links must set routing protocol as underlay
	make svi over vlan and run  routing protocol over them
	bgp can advertise to one of those routers on vpc
	secondary ip used for fabric negotiation and vxlan connectivities
	instead of use vtepp 1 or vtp x for destination or source must use virtual or secondary ip address

	peer link is important
		host a is single-attached to v1 (aka orphan host)
			mac and ip address is associated with the anycast ip address as the vtep

		source ip address for vxlan encapsulation will be the anycast ip address; consistent for vpc domain
			traffic returning to host a will use anycast ip	as destination

		if traffic gets hashed to v2, traffic will be de-capsulated and transported as classic ethernet (ce) via the vpc peer-link

		host a and host b are single-attached to v1 and v2 respectively (aka orphan host)
			mac and ip address is associated with the anycast ip address as the vtep

		communication between host a and host b	will travel in classic ethernet (ce) via the vpc peer-link
			the respective vlans should be allowed on the vpc peer-link
***************************
bum (broadcast, unicast, multicast) forwarding
	unicast
		replicate many times
		not scaleable without pim

	muticast
		run pim sparse mode onver underlay
		recommended
		for each vni create multi destination tree

	forwarding arp to mac address on another vtep will be used arp proxy concept so arp traffic will be not directly forward as classical method and beyond our links
	arp will be terminate on our links and use arp proxy instead of normal arp

	multicast method
		per vni we have different multicast group 
		by sending igmp report to specific address request and fetch the groups
		doesn't create igmp tree to vteps if doesn't exist
		cause we set vni over vteps our tree and traffics will be orgenazied
		just one time send

		interface nve1
		no shutdown
		source-interface loop 0
		host-reachability protocol bgp
		member vni 30000
		mcast-group 239.1.1.1
		member vni 30001
		mcast-group 239.1.1.1

	unicast method
		replication has too much load
		send route type 3 for each vni with bgp advertise that behinde ip has vni x
		has many packet generation
		scaleability issue
		bgp create replication list

		interface nve1
		no shutdown
		source-interface loop 0
		host-reachability protocol bgp
		member vni 30000
		ingress-replication protocol bgp
		member vni 30001
		ingress-replication protocol bgp

		label: 30000, tunnel id: 10.0.0.1

		#sh bgp 12vpn evpn 10.0.0.1
			bgp routing table information for vrf default, address family l2vpn evpn
			
			route distinguisher: 10.0.8,1:32800 (l2vni 30000)
			
			bgp routing table entry for [3]: [0] : [32] : [10.0 .. 1]/88, version 75
			!route type:3 - inclusive multicast | ip address length | ip address
			
			paths: (1 available, best #1)
			
			flags: (0x00000a) on xmit-list, is not in 12rib/evpn

			advertised path-id 1
			
			path type: local, path is valid, is best path, no labeled nexthop
			
			as-path: none, path locally criginated
			
			10.0.0.1 (metric 0) from 0.c.0.0 (10.0.0.1)
			
			origin igp, med not set, localpref 100, weight 32768
			
			extcommunity: rt:65501:30000
			!route target l2vni vlan
			
			pmsi tunnel attribute:a
			
			flags: 0x00, tunnel type: ingress replication
			!tunnel type
			
			label: 30000, tunnel id: 10.0.0.1
			!l2vni | vtep address
***************************
underlay
	vxlan is overlay
	vxlan use 50 byte as overhead
	avoid fragmentation by adjusting the ip networks mtu
	data centers often require jumbo mtu most server nics do support up to 9000 bytes
	using a mtu of 9216 bytes accommodates vxlan overhead plus server maximum mtu

	ip addressing
		know your ip addressing and ip scale requirements
			best to use single aggregate for all underlay links and loopbacks

			ipv4 only

			for each point-2-point (p2p) connection, minimum /31 required

			loopback requires /32

		routed ports/interfaces
			layer 3 interfaces between spine and leaf (no switchport)

		vtep uses loopback as source-interface

	*better use unnumbered and use less ip address

	routing protocol 
		ospf
			leaf
				interface loop 0
				ip address 10.10.10.1/32
				mtu 9192
				ip router ospf 1 area 0.0.0.0
				ip ospf network point-to-point

				interface ethernet 2/1
				no switchport
				ip address 192.168.1.1/31
				mtu 9192
				ip router ospf 1 area 0.0.0.0
				ip ospf network point-to-point
				!point-2-point (p2p) interface configuration

				*bypass dr and bdr

		isis
			leaf
				interface loopback 0
				ip address 10.10.10.1/32
				mtu 9192
				ip router isis 1

				interface ethernet 2/1
				no switchport
				ip address 192.168.1.1/31
				mtu 9192
				ip router isis 1
				!point-2-point (p2p) interface configuration

				*just use level 1

		ibgp + igp
			igp for underlay topology & reachability (is-is, ospf)

			ibgp for vtep (loopback) reachability

			ibgp route-reflector for simplification and	scale

			requires two routing protocols

		ebgp
			ebgp peer is ip interface
				loopback would require additional igp and ebgp multi-hop

			multiple autonomous systems (as)
				minimum amount of as is two

			many bgp neighbors
				for each neighboring p2p interface

			no route-reflector

	multicast
		pim-asm or pim-bidir (different hardware has different capabilities)
		
		spine and aggregation switches make good rendezvous-point (rp) much like rr
		
		pim-asm (sparse-mode)
			source-trees build a couple of unidirectional trees from rp (s,g)
			every vtep is source and destination
			pim-anycast rp vs msdp for example
		
		pim-bidir
			no sources tree use a bi-directional shared tree
			no (s,g) we have (*,g)
			phanton rp (leverages unicast for convergence)
		
		*each vni does not need the same a different multicast group

		nx 1kv > igmpv2/3
		nx 3k > pim asm
		nx 5600 > pim bidir
		nx 7k/f3 > pim asm / bidir
		nx 9k > pim asm
		asr 1k / csr 1k > pim bidir
		asr 9k > pim asm / bidir

		pim sparse
			spine
				ip pim rp-address 10.10.10.anycast
				ip pim anycast-rp 10.10.10.anycast 10.10.10.s1
				ip pim anycast-rp 10.10.10.anycast 10.10.10.s2
				!anycast-rp configuration
			
				interface loopback 0
				ip address 10.10.10.s1/32
				mtu 9192
				ip pim sparse-mode
				!loopback interface configuration (rp)
			
				interface loopback 1
				ip address 10.10.10.anycast/32
				mtu 9192
				ip pim sparse-mode
				!loopback interface configuration
			
			leaf
				ip pim rp-address 10.10.10 aanycast
				!using anycast rendezvous-point

				interface loopback 0
				ip address 10.10.10.l1/32
				mtu 9192
				ip pim sparse-mode
				!loopback interface configuration (vtep)

				interface ethernet 2/1
				no switchport
				ip address 192.168.1.1/31
				mtu 9192
				ip pim sparse-mode
				!point-2-point (p2p) interface configration
 
		pim bidir
			spine
				ip pim rp address 10.10.10.anycastl bidir group 239.0.0.0/24
				ip pim rp address 10.10.10.anycast2 bidir group 239.1.1.0/24
				!using phantom rendezvous-point

				interface loopback 0
				ip address 10.10.10.91/32
				ip pim sparse-mode
				!loopback interface configuration (rp) (redundancy)

				interface loopback 1
				ip address 10.10.10.anycast1 /32
				ip pim sparse-mode
				!loopback interface configuration (anycast1 rp - lb)

				interface loopback 2
				ip address 10.10.10.anycast2 /31
				ip pim sparse-mode
				!loopback interface configuration (anycast2 rp - lb)

				ip pim rp address 10.10.10.anycast1 bidir group 239.0.0.0/24
				ip pim rp address 10.10.10.anycast2 bidir group 239.1.1.0/24
				!using phantom rendezvous-point

				interface loopback 0
				ip address 10.10.10.82/32
				ip pim sparse-mode
				!loopback interface configuration (rp) (redundancy)

				interface loopback 1
				ip address 10.10.10.anycast1 /31
				ip pim sparse-mode
				!loopback interface configuration (anycast1 rp - lb)

				interface loopback 2
				ip address 10.10.10.anycast2 /32
				ip pim sparse-mode
				!loopback interface configuration (anycast2 rp - lb)
			
			leaf
				ip pim rp address 10.10.10.anycast1 bidir group 239.0.0.0/24
				ip pim rp address 10.10.10.anycast2 bidir group 239.1.1.0/24
				!using phantom rendezvous-point

				interface loopback 0
				ip address 10.10.10.11/32
				mtu 9192
				ip pim sparse-mode
				!loopback interface configuration (vtep)

				interface ethernet 2/1
				no switchport
				ip address 192.168.1.1/31
				mtu 9192
				ip nim marse-mode
				!point-2-point (p2p) interface configration

		*broadcast/unknown-unicast/multicast (bum) traffic in a vxlan overlay network can be transported through the underlay network

		*each vni is mapped to a multicast group. bum traffic in the vni will be encapsulated into multicast packets using this multicast  as the outer destination ip address and then sent to the remote vteps using the underlay network multicast replication and forwarding

		flood and learn mode vxlan
			vlan 2
			vn-segment 4098
			interface nve 1
			menber vni 10000
			ncast-group 225.1.1.1

		vxlan evpn:
			vlan 200
			vn-segment 20000
			interface nve 1
			host-reachability protocol
			bgp
			member vni 20000
			mcast-group 225.1.1.1

	no multicast
		broadcast/unknown-unicast/multicast (bum) traffic in a vxlan overlay network can be transported through the underlay network

		ingress replication (ir) ingress vtep generate a unicast copy of the overlay bum packet for each remote vteps and send them through the underlay network using unicast forwarding

		in flood-&-learn mode vxlan remote vteps for ir are statically configured

		in vxlan evpn remote vteps automatically learn through evpn inclusive multicast ethernet tag (imet) routes

		flood-&-learn mode vxlan:
			vlan 200
			vn-secment 20000
			interface nve 1
			member vni 20000
			ingress-replication protocol static

		*use route type 3

		vxlan evpn:
			vlan 200
			vn-segment 20000
			interface nve 1
			host-reachability protocol bgp
			member vni 20000
			ingress-replication protocol bgp

		doesn't flood just send to all same vni without route type 3 and doesn't send bum traffic

		if multicast is not available to enable / deploy

		smaller scale requirements limited number of vtep peers and limited number of vni(s)

		customer does not want to enable multicast in their environment

		where it is ok to multiply packets (plenty of bandwidth)

		need to remember ingress replication does not conserve bandwidth as the fabric scales addition bum traffic will be placed on the fabric
***************************
external connectivity
	how inject external networks (north to south traffic flow) into the vxlan
		l2 
			spanning-tree and vxlan
				 vxlan has no integration with spanning-tree for loop protection
				 vxlan does not forward bpdu

				 loop-free topologies required southbound of vxlan edge-devices
				 use vpc to provide ethernet-based loop-free topologies

			might have loop on layer 2 cause we have multihome connection for servers and end points
			recommended don't use multihome if use them with  spanning-tree and vxlan consider these
					virtual port-channel (vpc) will allow safe integration with spanning-tree
					no loop-protection required as per logical loop-free topology

					follow best practices to protect the network border as in classic ethernet
						networks
						bpdu guard
						root guard
						storm control

			if wanna use vpc
				provides redundant connectivity for various use-cases
					interconnect using ieee 802.1q for extensions via layer-2 technologies
						otv
						vpls
						eompls

					redundancy for layer4-7 services appliance 
						lb
						fw

					legacy network for migration or interconnection

		l3 
			vrf lite
				aka inter-as option a
				
				provides connectivity for external routing connectivity
					interconnect using subinterfaces for multitenant capable handoff 
					per-vrf routing adjacency based on ieee 802.1q tagging
					various routing protocols available (ebgp, ospf)
				
				could be combined with vpc for layer 2 connectivity

				per vrf needed for each network on each leaf and per vrf run different routing protocol

				border leaf
					interface ethernet1/1
					no switchport
					
					interface ethernet1/1.10
					mtu 9216
					encapsulation dotlq 10
					vrf member vrf-a
					ip address 10.254.254.1/30

					router bgp 65500
					!ebgp configuration

					vrf vrf-a
					address-family ipv4 unicast
					advertise 12vpn evpn
					aggregate-address 10.0.0.0/8 summary-only
					neighbor 10.254.254.2 remote-as 65599
					neighbor 10.254.254.2 update-source ethernet1/1.10
					neighbor 10.254.254.2 peer-type fabric-external
					address-family ipv4 unicast
					neighbor 10.254.254.2 send-community both

				external switch
					interface ethernet1/1
					encapsulation dot1q 10
					vrf member vrf-a
					ip address 10.254.254.2/30

					router bgp 65599
					!ebgp configuration

					vrf vrf-a
					address-family ipv4 unicast
					neighbor 10.254.254.1 remote-as 65500
					neighbor 10.254.254.1 update-source ethernet1/1
					address-family ipv4 unicast

			lisp
				locator identifier seperation protocol
				useful on mobility
				ip also will be used as location
				change location needed new ip address 

				mobility means after change location our identifier be fix and location get dynamic (used in isp)
				scaleability help us to route our locations not identifiers
					need resolution resolver from database

				security
				dci


				pull based model
				only interested parties (router) ask for the information
				small state in the router tables (conserves	hardware table space)


				traditional routing protocols
					push based model
					all routers become immediate updates on routing change
					large state router tables (hardware) based on routing policy

				applicated in vxlan
					we have 2 datacenter with as-number 65000 and 65001

					bgp update and lisp update
					use gratutitous arp will be used
					map system will be update with bgp and mapping system

			mpls
				similar to inter-as option b
				provides l3vpn connectivity via mpls integration
					interconnect using l3vpn for multi-tenant capable hand-off
					uses different bgp as# as within evpn fabric
					re-originates evpn into l3vpn address-family
				
				could be combined with vpc for layer-2 connectivity

				border leaf used here and must set import and export for some routes
				like mpls pe asbr and border leaf
					vrf context a
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target import 65599:1
					route-target export 65599:1
					route-target import auto evpn
					route-target export auto evpn

					router bgp 65500
					neighbor 10.254.254.2 remote-as 65599
					neighbor 10.254.254.2 update-source loopback0
					address-family vpnv4 unicast
					import 12vpn evpn
					neighbor 10.0.0.1 remote-as 65500
					address-family l2vpn evpn
					neighbor 10.0.0.1  import vpn unicast
					!ebgp configuration
					!import to each other
				
		use border leafs for external links and external connectivity
		a leaf with normal vxlan behavior and connect to external network
		border leafs has best performance for isolating north-south traffic flow

		why don't have border spine ?
			overload for  east-west and north-south traffic 

			multiple roles
				external connectivity
				route reflector
				rendezvous point
				spine
				potentially l4-l7 services (fw, lb)
***************************
labratory
	underlay
		spine-1 |----- 10.1.11.0/24 -------- leaf-1 -------- 10.2.11.0./24 -----| spine-2
				|								  								|
				|----- 10.1.12.0/24 -------- leaf-2 -------- 10.2.12.0./24 -----| 
				| 								 								|
				|----- 10.1.13.0/24 -------- leaf-3 -------- 10.2.13.0./24 -----| 
				| 								   								|
				|----- 10.1.14.0/24 -------- leaf-4 -------- 10.2.14.0./24 -----| 

		vmware esxi 
		nxos 9.2.4
			8 gig ram 
			4 core cpu

		for management need serial port for virtual machines
			must conencted to vmkernel and vswitch to bind on them
			must power off machines
			vmk0 > management 172.16.121.12

			use network as server for serial port
			serial port on virtual machines > telnet://172.116.121.12:2000

		on cli
			*if see these means ok
				loading [1271m/1271m]
				[initrd, addr=0x30354000, size=0x4f7a9000]

				segment header
				length: 4, vendor: 16 flags: 4, loadaddr: 2500000, image len: 800, memory length
				: 800
				reading data for vendor seg. length 2048
				leaving grub land

				image length read 1339749888

				image hash: c039454 a6c5954b 80acb297 f0d145a2

			*might goes to boot loader
				dir bootflash:
				dir
				boot nxos.9.2.4.bin
				config
				boot nxos nxos.9.2.4.bin
				show version

		after these use telnet 

		must add each leaf and spine link on different vlan tag and port group for multicast target

		esxi > network > properties
			create port group as each each conenction to each links 2 spine and 4 leafs means 8 port group
			for each spine connectivity toward leafs must add port groups then add vlans 
			each spine must be connect to this port group to run ospf

		configs
			spine-1
				hostname spine-1

				interface loopback0
				description ** ospf unnderlay **
				ip address 192.168.0.1/32

				feature ospf

				router ospf underlay-net
				router-id 192.168.0.1

				interface ethernet1/1
				description to leaf1
				no switchport
				mtu 9216
				ip address 10.1.11.1 255.255.255.0
				no shutdown
				no switchport
				ip ospf network point-to-point
				no ip ospf passive-interface
				ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				description to leaf2
				no switchport
				mtu 9216
				ip address 10.1.12.1 255.255.255.0
				no shutdown
				ip ospf network point-to-point
				no ip ospf passive-interface
				ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/3
				description to leaf3
				no switchport
				mtu 9216
				ip address 10.1.13.1 255.255.255.0
				no shutdown
				ip ospf network point-to-point
				no ip ospf passive-interface
				ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/4
				description to leaf4
				no switchport
				mtu 9216
				ip address 10.1.14.1 255.255.255.0
				no shutdown
				ip ospf network point-to-point
				no ip ospf passive-interface
				ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				description ** bgp evpn peering address **
				ip address 192.168.15.1/32
				ip ospf network point-to-point
				ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				description ** vtep ip address **
				ip address 192.168.30.1/32
				ip ospf network point-to-point
				ip router ospf underlay-net area 0.0.0.0
			
			spine-2
				hostname spine-2

				interface loopback0
				 description ** ospf unnderlay **
				 ip address 192.168.0.2/32

				feature ospf

				router ospf underlay-net
				 router-id 192.168.0.2

				interface ethernet1/1
				 description to leaf1
				 no switchport
				 mtu 9216
				 ip address 10.2.11.2 255.255.255.0
				 no shutdown
				 no switchport
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				 description to leaf2
				 no switchport
				 mtu 9216
				 ip address 10.2.12.2 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/3
				 description to leaf3
				 no switchport
				 mtu 9216
				 ip address 10.2.13.2 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/4
				 description to leaf4
				 no switchport
				 mtu 9216
				 ip address 10.2.14.2 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				 description ** bgp evpn peering address **
				 ip address 192.168.15.2/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				 description ** vtep ip address **
				 ip address 192.168.30.2/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0
				 
			leaf-1
				hostname leaf1

				interface loopback0
				 description ** ospf unnderlay **
				 ip address 192.168.0.11/32

				feature ospf

				router ospf underlay-net
				 router-id 192.168.0.11

				interface ethernet1/1
				 description to spine1
				 no switchport
				 mtu 9216
				 ip address 10.1.11.11 255.255.255.0
				 no shutdown
				 no switchport
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				 description to spine2
				 no switchport
				 mtu 9216
				 ip address 10.2.11.11 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				 description ** bgp evpn peering address **
				 ip address 192.168.15.11/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				 description ** vtep ip address **
				 ip address 192.168.30.11/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				feature bgp
				feature nv overlay
				feature vn-segment-vlan-based
				nv overlay evpn

			leaf-2
				hostname leaf2

				interface loopback0
				 description ** ospf unnderlay **
				 ip address 192.168.0.12/32

				feature ospf

				router ospf underlay-net
				 router-id 192.168.0.12

				interface ethernet1/1
				 description to spine1
				 no switchport
				 mtu 9216
				 ip address 10.1.12.12 255.255.255.0
				 no shutdown
				 no switchport
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				 description to spine2
				 no switchport
				 mtu 9216
				 ip address 10.2.12.12 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				 description ** bgp evpn peering address **
				 ip address 192.168.15.12/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				 description ** vtep ip address **
				 ip address 192.168.30.12/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

			leaf-3
				hostname leaf3

				interface loopback0
				 description ** ospf unnderlay **
				 ip address 192.168.0.13/32

				feature ospf

				router ospf underlay-net
				 router-id 192.168.0.13

				interface ethernet1/1
				 description to spine1
				 no switchport
				 mtu 9216
				 ip address 10.1.13.13 255.255.255.0
				 no shutdown
				 no switchport
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				 description to spine2
				 no switchport
				 mtu 9216
				 ip address 10.2.13.13 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				 description ** bgp evpn peering address **
				 ip address 192.168.15.13/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				 description ** vtep ip address **
				 ip address 192.168.30.13/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

			leaf-4
				hostname leaf4

				interface loopback0
				 description ** ospf unnderlay **
				 ip address 192.168.0.14/32

				feature ospf

				router ospf underlay-net
				 router-id 192.168.0.14

				interface ethernet1/1
				 description to spine1
				 no switchport
				 mtu 9216
				 ip address 10.1.14.14 255.255.255.0
				 no shutdown
				 no switchport
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface ethernet1/2
				 description to spine2
				 no switchport
				 mtu 9216
				 ip address 10.2.14.14 255.255.255.0
				 no shutdown
				 ip ospf network point-to-point
				 no ip ospf passive-interface
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback15
				 description ** bgp evpn peering address **
				 ip address 192.168.15.14/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

				interface loopback30
				 description ** vtep ip address **
				 ip address 192.168.30.14/32
				 ip ospf network point-to-point
				 ip router ospf underlay-net area 0.0.0.0

	///////////////////////	

	overlay with ibgp
		spine-1 |----- 10.0.0.22/30 -------- leaf-1 -------- 10.0.128.6/30 -----| spine-2								
				| 								 |  
				|								 |--------------------------------------- srv-1
				|	
				|----- 10.0.0.24/30 -------- leaf-2 -------- 10.0.128.10/30 -----| 
				| 								 								 |
				|----- 10.0.0.30/30 -------- leaf-3 -------- 10.0.128.14/30 -----| 
				| 								 |  
				|								 |--------------------------------------- srv-2 
				|	
				|----- 10.0.128.2/30 -------- leaf-4 -------- 10.0.128.18/30 -----| 
												|
											10.0.0.18/30
												|
												wan ------------------------------------- srv-3 172.21.1.10
											192.168.0.5

				as-number 65000

 				spine-1 loopback 192.168.0.6
				spine-2 loopback 192.168.0.7

				leaf-1 loopback 1 > 192.168.0.18 and loopback 0 > 192.168.0.8
				leaf-2 loopback 1 > 192.168.0.19 and loopback 0 > 192.168.0.9
				leaf-3 loopback 1 > 192.168.0.110 and loopback 0 > 192.168.0.10
				leaf-4 loopback 1 > 192.168.0.111 and loopback 0 > 192.168.0.11

		use symmetric routing
		overlay use ibgp 
		underlay is ospf
		bum is multicast
		ospf rid , ibgp neighborship will be on loop interface ip address

		ibgp will be on leafs and spines are route reflector
		our leafs will be advertise by route reflector
		we can use vxlan without bgp which in flooding learn macs

		we use anycast rendezvous point for scenario recommended use pim sparse (better performance) mode but could use pim bidir (scaleable)

		config
			spine-1
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.6
				address-family ipv4 unicast
				template peer leaf-peer
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				route-reflector-client
				neighbor 192.168.0.8
				inherit peer leaf-peer
				neighbor 192.168.0.9
				inherit peer leaf-peer
				neighbor 192.168.0.10
				inherit peer leaf-peer
				neighbor 192.168.0.11
				inherit peer leaf-peer

				show ip bgp summary

				!------------- pim-bidir ------------------
					feature pim
					interface loopback1
					ip address 192.168.0.100/32
					ip pim sparse-mode
					ip router ospf 1 area 0.0.0.0
					ip pim rp-address 192.168.0.100
					ip pim anycast-rp 192.168.0.100 192.168.0.6
					ip pim anycast-rp 192.168.0.100 192.168.0.7
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface e1/3
					ip pim sparse-mode
					interface e1/4
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- bgp ------------------
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					!when routers received packets must regenerate them and forward to destination, our route targets get replaced by default if set this command means bypass the replacing and forwarding as they are, cause route reflectors has no route target
	
					template peer leaf-peer
					address-family l2vpn evpn
					send-community both
					route-reflector-client

			spine-2
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.7
				address-family ipv4 unicast
				template peer leaf-peer
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				route-reflector-client
				neighbor 192.168.0.8
				inherit peer leaf-peer
				neighbor 192.168.0.9
				inherit peer leaf-peer
				neighbor 192.168.0.10
				inherit peer leaf-peer
				neighbor 192.168.0.11
				inherit peer leaf-peer

				show ip bgp summary

				!------------- pim-bidir ------------------
					feature pim
					interface loopback1
					ip address 192.168.0.100/32
					ip pim sparse-mode
					ip router ospf 1 area 0.0.0.0
					ip pim rp-address 192.168.0.100
					ip pim anycast-rp 192.168.0.100 192.168.0.6
					ip pim anycast-rp 192.168.0.100 192.168.0.7
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface e1/3
					ip pim sparse-mode
					interface e1/4
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- bgp ------------------
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					!when routers received packets must regenerate them and forward to destination, our route targets get replaced by default if set this command means bypass the replacing and forwarding as they are, cause route reflectors has no route target
	
					template peer leaf-peer
					address-family l2vpn evpn
					send-community both
					route-reflector-client

			leaf-1
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.8
				address-family ipv4 unicast
				neighbor 192.168.0.6
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				neighbor 192.168.0.7
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both

				show ip bgp summary

				!------------- pim-bidir ------------------
					feature pim
					ip pim rp-address 192.168.0.100
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
					interface loopback1
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- vxlan ------------------
					config t
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn
	
					spanning-tree vlan 1,140,141,999 priority 4096
					!these spanning-tree will be root for vxlan fabric 
	
					vlan 140
					name l2-vni-140-tenant1
					vn-segment 50140
					!use for server 1, layer 2 vni
	
					vlan 141
					name l2-vni-141-tenant1
					vn-segment 50141
					!use for server 2, layer 2 vni
	
					vlan 999
					vn-segment 50999
					!use for intervlan routing, layer 3 vni and symmetric routing , doesn't need on symmetric routing but cisco support this
					!doesn't need define on spine cause use spines as passthrough way access the nexthops
	
					vrf context tenant-1
					vni 50999
					rd auto
					!use for ovelaping mac and ip between different vrf, auto mode means bydefault use vrf id and router id for isolation
	
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn
					!bgp use this to place routes for which customer, auto use as-number and vni number for isolation and advertisement
					!here exporting frome one link will be imported to all another links
	
					fabric forwarding anycast-gateway-mac 0000.2222.3333
	
					interface vlan140
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.140.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan141
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.141.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan999
					no shutdown
					vrf member tenant-1
					ip forward
					!per vrf we have one vni layer 3 to make symmetric routing
	
					interface nve1
					!used for layer 2 and layer 3 multipoint connectivity
	
					no shutdown
					source-interface loopback1
					!use vtep address 192.168.0.18
	
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					interface nve1
					no shutdown
					source-interface loopback1
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					show nve vni
					!has 2 layer 2 vni tunnel one is 50140 another is 50141 and layer 3 is 50999

				!------------- bgp ------------------
					config t
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					neighbor 192.168.0.6
					remote-as 65000
					address-family l2vpn evpn
					send-community both
					neighbor 192.168.0.7
					remote-as 65000
					address-family l2vpn evpn
					send-community both

				!------------- rd and rt for mac ------------------
					evpn
					vni 50140 l2
					rd auto
					route-target import auto
					route-target export auto
	
					vni 50141 l2
					rd auto
					route-target import auto
					route-target export auto
	
					show bgp l2vpn evpn summary

				!------------- verification ------------------
					int e1/3
					description to server-1
					switchport mode access
					switchport access vlan 140
	
					show nve peers 
					sh ip route vrf tenant-1 
					show bgp l2vpn evpn
					clear bgp l2vpn evpn *
					show l2route evpn mac-ip all

			leaf-2
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.9
				address-family ipv4 unicast
				neighbor 192.168.0.6
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				neighbor 192.168.0.7
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both

				show ip bgp summary

				!------------- pim-bidir ------------------
					feature pim
					ip pim rp-address 192.168.0.100
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
					interface loopback1
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- vxlan ------------------
					config t
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn
	
					spanning-tree vlan 1,140,141,999 priority 4096
					!these spanning-tree will be root for vxlan fabric 
	
					vlan 140
					name l2-vni-140-tenant1
					vn-segment 50140
					!use for server 1, layer 2 vni
	
					vlan 141
					name l2-vni-141-tenant1
					vn-segment 50141
					!use for server 2, layer 2 vni
	
					vlan 999
					vn-segment 50999
					!use for intervlan routing, layer 3 vni and symmetric routing , doesn't need on symmetric routing but cisco support this
					!doesn't need define on spine cause use spines as passthrough way access the nexthops
	
					vrf context tenant-1
					vni 50999
					rd auto
					!use for ovelaping mac and ip between different vrf, auto mode means bydefault use vrf id and router id for isolation
	
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn
					!bgp use this to place routes for which customer, auto use as-number and vni number for isolation and advertisement
					!here exporting frome one link will be imported to all another links
	
					fabric forwarding anycast-gateway-mac 0000.2222.3333
	
					interface vlan140
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.140.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan141
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.141.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan999
					no shutdown
					vrf member tenant-1
					ip forward
					!per vrf we have one vni layer 3 to make symmetric routing
	
					interface nve1
					!used for layer 2 and layer 3 multipoint connectivity
					
					no shutdown
					source-interface loopback1
					!use vtep address 192.168.0.19
	
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					interface nve1
					no shutdown
					source-interface loopback1
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					show nve vni
					!has 2 layer 2 vni tunnel one is 50140 another is 50141 and layer 3 is 50999

				!------------- bgp ------------------
					config t
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					neighbor 192.168.0.6
					remote-as 65000
					address-family l2vpn evpn
					send-community both
					neighbor 192.168.0.7
					remote-as 65000
					address-family l2vpn evpn
					send-community both

				!------------- rd and rt for mac ------------------
					evpn
					vni 50140 l2
					rd auto
					route-target import auto
					route-target export auto
	
					vni 50141 l2
					rd auto
					route-target import auto
					route-target export auto
	
					show bgp l2vpn evpn summary

			leaf-3
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.10
				address-family ipv4 unicast
				neighbor 192.168.0.6
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				neighbor 192.168.0.7
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both

				show ip bgp summary	

				!------------- pim-bidir ------------------
					feature pim
					ip pim rp-address 192.168.0.100
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
					interface loopback1
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- vxlan ------------------
					config t
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn
	
					spanning-tree vlan 1,140,141,999 priority 4096
					!these spanning-tree will be root for vxlan fabric 
	
					vlan 140
					name l2-vni-140-tenant1
					vn-segment 50140
					!use for server 1, layer 2 vni
	
					vlan 141
					name l2-vni-141-tenant1
					vn-segment 50141
					!use for server 2, layer 2 vni
	
					vlan 999
					vn-segment 50999
					!use for intervlan routing, layer 3 vni and symmetric routing , doesn't need on symmetric routing but cisco support this
					!doesn't need define on spine cause use spines as passthrough way access the nexthops
	
					vrf context tenant-1
					vni 50999
					rd auto
					!use for ovelaping mac and ip between different vrf, auto mode means bydefault use vrf id and router id for isolation
	
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn
					!bgp use this to place routes for which customer, auto use as-number and vni number for isolation and advertisement
					!here exporting frome one link will be imported to all another links
	
					fabric forwarding anycast-gateway-mac 0000.2222.3333
	
					interface vlan140
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.140.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan141
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.141.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan999
					no shutdown
					vrf member tenant-1
					ip forward
					!per vrf we have one vni layer 3 to make symmetric routing
	
					interface nve1
					!used for layer 2 and layer 3 multipoint connectivity
					
					no shutdown
					source-interface loopback1
					!use vtep address 192.168.0.110
	
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					interface nve1
					no shutdown
					source-interface loopback1
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					show nve vni
					!has 2 layer 2 vni tunnel one is 50140 another is 50141 and layer 3 is 50999

				!------------- bgp ------------------
					config t
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					neighbor 192.168.0.6
					remote-as 65000
					address-family l2vpn evpn
					send-community both
					neighbor 192.168.0.7
					remote-as 65000
					address-family l2vpn evpn
					send-community both

				!------------- rd and rt for mac ------------------
					evpn
					vni 50140 l2
					rd auto
					route-target import auto
					route-target export auto
	
					vni 50141 l2
					rd auto
					route-target import auto
					route-target export auto
	
					show bgp l2vpn evpn summary

				!------------- verification ------------------
					int e1/3
					description to server-2
					switchport mode access
					switchport access vlan 140
					//////////////////////////
					int e1/3
					description to server-2
					switchport mode access
					switchport access vlan 141
	
					show nve peers 
					sh ip route vrf tenant-1 
					show bgp l2vpn evpn
					clear bgp l2vpn evpn *
					show l2route evpn mac-ip all

			leaf-4
				config t
				feature bgp
				router bgp 65000
				router-id 192.168.0.11
				address-family ipv4 unicast
				neighbor 192.168.0.6
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both
				neighbor 192.168.0.7
				remote-as 65000
				update-source loopback0
				address-family ipv4 unicast
				send-community both

				show ip bgp summary

				!------------- pim-bidir ------------------
					feature pim
					ip pim rp-address 192.168.0.100
					interface e1/1
					ip pim sparse-mode
					interface e1/2
					ip pim sparse-mode
					interface loopback0
					ip pim sparse-mode
					interface loopback1
					ip pim sparse-mode
	
					show ip pim neighbor

				!------------- vxlan ------------------
					config t
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn
	
					spanning-tree vlan 1,140,141,999 priority 4096
					!these spanning-tree will be root for vxlan fabric 
	
					vlan 140
					name l2-vni-140-tenant1
					vn-segment 50140
					!use for server 1, layer 2 vni
	
					vlan 141
					name l2-vni-141-tenant1
					vn-segment 50141
					!use for server 2, layer 2 vni
	
					vlan 999
					vn-segment 50999
					!use for intervlan routing, layer 3 vni and symmetric routing , doesn't need on symmetric routing but cisco support this
					!doesn't need define on spine cause use spines as passthrough way access the nexthops
	
					vrf context tenant-1
					vni 50999
					rd auto
					!use for ovelaping mac and ip between different vrf, auto mode means bydefault use vrf id and router id for isolation
	
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn
					!bgp use this to place routes for which customer, auto use as-number and vni number for isolation and advertisement
					!here exporting frome one link will be imported to all another links
	
					fabric forwarding anycast-gateway-mac 0000.2222.3333
	
					interface vlan140
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.140.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan141
					no shutdown
					vrf member tenant-1
					no ip redirects
					ip address 172.21.141.1/24
					fabric forwarding mode anycast-gateway
	
					interface vlan999
					no shutdown
					vrf member tenant-1
					ip forward
					!per vrf we have one vni layer 3 to make symmetric routing
	
					interface nve1
					!used for layer 2 and layer 3 multipoint connectivity
					
					no shutdown
					source-interface loopback1
					!use vtep address 192.168.0.111
	
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					interface nve1
					no shutdown
					source-interface loopback1
					host-reachability protocol bgp
					!means use bgp control the flooding
	
					member vni 50140
					mcast-group 239.0.0.140
					member vni 50141
					mcast-group 239.0.0.141
					member vni 50999 associate-vrf
	
					show nve vni
					!has 2 layer 2 vni tunnel one is 50140 another is 50141 and layer 3 is 50999

				!------------- bgp ------------------
					config t
					router bgp 65000
					address-family l2vpn evpn
					retain route-target all
					neighbor 192.168.0.6
					remote-as 65000
					address-family l2vpn evpn
					send-community both
					neighbor 192.168.0.7
					remote-as 65000
					address-family l2vpn evpn
					send-community both

				!------------- rd and rt for mac ------------------
					evpn
					vni 50140 l2
					rd auto
					route-target import auto
					route-target export auto
	
					vni 50141 l2
					rd auto
					route-target import auto
					route-target export auto
	
					show bgp l2vpn evpn summary

				!------------- insert externals ------------------
					router bgp 65000
					vrf tenant-1
					address-family ipv4 unicast
					advertise l2vpn evpn
					redistribute ospf 1 route-map permit-ospf-bgp
					router ospf 1
					!use this part as external connectivity
	
					vrf tenant-1
					redistribute bgp 65000 route-map permit-bgp-ospf
					redistribute direct route-map permit-bgp-ospf
					route-map permit-bgp-ospf permit 10
					route-map permit-ospf-bgp permit 10
					!use this part as external connectivity

					interface ethernet1/3
					mtu 9216
					vrf member tenant-1
					ip address 10.0.0.18/30
					ip ospf network point-to-point
					ip router ospf 1 area 0.0.0.0
					no shutdown
					router ospf 1
					router-id 192.168.0.11
					vrf tenant-1
	
					show ip ospf neighbor
	
			wan router
				config t
				int gig2
				mtu 9216
				description to leaf-4
				ip address 10.0.0.17 255.255.255.252
				ip ospf network point-to-point
				no shut
				exit

				router ospf 1
				router-id 192.168.0.5
				network 10.0.0.16 0.0.0.3 area 0

				show ip ospf neighbor

	///////////////////////	

	overlay with ebgp
		symmetric routing
		bum used unicast replication
		underlay is ospf
		overlay is ebgp
		make 3 loop back for ospf rid , bgp rid and vtep address

		spine-1 |----- 10.1.11.0/24 -------- leaf-1 -------- 10.2.11.0./24 -----| spine-2
				|								  								|
				|----- 10.1.12.0/24 -------- leaf-2 -------- 10.2.12.0./24 -----| 
				| 								 								|
				|----- 10.1.13.0/24 -------- leaf-3 -------- 10.2.13.0./24 -----| 
				| 								   								|
				|----- 10.1.14.0/24 -------- leaf-4 -------- 10.2.14.0./24 -----| 


					range 				spine-1 	spine-2 	leaf-1 		leaf-2 		leaf-3	 	leaf-4
					---------------------------------------------------------------------------------------
		loop 0 		192.168.0.0/24 		.1/32 		.2/32 		.11/32 		.12/32 		.13/32 		.14/32
					---------------------------------------------------------------------------------------
		loop 15 	192.168.15.0/24 	.1/32 		.2/32 		.11/32 		.12/32 		.13/32 		.14/32
					---------------------------------------------------------------------------------------
		loop 30 	192.168.30.0/24  					 		.11/32 		.12/32 		.13/32 		.14/32
					---------------------------------------------------------------------------------------

		spine-1 and spine-2 as-number 65000

		leaf-1 as-number 65011
		leaf-2 as-number 65012
		leaf-3 as-number 65013
		leaf-4 as-number 65014

		use loop 0 for ospf
		use loop 15 for bgp
		use loop 30 for vtep address

		configs
			spine-1
				!--------------- underlay ----------------
					hostname spine1

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.1/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.1

					interface ethernet1/1
					 description to leaf1
					 no switchport
					 mtu 9216
					 ip address 10.1.11.1 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to leaf2
					 no switchport
					 mtu 9216
					 ip address 10.1.12.1 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/3
					 description to leaf3
					 no switchport
					 mtu 9216
					 ip address 10.1.13.1 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/4
					 description to leaf4
					 no switchport
					 mtu 9216
					 ip address 10.1.14.1 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.1/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.1/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					route-map dynamic-bgp-as-list permit 10
					match as-number 65011, 65012, 65013, 65014

					route-map retain-nh permit 10
					set ip next-hop unchanged

					router bgp 65000
					router-id 192.168.15.1
					bestpath as-path multipath-relax
					!cause we have many path base on spine counts and same as-path count bydefault bgp see they are different recommended use relax mode and received as-numbers will be used

					address-family ipv4 unicast
					maximum-paths 8
					address-family l2vpn evpn
					maximum-paths 8
					retain route-target all
					!on spines we don't have route target and no vni so receiving these have filtering action, this command prevent filtering

					template peer ebgp-peers
					address-family l2vpn evpn
					rewrite-evpn-rt-asn
					!from every where our route targets get advertised so change as-numbers if don't change them could not import another place

					neighbor 192.168.15.0/24 remote-as route-map dynamic-bgp-as-list
					inherit peer ebgp-peers
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					route-map retain-nh out
					!each route learning by spines from leafs will be replaced the nexthops to advertise toward another leafs so nexthops will be spines not leafs, cause our ebgp is important for us and in evpn our nexthops must be on leafs so in this route map we say use same leafs nexthops don't replace them

					show bgp l2vpn evpn summary
					!in state if were blank means established

			spine-2
				!--------------- underlay ----------------
					hostname spine2

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.2/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.2

					interface ethernet1/1
					 description to leaf1
					 no switchport
					 mtu 9216
					 ip address 10.2.11.2 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to leaf2
					 no switchport
					 mtu 9216
					 ip address 10.2.12.2 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/3
					 description to leaf3
					 no switchport
					 mtu 9216
					 ip address 10.2.13.2 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/4
					 description to leaf4
					 no switchport
					 mtu 9216
					 ip address 10.2.14.2 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.2/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.2/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------	
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					route-map dynamic-bgp-as-list permit 10
					match as-number 65011, 65012, 65013, 65014

					route-map retain-nh permit 10
					set ip next-hop unchanged

					router bgp 65000
					router-id 192.168.15.2
					bestpath as-path multipath-relax
					!cause we have many path base on spine counts and same as-path count bydefault bgp see they are different recommended use relax mode and received as-numbers will be used

					address-family ipv4 unicast
					maximum-paths 8
					address-family l2vpn evpn
					maximum-paths 8
					retain route-target all
					!on spines we don't have route target and no vni so receiving these have filtering action, this command prevent filtering

					template peer ebgp-peers
					address-family l2vpn evpn
					rewrite-evpn-rt-asn
					!from every where our route targets get advertised so change as-numbers if don't change them could not import another place

					neighbor 192.168.15.0/24 remote-as route-map dynamic-bgp-as-list
					inherit peer ebgp-peers
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					route-map retain-nh out
					!each route learning by spines from leafs will be replaced the nexthops to advertise toward another leafs so nexthops will be spines not leafs, cause our ebgp is important for us and in evpn our nexthops must be on leafs so in this route map we say use same leafs nexthops don't replace them

					show bgp l2vpn evpn summary
					!in state if were blank means established

			leaf-1
				!--------------- underlay ----------------
					hostname leaf1

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.11/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.11

					interface ethernet1/1
					 description to spine1
					 no switchport
					 mtu 9216
					 ip address 10.1.11.11 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to spine2
					 no switchport
					 mtu 9216
					 ip address 10.2.11.11 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.11/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.11/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------	
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					router bgp 65011
					router-id 192.168.15.11
					bestpath as-path multipath-relax
					address-family l2vpn evpn
					maximum-paths 8
					neighbor 192.168.15.1
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn
					neighbor 192.168.15.2
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

				!--------------- l2vn segment ----------------	
					feature interface-vlan
					spanning-tree vlan 1,10,20,77 priority 4096
					fabric forwarding anycast-gateway-mac 0001.0001.0001

					vlan 1,10,20,77
					vlan 10
					vn-segment 30010
					!vni layer 2

					vlan 20
					vn-segment 30020
					!vni layer 2

					vlan 77
					name irb-tenant-1
					vn-segment 30077
					!vni layer 3

					vrf context tenant-1
					vni 30077
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					interface vlan 10
					no shutdown
					vrf member tenant-1
					ip address 172.16.10.1/24
					fabric forwarding mode anycast-gateway

					interface vlan 20
					no shutdown
					vrf member tenant-1
					ip address 172.16.20.1/24
					fabric forwarding mode anycast-gateway

					interface vlan77
					 description ** irb-tenant-1 **
					no shutdown
					mtu 9216
					vrf member tenant-1
					ip forward

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback30
					member vni 30010
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them
					
					member vni 30020
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them

					member vni 30077 associate-vrf

					evpn
					vni 30010 l2
					rd auto
					route-target import auto
					route-target export auto
					vni 30020 l2
					rd auto
					route-target import auto
					route-target export auto

					show nve vni
					show nve peers

					show ip route vrf tenant-1
					show bgp l2vpn evpn

					show l2route evpn mac-ip all

				!--------------- access interface ----------------	
					int e1/3
					description to server-1
					switchport mode access
					switchport access vlan 10

			leaf-2
				!--------------- underlay ----------------
					hostname leaf2

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.12/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.12

					interface ethernet1/1
					 description to spine1
					 no switchport
					 mtu 9216
					 ip address 10.1.12.12 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to spine2
					 no switchport
					 mtu 9216
					 ip address 10.2.12.12 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.12/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.12/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------	
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					router bgp 65012
					router-id 192.168.15.12
					bestpath as-path multipath-relax
					address-family l2vpn evpn
					maximum-paths 8
					neighbor 192.168.15.1
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn
					neighbor 192.168.15.2
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

				!--------------- l2vn segment ----------------	
					feature interface-vlan
					spanning-tree vlan 1,10,20,77 priority 4096
					fabric forwarding anycast-gateway-mac 0001.0001.0001

					vlan 1,10,20,77
					vlan 10
					vn-segment 30010
					!vni layer 2

					vlan 20
					vn-segment 30020
					!vni layer 2

					vlan 77
					name irb-tenant-1
					vn-segment 30077
					!vni layer 3

					vrf context tenant-1
					vni 30077
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					interface vlan 10
					no shutdown
					vrf member tenant-1
					ip address 172.16.10.1/24
					fabric forwarding mode anycast-gateway

					interface vlan 20
					no shutdown
					vrf member tenant-1
					ip address 172.16.20.1/24
					fabric forwarding mode anycast-gateway

					interface vlan77
					 description ** irb-tenant-1 **
					no shutdown
					mtu 9216
					vrf member tenant-1
					ip forward

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback30
					member vni 30010
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them
					
					member vni 30020
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them

					member vni 30077 associate-vrf

					evpn
					vni 30010 l2
					rd auto
					route-target import auto
					route-target export auto
					vni 30020 l2
					rd auto
					route-target import auto
					route-target export auto

					show nve vni
					show nve peer

					show ip route vrf tenant-1
					show bgp l2vpn evpn

					show l2route evpn mac-ip all

			leaf-3
				!--------------- underlay ----------------
					hostname leaf3

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.13/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.13

					interface ethernet1/1
					 description to spine1
					 no switchport
					 mtu 9216
					 ip address 10.1.13.13 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to spine2
					 no switchport
					 mtu 9216
					 ip address 10.2.13.13 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.13/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.13/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------	
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					router bgp 65013
					router-id 192.168.15.13
					bestpath as-path multipath-relax
					address-family l2vpn evpn
					maximum-paths 8
					neighbor 192.168.15.1
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn
					neighbor 192.168.15.2
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

				!--------------- l2vn segment ----------------	
					feature interface-vlan
					spanning-tree vlan 1,10,20,77 priority 4096
					fabric forwarding anycast-gateway-mac 0001.0001.0001

					vlan 1,10,20,77
					vlan 10
					vn-segment 30010
					!vni layer 2

					vlan 20
					vn-segment 30020
					!vni layer 2

					vlan 77
					name irb-tenant-1
					vn-segment 30077
					!vni layer 3

					vrf context tenant-1
					vni 30077
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					interface vlan 10
					no shutdown
					vrf member tenant-1
					ip address 172.16.10.1/24
					fabric forwarding mode anycast-gateway

					interface vlan 20
					no shutdown
					vrf member tenant-1
					ip address 172.16.20.1/24
					fabric forwarding mode anycast-gateway

					interface vlan77
					 description ** irb-tenant-1 **
					no shutdown
					mtu 9216
					vrf member tenant-1
					ip forward

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback30
					member vni 30010
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them
					
					member vni 30020
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them

					member vni 30077 associate-vrf

					evpn
					vni 30010 l2
					rd auto
					route-target import auto
					route-target export auto
					vni 30020 l2
					rd auto
					route-target import auto
					route-target export auto

					show nve vni
					show nve peers

					show ip route vrf tenant-1
					show bgp l2vpn evpn

					show l2route evpn mac-ip all
				
				!--------------- access interface ----------------	
					int e1/3
					description to server-2
					switchport mode access
					switchport access vlan 20

			leaf-4
				!--------------- underlay ----------------
					hostname leaf4

					interface loopback0
					 description ** ospf unnderlay **
					 ip address 192.168.0.14/32

					feature ospf

					router ospf underlay-net
					 router-id 192.168.0.14

					interface ethernet1/1
					 description to spine1
					 no switchport
					 mtu 9216
					 ip address 10.1.14.14 255.255.255.0
					 no shutdown
					 no switchport
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface ethernet1/2
					 description to spine2
					 no switchport
					 mtu 9216
					 ip address 10.2.14.14 255.255.255.0
					 no shutdown
					 ip ospf network point-to-point
					 no ip ospf passive-interface
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback15
					 description ** bgp evpn peering address **
					 ip address 192.168.15.14/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

					interface loopback30
					 description ** vtep ip address **
					 ip address 192.168.30.14/32
					 ip ospf network point-to-point
					 ip router ospf underlay-net area 0.0.0.0

				!--------------- bgp ----------------	
					feature bgp
					feature nv overlay
					feature vn-segment-vlan-based
					nv overlay evpn

					router bgp 65014
					router-id 192.168.15.14
					bestpath as-path multipath-relax
					address-family l2vpn evpn
					maximum-paths 8
					neighbor 192.168.15.1
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn
					neighbor 192.168.15.2
					remote-as 65000
					update-source loopback15
					ebgp-multihop 2
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

				!--------------- l2vn segment ----------------	
					feature interface-vlan
					spanning-tree vlan 1,10,20,77 priority 4096
					fabric forwarding anycast-gateway-mac 0001.0001.0001

					vlan 1,10,20,77
					vlan 10
					vn-segment 30010
					!vni layer 2

					vlan 20
					vn-segment 30020
					!vni layer 2

					vlan 77
					name irb-tenant-1
					vn-segment 30077
					!vni layer 3

					vrf context tenant-1
					vni 30077
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					interface vlan 10
					no shutdown
					vrf member tenant-1
					ip address 172.16.10.1/24
					fabric forwarding mode anycast-gateway

					interface vlan 20
					no shutdown
					vrf member tenant-1
					ip address 172.16.20.1/24
					fabric forwarding mode anycast-gateway

					interface vlan77
					 description ** irb-tenant-1 **
					no shutdown
					mtu 9216
					vrf member tenant-1
					ip forward

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback30
					member vni 30010
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them
					
					member vni 30020
					ingress-replication protocol bgp
					!used for bum unicast and route type 3 in bgp just send on interfaces which has them

					member vni 30077 associate-vrf

					evpn
					vni 30010 l2
					rd auto
					route-target import auto
					route-target export auto
					vni 30020 l2
					rd auto
					route-target import auto
					route-target export auto

					show nve vni					
					show nve peers

					show ip route vrf tenant-1
					show bgp l2vpn evpn

					show l2route evpn mac-ip all

	///////////////////////	

	multi-site
		normaly in datacenter or isp
		used for vmotion and sdwan
		it is important use for many sites on layer 2

		multipod
			single fabric with end-to-end encapsulation

			build hierarchy in the underlay - flatten it in the overlay

			each sites will be consider as one site make them single fabric and single overlays has vxlan tunnel like end to end 

			as65001	| ------------------ as65033 ----------------- | as65002
			------- 												 --------
			ibgp  	  /////////////////////////////////////////////  ibgp
			inside 						ebgp						 inside
										outside

			*end to end vxlan connectivities on same underlay

			nexthops directly points t vtep address
			single overlay and non hierarchy
				control plane
				data plane

			hierarchy underlay
			bum used replication domain and see all 

			single vni administrative domain 

		///////////////////////	

		multifabric
			multiple fabrics - normalized through ethernet

			multiple fabrics interconnect using dci (layer 2 and layer 3)

			many different fabrics connect 2 sites which has vxlan media with any layer 2 technologies otv or vpls ...

			fabric1 | ------------------------ otv -------------------------- | fabric2  (vxlan or ethernet native)
			l3 dci 	///////////////////////////////////////////////////////// 	l3 dci
			l2 dci  					underlay no extention 					l2 dci 

			*datacenter interconnection (dci)

			use many border leafs and set same vlans and transfer over vpls or otv ...

			separate overlay domains independent l2 and l3 dci (complexity)
			
			separate overlay control-plane domains manual configuration
			
			separate underlay domains isolated
			
			separate replication domains for bum independent bum transport/dci
			
			dedicated border leaf no local end-point attachment

			underlay isolation 
			separate dc interconnection (isolation for vlans)

			better performance from multipod
			our dci is not vxlan base

		///////////////////////	

		multisite
			multiple fabrics with integrated dci (dci2)

			integrated dci - scaling within and between fabrics

			best implementation model which has single end to end vxlan tunnel and hierarchy vxlan overlay

			multiple overlay domains interconnected & controlled
			multiple overlay control-plane domains interconnected & controlled (if need prune vlan from vxlan)
			multiple underlay domains isolated
			multiple replication domains for bum interconnected & controlled
			multiple vni administrative domains phase 2

			underlay isolation 
			overlay hierarchies

			end to end vxlan connection
			hierarchy vxlan tunnel whole path

			important part is border leafs which are hierarchy and called border gateway 
			main task is run vxlan as hierarchy mechanism
			just need line cart over spines like 
				nexus 9500 platform with x9700-ex line card
				nexus 9500 platform with x9700-fx line card
				
			on leafs 
				nexus 9300 ex platform
				nexus 9300 fx platform
				nexus 9300 fx2 platform
				nexus 9364c platform
				nexus 9332c platform

			ios needed
				software 7.0 (3) or 7 (1) or later

			we can set super spine to maange spines
			between border gateways we have layer3 connections

			control plane 
				both mp-ebgp or mp-ibgp peering supported intra-site between leaf nodes
				
				only mp-ebgp evpn sessions supported inter-sites > mandates that each site is part of a separate as
				
				full mesh of mp-ebgp evpn adjacencies only currently supported across sites
					recommended to deploy a couple of route-servers in the inter-site network when 3 or more sites are deployed
					
					route-servers only perform control plane functions ("ebgp route-reflectors")
					
					need to ensure that route-servers offer support for route type 4 evpn routes, required for df election

				if vlans were not inside vxlan will be prune (selective advertisement)
					the multi-site architecture provides granular control on how layer-2 and layer-3 communication is extended across sites
					
					layer-2 and/or layer-3 vnis configured on the border gateways (bgw) control the control-plane advertisement towards dci
					
					enhances the overall scalability of the solution
						scale up the total number of end-points supported across sites

			data plane
				end to end is vxlan base
				we have 3 isolated vxlan tunnel between each segment of network
					leaf 1 to bgw 1
					bgw 1 to bgw 2
					bgw 2to leaf 2

				border gateways will encapsulate and decapsulate all packets

				switching traffic and routing traffic

				bridging
					1* from host 1 to leaf 1
						source ip > leaf 1
						destination ip > border gateway 1
						vxlan > 30010
						----------------------------
							source mac > host 1
							destination mac > host 2
							source ip > host 1 
							destination ip > host 2
							payload

					////////////////////////////////////////////////////

					2* from border gateway 1 to border gateway 2
						source ip > border gateway 1
						destination ip > border gateway 2
						vxlan > 30010
						----------------------------
							source mac > host 1
							destination mac > host 2
							source ip > host 1 
							destination ip > host 2
							payload

					////////////////////////////////////////////////////

					3* from border gateway 2 to host 2

						*here open packet detect destination and encapsulate again

						source ip > border gateway 2
						destination ip > leaf 2
						vxlan > 30010
						----------------------------
							source mac > host 1
							destination mac > host 2
							source ip > host 1 
							destination ip > host 2
							payload

				routing
					1* from host 1 to leaf 1
						source ip > leaf 1
						destination ip > border gateway 1
						vxlan > 50001
						----------------------------
							source mac > leaf 1
							destination mac > border gateway 2
							source ip > host 1 
							destination ip > host 2
							payload

					////////////////////////////////////////////////////

					2* from border gateway 1 to border gateway 2
						source ip > border gateway 1
						destination ip > border gateway 2
						vxlan > 50001
						----------------------------
							source mac > border gateway 1
							destination mac > border gateway 2
							source ip > host 1 
							destination ip > host 2
							payload

					////////////////////////////////////////////////////

					3* from border gateway 2 to host 2

						*here open packet detect destination and encapsulate again

						source ip > border gateway 2
						destination ip > leaf 2
						vxlan > 50001
						----------------------------
							source mac > border gateway 2
							destination mac > host 2
							source ip > host 1 
							destination ip > host 2
							payload

				bum and multicast 
					1* from host 1 to leaf 1
						source ip > leaf 1
						destination ip > destination group
						vxlan > 30010
						----------------------------
							source mac > host 1
							destination mac > all
							source ip > host 1 
							destination ip > all
							payload

					////////////////////////////////////////////////////

					2* all routers here received traffic and wanna forward the traffic so we have mechanism here

						*designated forwarder is one of the edge routers which forward vnis

						*traffic will be forwarded on all interfaces except ingress port (cuase split-horizon rule)

						source ip > border gateway 1
						destination ip > border gateway 2 , border gateway 3 , border gateway 4
						vxlan > 30010

						*all routers received the forwarded packets, just forwarder router (designated forwarder) could forward traffic
						----------------------------
							source mac > host 1
							destination mac > all
							source ip > host 1 
							destination ip > all
							payload

					////////////////////////////////////////////////////

					3* from border gateway 3 to host 2

						*here open packet detect destination and encapsulate again

						source ip > border gateway 3 (designated forwarder)
						destination ip > destination group
						vxlan > 30010
						----------------------------
							source mac > host 1
							destination mac > all
							source ip > host 1 
							destination ip > all
							payload

							*again here designated forwarder after encapsulation forward packets on all interfaces except ingress port (split-horizon rule)
							*our vtep  or leaf behave again like designated forwarder

					*usually use pim between border	gateways and leafs and between edge routers or border gateway on internet runs ingress replication
					casue on wan side if set super spine we can run pim if wan side we have router
					pim has group forward and multicasting so one router must manage these as designated forwarder on wan side we could not manage something over
					base on split-horizon rule has no repeated message on same links (ingress and pim interfaces)

					*ingress replication means we don't have dedicated wan access and generate each packet many times
					*group sending means our wan media is dedicated access and run multicast pim 

			*Various options do exist but the recommended design choices are
				fabric internal
					igp underlay
					ibgp overlay

				dci (primary choice) (between 2 site)
					ebgp underlay
					ebgp overlay

					route server for dci overlay peerings if were too large fabric

					dc core for reachability across n sites
	
				dci (alternative option)
					any routing protocol underlay
					must use ebgp for overlay

					full-mesh for dci overlay peerings
					back-to-back site reachability (physical, full-mesh)

			import command active border gateway
				!--------------- border gateway activation -------------------
					feature nv overlay
					nv overlay evpn
	
					feature bgp
					feature interface-vlan
					feature vn-segment-vlan-based
	
					evpn multisite border-gateway

				!--------------- loop & vtep -------------------
					interface loopback0
					description *rid*
					ip address 10.10.10.101/32 tag 12345
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					!tag will be used for redistribution and detect loopbacks
	
					interface loopback1
					description *pip vtep*
					ip address 10.1.1.101/32 tag 12345
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					!tag will be used for redistribution and detect loopbacks
	
					interface loopback100
					description *vip multi-site 1*
					ip address 10.1.1.111/32 tag 12345
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					!tag will be used for redistribution and detect loopbacks

				!--------------- fabric link tracking -------------------
					interface ethernet1/53
					description to-spine1
					ip address 10.0.1.1/30
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					evpn multisite fabric-tracking

					interface ethernet1/54
					description to-spine2
					ip address 10.0.2.1/30
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					evpn multisite fabric-tracking
					!on spine side if our bgw get trouble, spines omit vip and don't received traffics

				!--------------- multisite underlay interface -------------------
					interface ethernet1/1
					description to-dc-core1
					ip address 10.111.111.1/30 tag 12345
					evpn multisite dci-tracking

					interface ethernet1/2
					description to-dc-core2
					ip address 10.111.222.1/30 tag 12345
					evpn multisite dci-tracking
					!same behavior as fabric link traking used for dci links

				!--------------- multisite overlay peering (to route server) -------------------
					router bgp 65501
					router-id 10.10.10.101
					address-family ipv4 unicast
					redistribute direct route-map redist-local
					neighbor 10.111.111.2
					remote-as 65599
					update-source ethernet1/1
					address-family ipv4 unicast
					neighbor 10.111.222.2
					remote-as 65599
					update-source ethernet1/2
					address-family ipv4 unicast
					neighbor 10.99.99.201
					remote-as 65599
					update-source loopback0
					ebgp-multihop 5
					peer-type fabric-external
					!make hierarchy and set nexthops on vteps (make change on nexthops)

					address-family 12vpn evpn
					rewrite-evpn-rt-asn
					!cause route target is automatic , each site route targets will be same as as-number

					send-community
					send-community both

				!--------------- anycast bgw vtep config -------------------
					interface nvel
					no shutdown
					host-reachability protocol bgp
					multisite ethernet-segment 7
					!domain isolation, can bypass it no effect

					system-mac 0000.0000.0001
					!multisite site id

					source-interface loopback1
					multisite border-gateway interface loopback100
					!define loop 100 as bgw

					member vni 30010
					multisite ingress-replication
					!between 2 site use ingress replication

					mcast-group 239.1.1.1
					!inside the site use multicast

					member vni 30011-30020
					mcast-group 239.1.1.2
					member vni 50001 associate-vrf

		///////////////////////	

		last scenario
			loop 100 used for bgw vip
			loop 250 used for multicast rp

			underlay and overlay be on ebgp between sites

			inside the sites we have ospf as underlay and ibg as overlay

			inside the sites we have bum pim
			between sites we have ingress replication

			spines has no vni just behave as route reflector

			configs
				dci-router
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					no switchport
					mtu 9216
					ip address 10.10.1.5/30 tag 54321
					no shutdown

					interface ethernet1/2
					no switchport
					mtu 9216
					ip address 10.10.1.1/30 tag 54321
					no shutdown

					interface loopback0
					ip address 100.100.100.100/32

					router bgp 65003
					address-family ipv4 unicast
					network 100.100.100.100/32
					maximum-paths 64
					maximum-paths ibgp 64
					neighbor 10.10.1.2
					remote-as 65002
					update-source ethernet1/2
					address-family ipv4 unicast
					next-hop-self
					neighbor 10.10.1.6
					remote-as 65001
					update-source ethernet1/1
					address-family ipv4 unicast
					next-hop-self

					route-map unchanged permit 10
					set ip next-hop unchanged

					 
					router bgp 65003
					address-family l2vpn evpn
					retain route-target all
					template peer overlay-peering
					update-source loopback0
					ebgp-multihop 5
					address-family l2vpn evpn
					send-community
					send-community extended
					route-map unchanged out
					neighbor 10.2.0.1
					inherit peer overlay-peering
					remote-as 65001
					address-family l2vpn evpn
					rewrite-evpn-rt-asn
					neighbor 20.2.0.1
					inherit peer overlay-peering
					remote-as 65002
					address-family l2vpn evpn
					rewrite-evpn-rt-asn

				----------------------

				bgw-1
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					no switchport
					mtu 9216
					ip address 10.10.1.6/30 tag 54321
					evpn multisite dci-tracking
					!force in this scenario

					no shutdown

					interface ethernet1/2
					description connected-to-spine-1-ethernet1/2
					no switchport
					mtu 9216
					ip address 10.4.0.6/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					evpn multisite fabric-tracking
					no shutdown

					interface loopback0
					description routing loopback interface
					ip address 10.2.0.1/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback1
					description vtep loopback interface
					ip address 10.3.0.2/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback100
					ip address 10.10.0.2/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 10.2.0.1

					ip pim rp-address 10.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8

					route-map rmap-redist-direct permit 10
					match tag 54321

					router bgp 65001
					router-id 10.2.0.1
					address-family ipv4 unicast
					redistribute direct route-map rmap-redist-direct
					maximum-paths 64
					maximum-paths ibgp 64
					neighbor 10.10.1.5
					remote-as 65003
					update-source ethernet1/1
					address-family ipv4 unicast
					next-hop-self

					vlan 1,250,2000

					vlan 2000
					vn-segment 50000
					interface vlan2000
					vrf member myvrf_50000
					ip forward
					mtu 9216
					no shutdown

					vrf context myvrf_50000
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					vlan 250
					vn-segment 30001

					evpn
					vni 30001 l2
					rd auto
					route-target import auto
					route-target export auto

					fabric forwarding anycast-gateway-mac 2020.0000.00aa

					router bgp 65001
					neighbor 10.2.0.3
					remote-as 65001
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					neighbor 100.100.100.100
					remote-as 65003
					update-source loopback0
					ebgp-multihop 5
					peer-type fabric-external
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

					evpn multisite border-gateway 65001
					delay-restore time 300

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback1
					multisite border-gateway interface loopback100
					member vni 50000 associate-vrf
					member vni 30001
					multisite ingress-replication
					mcast-group 239.1.1.0

				///////////////////////	

				spine-1
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature nv overlay

					interface ethernet1/1
					description connected-to-leaf-1-ethernet1/1
					no switchport
					mtu 9216
					ip address 10.4.0.2/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/2
					description connected-to-bgw-1-ethernet1/2
					no switchport
					mtu 9216
					ip address 10.4.0.5/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface loopback0
					description routing loopback interface
					ip address 10.2.0.3/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback254
					description anycast-rp loopback interface
					ip address 10.254.254.1/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 10.2.0.3

					ip pim rp-address 10.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8
					ip pim anycast-rp 10.254.254.1 10.2.0.3

					router bgp 65001
					router-id 10.2.0.3
					neighbor 10.2.0.1
					remote-as 65001
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					route-reflector-client
					neighbor 10.2.0.2
					remote-as 65001
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					route-reflector-client
				
				///////////////////////	

				leaf-1
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					description connected-to-spine-1-ethernet1/1
					no switchport
					mtu 9216
					ip address 10.4.0.1/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/2
					switchport access vlan 250
					spanning-tree port type edge
					mtu 9216

					interface loopback0
					description routing loopback interface
					ip address 10.2.0.2/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback1
					description vtep loopback interface
					ip address 10.3.0.1/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 10.2.0.2

					ip pim rp-address 10.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8
					vlan 1,250,2000

					vlan 2000
					vn-segment 50000

					interface vlan2000
					vrf member myvrf_50000
					ip forward
					mtu 9216
					no shutdown

					vrf context myvrf_50000
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					vlan 250
					vn-segment 30001

					interface vlan250
					vrf member myvrf_50000
					ip address 10.10.10.1/24 tag 12345
					fabric forwarding mode anycast-gateway
					no shutdown

					evpn
					vni 30001 l2
					rd auto
					route-target import auto
					route-target export auto

					fabric forwarding anycast-gateway-mac 2020.0000.00aa

					router bgp 65001
					router-id 10.2.0.2
					neighbor 10.2.0.3
					remote-as 65001
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback1
					member vni 50000 associate-vrf
					member vni 30001
					mcast-group 239.1.1.0

					show ip route vrf myvrf_50000
					show bgp l2vpn evpn
					show l2route evpn mac-ip all

				----------------------

				bgw-2
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					description connected-to-spine-2-ethernet1/1
					no switchport
					mtu 9216
					ip address 20.4.0.2/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					evpn multisite fabric-tracking
					!force in this scenario

					no shutdown

					interface ethernet1/2
					no switchport
					mtu 9216
					ip address 10.10.1.2/30 tag 54321
					evpn multisite dci-tracking
					no shutdown

					interface loopback0
					description routing loopback interface
					ip address 20.2.0.1/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback1
					description vtep loopback interface
					ip address 20.3.0.2/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback100
					ip address 10.10.0.1/32 tag 54321
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 20.2.0.1

					ip pim rp-address 20.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8

					route-map rmap-redist-direct permit 10
					match tag 54321

					router bgp 65002
					router-id 20.2.0.1
					address-family ipv4 unicast
					redistribute direct route-map rmap-redist-direct
					maximum-paths 64
					maximum-paths ibgp 64
					neighbor 10.10.1.1
					remote-as 65003
					update-source ethernet1/2
					address-family ipv4 unicast
					next-hop-self

					vlan 1,250,2000

					vlan 2000
					vn-segment 50000

					interface vlan2000
					vrf member myvrf_50000
					ip forward
					mtu 9216
					no shutdown

					vrf context myvrf_50000
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					vlan 250
					vn-segment 30001

					evpn
					vni 30001 l2
					rd auto
					route-target import auto
					route-target export auto

					evpn multisite border-gateway 65002
					delay-restore time 300

					fabric forwarding anycast-gateway-mac 2020.0000.00aa

					router bgp 65002
					neighbor 20.2.0.4
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					neighbor 100.100.100.100
					remote-as 65003
					update-source loopback0
					ebgp-multihop 5
					peer-type fabric-external
					address-family l2vpn evpn
					send-community
					send-community extended
					rewrite-evpn-rt-asn

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback1
					multisite border-gateway interface loopback100
					member vni 50000 associate-vrf
					member vni 30001
					multisite ingress-replication
					mcast-group 239.1.1.5
				
				///////////////////////	
				
				spine-2
					feature nxapi
					feature tacacs+
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature lldp
					feature nv overlay

					ip pim rp-address 20.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8
					ip pim anycast-rp 20.254.254.1 20.2.0.4

					interface ethernet1/1
					description connected-to-bgw-2-ethernet1/1
					no switchport
					mtu 9216
					ip address 20.4.0.1/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/2
					description connected-to-leaf-2-ethernet1/1
					no switchport
					mtu 9216
					ip address 20.4.0.5/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/3
					description connected-to-leaf-3-ethernet1/1
					no switchport
					mtu 9216
					ip address 20.4.0.9/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface loopback0
					description routing loopback interface
					ip address 20.2.0.4/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback254
					description anycast-rp loopback interface
					ip address 20.254.254.1/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 20.2.0.4

					router bgp 65002
					router-id 20.2.0.4
					neighbor 20.2.0.1
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					route-reflector-client
					neighbor 20.2.0.2
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					route-reflector-client
					neighbor 20.2.0.3
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended
					route-reflector-client
				
				///////////////////////	
				
				leaf-2
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					description connected-to-spine-2-ethernet1/2
					no switchport
					mtu 9216
					ip address 20.4.0.6/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/2
					switchport access vlan 250
					spanning-tree port type edge
					spanning-tree bpduguard enable
					mtu 9216

					interface loopback0
					description routing loopback interface
					ip address 20.2.0.2/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					interface loopback1
					description vtep loopback interface
					ip address 20.3.0.1/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 20.2.0.2

					ip pim rp-address 20.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8

					vlan 1,250,450,800,2000

					vlan 2000
					vn-segment 50000

					interface vlan2000
					vrf member myvrf_50000
					ip forward
					mtu 9216
					no shutdown

					vrf context myvrf_50000
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn
					 
					vlan 250
					vn-segment 30001

					interface vlan250
					vrf member myvrf_50000
					ip address 10.10.10.1/24 tag 12345
					fabric forwarding mode anycast-gateway
					no shutdown

					evpn
					vni 30001 l2
					rd auto
					route-target import auto
					route-target export auto

					fabric forwarding anycast-gateway-mac 2020.0000.00aa

					router bgp 65002
					router-id 20.2.0.2
					neighbor 20.2.0.4
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback1
					member vni 50000 associate-vrf
					member vni 30001
					mcast-group 239.1.1.5
				
				///////////////////////	
			
				leaf-3
					nv overlay evpn
					feature ospf
					feature bgp
					feature pim
					feature interface-vlan
					feature vn-segment-vlan-based
					feature nv overlay

					interface ethernet1/1
					description connected-to-spine-2-ethernet1/3
					no switchport
					mtu 9216
					ip address 20.4.0.10/30
					ip ospf network point-to-point
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					no shutdown

					interface ethernet1/2
					switchport access vlan 300
					spanning-tree port type edge
					spanning-tree bpduguard enable
					mtu 9216

					interface loopback0
					description routing loopback interface
					ip address 20.2.0.3/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode
					 
					interface loopback1
					description vtep loopback interface
					ip address 20.3.0.3/32
					ip router ospf underlay area 0.0.0.0
					ip pim sparse-mode

					router ospf underlay
					router-id 20.2.0.3

					ip pim rp-address 20.254.254.1 group-list 239.1.1.0/25
					ip pim ssm range 232.0.0.0/8

					vlan 1,300,450,800,2000

					vlan 2000
					vn-segment 50000

					interface vlan2000
					vrf member myvrf_50000
					ip forward
					mtu 9216
					no shutdown

					vrf context myvrf_50000
					vni 50000
					rd auto
					address-family ipv4 unicast
					route-target both auto
					route-target both auto evpn

					vlan 300
					vn-segment 30000

					interface vlan300
					vrf member myvrf_50000
					ip address 10.10.11.1/24 tag 12345
					fabric forwarding mode anycast-gateway
					no shutdown

					evpn
					vni 30000 l2
					rd auto
					route-target import auto
					route-target export auto

					fabric forwarding anycast-gateway-mac 2020.0000.00aa

					router bgp 65002
					router-id 20.2.0.3
					neighbor 20.2.0.4
					remote-as 65002
					update-source loopback0
					address-family l2vpn evpn
					send-community
					send-community extended

					interface nve1
					no shutdown
					host-reachability protocol bgp
					source-interface loopback1
					member vni 50000 associate-vrf
					member vni 30000
					mcast-group 239.1.1.0
***************************
firewall injection
	recommended use service leaf to control all traffics over them

	north-south (inside and outside)
		must use anycast gw
		vrf on leaf has default route to firewall 
		leaf has routing mechanism and make tunnel on l3vni to firewall leaf
		after firewall works on l3 vni

	east-west (server)
		doesn't need anycast gw cause gateway is on same vlan interface not on firewall
		routing is on firewall no leaf switch
		per vlan inside the leafs must define same vlan on firwall 
