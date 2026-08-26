MPLS L2 (VPLS)
****************************
general speaking
	more expensive than mpls layer 3
	cuase use many protocols on connectivities and isp will be like switch
	layer 2 vpns
		point to point
			atom
			l2tpv3(has ip packets)

		multipoint
			vpls

	pseudowire emulation framework
		simulate layer one and two over pacekt switch base network like tdm on layer one and frame relay on layer two
		virtual wireor connection (circuite)

											psn (packets switch base network)
								/////////////////////////////////////////////////////
	customer edge --------------- provider edge |---------- provider -----------|
												|								|
												|******pseudowire emulation******|
												|								|
												|----------	provider -----------| pe ---------- ce
																					*************
																					attachment circuite
																						ethernet / vlan / ppp / hdlc / framerelay / atm

	to make simulation we have to signaling mechanism
	it is not on transparent mode
	pe routers could not understand pseudowire and they are ip base

	payload
		emulation service

	pseudowire encapsulation layer
		encapsulation
			important fields will be encapsulation on sublayers for layer 2 connectivities like dlci

		demultiplexing
			demultiplexing sublayer set new headers with ce detection and isolation on pe routers
			or determine forward over which attachment circuite cause we have many pseudowire between pe and ce
			has sequence

	network
		psn tunnel
			isp information and headers

	link

	physical

	atm modes
		aal5
		cell

								control plane
					-------------------------------------
	ce 							system management							 			pe
			link layer 												pseudowire| network
			protocol 												protocol | protocol
			controller 												processor| processor
			------------------------------------------------------------------
									data plane

			1* native												2* pseudowire encapsulation
			service 												-----------------------
			processor 												4* network forwarding engine
			---------- 												-----------------------
			device driver 											3* device driver
			--------------											-----------------------
			physical 												physical 
			interface  												interface

			*1 > is necessary information on layer 2 aggregate all packets then on peer pe router reassemble the packets

			*2 > our labels between pe and ce

			*3 > layer 2 headers processing

			*4 > network forwarding engine network forwarding engine looks up the address in the ipv4, ipv6, or mpls forwarding tables
			if it finds an outgoing interface it encapsulates the packet with the appropriate link encapsulation and sends the packet out of the output interface
			otherwise it discards the data packet

			*pseudowire protocols processor and network protocols processor performs routing protocol signaling procedure and pseudowire
			*link layer protocols controller performs line protocol signaling like framerelay lmi and atm ilmi

	isp must use l2tpv3 and mpls must use atom for pseudowire protocols

	pseudowire
		manual
		dynamic
			manual neighborshiping and automatic parameters

		auto discovery
			every thing is automated
			like bgp distribution
****************************
vpn l2 protocols
	legacy vpn l2 
		framerelay
		atm
		vpn dialup > pptp, l2f (layer 2 forwarding), l2tpv2 (tunnel protection)

	atom 
		any transport over mpls

		*2 or one pair lsp that is unidirectional  one of them send another is receive

		targeted ldp protocol on pseudowire between pe routers

		ldp is multicast cause neighbors are connected
			this label advertisement used for tunnel creation beween pe routers

		targeted ldp > works on pe make connection between 2 indirect connected devices and unicast mode
			exchange labels pseudowire inside tunnels 

			will be between pe routers and create pseudowire atom

			atom header 
				native l2 header
				
				tunnel label
					top of label stack
				
				vc label
					bottom of label stack (used for client label definition)
					handled by targeted ldp

					*vc label and tunnle label are demultiplexing sublayer that connect 2 pe routers to eachother
				
				controll word (sequence) (optional)

				layer 2 payload

			layer 2 protocols on atom 
				ppp
					it is transparent
					pe routers don't participate on ppp protocol exhcnage

					just forward the traffic without any procedure and isp get transparent on connections
					has no manipulation on signals in isp like hdlc

					ppp just forward without chehcking or storing, just forward same as received packets

				hdlc
					just on cisco is transparent
					like ppp

				ethernet 
					tagged vlan
					untagged 
					same as ppp

				framerelay
					different with others
					framerelay siwthces must implement and advertise the lmi signaling on pe to ce simulation 
					isp must be on framerelay

					control word 
						command and response
						becn
						fecn
						de

				atm
					aal5
						dynamic bandwidth 
						aggregate many packets then forward
						efci and clp used for congest into the control world

						48byte
						atm header will be isolated if were same

					cell or aal0
						fixed bandwidth and set qos over them
						each cell sent seperately and direct management on atom

						52byte
						paste many cells together and send

						modes
							vc
							vp
							port 

	l2tpv3 
		layer2 tunnel protocol version3 

		isp works on ip base

		like l2tpv2 but extended which is works on layer2 and transmit ppp packets

		all layer2 will be forward

		control and signal packets are same

		data plane has bidirectional session used for transmit layer 2 payload between ce

		*each l2tpv3 session named as pseudowire

		ip encapsulation
			native l2 header
			ip header (isp)
			l2tp over ip header (encapsulation sublayer)
			l2tp specific sublayer (optional)
				don't pass through from pe routers but on peer need this so put in it

			l2 payload 
				received from ce

			*if session id were 
				control packet > 0
				every thing except 0 menas data packet
				
		udp encapsulation
			native l2 header
			ip header
			udp header
				faster forwarding on firewall and ipsec and nat

			l2tp over udp header
			l2tp specific sublayer
			l2 payload

			*a bit at the begining of l2tp header which set and determine controll packet is 0 or not means except 0 and data packet

		*mechanisms
			sliding windows (speed controll)
			keepalive
			authentication 

		like before on ppp and framerelay

		protocols get transfer
			ppp
				operates in the transparent mode
			
			hdlc
				transportation of cisco hdlc is in transparent mode
			
			ethernet
				untagged ethernet frame
				ieee 802.1q tagged ethernet vlan frame
			
			frame relay
				frame relay frames to different pseudowires based on receiving interface and dlci number
				
				pe routers also provide lmi signaling to ce routers as if they are framerelay switches
				
				unlike frame relay over mpls, the frame relay header is kept intact

			atm
				vc mode vp mode or port mode
					how atm packets and cells should be mapped to pseudowires
				
				atm aals
					pe routers reassemble atm cells into atm aal5 packets
					forward them to different pseudowires based on the receiving interface and the vpi or vci numbers
					atm flags such as efci and clp are carried in the l2tpv3 atm-specific sublayer
			
				atm cell services
					encapsulate a single atm cell at a time or pack multiple atm cells into one l2tpv3 packet
			
		features
			path mtu discovery
			ipsec
			fragmentation
			qos (difserv)

			*has no fast reroute and traffic engineering

										atom 						l2tpv3
		-----------------------------------------------------------------------------------------------------
		network infrastructure 	 		ip/mpls 					ip
		-----------------------------------------------------------------------------------------------------
		signaling protocol 				directed ldp 				l2tpv3
		-----------------------------------------------------------------------------------------------------
		transport layer 				mpls label 					ipv4
		encapsulation					encoding
		-----------------------------------------------------------------------------------------------------
		supported layer 2 				ppp, hdlc, ethernet,    	ppp, hdlc, ethernet,
		protocol						ethernet vlan, frame 		ethernet vlan, frame
										relay, atm aal5, atm 		relay, atm aal5, atm
										cell 
		-----------------------------------------------------------------------------------------------------								 
		authentication 					tcp md5						shared secret with message digest
		-----------------------------------------------------------------------------------------------------
		keepalive mechanism 			unreliable out-of-band 		reliable and simple in-band keepalive
										ldp keepalive; requires
										new protocol extensions
										for reliable connectivity
										report							
		-----------------------------------------------------------------------------------------------------
		advanced services 				traffic engineering, qos 	ipsec, ip diffserv, path
										guarantee, fast rerouting 	mtu discovery, ip fragmentation

		-----------------------------------------------------------------------------------------------------

		interoperability				wide vendor and carrier 	limited vendor and carrier support
										support, good and
										improving interoperability
		-----------------------------------------------------------------------------------------------------
****************************
lan protocols
	802.2 > llc / dsap and ssap used for data type
		src address
		dst address
		length
		dsap (0xaa) 
		ssap (0xaa)  
		ctrl
		data
		fcs

	802.3 > snap will be used for data type and dtetc more data types  
		src address
		dst address
		length
		dsap (0xaa)
		ssap (0xaa)
		ctrl
		snap (2byte)(data type)
		data
		fcs

	metro ethernet types
		distributed networks sites behave like l2 like arp using

		types
			connectivity type
				point to point
					permanent virtual circuite (pvc)

				multipoint
					many sites like cloud

			service type 
				wire service
				relay
					vlan
					or base on prt relaye like framerelay without vlan tags

			ptp
				ers (vlan)(ethernet relay service)
					vlan multiplexed poin to point service

				ews (wire) (ethernet wired service)
					non multiplexed point to point service
				----------
				one site
				atom and l2tpv3

			ptmp
				erms (vlan)(ethernet relay multipoint service)
					vlan multiplexed point to cloud service
					vpls

				ems (wire) (ethernet multipoint service)
					non multiplexed private to cloud service
					vpls
				----------
				many sites
				vpls

			ethernet private line (epl)
				like ews but on layer 1 conenctivities (oxcs)

			layer 2 vpn

			metro ethernet
			service 				architecture 	service definition 					connectivity
			-------------------------------------------------------------------------------------------											 
			epl						layer 1 		transparent(nonmultiplexed) 		ptp
			-------------------------------------------------------------------------------------------
			ews						vpwsi			transparent(nonmultiplexed)			ptp
			-------------------------------------------------------------------------------------------
			ers 					vpws 			multiplexed 						ptp
			-------------------------------------------------------------------------------------------						 
			ems 					vpls 			transparent(nonmultiplexed) 		multipoint to multipoint
			-------------------------------------------------------------------------------------------
			erms 					vpls 			multiplexed 						multipoint to multipoint
			-------------------------------------------------------------------------------------------
			atm/frame relay 		vpws 			multiplexed 						point to multipoint
			ethernet interworking
			-------------------------------------------------------------------------------------------

		use facing pe (upe)
			interface on ce side

		network facing pe (npe)
			interface on isp side

		metro ethernet   			epl 			ews 		erms			 	ems 		erms 			atm/frame relay ethernet interworking
		service 					
		----------------------------------------------------------------------------------------------------------------------------------------------
		u-pe <- > customer 			qinq 			qinq 		802.1q trunk 		qinq 		802.1q trunk 	frame relay or atm
		----------------------------------------------------------------------------------------------------------------------------------------------
		n-pe <- >  					wdmi21  		eompls141 	eompls 				ethernet  	eompls 			eompls
									wavelength										or eompls
		service provider 			sonet/sdhi1  			
									circuites
		----------------------------------------------------------------------------------------------------------------------------------------------

		(isp vlan q (service vlan q)) >>> qinq 

		trunk port and tunnel port (isp vlan tag for each customer)

		each vlan tag by isp can manage 4096 on one vlan tag 
			isp manage 4096 vlan tag each tag manage 4096 vlan > 4096*4096

			da , sa , lenght type , data , fcs (original frame)

			da , sa , e type , tag , lenght type , data , fcs (tagged frame from ce)

			da , sa , e type , tag , e type , tag , lenght type , data , fcs (double tagged frame in trunk links in sp network)
			

		has no qinq cause in isp we have no layer 2 ethernet (rarely see this type)

		our spanning tree , cdp , pagp , udld , vtp get trouble in transmission?
			make them relay

			dtp is not supported

			l2protocol-tunnel cdp
			l2protocol-tunnel stp
			l2protocol-tunnel vtp
			l2protocol-tunnel pagp
			l2protocol-tunnel udld

			*classificatin just worked on layer 2 and must enable layer 2 protocols and enable jumbo frames also on isp
			enable root gaurd on ce and ce must received bpdu and control inside the isp with bpdu filter to prevent sending pbdu pacekts to ce from pe
			on isp better don't enable root bridge
****************************
wan technologies
	hdlc header
		flag
			1 octets

		address
			1 octets

		controll
			1 octets

		protocol (cisco only)
			2 octets
			set payload 
			standard ethertype and cisco additional attributes

		information
			necessary

		fcs
			4 octets or 2

		flag
			1 octets

		same parts don't send again but towaard ce must put again


		0x7f > abort
		0xff > idle
		0x0f > unicast
		0x00 > broadcast
		0x20 > compressed frame
		0x40 > padded frame

	ppp features:
		protocol multiplexing
			support multiple higher level protocols by protocol field
		
		error detection
		
		network layer address negotiation
			dynamic learning of network layer addresses
			dialup
		
		error correction
		
		flow control
		
		sequencing

		ppp components:
			encapsulation method
				based on hdlc
			
			link control protocol (lcp)
				link establishment phase
				allows for establishment of link connectivity and dynamic configuration and testing of data link connection
			
			authentication phase optional
				pap or chap authentication
			
			network control protocol (ncp)
				an ncp is defined for each upper layer network protocol to allow for dynamic negotiation of its properties
			
				ipcp negotiates parameters such as ip address assignment
				
				ipxcp (ipx) and atcp (appletalk)

		flags 
			like hdlc

		0xff > address

		0x03 > controll
			acfc in control used for copression
			pfc in protocols make compression over lcp and in isp is invisible, has 2byte converting to 1 byte

	framerelay
		works with these
			link management interface protocols
				imi 
				simulate between isp routers and framerelay swithces for specific dlci transmission 

				flag 
					1 octet 
					0x7e

				address field
					dlci(addressing on framerelay) cr(command response) ea
					dlci fe(forward explicit congest notification) be(backward explicit congest notification) de(discard eligibility) ea (extended address)

				control ui
					1 octets fixed
					0x03 > lmi

				protocol
					1 octets
					0x09
				
				call reference
					1 octets
					0x00

				message type
					1 octets

					0x75 > status enquiry, dte periodically sends this
					0x7d > dce end response with this as status
					0x7b as update status

				information element
					ie identifier
						1octets
						one bit

					length 
						1octets
						one bit

					ie data

				additional information element
					information element

				fcs 
					2 or 4 octet

				flag 
					1 octet 
					0x7e

			encapsulation
			management traffic

		frame structure (has no protocol filed)
			flag 
				1 octet 
				0x7e

			address field
				dlci(addressing on framerelay) cr(command response) ea
				dlci fe(forward explicit congest notification) be(backward explicit congest notification) de(discard eligibility) ea (extended address)

			information

			fcs 
				1 octet

			flag 
				1 octet 
				0x7e

		 ietf 
		 	use nldp (snap + 0x80) to set protocols type 

		 	flag 
		 		1 octet 
		 		0x7e

		 	address field
		 		dlci(addressing on framerelay) cr(command response) ea
		 		dlci fe(forward explicit congest notification) be(backward explicit congest notification) de(discard eligibility) ea (extended address)

		 	control ui
		 		1 octets fixed
		 		0x03

		 	padding 
		 		1 octets and optional
		 		0x00
		 	
		 	nlpid
		 		1 octets

		 	information

		 	fcs 
		 		1 octet

		 	flag 
		 		1 octet 
		 		0x7e
		 	--------------------------------
		 	flag 
		 		1 octet 
		 		0x7e

		 	address field
		 		dlci(addressing on framerelay) cr(command response) ea
		 		dlci fe(forward explicit congest notification) be(backward explicit congest notification) de(discard eligibility) ea (extended address)

		 	control ui
		 		1 octets fixed
		 		0x03

		 	padding 
		 		1 octets and optional
		 		0x00
		 	
		 	nlpid
		 		1 octets

		 	snap header (set protocol type)
		 		oui > organizationlly unique identifier
		 			3 octets

		 		protocol identifier
		 			1 octets

		 	data

		 	flag 
		 		1 octet 
		 		0x7e

		 cisco
		 	flag 
		 		1 octet 
		 		0x7e

		 	address field
		 		dlci(addressing on framerelay) cr(command response) ea
		 		dlci fe(forward explicit congest notification) be(backward explicit congest notification) de(discard eligibility) ea (extended address)

		 	control ui
		 		1 octets fixed
		 		0x03

		 	protocol
		 		2octets

		 	information

		 	fcs 
		 		1 octet

		 	flag 
		 		1 octet 
		 		0x7e

	atm
		high-speed switching solution

		handle a variety of traffic types, bursty data services and delay/jitter-sensitive voice
		
		utilizes fixed-length cells to transport data
		
		traffic is carried on logical circuites identified by vpi/vci fields in header of each cell

		well-developed qos support because of its strict traffic class definitions

		protocol stack
			higher layer protocols

			atm adaptation layer
				cs sublayer
					converges sublayer

					important part of layers need to forward
					protocol data unit converting

					based on type forwarding and payloads has 5 style aal forwarding
					here set which virtual circuite wanna receive them  

					atm adaptation layer
						aal1
							carry connection-oriented, constant bit rate traffic with specific timing requirements
							typical aal1 traffic is ds-1 and e-1 circuites across an atm core

						aal2
							payloads with timing requirements but bursty traffic patterns
							compressed voice and video are examples of aal2 traffic
						
						aal3/4
							connection-oriented and connectionless variable bit rate traffic
						
						aal5
							connection-oriented and connectionless variable bit rate traffic
							simpler and more efficient than aal3/4

				sar sublayer
					segmentation and reassemble sublayer
					48 byte segmentation

			atm layer
				4 byte
			
			physical layer
				1 byte
				tc sublayer

				pmd sublayer
