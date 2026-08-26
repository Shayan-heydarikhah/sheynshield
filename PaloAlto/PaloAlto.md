PaloAlto
	company established on 2005 in santa clara
	licenses :
		security
		web filter

	series :
		220-800 > small office
		800-3200 > middel office
		5200-7000 > enterprise

	panos works on centos 

	management ip : 192.168.1.1/24
	user : admin
	password : admin

	show interface management
	config
	set deviceconfig system type static
	set deviceconfig system ip-address 1.2.3.4 netmask 1.2.3.4
	commit

	rest factory and maintrancer mode :
		debug system maintrance-mode
		password : MA1NT

	device >
		setup :
			management :
				general
					hostname
					domain
					dns
					login banner
					date time
					time zone
					ssl \ tls
					radius
					gtp or sctp security
					cert expiration check
					panorama
						latitude or longtitude
	
			interfaces :
				trusted host (must set zone to works correctly)
	
			telemntry :
				whole system control
	
		wild fire (zeroday)
			like talos and sandbox
		
		session
		
		hsm

		config audit
			accounting and uncommited configs can reachable

		password profile > password policy

		interfaces
			mgmt port
			serial port

		content id (url filter)

		authentication profile
			attemps
			2fa
			radius

		user define \ identifier
		log setting
		region (geoip)
		
		services
			dns
			proxy
			ntp
			password
			service route config
				with mgmt port manage some tasks automaticaly (all \ custom)
		
		operation
			shutdown\reboot
			custom log
			snmp setup
			config mgmt(history)
				config revision (works like git)

		admin
			can manage user access on role base config

		admin rule
			user login manage and access level to device

		server profile :
			ldap
			user authentication :
				group mapping setting
			snmp server
			http 
			netflow
		trouble shooting (for policy lookup test)

		respond pages

		schedule log export
		ha :
			must connect 2 port :
				1/8 and 1/9 (set type on ha)

			link control
			data control

			setup + control link ha + backup

			election setting (preemptive)
			transport (thernet)
			keepalive
			priority 100
			active\passive setting auto
			link monitoring
	
	in execute bgp we have better performance on paloalto

	network :
		interface :
			tap (sniffer (span))
			virtual wire (bridge and works like transparent firewall)
				just use 2 interfaces and deploye all firewall policies on them
				no process and transfer it
				
				linkstate pass through
					monitor 2 ports and transfer down state to each site

			ha
			l2 > type dmz on zones
			l3
				have ddns

			ipv4
				dhcp
				pppoe
				static

			find lldp in advance option

				in each interface type must check some setting in advance and ethernet info

			to create subinterface must create l3 then in bottom of box show us subinterface option, must tag them

			virtual router must be config and add our interfaces to routing table
				like define vrf and seperate routing tables
				to connect on many virtual router must use virtual wire

				on l3 and general tab we can set values
				ecmp (loadbalancing)
					strict source path
						use for ipsec 
							like glbp use a link for ever 
					hash seed
					bfd (bidirectional fail detection)

				for virtual routers we can use virtual wire and set next hops on static route table on nexhop

				path monitor
					preemtive hold time
					if users use link-1 and suddenly our isp goes down our policies make disable another link

					failor detection 
						any
							if lost one link make switch
						all
							if all cluster get fail make switch
		zone :
			we have various protection model in this box
			zone type must be same with interface type

			zone protection and dos protection
				flood attacks can handle
			user id acl
				depends on ip and make acls on it to give access

		virtual wire :
			like fortigate devices that link a devices port internal

		lldp (enable)

		dhcp
		
		qos (interface definition)
		
		network profile :
			qos profile
			lldp profile

			on this section we can use interface profile and management profile to give access our interfaces

	objects :
		address :
			address and address groups
			ip netmask
			ip range
			ip wold card mask

			(sctp)

		services :
			services and services group

		schedule
		tag
		application :
			group and filters

		security profile :
			licenses :
				antivirus
				url filter
				anti spyware (backdoors + malwares)(strict)
				vulnerability protect
				file block
				data filtering (dlp)
				security profile group

		log forwarding (syslog + snmp)

	policy :
		security :
			rule types :
				universal (defeault)
				interzone (take effects on specific zone)
				intrazone (any src to any des)

			general > name
			source > zone or address or interface
			destination > zone or address or interface
			users
			applications
			services or urls
			actions
			negated (make negative)

			commit

		profile setting :
			qos + security

		nat (must check flow of packet for dnat and snat):
			original packet
				srsc address \ interface
				src zone
				dst zone
				dst address \ interface

			translate packet
				source (bidirectional)
					dynamic ip and port
					translated address
						interface address (masquared)
						static ip
				
				destination
					static ip

			for destination nat and give access from outside to firewaal must set src and dsn zone on external zone

		dos protection :
			aggregate (whole data)
			classified (seperated)

		qos

		authentication :
			loginpage

	wan load balance :
		first must define zones and interfaces
		then add policy to permit transfering packets
		after these must define nat
		network > virtual router (edit default rule)
			router setting (enable ecmp and add interfaces)
				loadbalance mode :
					weighted + round roubin
					balance + round roubin
					ip module
					ip hash

			then define 2 default route in routing table with path monitoring (sla and track)

		must commite setting

	logging
		application command center (acc)
		without license we have no logs

		monitor > logs
		devices > server profile > syslog
		object > log forwarder
		must set on security profile our forwarding logs
		services > server route config (egres from special port)

	upgrade:
		devices > software
		devices > dynamic update

	ipsec :
		network :
			network profile :
				ike crypto
				ipsec crypto
				ike gateway :
					enable nat traversla (nat on ip public)
					advancee > ikev1 exchange mode(main)

			interface (tunnel)
			ipsec tunnels
			virtual routers :
				set static route and egress interfaces on tunnel

		policy :
			security :
				 1 (allow ipsec) :
				 	source > zone vpn
				 	destination > vpn
				 	apllication > ipsec , ike
				 	service > any
				 	permit

				 2 (hq-br) :
				 	source > zone lan
				 	destination > zones lan , vpn
				 	services > all
				 	apllication > all
				 	permit
