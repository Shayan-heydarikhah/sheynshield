ENSLD
**************************
general speaking
	protocol 									protocol number
	--------------------------------------------------------------
	internet control message protocol 				1
	internet group management protocol 				2
	transmission controll protocol 					6
	user datagram protocol 							17
	ipv6 encapsulation 								41
	encapsulation security payload 					50
	authentication header 							51
	icmp v6 										58
	eigrp 											88
	ospf 											89
	protocol independent multicast 					103
	vrrp 											112

	layer3 packet header
		version
			ipv4 or ipv6

		internal header length (ihl)(8bit)	
			if doesn't use options		

		tos (8bit)
			
		total length (16bit)
			mtu 1500 byte
				if were bigger than what set on router must fragment 
				routers first check this fields

			maximum size of packet which calculate data and header value together

		identity or identification (16bit)
			for each fragment set one id and prevent confusion

		flags (3bit)
			0----- (reserved)
			-1---- (don't frg)(last pacekt is fragment or not)
			--0--- (more frg)(our bits will be continue)

		fargment offset (13bit)
			which part of that fragment

		ttl (8bit)

		protocol (8bit)
			data will be froward to which protocol

		checksum (16bit)
			just for headers
			after receiving by router at the first step check this then decrease ttl after them regenerate another packet with different checksum value

		source ip address (32bit)

		destination ip address (32bit)

		ip option field (32bit)
			usefull on specific and security targets
			change ihl
			routing loos
			if doesn't contain 4 byte must fill as padding spaces

		padding

	best effort network is a flat and no diffserv or ipp traffic tags
		has no qos

		tos (8bit)
			--- ---- -
			ipp
				ipp has 3 bit and 2pow 3 means 8 alternates which has forwarding priority no drop priority so it's limited

				000 > routine
				001 > priority
				010 > immediate
				011 > flash
				100 > flash override
				101 > critic\ecp
				110 > internetwork control
				111 > network control

			--- ---- -
				tos
					tos has 4 bit and 5 alternates
					-	-	-	-
					3 	2 	1 	0

					3 > delay
						0 is normal
						1 is low

					2 > throughput
						0 is normal
						1 is high

					1 > reliability
						0 is normal
						1 is high

					0 > cost
						0 is normal
						1 is low

					1000 > minimum delay
					0100 > maximum throughput
					0010 > maximum reliability
					0001 > minimum montray cost
					0000 > normal service

			dscp (diffrentiated service code point)
				tos byte convert to ds field

				use 6 bit as alternates and 2 bit as ecn
				ecn has special target like explicit congest notification as congest avoidance

				6bit as alternates means 2 pow 6 is 64 alternate model

			data>000000 00 (0)|r1 works on dscp|--------000000 00 (0)--------|r2 works on ipp|000 0000 0 (0)>data
			voice>000000 00 (0)|r1 works on dscp|-------000001 00 (1)---------|r2 works on ipp|000 00100 (0)>voice

			on ipp our voice traffic has no more priority so use code 0 as qos
			we use converter and use value 8 for voice and 0 for data cause in dscp to ipp use code 1 as voice and 0 as data

			ipp (8bit) 					dscp (8bit) 			class selector (cs)
			000 0000 0  	0 			000000 00 		0 		cs 0 is default and best effort fo rnormal traffic
			001 0000 0 		1 			001000 00 		8 		cs 1
			010 0000 0 		2 			010000 00 		16 		cs 2
			011 0000 0 		3 			011000 00 		24 		cs 3
			100 0000 0 		4 			100000 00 		32 		cs 4
			101 0000 0 		5 			101000 00 		40 		cs 5
			110 0000 0 		6 			110000 00 		48 		cs 6
			111 0000 0 		7 			111000 00 		56 		cs 7

			*perhop behavior

			assued forwarding 
				dscp use 2 queue on router which one of them is high priority on forwarding another used as dropping

				af or xy

				dsccp 6 bit 
					------ --
					af 		ecn

					*ecn used congest avoidance
					*af divided to 2 part as --- 3 first bits use as forwarder and --- 3 second bits use as drop

					--- 	--0		--
					fwd 	drp 	ecn

					6th bit or last bit in drop path is fixed and default

					fwd contain 0 - 7 and higher means forwarding as high priority

					drp contain 0 - 3 and higher means drop as high priority

					001 > 1 	01 >1
					010 > 2 	00 > 0

					001 > 1 	01 	0 > 1 0 > 2 >>>>> 12 

					fwd 	drp 		dscp 	
					000 	00 	0 		cs0 > 0
					000 	01 	0 		2 		
					000 	10 	0 		4
					000 	11 	0 		6
					-------------------------------
					001 	00 	0 		cs1 > 8
					001 	01 	0 		af11 >  10	
					001 	10 	0 		af12 >  12
					001 	11 	0 		af13 > 	14
					-------------------------------
					010 	00 	0 		cs2 > 16
					010 	01 	0 		af21 > 18	
					010 	10 	0 		af22 > 20
					010 	11 	0 		af23 > 22
					-------------------------------
					011 	00 	0 		cs3 > 24
					011 	01 	0 		af31 > 26	
					011 	10 	0 		af32 > 28
					011 	11 	0 		af33 > 30
					-------------------------------
					100 	00 	0 		cs4 > 32
					100 	01 	0 		af41 > 34	
					100 	10 	0 		af42 > 36
					100 	11 	0 		af43 > 38
					-------------------------------
					101 	00 	0 		cs5 > 40
					101 	11 	0 		ef > 46 
					-------------------------------
					110 	00 	0 		cs6 >  48
					-------------------------------
					111 	00 	0 		cs7 > 56

					which one will be forward and drop faster
						0 , 8 , 2 , 12

						fwd > 12,8,2,0

						fwd 	drp
						001 	10 	0 		af12 >  12
						001 	00 	0 		cs1 > 8
						000 	01 	0 		2 		
						000 	00 	0 		cs0 > 0

						drp > 12,2,0 and 8

						fwd 	drp
						001 	10 	0 		af12 >  12
						000 	01 	0 		2 		
						001 	00 	0 		cs1 > 8
						000 	00 	0 		cs0 > 0

					ef stands for expedited forward use for voice traffic 

	ipv4 use ddn > doted decimal notation

	ipv6
		leading zero is compress method which omit 0 from left side, offer omit 0 from bigger part
		in ipv6 use multicast instead of broadcast

		unicast is one to one
			2000:/3 is global unicast 
			fe80::/10 is link local 
				extended unique id
					eui

				on hdlc , framrelay or serial interfaces enabling ipv6 contaian using lowe number on mac and interface

				ping fe80:c802:1fff:feec:8%fast0/0
				!set egress interface

				if packet contain link local ip address on source or destination couldn't route them

			::1/128 is loop back
			::/128 is unspecific address
				0:0:0:0:0:0:0:0/128
				used on dhcp

			fc00:/7 is unique local
			::/80 used as embeded ipv4

			ping ::1
				used for ipvd active etection

			global unique (gia) > public ip

			unique local > unique on lan (ala) > private ip

		anycast is one to nearest 
			combination of unicast and multicast
			many systems can use same ip address
			just one system can forward traffic not many forwarder

			used on fault telorance and load sharing

			on commands must set anycast term as paramters otherwise get trouble
			also share same data 

			in real world set same ip address on many different servers on different locations every body try connection on these as nearest source if couldn't use them forward to next source

		multicast is one to many
			ff00::/8

			our ipv6 address must starts with ff and ....

			ff00::/8

			f 		f 		0 		0
			1111 	1111 	0000 	0000
							   -
							   this bit if were 0 means permanent, 1 is temporary
							   also means flags
							   	known as 0rpt
							   		0 knwon as reservred
							   		r
							   			rendezvous point
							   			used on multicast routing
							   				0 is not embeded
							   				1 is embeded

							   		p
							   			prefix flag
							   				0 is without prefix flag
							   				1 is prefix flag

							   				at the most time doesn't have this

							   		t	
							   			transient flag
							   				0 is permanent or wellknown multicast address
							   				1 is temporary or dynamically assigned multicast address

			f 		f 		0 		0
			1111 	1111 	0000 	0000
									----
									2 is interanetwork  (local subnets)
									5 is inter network (global site)
									1 is in node
									8 is organization (mpls)
									e is internet
									4 is admin

			ff02::1 is clients broadcasting
			ff02::2 is routers broadcasting

			ff02::5 and ff02::6 is ospfv3
			ff02::9 is ripng
			ff02::a is eigrp
			ff02::b is mobile agent
			ff02::c , ff05::13 , ff02::12 is dhcp server and relay agent
			ff0x::fb is multicast dns or mdnsv6
			ff02::d is all pim routers

		convert ipv4 to ipv6
			mapped
				0:0:0:0:0:ffff:192.168.1.1

				conver 192.168.1.1 as hexadecimal
					c0a8:0101

				::ffff:c0a8:0101

			compatible
				0:0:0:0:0:0:192.168.1.1

				converted to ::c0a8:0101

		iana site
			ir 3bit (national internet registeries)
			rir 20 bit
				arin
				afrininc
				apinic
				lacnic
				ripncc

			lir 9 bit (local internet registeries)

			customer 48 bit
			subnetid 64 bit

			sla is site level aggregator

			tia is top level aggregator

			nli is next level identifier

		001 > tia id > res > nli id > sla id > interface id
		----------------------------  ------ 	-----------
		48 bit 						  16 bit
		as public topology 	          site 
		known as global 			  topology
		routing prefix 				  subnet id
		--------------------------------------- --------------
 					network 						host

 		header
 			version (4bit)
 			traffic class (tos)(8bit)
 			flow label (20bit)
 			payload length (16bit)(just data or payload length not whole packet or header length)
 			next header (4bit)(protocol)
 			hop limit (8bit)(ttl)
 			source address (128 bit)
 			destination address (128bit)
 			option (60 bit)
 			padding

 		icmp types	
	 		icmpv6 used protocol number 58
	 		icmp type 135 is neighbor solicitation
	 		icmp type 136 is neighbor advertise
	
	 		informational message 128 - 255
	 			echo request 128
	 			echo reply 129
	 			redirect 137
	 			na 136
	 			ns 135
	 			ra 134
	 			rs 133
	
	 		error message 0- 127
	 			destination unreachable > 1
	 			packet too big > 2
	 			time exceed > 3
	 			parameter problem > 4

 			group memebership (multicast)
 				query 130
 				report 131
 				reduction 132

 		ipv6 header with no option convert protocol
 			above tcp (protocol 6) and udp (protocol 17) called extentions

 			in ipv6 we have no fragmnet

 			extentions
 				routing	header > fragment
 				hop by hop headaer is jumbo gram

 		checksum will be check on layer2

 		dhcp
 			static
 				interface gig0/0
 				ipv6 enable
 				!enable without linklocal address for clients on fe02::1 and fe02::2 used for routers

 				ipv6 address 2001:db8:1:1:1/64
 				!set link local address, set another ip for router

 				ipv6 address fe80::1 link-local
 				!change linklocal ip range and after this use these range

 			static eui64
 				interface gig0/0
 				ipv6 address 2001:db8:1:1:::/64 eui-64

 			stetless autoconfig or slaac
 				something like apipa which automaticaly generated and use on negotiations

 				interface gig0/0
 				ipv6 address 2001:db8:1:1::/64

 				*each 200 seconds send route advertise
 				source is linklocal and destination is fe02::1

 				means ipv6 activation will be used
 				enable ff01 and ff02 at the same time

 				after advertisement ra  packets by router, routers send prefix to network adn clients use eui64 with combination on prefixes to made ipv6 global unique

 				clients consider the ip address of prefixes as gateway

 				clients after turning on send ff02::2 asa route solicitated (rs) toward router to received ra packets

 				if had mac address and prefix together doesn't need check eui64
 				microsoft has random generator which gnerate ipv6

 				neighbor discovery protocol (ndp) on icmpv6 contain ra and rs packets

 				in ipv6 need another services like dns ....
 				called stateless dhcp

 			slaac dhcp
 				like slaac but received services from dhcp server
 				first pacekts are route advertise and route solicitation packets 
 				create ipv6
 				then sent dhcp solicit which advertise dhcp options like dns ....(set at first steps)

 			stateful dhcpv6
 				behave like ipv4 dhcp server 
 				in ra packets we have m flag or managed config flag which is 0 by default and works on slaac but active on 1 mens use fully dhcp negotiations and ip addressing

 				interface gig0/0
 				ipv6 nd managed-config-flag

 				*received gateway address from ra pacekts

 				o flag or other configs
 					0 means doesn't receive other configs from dhccp means use slaac
 					1 is receive other configs from dhcp like slaac and dhcp

 		ndp is like arp but in ipv6 we have no normal arp packets
 			icmpv6 contain ra and rs
 				arp request behavior is neighbor solicitated (ns)
 				arp reply behavior is neighbor advertise (ra)

 			in ndp we have no broadcast mechanism and find destination mac address
 			we have discovery mac method

 			if set linklocal or global ip address at the same time set multicast ip address named solicited node

 			2001:db8:1:1:1234:1234:abcd:1234(unicast)

 			and 

 			multicast prefix > ff02::1:ff /104
 				ff02::1:ffcd:1234 + 3333:ffcd:1234 (mac address)
 						---------
 						32bit 			

 			pc1 > 2001:fb8:1:1:1234:5678:abcd:1235/64
 			pc2 > 2001:fb8:1:1:1234:5678:abcd:1234/64

 			ping eachother, and create neighbor table or neighbor cache

 			ndp
 				need mac address
 					use neighbor solicitated
 						source linklocal pc1 > destinatio ip multicast ff02ffcd:1234

 						source mac pc1 > solicitated multicast mac 3333:ffcd:1234

 						on linklocal advertise we send global unique address and mac and prevent sending ns packets from pc2
 						just request that mac address and global unique address from pc2 to pc1

 						with na pacekts determine and advertise mac then set on neighbor table
 						global unique address on solicitated part is same on ip 

 						show ipv6 neighbor

 				process
 					prefix
 					parameters
 					routes
 					address resolution
 					nexthop determine
 						compare with prefix on nd packet which advertised
 						same value means our destination must be like requested in packets
 						if were different must set gateway and nexthop on destination then forward it
 						we have destination cache which our comparission results are there

 					neighbor unreachability detection (nud)
 						check neighbor aliveness
 						send neighbor solicitation pacekt to neighbor

 					duplicate address detection
 						each host receive ipv6 address must check with garp gratuitous arp
 						this garp combination of dad procedure
 						send ns or do ping on ipv6 see responses if received means change the ipv if not means select them

 						source ::
 						destination targeted ip address

 					redirect function
 						if see inapprotiate path as gateway must hcange on icmpv6 redirect method and mechanism
 						check source ip address and detect this source ip nexthop address if has another reachabilties has no problem but unreachability means chnage gateway

 		path mtu discovery (pmtd)
 			if had mistakes on one hop like router and set mtu lower than other fabric cause make bottel neck
 			fragmentation and reasssemble need too much resources
 			pmtd help us automatic scan fabric and detect lowest mtu 
 			set all fabric on lowest mtu
 			we cann't change mtu on intermediate devices (as fragment)
 			on source of packet must do every thing like fragment ....

 			minimum mtu on routers 1280 bytes

 			on fabric has permission to forward fragment extention as ipv6 options and just on source of packets do this
 			pmtd is automated task and role

 			show ipv6 mtu

 		name resolution
 			security features in ipv6 is higher than ipv4

 			ipv6 + header extentions (esp (des-cbc) and ah (md5))

 			is-is has type length value (tlv) which has ipv6 route posibility on osi model
 			clnp > connection less network protocol

 			in tcpip our ipv4 and clnp get routable then integerated is-is invents

 			better use it on scop as /64 ipv6 block address
 			has aggregate and summary on it
 			useful on iot (internet of things) and future features
 			offer received ip ranges from rir instead of lir or isps cause need re-addressing on isp ranges

 			must be carefull on summary and ip addressing

 			better don't use nat or pat in ipv6 cause make trouble
 			gateway is recommended

 		ipv4 to ipv6 migration
 			dual stack
 				type 
 					ipv4 > 0x800
 					ipv6 > 0x86dd

 			tunneling
 				6to4
 					automatic tunnel

 					a tunnel without destination
 					loop back ip address must be on reserved ip

 					2002::/16 is reserved for 6to4

 					192.168.1.1 
 						192 > 1100 0000 > c 0
 						168 > 1010 1000 > a 8
 						1 >   0000 0001 > 0 1
 						1 >   0000 0001 > 0 1

 						2002:c0a8:0101::/48
 						use this range as vpn range

 					benefits of ip addressing is static route 2002:c0a8:0202::/48 (has no destination but mode is 6to4) 
 					to 2002::/16 tunnel 0
 					on tunnel 0 set use what source ipv4 and connect where
 					on header layer 2 and 3 doesn't check and detect destination ip just us 2002:ca08:0202::/48 and detect destination

 					r5
 						ipv6 unicast routing
 						interface gig0/0
 						ipv6 address 2002:ca08:0101::5:64
 						no shutdown

 						*set route
 							ipv6 route ::/0 2001:db8:1:5::5/64

 					r1
						ipv6 unicast routing
 						interface gig0/0
 						ipv6 address 2002:ca08:0101::1:64
 						no shutdown

 						interface tunnel 0
 						tunnel mode ipv6 ip 6to4
 						ipv6 address 2001:db8:123::1/64
 						tunnel source loop 0

 						ipv6 route 2002::/16 tunnel 0

 						*we set range as manual on 6to4 might config better?
 							2001:db8:2:6::/64 > 2002:c0a8:0202::/64
 							
 							2001:db8:3:7::/64 > 2002:c0a8:0303::/64

 						or set route as
 							ipv6 route 2001:db8:2:6::/64 2002:c0a8:02202::
 							!means binded

 							ipv6 route 2002::/16 tunnel 0

 				isatap
 					intrasize automatic tunneling address protocol

 					has no destination

 					on dual stack routers make 
 						interface tunnel 0
 						tunnel source loop 0
 						tunnel mode ipv6 ip isatap
 						ipv6 address 2001:db8:123::1/64

 						ipv6 route 2001:db8:8:6::/64 2001:db8:123:0:0000:5efe:ca08:0202
 						ipv6 route 2001:db8:8:7::/64 2001:db8:123:0:0000:5efe:ca08:0303
 																	--------- ---------
 																	*1 		  *2

 						*1 > not necessary
 							when isatap used we can set linklocal ip like this
 							fe80::5efe:ca08:0101

 						*2 > converted ipv4

 				6rd (rapid)
 					newer than 6to4
 					more complex than isatap
 					doesn't contain 32 bit inside it
 					some informations used on border relay router
 					if connect directly use ipv6  and discover mechanism but on ipv4 has discovery

 					border relay interface virtual loopback and endpoint

 				manual configured tunnel (mct) to point-to-point tunnel or gre
 				mct just used for ipv6 over ipv4 

 				*ondemand mode, if had traffic see tunnel is up if were not see down unlike point-to-point tunnel interfaces that was same as dedicated interfaces

 					r1
 						ipv6 unicast routing
 						interface tunnel 0
 						ipv6 address 2001:db8:2:4::/64 
 						tunnel source gig0/0
 						tunnel destination 10.3.4.4
 						tunnel mode ipv6 ip

 						ipv6 route 2001:db8:4:5::/64 2001:db8:1:2::2

 						show ipv6 interface tunnel 0
 						show interface tunnel 0

 			nat  
 				nat64
 					not so useful

 					ipv6 nat prefix 2001:db8:feed::/96
 					ipv6 nat v4v6 source 10.10.10.10 2001:fb8:feed::12
 					!set edge v4, cleint ipv4

 					ipv6 nat v6v4 source 2001:db8:cafe:ffff:2 10.10.10.5
 					!client iipv6 and edge v6

 					operation
 						satetfull
 							ipv6 only translate to ipv4 only

 							nsp > network specific prefix
 							wkp > wellknwon prefix
 							*-*-*-*-*-*-*-*-*-*-*-*-*-*-*
 							dns 64 prefix and AAAArecord
 							nat64 router
 								scenario 1

 									dns 64 server ipv6 ------- client ipv6 ------- r1 --------- ipv4 only webserver
 									--------------------------------------
 									ipv6 only

 									on dns6only we have nat 64 prefix inside on dns 64 we use prefix /96
 										nat64 prefix > 2001:db8:cafe:aaaa::/96

 									from dns6only we have converting ipv4 queries like a record to aaaa records posibilty
 										wwww.example.com
 											check by it self or retrieve from root hits (autrative)
 											so on simple response from ipv4 see black response
 											use ipv4 autrative mode to reach ip and arecord

 										10.10.10.10 > 0a0a:0a0a

 									then mix 
 										2001:db8:cafe:aaaa::0a0a:0a0a
 										-----------------------------
 										destination server

 									responses must reached to edge and clients but this is not real server responses
 									routers received this what should to do?
 										nat 6to4
 										check prefix part
 											prefix part is 2001:db8:cafe:aaaa/96

 										omit from request
 										and translate another parts 0a0a:0a0a and conevrt to ipv4

 										nat 64 prefix
 											nsp > define by admin
 											wkp > a specific format like 64:99fb::/96

 								scenario 2

 									ipv4 dns server---- client ipv4 only ------ r1 -------- webserver ipv6 only

 									router has nat 64 toward webser side

 									on client request www.example.com
 									translate as a-record on ipv4 with fake id like 172.16.1.10

 									our real ip is ipv6 is 2001:db8:feed:1::e 
 									before request 172.16.1.10 on nat64 server  created map from ipv4 to ipv6

 									router received a packet with
 										destination > 172.16.1.10 : 80
 											on destination router has map table
 											knwo this ip will be mapped to 2001:db8:feed1:e

 										source > 192.168.2.10
 											here must map and translate ipv4 address to ipv6

 											2001:db8:cafe:aaaa:c000:020a
 															   ----------
 															   192.168.2.10

 						stateless
 							scenario 1
											    		  (ipv6-ipv4)
 								clint (ipv6) ---------------- r1 ------------------- webserver (ipv4)

	 							like statefull but many client request
	 							on stateless we have no state or table for them
	 							doesn't store sources 
	 							use ipv4 range which injected to clinets ipv6
	 							bat 64 router received ip and requests from clients which has injected ranges
	
	 							client has aaaa-record request

	 							router omit prefix part and remained 32 bitwill be converted to ipv4

	 							prefix and 32 bit ipv4 combined then goes to ipv6 client (realtime binding no table or storing state)

	 							need one ipv4 for each ipv6 

 				nat-protocol translation (nat-pt)

 		deployement models
 			dualstack
 				each device must support ipv6
 				if were not in backbone get trouble

 				no tunnel
 				qos
 				performance
 				security
 				multicast

 				need buy newe equipments

			hybrid
				isatap + mct + dualstack

				in datacenter we have dualstack stet one part is ipv4 only
				solution is tunneling
				we don't need upgrade
				has no multicast
				our access connectivity to core layer is not correct
				has qos, security and routing 
				too many tunnels

				need too many monitoring and management tools
				must buy mls and config on them

	nat
		network address translation
			static nat
				each system has own and specific ip publuc without release

				known as one to one nat

			dynamic nat
				known as many to one nat
				has release public ip

			dunamic nat overload (pat)
				port address translation

				lan------router---------internet

				lan range were 192.168.1.0/24
				inside global address is ip which set on organization routers and make translate from ip private to ip public then goes to internet

				outside global address
					is address whic from internet wanna access to inside and translate from public ip to private lan

				outside local address
					is range of private lan behinde the public ip address 

			overlapping means soem organizations use same ip addressing as lan ranges also translate and connect eachother
**************************
routing protocols
	isis
		flat and hierarchical like link states

		distance vector protocol use bell man ford algorithem

		error-prone

		poison reverse complementray of split horizon

		isis cost interface is 10 (not so handsome on selections)
		send other networks with infinit metric

		counting infinit on isis cause loop advertise and network learning on fabric

		dijekstra algorithem

		lsp (link state packet)

		osi > clnp (connection less network protocol) (routed protocol)
			end system > es or client
			intermediate system > is  or routers

			between es and is we have path and routing will be on them
			between is and another is we have is-is routing protocol

		tcpip > arp and ip (isis)(is routing protocol)

		differences between now days is-is routig protocol and old days
			old versions were used for osi and new versions are tcpip base

		ipv4 and ipv6 on isis name dual or integerated

		old version each device has own uniques address not interface address which called n-sap
			like router-id we had
			nsap (network service access point)
				es
				is

				*each system address on osi model

				base on operating system

		*still works on nsap
		is contain circuite which worked on layer 2 address not layer3 (like interface on tcpip)
		on osi model we had domain term which migraate to tcpip and known as autonomous system (as)

		nsap
			used in isis and router id used on ospf

			initial domain part (idp) > include 2 part which made organization code
				afi > authority and format identifier
					organization identifier and formats code of dsp

				idi > initial domain id

			domain specific part (dsp) > iinclude 3 part which made inisde organization address
				hodsp > high order bit dsp

				system id
					determine the system-id 

				sel
					inside codes which name network selector

		for every thing must set a code
			47 is international
			45 is international phones

			like 0098 or 21 define organization or city or....

			afi > 39 define data country code
			idi is 2 octed
			hodsp is 10 octed area

			iran 
				39.0098.
				---------
				idp

			localy define or private is code 49

			49		.0001.		1111111111111. 		80
			-- 		-----		------------- 		---
			idp  	hodsp 		mac 				application
					-------------------------------------------
					dsp

			idp is 49
			hodsp is area number
			system id is unique id
			nsel is 80 or 00 

			isis is working on clnp

			isis works on nsap and advertise ip
			just used on routers not application

			usually set nsel or application part at above like 00 in dsp to define devices self generated packets
			also we have no program on clnp framework

			at the config networks offers set code 49

			capacity from 20 to 100 byte
			at&t organization used isis inisde their networks 

			router isis
			net 49.001.1111.1111.1111.00 

		multi topology is model of combination ipv4 and ipv6 together

								domain 2
							*////////////////////*
									is --- es5
									|
									|
									|
		es1 --- is ---- is -------- is -------- is ---- is --- es4
				|		| 						|
				es0  	es2 					es3
		***************************	*****************************
		area 1 								area 2
		*///////////////////////////////////////////////////////*
		domain 1

		l0 > intr-link
			many es on one link

			es conencted to is

		i-isis
			l1 > intra-area
			l2 > inter-area l
	
		l3 > inter-domain
			idrp
				inter domain routing protocol
				named isoigrp , depricated

		on lsp (link state packets) advertise links attributes and states 
		create lsdb and run spf (shortest path firs) to select the best path toward lsdb networks
		cause spf is resource intensive we have trouble on large scales 
		so make area to segment them
		each router has owned route map as same area

		80.001.6666.6666.6666.80
		-- --- -------------- --
		1   2        3         4

		1 is afi
		2 is hodsp area
		3 is system unique id
		4 is application base on clnp

																area 001
								////////////////////////////////////////////////////////////////////				
												(lp1 1.1.1.1/32)			|r2 (lp2 2.2.2.2/32)
								|-----------------r1------------------------| 						level 1
		(lp4 4.4.4.4/32)		|											|r3 (lp3 3.3.3.3/32)
		r4----------------------| 				level 1/2
		area 000				|								|r6 (lp6 6.6.6.6/32)
		level 2 				|-----------------r5------sw----| 						level 1
											(lp5 5.5.5.5/32)	|r7 (lp7 7.7.7.7/32)

								\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
																area 002

		192.168.14.0/30
		192.168.45.0/30
		192.168.12.0/30
		192.168.13.0/30
		192.168.5.0/29 (cuase we have switch on area 2)

		on osi each system has one nsap
		each computer might have a nsap 
		each is has nsap on area

		r1 nsap > 49.001.1111.1111.1111.00
		r2 nsap > 49.001.2222.2222.2222.00
		r3 nsap > 49.001.3333.3333.3333.00

		r4 nsap > 49.000.4444.4444.4444.00

		router isis
		net 49.001.1111.1111.1111.00

		*must define routers type
		isis is same with ospf but different will be on area types and areas

		area types
			level 1 > intra-area
			level 2 > inter-area (abr) (or level 1/2)
			level 2 only > backbone (area 0)

		inside area with level 1/2 could complete the lsdb

		r1 is advertising the connected networks and nsap will be part of advertisement
		on level 1 say i have nsap of r2 and r3

		also r3 say i have nsap and network include r1 and r2 on level 1

		on isis we have different zoning like area and each router could be on one area at maximum
		border of routers are links

		r3 in additional informations send attached to bit which said i am conencted to level 1/2 or level 2

		on level 1 routers if need connection with other routers which are on another area must send to r3 that contain and have attached to bit

		this bit (atached to) behave like default route

		our inter-area traffics will be on level 2 ddatabaase

		abr routers or level 2 (1/2) advertise each network to fabric
		this advertisement will be like dominos patterns toward other abr
		metric parameters also will be on many abr help us access to other networks

		router isis 1
		!like ospf has process id and help us to redistribute the process to eachother

		*isis has no log on neighborship

		on intermediate systems (is) each level assignement will be effect on circuite or interfaces so on levl 2 or 1/2 abr routers must set which interface faced level 2 only and which one faced on level 1

		on r1 cause router is level1/2 or level 2 one and goes to levl 2 only and another one goes to  level 1

			r1
				router isis 1
				net 49.001.1111.1111.1111.00
				is-type level 1-2
				log-adj-changes all

				interface gig 0/0
				ip router isis 1
				isis circuite-type level-1

				interface gig 0/1
				ip router isis 1
				isis circuite-type level-2

			r2
				router isis 1
				net 49.001.2222.2222.2222.00
				log-adj-changes all
				is-type level-1

				interface gig0/0
				ip router isis

				*here doesn't need set circuite-type cause router is level 1 if set get omit on config

			r4
				router isis 1
				net 49.000.4444.4444.4444.00
				log-adj-changes all
				is-type level-2-only

				interface gig 0/0
				ip router isis

			*for loop back interfaces must use another method
				interface loop 0
				router isis 1
				passive-interface loop 0
				!isis is active without hello, metric is 0

			show ip route
			show ip route isis
			!code i means isis with administrative distance 115 and metric 10

			on level-2-only routers must be careful connection type and neighborship
			*better set all zones and routers as level 2 if use fla design

			level 2 is more flexible

			hierarchical design has backbone area

			show isis neighbors
			show isis database details

		*metric 
			default is 10 and ospf run on this

			delay

			expenses

			error
				lowest error rate

			router isis 1
			metric 20 level-1
			-------------------------
			interface gig0/0
			isis metric 5 10 10 10 level 2
			isis metric 5 10 10 10 level 1-2
			isis metric 5 10 10 10 level 1
			!default delay expenses error

			*now use metric on value 63 that is the maximum number in isis not higher

			narrow is first metric type with  6 bit
			wide metric is second metric type with 24 bits then converts to 32 bit

			show clns interface gig0/0
			show ip protocols
			show isis protocol
			!wide metric or narrow metric

			router isis 
			metric-style transition
			!accept wide and narrow at the same time, then insert next command and force them to use wide metric

			metric-style wide

		types
			broadcast 
			point-to-point

		passive interface advertise
			router isis
			no advertise-passive-only

		packets and mecahnisms
			hello is 10 and dead is 30 seconds

			after lsdb creation our spf put all best records on rib
			doesn't have define many spokes posibilty
			must define a router as designated is (dis)
			send informations as psudonode with metric 0 like no distance feeling

			*if used ethernet on each side doesn't need dis 
			better set this
				interface gig0/0
				isis network point-to-point

			*each psudonode records on cisco get omit after 20 minuts and other on 1 minuts

			dis help broadcst and database synchronization, also simulate network on many links and point-to-point 

			dis on 10 seconds interval send csnp
				sequence number which is revision number or code
				and lsp that advertised and compare sequence numbers 
				if were different sent psnp as update database request

			level 1 packet > 0180.c200.0014
			level 2 packet > 0180.c200.0015

			level 2 is on 802.2 and 802.3

			on ip header
				ssap and dsap > 0xfe

			dis election
				higher priority on interface 0-127
					127 is highest and 0 means disable dis 
					default is 64

					interface gig0/0
					isis priority 1 level-2

				highest snpa (subbnetwork point attachement)
					on ethernet means mac address
					highest mac address

					on framerelay and atm justuse highest system-id

				highest system-id

		authentication
			hello packets and lsp can be authenticate

			link authentication has md5

			area authentication (level 1)(plaintext)
			domain authentication (level 2)(plaintext)

			subnetworks must be on link authentication

		ipv6
			use newest tlv
	-------------------------------
	eigrp
		eigrp use acknowlege as reliable mechanism with reliable transport protocol (rtp)

		summary administrative distance is 5 on rib
		255 router can be on eigrp and by default use 100
		between core and distribute design layer be best choice

					multicast 	unicast 	reliable 	unreliable
		-------------------------------------------------------
		hello 			* 			* 						*
		-------------------------------------------------------
		ack 						*
		-------------------------------------------------------
		update 			* 			* 			*
		-------------------------------------------------------
		query 			* 						*
		-------------------------------------------------------
		reply	 					*			*
		-------------------------------------------------------
	-------------------------------
	bgp
		qos policy propagation on bgp (qppb)
	-------------------------------
	ospf
		for hub&spoke scenarios must use same area on spokes and placed abr on hub

		lsa header is 80 byte
			lsa age
			lsa type
			link state id
			advertise router
			ls sequence number
			ls checksum
			length

			lsa 8
				sbit
					00 > link local
					01 > area
					10 > as
					11 > not use

				abit
					0 > localy
					1 >

	route manipulate > pbr

	route feedback
		2 edge for organization unit make redistribute sme routes

	bfd (bidirectional forwarding detection)
		igp has 1-2 seconds fault detection

		bfd is fater than auto detection
		is a independent protocol tool that performs on data plane not control plane

		some equipments are modular
		some of them has many supervisors and has fault telorance
		on fsso with fast convergence might lost neighborshiping so many neighbors will not be valid 
		for these reasons must set bfd on all edge routers

		nonstop forwarding mode (nsf)(nsr)
			need more resources

			bgp
			isis

		gracefull start (gfs)(gr)
			ospf
			isis
			eigrp
			ldp

		on 7604 routers
**************************
ip multicast and network management
	multicast
		distance vector multicast routing protocol (dvmrp)
			224.0.0.4

		oui 			vendor assign address (vaa) or nic
		1 	2 	3  |	4 	5 	6

		on part 1
			7 6 5 4 3 2 1 0
						  -
						  might be 0 or 1
						  	0 > unicast
						  	1 > multicast

		layer 3 to layer 2 mapping
		specific virtual ip and mac

		aafter ip multicast set virtual mac

		ip 226.1.1.1 (has no arp on multicast)

		ip 239.192.44.56
		make multicast mac based on ip multicast
			  1110 1111.1100 0000.0010 1100.0011 1000
			  e 	f 	c 	 0 	  2    c 	3 	 8
			  			*----------------------------
			  			 last 23 bits will be fixed and consider as mac 
			  			 * point means set and inject 0 as default

			  0100.5e is prefix of multicast mac address (6*4bit > 24 bit)
			  another 24 bits will be mixed of ip address fixed part	

			  0100.5e + 0 (default as 25th bit) + 23 bits of ip binary values (100 0000.0010 1100.0011 1000)

			  0100.5e +(0 100 0000.0010 1100.0011 1000)
			  			4 	  0    2    c 	 3 	  8

			  0100.5e40.2c38

		ipv6 use multicast listener discovery (mld) instead if igmp

		igmp is internet group message protocol protocol number > 2 over ip

			asm (any source multicast)
				igmp versions > 1, 2, 3

			ssm (source specific multicast)
				igmp versions 3

				more secure

		cisco group management protocol (cgmp)
			router forward multicast traffic to lan
			switches convert multicast ip to multicast mac address
			just learn source mac, no body learned multicast source mac

			flood on all interfaces except igress port
			instead of multicast here we are using broadcast

			need just forward traffic to one client which requested multicast traffic

			on switch must run 
				cgmp
					client make membership report toward switch
					router received membership then set cgmp packets
					and detect client mac then request for multicas mac address
					with cgmp detect which port need multicast

					cgmp is not working on cisco switches
					has fast leave mechanism

				igmp snooping
					switch listen to igmp pacekts each client sent memebership report to router must forward from switch so cache the mac on mac table
					need hardware equipments
					doesn't have too many loads


				rgmp

				convert all broadcast to multicast

		*in multicast doesn't advertise or learn routes seperatly to some one

		sparse mode
			cbt (cone base tree)
				cisco doesn't have this

			pim-sm

		*dense mdoe consider all nodes need multicast traffics
		*sparse mdoe consider some nodes need multicast traffics

		dense mode
			dvmrp
			mospf
			pim-dm

			source ip in multicast will be like unicast
			destination ip will be any thing

			open an application on system and active multicast ip on client so start multicast negotiations

			in dense mode saay if router received multicast packets flood them cause many users need multicast traffics
			some routers didn't request igmp so waste resources
			here must prone the traffic 
			in multicast we have a tree which called multicast source
			from multicast source send to all nodes then prune them and get shorter each 3 minuts

			(s,g)(source, group)

			source tree
			multicast routing table
				(192.168.1.1,226.1.1.1)
				-----------  ---------
				s  			 d

				incoming interface > gig0/0
				outgoing interface > gig0/1

			show ip mroute

		sparse mode
			use pool or rendezvous point (rp)

			multicast source send traffic to rp and each node need multicast traffic send connection request (pim) to rp
			source is like (8,g)

			use share tree

			distance vector multicast routing protocol doesn't support on cisco but could be activate

			mospf doesn't work on cisco and doesn't support it 

		bidirectional pim

		multicast source discovery protocol (msdp)
			we have 2 as-number one of them we have multicast with pim-sm

			our traffic will be forward as usual but clients on another as-number couldn't receive traffic
			cause couldn't connect to rp or see that
			msdp make conenction between another domains and make inter-domain tree

			msdp neighbor will be happen 
			if sources wanna register to rp 
			rp send as-numbers 
			source active message will be sent
				source address
				group
				rp ip address 

	network management
		rmon agent > advance monitoring
			remote monitor
			extentions on snmp with more information collection than snmp
			structure like snmp
			behave like snmp
				rmon probe is like snmp agent
				rmon console is like snmp server

			has mib on mobile devices
				solicited and unsolicited to nms

			snmp focus on device information and attributes

			focus on network traffics like netflow and sflow

			version 1 
				rmon has 10 categoriesed
					statistics
					history
					alarm
					host
					host top n
					filter
					capture
					events
					token ring
					matrix

				placed on layer 2 and layer 1

			version 2
				has baackward compatibility
				also more features like
					protocol discovery
						list of device supported protocols 

					address mapping
						mac to ip binding

					nlhost
						network layer host
						specific host and ip , traffic count

					nlmatrix
						network layer metric

					alhost
						application layer host

					almatric
						application layer metric

						base on ip address check many application traffic

					user history

					prob config 

		fcaps
			fault management
			config management
			security management
			accounting management
			performance management

		snmp
			unsolicited  is mecahnism which has no request before
				agent push
				like trap or inform on udp 162

			solicited
				agent pull
				like 161 udp

			mib
				management information base
				store some informations about device

				mib module
					manageable documents which controled by nms

					abstract syntax notation (asn)
						name > oid (object)
						syntax > data types
						encode

			versions
				v1
					get (request)
					get (response)
					trap
					set request

					community ro (readonly) and wr (read and write)
					simple authenticate and no hash
					32 bit slow connections

				v2c
					more speed
					use inform mechanism as ack and tcp behavior
					use get bulk

					management traffic use seperated  vlan and loop back for management

					out of bound management is useful way to manage devices from seperated network

		cdp
			show cdp neighbor details
			!device id, local interface, capability, platform, ip address, ios version

			show cdp entry device-id

			show cdp traffic

			*cdp mac > 0100.0ccc.cccc

			cdp timer 50
			!default is 60

			cdp hold-time 150
			!default is 180

		lldp
			lldp run
			lldp transmit
			lldp receive
			!seperated send and received

			lldp timer 30
			lldp hold-time 120

			*bydefault is disable  802.1 ab
			instead of showing switch term use b

			just show aactive features

			ether type 0x8cc

		syslog
			udp 514

			attributes
				timestamp
				facility (type)
				level (severity)
				events

			logging host 192.168.1.1
			!define syslog

			logging traps

			interface gig0/0
			logging event lin-state
			show logging


		netflow
			replaced with internet protocol flow information export (ipfix)
			works on version 9

			contains 
				destination ip and port
				source ip and port
				tos

				routers collect flow of transmit and received data send to flow collector

			useful on network planning

			rmon app
			network data analyzer
				data presentation
				network planning
				accounting and billing

			network tools planning
			accounting or billing app
				data flow in switching
				data flow aggregate
				data flow export

			netflow collect
				data flow collector
				data flow filter
				data flow summary
				data flow aggregator
				data flow storage
				file system management

				like cisco netflow collect, solarwinds, ca netqos

			netflow accounting is like snmp agent

			flexible netflow
				more options on netflow
					source and destination mac address
					fow time stamp
					source and destination ipv4 and ipv6
					tos
					dscp
					ports
					in or out interfaces
					byte and packets count

			*snmp use pull and device base statistics
			*rmon contain packet statistics and collect data then analyze them
			*netflow control packets statistics, less resource dependency, more items than rmon, realtime monitoring
**************************
enterprise lan design and technologies
	lan design
		tcpip
			enhanced
				application
				transport
				network
				datalink
				physical

			traditional
				application
				transport
				network
				datalink

			*datalink and physical is inseparable

		ethernet placed on layer 1 and layer 2

		dix (digital, intel, xerox) made ethernet 10mb in 1980

		ieee802.3a or ethernet v2

		8 is year 1980
		02 is second month of year
		3 is standard number of ethernet 

		ethernet frame is same on each version

		csmacd is on every model
			on sending data check line with signals if had checked the line means same value has no collision if were different means collisioin
			on ffirst 64 byte of packets must detect the collision
			on fast ethernet we have 100 mbps means 1 bit transmit on 0.01 msec
			on ethernet with 10 mbps and 1 bit has 0.1 msec

		twisted pair 
			100mbps base
			full duplex
			cat5 still usefull which has 2 active pair not 4 
			above cat5 we have higher speeds

			coding 5b and 4b means which bits transmit on which pins

		twisted 4 
			100mbps base
			more pairs used
			at the start were cat 3 on 10mbps
			then get fast ethernet on 100mbps
			without replacing cat 5 on cat 3 provide speed as 100 mbps
			8b8t coding
			half duplex		

			coding 5 level
			1250 mhrz
			100 meter	

		fiber optic
			fast ethernet over fiber
			used on long distance
			4b5b coding

			sc > straight connection 
				stub and click

			st > straight tip
				stub and twist

			mic > medium interface connector

		gigabit ethernet
			1998
			802.3z > coxial
			802.3ab > nrz
				cat5e
				simple non-return-to-zero
				1250 mhrz

		1000base lx(long wave length)
			550 meter smf(single mode fiber) or mmf(multi mode fiber)

		1000base sx(short wave length)
			100 or 500 meter mmf

		*long wave is better cause has less contact with edges

		1000base cx(coaxial)
			25 meter
			150cb tanax

			server connections 8b10b coding (snrz)

		wave length (nm)
			850 is short > 8b10b coding
				on mmf 62.5u and short nm we have 260 meter
				on 50u short nm we have 550 meter

			1300 or 1310 long
			1550 extra long

		mmf use led and thicker 50u (550 meter) or 62.5 u (440 meter)
		smf use lazer and thiner 8u or 9u (5k meter)

		coding 8b10b is simple nzr 

		802.3ae
			fullduplex 10 g  over fiber

		802.3an
			10 g over cable cat6

		10g is fullduplex


		man > wan > dataacenter > backbone > server farm

		r term in cables means lan and w means wan
		66b coding

		10gbase-sr
		10gbase-sw

		wan interface sublayer (wis)
		sonet
			synchronous optical	network

		long wave + 10k meter + 1310 nm
			10g base lr (lan smf)
			10g base lw (wan smf)

		10g base er(lan smf) and ew (wn smf) extra wave > 1550 nm 40 km
		
		10gbase lx4 multiplexer mmfand smf 10 km 802.3ak
		
		10gbase cx4 4pair coaxial tanax 15meter 802.3an

		10gbaset cat6 utp 100meter
		
		10gbase zr longwave smf 80 km
		
		10gbase pr passive fiber 20 km

		10g epon 802.3 av

		802.3ba > 40 and 100 gbps

		ether channel
			manual
				channel-group 2 mode on
				!without determine peer values make aggregate and goes to error disable, prevent cdp
				!channle misconfig

			show ether-cahnnel 1
			show ether-cahnnel summary

			802.3ad(lacp)

			recommended modes are auto be on one side and another side be desireable

							auto 		desireable

			auto 							+
			desireable 		+				+

							active 		passive
			passive 		+ 			+
			active 						+

			auto with passive doesn't offer aggregation just accept aggregate

			after aggregation what happen on macs in mac table?
				usage from aggregation links are lower

			9 loadbalance model
				src-mac 
				dst-mac
				src-dst-mac
					*higher accuracy on swithces

				src-ip 
				dst-ip
				src-dst-ip

				src-port 
				dst-port
				src-dst-port

			the least valuable will be calculate on loadbalance algorithem

				mac1 > 1111.1111.1111 > 01 > 1 > goes on port 1
				mac2 > 2222.2222.2222 > 10 > 0 > goes on port 0
				mac3 > 3333.3333.3333 > 11 > 1 > goes on port 1

				if had 2 links on loadbalance algorithem device consider 1 bit
				if were 4 links consider 2 bits

				if set algorithem on destination mac our ether-cahnnel is not so useful
				better set on ip instead mac
				set algorithem on parameters which has various types

				src-dst-mac
				src-xor-max
					1 xor 1 means 0
					1 xor 0 means 1
					0 xor 1 means 1
					0 xor 0 means 0

					pc 1 means 0001 to 	  pc 1	links
						0101 > 5 > 01 xor 01 	> 00
						0110 > 6 > 10 xor 01  	> 11 
						0111 > 7 > 11 xor 01 	> 10 
						1000 > 8 > 00 xor 01 	> 01

					pc 2 means 0010 to 	  pc 2	links
						0101 > 5 > 01 xor 10 	> 11
						0110 > 6 > 10 xor 10  	> 00 
						0111 > 7 > 11 xor 10 	> 01 
						1000 > 8 > 00 xor 10 	> 10

				static persistance config 
					mode on 
					no traffic will be control on aggregate

		vlan assignment
			static

			dynamic
				doesn't support pagp

			vlan memebership policy server (vmps)

			between access and distribute layer offers use mmf
			better use smf between core and distribute

		poe
			inline power (ilp) > 7w
			poe > 15.4w 802.3af 2003 (ieee proprietary)
			poe + > 25.5w 802.3at 2009 (ieee proprietary)
				*2pair of twisted pairs (1236)

			cisco universal poe (upoe) > 60w 2011 (cisco proprietary)
			upeo + > 90w 802.3bt 2013 (cisco proprietary)
				*use all pairs

			2960x and 2960si support poe on cisco devices

			poe class
				0 is default 15.4w
				1 > 4w
				2 > 7w
				3 > 15.4w
				4 > 802.3at on 30w an d higher

				use cat 5e (poe and poe + and upoe +) and cat 6a (upoe +)

			power source equipments > switch

			powered devices
				devices which need poe

		wake on lan (wol)
			pc with received signal on nic get start (wol magic pacekt)
			switch must forward magic packets

		stp and lan design
			spanning-tree mode pvrst
			spanning-tree link-type point-to-point
			!use this in emulators

			show spanning-tree root
			!shr means share and halfduplex
			!p2p means point-to-point and fullduplex

			show spanning-tree bridge
			!changed priority modified our bridge id like *4096
			!default is 32768

			spanning-tree vlan 1 priority 4096

			root bridge
				lowest bridge-id
				root port (rp)
					ports toward root bridge with lowest cost

				designated port (dp)

				4mbps > 250
				10mbps > 100
				16mbps > 62
				100mbps > 19
				1gbps > 4
				2gbps > 3
				10gbps > 2 


			spanning-tree path-cost method short
			spanning-tree path-cost method long
			!16bit, for 100 gbps use long and 32bit

			show spanning-tree summary

			*root bridge doesn't have root port

			spanning-tree cost 100
			!change cost


						received bpdu 	send bpdu 	learn mac 	forward
			----------------------------------------------------------------
			disable 	
			----------------------------------------------------------------
			block 		* 				
			----------------------------------------------------------------
			listen 		* 				*
			----------------------------------------------------------------
			learn 		* 				* 			*
			----------------------------------------------------------------
			forward 	* 				*			*			*
			----------------------------------------------------------------

			on backbone fast if get trouble on one link wait till 30 seconds
			then
			 	indirect > 30 (wait) + 15 (listen) + 15 (learn)


			 	a packet called rlq (root link query)
			 		are you alive as root bridges
			 		doesn't wait as 20 seconds
			 		doesn't wait for bpdu just ask and make query then goes to listen state

			 		switch 1 (root bridge)
			 			|\ 	
			 			| \
			 			|  \
			 			*   \
			 			|    \
			 			|     \
			 			|      \
			 	switch 3-------switch 2

			 	switch 3 send rlq to switch 2 and wanna fetch root bridge  so switch 2 response and reqeust rlq to switch 1 and proxy that

			 	better set between  distribute and core (backbone fast)

			 port fast
			 	doesn't work with listen and learn
			 	no tcn (topology change notification)

			 	if received bpdu get error disable on manual mode must set no shut command under interfaces

			 *root gaurd protect root bridge (inconsistent)
			 *loop gaurd used for loop protection and bpdu packets on alternates and root ports check this if did not received make inconsistent if received make root port and bring up
			 *bpdu filter must be enable on isp or lan side to make isolation on stp

			 udld
			 	unidirectional link detection
			 	udld echo received on switch 1 and transmit on switch 2
			 	2 packets with no dependencies sen at the same time

			 	udld echo *3
			 	udld wait or try 8 seconds

			 	*unless receive udld echo or response our udld mechanism doesn't get start

			 	each 15 seconds send udld echo

			 stp > 802.1 d
			 rstp > 802.1 w
			 mstp > 802.1 s

			 rstp focus on interfaces turning up

			 stp > 15 listen + 15 learn + 20 inidrect > 50 second
			 rstp > 15 learn +  6 indirect > 20 seconds
			 	use synchronous mechanism instead of listen
			 	try to be up not use timers

			 	flags in rstp are like bpdu
			 		aggrement
			 		proposal
			 			if doesn't use listen state goes to loop 
			 			here we use designated ports use proposal
			 			other switches on fabric must compare their proposal with the main switch or root bridge
			 			block all interfaces except edge with prevention on loop for root bridge
			 			then sent aggrement packets to make negotiations and connectivity
			 			this mechanism happen on another root bridge interfaces

			 	interface gig0/0
			 	spanning-tree portfast 
			 	spanning-tree portfast edge
			 	!edges are like port fast and connected to clients
			 	!edges doesn't be on synchronous mechanism

			 	spanning-tree portfast edge default
			 	!set on global

			 	*if rstp didn't receive aggrement packets works on normal stp so better use edge command on client sides

			 	tcn (topology change notification)
			 		used for databases
			 		on rstp we can send bpdu as configuration target
			 		on rstp after interface turning on send this
			 		on rstp automaticaly notify about this so tcn doesn't matter

			 		aging time = 0 and tables get change fast
			 		portfast omit the tcn pacekts

			 		on normal stp we had 15 seconds for tcn

				congest 1-2 seconds and timeout 1 or 2 times

				in mst better use automated proning mechanism 

		ha and campus
			messangers are peer to peer or instance messaging

			lan
				access to distribute layers
					traditional
						hsrp

					updated
						vss
							no hsrp + multi chassis ether channel
							no stp

							4500, 6500, 6800

					layer 3 modes
						mls

			better use dynamic trunking protocol on switching design
			also use manual trunk and no negotiations
			on distribute layer prone all unused vlans
			vlan trunking protocol might have bugs and troubles, better design on geographical design
			vtp transparent help us to see less problems and troubles

			distribute layer must be fastest layer in design so must be careful and set redundancy
				standby 1 authentication txt 123
				standby 1 authentication md5 key-string abc
				standby 1 authentication key-chian chain-1
				standby 1 track 1 decrement 50
				!default is 10

				*authentication is enable bydefault with cisco key

				standby v2
				interface vlan 1
				standby ipv6 autoconfig

				track 1 interface g0/0 iprouting
				!layer 3

				track 1 interface g0/0 line-protocol
				!layer 2

				switch 2
					interface vlan 2
					ip address 192.68.2.2 255.255.255.0
					no shutdown
					standby 2 priority 200
					standby 2 ip 192.168.2.5
					standby 3 priority 150
					standby 3 ip 192.168.2.4
					
				switch 3
					interface vlan 2
					ip address 192.68.2.3 255.255.255.0
					no shutdown
					standby 2 priority 150
					standby 2 ip 192.168.2.5
					standby 3 priority 200
					standby 3 ip 192.168.2.4

			load balancing
				vrrp
					states
						disable
						initiate
						backup (master)

						bigger ip will be master and others will be backup

						mac 0000.5e00.01xx
							xx is group memebrship

					bydefault preemtive is enable
					cisco swithces on 4k and 6k series use vrrp
					3750 and 3850 just has hsrp

					pid 112 on ip
					ip address 224.0.0.18

					version one is ipv4
					version 2 is ipv6 and ipv4 

				glbp
					224.0.0.102
					udp 3222

					0007.b4xx.yyyy
					xx is group memebrship
					yyyy is router number in group

					manage 4 routers at the smae time more than 4 will be backup

					avg
						replied arp requests on vmac and vip

					avf 
						forward traffics

				phantom routers are virtual routers works like arp rpoxy and has some connectivities

				vss (virtual switching system)
					4500
					6500
					6800

					2 chassis at the same time works like one device or switch 
					no stp
					has ether channel
					doesn't need hsrp no fhrp no vrrp 
**************************
wan
	l1
		dwdn
			dense wave length division multiplexing
				use fiber optic as better ways
				convert one fiber to many channels and works better than normal 

		cwdm

	l2
		frames will be transmit 
			layer 2 vpn - virtual private lan services (vpls)
				point to multipoint

				our isp will be like media as largest bridge

				ethernet multipoint services (ems)

				independent on layer 2 but has dependencies to isp layers

			virtual private wire services (vpws)
				point-to-point
				wired connection or dedicated line to isp
				leased line
				psudowire
					vpn without protection and security

	l3
		mpls
			layer 3 vpn
				use vpnv4 and mpls also ibgp for conenctions

		fast reroute has 5 level sla

	metro ethernet
		mpls over ethernet
		omit ethernet limitations

	optical carrier
		sdh or sonet
		works like ring
		has self recovery
		layer 1
		cisco ons 15454

		oc1 51 mbps
		oc3 155 mbps

	wan wireless
		gsm
			global system for mobile communication 

			tdma 900 1800 1900 mhrx

		gprs
			extended gsm
			general packet raio services

		umts 3g
			universal mobile telecommunications services

		lte
			combination of many frequency bands 

		lte advance pro (gigabit lte)

	qos
		strategies
			best effort (be) has no qos
				used for normal data types and traffics

			deffrentiated service code point (diff serv)
			integerated services (int serv)
				policing
				queueing
				admission control
				classification
				schedule

		network control > cs 7 56
		internetwork control > cs 6 48
		voip > ef 46
		broadcast video > cs 5 40
		multimedia confernces > af 4 34-48
		realtime interaction > cs 4 32
		multimedia stream > af 3 24
		transactional data > af 2 18-22
		network management > cs2 16
		bulk data > af 1 10-14
		scavenger > cs 1 8
		best effort > cs 0 or default

		nbar
			layer 7 inspection 

	green field means all fabric managed by admin
	brown field means network on change
**************************
sd access
	overlay design
		segmentation
			macro segmentation
				vlan segmentation
					control and data plane

				intervaln routing
				inter vn communication

			micro segmentation
				sgt tag

				has acl and manage control plane

		reduce ip subneting
		avoid overlapping ip address

	fabric design
		control plane design
			6 is wire and 4 is wireless

			database for mapping and routing locator

	sdwan design
		app1, app2 > applications
		api/yaml sdk > api
		xml, json > encode
		netconf, gprc, restconf > protocols
		ssh, http > transport
		yang model > models

	simple object access protocol (soap)