****************************
atom
	between 2 pseudowire has unidirectional lsp for vc label for inside label and managed by targeted ldp
	our ldp will be on mpls and between pe routers of isp and mpls

	after ce tunring on in pe we see a pseudo id and pseudo label  on random
	then send labels to another pe router
	on aal 5 must set control world for atm
	layer 2 might be optional
	if one peer want and another don't want use this could not negotiate

	pseudo forward the hdlc , hdlc , control , protocol (transparent) 
	ppp could not transmit fcs and address and  control, field pfc will be transparent 
	in ethernet omit preeamble and fcs then forward
		raw mode (port mode)
			valn tag is not process or important

		tagged mode
			isp must process or rewrite tags

	framerelay don't send dlci just forward control world
****************************
eompls
	ce and pe type must be same if :
		router
			7600
			12000

			port
				base on ingress port select virtual circuite

			vlan
				base on ingress vlan select virtual circuite

		vlan rewrite
			help us to map the vlans

		switch
			6500

			port
			vlan

					mpls inside isp 			192.168.2.0/24
				*********************	/////////////////
	ce1 ------- pe1 ------- p ------- pe2 ---------- ce2
				*********************
					10.10.1.0/24
					10.10.2.0/24

	on isp
		p
			interface gig0/0
			no shut
			ip address 10.1.1.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 10.1.2.1 255.255.255.0

			interface loop 0
			ip address 10.10.10.10 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0
			interface gig0/0
			mpls ip
			interface gig0/1
			mpls ip

			router ospf 1
			network 10.0.0.0 0.255.255.255 area 0

		p1
			interface gig0/0
			no shut
			ip address 10.1.1.2 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 192.168.1.1 255.255.255.0
			!ce1 gateway

			interface loop 0
			ip address 10.10.10.1 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0
			interface gig0/0
			mpls ip

			router ospf 1
			network 10.0.0.0 0.255.255.255 area 0

			xconnect 10.10.10.2 100 encapsulation mpls
			
			show mpls l2transport vc
			show mpls l2transport vc 100 details
			
			do show arp
			
			clear arp-cache

		p2
			interface gig0/0
			no shut
			ip address 10.1.2.2 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 192.168.1.1 255.255.255.0
			!ce2 gateway

			interface loop 0
			ip address 10.10.10.1 255.255.255.255
	
			mpls label protocol ldp
			mpls ldp router-id loop 0
			interface gig0/0
			mpls ip

			router ospf 1
			network 10.0.0.0 0.255.255.255 area 0

			xconnect 10.10.10.1 100 encapsulation mpls

			show mpls l2transport vc
			show mpls l2transport vc 100 details
			
			do show arp
			
			clear arp-cache

		ce1
			interface gig0/0
			no shut
			ip address 192.168.1.2.255.255.255.0

		ce2
			interface gig0/0
			no shut
			ip address 192.168.1.2.255.255.255.0

	eompls label stack
		tunnel header (4byte)(outer) (lse)
			tunnel labels
			exp > 0
			ttl

		vc header(4byte) (inner)(lse)
			vc label
			exp > 1
			ttl

		control word (4byte)
			0000

		reserved
		sequence number
		ethernet frame

		l2 header is 14b

		l2h is 18 byte if dot1q is tagged
		mtu configuration of the mpls network should accommodate this
		
		18 + 4 + 4 + 4 = 30 worst

		14 + 4 + 4 = 22 best

		mtu must be 1470 or 1530 (usually use 1530)

	router to router port base
		ce1
			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.1.2 255.255.255.0

		ce2
			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.1.1 255.255.255.0

		pe
			interface gig0/0.100
			encapsulation dot1q 100
			xconnect 10.10.10.1 100 encap mpls
			!100 is virtual circuite id and mpls is targeted ldp

			mpls l2transport route 10.10.10.2 100

			show mpls l2transport vc
			show mpls l2transport vc 100 details
			
			do show arp
			
			clear arp-cache

	router to router vlan rewrite
		ce1
			interface gig0/0.200
			encapsulation dot1q 200
			ip address 192.168.1.2 255.255.255.0

		ce2
			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.1.1 255.255.255.0

		pe1
			interface gig0/0.200
			encapsulation dot1q 200
			xconnect 10.10.10.2 100 encap mpls
			
			mpls l2transport route 10.10.10.2 100

		pe2
			interface gig0/0.100
			encapsulation dot1q 100
			xconnect 10.10.10.1 100 encap mpls

	switch to switch vlan base
		sup 7200
			interface gig2/4
			no ip address

			interface giga2/4.1
			encapsulation dot1q 100
			xconnect 192.168.1.102 100 encapsulation mpls

		sup 2
			vlan 100
			interface gig1/4
			switchport
			switchport trunk encapsulation dotiq
			switchport trunk allowed vlan 100
			switchport mode trunk

			interface vlan 100
			mpls l2transport route 192.168.1.103 100

	switch to switch port base
		on one peer
			interface gig2/4
			xconnect 192.168.1.102 100 encapsulation mpls

		on another peer
			vlan 100
			interface gig1/4
			switchport
			switchport trunk encapsulation dotiq
			switchport trunk allowed vlan 100
			switchport mode dot1q-tunnel
			spanning-tree bdpufilter enable

			interface vlan 100
			mpls l2transport route 192.168.1.103 100
****************************
wan protocol over mpls
	wan protocol over mpls
		tunnel header (4byte)(outer) (lse)
			tunnel labels
			exp > 0
			ttl

		vc header(4byte) (inner)(lse)
			vc label
			exp > 1
			ttl

		control word (4byte)
			used for reassemble packets on peer

			0000
			--------------
			ppp and hdlc
				0000

				0000
				
				be 
				
				length

				sequence number
			--------------
			frompls one to one mapping
				res
				f > fecn
				b > becn
				d > discard eligibility
				c > command and response
				b
				e
				length
				sequence number
			--------------
			aal5 pcs sdu encapsulation
				0000
				t > transport any type admin cell or aal5 payload
				e > efci
				c > clp
				u > command and response
				reserved
				length
				sequence number
			--------------
			atm n to one cell mode


			*length, sequence number , b , e is same on many parts
			*all encapsulation 4 first bits are same and 0x4 means ipv4 and 0x6 is ipv6
			*b and e means fragmentation information
			*clp is same as de

		reserved
		sequence number
		ethernet frame

	mtu
		mpls overhead
			2* 4 byte > 8 byte

		atom overhead
			4 byte

		totaly use 12 byte

		transport overhead
			frame relay, cisco encapsulation
				type [2 byte]
			
			frame relay, ietf encapsulation
				snap=> control [1] + pad [1] + nlpid [1] + oui [3] + ethertype [2] = 2-8 byte
			
			cisco hdlc
				address [1] + control [1] + ethertype [2] = 4 byte

			ppp
				ppp protocol [2 byte]
	
			aal5
				header (0-32 byte)

		core mtu >= edge mtu + transport header + atom header + (mpls label stack * mpls header size)

	hdlc over mpls
		flag
			0x7e

		transported in atom

			addr
			ctrl
				0x00

			protocol
				etype

			data

		fcs
		flag
			0x7e

		on isp
			p
				interface gig0/0
				no shut
				ip address 10.1.1.1 255.255.255.0
				
				interface gig0/1
				no shut
				ip address 10.1.2.1 255.255.255.0

				interface loop 0
				ip address 10.10.10.10 255.255.255.255

				mpls label protocol ldp
				mpls ldp router-id loop 0
				interface gig0/0
				mpls ip
				interface gig0/1
				mpls ip

				router ospf 1
				network 10.0.0.0 0.255.255.255 area 0

			p1
				interface gig0/0
				no shut
				ip address 10.1.1.2 255.255.255.0
				
				interface gig0/1
				no shut
				ip address 192.168.1.1 255.255.255.0
				!ce1 gateway

				interface loop 0
				ip address 10.10.10.1 255.255.255.255

				mpls label protocol ldp
				mpls ldp router-id loop 0
				interface gig0/0
				mpls ip

				router ospf 1
				network 10.0.0.0 0.255.255.255 area 0

				xconnect 10.10.10.2 100 encapsulation mpls
				
				show mpls l2transport vc
				show mpls l2transport vc 100 details

				show mpls l2transport binding
				
				do show arp
				
				clear arp-cache
				-----------------
				we can make pseudowire class
					pseudowire-class atom-hdlc
					sequence transmit
					encapsulation mpls
					interface gig0/0
					xconnect 10.10.10.1 100 pw-class atom-hdlc

			p2
				interface gig0/0
				no shut
				ip address 10.1.2.2 255.255.255.0
				
				interface gig0/1
				no shut
				ip address 192.168.1.1 255.255.255.0
				!ce2 gateway

				interface loop 0
				ip address 10.10.10.1 255.255.255.255
			
				mpls label protocol ldp
				mpls ldp router-id loop 0
				interface gig0/0
				mpls ip

				router ospf 1
				network 10.0.0.0 0.255.255.255 area 0

				xconnect 10.10.10.1 100 encapsulation mpls

			ce1
				interface gig0/0
				no shut
				ip address 192.168.1.2.255.255.255.0

			ce2
				interface gig0/0
				no shut
				ip address 192.168.1.2.255.255.255.0

	ppp over mpls
		flag
		address
		control
		transported in atom
			protocol
			payload + padding

		checksum
		flag

		*address and control field compression (acfc) will not work 
		pfc, however will work
		in pppompls the ppp negotiation takes place directly between ce devices

	framerelay over mpls
		frompls is not transparent because lmi runs between pe and ce devices
		c/r, fecn, becn, and de are sent on the control word flags it requires control word
		dlci pes do not exchange dlci
		it is a local pe responsibility to map vc to dlci

		p
			interface gig0/0
			no shut
			ip address 10.1.1.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 10.1.2.1 255.255.255.0

			interface loop 0
			ip address 10.10.10.10 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0
			interface gig0/0
			mpls ip
			interface gig0/1
			mpls ip

			router ospf 1
			network 10.0.0.0 0.255.255.255 area 0

		*create ospf os fabric

		pe
			frame-relay switching
			
			interface gig0/0
			encapsulation frame-relay
			-------------------------
			encapsulation frame-relay ietf

			frame-relay intf-type dce
			exit

			connect x gig0/0 100 l2transport
			!frame relay dlci tag

			xconnect 10.10.10.1 100 encap mpls
			!virtual circuite id this is important

			show connection all
			show connection id 1

		ce
			interface ser 0/0
			encapsulation frame-relay

			interface serial0/0.100 point-to-point
			encapsulation frame-relay
			ip address 192.168.1.1 255.255.255.0
			frame-relay interface-dlci 100
			------------------------------
			frame-relay interface-dlci 100 class myfrpve
			-------------------------------------------
			frame-relay interface-dlci 100 ietf class myfrpve

			map-class frame-relay myfrpve
			frame-relay cir 64000
			frame-relay bc 8000
			frame-relay be 0
			frame-relay mincir 32000

			show frame-relay lmi
			show frame-relay pvc

		*pseudowire atom doesn't support multipoint

		*c7200 vxr series router with two atm port adapters (pa)
			pa-a1 in slot 3 that does not support atom (gns3)
			pa-a3 version 2.0 in slot 4 that does support atom 

		show mpls l2transport hw-compatibility interface atm 0/0 

	atm over mpls
		atm aal5 over mpls
			transport of aal5 sdus over mpls

			virtual path identifier (vpi) and virtual circuite identifier (vci)
				do not exchange vpi and vci. vpi and vci values are kept in the	pseudowire
			
			payload type identifier (pti) contains three fields
				c
					t-bit in the aal5 control word
			
				efci
					the efci is transported in the e-bit in the control word
			
				end of packet (eop)
					you do not need this because cells are reassembled in ingress pe

			clp
				the clp-bit is carried in the c-bit in the control word

			pe (vc mode)
				interface atm0/0
				interface atm0/0.100 point-to-point
				pvc 1/100 l2transport
				encapsulation aal5snap
				!aal5
				--------------------
				encapsulation aal0
				!cell relay toward pe

				xconnect 10.10.10.1 100 encap mpls

				show mpls l2transport vc
				show mpls forwarding-table
				show mpls l2transport vc 100 details

			ce
				interface atm0/0.100 point-to-point
				ip address 192.168.1.1 255.255.255.0
				pvc 1/100
				encapsulation aal5snap

			*vp mode command will be under [atm pvp l2transport]


		atm cell over mpls
			relay of atm cells over mpls
			
			port mode
				transport of atm cells from an atm interface
			
			vp mode
				transport of atm cells from an atm virtual path (vp)
			
			vc mode
				transport of atm cells from an atm virtual circuite (vc)

		*maximum atom packet = (52 * max number of packed cells) + atom header + (depth of mpls label stack * mpls header size) 

		atm tansport overheads
			aal5 > 48 byte
			single cell relay > 112 byte
			package cell relay > 108 byte
				packed cell relay over mpls
					specifying three timers for the length of time that a pe router can wait for cells to be concatenated into the same mpls packet. the maximum cell packing timeout (mcpt) configuration is performed at the atm interface level

					specifying the maximum number of cells to be concatenated in an mpls packet and the timer that is available for use
					you perform this step at the i2transport pvc configuration

					interface atm0/0
					atm mcpt-timers 500 800 4095
					!microsecond
					-----------------------------
					interface atm0/0.100 point-to-point
					cell-packing 10 mcpt-timers 2

					show mpls l2transport binding 100
					!show us bindin
