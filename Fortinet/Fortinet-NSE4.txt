Fortinet NSE 4
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
General :
		NSE 1: Threat Landscape
		NSE 2: Fortinet Solutions for CyberSecurity
		NSE 3: Detailed info about solutions (Sales)

		NSE 4 professional
			FortiGate Security (policy base)
			FortiGate Infrastructure (network base)

		NSE 5 analyst (at least 2 exam)
			forti analyzer
			forti manager
			forti client EMS
			forti SIEM

		6 specialist (at least 4 exam)
			security advance
				FortiWeb
				FortiMail
				FortiAuthenticator
				FortiDDoS
				FortiNAC
				Integrated and Clud Wireless
				FortiWLC

		7 tshooter (at least 1 exam)
			Advanced Threat Protection
			Enterprise Firewall (FortiGate Tshooting)
			Secure Access
			Cloud Security

		8 expert
			Written Exam
			Practical Exam (lab)

	before 2000 the frotiner checks our data till l4 (protocol and port) after 2000 grow up and checks hole the 7 layers

	palo alto is same in most fields

	fortinet define first  worlds utm (unified thread management) in 2004 
	
	next generation means our device can analyze and recommond some rules by behavior measurement also use central unit to detect unomallies and zero day attacks

	ngfw modes in flow base mde of inspection:
		profile mode > works on profiles
		policy base > works on policy	

		if use ips (Intrusion Prevention System) + ids (Intrusion detection system) > ngips
		ngips is a package of all features we need for detection and automation tasks to safe network

	fortinet has some product series :
		high end (Enterprise) : 1000 , 2000 , 3000 , 5000 , 7000
		midrange (Medium-Level) : 100 , 200 , 300 , 400 , 500 , 600 , 700 , 800 , 900
		Soho (Entry-Level) : 30 , 50 , 60 , 80 (d , f ,g ,.... these series are soho use kit (rakmountable))(power, g is most powerfull)

				FG60f
				FG61f > use ssd  for logs + wan optimization (like cache for web proxy)(2x price than 60f)

				soho use port 1 to manage device and dhcp (default allow access protocols are enable)
				mid and high end use management (MGMT) port for device also use port1

				management port (by default http, https, ping, ssh is enable) :
					192.168.1.99/24

				port number 1 (on this port we have dhcp server)soho level :
					192.168.2.99/24

					user : admin
					password :    (blank)
						(case sensetive)

					each device has management , if we want access them must set range 192.168.1.0 /24

				can't add mgmt port in to policy

				if need access device must use :
					show system interface

					config  system  interface 
					edit port1
					set ip address 192.168.1.1 255.255.255.0
					set allowaccess https ping (set allowed protocols on ip range)
					end	

				if need change url and show different value in header :
					config firewall policy
					edit 1
					show
					set authentication-redirect-address tet.com (use dns value)
					set redirect-url
					end

				we can use certificate to connection web gui and use ca server and active directory to use better it
				if use firewall behinde router and nat we can get hide our firewall vendor in this mechanism

			avfirewalls.com
			gartner
			netscreen.com
			getwanip
			eicar.org

			bchoosing fortinets serries are depends on forti matrix
			inbox must be rj45 + quick guide + console cable + power cale+ stack power cable
			
		network processor (firewall + vpn) + cpu + content processor (antivirus + ips + ssl)
		npu > interfaces + vdom + routing
		soc > optimization (use for accelerator)
		spu > av + ips + webfilter + app controll + ssl + inspection + antispam + waf

		if soc can't manage spu and npu the device ges to conserene and block some packets on auto mode

		cpu in fortinet devices has v9 and v10 serries
		
		system > frimware 
	
		fortigaurd use sandbox 

		fortidb (database protection)
		forti mail (like esa cisco)
		forti adc (like f5 loadbalancing)
		fortiweb (waf)
		forti ap (Access Point)
		forti switch (Switch Device)
		forti authenticator (Authentication)
		forti analyzer (Logging + Reporting)
		forti manager (central management)
		forti token (2factor authentication)
		forti hard token
		forti client (Endpoint vpn)
		forti client (Endpoint Management System (antivirus and posture)) > licnese
		fmg access (connectivity between forti manager and other devices)
		fct access (accessablity of forti clinet)
		forti Telemetry (forti client controller)(port 8013)
		ftm (forti token mobile path)
		
			capwap (centralize authentication and policy enforcement functions in wireless networks)port 5246(manage wireless devices and fortinet)
	
		the term csca means first step sof security that must be performd in organizations

	one of the most obvious features of firewalls is all to all connectivities are block and deny so static default route just will be accessible for router and each interface can use default route
		in firewall use any to any deny
		must set policy to wan reachability

	if need management on lan must use snmp feature on administrativly access in interface definition and must add features on vdoms carefully

	system > setting 
		allow concurrent session (how many users can login with on username)
		adminstratin setting (change ports and idle time outs)

		allow concurrent session : 
			it's better disable
			allow to users if want have one more session by one id in many devices or location

				config system setting
				show
				config system global
				show
				set admin-concorent enable

	 system  > admins 
		shows the users
		admin profile : 
			super admin : root + global + vdoms
			prof admin : other users - vdom can be reachable

		each profile contain some accessablities and set them on a user admin

	system > feature visibility > 
		allow unnamed policy (allow use or set a policy without name)

		interface policy > policy lookup

		also here we have implicit firewall policies  > means by default in last line of firewalling and policy rols must use deny  all to all 
			if turn it off our firewall works permit any to any (100% not recommonded)

	system > advance
		check compliance
		config script
		debug logs
		use custom email server
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
OS :
	threat intelligent			security update for fortinet 		centralized manager
	----------------------------------------------------------------------------------
							forti and subscribtion services
	----------------------------------------------------------------------------------
	ngfw(AntiVirus, Mobile Malware, Internet Service Database, Botnet IPs)
	ips (Intrusion Prevention System)
	Application Control
	web filtering
	AntiSpam
	Industrial Security
	----------------------------------------------------------------------------------
							fortios
	----------------------------------------------------------------------------------
							forti asic optimized hardware\ hypervisior

	fortios version 5.4.0 (5 is generations,4 is utilities,0 is bug fixing)

	auto install os :
		system > advance > usb auto install
		make bootable image from usb (not recommonded)

	disable backup and upgarde
			config system auto-install
			set auto-install-config disable
			set auto-install-image disable
			end

	diagnose sys top (live cpu usage and process)
			d term at end means deamon

	if need proxy setting we should use forti v 5.4

	for backupp must select admin icon on up right screen :
		config > backup 
			by default is plane text
				in this mode our password is hash 
			we can use hash method
				in this mode our configs are hash (all configs)

				name > devicename\hostname-date-time.conf

		config > restore
			just upload backups in same version and same frimmware
			after do this we have restart device

		in vdom mode can seperated backups for itself

		clie :
			execute backup config flash
			
			get system status

			execute factoryreset

			execute reboot
			execute shutdown

	ios upgarde 
			then restart the fortigate device by the execute reboot command, and when the console displays the message press any key to display configuration menu, click one of the keyboard buttons to enter the configuration menu
		then import term f in cli
		then accept whit term y to mak format device
		after the device is completely formatted, you must select the g key to get the firmware version. but before this step, first run the tftp server software and enter the path to save the firmware version that you downloaded on your computer in the current directory field. in the server interface section, the ip address of your computer is usually displayed
		then select the g key. as you can see in the figure below, you are asked to connect the network cable to the port of the forigate device, which in this example is the wan1 port. it also asks you to enter the ip address of the tftp server, which is usually the ip address of your computer.
		then you must enter the local address. you must enter an ip address with the same subnet as the tftp server ip address. then you must enter the name of the downloaded file of the firmware version.
		at the end, you will see the message save as default firmware/ backup firmware/ run image without saving: [d/b/r]. select the d key
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
License :
	in fortinet products just for signatures update need license by default device works in noral mode can use all features License has no effect on device performance

	online :
		fortiguard (all features)
	offline :
		manual
		no antispam and use 50% of webfilter

			antivirus + ips + application controll > use files for update (update.fortiguard.net)
			webfilter + antispam > use queries for update (service.fortiguard.net)

			also have bdl term in devices information means bundle > device + licenses

	must use release notes before updgrade firmware like upgrade path tools + known issues (knwon bugs) + resolved issues (bug fixes)

	
		update.fortiguard.net tcp 443these links and ports must be visible from web (online) :
		services.fortiguard.net port 8888 +53 udp (webfilter + antispam)

	if send traffic from iran our device will be block by fortiguard
	must use proxy (ccproxy) to use license
		config system autoupdate tunneling
		set address 9.9.9.9 (vps ip)
		set port 64000
		set username ali
		set password ali
		set status enable
		end
		abort (cancel)

		show/get system auto-update tunneling 

	in offline mode we use :
		execute restore ips ftp\tftp ips.pkg 192.168.200.1 username password
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Password Recovery , Login Option :
	admin: maintainer
	connect with console to device then resset system by hard way :)) 10-60 econds
	just use console cable
	must do it with maintainer user
		password is : bcpbFGT100E654321 (bcpb+serialnumber) (Serial Number should be upper-case)

		config system admin
		edit admin
		set password ....
			
	password recovery mode disable :
		config system global
		set admin-mainter disable
		end
			
			after this for reset password must reset factory device because you dont have maintainer :)

			with the lockout threshold feature, you can set a limit for the number of times the login operation to the device encounters an error. if the specified number of login operations to the device encounter an error, you will not be able to login to the device for a certain period of time. lockout threshold can be set from 1 to 10. the default value for the above option is 3.

				config system global
				set admin-lockout-threshold 80

			if a person enters the username and password incorrectly for the specified number of times, the lockout duration feature specifies the time period that the above user account is locked and cannot login to the device. you can set lockout duration from 1 to 4294967295 seconds. the default value for the above option is 60 seconds.

				config system global
				set admin-lockout-duration integer

	system > administrator > security
		restrict login to host (just the specific user system can connect)

	system > setting > password policy
		allow pasword reuse
		password expiration
		Number of Characters
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Device Operation Mode :
			transparent (layer 2 like switch)
				have problem in arp

				in transparent mode, you can use forward-domain command. using this command, a tag is added to vlan traffic, as a result, this traffic belongs to a specific collision group. and only other vlans that have this tag receive this traffic. by default, all interfaces and vlans are in collision group number 0. the advantages of this solution include ease of management, the need for fewer physical ports, and higher accessibility and flexibility. in the following, we will examine the above command with an example.			
					config system interface
					edit port1
					next
					edit port2
					set forward_domain 340
					next
					edit port3
					set forward_domain 341
					next
					edit port1-340
					set forward_domain 340
					set interface port1
					set vlanid 340
					next
					edit port1-341
					set forward_domain 341
					set interface port1
					set vlanid 341
					end
		
			if use spanning tree must use these do forward traffics
				config system interface
				edit name of interface
				set l2forward enable
				set stpforward enable
				end

			each fortigate device can have up to 255 ports in transparent mode without vdom activation. it should be noted that in case of vdom activation, this value also applies to each vdom. in nat mode, the number of ports can be from 0 to 8192 per vdom. of course, the above number depends on the model of the fortigate device. this number of ports includes vlans, other logical ports and physical ports. in order to have more than 255 ports in transparent mode, several vdoms must be set on the device.

			nat (default) (layer 3 like router) (Mostly use this mode)
				in nat mode, you can use vlanforward command. by default, vlanforward is enabled and the traffic of a vlan is sent to all other vlans located in this interface. by disabling vlanforward of any vlan on the desired interface, traffic will be sent only to the same vlan. and there will be no cross-talk between vlans. as a result, arp packets are sent from only one path in the network.
					config system interface
					edit port1
					set vlanforward disable
					end

				netbios name resolving with host nd wins server
					config system interface
					edit internal
					set netbios_forward enable
					set wins-ip 192.168.111.222
					end

			config system settings
			set opmode transparent
			set manageip 192.168.20.250 255.255.255.0
			set gateway 192.168.20.254
			end
			
			config system settings
			set opmode nat
			set ip 192.168.20.252 255.255.255.0
			set device port 1
			set gateway 192.168.20.254
			end

	by default, the fortigate device does not pass layer 2 traffic. if layer 2 protocols such as ipx, pptp or l2tp are used in your network, you need to configure the fortigate device to allow these protocols to pass through. be done.

		config system interface
		edit port1
		set l2forward enable

	inspection mode :
		proxy-based (in policy must use this mode if have webfilter,dlp,waf)(store and forward)
		flow-based
			antivirus works in this mode also works in proxy mode
	
			system > config >  features
				explicit proxy 
				wan optimization and cache
				dlp

	snmp
		system > config > snmp
			trap : port 162 (trigger update)
			querry : port 161 (from snmp application)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Befor Device Implement :
	sfp > 1g
	sfp + > 10g
	qsfp > 40g

	must change hostname and set timezone :
		system > setting
			hotname + central management (fortimanager)
			system time > ntp server (fortiguard)
			
				config system global
				set timezon 41
				end
				config system ntp
				get
				set type custom
				config ntpserver
				edit 1
				set server 219.93.105.83
				next
				set source-ip 219.93.105.83
				end

	each section we have ref part
	refrence means where did you use this parameters and  if make some cahnges in your topologies must reset all setting in fortinet

	if need make changes must see number 0 on ref aprt

	1 : interface setting
			we have redundant interface type in fortinet that use for aggregation or portchannel concept (802.3ad)
			network > interface 
				name (mac addres)
				alias (easy define)
				roles :
					wan (enable ping on this interface and botnets block, also can set many ip)
						we have retrive default gateway from server
						this option provide internet access without set default static route in routing table

						if turn off or use 2 isp for wan access must turn it off
						and set default static route for wan access also must set policy to access internet 

					lan
					dmz (dhcp get disable)
					undefined
			type :
				802.3ad link aggregate: 
					this option provides you with the ability to bind two or more interfaces together and thus create an aggregated Link or combined Link the bandwidth of the new link is equal to the total bandwidth of all the combined interfaces if one of the interfaces fails for any reason the traffic will automatically continue on its way through the rest of the interfaces forming the above link and only the bandwidth will decrease this action is similar to redundant interface with the difference that redundant interface uses only one Interface at a time while aggregated link uses the entire Bandwidth of combined Interfaces
					interface can be aggregated or redundant with other interfaces under certain circumstances

					*it should be a physical interface and not a vlan interface or sub-interface
					*it is not currently part of aggregate interface or redundant interface
					*interfaces must all be from the same vdom


					interface should not have ip address dhcp or pppoe settings should not be done on it
					interface should not be used in any security policy, vip, ip pool or multicast policy
					interface must not be an ha heartbeat interface
					reach out from policy or vip and routing
					*after these process our interfaces get hide and 

				wirtual wire pair :
					connect 2 physical port to each other with logical wire
					the port pairing feature introduced in fortios 5.2 
					is related to transparent mode. 
					but in fortios 5.4 version, the virtual wire pair feature is provided, which can be supported in both transparent and nat modes.

					we can make virtual wire pair to forward traffic into each pair
					here we want forward traffics between webserver and lan

					make a role with httpp service access between lan too wan and we apply the opposite

					a virtual wire pair consists of two interfaces that do not have ip addressing and are treated like a transparent mode vdom. 
					all traffic received by one interface in the virtual wire pair can only be forwarded to the other interface, provided a virtual wire pair firewall policy allows this traffic. 
					traffic from other interfaces cannot be routed to the interfaces in a virtual wire pair. 

					redundant and 802.3ad aggregate (lacp) interfaces can be included in a virtual wire pair.

					virtual wire pairs are useful for a typical topology where mac addresses do not behave normally. 
					for example, port pairing can be used in a direct server return (dsr) topology where the response mac address pair may not match the request’s mac address pair.

						config system virtual-wire-pair
						edit "VWP-name"
						set member "port3" "port4"
						set wildcard-vlan disable
						next
	
						onfig firewall policy
	    				edit 1
	       				set name "VWP-Policy"
	       				set srcintf "port3" "port4"
	       				set dstintf "port3" "port4"
	       				set srcaddr "all"
	       				set dstaddr "all"
	       				set action accept
	       				set schedule "always"
	       				set service "ALL"
	       				set utm-status enable
	       				set fsso disable
    					next

    			forti telementry :
					register clients with forti client
					we can set block or warning
					make integrity
					like network discovery in firepower
	
					it's better enable scan mac and ip address o network devices section
						device detection (some nics use auto send in newest version get omit) + active scanning

					User & Device > Device> Device Definitions

				software switch :
					hardware switch and software switch use same concepts and asiics but hardware switch has more power
	
					if need special interface type we can use software switch in fortinet interface types
					2 interface consider 1 and we can set ip and vlan on newest interface 
					we set vlan at first step then we can set ip address on parent interface with selecting interface type vlan
					software switch is like briidge in mikrotik

						switch mode: in this mode, all the interfaces are in one subnet and are considered as one interface, and based on the model of the fortigate device, they are known by one of the names lan or internal. this mode is used when the network topology is simple and users are in a subnet.

						interface mode: in this mode, each interface has its own ip address. in this case, the interfaces can be configured as a combination of hardware switches or software switches. in this way, several interfaces can be considered as one interface. this mode is suitable for networks with complex topology and several subnets are used to divide network traffic.

					to identify the mode, go to system > network > interfaces from the left menu, then check the type field in the interface specifications:
					
					if the interface was a physical interface, the fortigate device is in interface mode.
						
					if the interface was a hardware switch, the fortigate device is in switch mode. in fact, the interfaces of the fortigate device act like switches in this mode.

					config systel global
					sh

						we have a section in set internal-switch-mode define mode
						
					config system global
					set internal-switch-mode switch
					exit
						
					config system global
					set internal-switch-mode interface
					exit

				zone and vlan :
					for each reachability of vlans we must set rule but easier way = definitiation a zone
					zones are a group of one or more physical or virtual FortiGate interfaces that you can apply security policies to control inbound and outbound traffic.
	
					if enable block intera zone traffic we can enable private vlan in fortinet
					if need some traffics comes to each zone must define rules
	
					for one vlan we can set ip on interface

					creating vlan subinterfaces with the same vlan id does not create an internal connection between them. for example, it is allowed to create vlan id number 300 on port 1 and vlan id number 300 on port 2, but they are not related internally. rather, the connection between these two is the same as the connection of two physical ports in fortigate.
					
					usually, in vlan settings, the internal port of the fortigate device is connected to a vlan trunk, and the external port is connected to an internet router on which vlan settings are not done. in these settings, you can apply different policies on each vlan interface that is connected to the internal port.

						config system interface
						edit VLAN_100
						set interface port4
						set type vlan
						set vlanid 100
						set ip 172.100.1.1 255.255.255.0
						set allowaccess https ping telnet

					for intervlan routing must make policy access also use nat option in policies to make log reading easier than ago
					in fortigate we have trunk mode in default 
	
					in transparent mode, the fortigate device works like a layer-2 bridge, but it can still apply services such as antivirus, web filtering, spam filtering and intrusion protection to the traffic.
					of course, there are limitations in transparent mode, for example, you cannot use ssl vpn, pptp/l2tp vpn, dhcp server or nat.

			you can enable overlapping option :
				config system settings
				set allow-subnet-overlap enable

			dhcp :
				for lan interface and dhcp we can bind mac on ip address with set reservation on :
					monitor > dhcp monitor
					select item then click create/edit ip reservation

				time for lease time on dhcp is 7 day but can change in cli
					config system dhcp server
					edit server_entry_number
					set lease-time seconds
					end
	
				you can also use the following command to retrieve an assigned address
					execute dhcp lease-clear ip_address


				also in dhcp we should use ntp server in local mode on interface

			explicit web proxy
				is exchanged on the desired interface according to the web proxy settings. as a result all users must first activate the proxy with the specified port in the web proxy settings in their system in order to be able to open their desired web pages. by activating the above option if you go to 
					system > network > explicit proxy
				you will see the desired interface in the listen on interfaces section

				system > network > interfaces
					enable proxy on interface

				system > network > explicit proxy
					listen interface > means which vlan or lan can use this feature
					set http port
					set proxy fqdn
					proxy auto config (pac file) (with this our systems use automatic proxy no need set manualy or inject in ad and group policy)
					ftp over htp
					unknown http version
						reject
						best effort
					web caching with max http request and message legth
					default firewall policy action
						accept or deny
					web proxy forwardiing (back to back proxies)

			security mode 
				captive portal
					must set passive mode 
					then set sso (single sign on)
						users > authentication > single sign on > forti single sign on (fsso)
						must set ip address (dc server ip address) 

					must install agent in dc server 
						must disable firewall of windows dc server 
						must set role for make relation between agent and server in firewall

							execute fsso refresh

						it's better assign sso option to a group :
							user > groups > fsso

					works on port 8002

					in policy must set source users on fsso and also set set dns and interface (dc)
					must access the windows server to internet on policy
					better set user access to resdicted goup because allow all just make a alert and notification
					
					System >Config > Replacement Messages

					incoming interface > guest interface

					user and devices > guest management
					network > interfaces > guest interface
					
					policy
						outgoing interface > internet / lan / dmz
						source > all + guest users
						destination > lan \ dmz \ all
						service > all
						nat enable
						inspection mode > flow base
						must enable many security profile

					firewall user monitor is a option for captive portal monitor part

			endpoint control : 
				in setting of each interface we have access modes like ping .... also have fgt-access
				fgt access can use client vpn like posture and ...
				devices management 
					detect and identity devices (enable)
					broadcast discovery message (enable)

			we have blocking mode in botnet negotiate with our clients
			we can add secondar ip on interfaces

	2 : dns
		network > dns
			must disablle fortigaurd dns ip address and use specific internal ip for dns server
			we have same concept like retrive default gateway from dhcp server in dns
			in iterface field must set or use override internal dns if use internal dns server if not use specific raneg

			if set our dns server in dhcp server interface config on same as interface ip
			must do this to make forwarding
				config system dns-server
				edit port3
				set mode forward-only
				next
				
	3 : sdwan (internet links) & routing
		loadbalancing of Internet Links
		network > sdwan 
			in version 5.2 of the fortigate operating system and higher versions, a new type of interface named virtual wan link has been added. this link includes two or more physical interfaces that connect to two or more isps and require load balancing and routing settings to support redundant internet connections. in fact, internet links are combined and considered as an interface.
			
			enable the sdwan and set ip address on interfaces with aliases
			spill over is smart mechanism for changing the default route
			it's beeter set source destination ip 
	
			sdwan status check :
				is like sla and must enable ping option
	
			sdwan rule :
				source ip base : each system use one link
				
				weighted round robin : based on links weight

				spill over : threshold (if use 80 percentage of link transfer clients to another link)
					spill-over or usage-based actually determines a path for new sessions to ports that have not exceeded the applied bandwidth limit
					you can consider the allowed value for bandwidth from 0 to 2097000 It should be noted that the bandwidth limit is considered only for outgoing traffic
					when the bandwidth used by this port is equal to the spillover threshold, the rest of the sessions are sent to other ports on which ecmp route settings have been made

				src and dest 

				measured ip base : measured links process then assign way

					config system settings
					set v4-ecmp-mode usage-based
					end

			load balancing :
				system > network > wan link load balancing
					 set members on wan interfaces
					 name a group for reachablity
					 	interface > wan 1
					 	weight > 3
					 	gateway 172.16.16.1
					 	health check > enable
					 	probe type > ping
					 	probe server > 172.16.16.1
					 	probe interval > 5
					 	failure threshold > 5
					 	recover threshold > 5
	
				set static route t reach isp edge
					then
						router > static > static route
							destination > 0.0.0.0/0
							devices > wan-lb-1
							distance > 10
							priority > 0
	
				access policy :
					policy & objects > policy > ipv4
						incoming interface > lan
						source address > all
						source users > all
						source device > all
						destination address > all
						outgoing interface > wan-lb-1
						schedule > always
						service > all
						action > accept
						nat > enable
		
				network > static route
					interface (egress interface)
					destination : range of destination network
					Gateway: Next-hop IP address
					default ad : 10
					priority : 0 (lower is better, use for fault tellorance)
					blackhole interface :if put addres on this field our traffic get block (can block fortiguard ip address)
					
				get router info routing-table all
	
				define access list
					config router access-list
					edit access_list_name
					set comments
					config rule
					edit access_list_id
					set action deny | permit
					set exact-match enable | disable
					set prefix IPv4 address and network mask | any
					set wildcard ipv4_address  wildcard_mask
	
			asymetric route
				at some point, you may find that hosts on one network may not be able to communicate with other networks. this problem occurs when requests and response packets go in different directions. in simpler terms, response packets do not return the same path as request packets. if fortigate detects response packets while it does not identify the corresponding request packets, then it blocks packets as invalid packets. fortigate also blocks a packet that has been repeatedly entered in multiple interfaces as a possible attack, known as asymmetric routing. by default, fortigate blocks packets or drops the session. you can enable and allow asymmetric routing by entering the following commands

				config system settings
				set asymroute enabe

				allowing asymmetric routing is not the best solution. because with this action, you will reduce the security of your network must use better routng mechanism

		we have BPR
			network > policy route
			we can define if source was 100.100.100.2 redirect it to 10.10.20.30
			incoming interface > port kerio
			protocol > any

			then
				action forward traffic
					out interface
					gateway 

		Dynamic Routing :
			BGP :
					network > dynamic route
						local as
						router id
						neighbor ip and as
						network ip and mask

						first set  interfaces ip address and set these on routing table 

							config system interface
							edit port1
							set alias internal
							set ip 10.11.101.110 255.255.255.0
							set allowaccess http https ssh ping
							set description Company internal network
							set status up
							next
							edit port2
							set alias external1
							set ip 172.21.111.5 255.255.255.0
							set allowaccess https ssh ping
							set description ISP1 External BGP network
							set status up
							next
							edit port3
							set alias external2
							set ip 172.22.222.5 255.255.255.0
							set allowaccess https ssh ping
							set description ISP2 External BGP network
							set status up
							next
							
							config router static
							edit 1
							set device port2
							set distance 10
							set gateway 172.21.111.5
							next
							edit 2
							set device port3
							set distance 15
							set gateway 172.22.222.5
							next
							en
				
				
							config firewall service group
							edit "Basic_Services
							set member BGP DNS FTP FTP_GET FTP_PUT HTTP HTTPS
							next
				
				
							config system zone
							edit ISPs
							set interface port2 port3
							set intrazone block
							next
				
							policy & object > objects > address
								make internal network object addresss > 10.11.101.0 /24
								on port
						
							config firewall policy
							edit 1
							set srcintf port1
							set srcaddr Internal_network
							set dstintf ISPs
							set dstaddr all
							set schedule always
							set service Basic_services
							set action accept
							set nat enable
							set profile-status enable
							set logtraffic enable
							set comments ISP1 basic services out policy
							next
							edit 2
							set srcintf ISPs
							set srcaddr all
							set dstintf port1
							set dstaddr Internal_network
							set schedule always
							set service Basic_services
							set action accept
							set nat enable
							set profile-status enable
							set logtraffic enable
							set comments ISP1 basic services in policy
							next
							end
							set comments ISP1 basic services in policy
							next
				
							config router BGP
							set as 1
							set router-id 10.11.101.110
							end
							
							config router bgp
							config network
							edit 1
							set prefix 10.11.101.0 255.255.255.0
							next
							end
							end
							
							config router BGP
							set as 1
							config neighbor
							edit 172.21.111.4
							set remote-as 650001
							next
							edit 172.22.222.4
							set remote-as 650002
							next
							end
				
							config router bgp
							set bestpath-med-missing-as-worst enable
							set fast-external-failover enable
							set graceful-restart enable
							set graceful-restart-time 120
							set graceful-stalepath-time 180
							set graceful-update-delay 180
							set holdtime-timer 120
							set keepalive-timer 45
							set log-neighbor-changes enable
							config neighbor
							edit 172.21.111.4
							set connect-timer 60
							set description ISP1
							set holdtime-timer 120
							set keepalive-timer 45
							set weight 250
							next
							edit 172.22.222.4
							set connect-timer 60set description ISP2
							set holdtime-timer 120
							set keepalive-timer 45
							set weight 100
							next
							end
							
							get router info routing-table details
							get router info routingtable bgp
							get router info routing-table databas

			OSPF :
					network > dynamic route
						ospf 
							router id
							area
								area
								type
								authentication
							networks
							interfaces
				
							config router ospf
							config ospf-interface
							edit Router1-Internal-DR
							set priority 255
						
							config router ospf
							config ospf-interface
							edit Router2-Internal
							set priority 250
							end
						
							execute router clear ospf
						
							get router info ospf neighbor all

		monitor > routing monitor (sho routing table)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Reditribution rip , ospf and bgp ,a isis :
	to rip :
		config router rip
    	config redistribute connected  
    	set status enable  
    	end  
    	config redistribute isis  
    	set status enable  
   		config redistribute ospf  
    	set status enable  
    	end  
    	config redistribute static  
    	set status enable  
    	end  
		config redistribute bgp  
    	set status enable  

    to bgp :
		config router bgp
		config redistribute connected  
		set status enable  
		end  
		config redistribute isis  
		set status enable  
		config redistribute rip  
		set status enable  
		end
		config redistribute static  
		set status enable  
		end  
		config redistribute ospf  
		set status enable  
		end

	to ospf :
		config router ospf
		config redistribute connected  
		set status enable  
		end  
		config redistribute isis  
		set status enable  
		config redistribute rip  
		set status enable  
		end  
		config redistribute static  
		set status enable  
		end  
		config redistribute bgp  
		set status enable  
		end  

	to isis :
		config router isis
		config redistribute connected  
		set status enable  
		end  
		config redistribute ospf  
		set status enable  
		config redistribute rip  
		set status enable  
		end  
		config redistribute static  
		set status enable  
		end  
		config redistribute bgp  
		set status enable  
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Policy & Objects :
	all protocols and ways are block in fortinet by default and must access them to allow
		1 : incoming interface 
		2 : outgoing interface
		3 : source address (which ip address from incoming interface)
		4 : destination address (which ip address from outgoing interface)
		5 : services (which protocols must be allowed)
		6 : schedule (define the time that policy should work)
		7 : nat (enable or disable) > if enabled, use outgoing interface ip or IP-Pool\ route
			we must set specific values for each section just for (2 + 4 + 6) can set all
			interface means policy
			object means items in service s ......


	object contains :
		address
			we have 31 object in address by default
			in address mode object accessablities must set url or ip or fqdn must set public dns servers in rols

			it's better enable checbox of :
				static route configuration 
				show address list

		schedule
			2 type:
				recurring (periodic)
				one-time (just one time)

		service
		    should specify the protocol or port number

		virtual ip (vip)
			destination nat
			external ip address > public ip address
			mapped ip address > convert to internal \ private ip address and open all ports must port forward

			for lab design we should 
			interface > any
			external ip > 172.20.20.50 (must use wan ip)(here can use third party ip in private mode)
			mapped ip > owaspbwa ip (internal webserver)
			portforward > 80 to 80

				the best practice is dnat or vip to redirect a traffic access from wan into lan
				in fortigate we must use this
				but in cisco asa if access a traffic to lan means have all permission

				*if add in destination nat our source nat no ip address could be reachable

			make loadbalance between some servers in dnat
				system > config > feature visibility
					loadbalancer
				Policy & Objects > Load Balance > Health Check
					type > http
					port > 80
					
					for the port field, if you select zero value, health check monitor will consider the port number defined in real server as the port number. its use is when you use a health check monitor for several real servers. in this example, the value of 80 was considered for the above field

					url > http://10.10.10.10
					interval > 10 seconds
					timeout > 2 seconds
					retry > 3
					
					the matched content field is used to test the correct operation of a web service. the way it works is that if the real server returns the value set in this field in response to the request, the fortigate device realizes that the web server is working correctly.

				Policy & Objects > Load Balance > Virtual server
					type > http
					viertual server ip > 192.168.10.1 (can set ip public)
					virtual server port > 80
						HTTP: Port 80
						HTTPS: Port 443
						IMAPS: Port 993
						POP3S: Port 995
						SMTPS: Port 465

				loadbalance method > least session is better than other
					source ip hash: in this method, traffic is distributed statically and equally between real servers. this method provides you with persistence because in this method the source addresses are always sent to the same real server that they went to in the previous times. this method is stateless, which means that if one of the real servers goes out of service for any reason, the distribution process changes and persistence is lost.

					round robin: in this method, the new request goes to the next real server in turn. in other words, real servers receive requests in turn. in fact, all real servers are treated the same, regardless of the response time or the number of connections related to each real server, and all are placed in the same priority to receive traffic. of course, traffic is not assigned to real servers that are off or unavailable.

					weighted: in this method, real servers with a higher weight receive a higher percentage of traffic. if you choose this option, you must assign a certain amount of weight to each real server.

					first alive: in this method, sessions are always sent to the first responsive real server. this method provides fail over protection. in this way, it always sends sessions to the first ready and responsive real server, and if the above real server fails, it sends it to the next responsive real server. checking ready and responsive real servers is done in the order of adding them during the initial configuration of the virtual server. that is, the sessions are sent to any server that was previously added in the virtual server settings, as long as the above server is ready and responsive.

					least rtt(round trip time): in this method, sessions are directed to the real server with the lowest round trip time. round trip time is determined by ping health check monitor and if ping health check monitor is not configured in virtual server, its value is considered equal to zero.

					least session: in this method, sessions are directed to the real server that has the lowest number of connections at the moment. this method is used in cases where real servers have similar and identical capabilities and has better performance.
						
					http host: in this method, the host field in the http header is used to direct the session to the desired real server.

				persistance > http cookie
					by using the persistence field, you ensure that a user connects to the same real server that was connected to in the previous request. available options for this field are none, http cookie, and ssl session id. if you use persistence in your settings, the fortigate device will direct a new session to an existing real server based on the settings you made in the load balance method field. if this session contains an http cookie or an ssl session id, the fortigate device sends all previous sessions that have the same http cookie or ssl session id to the real server.

				http multiplexing 
					preserver client ip
						preserve client ip: this option is used to preserve users' ip address in x-forwarded-for http header. this option is used when you want to send log messages to the user. if this option is not activated, the ip address of fortigate is included in the above http header.
					multiplexing http requests/response over single tcp connection
						multiplex http requests/responses over a single tcp connection: by selecting this option, you can assign multiple requests from users to an existing tcp connection between fortigate and real server.
				health check
				ssl and ceertificate must set on 1024 or 2048 (offloading)

				Policy & Objects > Load Balance > Real Servers
					virtual server
					ip address >10.10.10.10 (real server ip)
					port > 80
					max connection > blank
						means no limit
					weight > 1
						cause don't use weight mod in loadbalancing here see 1
					http host
					mode > active/standby (if another device is active goes to standby)/disable

				Policy & Objects > Policy > IPv4 
					incoming interface > wan 1
					source address > all
					source user > all
					source device > all
					destination address > loadbalance virtual serrrver 1
					outgoing interface> dmz
					service > http
					schedule > always
					action > accept
	
				*if write some roles for dnat must set like this
				
				policy & objects > ipv4 policy
					incoming interface > internet
					outgoing interface > dmz
					source > all
					destination > our-dmz-srv-address
					service > all \ http \ https
					security > av + ips + ids
					nat disable
		
		ip pool
			use arp reply option in this part
			overload
			one to one :
				first extenal ip address (edge) > internal range 1
				second extenal ip address (edge) > internal range 2

			fixed port range : 
				in last mode of this we didn't need define internal ranges

			port block all action :
				flexiable than others can add port size .....
	
		traffic shaper :
			shared shaped > inside to outside (upload)
			reverse shaped > outside to inside (download)
		
			if run idm our bw in isp get falt

			the working mechanism of traffic shaping does not appear to be a very complicated problem, this service first identifies the types of information packets that are important in your network and have a higher priority, and when the network traffic increases, it reduces the priority of other traffic and prioritizes it.
			it boosts the more important traffic and makes the service you want always have a more reliable quality degree, thus ensuring that the above network always provides a quality default for a series of transmitted data, for example priority traffic on your network is voip service. Traffic shaping serves this type of traffic for you with a higher degree of priority when there is a shortage of bandwidth.
		
			system > config > features
				viop
				traffic shaper
		
			policy & objects > traffic shaper
				type 
					shared
					per ip
				apply shaper
					per policy
						if use bw 1m nd use 4 policies 
						each policy use 1m
						policy 1 > 1m
						policy 2 > 1m
					all policies use this shaper
						if use bw 1m and use 4 policies
						policy 1 > 250k
						policy 2 > 250k
				priority
					high
				max bw
				guaranteed bw
				dscp (diffrentiated service point)
		
				the fortigate device performs processing on traffic when traffic enters an interface or when traffic leaves an interface. in the next phases of traffic processing, bandwidth checking operations are performed. if the traffic exceeds the bandwidth specified in the settings, the fortigate device will drop the extra packets. the time spent on the traffic before dropping the packet for previous processing such as web filtering, decryption or ips will actually be considered a waste of time. you can avoid such time wasting by making settings on the input port
	
				config system interface
				edit port1
				set inbandwidth 1500
				set outbandwidth 1500 (kb)(if set 0 means no limit)
				next

	*if need proxy setting we should use forti v 5.4
		proxy is layer 7
		vpn is layer 3/2

		proxy policy (use for explicit proxy)
			type
				explicit web
				transparent web
				ftp
			enbaled on > show interfaces
			outgoing interface > wan
			source > all + some users
			destination > all
			schedule > always
			action > accept

			display disclaimer (authentication with which type)
				disable
				by domain
				by policy
				by user
			customize message disclaimer
			security profile
			web proxy forwaring

			after this we need add policy in ipv4 policy for clients on firewall(norml policy access to internet)

		proxy option 
			works on device full proxy mode 
			regenerate each packet recieve or transfer
			block oversize  file or email

	in policy page we have interface pair view or sequence type for policy show
	we have policy lookup in routing and policy also its accessible from fortiveiw and monitor route monitor
	we can use inser above or bottom on policy and make a clone or copy from our source by default new insert is disable so make it enable
	in policy definition we have internet service database 
		this feature use a license not so applicate for us
		database for ip changed in each applications and communities

	in policy rule we have a type of action like learning 
		this feature help us to monitor port for 24 hours
		and recommond some things
		can't use security profiles
		report all activities

	dos policy make a little dos protection we can set threshold
		must set incoming interface and source , destination also use services

	internet access rule simple :
		incoming interface > zone lan
		outgoing interface > sdwan
		source > vlan 10 + vlan 20
		service > webaccess (dns + http + https) + ping
		schedule > always (important in policy)
		nat enable

	inter area :
		incoming interface > zone lan
		outgoing interface > dmz
		source > vlan 10 + vlan 20
		destination > srv-dmz
		service > http  + ping
		schedule > always (important in policy)
		route enable (must have route not nat)
		nat >disable

	dmz access from wan :
		incoming interface > sdwan
		outgoing interface > dmz/owaspbwa
		source > http + https
		destination > vip-site (created virtual ip last step and dnat)
		service > http
		nat disable (detect hackers)
		security profile > ips antivirus
		log all sessions

	restrict internet access :
		incoming interface > lan
		outgoing interface > internet
		source > all + some-trusted-users
		destination > site-x.com + google-dns
		service > all
		nat enable
		log all sessions
		inspection mode > flow base

	in some version we have web proy or explicit proy roles :
		explicit proxy type > web
		enbaled on interface ether x
		source address > all
		destination address > all
		outgoing interface > wan1
		schedule > always
		action > accept
		also must enable web cache

		incoming interface > ether x
		source address > all
		destination address > all
		outgoing interface > wan1
		service > https + http
		schedule > always
		action > deny

		monitor > wan opt. monitor

	sflow
		analyz traffic flow send to collector (solarwinds nta (network traffic analyzer)) like torch in mikrotik :

		config system sflow
		set collector-ip 192.168.1.1
		set collector-port 2055
		config system interface 
		edit port 1
		set sflow-sample enable
		set sflow-direction both
		set polling-interval 3000 (seconds)
		end

	session helper
		the fortigate device often analyzes traffic related to the tcp/ip protocol by comparing the information in the header of a packet with the policy. by this comparison, it will be determined whether the packet and session to which the above packet belongs should be accepted or denied.

		some protocols include information in the packet body that must be analyzed in order to process sessions. for example, the sip protocol, which is related to voip, uses tcp with a standard destination port number to control packets and establish a sip call. but the packets that communicate can use different types of udp protocol along with different port numbers.

		the fortigate device uses session helper to analyze data in the packet body and also allows protocols to send packets through fortigate.

		show system session-helper
		config system session-helper
		edit 1
		set name pptp
		set port 1723
		set protocol 6
		next
		set name h323
		set port 1720
		set protocol 6
		end
	
		show system session-helper 11
		config system session-helper
		edit 11
		set name pmap
		set port 111
		set protocol 6
		next
		end

		config system session-helper
		delete 19

	fortiview > show live traffics
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Security Profile :
	we can find these option and turn them to on mode in :
		system > config > features

	antivirus 
		based on fortigaurd 
		use network antivirus
		packet sniffer
		limit scan cause can't use ssl
		system > setting > system opertion setting 
			inspection mode :
				actions mode :
					quick scan
					full scan (deeper)
						mobile malware 
						email attackers
					block
					monitor (logs)

				flow base :
					faster than proxy mode and a sightly less security when our device cpu is 100 % better enable this
					can't use dlp , expilicite proxy , waf

					modes :
						profile base
						policy base (more accurate)

				proxy :
					more secure and works like proxy
					action mode :
						block
						monitor

			inspection mode: the fortigate device uses two methods to scan for malware:
				proxy: in this method, the antivirus buffers a file when it arrives, and when the file transfer is finished, the antivirus checks the file. if no virus is found, the desired file will be sent to the destination. if a virus is found, a message will be sent to the destination.
				
				flow-based: in this method, when the packets of a file enter the fortigate device, a copy of the packet is cached before delivering the packet to the receiver. when the last packet from the file enters the fortigate device, this packet is also cached but not sent to the receiver. at this stage, the entire file is delivered to the antivirus for complete scanning. if the file contains malware, the last packet is dropped and not sent to the receiver, so the session is reset.
				
				detect virus: through this field, you specify what operation should be applied if a virus is detected. this field has two options, block and monitor.
				
				botnet: a botnet is a collection of devices connected to the internet, which can include computers, servers, mobile phones, etc. these devices can be affected and controlled by some malware. usually, users are not aware of the above effects on their systems. botnets are often used to send email spam and generate malicious traffic.

				grayware is a type of program that is unintentionally installed on the user's computer. these types of programs are installed on the system without the consent and knowledge of the user and reduce the efficiency of the system. if the file passes the virus scanning stage, it will be checked and tested for grayware. at this point, you need to enable grayware scanning.

					config antivirus settings
					set grayware enable
			Policy & Objects > Policy > Proxy Options
				block oversize file/email > threshold 8
				then create a policy ipv4 in firewall and set proxy options on these profile

	ICAP
		is the acronym for internet content adaptation protocol the purpose of the feature is to off load work that would normally take place on the firewall to a separate server specifically set up for the specialized processing of the incoming traffic. this takes some of the resource strain off of the fortigate firewall leaving it to concentrate its resources on things that only it can do
		off-loading value-added services from web servers to icap servers allows those same web servers to be scaled according to raw http throughput versus having to handle these extra tasks
		icap servers are focused on a specific function, for example:
			ad insertion
			virus scanning
			content translation
			http header or url manipulation
			language translation
			the protocol
			offloading using icap
			configuration settings
			example icap sequence
			example scenerio

			the protocol is a lightweight member of the tcp/ip suite of protocols. it is an application layer protocol and its specifications are set out in rfc 3507. the default tcp that is assigned to it is 1344. its purpose is to support http content adaptation by providing simple object-based content vectoring for http services. icap is usually used to implement virus scanning and content filters in transparent http proxy caches. content adaptation refers to performing the particular value added service, or content manipulation, for an associated client request/response

	application control
		layer 7 control 
		system > application control
		must enable 3 last option in list

		for each policy must set special profile 
		allow access on browser and http services

		action mode :
			monitor (log + allow access) for iranian application must use this mode
			allow (just allow access) some needed applications take thi
			block all another applications
			qarantuntine (ban if some body use it)

		it's better use unknown app on enable mode

		application override (if all applications was block we can set specification access)
		filter override (more priority) (set blocking filter depends on critial risks level)(use when all applicationss are open)

		activate the deep inspection of cloud applications option. this option allows you to monitor web-based applications such as video streaming. in the security profiles section, you must activate application control and select default from the available options. by activating the above option, ssl inspection is also activated, and you must choose deep-inspection from the available options.

	dlp
		system > config > features
			dlp
			multiple security profile

		you must apply a dlp watermark to a file. it should be noted that dlp watermarking client is available in fortiexplorer. therefore, you need to download the version of fortiexplorer for microsoft windows.	

		after downloading and installing fortiexplorer, open the program and go to the tools section and select the dlp watermark option. 
		then select the select file option for the apply watermark to section and then select the desired file. also, for the sensitivity level section, do not select the critical option from among the available options. 
		in the same way and according to the figure, set the identifier and output directory fields and finally click on apply watermark. as shown in the figure, you will see that the desired file has been successfully processed

		message (every thing was equall to this parameter be ommit)
		files (data type)

		use filter mode 
		symantec is the best product

		after choosing a name for the profile, select the create new option and create a filter.
		for the filter section, select files and for the watermark sensitivity and corporate identifier sections, enter the values you specified for fortiexplorer in the previous step.
		next, in the examine the following services section, activate the required services of your network and select the block option for the action section.

		for the filter section, select the files option, then activate the specify file types option, and select the executable - exe option for the file types section. next, in the examine the following services section, activate the required services of your network and select the block option for the action section.

		must use ssl inspection in deep inspection mode

	ips
		ips signature (block some id or cve)
		ips filter
		severity (sensetive level)
		track by (better set on source \ destination must set connection limit)
		rate base signatuure (dos + bruteforce)

		System > Config > Features
			dlp
			multiple security profile (can add more than one or default profiles)
			Intrusion protection

		 policy & objects > policy > dos 
		 	destination address > all
		 	source address > all
		 	incoming interface > wan 1
		 	service > all

		 	also enable logging and status checked
		 	action block

		 	and set this policy enable

		 system > fortiview > threats 

		 set action in ips definition on monitor or defualt to use fortigaurd actions
		 use severity 4/5 -5/5

		 log and report > ips

		 in not licensed signature count are 5k
		 used for many utems in firewall

	dns 
		works in web filter and anti spam also antivirus

	antispam
		fortiguard antispam service uses sender ip reputation and spam signature databases along with other sophisticated spam filtering tools to detect and block spam messages. it should be noted that you need to purchase a license for the fortiguard antispam service. otherwise, you can use the local spam filtering option. this option works based on your internal dns server or the lists you create.
		ssl inspection get automaticaly up
		must use ssl deep inspectio

		we can set some tags in subject or mime (cloud based email filtering and archiving service)

		dns filter (like webfilter)

		local spam filter (inter lan pollution)
	
		hello dns lookup (to send email use this option in spam mode by pass this)
			this option is related to smtp traffic. every time a user opens an smtp session through a server, the user sends a hello command along with his domain name to the fortigate device. the fortigate device takes this domain name and performs a dns lookup to determine whether this domain name exists externally. if the above lookup encounters an error, all messages received in this smtp session will be considered as spam.
	
		return eamil dns check
			fortigate takes the domain name by replying to the desired email and checks the presence or absence of a or mx records through dns server. in the absence of the above records, the email is considered as spam
	
		black and white list
			if the value set for the type field is the ip/netmask option, you must use an ip address along with a subnet mask to determine the pattern.

			if the value set for the type field is the email wildcard option, you must use an email address along with a wildcard symbol to specify the template. for example example.com.* or fred@*.com

			if the value set for the type field is the email regular expression option, you must use an email address along with a wildcard symbol to specify the pattern. for example, ^[_a-z0-9-]+(\.[_a -z0-9-]+)*@(example|xmple|examp).(com|org|net)

	web filtering
		if device has license we can use all features if don't just can use
			search engine
			static url filter
			rating option
			prox option 
		fortigaaurd category base filter (risk control (malware , phishing))
		fortigaurd.com > thread lookup
		we can make category by qoata and time
		actions :
			block
			accept
			monitor
			customize
			warning
			authentication (can set time + login)

		parental controll can be set with some levels g is highest
		allow users to override blocked categories (all sites are block we can make exception some users) 

		search engine 
			force safe search
			restrict youtube users
			key  word loging (log search word)

		modes on another versions :
			proxy : like proxy in antivirus
			dns : inspect dns names
			flow base : session base and faster than proxy has no warning page

			system > config > fortigaurd
				has link in below of page
				check type or category of site

		static url filtering :
			block invalid url
			url filter
			block malicious (sand box)
			web content filter (words in page)
			action mode: 
				exempt : first scanned by antivirus then if our connections were valid all subdomain will be reachable	
				monitor : allowed and log them
	
			rating option :
				allow website when a rating error occurs 
					if fortigaurd was unreachable site cant be open
				
				block redirection of https 
					block redirects
	
				rate image by url 
					check images inside of site
	
			restrict google account usage to specific domain :
				if some body login with his/her self to chrome some features inside chrome get disable

		*if write a policy ipv4 roll in firewall must use ssl certificate inspection with web and static url filters
		
	content filtering : aggregation of dlp and web filtering
		must use ssl deep inspection fro this

	forticlient compliance profile :
		posture & end point security
		network > interface > administration access
			forti telementry
				register clients with forti client
				we can set block or warning
				make integrity
		endpoint vulnaurability scan on client
		security posture check
			realtime protecct( forti is av)
			webfilter (on client side filter some things)
			.....
	
			user and devices > authentication > forti client profile > vpn > client provisioning

			also in ipv4 policies must define compliant with forti client profile on nat section 

	ssl /ssh inspection
		enable ssl inspection
			modes :
				multiple client connection to multiple server (lan to wan)
					inspection mothod
						ssl certificate inspection
							if website were valid in certs is ok to incoming
						
						full ssl inspection
							i  will check dta like ssl offloading 
							device use fortigate slf certificate and negotiate with other websites 

				*if don't have license can't use inspection or web filter
					https must use inspection and use content filtering
				
				protecting ssl server (wan to lan or dmz servers)
					our publishment are use our certificates to negotiate with public
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
SSL & Certification :
	system > certification
		ssl inspection :
			1 : multiple clients :
				multiple servers (lan > wan)
				full ssl (man in the middle)(deep inspection)
				valid certs (ssl cert inspection)

			2 : protecting ssl server :
				we have servers in dmz must define cas in servers to use it normally


		ssl inspection model :
			wan -------------------(cert wan)--------------------edge device ---------------(cert dmz)---------------- dmz
									encryption												encryption

		ssl uploading model :
			wan --------------(cert wan + port 443)--------------edge device ---------------(port 80)------------------ dmz
									encryption												plane text

			web rating override : close all sites except here
			web profile override : users changing
			custom signature

	ssl decryption :
		certificate > local certificate
		policy and objects >  policy > ssl inspection
			add
				ca certificate
				full ssl inspection
				allow invalid ssl certificate (must enable)

				in ipv4 policies must set security profile on ssl inspection and web filter

		install certificate on clients
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Security Fabric :
	after 5.6 get features
	in v6 use automation for restore pkg signature files for license offline
	we can see all events and devices of fortinet company on all devices

	audting > automation in policies
		offer some roles and some cve ...

	setting (this section help us to add forti products)
		secure conectivity beetwen products

	if enable security fabric > setting then goes to telemetry and analyzer activation our topology get run
		group name
		group pasword
		connect to upstream fortigate (if be master don't enable if be slve must enabe)
		forti telementry interfaces (the protocol used in security fabric connectivity)
		management interface
		analayzer part contain ip of analyzer and uploading log option like
			real time 
			every minut
			every 5 minut
		also can encrypt log transmission

	must define rol in ipv4 policy to reachable devices with specific ip range
	on fortigate
		user& device > inventory devices

	on faz 
		log and report > security audit events
		user& device > inventory devices
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
BYOD :
	we can use vlan or zone to use byod feature
	interfaces > devices management > detect and identity define
	user and devices > devices > device define

	in ipv4 policies we can set incoming inteface on byod

	policies and objects > ssl inspection
		ssl inspection option
			enable ssl inspection 
				multiple clients connection
			ca cert
			inspection mode
				ssl cert
				full 
			inspect all ports
		common
			allow invalid ssl cert
			log invalid cert
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
HA (high availability)
	has 2 mode :
		active-passive : one active one deactive and on circute
		active-active : both of them are in circute and active(more throughput)(loadbalance)

	first, it must be ensured that both fortigate devices are of the same model and use the same firmware version. it should also be noted that register and license operations must be performed on the new fortigate device before adding it to the circuit.

	system > dashboard > status > system information
		change host name

	system > ha
		device priority : 0 - 255
		default is 128
		higher is boss
		
	how detect master :
		priority number
		serial number

	session pickup : while users are sending data and requests to forti maybe our devices crashed so this option make live replication
	monitor interface : important port in our networks must define here to be visible   like wan dmz ... if didn't reachable must change boss
		no lan no zone and make it redundant also can use redundant ports
	heart beat interface : priority + config + synchronise
	management interface reservation 
		can set ip and gateway on interfaces

		get system ha
		show system ha
		execute ha sync start
		diagnose sys ha check recalc

	what is the override in fortinet ha?
	like cisco preemt
	by default is disable
	in disable mode steps of monitoring is :
		monitor port
		uptime (ha, higher is primary)
		priority
		serial number

		to change primary and secondary must change uptime:
				diagnose system ha restet-uptime

			config system ha
			set override enable
			end

	if be enable steps of monitoring will be :
					1. The most available ports
					2. The smallest Device Priority number (0 has the lowest priority)
					3. The highest uptime value
					4. The highest-sorting serial number

		if make it enable we don't need do it manually to change primary and secondary

	in real device we have group id must change from default value > 1 to another number

	if need disconnect the device must use slave if master get disconnect all configs get purge
	in vm we have same serial number and could not make ha 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Logs :
	in fortigate os we have trouble must enable log
	log&report > log setting 
		must enable
	for logs must format the disk 
		get system status
		execute formatloglist (if wre not available must use this command)
		
	show matching logs > policy logs
	log & report > log setting > log on forti analyzer \ fortimanager (if don't have hdd or ssd in device)
		real time
		every minut
		each 5 mmin

	also enable local log

	we can customize logs in this section

		our logs saved a week after that get clean altho be on hdd

	we can use send email in hih severity logs by log and report on fortigate
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Virtual Domain :
	we can have vrf or evn on fortinet like cisco cef and context
	virtualy have many device on it

		cli :
			config system global (global user has full and root access)
			set vdom-admin enable (don't use auto complete)
			system vdom x (define vdom x)
			set vdom-mod split-vdom (means 1) \ multi-vdom (many) \ nvdom (0)
			end

		vdom link is the connectivity beetwen vdoms(in global shown)
		network > interface
			vdom link

	by default we haven't ths option with commands at above we can active it and use it on 10 virtual customers
	if need more add more :)

	virtual domain > root (see total setting and config (like last view))

	global > vdom > vdom > new domain

	config global
	execute ping 8.8.8.8 (get error)
	config vdom
	edit root
	execute ping 8.8.8.8
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
2 Factor Authentication :
	user and devices > forti token
		create new

	two-factor authentication fortios 5.0 and above supports fortitoken and fortitoken mobile. fortitoken mobile is one of fortinet's applications that allows you to generate one-time otps on a mobile phone for two-factor authentication to assign a token to an administrative account, go to 
		
		system > admin > administrators 
		
		from the left menu and after selecting the desired administrative account, enable the enable two-factor authentication option and select the desired token. it should be noted that the email address and mobile number of the relevant manager must be entered in the management account settings to receive the fortitoken mobile activation code. to set up the fortigate device to send email or sms, go to 

			system > config 

			 from the left menu. go to advanced and make settings in email service and sms service.

	user and deevices > user > user define
		remote ldap user
		select users and groups	
			enable 2 factor authentication 
			add this user to group
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Users Authentication :
	in forti we have just authentication no authorization or accounting
	users can be :
		local
		remote : ldap directory(port 389)

	user modes : 
		firewall (hotspot or captive portal)
			local
			group
		fsso (sysytem login)(port 8002)(just users login use for authentication with system login)

	users & devices > user definition 
		group users 
			type :
				hotspot :
					firewall
					guest (we can set limited time access)
				sso :
					radius
					fortinet
		authentication setting
			time out 1440 maximum time each user can login on day
			login pages
			ftp + http + https + redirect http to https (certificate) + telnet

		device inventory
			detect dvices type

	monitor > firewall user monitor

	users and devices > guest manager :
		define range or batch of users and set details

	to use active directory or any directory service :
		users & devices > ldap server :
			for poll mode of single sign on must set this at first

		users & devices > single sign on :
			poll mode :
				add ip and port
				then add destination name in browser list after successfull connection
				security fabric > fabric connectors 
					fsso + polll active directory
						works with third party
				then users and devices > user groups (add local agents)
				after this goes to policy > ipv4 set source (range and users)
			
			fsso (passive mode) :
				if use many active directory in your network
				must install fsso agent on clients on advance setup mode
				after installation agent must use dc agent in wizard
				
					before these must set ldap server if need install fsso on advance mode and set them on user and deevices
					
					user > domain\administrator
					pass >  
					select advance mode installation
					save ntlm (enable) (network lan manager)
					monitor (enable)
				
				works on poort 8002

				use dc agent mode after groups an domain selection
				after reboot agent system must run collector agent in automatic mode
				in fsso app we ave authentication part
					here use
						Require authenticated connection from FortiGate
							this option help to authentication in fetching data
						also enable log logon events and could set cache 
				
				if have many ad in our organization must setup this

				security fabric > fabric connectors (fsso)

				user group source > collector agent (active directory) , local (ldap server)

				user and devices > authentication > sigle sign on >fsso
					name
					primary ip
					password

				must disable agent firewall
				must use dns on interface like organization dns
				also set access for active directory windows to internet base on ipv4 policy

				execute fsso refresh

				users and devices > user groups > fsso
				after this goes to policy > ipv4 
					incoming interface > port1
					source address > all
					source users > fsso-users
					source device > all
					outgoing interface > wan
					destination address > all
					schedule > always
					action >accept
					service > all
					nat > enable

				use & device > moonitor > firewall (show fsso)

			radius sso agent
				nas ip > network access server is the frontline of authentication it's the first server that fields network authentication requests before they pass through to the radius

		users & devices > user groups :
			types :
			fsso : add groups
			firewall : ldap users add (domains user pass get authenticate)

	* after this must add rule in ipv4 policies and define microsoft ad service to it services (ping + dhcp + dns + windows ad)

	users and devices 
		 device inventory :
			scan device on interface setting (l2 get analyze)
		custom devices & groups (add filter type by devices in source rule adding)
		authentication setting (authentication for fortinet with telnet, ftp, https, http)
		timeout for loged in users
		forti token

	user and devices > authentication
		ldap server
			server ip 
			cnid > sAMAccountName\cn
			dn > dc=respina,dc=local
			bind type > regular
			user dn > administrator@respina.local

			can test connectivity

			config user ldap
			edit LDAP
			set server 10.10.20.3
			set cnid sAMAccountName\cn
			set dn dc=respina,dc=local
			set type regular
			set username administrator@respina.local
			set password
			next

		user and devices > user > user group
			remote gateway > ldap
			dn > firewall
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
VPN :
	better use site to site vpn with routers 
	use firewall for ra vpn

	ssl or certificate vpn :
		system > setting
			certificate to join (after check this must set required clients cert)
				system > certificate
					add server side and client side certificate with importing ca mode cert to create cert
					to use certificate authentication, you must first create pki users
						config user peer
						edit rdiaz
						set ca CA_Cert_1
						set subject User01(must be same with user certificate)

					after this must make our page refresh then ges to 
						user & devices > pki
							select 2 factor authentication
							give a password to users 
								name > rdiaz
								subject > user01
								ca > ca-cert-1

								required 2 factor authentication (enable it)
								password

						user & devices > users > user group
							name > ssl-users
							type > firewall
							memeber > rdiaz

		vpn > ssl > portals 
			here we have splite tunnel option
				can add rotes to inject in agent routing table
			use tunnel mode also can usse web mode
				in web mode we can add some bookmarks and addresses
			limit users to one connection (better disabel it make our connection per user one connection)
			define ip pool for ssl users (default use 10.212.134.200 - 10.212.134.210)
			allow users save password
			allow users connect auto
			allow users keep connection alive
			split tunnel dns
			enale user bookmark

		vpn > ssl > setting 
			select the port connected to the internet for the listen on interface(s) option. you can also select your desired port in the listen on port option. 
			better chane port in this section
			in the specify custom ip ranges section, you can select the desired ip address to assign to remote users who connect to the internal network. it should be noted that by default an ip range is considered for this purpose in fortigate, and if you wish, you can change it and use your desired ip addresses

			authentication portal mapping (who can access)
			idle time out
			restrict source to connect \ allow any source (better use this then set authorization in policy)

			for the server certificate option, select the same certificate that you created in the previous steps. also activate the require client certificate option.

			we have ip address range deployement for connected ssl vpn users
			dns and allow end point registeration are here

			then set portls

		policy & objects > ipv4 policy
			incoming interface > ssl-vpn
			outgoing interface > lan + dmz
			source > ssl-vpn + all
			destination > all \ specific server
			service > all
			nat enable
			inspection > flow base
			action > enable

				*can use regract src and dst to set bidirectional policy*

		then install forti client on pc and setup it to connect server next try connection
		also can use https://192.168.1.1:4433 in browser
		if want use web mode must use firefox 13 and change > about:config > security >  security ssl3 rsa rc4 40 md5  > true

	site-site (untrust media)(private ip used in senanio):
		gre (cli)
		ipsec
		gre + ipsec

		GRE:
			section 1 : 
				config system gre tunnel
				edit tunn- 1 (the name)
				set interface port 1 (wan nterface or egress interface)
				set localgw 9.9.9.9 (ip public of this site edge)
				set remotegw 8.8.8.8 (front veiw ip public)
				end
	
			section 2 :
				config system interface
				edit tunn-1
				set interfcae 
				set allow access ping
				set ip 10.10.10.1 255.255.255.255
				set remote ip 10.10.10.2 255.255.255.252
				end
	
				show system interface gre \ tunn-1
	
			section 3 :
				config route static
				edit 7
				set dst 192.168.30 255.255.255.0 (private ip range front view site wiil be visible on tunn-1)
				set device tunn-1
				next 
				edit 8
				set dst 172.16.100.0 255.255.255.0 
				set device tunn-1
				exit
	
				must set policy :
					config firewall policy
					edit 4
					set name lan-shiraz-gre
					set srcintf zone-lan
					set dstintf tunn-1
					ser srcaddr vlan10 vlan20
					ser dstaddr all
					set service ping
					set schedule always
					set logtraffic all
					set action allow
					end
	
			to set ip in cli must #set mode static because it is dynamic
			must set these to front view site
	
				edit 85 (sequence number)
				set dst 0.0.0.0 (destination)
				set gateway 192.168.10.1 \ 9.9.9.9 (edge or egress interface)
				set device port 80 (where did we set this)
		
				port 2
				setstatus down (shutdown)
		-------------------------------------------
		IPSec :
			vpn > wizard
				remote device type :
					forti
					cisco
					others (custom)
	
				nat config :
					no nat
					i am behinde nat (if had ip public by default can detect that)
					remote side is behinde nat
	
				if  have fortinet device on remote site must :
					last step must set local on zone-lan then front veiw ip address
					must add blackhole if our ipsec was down don't read routes
	
				monitro > ipsec monitor
	
				if have cisco device on remote side must define and set these same things each side :
					phase 1 of ipsec :
						des sha 1
						des sha 5
						 group 5
						 dpd > on demand
	
					phase 2 of ipsec :
						group 5
						proposal des md5

					config vpn ipsec phase1-interface
    				edit "Cisco-VTI"
       				set interface "port1"
       				set dhgrp 2
       				set proposal aes128-sha1
       				set remote-gw 172.16.55.1
       				set psksecret pass123
   					next
					end
					config vpn ipsec phase2-interface
				    edit "Cisco-P2-1"
				    set phase1name "Cisco-VTI"
				    set proposal aes128-sha1
				    set dhgrp 2
				    next
					end

					edit "Cisco-VTI"
        			set vdom "root"
        			set ip 1.1.1.1 255.255.255.255
        			set allowaccess ping https ssh
        			set type tunnel
        			set remote-ip 192.168.111.2
        			set interface "port1"

        			config firewall policy
   					edit 1
        			set srcintf "port2"
        			set dstintf "Cisco-VTI"
        			set srcaddr "all"
        			set dstaddr "all"
        			set action accept
        			set schedule "always"
        			set service "ALL"
				    next
    				edit 2
			        set srcintf "Cisco-VTI"
			        set dstintf "port2"
			        set srcaddr "all"
			        set dstaddr "all"
			        set action accept
			        set schedule "always"
			        set service "ALL"
	
					on cisco router :
						must set nat at the first step
						phase 1 :
							crypto isakmp policy 1
							authentication pre
							encryption des
							hash  md5
							group 5
							exit
		
							crypto isakmp key 123456 address 192.168.20.252
	
						phase 2 :
							ip access-list extended my-acl
							permit ip 192.168.50.0 0.0.0.255 any
							permit ip any 192.168.50.0 0.0.0.255
							exit

							crypto ipsec transform-set tr-test esp-des esp-md5-hmac
							mode tunn
							
	
							crypto ipsec profile pr-test
							set transform-set tr-test
							set group 5

							crypto map cm-test 1 ipsec-isakmp
							set peer 192.168.20.252
							set pfs group5
							set ipsec-profile pr-test
							match address my-acl
							

						optional	
							int tun 2
							ip address 10.11.13.2 255.255.255.0
							tun source 192.168.10.2
							tun destination 192.168.20.252
							tun mode ipsec ipv4
							tun protec ipsec profile pr-test\ crypto map cm-test

							ip route 8.8.8.8 255.255.255.255 tun 2
	
						do show crypto isakmp sa
						do show crypto ipsec
						tunnle add 10.10.10.2
	
				if have palo alto or sophos or junipper or ... on remote side :
					ipsec interface mode enable
					ip address (front view ip)
					must use same groups in ipsec
		-------------------------------------------
		GRE Over IPSec :
			config system global
			set hostname fortinet1
			end
			
			config system setting
			set allow-subnet-overlap enable (if you see overlap ip address take it easy)
			end
	
			config system interface 
			edit port 1
			set vdom "root"
			set mode static
			set ip add 172.16.200.1 255.255.255.252
			set allow access https http ping ssh
			set type physical
			set snmp-index 1
			next
			edit port2 
			 * here define client side (lan)
			end
	
			config system route static
			edit 1
			set gateway 172.16.200.1
			set device port 1
			end
				* if dst didn't set means this rule is default route
	
			config vpn ipsec phase1-interface
				edit greipsecif
				set interface port3
				set preetype any
				set propsal des-sha1
				set remote-gw 192.168.10.2
				set psk-secret 12345678
				next 
				end

			sh vpn ipsec phase1-interface
	
			config vpn ipsec phase2-interface 
				edit GREipsecif
				set phase1-name greipsecif
				set propsal des-sha1
				set protocol 47
				next
				end

			sh vpn ipsec phase2-interface
	
			config system interface 
			edit greipsecif
			set ip 10.10.10.1 255.255.255.252
			set remote-ip 10.10.10.2 255.255.255.252
			next
			end
	
			config system gre-tunnel
			edit gre-hq
			set interface greipsecif
			set remote-gw 10.10.10.2
			set local-gw 10.10.10.1
			next 
			end

			config router static
			edit2
			set device gre-hq
			set dst 10.1.100.0/24 (branch subnets)
			end
	
			config firewall policy
			edit 1
			set srcintf port2
			set dstintf gre-hq
			set srcaddr all
			set dstaddr all
			set action accept
			set schedule always
			set service all
			next
				*define same policy to recieve data
			edit 2
			set srcintf gre-hq
			set dstintf port2
			set srcaddr all
			set dstaddr all
			set action accept
			set schedule always
			set service all
			next
	
			if we want to enable gre on device must write rule that has same src and dst :
				edit 3
				set srcintf greipsecif
				set dstintf greipsecif
				set srcaddr all
				set dstaddr all
				set action accept
				set schedule always
				set service all
				next
				end
	
			config route static
			edit 2
			set dst 172.16.101.0 255.255.255.0
			set device gre-hq
			end

	remote-access :
		ipsec
		ssl (special mode)
		l2tp
		pptp

		*for remote access vpns must set excepted ip range fom our lan*

		config system global
		set ssl
		config system global
		set admin-https-ssl-version tlsv1.2

		ra ipsec vpn :
			agent base (fortinet or cisco client)
				posture mode and scanning files viruses ...:
					can use 10 devices in this mode for more must pay
						system > interface (the interface which use for incoming or ingress)
							must enable fct-access and identity devices 
							also in nat section enable forticlient compliant and broadcast discovery message

						policy & object > policy ipv4
							incoming > port1 (internal)
							source > all users and windows pc devices
							outgoing > port2 (external)
							destination > all
							schedule > always
							service > all
							action > accept
							nat > enable
							compliant with forticlient profile > enable

							log all traffics

						user&devices > forticlient profile
							windows and mac	
								antivirus protection > enable (use agent)
								
								web category filter > enable
									client web filtering when on-net ()
								
								vpn
									client vpn provisioning (deploy on users connected to organization with vpn)
	
								application firewall
								upload logs to faz or fortimanager
								use fortimanager for clients signature update
								dashboard banner
								client base logging when on-net (when forticlient connected through fortigate generate logs if get on)

							ios
								web category filtering
								client vpn provisioning
								distribute configuration profile (.mobilingconfig file)

							android
								web category filtering
								client vpn provisioning

						install agents on clients
						after installation our devices get detec by fortigate then check attributes and get compliance

						monitor > forticlient

						in agent after downloading full parameters must use agent application and request registeration to fortigate if didn't find search manualy
				
				ipsec with forticlient
					user & devices > user > user difinition
						define users
						
					user & devices > groups > group difinition
						define group
		
					policy  object > objects >> address
						define a range for vpn connections
		
					vpn > ipsec wizard (remote access vpns / dial up / forticlient)
						incoming interface >wan1
						authentication method > preshared key
						user groups > x
		
						policy & routing header set who can access and where did can they go
						local address > lan
						client range address > vpn range like 10.10.111.2-10.10.111.254 
						if set /32 can be isole
						use dns in specific mode or system defaults
						splite tunnel
						allow endpoint registration (posture)
		
					policy and object > ipv4
						source address > ip-vpn-range
						incoming interface > ipsecvpn
						outgoing interface > wan
						destination > all
						schedule > always
						service > all
						action > accept
						nat > enable
		
					use forti client to connection
				-----------------------------------------------------
				ssl vpn :
					modes :
						tunnel mode :
							forticlient + policy access & conection
							all clients has forticlient agent and tunnel hole connectivity (protocols and data ....)
						web mode :
							if can't buy posture mode use this
							bookmarks + policy access & conection
							clients are not entirely in organizations so didn't use specific ip range just use a little features on web panle
							protocols like : rdp, telnet, ssh, vnc, http, https, citrix, smb, cifs, portforward
		
					almost we use in organizations web mode accessability
		
					portal manager :
						users & devices > user definition
						users & devices > user groups
						vpn > ssl-vpn portals
							limt users to one ssl-vpn connection at a time
							enable tunnel mode
								splite tunnle (it's better set a special ip like 1.1.1.1 on interface and inject here to seperate traffics)
								source ip range (define a different range from our lan ip range)
							enable web mode
								user bookmark (users can create bookmarks better be disable)(except admins)
							enable forti client download
								ssl vpn proxy
								direct (better disable because use iran ip)
						vpn > ssl vpn setting
							listen interface > wan
							listen port > 443
							resrtrict access 
								any
								specific host
							tunnel mode clients (ip range)
								auto
								specific
							dns
							allow endpoint registration
								vpn or posture
							authentication portal mapping (who joined to which portl)(default full access and use entire bookmarks)

							certificate (required means most use cert if use forticert don't set enabe if use ca use required then send to users and install on pc)
		
						policy & objects > ipv4
							incoming > ssl vpn interface
							outgoing > dmz \ srv
							source > group x + all devices
							destination > srv-dmz
							service > all
							schedule > always
							nat > enable
							action > accept
		
						*for each portal need policy

			aget less (windows client) (l2tp \ ipsec)
				ipsec with l2tp :
					config vpn l2tp
					set status enable
					set enforce-ipsec
					set sip 10.10.10.1 255.255.255.0 (start and end ip range)
					set eip 10.10.10.254 255.255.255.0
					set usrgrp it-local 
					end
		
					policy & object
						addrress 
							create a range like down
					
					vpn > ipsec > tunnels
						custom vpn
							remote gw > dialup users
							interface > wan
							nat traversal > enable
							keepalive > 10
							dead peer detection > enable
							authentication method > preeshared key
							ike > v1
							mode main
							accept typrs > any peer id
		
							then goes to ipsec pannel 
		
							define phase 2 with name and set transparent mode
								config vpn ipsec phase2
								edit IPSec-VPN-Test2
								set encapsulation transport-mode
						
					system > config > feature
						policy base ipsec vpn
		
					policy & objects > ipv4
						incoming interface > lan/dmz
						outgoing interface > wan
						source > it-local + all (it users can login from any deviecs)
						destination > all
						services > all
						action > ipsec
							use exist tunnel 
							allow traffic to be initiated from the remote site
		
		
					policy & objects > ipv4
						incoming interface > wan
						outgoing interface > lan
						source > it-local + all (it users can login from any deviecs)
						destination > lan
						services > all
						nat > enable
						action > accept
				----------------------------------------------
				pptp (l2tp is same with this) :
					config vpn pptp
					set status enable
					set ip-mode range
					set sip 10.10.10.1 255.255.255.0 (start and end ip range)
					set eip 10.10.10.254 255.255.255.0
					set usrgrp it-local 
					end
		
					policy & object
						addrress 
							create a range like down
		
					policy & objects > ipv4
						incoming interface> wan
						outgoing > lan/dmz
						source > it-local + all (it users can login from any deviecs)
						destination > srv-dmz
						services > all
						nat > enable
						action > accept	
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
