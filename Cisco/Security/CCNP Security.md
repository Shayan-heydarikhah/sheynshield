CCNP Security
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	security has dimensions :
		1 : confidentiality > encryption
		2 : integrity > hashing

		hmac > hash message authentication code 
			has same key this is the confilict reason
	
			deffihelman : same key which encrypt data 
	
		why buy hardware ? assic co processors
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	History :
		pix was a product from cisco hasn't web vpn so buy altiga company that make web vpns
		pix + altiga > asa

			asa instands for adaptive security appliance for some reasons cisco buy source fire company at 2013 and release fire power product
			l2 > l4 statefull fw + scaleable  + cgnat acl + routing + application inspection

		asa x > asa + source fire 
			source fire was the best company in ips generating 

		firepower > asa x + snort
			ftd (firepower threat defence)
			fdm (firepower device manager)
			fmc (firepower management center)(v6.66)
				if use thi we can set :
					correlation 
					ha
					central management

						beetwen ftd and fmc we have sftunnel on port 8305 tcp 

			snfc (securing network with cisco firepower) 

				ngips + threat +centric + avc + url filtering + amp cisco + clamav

				9300 series was 3 ru (rack unit) , security module with custering model and one supervisior (su has outband management that if our device get crash we can fix it)+ many cards for nic
				third party (radware + ddos) protection ( 4100 (1 ru) + 9300)

		in ftd we have pre co-processor  :
			smart nic > nat and rewrite 
			crypto accelerator > ipsec + encryption
			npu > droping packets

		breakout cable is aggregating model in ports that aggregate 4 * 10 g  >  1 (40 g)

		fxos : handel sup and another parts

		if our ftd turn off and enable fail to option we can use ips and transfer traffics
		we can add firepower on isr cisco
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	ASA:
		each interface must have these values ;
			1 > nameif ( interface name)
				case sensetive
			2 > security level
					we can set values on interfaces if set same value make them isolate to see each other and higher level can goes to lower level but can't do it upsite down use establishe and syn-ack method
			3 > ip address

		we have asa module on switches 6500

		default route on asa :
			route outside 0 0 10.1.102.2

		access-list out-to-in permit icmp any any echo reply (just return replies)
		access-group out-to-in in interface outside

		icmp is stateless 
		telnet is statefull

		lazy mode for acl config :
			object-group network mgmt-host
			network-object host 00000
			exit

			object-group service telnet-ssh tcp
			port-object eq telnet
			port-object eq ssh
			exit

			access-list outside-in permit tcp object-group mgmt-host
			host 1.1.1.1 object-group telnet-ssh

		keychain:
			key chain hasan
			key 1 
			key string 123
			accept-timelife
			send-lifetime
	
			do sho key chain
	
		 for eigrp on asa we use netmask

		 on router :
		 	router eigrp 1
		 	network 10.1.104.0 255.255.255.0
		 	interface fas 0/1
		 	ip authentication mode eigrp 1 md5
		 	ip authentication key-chain eigrp 1 hasan

		 on asa :
		 	interface fas 0/0
		 	authentication mode eigrp 1 md5
		 	authentication key eigrp 1 123 key-id 1

		 	if key name was different thats ok
		 	if key first key be different on eigrp key chain we can't make neighboring
		 	but in ospf check hole chain and sync with a key then make neighboring

		 	on asa making second key means key 2 will replace with key 1

		 	ofpf authentication with keychain:
		 		ip ospf authentication messag-digest
		 		ip ospf messag-digest key 1 md5 123

		 		on router and ospf are same

		asdm image :
			int g0/0
			no sh
			ip address 192.168.1.1 /24
			nameif inside

			do sh flash

			copy tftp: flash:
			http server enable
			http 0 0 inside
			asdm image flash:
			ssh 0 0 inside
			domain-name sematec
			crypto key gen rsa mod 102

			username shayan sec 123
			enable secret 123

			aaa authentication ssl local
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Others :
		ips and application visibility controll (avc) is specific option on ngfw
		if use antivirus and anti spam we could call them utm
	
			why we use gre?
				gre is like ipip but has more 4 bytes on header and we can transfer routing protocols multicasts on this tunnel type
			must secure tunnels with ipsec (esp)

		anti-replay :
			if somebody retransfer our esp packets to us can make much load and attack
			prevent method is esp numbering, if see repeated number packet get drop
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IPSec Concept:
		has 2 phase:
			phase 1 > management traffics 
				we must negotiate on this step:
					isakmp :
						version 1 (ikev1) > tunnel get up with what status
							has problem : for each tunnel connection profile must set seperated values but in ikev2 we set group of values with proposal
						version 2 (ikev2) > data can transfer through the phase 1 (flex vpn use this)(ipsec)
							more secure and faster on gdoi 
							in version 1 send 6 packet but in version 2 sends 4 packets
							assymetric authentication in verson 2
			phase 2
				in eeach version is same but firs phase in each version is different

		default type of tunnels on cisco is gre
	
		int tunnel 0 
		keep alive 3 3 (3 time chck with first value, 3 seconds wait fith second parameter)

		for site to site vpn must use ipsec
		and for ra (remote access) vpn must use ssl-vpn
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Site-Site VPN (tunnelless) Static Crypto Map :
		we can't run routing protocols with this

		crypto isakmp policy 1 
		hash sha
		encryption aes
		authentication preshared
		group 5 
		exit

		crypto isakmp key cisco123 address 8.8.8.8 (remote point)
		crypto ipsec transform-set ali esp-aes esp-sha-hmac

			esp (encapsulation security payload)

		must write discard route to route our traffics with acl classification (just for tunnelless and static crypto map)

		access-list 100 permit ip 192.168.1.0 /24 192.168.2.0 /24

		crepto map ali 10 ipsec isakmp
		set transform-set ali
		match address 100
		set peer 8.8.8.8
		exit
		int fas 0/0
		crypto map ali
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Client Less SSL VPN On ASA :
		secure cconnections from wan to lan 
		simple example is web sserver connection
		disadvantage is url monitoring
		has limited access
		ftp http https cifs

		differences with cisco any connect is (these are anyconnect attributes):
			dtls : udp 510 (fast)
			rfc 6344
			full tunnel on anyconnect(you are in lan)

			asa :
				web vpn
				enable outside (on witch interface?)
				group-policy farzad internal
				group-policy farzad attribute
				vpn-tunnel-protocol ssl-clientless
                vpn-tunnel-protocol ikev2
				banner value "salam khobi"
				username farzad password 123
				username farzad attribute
				vpn-group-policy farzad
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Anyconnect With Firepower :
		prerequists :
			ip pool
			users
			ssl certificate
			anyconnect image

			object > object management > address pool > ip pool

			must upload any connect images for users on ftd

			object > image > vpn > anyconnect file
				eport controll license
				strong encryption

			device > vpn > remote access (wizard)

			usaully use ssl but ipsec can be usefull

			must set dns on first tab (general)
			last option is splite tunnel must set that on tunnel network on specific below

			in 2 steps:
				group policy sends parameters to client on vpn media (must splite tunnel)

			must define outgoing interface and certificates

			on evaluate license we can't add export controll

			pptp is faster than l2tp and has internal encryption
			l2tp is slower than pptp and don't have internal encryption +ipsec (l3)

			anyconnect dart is like show tech cisco send logs to cisco

			on anyconnect we have local policy if our strikt cert trust get false ,change permission deny 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	NHRP And DMVPN :
		works like ip to ip and arp
		head quarter is nhrp server (next hop resolution protocol)
		nhrp database > works with some ip publics
		dmvpn applicated on intranet

		has versions : 1 , 2 , 3

		gre is point to point and must set keep-alive + tunnel (source + destination)
		mgre is point to multipoint and have not set keep-alive just set tunnel source

		R1 or hub or nhs :
			interface tunnel 0
			tunnel source 9.9.9.9
			tunnel mode gre multipoint
			ip address 10.10.10.1 /30
			ip summary-address eigrp 1 192.168.1.0 (with this our routing table get smaller)
			ip nhrp network-id 1 (when we have 2 hub must set this)
	
			ip nhrp map multicast dynamic (register clients with routing protocols model)
			no ip splite horizen eigrp 1 
			no ip nexthop self eigrp 1

			ip nhrp redirect (define better path to access instead of hub and spoke)
			ip nhrp shortcut (works with shorter values in cef)

			ip nhrp holdtime

			router eigrp 1
			network 10.10.10.0
			network 192.168.1.0 
			
				resolution request :
					here if we asked specific route our request goes to hub then goes to every where first packet will be transfer others use shortcut and redirects

			ip sec with dmvpn :
				crypto isakmp policy 1
				authentication preshared key
				hash sha
				group 5
				encryption aes
				exit

				crypto isakmp key 123 address 0.0.0.0
				crypto ipsec transform-set ali esp-sha-hmac esp-aes

					esp > has encryption
					aeh > hasn't encryption

					gre add 4 byte and thse commands add 20 byte to header we must get them lower wiyh:
				
				mode transparent

				crypto ipsec profile ali
				set transform-set ali

				int tunnel 0
				tunnel protection ipsec profile ali

				add to others

		Rx or spokes or nhc :
			interface tunnel 0 
			tunnel source 8.8.8.8
			tunnel mode gre multipoint (if set gre ip this make our topology hub and spoke)
			ip address 10.10.10.2 /30
			ip nhrp network-id 1
			ip nhrp nhs 10.10.10.1 (our db is on which range 10)
			ip nhrp map 10.10.10 9.9.9.9 (define range 10 be readable from wich public gateway)
			ip nhrp multicast 9.9.9.9 (convert unicast to multicast and solve problem for advertise routing protocols)
			ip nhrp redirect (define better path to access instead of hub and spoke)
			ip nhrp shortcut (works with shorter values in cef)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	GETVPN :
		doesn't have tunnel works on mpls and implicate in reachable places that need security
		doesn't work on internet
		has a key server
		group member include some clients cant be key server

		on key server :
			crypto isakmp policy 1
			authentication preshared
			hash sha
			group 5
			encryption aes
			exit

			crypto isakmp key 123 add 0.0.0.0
			crypto ipsec transform-set ali esp-aes esp-sha-hmac

				must classificate traffics with acl

			access-list 100 permit ip 192..168.0.0 /24 192.168.0.0 /24 (each client in this range has access)

			int tunn 0
			crypto ipsec profile ali
			set transform-set ali

		gdoi > group domain of interpation 
			udp 848
			getvpn protocol
			key management

			crypto gdo group abc
			identity number 100 (must be same)
			server local
			sa ipsec 10 (sa stands or security associate)
			match add ipv4 100
			profile ali
			exit

			add ipv4 10.0.0.17 (listen keyserv on wich ip)
			exit

		on group membr :
			crypto isakmp policy 10
			authentication preshared
			group 5
			encryption aes
			hash sha
			exit

			crypto isakmp key 123 add 10.0.0.17
			crypto map ali 10 gdoi
			set group abc

			int fas 0/0
			crypto map ali
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IKEV 1 + SVTI :
		int tunnel 0
		tunnel source 9.9.9.9
		tunnel destination 8.8.8.8
		ip address 10.10.10.1
		tunnel mode ipsec ipv4 (native ipsec)

		crypto isakmp policy1
		authentication preshared
		hash sha
		encryption aes
		group 5
		exit

		crypto isakmp key 123 address 8.8.8.8
		crypto ipsec transform-set ali aes-esp esp-sha-hmac
		exit

			transform-set : must set on profile to be usable on tunnel interface

		crypto ipsec profile ali
		set transform-set ali
		exit

		int tunnel 0
		tunnel protection ip sec profile ali

			here we don't need acl but neet route

		ip route 192..168.1.0 /24 tunnel 0

			cause ptp mode tunnels we can set and run routing protocols

		show crypto session
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IKEV 2 + SVTI:
		r1 :
			crypto ikev2 proposal a
			integrity sha,md5, .....
			encryption 3des,aes,des,...
			group 5,14,..
			exit

			crypto ikev2 policy b
			proposal a

			crypto ikev2 keyring c
			peer hasan-router
			identity address 8.8.8.8 (for dmvp must set 0.0.0.0)
			preshared-key yazeynab
				or
			preshared-key remote ...
			preshared-key local ...

			crypto ikev2 profile d
			match address local 9.9.9.9
			match identity remote address 8.8.8.8 (for vrf how can detect)
			authentication local preshared
			authentication remote preshared
			keyring c
			exit

			crypto ipsec transform-set ali esp-aes esp-sha-hmac

			crypto ipsec profile e
			set transform-set ali
			set ikev-profile d

			int tunnel 0
			ip address 10.10.10.1 /30
			tunnel source 9.9.9.9
			tunnel destination 8.8.8.8

			tunnel mode ipsec ipv4
			tunnel protection ipsec profile e

			ip route 192.168.1.0 /24 10.10.10.2 /30

			show crypto ikev2 sa (security associative)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IKEV 1 + Crypto Map
		router and asa
		r1:
			crypto isakmp policy 1 
			authentication preshared
			encryption aes
			hash sha
			group 5
			exit

			crypto isakmp key 123 address 9.9.9.9
			crypto ipsec transform-set ali esp-aes esp-sha-hmac
			exit

			access-list 100 permit 192.168.0.0 /24 192.168.0.0 /24

			crypto-map ali 10 ipsec isakmp
			set peer 9.9.9.9
			set transform-set ali
			match address 100

			interface g 0/0
			crypto map ali

		asa :
			crypto ikev1 enable outside
			crypto ikev1 policy 1
			authentication preshared
			encryption aes
			hash sha
			group 5
			exit

			must create tunnel groups and set remote side name

			tunnel-group 8.8.8.8 type ipsec-l2l
			tunnel-group 8.8.8.8 ipsec-attribute
			ikev1 preshare-key 123
			exit

			crypto ipsec ikev1 transfer ali esp-aes esp-sha-hmac

			access-list 100 permit ip 192.168.0.0 /24 192.168.0.0 /24

			crypto map ali 10 match address 100
			crypto map ali 10  peer 8.8.8.8
			crypto map ali 10  set ikev1 transform-set ali
			crypto map ali 10  interface outside
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IKEV 2 + Crypto Map:
		r1 :
			crypto ikev2 proposal a
			integrity sha,md5, .....
			encryption 3des,aes,des,...
			group 5,14,..
			exit

			crypto ikev2 policy b
			proposal a
	
				crypto ikev2 keyring c
				peer hasan-router
				identity address 8.8.8.8 (for dmvp must set 0.0.0.0)
				preshared-key yazeynab
					or
				preshared-key remote ...
				preshared-key local ...
	
				crypto ikev2 profile d
				match address local 9.9.9.9
				match address 8.8.8.8 (for vrf how can detect)
				authentication local preshared
				authentication remote preshared
				keyring c
				exit
	
				crypto ipsec transform-set ali esp-aes esp-sha-hmac
	
				access-list 100 permit 192.168.0.0 /24 192.168.0.0 /24
	
				crypto map ali 10 ipsec isakmp
				set peer 9.9.9.9
				set transform-set ali
				set ikev2-profile d
				match address 100
				exit
	
				int fas 0/0
				crypto map ali

		asa :
			crypto ikev2 enable outside
			crypto ikev2 policy 1
			integrity sha
			encryption aes-256
			group 5
			exit

				must create tunnel groups and set remote side name

			tunnel-group 8.8.8.8 type ipsec-l2l
			tunnel-group 8.8.8.8 ipsec-attribute
			ikev2 local-authentication preshare-key 123
			exit

			crypto ipsec ikev2 ipsec-profile ali
			protocol esp integrity sha 
			protocol encryption aes
			exit

			access-list 100 permit ip 192.168.0.0 /24 192.168.0.0 /24

			crypto map ali 10 match address 100
			crypto map ali 10  set peer 8.8.8.8
			crypto map ali 10  set ikev2 ipsec-profile ali
			crypto map ali 10  interface outside
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	FirePower Inside :
		adding device :
			policy

		on ftd cli we can see managers with #show managers

		physical management interface use for manage and diagnose

		how use web ui :
			configure management address 192.168.10.1 123

			must set ntp and then set management ip add

		if fmc can't detect ftd device must :
			ping system 0.0.0.0
			fmc > device > device manager > edit > set ip address
			in firepower must set name and zone

			if management has overlap or duplicate in network , it's ok

		for routing protocols must goes to :
			ospf :
				fmc > device > device manager > edit > routing > ospf 
					for authentication must goes to interface tab

			bgp :
				fmc > device > device manager > edit > routing > bgp
					general setting (enable)
					for authentication :
						bgp ipv4 > neighbor > advance 

					for clear bgpp from ftd must delete from policy then flex config and deploy and also must set eigrp unconfig all 

			eigrp :
				must use flex config (pbr)
				fmc > devices > flex config > new policy > flex conffig object
					(flex config make asa configs available on ftd with native option)
	
		for licenses must check satelite server (must always go with proxy)
			fmc > system > config > management interface (proxy)

		for routing table view must take ssh to ftd

		fmc > system > monitor

		ngfw deployment:
			1 > transparent : l2 (not recommended cause ips using)(bvi must set ip)
			2 > routed : l3
			3 > irb (itegrated router & bridge) : bridge

				it's better use ips mode because it's transparrent
				if get transparet our configs get lost
				on transparent our multicast traffics can't be reachable

				show firewall (see mode of firewall)
				config firewall transparent (if bee in ha get confoloct)
				config high-availiability disable
				show networ
				config manager add 192.168.60.1 123 (key)

				if launch fmc and set ip address to interfaces must set bridge group interface
				must enable no shut option and name 

		ips interface type :
			passive
			inline
			inline tap
			erspan

		if had pppoe connection what should we do?
			must make bridge our modem and our ftd port take dhcp client
			also we can make ether channel and set ip on this

		config > process > fmc reboot , shutdown

		network discovery:
			passive mode
				scan client side and recommende some policies
				ips and avc works with this option 
				default
			active mode

			policy > network discovery
				0.0.0.0/0 (default)
				its better change to lan range
				must checked host and users checkbox
				we have exclude range ip

				fmc network map > count of net discovery     
		we can migrate our roles and configs from palo alto fortinet and check point to fireppower
		but it's better migrate from asa to fireppower

		object > object management > port 
			we can add many ports and use them like group port

		if have one fmc and many ftd and use one term like lan for an object can make confilict so must resolve this;
			allow override is solution

		syslog (just send logs)
		netflow (analys the logs and traffics)
		stealthwatch (has too much load on device)

		system > configs > vmware tools (shutdown usage)
			audit log (if some one comes in fmc and do some thing mail me)
			change reconciliation
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Platform setting :
		we have one fmc and many many ftd
		need to config special concept on one ftd what should we do?

		devices > Platform setting
			ping 

		for ssh to ftd we can use management but we can add ftd in above path and enable secure shell with this trick
		inside lan and managements can use ssh on device
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Corelation :
			corelation is fmc exclusive make tasks automatic and don't need deploy
			object > intrusion
				sid 718
				content show run
				save as new
	
			policy > intrusion 
				rule > get events > commit change

			if some body telnet to device and take sho run goes to blacklist

			policy > corelation
				rule management :
					rul name : x-corelation
					if select the type of event for this rule an intrusion event occurs
					add condition (just condition)
					add compelex condition (and \or)
					rule sid (is) 1 000 000

				add inactive period
					disable for 1 hour then get active

			must set in acp 

			policy > corelation > policy management
				by default is disable because do'nt need deploy
				after show run we have intrusion event and corelation event
				still do'nt work because do'nt have response
				must edit and click on folder and select response

			policy > alert 
				syslog
				mail
				snmp
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Black List Module :
		remediation > if you sense some thing is happening so take it to null 0
		policy > module 
			add modules

		policy > instance 
			add cisco ios null 0 route

		source fire (snort write and design black list module we download it and upoad to fmc then just need make list and make instances)
		black list destination ip (add)
		si use feed for list and use md5 and farzad.html

		policy corelation > policy management
			response (blacklist)
			we added sid and ...

		object > object management > si (network)
			https://192.168.60.1/farzad.html	(make list)

		policy > acp > si (black list)

		analysis > status
			must wait 5 min
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	HA :
		need license
		we have many concepts in many series and clustering
		must bee same 

		active - active
		active - passive

		state link (link for syncronise data)
		ha link (heart beat link)

			its better use one link for these
			does'nt matter use encryption

		devices > device manager > add (high-availiability)

			device: 
				1 : firepower threat defence (ftd) (select this)
				2 : firepower
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Packet Flow :
		1 - ingress traffic (over flow)!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!(*1)!
			1 > 2 - blocktraffic with prefilter roles							!
		2 - security intelligence (black list and bad ip)						!
		3 - access control l3 + l4 (on this layer and above have ssl)			!
			3 > 4 - trust rules													!
		4 - security intelligence (dns + url) 									!
			4 > 5 - user id 													!
		5 - access control url (l7) 											!
			5 > 6 - rate limit 													!
		6 - file policy (file + malware) 										!
		7 - snort rules 														!
		8 - next hop !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!(1*)!!

		prefilter actions :
			analyzeed tunnel traffic
			blocked tunnel traffic
			fast path (if enabled our acl l3 l4 and si and que will be bypass)

				if had incoming traffic like tunnel get block (tunnel means has source and destination)

			fmc > policy > prefiltering

			clear connection all

			fmc > policies > data purge (clear all traffic datas)
			fmc > policies > access controll 
				has a default value must change it and set prefilter policy priority on our mind

			it's beeter use prefilter to deny some things at the begining of the packet flow
			it's normal filter with simple condition like ip and port
			if need url filtering must goes to trust rules 

		security intelligence :
			nee license
			250 person who works in talos
			talos include cisco and snort team
			detect bad ip addresses
			put in db
			our ftd has internal d each 2 hourse get update with talos
			we can find this optio in poicies and access controll policy
			we have many ways to define source :
				feed (talos)
				list (manual)
				immediate black list

			object > object management > security intelligence

			policies > access controll (security intelligence or si tab)

			to see logs :
				events > analysis > si events

			benefits of si is bad ip black list in immediately
			see block and white listmust goes to:
				policy > access controll and network discovery
					in  si tab 	

			in object management and si section must define list , feed , immediately black list
			feed is third party 
				corelation blacklist in ips make event generating and do special task like wrong password in rdp and counting this
				useable with snort ips roles
			feed frequency
				can check our bad ip db less than 2 hours but need md5 url 
				to fetching data from url need make hash md5 
			with feed we can set another sources

			si is resource intensive

			policy > access controll > security intelligence (here we have dns)

		bright cloud is a collection 

		policy > dns
			default dns policy :
				white list
				black list

			must apply our policy and add rule on dns tab and add list
			then set action to sinkhole

			actions :
				don't block (some time need change talos policy)
				domain not found (sounded drop)
				drop (silent drop)
				sinkhole (dns spoofing)

			policy > access controll policy > security intelligence > dns role
				here must set with access controll policy and si and dns policy

			object > object management > security intelligence > sinkhole
				ipv 6 > ::1
				peyvandha 10.10.34.36

			events :
				analysis > si events

		rate limit :
			it's better use when we have trust action
			qos in fire power doesn't have effects
			pollicing (ommit)
			shaping (queue)

			device > qos

			we can't add more than 32

		fast path :
			don't need rate limit

		url filter :
			need license
			has categories
			trust level:
				high risk 1-20
				suspicious 21-40
				moderated risk 41 60
				low risk 61-80
				trust worthly 81-100

			use bright cloud
			has 2 type db :
				20 bilions url (3 or 4 gig and upper ram)
				1 bilions url

			ingress traffic checks these steps:
				ftd local db
				fmc local db
				cisco collective security intelligence (ccsi)
					ccsi :
						setting > integration > query cisco cloud
				uncategorized

			ulr filter and si diffrences :
				si > url > bad url > talos > update
				url filter > manual > bright cloud > category

		application filter :
			different with url filtering
			detailes are important

			object > object management > appicaton control

			policies > access control policy > application

			it's better set  relevence low and risk high
			application contro works with open app id not port of the applicationss

			if we need block psiphone activity in our lan must make ssl connection

				object > object management > pki > internal ca (generate ca)
				policy > ssl > action (decryption - resign) with (ca name) > interface definition
				then must set our ssl on acp and network discovery

		must set ips and file control at the end of the roles and blocking must be at the begining packet flow

			file control :
				for example no body has permission to download mp3
				dlp (data leak prevention)
				advance malware protection
				amp license

				policies > malware & files 
					actions :
						files :
							detect file
							block files

						application protocol : any or smb
						direction : any or upload or download
						file type
	
						now must set our file policy on acp
	
						policy > acp (new role)
							action (allow)
							on inspection tab we have file policy and must set option
	
				file policy take effect when our traffic flow through the firepower and never use network discovery mechanism to detect files
					
					analysis > file event
						in event view or logs just see netbios name
						destination smb port tscp 445
						source port random
	
						malware :
							malware cloud lookup
							block malware
								store file (spero analys for msexe (just analys microsoft exe files use signature))
								
								capacity handling depends on send files count to amp or threat grid community 200 on 24 hours if need transfering more files must buy license
								if we done 200 and need to send more must wait

								advance option :
									archive and pasword (encryption with rar or zip)
									max depth archive (if 3 time make archive must check inside it)

									hsts (http strict transport security)

								private cloud is offline db from am cloud and thread grid
									amp > amp management
									hybrid cloud
									panacea.threadgrid.com (cisco sandbox)

							local malware analys is clamav

						fire powers advantage is ips (deep packet inspection)
						amp license (advance malware protection)
						cisco did'nt send any files to cloud except:
							when traffic comes to device checking data is truth or not
							1 : ftd inside db
							2 : if not find on ftd db must make sha 256 and send it to fmc 
							3 : if not find on fmc must make sha 256 and send it to amp
							4 : if amp detect this is unknown file and our ftd and fmc deployed on dynamic analysis and thread grid , amp sends file to sandbox or thread grid and ope it in sandbox and analys file then make it sha 256 then advertise it to cloud

						policy > access controll > intrusion
							generate recommendation (with network scanning offer some policy)

						in analysis and malware section we can add them to white list (add to clean list) and black list (add to custom detection list)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IPS & Snort :
		the older versions just had signiture but these days ips has signiture and suspicious behavior detection
		after malware we have snort engine ips
		some roles are system provided on base policy

		policies > intrusion

		all the items set with cvss
			cvss : common vulnerability acanning system
			cve : common vulnerability exposures 
				the code of each exposures in the world
				if be closer to 10 is more dangerous
					connectivity over security (10) (if our cve bee 9 -10 i'll be effect on system) (speed is important)
					balanced security and connectivity (9 or higher)
					security over connectivity (8 or higher) (security is important)
					maximum detection (7/5 or higher)(all the network get down and crash device)

		on ips we have 2 mode :
			event 
			drop & event

		generator id (gid) (define standards)
		snort id (sid)

		standard text rule > gid 1 sid less than 1 000 000
		shared object rule > gid 3
		preprocessor rule > gid except 1-3 (snort hand write)
		local rule > sid more than 1 000 000

		if our events generate with sid x do task x
		cisco has recommendation for ips rules but prerequists are network discovery
			policy > networ discovery 
				active > analysis > network map
				passive > transfering file get analys

		ips rules + snort rules :
			alert (define action matching packet) + tcp (protocols for analysis) + $telnet-server (variable define source ip add) 23 (source port of packet) --> $external-net (vulnerability define destination ip add) any (define destination port)

			this event generate when our traffic wants back

			if some body telnet to our lan and write 3 times wrong password do this ...

			body*
			msg > "protocol-telnet login incorrect"; display message on gui when a rule triggers
			flow > to_client,established; defines the traffic flow for which a rule is applied
			content >login incorrect; searches for specefic content in the packet payload
			metadat >rule set community service telnet;  associates additional data withh a rue for future use
			class type >bad -unknown; categorized a rule as certain type of attacks
			sid >718; maps a rule with a unique number
			rev >16; increase by one whenever a rule is revised

		object > intrusion (rules create rule for ips)
			for create local roles must save as the rules
			directions is important

			rule state & status (show this is disable)
				geneate events

			policy information > commit change (deploy roles on ips)

		policy > acp
			create new rule (allow \ interctive block)
				right click on intrusion policy

		analysis > intrusion > events

		policy > intrusion (edit)
			each variable set has unique concept so use many process
			object > object management > variable set
				$home-net > any destination (edit) -- > $home-net > lan60

		sensetive data detection :
			nap (network analysis policy)
			policy > intrusion > network analysis policy
				must enable in setting
				policy > intrusion > advance setting of ips (edit)
				we can enable sensetive data detection 
				we can use preprocessor rules and define them with our favourite name and set the pattern for dlp :
					patern \d(5) (number with 5 digits)

		tunning false positive :
			1 : disable rule
			2 : suppress
				policy > intrusion > rule > suppress (rule type)
					stop generating some events that not important
			3 : just geneate event
			4 : in acp set intrusion and set allow (not recommended)
			5 : rewrite snort rule
			6 : write pass rule (recommended)
				--> intrusion pass rule (process faster) --> intrusion rule -->
				policies > intrusion > rules
				object > intrusion rules > select rules (pass action must be set)

				must define two same rules and save as one of them and make action pass on it
				with priority of checking the steps pass actions checked first so must set server ip address on rules

		if some body run port scann what shold we do ?
			network analysis policy
			ingress traffic checked with nap then get normalized (means ips and deep inspection happend)
			normalized :
				1 : policy > nap 
					preprocessor must be enable and active (default is active)
					create new or edit
						scada in nap is enable
						setting must show us the portscan and edit it then save it as new policy with new name and select protocols
						portscan id is 122
						then commit change
				2 : policy > intrusion
					create ips rule 
					enable gid 122
					enable events and then commit change
				3 : policy > acp
					allow \ interactive block
					intrusion > ips
				4 : policy > acp
					advance tab section has nap option that must be edit and set on our nap name
					deploy

		for dos we have shun (syn flood) :
			flex config
			policy > network analysis
			setting > rate base attack prevention (edit)
			object > intrusion object rules
				remote desktop > gid 1 sid 21232
			policy > intrusion > rules > generate event of our new ips rule
			commit change
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	SSL Decryption :
		has many tools to use like amp and ips

		policy > ssl > add new (default action)

		if we can't handle the traffic and packets in ssl our device load get too high
		why we decrypt the data?
			if some data or things with ssl validation wasn't valid or expire or revoke or .... what should we do

		security intelligence url (feed)

		some times in some fields we can't use ssl like apple company because all apple services are on range 17.0.0.0/8
		apple pin the ssl

		http v2 :
			parallel use and transmit traffics with google on ssl base

		ssl versions (deprecated) :
			1
			2
			3

		tls version :
			1 (deprecated)
			1/1
			1/2
			1/3 (just in this version we need ssl decryption)
				in url filter and cleartext mode recieve data then encrypt data

			pki : policy key infrastructure use for gainsay

			if client send traffic to server our policies control is like this:
				1 : client send hello (clear text to server)
				2 : server send hello + cypher chain (server certs)
				3 : .....

				do the clients send original server certs to another servers ?

			3way handshake at the begining was tcp procedure then send hello ?
				in hello we have sni
				sni (serve name identification)
					say i have to see ....

			our connectivity in firts steps are asymetric-steps 1-3 but in next steps we have symetric  type connection
			in tls 1/2 :
				server certificate (clear text)
				url filtering (don't need ssl decryption except tls 1/3) :
					works with server certificate
					when send hello we don't filter it so when our traffics come back our filtering take effects 

			on tls 1/3 :
				we have encrypted sever certificate filter
				so our firewalls can't see inside and make policy on them

			firepower at the begining of the decryption use clear text hello
			firepower use aggresive downgrade (this downgrading make tls 1/3 to 1/2 and no one can't works with tls 1/3)
			decrypt-resign
			decrypt-known key
				use inside private keys and outside pricate keys
				like man in the middle job and task

			distinguished name : with this field we can't encrypt some things
			must set block with reset because our connections in another way will be open and has much load on device
			our policies are top\down and if write too much maybe get ambigous

			undycriptable action :
				policy > ssl > dn (add)

				end user notification till use tls 1/3 did'nt happen so must set http respons

			it doesn't effects on hole acp 

			where we don't need run ssl decryption?
				app discovery
				prefilter
				avc
				url filtering
				si (ip , dns)
				doh

			where we need ?
				amp
				ips
				acp micro app (inside the instagram users can't like or comment)

			clients hello :
				initial match  (what should i do?)
				when packet goes to fire power :
					encrypt 
					not encrypt

				if cached data we can open it

				clear text , if inside it self don't have sni what did happen?
					if add another thing insid sni (like decrypt job searc categories) attacker inject another values on sni header
					this is the real reason that reply packets get check with firepower and url filter  in global becuase sni spoofing

			fire power expert space :
				ftd cli:
					cat etc/sf/ssl-tunning.conf
					ssl-allow-match-sni-subject-dn=false
					sftls-max-tcp-tracked=150000
					max-ssl-session=32000
					max-tcp-tracked=50000

			server missmatch = yes
			(block with reset)

			these are for tls 1/3 using in circute and convert it to 1/2
			if send hello in cleartext mode on tls 1/3 and recieve replies from server with encrypteed mode, has confilict

			how enable downgrade :
				system support ssl-client-hello-enabled aggressive-tls 1/3 -downgrade true

			must deploy on acp ti se ssl policy

			it's beeter use cert authority to firepower for defining and advertise to anothers
			self sign certs isn't ok
			we can make microsft cert and define to clients with group policy

			object > object management > pki > internal ca

			cmd : certmgr.msc > trusted root certificate
			if get certs and make openssl on our system we can take private key
			has acceleration
			if use vpn ssl must set ssl or tls on decrypt - known key 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Flex Config & EIGRP & PBR :
		asa options that in fmc is not be like native model
		has some object like asa
		devices > flex config
			add new policy

		append (after fmc policy will performd) <-- fmc policy <-- prepend (before fmc policy performd)

		once > after each time deployment our commands applied
		every time

		EIGRP :
			flex config object :
				name eigrp
					router eigrp 1
					network 192.168.60.1
					no autosu
					
					append

			text object (secret key for eigrp authentication)
			object > object management > flex config object
				it's better make object for networks
			object > object management > network

			we cant make clear text object for eigrp authentication must create object in flex config
				inject secret key

			obect > object management > keychain
			device > device management > routing

		BPR :
			1 : classification
			2 : route map

			object > object management > access list > extended

			object > object management > route map
				create object here and add them to flex config

			object > object management > flex config
				add objects
				route-map $pbr permit 10
				set ip next-hop 9.9.9.9
				int g0/0
				policy-route route-map pbr1

			setting > monitor > ftd > sho..... > advance ...
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	ACP (Access Control Policy) : 
		allow : pass thi step maybe have problem in file control or ips
		trust : pass this step means our traffic is clear and ok and goes to next hop not file contro or ...
		monitor : has log option and counter get increase but different with allow is allow by pass another roles checking and monitor check anothers
		block 
		block with reset : if our tcp drop happend our connection will be open this option is better reset the open connections to close
		interactive block : if block url with policies but accept risk is users side in popup (http respond)(must check dns)
		interactive block with reset

		how our policies get check?
			row > and
			col > or

		sgt (security group tag) :
			we can add tags on layer 2 and manage the trafficsin better situaton
			policy > acp > sgt attribute
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Captive Portal On Firepower :
		realms :
			for read users and identity valus from active directory or eeach directoy service must use this option :
				system > integration > realms
				add ip and user pass

				fqdn > active directory
				name > domain
				basedn > distinguished name> search
					(this used for fmc search index)

				basedn : dc=farzad.dc=.com
				groupdn : dc=farzad.dc=.com

		at the first add realms data then add (active) , ip port and ssl (start tls) , at the end must enable it
		to lear users must edit it and click on user download
		on windows we could install agents but in version 6.6.6 our user agent converts to user servers or identity source

		policy > identity (add)
			passive authentication
			active authentication (captive portal mode)
			none authentication

			policy > acp > add (identity policy)
				users tab set that users mst authenticate

					must add ssl and identity in acp policy

			passive identity connector (user ip mapping)

			need some prerequists like certs and realms

			policy > identity > active authentication
				redirect port (885)
				maximum login attemp
				ssl decryption

					identity as special guest (guest login page)

				we have 2 mode of login:
					popup (http basic)
					page (http response pge)

			object > object management > pki 
				internal cas (fmc internal certificate production)
				internal certs (inter network certificate production from another vendors)

			analysis > user activity

			for log out with recommended base way we should write role on acp and allow it on port 885
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	license:
		base (default is on device)(routing and nat)
		term base licenses :
			thread (ips + si + base)
			tmc (url filtering (c) + base + threat + tm)
			tm (malware + threat + amp + base)
		plr 
		tamc was old version of licenses

		smart license > satelite server > cssm
			cssm (cisco smart software management)(smart account)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	NAT :
		manual nat : write all parameters in a line (not recommended)
		auto nat : object nat
			dynamic nat : overload (pat (mikrotik masquared))
				appropriate when we have ppoe connection and outgoing interface
				we have ip pool
			static nat
				one to one

		in cisco we have bydirectional (2 way) nat and write roles

		fmc > device > nat 
			fire power 
			threat defense

			must define internla and external interfaces then set values on translation tab
				original source (lan side)
				translattion packet or trusted source (outside pool)
					2 way to define :
						destination interface ip 
							ip nat inside source list 1 interface g 0/0 overload
						ip pool that contain one ip address

				show xlate
				clear xlate

			destination nat act first then source nat take effect

		static pat or port forwarding
			fmc > device > nat >auto nat > static type
				translattion source 192.168.60.20 port 3389 translated port 172.16.207.15 port 5000
					if some body search 172.16.207.15:5000 divert him to  192.168.60.20:3389

		manual nat :
			if some body comes with 192.168.60.2 and destination was 9.9.9.9  and destination port was 5000 convert source to 172.16.0.1 and destination 10.0.0.2 nd port 33
			translation :							
				original source 					translated port
				source  192.168.60.20 				172.16.0.1
				destination 9.9.9.9 				10.0.0.1			
				port 5000							33

				must set back route for this nat on next hop or in tunnel

			* its beeter use range instead of network into the defining pool with + button
			if our ip addresses bee in another pool we have confilict
			if add overload in static or dynamic nat we can nat our lan range ip address to one ip in edge device
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	FTD Password Recovery :
		gui and clie
		 cli must make reboot device and write boot:7.0.0 single (version)
		 	the older versions open login page but on v7 must write commands

		 	passwd admin newpass
		 	init6 or restart

		 gui recovery :
		 	sudo su
		 	usertool.pl -p 'admin newpassword'
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	CBAC (context base access controll) :
		R2 :
			access-list 100 permit icmp any any
			access-list 100 permit tcp any any eq 23
			access-list 101 deny ip any any
	
			interface fas 0/11
			ip access group 100 in
	
			interface fas 0/12
			ip access group 101 in
	
				send traffic but not recieve traffic
	
			ip inspect name shayan icmp
			ip inspect name shayan telnet
			interface fas 0/0 
			ip inspect shayan in

				now r3 has ping but r1 don't have

			show ip inspect all
			show ip inspect session 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Private Vlan :
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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	VLAN And MAC ACL:
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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Zone Base Poicy FireWall:
		zone base policy firewall 1
			int gig0/0
			no sh
			ip add 192.168.10.1 255.255.255.0
			int gig0/1
			no sh
			ip add 192.168.20.1 255.255.255.0
		
		1 > create zones:	
			zone security inside
			zone security outside
			
		
		2 > define traffic class	
			access-list 100 permit tcp any any eq 22
			access-list 101 permit tcp any any eq 80
	
			class-map type inspect match-all myssh
			match protocol ssh (or) match access-group 100
				
				for wellknown traffics we don't need add to acl
				match-all > use protocols and acls togheter but, reciever and transfer traffics like web must be in same range
				match-any > if sync with one of them access to them
		
		3 > set classification, then what happend to groups:
			policy-map type inspect in2out
			class type inspect myssh
			inspect
	
				type isnpect means security options, if didn't see this means using qos for collaboration
				nbar serach plz
	
		4 > with wich zones we apply policies:	
			zone-pair security lan2wan secure inside desti outside (we have self term in commands that means destination of packets are this router)
			service-policy type inspect in2out
		
		5 > assign interfaces	
			int g0/0
			zone-member security inside
			in gig0/1
			zone-member security outside
			
			do show policy-map type inspect zone-pair session
		
		zone base policy firewall 2:
			
			class-map type inspect match-any in-out-c
			match access-group 100
			ex
			
			access-list 100 permit ip any any
			class-map type inspect match-all in-dmz-c-icmp
			match access-group 101
			match proto icmp
			ex
			
			access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
			class-map type inspect match-all in-dmz-c-ssh
			match access-group 101
			match proto ssh
			ex
			
			class-map type inspect match-any dmz-out-c
			match proto icmp
			class-map type inspect match-any out-dmz-c
			match proto http,https
			ex
			
			policy-map type inspect in-out-p
			class in-out-c 
			inspect
			ex
			
			policy-map type inspect in-dmz-p
			class in-dmz-c-ssh 
			inspect
			ex
			class in-dmz-c-icmp 
			inspect
			ex
			
			policy-map type inspect dmz-out-p
			class dmz-out-c 
			inspect
			ex
			
			policy-map type inspect out-dmz-p
			class out-dmz-c 
			inspect
			ex
			ex
			
			zone sec inside,outside,dmz
			
			zone-pair sec in-out source inside des outside
			service-policy type inspect in-out-p
			ex
			
			zone-pair sec in-dmz source inside des dmz
			service-policy type inspect in-dmz-p
			ex
			
			zone-pair sec dmz-out source dmz des outside
			service-policy type inspect dmz-out-p
			ex
			
			zone-pair sec out-dmz source outside des dmz
			service-policy type inspect out-dmz-p
			ex
			
			now set to interfaces	
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Devnet :
		nas :
			network as a sservice
			nfv : network function virtualization (nexus 1000 + isr 4326) 
			sdn : software define networking 

		vwaas (cisco virtual wide area application service)

		sdn :
			control plane
				responsible for routing table handeling
			data plane
				routing table exports on each media type
			management plane
				remote controll  with snmp or...

			on each vendor we have these parameters

		we don't need controller in sdn
		we have a central controller for controll plane and has :
			apic (application policy infrastructure control) 
			aci (application centric infrastructure)(api is inside this)

		works on udp

		api (application program infrastructure) :

			sbi (south bound interface or api)
				connection with hardware devices
				use cli , cisco opflex , openflow

			nbi (north bound interafce or api)
				gui , python , java , app


			user <---------------> api
									^
									|	(nbi)
									˅
					(control plane)	sdn (controller)
									^
									|	(sbi)
									˅
									sw1 (data plane)

			net code us :
				our config and structure convert to a code , can recover that in another zone , has versions (git)

			sdn:
				automation 
				abstraction : 
					yang model
						has many protocol like :
							restconf
							netconf
							postman
							rest api
					vendor independent
				vcs (version controll system)

			for data transmission on sdn we use mib (management information base) inside snmp

			yang :
				snmp algorythem
				rfc :
					rest conf : ssh
						xml
						yaml (ansible)
						json
						java script
					net conf : ssl , http
						xml
					gprc : google

				industry (standard)
				vendor (specific)

				ethernet header + ip header + tcp header + data
				(        transparent model             ) (data model)
				rest conf , netcoonf , gprc					yang
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	DHCP :
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
				ip dhcp snooping limit rate 5 (if be 1 goes to error disable)
		
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

			show interface status error-disable

			we can set ip on ios 12.2 and sw 2960 but our routing table works with mac
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	Rate Base Attacks :
		int fast 0/0
		strom-controll action shutdown
		strom-controll action trap
		strom-controll unicats level pps 1
		strom-controll multicast level pps 1
		strom-controll broadcast level pps 1
		strom-controll broadcast level 50
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	COPP (Control Plane Policy) :
		if attackers want attack our router cpu what should we do?
			access-list 100 permit icmp any any
			class-map icmp
			match access-group 100
			policy-map icmp
			class icmp
			policy 8000 conform-action transmit exceed-action drop (if see more than 8k drop)
			exit

			contro-plane
			service policy input icmp
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	SPAN & Remote SPAN :
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

		wich traffic types can analyzed from which ports
		after running span our port status > port up  + line protocol down
			because monitor an mirror is enable all protocols are disable
		we can set ingress traffic
		monitor session 1 destination int f 0/12 ingress vlan 1
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	WCCP :
		transparent
		direct

		wsa :
			interface config
			edit
			1
			comite
			set gateway
			1

			wsa > system > system setup wzard

		r1 :
			show license udi

			access-list 1 permit 192.168.60.5 (wsa ip)
			access-list 100 permit tcp any any eq 80
			access-list 100 permit tcp any any eq 443

			ip wccp version 2
			ip wccp 100 group-list 1 redirect-list 100 (firs 100 is service id and second is acl)
			interface gig0/1 (client side)
			ip wccp 100 redirect in
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	VRF Aware :
		R1 : 
			ip route vrf a 192.168.2.0 /24 10.0.0.2
			ip route vrf a 192.168.1.0 /24 172.16.1.2

				here we have vrf and it's posible to conflict when need to create isakmp phase 1

			crypto isakmp policy 1
			authentication preshared
			hash sha
			group 5
			encryption aes
			exit

			add these on r2

			crypto isakmp key 123 address 10.0.0.2 (wich 10.0.0.2?)

				solution is keyring

			crypto keyring a vrf a
			preshared-key address 10.0.0.2 key 123

			add on r2

			crypto isakmp profile a
			match identity address 10.0.0.2 /24 a
			vrf a
			exit

			svti steps:
				1 : isakmp policy
				2 : keyring + vrf
				3 : isakmp profile
				4 : ipsec
				5 : profile ipsec > transform-set > isakmp profile
				6 : interface tunnel
				7 : protection tunnel > tunnel mode ipsec
				8 : route

			crypto ipsec transform-set ali esp-aes esp-sha-hmac

			svti + vrf aware :
				crypto ipsec profile 
				set transform-set ali
				set isakmp-profile a
				exit

				int tunnle 0
				ip vrf forwarding a
				ip add 11.0.0.1 /8
				tunnel mode ipsec ipv4
				tunnel protection ipsec profile a

				tunnel source 10.0.0.1
				tunnel destination  10.0.0.2
				exit

				ip route vrf a 192.168.2.0 /24 tun 0

				show crypto isakmp sa

			crypto map steps :
				1 : isakmp policy
				2 : keyring + vrf
				3 : isakmp profie
				4 : ipsec
				5 : acl (classification)
				6 : crypto-map
				7 : apply on interface

			crypto map + vrf aware :
				access-list 100 permit ip 192.168.2.0 /24 192.168.1.0 /24
				crypto-map ali 10 ipsec isakmp
				match add 100
				set peer 10.0.0.2
				set transform-set ali
				set isakmp-profile a
				exit

				int fas 0/0
				crypto map ali
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	ISE & AAA :
		authentication	(who want access)
		accounting	(monitor the traffics till specific hour)	
		authorization (access levels)
	
		this is a concept not a protocol 
		use some protocol inside it like tacacs+ and radius
	
		radius :
			standard verson of aaa protocol
			just encrypt passwords
			port 1812/1813
	
		tacacs+ :
			cisco exclusive
			encrypt all data (pass + username + commands on telnet )
			port 49

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

				before send this our ports are off (amber)
	
				we should goes to services.msc and (wired auto config), set on automatic, start
				make dot1x enable on windows
				ncpa.cpl > ether0 > properties > authentication (header) > uncheck 2 check boxs on bottom page (remember and fail) > in setting uncheck verify... > in setting set 	mschap v2 and click on configure button cheched automatic option > go back to authentication (header) and goes to additional > set specify option on user authentic	ation
	
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
					dot1x port-control auto
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

		>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
			ise and aaa farzad heydari 2022:
				ise is like inside firewall
				device access (tacacs +)
				network access (dot1x + radius)
				byod (bring your own device)
				posture	

				has many terms:																		maximum count
					persona : rules 		
					pan : policy administartion node 														2
					psn : policy service node 																50
					mnt : monitoring and troubleshooting 													2
					pxgrid : a platform that secure all devices and multiplexer (platform exchange grid)

						we can launch one ise server (standalone) / distributed roles

				user : admin
				pass : password

				application stop ise
				application start ise 
				show application status ise

				work center > system > persona
				at the end of the page we have enable device admin must enable it

				we can define a range of switches ip and ... with excel

				administartion :
					identity
						identity management > groups 
							end point identity group 
							identities > users
							mab (mac authentication bypass) : printers
					network resources 
						network devices group
						network devices :
							ip 
							name

								add each link or object beetwen ise and ... must set password

					policy > policy sets > 	authorized policy > conditions (attribute)
						network devices (all switches type)
							and
						identity group (admin)

					policy > policy elements > authorization > results 
							permit access \ vlan 10
							profile name (vlan 10)
							common task (vlan)
							idname > vlan number vlan 10

				priority of reading policy is :
					ise then ad


				download able acl (dacl) :
					policy > elements > result > authorization > dacl
						write acl and save them

					policy > elements > result > authorization > profile
						each user comes to vlan 20 add acl under port

				content filtering :
					iron port > asyncos (freebsd)

		>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
				1 ==> join domain(external identity source)
				2 ==> administartion >>> identityy management >>> identity source
				3 ==> policy >>> authentication
				4 ==> policy >>> policy elements >>> results >>> authorization >>> authorize
				5 ==> policy >>> authorization
	
			mab > mac authentication bypass
			must create vlan in local not in remote or ssh
		
			ise on switch :
				vlan 100 ,200
				username shayan sec 123
				int gig0/0
				no sh
				ip add 192.168.4.150 255.255.255.0
				aaa new-model
				radius server ise1
				add ipv4 192.168.4.200 auth-port 1812 acct-port 1813
				key cisco123
				aaa group server radius myradius 
				server name ise1
				aaa authentication login default group myradius local
				aaa authentication dot1x default group myradius local
				aaa authorization network default group myradius local
				dot1x system-auth-control
				
				for detailes:
					radius-server vsa send authentication
					radius-server vsa send accounting
					radius-server attr 6 on-for-login-auth
					radius-server attr 8 include-in-access-req
					radius-server attr 15 access-req include
				
					aaa server radius dynamic-author
					client 192.168.4.200 server-key cisco123
					ip device tracking
				
				host side:
					int gig0/0
					sw host
					auyhenticatin port-control auto
					dot1x pae authentication
				
				on other look:
					aaa new-model 
					radius server ise
					add ipv4 172.16.0.100
					key cisco123
					ex
					aaa authentication login default group myise local
					aaa group server radius myise
					server name ise
					line vty 0 4
					login authentication default
					do test aaa group radius alireza P@ssw0rd! new code 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	NTP Authentication :
		ntp server 10.0.0.1
		ntp authentication
		ntp authentication-key 1 md5 84684648
		ntp trust-key 1
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IOS Resiliance :
		to hide image of router or switches os must set these:
		secure boot-image (don't change)
		secure boot-config (use hash md5 to compare local ios and new os)

			on remote and ssh we cant no... this command(just console)
			real time running config get hide
	
		show bootset	
		verify (cath md5 from ios)

		secure boot-config restore <filename>

		 we can't upgrade ios
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