****************************
atom advance features
	load sharing
		 packets being transmitted over a given pseudowire follow the same path

		 packets of different pseudowires are spread across all available equal-cost paths

		 utilizes a hashing algorithm to assign pseudowires to equal-cost paths

		 By default, remote VC label of a pseudowire acts as the hash key to calculate the outgoing path (might have many path on one outgoing interface cause we have hash on interface)

		 	pseudowire packets that have the same remote VC label are sent through the same path
		 	minimize out-of-order packets in the same data flow


		 ce1 --------- pe1 ---------------- p1 ---------------- pe2 ------------ ce2
		 				|										|
		 				|										|
		 				|				osfp area 0				|
		 				| 					mpls				|
		 				p2--------------------------------------p3

		 if igp metric were same for above and below links can make load share for atom
		 this loadshare depends on remote vs hash

		 p1
		 	interface gig0/0
		 	no shut
		 	ip address 10.1.1.1 255.255.255.0
		 	
		 	interface gig0/1
		 	no shut
		 	ip address 10.1.2.1 255.255.255.0

		 	interface gig0/2
		 	no shut
		 	ip address 10.1.5.1 255.255.255.0

		 	interface loop 0
		 	ip address 10.10.10.1 255.255.255.255

		 	mpls label protocol ldp
		 	mpls ldp router-id loop 0
		 	interface gig0/0
		 	mpls ip
		 	interface gig0/1
		 	mpls ip
		 	interface gig0/2
		 	mpls ip

		 	router ospf 1
		 	network 10.0.0.0 0.255.255.255 area 0

		 p2
		 	interface gig0/0
		 	no shut
		 	ip address 10.1.3.1 255.255.255.0
		 	
		 	interface gig0/1
		 	no shut
		 	ip address 10.1.5.1 255.255.255.0

		 	interface loop 0
		 	ip address 10.10.10.2 255.255.255.255

		 	mpls label protocol ldp
		 	mpls ldp router-id loop 0
		 	interface gig0/0
		 	mpls ip
		 	interface gig0/1
		 	mpls ip

		 	router ospf 1
		 	network 10.0.0.0 0.255.255.255 area 0

		 	*here must check ad and metric of pe2 on show ip route then set metric on ospf
		 		just need change metric connection between pe1 and p2

		 		interface gig0/1
		 		ip ospf cost 63

		 p3
		 	interface gig0/0
		 	no shut
		 	ip address 10.1.4.1 255.255.255.0
		 	
		 	interface gig0/1
		 	no shut
		 	ip address 10.1.5.1 255.255.255.0

		 	interface loop 0
		 	ip address 10.10.10.3 255.255.255.255

		 	mpls label protocol ldp
		 	mpls ldp router-id loop 0
		 	interface gig0/0
		 	mpls ip
		 	interface gig0/1
		 	mpls ip

		 	router ospf 1
		 	network 10.0.0.0 0.255.255.255 area 0

		 	*here must check ad and metric of pe1 on show ip route then set metric on ospf
		 		just need change metric connection between pe2 and 3

		 		interface gig0/1
		 		ip ospf cost 63

		 pe1
		 	ip cef
		 	mpls label protocol 1dp
		 	mpls ldp router-id Loop 0

		 	interface gig0/0
		 	no shut
		 	!ce side
		 	
		 	interface gig0/1
		 	no shut
		 	ip address 10.1.1.2 255.255.255.0

		 	interface gig0/2
		 	no shut
		 	ip address 10.1.3.2 255.255.255.0

		 	interface loop 0
		 	ip address 10.10.10.10 255.255.255.255

		 	mpls label protocol ldp
		 	mpls ldp router-id loop 0
		 	interface gig0/0
		 	mpls ip
		 	interface gig0/1
		 	mpls ip
		 	interface gig0/2
		 	mpls ip

		 	router ospf 1
		 	network 10.0.0.0 0.255.255.255 area 0

		 	interface gig0/0.100
		 	encapaulation dot1Q 100
		 	xconnect 10.10.10.20  101 encapsulation mpls

		 	interface gig0/0.200
		 	encapsulation dot1Q 200
		 	xconnect 10.10.10.20 102 encapaulation mpls

		 	interface gig0/0.300
		 	encapsulation dot1Q 300
		 	xconnect 10.10.10.20 103 encapsulation mpls

		 	show ip route 10.1.1.2
		 	!show us metric 31 and reachability on interface0/1

		 	show mpls l2transport summary
		 	!here make pseudowire on one interface

		 	show mpls l2transport details
		 	!show us labels

		 	*here must check ad and metric of pe2 on show ip route then set metric on ospf
		 		just need change metric connection between pe1 and p2

		 		interface gig0/1
		 		ip ospf cost 63

		 pe2
		 	ip cef
		 	mpls label protocol 1dp
		 	mpls ldp router-id Loop 0

		 	interface gig0/0
		 	no shut
		 	!ce side
		 	
		 	interface gig0/1
		 	no shut
		 	ip address 10.1.4.2 255.255.255.0

		 	interface gig0/2
		 	no shut
		 	ip address 10.1.2.2 255.255.255.0

		 	interface loop 0
		 	ip address 10.10.10.20 255.255.255.255

		 	mpls label protocol ldp
		 	mpls ldp router-id loop 0
		 	interface gig0/0
		 	mpls ip
		 	interface gig0/1
		 	mpls ip
		 	interface gig0/2
		 	mpls ip

		 	router ospf 1
		 	network 10.0.0.0 0.255.255.255 area 0

		 	interface gig0/0.1
		 	encapaulation dot1Q 100
		 	xconnect 10.10.10.10 101 encapsulation mpls

		 	interface gig0/0.2
		 	encapsulation dot1Q 200
		 	xconnect 10.10.10.10 102 encapaulation mpls

		 	interface gig0/0.3
		 	encapsulation dot1Q 300
		 	xconnect 10.10.10.10 103 encapsulation mpls

		 	*here must check ad and metric of pe1 on show ip route then set metric on ospf
		 		just need change metric connection between pe2 and p3

		 		interface gig0/1
		 		ip ospf cost 63

		 ce1
		 	interface gig0/0
		 	no shut
		 	!pe1 side

		 	interface gig0/0.100
		 	encapsulation dot1q 100
		 	ip address 192.168.100.1 255.255.255.0

		 	interface gig0/0.200
		 	encapsulation dot1q 200
		 	ip address 192.168.200.1 255.255.255.0

		 	interface gig0/0.300
		 	encapsulation dot1q 300
		 	ip address 192.168.250.1 255.255.255.0

		 ce2
		 	interface gig0/0
		 	no shut
		 	!pe1 side

		 	interface gig0/0.100
		 	encapsulation dot1q 100
		 	ip address 192.168.100.2 255.255.255.0

		 	interface gig0/0.200
		 	encapsulation dot1q 200
		 	ip address 192.168.200.2 255.255.255.0

		 	interface gig0/0.300
		 	encapsulation dot1q 300
		 	ip address 192.168.250.2 255.255.255.0

		 *if change cost cause make loadshare depends on label for virtual circuite

	preferred path
		need ios 7200 adv ip service k9 smz 12233 r3 

		pseudowire path can use best path and many pseudowire on same path selction base on remote virtual circuite hash

		allows pseudowire data packets to flow through a different path from pseudowire control packets

		also possible to provide differentiated services to pseudowires with different forwarding requirements
			place pseudowires that carry voice traffic to a special traffic	engineered path with low latency and jitter
			place pseudowires that remotely back up a large amount of data for file servers to a best-effort path that allows high bursts.


		ce1 --------- pe1 ---------------- p1 ---------------- pe2 ------------ ce2
						|										|
						|										|
						|				osfp area 0				|
						| 					mpls				|
						p2--------------------------------------p3

		solutions
			ip routing
				make loopbacks on pe routers and set static route for them 

				p1
					interface gig0/0
					no shut
					ip address 10.1.1.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.2.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.1 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0
					interface gig0/0
					mpls ip
					interface gig0/1
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

				p2
					interface gig0/0
					no shut
					ip address 10.1.3.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.5.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.2 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0
					interface gig0/0
					mpls ip
					interface gig0/1
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

				p3
					interface gig0/0
					no shut
					ip address 10.1.3.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.5.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.3 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0
					interface gig0/0
					mpls ip
					interface gig0/1
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0
						
				pe1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
					no shut
					ip address 10.1.1.2 255.255.255.0

					interface gig0/2
					no shut
					ip address 10.1.3.2 255.255.255.0

					interface loop 0
					ip address 10.10.10.10 255.255.255.255

					interface loop 1
					ip address 10.20.20.10 255.255.255.255
					ip address 10.20.20.11 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0
					interface gig0/0
					mpls ip
					interface gig0/1
					mpls ip
					interface gig0/2
					mpls ip

					ip route 10.20.20.10 255.255.255.255 gig0/0
					ip route 10.20.20.11 255.255.255.255 gig0/1

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class up
					encapsulation mpls
					preferred-path peer 10.20.20.20

					pseudowire-class down
					encapsulation mpls
					preferred-path peer 10.20.20.21

					interface gig0/0.100
					encapaulation dot1Q 100
					xconnect 10.10.10.20 101 pseudowire-class up

					interface gig0/0.200
					encapsulation dot1Q 200
					xconnect 10.10.10.20 102 pseudowire-class down

					show mpls l2transport summary

				pe2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
					no shut
					ip address 10.1.4.2 255.255.255.0

					interface gig0/2
					no shut
					ip address 10.1.2.2 255.255.255.0

					interface loop 0
					ip address 10.10.10.20 255.255.255.255

					interface loop 1
					ip address 10.20.20.20 255.255.255.255
					ip address 10.20.20.21 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0
					interface gig0/0
					mpls ip
					interface gig0/1
					mpls ip
					interface gig0/2
					mpls ip

					ip route 10.20.20.20 255.255.255.255 gig0/0
					ip route 10.20.20.21 255.255.255.255 gig0/1

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class up
					encapsulation mpls
					preferred-path peer 10.20.20.10

					pseudowire-class down
					encapsulation mpls
					preferred-path peer 10.20.20.11

					interface gig0/0.1
					encapaulation dot1Q 100
					xconnect 10.10.10.10 101 pseudowire-class down

					interface gig0/0.2
					encapsulation dot1Q 200
					xconnect 10.10.10.10 102  pseudowire-class up

				ce1
					interface gig0/0
					no shut
					!pe1 side

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.100.1 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.200.1 255.255.255.0

				ce2
					interface gig0/0
					no shut
					!pe1 side

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.100.2 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.200.2 255.255.255.0

			mpls traffic engineering
				must disable fallback on preferred-path (means if were not reachabile don't switch to another path)
				we can set bandwidth on some attributes which we need them

				p1
					interface gig0/0
					no shut
					ip address 10.1.1.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.2.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.1 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0

					interface gig0/0
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/1
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					router ospf 1
					mpls traffic-eng router-id loop 0
					mpls traffic-eng area 0
					network 10.0.0.0 0.255.255.255 area 0

				p2
					interface gig0/0
					no shut
					ip address 10.1.3.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.5.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.2 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0

					interface gig0/0
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/1
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					router ospf 1
					mpls traffic-eng router-id loop 0
					mpls traffic-eng area 0
					network 10.0.0.0 0.255.255.255 area 0

				p3
					interface gig0/0
					no shut
					ip address 10.1.3.1 255.255.255.0
					
					interface gig0/1
					no shut
					ip address 10.1.5.1 255.255.255.0

					interface loop 0
					ip address 10.10.10.3 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0

					interface gig0/0
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/1
					mpls ip
					mpls ipmpls traffic-eng tunnels
					ip rsvp bandwidth 

					router ospf 1
					mpls traffic-eng router-id loop 0
					mpls traffic-eng area 0
					network 10.0.0.0 0.255.255.255 area 0
						
				pe1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0
					mpls traffic-eng tunnels

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
					no shut
					ip address 10.1.1.2 255.255.255.0

					interface gig0/2
					no shut
					ip address 10.1.3.2 255.255.255.0

					interface loop 0
					ip address 10.10.10.10 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0

					interface gig0/0
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/1
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/2
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					pseudowire-class down
					encapsulation mpls
					preferred-path interface tunnel 0 disable-fallback

					pseudowire-class up
					encapsulation mpls
					preferred-path interface tunne1 1

					router ospf 1
					mpls traffic-eng router-id loop 0
					mpls traffic-eng area 0
					network 10.0.0.0 0.255.255.255 area 0
					
					interface tunnel 0
					ip unnumbered loop 0
					tunnel destination 10.10.10.20
					tunnel mode mpls traffic-eng
					tunnel mpls traffic-eng path-option 10 explicit name up
					
					interface tunnel 1
					ip unnumbered loop 0
					tunnel destination 10.10.10.20
					tunnel mode mpls traffic-eng
					tunnel mpls traffic-eng path-option 10 explicit name down

					ip explicit-path name up enable
					next-address 10.1.1.2
					next-address 10.1.2.2
					next-address 10.10.10.20

					ip explicit-path name down enable
					next-address 10.1.3.2
					next-address 10.1.5.2
					next-address 10.1.4.1
					next-address 10.10.10.10

					show mpls traffic-eng tunnels tun 0
					show mpls traffic-eng tunnels brief

					interface gig0/0.100
					encapaulation dot1Q 100
					xconnect 10.10.10.20 101 encap mpls

					interface gig0/0.200
					encapsulation dot1Q 200
					xconnect 10.10.10.20 102 pw-class up

					interface gig0/0.300
					encapsulation dot1Q 300
					xconnect 10.10.10.20 103 pw-class down

				pe2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0
					mpls traffic-eng tunnels

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
					no shut
					ip address 10.1.4.2 255.255.255.0

					interface gig0/2
					no shut
					ip address 10.1.2.2 255.255.255.0

					interface loop 0
					ip address 10.10.10.20 255.255.255.255

					mpls label protocol ldp
					mpls ldp router-id loop 0

					interface gig0/0
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/1
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					interface gig0/2
					mpls ip
					mpls traffic-eng tunnels
					ip rsvp bandwidth 

					pseudowire-class down
					encapsulation mpls
					preferred-path interface tunnel 0 disable-fallback

					pseudowire-class up
					encapsulation mpls
					preferred-path interface tunne1 1

					router ospf 1
					mpls traffic-eng router-id loop 0
					mpls traffic-eng area 0
					network 10.0.0.0 0.255.255.255 area 0

					interface tunnel 0
					ip unnumbered loop 0
					tunnel destination 10.10.10.10
					tunnel mode mpls traffic-eng
					tunnel mpls traffic-eng path-option 10 explicit name up
					
					interface tunnel 1
					ip unnumbered loop 0
					tunnel destination 10.10.10.10
					tunnel mode mpls traffic-eng
					tunnel mpls traffic-eng path-option 10 explicit name down

					ip explicit-path name up enable
					next-address 10.1.2.1
					next-address 10.1.1.1
					next-address 10.10.10.10

					ip explicit-path name down enable
					next-address 10.1.4.2
					next-address 10.1.5.1
					next-address 10.1.3.1
					next-address 10.10.10.10

					show mpls traffic-eng tunnels tun 0
					show mpls traffic-eng tunnels brief

					interface gig0/0.100
					encapaulation dot1Q 100
					xconnect 10.10.10.10 101 encap mpls

					interface gig0/0.200
					encapsulation dot1Q 200
					xconnect 10.10.10.10 102 pw-class up

					interface gig0/0.300
					encapsulation dot1Q 300
					xconnect 10.10.10.10 103 pw-class down

				ce1
					interface gig0/0
					no shut
					!pe1 side

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.100.1 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.200.1 255.255.255.0

				ce2
					interface gig0/0
					no shut
					!pe1 side

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.100.2 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.200.2 255.255.255.0

	atom pseudowires with mpls traffic engineering fast reroute
		each feature on mpls could be used on vpls

		on preferred-path we must have set path and dedicated bandwidth to use them
		we can make backup tunnel to make fast redundant tunnels
		like protection feature

		must define protection links and protection path and set protection on tunnels

		ce1 --------- pe1 ---------------- p1 ---------------- pe2 ------------ ce2
						|			*******						|
						|	      *								|
						|	   *	osfp area 0					|
						|   *			mpls				 	| 
						p2--------------------------------------p3

		p1
			interface gig0/0
			no shut
			ip address 10.1.1.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 10.1.2.1 255.255.255.0

			interface gig 0/2
			no shut
			ip address 10.1.6.1 255.255.255.0

			interface loop 0
			ip address 10.10.10.1 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface gig0/0
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/1
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/2
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			router ospf 1
			mpls traffic-eng router-id loop 0
			mpls traffic-eng area 0
			network 10.0.0.0 0.255.255.255 area 0

		p2
			interface gig0/0
			no shut
			ip address 10.1.3.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 10.1.5.1 255.255.255.0

			interface gig 0/2
			no shut
			ip address 10.1.6.2 255.255.255.0

			interface loop 0
			ip address 10.10.10.2 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface gig0/0
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/1
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/2
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			router ospf 1
			mpls traffic-eng router-id loop 0
			mpls traffic-eng area 0
			network 10.0.0.0 0.255.255.255 area 0

		p3
			interface gig0/0
			no shut
			ip address 10.1.3.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 10.1.5.1 255.255.255.0

			interface loop 0
			ip address 10.10.10.3 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface gig0/0
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/1
			mpls ip
			mpls ipmpls traffic-eng tunnels
			ip rsvp bandwidth 

			router ospf 1
			mpls traffic-eng router-id loop 0
			mpls traffic-eng area 0
			network 10.0.0.0 0.255.255.255 area 0
				
		pe1
			ip cef
			mpls label protocol 1dp
			mpls ldp router-id Loop 0
			mpls traffic-eng tunnels

			interface gig0/0
			no shut
			!ce side
			
			interface gig0/1
			no shut
			ip address 10.1.1.2 255.255.255.0

			interface gig0/2
			no shut
			ip address 10.1.3.2 255.255.255.0

			interface loop 0
			ip address 10.10.10.10 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface gig0/0
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/1
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 
			mpls traffic-eng backup-path tunnel 100

			interface gig0/2
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 

			pseudowire-class down
			encapsulation mpls
			preferred-path interface tunnel 0 disable-fallback

			pseudowire-class up
			encapsulation mpls
			preferred-path interface tunne1 1

			router ospf 1
			mpls traffic-eng router-id loop 0
			mpls traffic-eng area 0
			network 10.0.0.0 0.255.255.255 area 0
			
			interface tunnel 0
			ip unnumbered loop 0
			tunnel destination 10.10.10.20
			tunnel mode mpls traffic-eng
			tunnel mpls traffic-eng fast-reroute
			cunnel mpls traffic-eng path-option 10 explicit name up
			
			interface tunnel 1
			ip unnumbered loop 0
			tunnel destination 10.10.10.20
			tunnel mode mpls traffic-eng
			cunnel mpls traffic-eng path-option 10 explicit name down

			interface tunnel 100
			ip unnumbered loop 0
			tunnel destination 10.10.10.1
			tunnel mode mpls traffic-eng
			cunnel mpls traffic-eng path-option 10 explicit name backup-up
			
			ip explicit-path name up enable
			next-address 10.1.1.2
			next-address 10.1.2.2
			next-address 10.10.10.20

			ip explicit-path name down enable
			next-address 10.1.3.2
			next-address 10.1.5.2
			next-address 10.1.4.1
			next-address 10.10.10.10

			ip explicit-path name backup-up enable
			next-address 10.1.3.2
			next-address 10.1.6.1
			next-address 10.10.10.1

			show mpls traffic-eng tunnels tun 0
			show mpls traffic-eng tunnels brief
			show mpls traffic-eng tunnels protection
			show mpls l2transport vc 200 details
			show mpls traffic-eng fast-reroute

			interface gig0/0.100
			encapaulation dot1Q 100
			xconnect 10.10.10.20 101 encap mpls

			interface gig0/0.200
			encapsulation dot1Q 200
			xconnect 10.10.10.20 102 pw-class up

			interface gig0/0.300
			encapsulation dot1Q 300
			xconnect 10.10.10.20 103 pw-class down

		pe2
			ip cef
			mpls label protocol 1dp
			mpls ldp router-id Loop 0
			mpls traffic-eng tunnels

			interface gig0/0
			no shut
			!ce side
			
			interface gig0/1
			no shut
			ip address 10.1.4.2 255.255.255.0

			interface gig0/2
			no shut
			ip address 10.1.2.2 255.255.255.0

			interface loop 0
			ip address 10.10.10.20 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface gig0/0
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/1
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 

			interface gig0/2
			mpls ip
			mpls traffic-eng tunnels
			ip rsvp bandwidth 

			pseudowire-class down
			encapsulation mpls
			preferred-path interface tunnel 0 disable-fallback

			pseudowire-class up
			encapsulation mpls
			preferred-path interface tunne1 1

			router ospf 1
			mpls traffic-eng router-id loop 0
			mpls traffic-eng area 0
			network 10.0.0.0 0.255.255.255 area 0

			interface tunnel 0
			ip unnumbered loop 0
			tunnel destination 10.10.10.10
			tunnel mode mpls traffic-eng
			cunnel mpls traffic-eng path-option 10 explicit name up
			
			interface tunnel 1
			ip unnumbered loop 0
			tunnel destination 10.10.10.10
			tunnel mode mpls traffic-eng
			cunnel mpls traffic-eng path-option 10 explicit name down

			ip explicit-path name up enable
			next-address 10.1.2.1
			next-address 10.1.1.1
			next-address 10.10.10.10

			ip explicit-path name down enable
			next-address 10.1.4.2
			next-address 10.1.5.1
			next-address 10.1.3.1
			next-address 10.10.10.10

			interface gig0/0.100
			encapaulation dot1Q 100
			xconnect 10.10.10.10 101 encap mpls

			interface gig0/0.200
			encapsulation dot1Q 200
			xconnect 10.10.10.10 102 pw-class up

			interface gig0/0.300
			encapsulation dot1Q 300
			xconnect 10.10.10.10 103 pw-class down

		ce1
			interface gig0/0
			no shut
			!pe1 side

			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.100.1 255.255.255.0

			interface gig0/0.200
			encapsulation dot1q 200
			ip address 192.168.200.1 255.255.255.0

		ce2
			interface gig0/0
			no shut
			!pe1 side

			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.100.2 255.255.255.0

			interface gig0/0.200
			encapsulation dot1q 200
			ip address 192.168.200.2 255.255.255.0

	atom pseudowire over gre tunnel
		on network base ip we can not use atom 
		but gre help us to run them over
		cause we cave connected label we just see one another is pop

						mpls over gre tunnels
		////////////////////////////////////////////////////////////
		ce1 ------- pe1 ---------- p1 ---------- pe2 ----------- ce2
					******************************
							ip base network 

		p1(internet)
			interface gig0/0
			no shut
			ip address 12.1.1.1 255.255.255.0
			
			interface gig0/1
			no shut
			ip address 12.1.2.1 255.255.255.0

		pe1
			ip cef
			mpls label protocol 1dp
			mpls ldp router-id Loop 0

			interface gig0/0
			no shut
			!ce side
			
			interface gig0/1
			no shut
			ip address 12.1.1.2 255.255.255.0

			interface loop 0
			ip address 10.10.10.1 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface tunnel 0
			mpls ip

			ip route 0.0.0.0 0.0.0.0 12.1.1.1
			ip route 10.10.10.2 255.255.255.255 tun 0
			
			interface tunnel 0
			ip address 10.1.1.1 255.255.255.0
			tunnel source  gig0/1
			tunnel destination 12.1.2.2 

			show mpls traffic-eng tunnels tun 0
			show mpls traffic-eng tunnels brief
			show mpls traffic-eng tunnels protection
			show mpls l2transport vc 200 details
			show mpls traffic-eng fast-reroute

			interface gig0/0
			xconnect 10.10.10.20 101 encap mpls

		pe2
			ip cef
			mpls label protocol 1dp
			mpls ldp router-id Loop 0 

			interface gig0/0
			no shut
			!ce side
			
			interface gig0/1
			no shut
			ip address 12.1.2.2 255.255.255.0

			interface loop 0
			ip address 10.10.10.2 255.255.255.255

			mpls label protocol ldp
			mpls ldp router-id loop 0

			interface tunnel 0
			mpls ip 

			ip route 0.0.0.0 0.0.0.0 12.1.2.1
			ip route 10.10.10.1 255.255.255.255 tun 0

			interface tunnel 0
			ip address 10.1.1.2 255.255.255.0
			tunnel source gig0/1
			tunnel destination 12.1.1.2			

			interface gig0/0
			xconnect 10.10.10.10 101 encap mpls

		ce1
			interface gig0/0
			no shut
			!pe1 side

			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.100.1 255.255.255.0

		ce2
			interface gig0/0
			no shut
			!pe1 side

			interface gig0/0.100
			encapsulation dot1q 100
			ip address 192.168.100.2 255.255.255.0

	pseudowire emulation in multi-as networks
		each city tci has a direct connection on as-number to irans tci as-number

		inter as-number pseudowire menas 2 branch of market in different province has connection without routing on as-numbers

										as 200
										
		ce1 --------- pe3 ---------------- p4 ---------------- pe4 ------------ ce2
											|
											|
											|
										  asbr2
											|
											|
											|
										  asbr1
											|
											|
											|
		ce1 --------- pe1 ---------------- p1 ---------------- pe2 ------------ ce2
						|										|
						|										|
						|										|
						| 					as 100				|
						p2--------------------------------------p3


		has 3 model
			1- between 2 pe router make atom connection on 2 province

				pe1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 10.1.1.1 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 10.10.10.3 100 encap mpls

				pe2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 10.1.1.2 255.255.255.255

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 10.10.10.3 200 encap mpls

				pe3
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.1 255.255.255.255

					interface gig0/0.200
					encapsulation dot1q 200 
					xconnect 172.16.1.3 200 encap mpls

				pe4
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.2 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 172.16.1.3 100 encap mpls

				ce1
					interface gig0/0
					no shut

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.1.1 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.2.1 255.255.255.0

				ce2
					interface gig0/0
					no shut

					interface gig0/0.100
					encapsulation dot1q 100
					ip address 192.168.1.2 255.255.255.0

					interface gig0/0.200
					encapsulation dot1q 200
					ip address 192.168.2.2 255.255.255.0 

				asbr1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.1 255.255.255.0
					
					interface loop 0
					ip address 10.10.10.3 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 10.10.10.1 100 encap mpls

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 10.10.10.1 200 encap mpls

					interface gig0/1
					no shut
					ip address 10.43.11.2 255.255.255.0
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					router bgp 100
					no synchronization
					neighbor 172.16.100.2 remote-as 200
					no auto-summary

				asbr2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.2 255.255.255.0
					
					interface loop 0
					ip address 172.16.1.3 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect  172.16.1.2 100 encap mpls

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 172.16.1.1 200 encap mpls

					interface gig0/1
					no shut
					ip address 172.16.24.2 255.255.255.0
					mpls ip

					router ospf 1
					network 172.16.0.0 0.0.255.255 area 0

					router bgp 200
					no synchronization
					neighbor 172.16.100.1 remote-as 100
					no auto-summary

			---------------------------------------------------------------

			2- from each asbr on same as-number to pe router make pseudowire, between each asbr make attachment circuite, between each asbr 
				make subinterface for each pseudowire
				between as-number doesn't need routing on pe routers
				in this condition make interconnectivity between pe routers
				ebgp connection on asbr toward pe routers and redistribute into the igp
				*cause media is mpls and bgp has no ldp must use bgp to forward labels
					bgp send label

				advertise loop back interfaces into bgp and into the igp must get redistributed

				pe1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 10.1.1.1 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 172.16.1.2 100 encap mpls

				pe2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 10.1.1.2 255.255.255.255

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 172.16.1.1 200 encap mpls

				pe3
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.1 255.255.255.255

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 10.10.10.3 200 encap mpls

				pe4
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.2 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 10.10.10.1 100 encap mpls

				asbr1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.1 255.255.255.0
					
					interface loop 0
					ip address 10.10.10.3 255.255.255.255

					interface gig0/1
					no shut
					ip address 10.43.11.2 255.255.255.0
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0
					default-metric 20
					redistribute bgo 100 subnets

					router bgp 100
					neighbor 172.16.100.2 remote-as 200
					address-family ipv4
					neighbor 172.16.100.2 active
					neighbor 172.16.100.2 send-label
					no synchronization
					no auto-summary
					network 10.10.10.1 mask 255.255.255.255
					network 10.10.10.2 mask 255.255.255.255

				asbr2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.2 255.255.255.0
					
					interface loop 0
					ip address 172.16.1.3 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect  172.16.1.2 100 encap mpls

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 172.16.1.1 200 encap mpls

					interface gig0/1
					no shut
					ip address 172.16.24.2 255.255.255.0
					mpls ip
					
					router ospf 1
					network 172.16.0.0 0.0.255.255 area 0
					default-metric 20
					redistribute bgo 200 subnets

					router bgp 200
					neighbor 172.16.100.1 remote-as 100
					address-family ipv4
					neighbor 172.16.100.1 active
					neighbor 172.16.100.1 send-label
					no synchronization
					no auto-summary
					network 172.16.1.1 mask 255.255.255.255
					network 172.16.1.2 mask 255.255.255.255

			---------------------------------------------------------------

			3- between each asbr has ebgp , between asbr and pe router has ibgp
				benefits is load is on pe routers
				igp has no process on routing
				must set bgp send label for this

				pe1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side

					interface gig0/1
					no shut
					ip address 10.23.12.1 255.255.255.0
					mpls ip

					interface gig0/1
					no shut
					ip address 10.23.11.1 255.255.255.0
					mpls ip
					
					interface loop 0
					ip address 10.10.10.1 255.255.255.255

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 172.16.1.2 100 encap mpls

					router bgp 100
					neighbor 10.10.10.3 remote-as 100
					neighbor 10.10.10.3 update-source loop 0
					address-family ipv4
					neighbor 10.10.10.3 active
					neighbor 10.10.10.3 send-label


				pe2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side

					interface gig0/1
					no shut
					ip address 10.23.21.1 255.255.255.0
					mpls ip

					interface gig0/1
					no shut
					ip address 10.23.23.1 255.255.255.0
					mpls ip
					
					interface loop 0
					ip address 10.10.10.2 255.255.255.255

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					interface gig0/0.200
					encapsulation dot1q 200
					xconnect 172.16.1.1 200 encap mpls

					router bgp 100
					neighbor 10.10.10.3 remote-as 100
					neighbor 10.10.10.3 update-source loop 0
					address-family ipv4
					neighbor 10.10.10.3 active
					neighbor 10.10.10.3 send-label

				pe3
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.1 255.255.255.255

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 10.10.10.1 100 encap mpls

					interface gig0/1
					no shut
					ip address 172.16.54.1 255.255.255.0

					router ospf 1
					network 172.16.0.0 0.0.255.255 area 0

					router bgp 200
					neighbor 172.16.1.3 remote-as 200
					neighbor 172.16.1.3 update-source loop 0
					address-family ipv4
					neighbor 172.16.1.3 active
					neighbor 172.16.1.3 send-label

				pe4
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					!ce side
					
					interface loop 0
					ip address 172.16.1.2 255.255.255.255

					interface gig0/1
					no shut
					ip address 172.16.34.1 255.255.255.0

					interface gig0/0.100
					encapsulation dot1q 100
					xconnect 10.10.10.1 100 encap mpls

					router ospf 1
					network 172.16.0.0 0.0.255.255 area 0

					router bgp 200
					neighbor 172.16.1.3 remote-as 200
					neighbor 172.16.1.3 update-source loop 0
					address-family ipv4
					neighbor 172.16.1.3 active
					neighbor 172.16.1.3 send-label


				asbr1
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.1 255.255.255.0
					
					interface loop 0
					ip address 10.10.10.3 255.255.255.255

					interface gig0/1
					no shut
					ip address 10.43.11.2 255.255.255.0
					mpls ip

					router ospf 1
					network 10.0.0.0 0.255.255.255 area 0

					router bgp 100
					neighbor 172.16.100.2 remote-as 200
					neighbor 10.10.10.1 remote-as 100
					neighbor 10.10.10.1 update-source loop 0
					neighbor 10.10.10.2 remote-as 100
					neighbor 10.10.10.2 update-source loop 0
					address-family ipv4
					neighbor 172.16.100.2 active
					neighbor 172.16.100.2 send-label
					neighbor 10.10.10.2 active
					neighbor 10.10.10.2 send-label
					neighbor 10.10.10.2 next-hop-self
					neighbor 10.10.10.1 active
					neighbor 10.10.10.1 send-label
					neighbor 10.10.10.1 next-hop-self
					no synchronization
					no auto-summary
					network 10.10.10.1 mask 255.255.255.255
					network 10.10.10.2 mask 255.255.255.255

				asbr2
					ip cef
					mpls label protocol 1dp
					mpls ldp router-id Loop 0

					interface gig0/0
					no shut
					ip address 172.16.100.2 255.255.255.0
					
					interface loop 0
					ip address 172.16.1.3 255.255.255.255

					interface gig0/1
					no shut
					ip address 172.16.24.2 255.255.255.0
					mpls ip
					
					router ospf 1
					network 172.16.0.0 0.0.255.255 area 0

					router bgp 200
					neighbor 172.16.100.1 remote-as 100
					neighbor 172.16.1.2 remote-as 200
					neighbor 172.16.1.2 update-source loop 0
					neighbor 172.16.1.1 remote-as 200
					neighbor 172.16.1.1 update-source loop 0
					address-family ipv4
					neighbor 172.16.100.1 active
					neighbor 172.16.100.1 send-label
					neighbor 172.16.1.2 active
					neighbor 172.16.1.2 send-label
					neighbor 172.16.1.2 next-hop-self
					neighbor 172.16.1.1 active
					neighbor 172.16.1.1 send-label
					neighbor 172.16.1.1 next-hop-self
					no synchronization
					no auto-summary
					network 172.16.1.1 mask 255.255.255.255
					network 172.16.1.2 mask 255.255.255.255

			*each pseudowire has seperated subinterface on asbr

	ldp authentication for pseudowire signaling
		authentication
			mpls ldp neighbor 10.10.10.2 password 123
			!works on md5
		
			show mpls ldp neighbor 10.10.10.2 details
 			
 			debug mpls packet

	quality of service in atom
		pe1
			class-map match-any all-traffics
			match any
	
			policy-map exp3
			class all-traffics
			set mpls expriment 3
	
			policy-map exp5
			class all-traffics
			set mpls expriment 5

			interface atm0/0.100 point-to-point
			pvc 0/100 l2transport
			encapsulation aal5
			xconnect 10.10.10.2 100 encap mpls
			service-policy input exp5

			interface atm0/0.200 point-to-point
			pvc 0/200 l2transport
			encapsulation aal5
			xconnect 10.10.10.2 200 encap mpls
			service-policy input exp3

			debug mpls pacekt

		qos group
			internal taging and set policy on them
			we can change bandwidth

			swapping and hopping on mpls will be happen but qos group don't omit

			pe1				
				class-map match-all all-traffic
				match any

				policy-map policing
				class all-traffics
				policy cir 128000
				confirm-action set-mpls-exprimental transit 5 exceed-action drop

				interface atm0/0.100
				pvc 0/100 l2transport
				encapsulation aal5
				xconnect 10.10.10.1 200 encap mpls
				service-policy in policing


		queuing and shaping
			low-latency queuing (llq)
			class-based weighted fair queuing (cbwfq)
			byte-based weighted random early detection (wred)

			pe1
				class-map match-all customer-a
				match fr-dlci 100

				class-map match-all customer-b
				match fr-dlci 200
		
				policy-map cir_guarantee
				class customer-a
				bandwidth 128
				class customerb
				bandwidth 256
		
				interface seria13/1
				no ip address
				service-policy output cir_guarantee
				encapsulation frame-relay
				frame-relay intf-type dce


			layer 2 protocol 		matching 			setting
			-------------------------------------------------------------- 
			ethernet 				match cos 			set cos
									match vlan
			--------------------------------------------------------------
			frame relay 			match fr-de 		set fr-de
									match fr-dlci		set fr-fecn-becn
			--------------------------------------------------------------
			atm 					match atm clp 		set atm-clp

			
			how transfer voice data on pseudowire 
				mpls te
				fast reroute
			
			pe1
				class-map match-any fr dlci 100
				match fr-dlci 100
				class-map match-any fr deo
				match not fr-de

				policy-map fr_policing
				class fr dlci 100
				police cir 64000 pir 128000
				conform-color fr deo
				conform-action set-mpls-exp-transmit 5
				exceed-action set-mpls-exp-transmit"2
				violate-action drop
				class class-default
				set mpls experimental 0

				interface pos4/0
				encapsulation frame-relay cisco
				
				interface pos4/0.1 point-to-point
				switched-dlci 100
				service-policy input fr_policing
				
				connect frompls101 pos4/0 100 12transport
				xconnect 10.10.10.3 70 encap mpls

			*many pseudowire on many equal path is efficient
			many pseudowire on one path is not efficient

			hash value for vc label base on destination routers will be calculated and select egress path

			has no out of order packet
			if has no same path select best paths

			normally we have unequal multipathing and set cost help us to have equal multipathing
			if has connected clients advertise without label
****************************
l2tpv3
	incompatibility with l2tp but extended of version 2
	over ip make pseudo that transfer layer 2 

	between 2 pe might have many sessions
	between 2 pe we have control channel same as tunnel inside atom which contain many sessions
	session same as pseudowire mpls

	use same data packet path for control message as inband (same format for control and data message)

	session establishment
		manual
			defined static sessions with or without a keepalive mechanism

		dynamic
			l2tpv3 signaling protocol enabled

			extending l2tpv2's control channel signaling

	data encapsulation
		packet-switched network (psn layer)
			ip/udp delivery header
				psn
					ipv4 header (ip protocol 17)
					udp header

				demultiplexing sublayer
				encapsulation sublayer (optional)
				payload

				*help us on nat and firewall and ipsec
				*we have checksum here to check data

			ip delivery header
				psn
					ipv4 header (ip protocol  115)
				
				demultiplexing sublayer
				encapsulation sublayer (optional)
				payload

		demultiplexing sublayer
			label stack on atom
				session id (local) (4 octets)

				cookie (authentication)(0,4 or 8 octets)
		
		encapsulation sublayer (optional)(3octets)
			same control word
			also contain sequence number 

		payload
			frame relay
			hdlc
			ppp

	control connection
		utilizes a single reliable, inband control plane for both
			Step1 (like ldp on atom)(dynamic)
				 L2TPV3 tunnel establishment phase

			Step2 (might be manual or dynamic)
				 individual pseudowire sessions (L2TP sessions)

				 modes
				 	Manual Mode
				 		predefined session IDs and cookies

				 	Manual Mode with Keepalive
				 		negotiates the Control Connection phase but not the Session Negotiation phase
				 		Offers dead-peer detection mechanism

				 	Dynamic Mode
				 		negotiates both the Control Connection phase as well as the Session Negotiation phase for each pseudowire session

		control message encapsulation
			ipv4 header (ip proto = 115)

			session identifier (4 octets)
			[=0x00000000]

			control message header
				t l x x s x x x
				x x x x	version (4 bits)		
				length (2 octets)
				control connection id (4 octets)
				  			
				ns (2 bytes) 	nr (2 bytes)
				----------------------------
				t > control base packets
				
				l means have length
				
				s means sequence
				
				
				version 3 means l2tpv3

				ns > sequence number sent

				nr > sequence number expected to be received in the next message

				control connection id > locally significant field to represent control channel

			attribute value pair (avp)
				m 	h 	rsvd (4 bits)

				avp length (10 bits)
				vendor id (2 octets)
				attribute type (2 octets)
				attribute value (variable)
				------------------------------------
				m >  it indicates that associated control connection or pw session must be shut down if recipient does not recognize this avp
					if a control connection occurs, a stop control connection (stopccn) message is sent
					if a pw session occurs a call disconnect notification (cdn) message is sent

				h > the hidden bit (h-bit) indicates to the recipient whether the avp content is passed in clear text or obfuscated in some manner to hide sensitive information
				a shared secret must be defined on both endpoints control message authentication must be enabled, and a random vector avp must be sent

				vendor id 0 menas ietf

			additional avp
				...
	
				avp

		phase 1
			start control connection request (sccrq)

			start control connection reply (sccrp)

			start control connection connected (scccn)

			hello

			stop control connection notification (stopccn)

		phase 2
			session establishment phase
				incoming call request (icrq)
	
				incoming call reply (icrp)
	
				incoming call connection (iccn)

			set link info (sli)

			call disconnect notify (cdn)
****************************
lan over l2tpv3
	signaling 
		control channel like lsp tunnel on atom

		pseudowire session between attachment circuites like pseudowire

		*on one control channel might have many sessions

	attachment circuite types
		port mode
			works on which port and also contain vlan

		vlan mode
			just works on vlan tag

		isp has no process on these

	config models
		manual mode
			all session characteristics to be configured on each end of l2tpv3 endpoint
			manual definition of session parameterssuch as session cookies and session ids

		manual mode with keepalive
			the same manner as manual mode
			enables a simple peer keepalive mechanism for dead peer detection
		
		dynamic mode
			utilizes control channel for peer capability and pseudowire session negotiation so that manual preconfiguration is unnecessary

		pseudowire-class x
			encapsulation {l2tpv3 | mpls}

				is important
			
			ip local interface interface-name
				source address of the l2tpv3 control and data packets
				encapsulation and ip local interface definitions are minimum arguments

				is important

			protocol {12tpv3 | none} [l2tp-class-name]
				for dynamic session negotiation, configure protocol l2tpv3 (default)
				
				optionally reference an i2tp-class template so that multiple sessions can share the same control channel characteristics

				if no session negotiation is required, select protocol none (manual)

				is important

			sequencing {transmit | receive | both}

			ip dfbit set
				don't fragment (df) bit is set
			
			ip pmtu
				l2tpv3 supports discovery of mtu to reach the remote l2tpv3 endpoint

				is important

			ip tos {value value | reflect}
				reflect option reflects the tos value that is stored in the inner ip header to the outer ip header
				if ip tos value and ip tos reflect are configured simultaneously the configured tos value is used on the outer ip header when the layer 2 frame payload is not ip, while reflection would occur when the payload is ip
			
			ip ttl value

		l2tp-class x
			cookie size [4 | 8] [size]
				the default, the peer does not pass a cookie

				tunnel key in gre has no security target just blind insertion protection 

				is important

			receive-window [size]
				utilizes a sliding window using ns, and nr 
				maximum frame sent and acknowlege received

			retransmit {initial retries initial-retries | retries retries | timeout	{max | min} timeout}
				defines the number of retransmission attempts before declaring the remote end as unresponsive
				if get trouble wait on minimum time and repeated attemps need more waiting
				this threshold is between min and max

			timeout setup [seconds]
				maximum amount of time permitted to set up the control channel

			authentication
				enables the old chap-like lightweight control channel authentication between peers

				is important

			hostname [host name]
				explicitly defines host name to identify local device in old chap-like authentication
				if you do not explicitly use hostname, host name of router is used

				known as username

			password {encryption-type}[password]
				establishes predefined shared secret between peers used in old chap-like
				if not specified, password value is taken from globally configured username
			
				[username] password [password] value, where username is host name of local device

			digest [secret [0 | 7] password] [hash {md5 | sha}]
				defines shared secret and hashing mechanism
				the default assumes a 0 input type option and hash md5
			------------------------------------------------------------

			pe1
				interface loop 0
				ip address 10.10.10.2 255.255.255.255

				pseudowire-class pw-x
				encapsulation l2tpv3
				protocol l2tpv3 pw-x
				ip local interface loop 0
				ip protocol l2tpv3

				l2tp-class l2-x
				authentication
				password md5 123
				hello 5

				interface gig0/0
				no shut
				interface gig0/0.100
				xconenct 10.10.10.3 100 encap l2tpv3 pw-class pw-x
				--------------------------------------------------

				manual mode
					pseudowire-class pw-x
					encapsulation l2tpv3
					protocol none
					ip local interface loop 0

					interface gig0/0
					no shut
					interface gig0/0.100
					xconnect 10.10.10.3 100 encap l2tpv3 manual pw-class pw-x 
					l2tp id 1000 2000
					l2tp cookie local 10
					l2tp cookie remote 20
					l2tp hello 5

			gsr 12k 
				interface gig0/0
				no keepalive
				ip unnumbered loop 0
				loop internal

				hw-module slot 1 mode server


											ospf area 0
								*******************************
						10.10.10.1/32 	10.10.10.10/32 		10.10.10.2/32

		ce1 ---------------- pe1 ------------- p1 ------------ pe2 ---------------- ce2
								10.1.1.0/24 		10.1.2.0/24

		*ip base media no mpls

		manual session
			port mode
				p1
					interface gig0/0
				 	no shut
				 	ip address 10.1.1.1 255.255.255.0
				 	
				 	interface gig0/1
				 	no shut
				 	ip address 10.1.2.1 255.255.255.0

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

				pe1
					interface gig0/0
					no shut
					!ce side

					interface gig0/1
				 	no shut
				 	ip address 10.1.1.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.1 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-manual
					encapsulation 12tpv3
					protocol none
					ip local interface Loop 0
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 manual pw-class pw-manual
					12tp id 1 2
					!control channel id can be different cause local significant from session id which is 33

					12tp cookie local 4 1
					12tp cookie remote 4 2

					*if need manual and keep alive need l2tp class and set protocol in pseudowire class as none and called l2tp class under xconnect and l2tp parameters 

					show l2tun
					show l2tun all
					!just show dynamic tunnels

					show l2tun session all
					!if see n\a means manual mode definition

				pe2

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
				 	no shut
				 	ip address 10.1.2.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.2 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-manual
					encapsulation 12tpv3
					protocol none
					ip local interface Loop 0
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 manual pw-class pw-manual
					12tp id 2 1
					12tp cookie local 4 2
					12tp cookie remote 4 1

				ce1
					interface gig0/0
					no shut
					ip address 192.168.1.1 255.255.255.0
				
				ce2
					interface gig0/0
					no shut
					ip address 192.168.1.2 255.255.255.0

			vlan mode
				just set them on subinterface

		manual session with keepalive
			port mode
				p1
					interface gig0/0
				 	no shut
				 	ip address 10.1.1.1 255.255.255.0
				 	
				 	interface gig0/1
				 	no shut
				 	ip address 10.1.2.1 255.255.255.0

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

				pe1
					interface gig0/0
					no shut
					!ce side

					interface gig0/1
				 	no shut
				 	ip address 10.1.1.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.1 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-manual
					encapsulation 12tpv3
					protocol none
					ip local interface Loop 0

					12tp-class l2-keepalive
					hello 30
					retransmit retries 5
					retransmit timeout max 4
					retransmit timeout min 2
					retransmit initial retries 3
					retransmit initial timeout max 7
					retransmit initial timeout min 2
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 manual pw-class pw-manual
					12tp id 1 2
					!control channel id can be different cause local significant from session id which is 33

					12tp cookie local 4 1
					12tp cookie remote 4 2
					l2tp hello l2-keepalive

					show l2tun
					!show us l2tp class

					show l2tun all
					!just show dynamic tunnels

					show l2tun session all
					!if see n\a means manual mode definition

				pe2

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
				 	no shut
				 	ip address 10.1.2.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.2 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-manual
					encapsulation 12tpv3
					protocol none
					ip local interface Loop 0

					12tp-class l2-keepalive
					hello 30
					retransmit retries 5
					retransmit timeout max 4
					retransmit timeout min 2
					retransmit initial retries 3
					retransmit initial timeout max 7
					retransmit initial timeout min 2
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 manual pw-class pw-manual
					12tp id 2 1
					12tp cookie local 4 2
					12tp cookie remote 4 1
					l2tp hello l2-keepalive

				ce1
					interface gig0/0
					no shut
					ip address 192.168.1.1 255.255.255.0
				
				ce2
					interface gig0/0
					no shut
					ip address 192.168.1.2 255.255.255.0

			vlan mode
				just add on subinterface

		dynamic
			port mode
				p1
					interface gig0/0
				 	no shut
				 	ip address 10.1.1.1 255.255.255.0
				 	
				 	interface gig0/1
				 	no shut
				 	ip address 10.1.2.1 255.255.255.0

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

				pe1
					interface gig0/0
					no shut
					!ce side

					interface gig0/1
				 	no shut
				 	ip address 10.1.1.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.1 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-dynamic
					encapsulation 12tpv3
					protocol l2tpv3 l2-keepalive-&-pass
					ip local interface Loop 0

					12tp-class l2-keepalive-&-pass
					hello 30
					retransmit retries 5
					retransmit timeout max 4
					retransmit timeout min 2
					retransmit initial retries 3
					retransmit initial timeout max 7
					retransmit initial timeout min 2
					authentication 
					password 123
					cookie size 8
					!cookie get auto negotiate
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 pw-class pw-dynamic

					show l2tun
					!show us l2tp class
					
					show l2tun all
					!just show dynamic tunnels

					show l2tun session all
					!if see n\a means manual mode definition

				pe2

					interface gig0/0
					no shut
					!ce side
					
					interface gig0/1
				 	no shut
				 	ip address 10.1.2.2 255.255.255.0
				 	
				 	interface loop 0
				 	ip address 10.10.10.2 255.255.255.255

				 	router ospf 1
				 	network 10.0.0.0 0.255.255.255 area 0

					pseudowire-class pw-dynamic
					encapsulation 12tpv3
					protocol l2tpv3 l2-keepalive-&-password 
					ip local interface Loop 0

					12tp-class l2-keepalive-&-password
					hello 30
					retransmit retries 5
					retransmit timeout max 4
					retransmit timeout min 2
					retransmit initial retries 3
					retransmit initial timeout max 7
					retransmit initial timeout min 2
					authentication
					password 123
					cookie size 8
					
					interface gig0/0
					no ip address
					no cdp enable
					xconnect 10.10.10.2 33 encapsulation 12tpv3 pw-class pw-dynamic

				ce1
					interface gig0/0
					no shut
					ip address 192.168.1.1 255.255.255.0
				
				ce2
					interface gig0/0
					no shut
					ip address 192.168.1.2 255.255.255.0

			vlan mode
				just set on subinterface
****************************
wan over l2tpv3
	ppp
	hdlc
	framerelay

	l2tp data message header
		l2tp session header
			l2tpv3 session header over ip
				session id
				cookie (optional, maximum 64 bits)
		
		l2-specific sublayer
			default layer 2-specific sublayer format
				sequence number

		tunnel payload

	default layer 2-specific sublayer format

	t-bit atm admin cell or aal5 payload
	g-bit efci
	c-bit clp
	u-bit c/r

	here our control word is not important to send so our payload tunnel has no sublayer except sending sequence number that needs sublayerc

	in contrast to frompls which requires control word, frol2tpv3 does not require layer 2-specific sublayer
		fecn
		becn
		de
		command response field
		in frol2tpv3, the q.922 header is transported the layer 2-specific sublayer header is not needed
		whole payload will be collect all frames

	another protocols we have same concept as before

	hdlc over l2tpv3
		flag
			1
			0x7e

		transported in l2tpv3 (hdlc pseudowire)
			addr
				1

			ctrl
				1
				0x00

			protocol
				2
				[etype]

			data

		fcs
			2

		flag
			1
			0x7e

	ppp over l2tpv3
		flag
			1
			0x7e

		addr
			1
			0xff
			omitted when using address and control field compression (acfc)

		ctrl
			1
			0x00
			omitted when using address and control field compression (acfc)

		transported in l2tpv3 (ppp pseudowire)
			ppp dll
				2
				only 1 byte when using protocol field compression (pfc)

			data

		fcs
			2

		flag
			1
			0x7e
			ending flag only needed on single frame or final frame of a sequence

			

	atm over l2tpv3
		atm aal5-sdu mode
			efci, clp, and c/r bits are transported in the required atm-specific sublayer

		atm cell mode
			single cell relay
			cell concatenation
				vcc
				vpc
				port

				we have special avp and maximum number of packets on atm cell

	mtu
		transport overhead
			cisco frame relay dlci: 4 byte

			ietf frame relay: 10 bytes
			
			cisco hdlc: 4 byte
			
			ppp: 2 byte
			
			aal5: 0-32 byte
		
		l2tpv3 overhead
			l2tp session overhead
		 		session id: 4-byte
		 		cookie (optional)
		 			null, 4 bytes, or 8 bytes
		
			l2-specific overhead (control word and 4 byte)
		
		outer ip overhead : 20 byte
			if use  udp over ip need 28 byte
		--------------------------------
		1500 on isp + these above values

		worst is 1574 byte for isp mtu and forward all layer 2 frames

	same config as before on lan


						10.10.10.1/32 	10.10.10.10/32 		10.10.10.2/32

		ce1 ---------------- pe1 ------------- p1 ------------ pe2 ---------------- ce2
								10.1.1.0/24 		10.1.2.0/24

											ospf area 0
								*******************************
	
		ppp over l2tpv3
			dynamic
				pe1
					12tp-class 12tpv3-wan
					authentication
					password 0 cisco
					cookie sizg 4
		
					pseudowire-class wan-12tpv3-pw
					encapsulation 12tpv3
					protocol 12tpv3 12tpv3-wan
					ip local interface Loop 0
		
					interface gig6/0
					no ip address
					encapsulation ppp
					no cdp enable
					xconnect 10.10.10.3 60 pw-class wan-12tpv3-pw sequencing both
	
		framerelay over l2tpv3
			dynamic
				pe1
					frame-relay switching

					12tp-claRs 12tpv3-wan
					authentitation
					password 0 cisco
					cookie size 4
					
					pseudowire-class wan-12tpv3-pw
					encapsulation 12tpv3
					protocol 12tpv3 12tpv3-wan
					ip local interface Loop 0

					interface Seria17/0
					no shut
					no ip address
					encapsulation frame-relay
					frame-relay intf-type dce
					!ce side
					
					connect 12tpv3-fr-dlci Seria17/0 100 12transport
					xconnect 10.0.0.203 70 pw-class wan-12tpv3-pw

				pe2				
					frame-relay switching

					12tp-claRs 12tpv3-wan
					authentitation
					password 0 cisco
					cookie size 4
					
					pseudowire-class wan-12tpv3-pw
					encapsulation 12tpv3
					protocol 12tpv3 12tpv3-wan
					ip local interface Loop 0

					interface Seria17/0
					no shut
					no ip address
					encapsulation frame-relay
					frame-relay intf-type dce
					!ce side
					
					interface Seria17/0.1 point-to-point
					ip address 192.168.102.1 255.255.255.252
					frame-relay interface-dlci 100

				ce1
					interface gig0/0
					no shut
					encapsulation frame-relay

					interface gig0/0.100 point-to-point
					frame-relay interface-dlci 100
					ip address 192.168.1.1 255.255.255.0

				ce2
					interface gig0/0
					no shut
					encapsulation frame-relay

					interface gig0/0.100 point-to-point
					frame-relay interface-dlci 100
					ip address 192.168.1.2 255.255.255.0

		aal5 sdu and aal0 over l2tpv3
			dynamic
				aal5
					pe1
						pseudowire-class pw-12tpv3-atm
						encapsulation 12tpv3
						ip local interface Loop 0
						
						interface ATM5/0
						no ip address
						pvc 0/100 12transport
						encapsulation aal5
						xconnect 10.0.0.203 27 pw-class pw-12tpv3-atm
						!don't have this on 7200

						show atm pvc interface atm0/0.100
						!on pe router show aal5 and ce router show snap
						
						show atm pvc interface atm0/0.100 details

					ce1
						interface ATM6/0.1 point-to-point
						ip address 192.168.103.1 255.255.255.252
						pvc 0/100
						oam-pvc 0
						encapsulation aal5snap

				cellmode
					pe1
						pseudowire-class pw-12tpv3-atm
						encapsulation 12tpv3
						ip local interface Loop 0
						
						interface ATM5/0
						no ip address
						pvc 0/100 12transport
						encapsulation aal0
						xconnect 10.0.0.203 27 pw-class pw-12tpv3-atm
						!don't have this on 7200
						-------------------------------------------
						atm cell relay vc mode configuration
							interface atm5/0
							pvc 0/200 12transport
							encapsulation aal0
							xconnect 10.10.10.3 28 pw-class pw-12tpv3-atm

							show l2tun session all vcid 27 | inc type is
						//////////////////////////////////////////////
						atm cell relay vp mode configuration
							interface atm5/0
							atm pvp 5 12transport
							xconnect 10.10.10.3 5 pw-class pw-12tpv3-atm

							show l2tun session all vcid 5 | inc type is
						//////////////////////////////////////////////
						atm cell relay port mode configuration
							interface atm3/0
							xconnect 10.10.10.3 3 pw-class pw-12tpv3-atm

							show l2tun session all vcid 3 | inc type is
						//////////////////////////////////////////////
						-------------------------------------------
						
						show atm pvc interface atm0/0
						!on pe router show aal5 and ce router show snap
						
						show atm pvc interface atm0/0 details

						show l2tun session
						!show aal5 

					ce1
						interface ATM6/0.1 point-to-point
						ip address 192.168.103.1 255.255.255.252
						pvc 0/100
						oam-pvc 0
						encapsulation aal0
****************************
l2tpv3 advance features
						10.10.10.1/32 	10.10.10.10/32 		10.10.10.2/32

		ce1 ---------------- pe1 ------------- p1 ------------ pe2 ---------------- ce2
								10.1.1.0/24 		10.1.2.0/24

											ospf area 0
								*******************************

	atm cell packing
		pe1
			interface atm0/0
			shutdown
			atm mcpt-timers 100 1000 4095
			no shut
			pvc 0/200 l2transport
			encapsulation aal0
			cell-packing 14 mcpt-timers 3
			!14 packet on each packing and wait 3 microsecond
			xconnect 10.10.10.3 28 pw-class pw-l2tpv3-atm

			show atm cell-packing

			*example 48byte'cell *  14 cell > 672 byte
			16 byte aal5/snap encapsulation ce

			672 - 16 > 656

			on ping and test we set 656 byte as datagram size of icmp
			*higher value of datagram size make 2 packet and cell transmission
 
	l2tpv3 path mtu discovery	
		don't send ip packets larger than core mtu minus 36 bytes
			20 bytes of ipv4 delivery header
			4 bytes of l2tpv3 session id
			4 bytes of l2tpv3 cookie
			4 bytes layer 2-specific sublayer used for sequencing
			4 bytes hdlc

			1500-36 = 1464

			*more than this must get fragment and too much load on cpu

		show process cpu | inc util|pid|ip input
		
		ping 192.168.1.1 rep 500 size 1465 df-bit
		!just one byte bigger than suitable mtu

		show ip traffic |inc ip stat|frag|reass
		!make sure on fragmentation

		show interface gig0/0 stats
		!route cache menas cef

		show interface gig0/0 switching

		how does pmdtud work ?
			 copies df bit from inner ip header into outer ipv4 header

			 find out and record path mtu for the session

			 if received ipv4 packet from ce has df bit cleared and resulting l2tpv3 packet exceeds discovered mtu
			 	it fragments ce ipv4 packet, copies original layer 2 header and appends it into each of the generated fragments
			 	pushes computational expensive ipv4 reassembly into the receiving ce device and relieves the pe from being a centralized 			 reassembly point

			 if received ipv4 packet from ce has df bit set and resulting l2tpv3 packet exceeds discovered mtu
			 	generates icmp unreachable messages to the ce device

		pe1
			12tp-class l2tp-class
			authentication
			password 7 123
			cookie size 4

			pseudowire-class pw-class
			encapsulation 12tpv3
			sequencing both
			protocol 12tpv3 l2tp-class
			ip local interface loop 0
			!no ip pmtu
			!cause disable pmtud 

			ip pmtu
			!enable path mtu discovery

			ip df-bit set
			!if received one large packet than base mtu convert and make path mtu discovery after this could turn off

			int gig1/1
			xconnect 10.10.10.2 100 pw-class pw-class

		pe2
			12tp-class l2tp-class
			authentication
			password 7 123
			cookie size 4

			pseudowire-class pw-class
			encapsulation 12tpv3
			sequencing both
			protocol 12tpv3 l2tp-class
			ip local interface loop 0
			ip pmtu
			!enable path mtu discovery

			ip df-bit set

			int gig1/1
			xconnect 10.10.10.3 100 pw-class pw-class

		*headers after fragmentation comes to one packet and another packets just contain data, instead process one packet must process 2 packet this is cause the load, recommended use pmd on pe routers

		before send ip header and layer2 payload make fragmentation then send them with their headers, in this condition we have no load

		now loads will be on ce routers

		show l2tun session all | inc pmtu

		*must restart sessions

		after do these say path mtu is not enable, must force ce routers to send pacekt with fragment packet
			on ce
				ping 192.168.1.2  size 1465 rep 1 df-bit 

		*after this discovery get compelete
		instead of these we can set on pe (ip df-bit set)

	qos
		mark outer layer, cause ip base media on isp

		services
			llq
			cbwfq
			wred

		policing will be on ingress of pe routers from ce routers

		ingress interface to pe routers is layer 2 in layer2 vpn we can set vlan, cos, framerelay dlci for classification not precedence or dscp

		pe1
			class-map match-all all_traffic
			match any

			policy-map prec-2
			class all traffic
			set ip precedence tunnel 2

			interface atm5/0
			pvc 0/100 12transport
			oam-ac emulation-enable 2
			encapsulation aal5
			xconnect 10.10.10.3 27 pw-class pw-12tpv3-atm
			service-policy in prec-2
			-------------------------------------------
			class-map match-all all_traffic
			match any

			policy-map my_policer
			class all_craffic
			police cir 64000 pir 128000
			conform-action set-prec-tunnel-transmit 5 exceed-action set-prec-tunnel-transmit 1	violate-action drop

			interface atm5/0
			pve 0/100 12transport
			cam-ac emulation-enable 2
			encapsulation aa15
			xconnect 10.10.10.3 27 pw-class pw-12tpv3-atm
			sexvice-policy in my_policer
			------------------------------------------
			class-map match-all cust1
			match fr-dlci 100

			class-map match-all cust2
			match fr-dlci 101

			policy-map cir_guarantee
			class cust1
			bandwidth 128

			class cust2
			bandwidth 256

			interface seria13/1
			no ip address
			service-policy output cir_guarantee
			encapsulation frame-relay
			frame-relay intf-type dce
****************************
internetworking & local switching
	internetworking  
		any to any pseudowire connection on different datalink layer

		normally use in isp on atom

		modes
			bridge mode (l2)
				headers and payload otherwise whole frame will transmit

			routed mode (l3)
				contains payload from layer2
				don't forward layer2 headers

			could not  use on different technologies for bridge mode recommended use routed mode
			ethernet and vlan mode recommended use bridge or routed
			ppp recommended use routed also ethernet use routed

		Limitations
			ethernet/vlan
				multipoint corfigurations are not supported

				for ethernet to frame relay interworking in ospf, one site operating in broadcast and the other site on nonbroadcast or 		point-to-point would result in ospf adjacency not forming across the pseudo wire
					ensure that ospf operates in a single mode on both ends
				
				the pe router acts as a proxy arp server and responds with its own mac address to ce routers arp requests
					when you change the interworking configuration on ethernet pe router ensure that the arp entry on the adjacent ce router is cleared

			frame relay
 				inverse arp is not supported with ip interworking
					ce routers must use point-to-point subinterfaces or static maps

			atm or aal5
				only atm aal5 vc mode is supported
					both aal5mux and aal5snap encapsulations are supported
				
				arp is not supported with ip interworking
					ce routers must use point-to-point subinterfaces or static maps

		mtu considerations
			serial
			ethernet
			----------------
			1500 bytes

			hssi
			atm
			pos
			---------------
			4470 bytes

			*doesn't matter on atm cell mode

		config
			pe1
				routed mode
					pseudowire-class ri-pwc
					encapsulation mpls
					internetworking ip
					!use this when need every thing except vlan, ethernet or port
					!usually use this
				++++++++++++++++++++++++++++++++++++
				bridge mode
					pseudowire-class bi-pwc
					encapsulation mpls
					internetworking ethernet

					frame-relay switching
					interface gig0/0
					encapsulation frame-relay
					frame-relay intf-type dce
					connect fr gig0/0 100 l2transport
					xconnect 10.10.10.3 100 pw-class ri-pwc
					------------------------------------------
					xconnect 10.10.10.3 100 pw-class bi-pwc
					!frame-relay base
					///////////////////////////////////////////////
					interface gig0/0
					encapsulation ppp
					xconnect 10.10.10.3 100 pw-class ri-pwc
					------------------------------------------
					xconnect 10.10.10.3 100 pw-class bi-pwc
					!ppp and vlan base
					///////////////////////////////////////////////
					interface gig0/0
					xconnect 10.10.10.3 100 pw-class ri-pwc
					------------------------------------------
					xconnect 10.10.10.3 100 pw-class bi-pwc
					!ethernet base
					///////////////////////////////////////////////
					interface gig0/0.100 point-to-point
					pvc 0/100 l2transport
					encapsulation aal5snap
					xconnect 10.10.10.3 100 pw-class ri-pwc
					------------------------------------------
					xconnect 10.10.10.3 100 pw-class bi-pwc
					!atm aal5 base

			
							10.10.10.1/32 	10.10.10.10/32 		10.10.10.2/32
			
			ce1 ---------------- pe1 ------------- p1 ------------ pe2 ---------------- ce2
									10.1.1.0/24 		10.1.2.0/24

												ospf area 0
												 ip base sp
									*******************************

				base on ip and mpls with l2tpv3
					p1
						interface gig0/0
					 	no shut
					 	ip address 10.1.1.1 255.255.255.0
					 	
					 	interface gig0/1
					 	no shut
					 	ip address 10.1.2.1 255.255.255.0

					 	interface loop 0
					 	ip address 10.10.10.10 255.255.255.255
					 	
					pe1	
						interface gig0/0
						no shut
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.1.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.1 255.255.255.255

						pseudowire-class l2tpv3-pw-cls
						encapsulation 12tpv3
						internetworking ethernet
						ip local interface Loop 0

						interface gig0/0
						no ip address
						xconnect 10.10.10.2 100 encapsulation mpls pw-class l2tpv3-pw-cls

						show l2tun 
						show l2tun session interworking

					pe2					
						interface gig0/0
						no shut
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.2.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.2 255.255.255.255

						pseudowire-class l2tpv3-pw-cls
						encapsulation 12tpv3
						internetworking ethernet
						ip local interface Loop 0

						interface gig0/0.100
						no ip address
						encapsulation dot1q 100
						xconnect 10.10.10.1 100 encapsulation mpls pw-class l2tpv3-pw-cls

					ce1
						interface gig0/0
						no shut
						ip address 172.16.10.1 255.255.255.0
						
					ce2
						interface gig0/0
						no shut

						interface gig0/0.100
						no shut
						encapsulation dot1q 100
						ip address 172.16.10.2 255.255.255.0

				base on ip and mpls with atom
					p1
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/0
						mpls ip
						interface gig0/1
						mpls ip

						interface gig0/0
					 	no shut
					 	ip address 10.1.1.1 255.255.255.0
					 	
					 	interface gig0/1
					 	no shut
					 	ip address 10.1.2.1 255.255.255.0

					 	interface loop 0
					 	ip address 10.10.10.10 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0
					 	
					pe1	
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip

						interface gig0/0
						no shut
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.1.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.1 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ethernet

						interface gig0/0
						no ip address
						xconnect 10.10.10.2 100 encapsulation mpls pw-class atom-pw-cls

						show mpls l2transport summary
						show mpls l2transport vc 100 
						show mpls l2transport vc 100 details

						show arp
						!show us mac address of pe routers cause we have arp proxy

					pe2				
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip

						interface gig0/0
						no shut
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.2.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.2 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ethernet

						interface gig0/0.100
						no ip address
						encapsulation dot1q 100
						xconnect 10.10.10.1 100 encapsulation mpls pw-class atom-pw-cls

					ce1
						interface gig0/0
						no shut
						ip address 172.16.10.1 255.255.255.0
						
					ce2
						interface gig0/0
						no shut

						interface gig0/0.100
						no shut
						encapsulation dot1q 100
						ip address 172.16.10.2 255.255.255.0


				for  frame relay and aal5 need internetworking on ip cause don't understand eachother

				base on ip and mpls with ethernet to ppp
					p1
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/0
						mpls ip
						interface gig0/1
						mpls ip

						interface gig0/0
					 	no shut
					 	ip address 10.1.1.1 255.255.255.0
					 	
					 	interface gig0/1
					 	no shut
					 	ip address 10.1.2.1 255.255.255.0

					 	interface loop 0
					 	ip address 10.10.10.10 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0
					 	
					pe1	
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip
						
						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ip

						interface gig0/0
						no shut
						encapsulation ppp
						xconnect 10.10.10.2 100 encapsulation mpls pw-class atom-pw-cls
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.1.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.1 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						show mpls l2transport summary
						show mpls l2transport vc 100 
						show mpls l2transport vc 100 details

						show arp
						!show us mac address of pe routers cause we have arp proxy

					pe2				
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip

						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ip

						interface gig0/0
						no shut
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.2.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.2 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						interface gig0/0.100
						no ip address
						encapsulation dot1q 100
						xconnect 10.10.10.1 100 encapsulation mpls pw-class atom-pw-cls

					ce1
						interface gig0/0
						no shut
						encapsulation ppp
						ip address 172.16.10.1 255.255.255.0
						
					ce2
						interface gig0/0
						no shut

						interface gig0/0.100
						no shut
						encapsulation dot1q 100
						ip address 172.16.10.2 255.255.255.0


				base on ip and mpls with ppp to framerelay
					p1
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/0
						mpls ip
						interface gig0/1
						mpls ip

						interface gig0/0
					 	no shut
					 	ip address 10.1.1.1 255.255.255.0
					 	
					 	interface gig0/1
					 	no shut
					 	ip address 10.1.2.1 255.255.255.0

					 	interface loop 0
					 	ip address 10.10.10.10 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0
					 	
					pe1	
						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip
						
						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ip

						interface gig0/0
						no shut
						encapsulation ppp
						xconnect 10.10.10.2 100 encapsulation mpls pw-class atom-pw-cls
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.1.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.1 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						show mpls l2transport summary
						show mpls l2transport vc 100 
						show mpls l2transport vc 100 details

						show arp
						!show us mac address of pe routers cause we have arp proxy

					pe2				
						frame-relay switching

						mpls label protocol ldp
						mpls ldp router-id loop 0
						interface gig0/1
						mpls ip

						pseudowire-class atom-pw-cls
						encapsulation mpls
						internetworking ip

						interface gig0/0
						no shut
						encapsulation frame-relay
						frame-relay intf-type dce
						!ce side

						interface gig0/1
					 	no shut
					 	ip address 10.1.2.2 255.255.255.0
					 	
					 	interface loop 0
					 	ip address 10.10.10.2 255.255.255.255

					 	router ospf 1
					 	network 10.0.0.0 0.255.255.255 area 0

						interface gig0/0.100 point-to-point
						no ip address
						connect fr gig0/0 100 l2transport
						xconnect 10.10.10.1 100 encapsulation mpls pw-class atom-pw-cls

					ce1
						interface gig0/0
						no shut
						encapsulation ppp
						ip address 172.16.10.1 255.255.255.0
						
					ce2
						interface gig0/0
						encapsulation frame-relay
						no shut

						interface gig0/0.100 point-to-point
						frame-relay interface-dlci 100
						ip address 172.16.10.2 255.255.255.0

	local switching
		switch layer 2 frames between two different ac on the same pe
		interfaces of the same type
			atm to atm
				pvc encapsulation supported
					aal5 using encapsulation aal5
						vpi/vci do not need to match in both endpoints
				
				scr single cell relay vc mode using encapsulation aal0
					vpi/vci must match in both endpoints
						oam cells are transported as cells

			frame relay to frame relay
			ethernet to ethernet/vlan to vlan

		interfaces of different types
			atm to ethernet/vlan
			atm to frame relay
			frame relay to ethernet/vlan

	
		ce1 ----------------|  
							|		
							| pe1	 	
							|					
		ce2 ----------------|

		frame-relay to frame-relay
			pe1
				frame-relay switching
				
				interface gig0/0
				no shut
				no ip address
				encapsulation frame-relay
				frame-relay intf-type dce
				---------------------------
				frame-relay interface-dlci 100 switched
				
				interface gig1/0
				no ip address
				no shut
				encapsulation frame-relay
				frame-relay intf-type dce
				---------------------------
				frame-relay interface-dlci 200 switched

				connect fr gig0/0 100 gig1/0 200
				connect fr gig1/0 200 gig0/0 100

			ce1
				interface gig0/0
				no shut
				encapsulation frame-relay

				interface gig0/0.100 point-to-point
				frame-relay interface-dlci 100
				ip address 192.168.1.1 255.255.255.0
			
			ce2
				interface gig0/0
				no shut
				encapsulation frame-relay

				interface gig0/0.200 point-to-point
				frame-relay interface-dlci 200
				ip address 192.168.1.2 255.255.255.0

		atm to atm
			pe1
				interface atm1/0
				pvc 0/100 12transport
				encapsulation aal5
				
				interface atm2/0
				pvc 0/200 12transport
				encapsulation aal5
				
				connect aal5 local sw atm 1/0 0/100 atm 2/0 0/200

		ethernet to ethernet
			pe1
				interface gig0/0
				no shut

				interface gig0/1
				no shut

				connect eth-eth gig0/0 gig0/1

				show connection name eth-eth 

			ce1
				interface gig0/0
				no shut
				ip address 192.168.1.1 255.255.255.0

				show arp
				!pass through the mac from router
			
			ce2
				interface gig0/0
				no shut
				ip address 192.168.1.2 255.255.255.0

	local switching with internetworking
		ethernet to vlan local switching
		atm attachment circuits and local switching
 
		ce1 ----------------|  
							|		
							| pe1	 	
							|					
		ce2 ----------------|

		pe1
			interface gig0/0
			no shut

			interface gig0/1
			no shut

			interface gig0/1.100
			encapsulation dot1q 100

			connect eth-vlan gig0/0 gig0/1.100 internetworking ethernet

			show connection name eth-vlan

		ce1
			interface gig0/0
			no shut
			ip address 192.168.1.1 255.255.255.0
		
		ce2
			interface gig0/0
			no shut
			
			interface gig0/0.200
			encapsulation dot1q 200
			ip address 192.168.1.2 255.255.255.0

	atm attachment circuits and local switching

		segment 1 		segment 2 			internetworking type 		atm pvc encapsulation
		---------------------------------------------------------------------------------------		
		atm pvc 		atm pvc 			n\a 						aal0 (default) 
																		aal5
		---------------------------------------------------------------------------------------
		atm pvc 		ethernet / vlan 	ethernet					aal5snap

											ip							aal5snap (default)
																		aal5mux
		---------------------------------------------------------------------------------------
		atm pvc 		framerelay dlci 	ip 							aal5snap (default)
																		aal5nlpid
****************************
vpls 
	virtual private lan service

	concept
		extended atom service
		virtualize ethernet switches like mac learning and aging, bpdu, spanning-tree ....
		many connection sites over mpls with private lan behavior

		service
			*they are differentiated by mac learning and bpdu processing
 
			transparent lan service (tls) > unqualified learning
				in tls all customer vlans of a layer 2 vpn are treated as if they were in the same broadcast domain
				
				source mac addresses are learned and forwarding entries are populated in the same layer 2 forwarding table regardless of whether they are tagged or untagged
				
				tls also forwards bpdus received from the ce-facing interface to other interfaces without processing as they were directly connected through an ethernet hub

				each customer is independent from vlans mean mac address table on isp is seperated and forward
				
				our bpdu and spanning-tree will be forward on transparent

				isp will be like a hub

				customers vlan tagging doesn't matter to isp

			ethernet virtual connection service (evcs) > qualified learning
				for customers who want to keep a separate broadcast domain for each vlan, evcs is a more appropriate choice
				
				in evcs, the outer vlan tag on the ethernet packet differentiates one customer vlan instance from another
				
				mac addresses of different vlans might overlap with one another, and each vlan has a separate layer 2 forwarding table
				
				bpdu packets from ce routers are dropped or processed at pe routers (behave like switch and spanning-tree aware)
				
				pe routers can provide layer 3 connectivity between vlans using the integrated routing and bridging (irb) capability

				process vlan tags and base this forward on pseudowire

				each vlan has mac address table

				each customer for each vlan on their self organization must have a table

		vfi
			virtual switch

			each vpls set os Offered ny a virtual switch inside pe router

								layer 2 forwarding table
								| 						|
								| 						|
					bridge module ------ virtual ------- emulated lan interface + virtual forwarding instance
						|				 switch  									|
						| 															|
						|															|
			to ce	<	attachment 													pseudowire > to core 
						circuites

		on customer link we have normal mac learning 
		on core network for each user we have one vswitch

		*we have internal routing

		*the bridge module maintaining a forwarding table that maps mac addresses to attachment circuits, it can run spanning-tree protocols on them

		*a vfi maintains a forwarding table that maps mac addresses to pseudowires based on packets received on pseudowires

		components
			attachment circuit
				Port mode
					only sends /accepts tagged or untagged Ethernet packets
				
				802.1Q VLAN or trunk mode
					sends /receives only tagged and native Ethernet packets
				
				Dot1q tunnel mode
					802.1Q tunnel is configured and an access VLAN tag is added to the packet at the ingress tunnel interface and removed at the egress tunnel interface

			packet switched network (psn) tunnels
				in vpls, transport is through mpls lsp

			pseudo wires
				
			auto-discovery (advance vpls)
				mechanism that enables multiple pe routers participating in a vpls domain to find each other automatically. otherwise for 			every vpls instance on a pe, the sp would have to configure the pe with addresses of all other pes in that vpls instance 

				cisco feature navigator 7600 catalyst 6000 ios xe 1000 ios xr 9000 csr 12000

			auto configuration (advance vpls)
				automatically establishes pseudo wires for newly discovered ces

				cisco feature navigator 7600 catalyst 6000 ios xe 1000 ios xr 9000 csr 12000

			vsi (virtual switching instance) or vfi (virtual forwarding instance)
				resembles virtual switches on pe routers

		vpls forwarding and flooding
			when packets arrive on attachment circuits or pseudowires, mac address learning takes place as part of the forwarding process

			vpls employs a flooding process when the virtual switch receives a packet that has an unknown destination mac address

			flooding process also applies to multicast and broadcast packets

			when a packet with an unknown destination mac address arrives on an attachment circuit, it is flooded to all other attachment circuits and all pseudowires that are bound to the virtual switch

			between pe routers wanna flood depends on split-horizon 

			when layer 2 split horizon is enabled on a pseudowire, packets that arrive on this pseudowire are flooded to all attachment circuits, but not a pseudowire
			when layer 2 split horizon is disabled, packets are flooded to all other pseudowires and all attachment circuits that are bound to the virtual switch

			on ce routers over pe routers and on all pseudowires also attachment circuite get flood

		vpls signalling
			based on targeted ldp

			n * (n -1) / 2 ldp sessions and psudowires 

			the pseudowire id field serves the binding of two remotely located entities, such as attachment circuits or vfis
			for vpls, each vpls domain is identified by a globally unique vpn id, which means that you have to provision vfis of the same vpls domain the same vpn id on all participating pe routers

			mac address learning
				from ce router of users we received flooded mac toward pe routers and all pseudowires received the flooded mac base on vc label and mac stored over pe router and mac address tabel on pe router  or pe routers virtual switch detect this vc label is user x or user y
				then select pseudowire of the customer

			mac address withdrawal
				label withdrawal message

				if waanna remove or withdrawal for multihoming conditions need spanning-tree get start and process states to detection faild links and change path if get trouble on starting spanning-tree our topology and connections will be unusfull

	basic topologic models
		full mesh	
			every virtual switch has exactly one pseudowire to every other virtual switch in the same vpls domain

			the loop-free forwarding is guaranteed by enabling layer 2 split horizon on every pseudowire in this topology

			it also eliminates the need to run spanning-tree protocols over the backbone

			each pe has direct and seperated onnection to another pe router

			on pseudowire definition
				neighbor 1.2.3.4 encapsulation mpls no-split-horizon
				!bydefault is enable

		hub and spoke	
			exactly one pe router (each pe router) that is acting as a hub connects all other pe routers that act as spokes in a given vpls domain a hub-and-spoke topology by definition is loop-free so does not need to enable spanning-tree protocols or split horizon on pseudowires

			to provide layer 2 connectivity among the virtual switches on spoke pe routers, the hub pe router must turn off split horizon on the pseudowires

		partial mesh
			To guarantee loop-free forwarding in an arbitrary partial-mesh model, yow need to run spanning-tree protocols on pseudowires throughout the backbone 

			Service providers typically do not deploy a partial-mesh model because they want to avoid running spanning-tree protocols in the core network

	*if have 50 pe routers on full mesh topology need 2500 pseudowire for each customer so this is worst condition
		here we have hierarchical vpls mode help us to decrease connections
		contain top tiers on full mesh and bottom tiers 
		top tiers also called as network pe (network facing pe or npe and mpls core)
		bottom tiers called users pe (user facing pe or upe and mpls access)

		for each customer must make pseudowire connection on bottom tiers then from bottom tiers have conenction toward top tiers 

		top tiers have full mesh connection and seperated pseudowire per customer and has no loop

		on top tiers must enable split-horizon and disable spanning-tree
		on bottom tiers connections must disable split-horizon
		if received from top to bottom or opposite of this direction, just happen on one way

	hierarchical vpls
		with mpls access network
			to ensure loop-free forwarding
				n-pe router must enable split horizon on all pseudowires that connect to other n-pe routers (top)
				
				disable split horizon on all pseudowires that connect to u-pe routers (between top and bottom)
				
				on an n-pe router, packets are forwarded to other pseudowires only if they arrive on a pseudowire that connects a u-pe router
				
				packets that arrive on a pseudowire that connects an n-pe router can be forwarded to pseudowires that connect to u-pe routers only

		with qinq access network
			per vlan pseudowire and connections
			less signaling
			between top and bottom must use ethernet

			has double tagging

			n-pe router forwards packets to pseudowires that connect to other n-pe routers only if they arrive on qinq tunnels that connect to u-pe routers

	*can deploy VPLS in an inter-autonomous system (AS) using a hierarchical model
		In their simplest form, peering VPLS PE routers of different administrative domains operate in such a fashion that treats itself as an N-PE	router, and the peering PE as a U-PE router in the hierarchical model

	redundancy
		pe router can still be a single point of failure for attached u-pe routers each u-pe can connect to multiple n-pe routers through redundant pseudowires or qinq tunnels (multihoming)
		
		layer 2 split horizon alone is no longer sufficient for providing loop-free forwarding you need to enable spanning-tree protocols between u-pe and n-pe routers
		
		when a u-pe router multihomes with n-pe routers, you must enable spanning-tree protocols on the u-pe router for all the pseudowires or	qinq tunnels that exist between the u-pe and n-pe routers
		
		however, an n-pe router can choose whether to participate in spanning-tree protocols
			if it does, it behaves like an ethernet bridge that exchanges and processes bpdus with u-pe and other n-pe routers of the same island
			if it does not, it acts as an ethernet hub that simply relays bpdus without processing

	config
		configuring attachment circuit (pe connect to ce)
			configuring access mode
				pe1
					interface gig0/0
					no ip address
					switchport
					switchport access vlan 2
					switchport mode access

			configuring trunk mode
				interface gig0/0
				no ip address
				switchport
				switchport trunk encapsulation dotlg
				switchport trunk allowed vlan 2-10
				switchport mode trunk

				*per tag has different and seperated virtual switch
				*rarely use this

			configuring dot1q-tunnel mode
				interface gig0/0
				no ip address
				switchport
				switchport access vlan 2
				switchport mode dot1q-tunnel

				*usually use this
				*per service provider vlan tag has one virtual switch and pass through customers vlans into the this virtual switch 

		configuring vfi (per customer has virtual switch and vlan base service must create seperated virtual switch for customers vlan)
			12 vfi blue manual
			vpn id 100
			neighbor 10.10.10.2 encapsulation mpls
			neighbor 10.10.10.3 encapsulation mpls
			neighbor 10.10.10.4 encapsulation mpls
			!conenct to which

			*has auto discovery and auto config features that is not on all devices like carrier cisco, these auto options are advance concepts we just use manual cause we see on common os and service provider devices
			
			*every one need negotiate with this must have same vpn id
			*like atom use same vc id
			
			*decrease config use hierarchical method

		associating attachment circuits to the vfi (mapping attachment circuits to virtual switch)
			interface vlan 2
			xconnect vfi blue

		------------------------------------
		example 1
			pe1
				vlan 100
				state active

				vlan 200
				state active

				interface gig 4/1
				switchport
				switchport access vlan 100
				switchport mode access
				!ce1 site a

				interface gig 4/2
				switchport
				switchport trunk encapsulation dot1q
				switchport trunk allowed vlan 200
				switchport mode trunk
				!ce1 site b
				!on trunk and this condition per customers vlan must define virtual switch

				12 vfi cust_a manual
				vpn id 100
				neighbor 10.10.10.102 encapsulation mpls
				neighbor 10.10.10.103 encapsulation mpls

				12 vfi cust_b
				vpn id 200
				neighbor 10.10.10.102 encapsulation mpls
				neighbor 10.10.10.103 encapsulation mpls

				interface vlan 100
				xconnect vfi cust_a
				interface vlan 200
				xconnect vfi cust_b

				show mpls l2transport vc 100 details
				!show 2 pseudowire

				show mac-address-table vlan 100
				show mac-address-table vlan 200
				!show mac of peer ce routers not pe routers
		------------------------------------
		example 2
			pe1
				mpls label protocol ldp
				mpls ldp logging neighbor-changes
				mpls ldp router-id Loopback0
				
				12 vfi 12vpn manual
				vpn id 1
				neighbor 10.0.0.2 encapsulation mpls
				neighbor 10.0.0.3 enca>sulation mpls
				neighbor 10.0.0.4 encapsulation mpls
				
				interface Loopbacko
				ip address 10.0.0.1 255.255.255.255

				interface POS3/1
				ip address 10.0.1.1 255.255.255.252
				mpls ip
				
				interface gig4/2
				no ip address
				switchport
				switchport access vlan 2
				switchport mode dotiq-tunnel
				
				interface Vlan2
				no ip address
				xconnect vfi 12vpn

				show vfi l2vpn
				show mpls l2transport vc
				show mac-address-table vlan 2
				show mac-address-table vlan 4
				show mac-address-table vlan 8
				show mac-address-table vlan 10

			pe2
				mpls label protocol ldp
				mpls ldp logging neighbor-changes
				mpls ldp router-id Loopback0
				
				12 vfi 12vpn manual
				vpn id 1
				neighbor 10.0.0.1 encapsulation mpls
				neighbor 10.0.0.3 encapsulation mpls
				neighbor 10.0.0.4 encapsulation mpls
				
				interface Loopback0
				ip address 10.0.0.2 255.255.255.255

				interface POS3/1
				ip address 10.0.2.1 255.255.255.252
				mpls ip
				
				interface gig4/2
				no ip address
				switchport
				switchport trunk encapsulation dotiq
				switchport trunk allowed vlan 4
				switchport mode trunk

				interface Vlan4
				no ip address
				xconnect vfi 12vpn

			pe3
				mpls label protocol 1dp
				mpls ldp logging neighbor-changes
				mpls ldp router-id Loopbacko
				
				12 vfi 12vpn manual
				vpn id 1
				neighbor 10.0.0.1 encapsulation mpls
				neighbor 10.0.0.2 encapsulation mpls
				neighbor 10.0.0.4 encapsulation mpls
				
				interface Loopbacko
				ip address 10.0.0.3 255.255.255.255

				interface POS3/1
				ip address 10.0.3.1 255.255.255.252
				mpls ip

				interface gig4/2
				no ip address
				switchport
				switchport access vlan 8
				switchport mode access

				interface Vlan8
				no ip address
				xconnect vfi 12vpn

			pe4
				mpls label protocol 1dp
				mpls ldp logging neighbor-changes
				mpls ldp router-id Loopbacko
				
				12 vfi 12vpn manual
				vpn id 1
				neighbor 10.0.0.1 encapsulation mpls
				neighbor 10.0.0.2 encapsulation mpls
				neighbor 10.0.0.3 encapsulation mpls

				interface Loopbacko
				ip address 10.0.0.4 255.255.255.255

				interface POS3/1
				ip address 10.0.4.1 255.255.255.252
				mpls ip

				interface gig4/2
				no ip address
				switchport
				switchport trunk encapsulation dotig
				switchport trunk allowed vlan 10
				switchport mode trunk

				interface Vlan10
				no ip address
				xconnect vfi 12vpn
		------------------------------------
		per vlan mac address limiting
			pe1
				mac-address-table limit vlan 2 maximum 1000
				!prevent mac flooding

				show mac-address-table limit vlan 2 
		------------------------------------
		qos
			pe1
				class-map match-all all-traffic
				match any

				policy-map vpls-policy
				class all-traffic
				shape average 1000000 4000 400

				interface Vlan2
				no ip address
				xconnect vfi 12vpn
				service-policy output vpls-policy

				show policy-map interface vlan 2

				*marking and classificatin on vpn layer2 on attachment circuite will be on layer 2 not layer 3
		------------------------------------
		layer 2 protocol tunneling
			allows layer 2 pdus, s cdp, stp, and vtp, to be tunneled through an ethernet-switched network
				without layer 2 protocol tunneling, layer 2	switchport interfaces drop stp and vtp packets and process cdp packets

				pe1
					interface gig 4/2
					12protocol-tunnel cdp
			
				pe2
					interface gig 4/2
					l2protocol-tunnel cdp

				*backdoor on ce routers must enable spanning-tree to be transparent over isp
					disjointed spanning tree domains do not lead to potential forwarding loops because of the use of Layer 2 split horizon in the service provider network 
					if the customer sites have backdoor links, it is imperative that you have a single spanning-tree domain for the VPLS customer to avoid forwarding loops in the customer network

					pe1
						interface gig 4/2
						12protocol-tunnel stp

						show spanning-tree vlan 2
					
					pe2
						interface gig 4/2
						l2protocol-tunnel stp

						show spanning-tree vlan 2
		------------------------------------
		multihoming (redundancy)
			ce routers connected to 2 pe routers or bottom tiers connect with many links to many top tiers
			must have spanning-tree 
			need make transparent on bpdu packets

			if have many ce routers that connected to many pe routers better run spanning-tree on ce and forward as transparent over isp network

			if were on bottom and top tiers we had spanning-tree over these so must been as transparent and top tiers must have no spanning-tree

			u-pe1 (bottom tiers)
				spanning-tree mode mst
				
				spanning-tree mst configuration
				name mst-1
				revision 1
				instance 1 vlan 2
				
				vlan dotiq tag native
				
				interface gig0/1
				switchport trunk encapsulation dot1q
				switchport trunk native vlan 200
				switchport trunk allowed vlan 2,200
				switchport mode trunk
				no ip address
				
				interface gig0/2
				switchport trunk encapsulation dot1q
				switchport trunk native vlan 200
				switchport trunk allowed vlan 2,200
				switchport mode trunk
				no ip address

			u-pe2 (bottom tiers)
				spanning-tree mode mst
				
				spanning-tree mst configuration
				name mst-2
				revision 1
				instance 1 vlan 2
				
				vlan dotiq tag native
				
				interface gig0/1
				switchport trunk encapsulation dot1q
				switchport trunk native vlan 400
				switchport trunk allowed vlan 2,400
				switchport mode trunk
				no ip address
				
				interface gig/2
				switchport trunk encapsulation dot1q
				switchport trunk native vlan 400
				switchport trunk allowed vlan 2,400
				switchport mode trunk
				no ip address

			n-pe1 (top tiers)
				mpls label protocol ldp
				mpls ldp router-id loop 0
				
				l2 vfi l2vpn manual
				vpn id 1
				neighbor 10.0.0.2 encapsulation mpls
				neighbor 10.0.0.3 encapsulation mpls
				neighbor 10.0.0.4 encapsulation mpls
				
				l2 vfi mst-1 manual
				vpn id 1001
				neighbor 10.0.0.2 encapsulation mpls
				
				l2 vfi mst-2 manual
				vpn id 2001
				neighbor 10.0.0.2 encapsulation mpls
				
				no spanning-tree vlan 2,200,400
				
				vlan dot1q tag native
				
				interface loop 0
				ip address 10.0.0.1 255.255.255.255
				
				interface pos3/1
				ip address 10.0.1.1 255.255.255.252
				mpls ip

				interface gig0/2
				no ip address
				no keepalive
				switchport
				switchport trunk encapsulation dotiq
				switchport trunk native vlan 200
				switchport trunk allowed vlan 2,200
				switchport mode trunk
				l2protocol-tunnel stp
				
				interface gig0/3
				no ip address
				switchport
				switchport trunk encapsulation dotiq
				switchport trunk native vlan 400
				switchport trunk allowed vlan 2,400
				switchport mode trunk
				l2protocol-tunnel stp
				no cdp enable

				interface vlan 2
				no ip address
				xconnect vfi l2vpn

				interface vian 200
				no ip address
				xconnect vfi mst-1

				interface vlan 400
				no ip address

				show spanning-tree mst 1

			n-pe2 (top tiers)
				mpls label protocol ldp
				mpls ldp router-id loop 0
				
				l2 vfi l2vpn manual
				vpn id 1
				neighbor 10.0.0.1 encapsulation mpls
				neighbor 10.0.0.3 encapsulation mpls
				neighbor 10.0.0.4 encapsulation mpls
				
				l2 vfi mst-1 manual
				vpn id 1001
				neighbor 10.0.0.2 encapsulation mpls
				
				l2 vfi mst-2 manual
				vpn id 2001
				neighbor 10.0.0.1 encapsulation mpls
				
				no spanning-tree vlan 2,200,400
				
				vlan dot1q tag native
				
				interface loop 0
				ip address 10.0.0.2 255.255.255.255
				
				interface pos3/1
				ip address 10.0.2.1 255.255.255.252
				mpls ip

				interface gig0/3
				no ip address
				no keepalive
				switchport
				switchport trunk encapsulation dotiq
				switchport trunk native vlan 200
				switchport trunk allowed vlan 2,200
				switchport mode trunk
				l2protocol-tunnel stp
				
				interface gig0/2
				no ip address
				switchport
				switchport trunk encapsulation dotiq
				switchport trunk native vlan 400
				switchport trunk allowed vlan 2,400
				switchport mode trunk
				l2protocol-tunnel stp
				no cdp enable

				interface vlan 2
				no ip address
				xconnect vfi l2vpn

				interface vian 200
				no ip address
				xconnect vfi mst-1

				interface vlan 400
				no ip address

				show spanning-tree mst 1
