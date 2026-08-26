PFsense
	installation :
		in 2 version we can use 
		amd64 (intel and amd)
		netgate (appliance)
			c series is more power full

		image of os can be usb (vga \ usb) or iso image

		tnsr use for centos

		in older versions we use ufs but in earlier versions zfs
			zfs
				oracle
				2005
				maximum volume size 2 pow 128
				maximum file size 2 pow 64
				maximum filename length 255 byte
				case sensetive yes
				support for file system level encryption yes 
				data duplication yes
				data checksums yes
			ufs
				unix
				1983
				maximum volume size 2 pow 73
				maximum file size 2 pow 73
				maximum filename length 255 byte
				case sensetive yes
				support for file system level encryption no
				data duplication no
				data checksums no

			also can create apple partition map (APM)

		better use zfs on apk gate
		after these define interfaces with console (assign interface)
		in installation must set swap size on 8 gig

		login on shell apk gate :
			user : root
			password : APKSec3444@ateway

		login web :
			first time 
				http://10.10.10.1

			another times
				https://10.10.10.1:3444

				user : apksup
				password : @pk-g@te@!

				user : admin
				password : @pk-g@te

		ssh apk gate :
			port 2332
			user : root
			password : APKSec3444@ateway					

		we have options on consol :
			options :
				1 > ssh logout
				2 > ip set
					better don't use dhcp on apk gate interface management
					don't set vlan number
					better set ip range
				3 > reset gui password or admin to pfsense
				6 > safe  power off (halt)
				10 > pfsense logs
				12 > php shell can change access from web
					playback alloaccesswan
				13 > update
				15 > restore points
	
				on shell must set :
					pfctl -d
						disable all rules till 2 minutes to access web and change values and set all services to management interface
	
			cp /etc/rc.banner /root/
			
			ee /etc/rc.banner
			ee /etc/rc.initial
				we can set many more options in this section (script shell use casesystem)
					'' echo 200)	network reset
						200)	/etc/rc.d/netif restart
						;;
			pkg install bash
	
			chmod 777 (on rc.initial)
			mount -a
			rm /etc/rc.initial
			mount /dev/ufsid/5a....	/mnt/
			unmount -f /dev/ufsid/5a....

	sql on gate :
		on putty must ssh to device then :

		pkg add /etc/manual/phpMyAdmin/phpMyAdmin-php72-4.8.4.txz
		pkg add /etc/manual/phpMyAdmin/jpeg-turbo-1.5.3.txz

		restart support > services > mysql

		10.10.10.1\phpMyAdmin

	boot on usb :
		freebsd
		use virtual box
			add storage hardware
			usb > add usb

			if use rufus :
				must use these setting
					mbr (master boot record) and bios 

					guid partition table (gpt)
					unified extensible firmware interface (UEFI)

	pfsense menue :
		system 
			advance
				admin access
					web configurator
						sorting of gui and change theme

						protocols
							ssl\tls (default)
							http

						ssl\tls certificate
							certificates known to be incompatible with use for https are not included in this list

						tcp port

						max processes
							enter the number of webConfigurator processes to run this defaults to 2 increasing this will allow more users/browsers to access the gui concurrently

						webgui redirection
							when this is unchecked, access to the webConfigurator is always permitted even on port 80, regardless of the listening port configured check this box to disable this automatically added redirect rule

						hsts

						oscp must staple
							when this is checked oscp Stapling is forced on in nginx remember to upload your certificate as a full chain not just the certificate or this option will be ignored by nginx
						
						webgui login autocomplete
							default is enable
								login credentials for the web configurator may be saved by the browser while convenient some security standards require this to be disabled check this box to enable autocomplete on the login form so that browsers will prompt to save credentials (some browsers do not respect this option)
						
						webgui login message
							disable successful login message on logs
						
						anti lockout
							access to the webconfigurator on the WAN interface is always permitted regardless of the user-defined firewall rule set check this box to disable this automatically added rule so access to the webconfigurator is controlled by the user-defined firewall rules (ensure a firewall rule is in place that allows access to avoid being locked out) Set interface ip address option in the console menu resets this setting as well
						
						dns rebind check
								when this is unchecked the system is protected against dns rebinding attacks this blocks private ip responses from the configured dns servers check this box to disable this protection if it interferes with webconfigurator access or name resolution in the environment
						
						alternative hostnames
							for dns rebinding and http_referer checks specify alternate hostnames by which the router may be queried to bypass the dns rebinding attack checks separate hostnames with spaces
						
						browser http_referer enforcement
							when this is unchecked access to the webconfigurator is protected against http_referer redirection attempts check this box to disable this protection if it interferes with webConfigurator access in certain corner cases such as using external scripts to interact with this system
						
						browser tab text
							when this is unchecked the browser tab shows the host name followed by the current page check this box to display the current page followed by the host name
					
					secure shell (ssh)
						if need use ssh in organization must enable this feature at this path
						ssh key only 
							password or public key
							public key only
							password and public key (both)
						
						allow agent forwarder (on gate is disable)
						
						ssh port (can cahnge port)(default is 22)(on gate is 2332)

						https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

							we can generate a key ssh-v2 from this then paste on pfsense (system > usermange) in key section and authorize ssh key

								then run putty and add ip address to session
					
					login protection
						threshold
							block attackers when their cumulative attack score exceeds threshold most attacks have a score of 10
						
						block time
							block attackers for initially blocktime seconds after exceeding threshold subsequent blocks increase by a factor of 1.5 attacks are unblocked at random intervals so actual block times will be longer
						
						detection time
							remember potential attackers for up to detection time seconds before resetting their score
						
						passlist
							ipv4 
							ipv6
					
					serial communicatons
						serial terminal
							enables the first serial port with 115200/8/N/1 by default or another speed selectable below
						
						serial speed
							default is 115200
						
						primary console
							serial console
							vga console
								select the preferred console if multiple consoles are present The preferred console will show pfsense boot script output all consoles display os boot messages console messages and the console menu
					
					console options
						console menue
							password protect the console menu
				
				firewall&nat
					packet processing
						ip don't fragment compatibility
							this allows for communications with hosts that generate fragmented packets with the don't fragment (df) bit set linux nfs is known to do this this will cause the filter to not drop such packets but instead clear the don't fragment bit
						
						ip random id generator
							replaces the ip identification field of packets with random values to compensate for operating systems that use predictable values this option only applies to packets that are not fragmented after the optional packet reassembly
						
						firewall optimization options
							normal
							aggressive
							high latency
							conservative
						
						disable firewall scrub
							disables the pf scrubbing option which can sometimes interfere with nfs traffic
						
						firewall adaptive timeout
							option one
								when the number of state entries exceeds this value adaptive scaling begins all timeout values are scaled linearly with factor
								default is 60 percent
							
							option two
								when reaching this number of state entries all timeout values become zero effectively purging all state entries immediately this value is used to define the scale factor it should not actually be reached
								default is 120 percent

							timeouts for states can be scaled adaptively as the number of state table entries grows leave blank to use default values set to 0 to disable adaptive timeouts
						
						firewall maximum state
							403000
							max number of connections to hold state table
						
						firewall maximum table entires
							400000
							maximum number of table entries for systems such as aliases sshguard, snort, etc, combined
						
						firewall maximum fragment entires
							5000
							maximum number of packets fragment to hold for reassembly by scrub rules
					
					vpn packet processing
						these setting use for ipsec openvpn pppoe
						ip don't fragment compatibility
							clear invalid df bits instead of dropping the packets
						ip fragment ressemble
							reassemble ip fragments until they form a complete packet
								reassemble ip fragments for normalization in this case, fragments are buffered until they form a complete packet and only the completed packet is passed on to the filter the advantage is that filter rules have to deal only with complete packets, and can ignore fragments the drawback of caching fragments is the additional memory cost
						enable maximum mss
							enable mss clamping on vpn traffic
					
					advance options
						disable firewall
							make our device routing mode and disable nat

						static route filtering
							this option only applies if one or more static routes have been defined if it is enabled traffic that enters and leaves through the same interface will not be checked by the firewall this may be desirable in some situations where multiple subnets are connected to the same interface

						disable auto added vpn rule

						disable reply to
							with multi-wan it is generally desired to ensure traffic leaves the same interface it arrives on, hence reply-to is added automatically by default. When using bridging, this behavior must be disabled if the wan gateway ip is different from the gateway ip of the hosts behind the bridged interface.

						disable negate rules
							with multi-wan it is generally desired to ensure traffic reaches directly connected networks and vpn networks when using policy routing this can be disabled for special purposes but it requires manually creating rules for these networks

						allow apipa
							some providers may utilize apipa space for interconnect interfaces

						aliases hostnames resolve interval
							300 seconds

						check certificate of aliases urls
							verify https certificates when downloading alias urls
							if was revoke don't download it

					bogon networks
						update inervals 
							monthly
							weekly
							daily

					state timeouts
						tcp first, tcp opening, tcp established, tcp closing, tcp fin wait, tcp closed, tcp tsdiff, icmp error, udp first, udp multiple, udp single, icmp first, other first, other single, other multiple
				
				networking
					ipv6
						allow ipv6
							if disable this , block ipv6 traffics
							all ipv6 traffic will be blocked by the firewall unless this box is checked (is enable)

						ipv6 over ipv4 tunneling
							These options create an rfc 2893 compatible mechanism for ipv4 nat encapsulation of ipv6 packets that can be used to tunnel ipv6 packets over ipv4 routing infrastructures (is disable)
						
						prefer ip4 over ipv6
							by default, if ipv6 is configured and a hostname resolves ipv6 and ipv4 addresses, ipv6 will be used (is disable)
						
						ipv6dns entry
							do not generate local IPv6 DNS entries for LAN interfaces
							

						dhcp6 duid
							raw duid as store in duid file or seen in firewall logs
							duid llt base on link layer address plus time
								time : 708859507
								linklayer address
							duid en assigned by vendor base on enterprise network
								variable length
								iana private enterprise number
							duid ll base on link layer address
								link layer address
							duid uuid base on universally unique identifier
								universally unique identifier

					network interfaces
						hardware chechsum ffloading
							checksum offloading is broken in some hardware particularly some Realtek cards. Rarely drivers may have problems with checksum offloading and some specific nics this will take effect after a machine reboot or re-configure of each interface
							
							Disable hardware checksum offload (default is disable)

						hardware tcp segmentation offloading
							disable hardware tcp segmentation offload (default enable)
							checking this option will disable hardware tcp segmentation offloading (TSO, TSO4, TSO6)

						
						hardware large recieve offloading
							disable hardware large receive offload (default is enable)
							the altq support disables the multiqueue api and may reduce the system capability to handle traffic this will take effect after a machine reboot

						hn altq support
							enable the altq support for hn nics (default is enable)

						arp handling
							Suppress ARP messages (default is disable)

						reset all state
							this option resets all states when a wan ip address changes instead of only states associated with the previous ip address
							default is disable

				miscellaneous
					proxy support
						proxy url
						proxy port
						proxxy username
						proxy password

					loadbalance
						loadbalancing
							use sticky connection
								successive connections will be redirected via gateways in a round-robin manner with connections from the same source being sent via the same gateway will exist as long as there are states that refer to this connections

							second box
								set the source tracking timeout for sticky connections in seconds by default this is 0 so source tracking is removed as soon as the state expires

					power saving
						pwerd
							the powerd utility monitors the system state and sets various power control options accordingly. It offers four modes (maximum, minimum, adaptive and hiadaptive) that can be individually selected while on AC power or batteries

								adaptive mode attempts to strike a balance by degrading performance when the system appears idle and increasing it when the system is busy. It offers a good balance between a small performance loss for greatly increased power savings. hiadaptive mode is alike adaptive mode, but tuned for systems where performance and interactivity are more important than power consumption. It raises frequency faster, drops slower and keeps twice lower cpu load.

								ac power
								battery power
								unkown power

					cryptographic & thermal hardware
						crypto graphic hardware
							none
							aes ni cpu base accceleration
							bsd crypto devices (cryptodev)
							aes ni and bsd crypto devices (aesni, cryptodev)

								cryptographic accelerator module will use hardware support to speed up some cryptographic functions on systems which have the chip
								loading the bsd crypto device module will allow access to acceleration devices using drivers built into the kernel, such as hofn or ubsec chipsets.
								if the firewall does not contain a crypto chip, this option will have no effect. To unload the selected module, set this option to "none" and then reboot.

						thermal sensor
							none\acpi
							intel core cpu ondie thermal sensor
							amd k8 k10 k11 cpu on die thermal sensor

								with a supported cpu, selecting a thermal sensor will load the appropriate driver to read its temperature
								setting this to "none" will attempt to read the temperature from an acpi-compliant motherboard sensor instead, if one is present
								if there is not a supported thermal sensor chip in the system, this option will have no effect to unload the selected module, set this option to "none" and then reboot


					kernel page table isolation
						meltdown workaround if disabled the kernel memory can be accessed by unprivileged users on affected cpu option forces the workaround off, and requires a reboot to activate

						pti is active by default only on affected CPUs, if PTI is disabled by default then this option will have no effect current pti status:
						 	Enabled

					microarchitectual date sampling mitigation
						 if disabled the kernel memory can be accessed by unprivileged users on affected cpu this option controls which method of mds mitigation is used, if any

						 values :
						 	default
						 	mitigation disabled
						 	verw instruction (microcode) mitigation enable
						 	software sequence mitigation enabled (notrecommend)
						 	automatic verw or software selection

					schedules
						do not kill connections when schedule expires (default is disable)
						when a schedule expires, connections permitted by that schedule are killed option overrides that behavior by not clearing states for existing connections

					gateway mirroring
						state killing on gateway failure
							flush all states when a gateway goes down
							disable is default

						skip rule when gateway is down
							do not create rules when gateway is down
								disable is default
								when a rule has a gateway specified and this gateway is down, the rule is created omitting the gateway overrides that behavior by omitting the entire rule instead

					ram disk setting (reboot to apply changes)
						use ram disk
							use memory file system for /tmp and /var
							set this to use /tmp and /var as RAM disks (memory file system disks) on a full install rather than use the hard disk
							the directories data ommit
							rdp + dhcp + captive portal + ad logs save	

						ram disk size
						periodic ram disk dat backups
							interval of each part backup

					hardware setting
						hard disk standby time
						puts the hard disk into standby mode when the selected number of minutes has elapsed since the last access do not set this for cf cards

						always on or 3x timming

					installation feedback
						don't send netgate device id with user agent

				system tunables
					net.inet.ip.portrange.first		1024
					net.inet.tcp.blackhole	Do not send RST on segments to closed ports	2	
					net.inet.udp.blackhole	Do not send port unreachables for refused connects	1	
					net.inet.ip.random_id	Assign random ip_id values	1	
					net.inet.tcp.drop_synfin	Drop TCP packets with SYN+FIN set	1	
					net.inet.ip.redirect	Enable sending IP redirects	1	
					net.inet6.ip6.redirect	Send ICMPv6 redirects for unforwardable IPv6 packets	1	
					net.inet6.ip6.use_tempaddr	Create RFC3041 temporary addresses for autoconfigured addresses	0	
					net.inet6.ip6.prefer_tempaddr	Prefer RFC3041 temporary addresses in source address selection	0	
					net.inet.tcp.syncookies	Use TCP SYN cookies if the syncache overflows	1	
					net.inet.tcp.recvspace	Initial receive socket buffer size	65228	
					net.inet.tcp.sendspace	Initial send socket buffer size	65228	
					net.inet.tcp.delayed_ack	Delay ACK to try and piggyback it onto a data packet	0	
					net.inet.udp.maxdgram	Maximum outgoing UDP datagram size	57344	
					net.link.bridge.pfil_onlyip	Only pass IP packets when pfil is enabled	0	
					net.link.bridge.pfil_member	Packet filter on the member interface	1	
					net.link.bridge.pfil_bridge	Packet filter on the bridge interface	0	
					net.link.tap.user_open	Allow user to open /dev/tap (based on node permissions)	1	
					net.link.vlan.mtag_pcp	Retain VLAN PCP information as packets are passed up the stack	1	
					kern.randompid	Random PID modulus. Special values: 0: disable, 1: choose random value	347	
					net.inet.ip.intr_queue_maxlen	Maximum size of the IP input queue	1000	
					hw.syscons.kbd_reboot	enable keyboard reboot	0	
					net.inet.tcp.log_debug	Log errors caused by incoming TCP segments	0	
					net.inet.tcp.tso	Enable TCP Segmentation Offload	1	
					net.inet.icmp.icmplim	Maximum number of ICMP responses per second	0	
					vfs.read_max	Cluster read-ahead max block count	32	
					kern.ipc.maxsockbuf	Maximum socket buffer size	4262144	
					net.inet.ip.process_options	Enable IP options processing ([LS]SRR, RR, TS)	0 (0)	
					kern.random.harvest.mask	Entropy harvesting mask	351	
					net.route.netisr_maxqlen	maximum routing socket dispatch queue length	1024	
					net.inet.udp.checksum	compute udp checksum	1	
					net.inet.icmp.reply_from_interface	ICMP reply from incoming interface for non-local packets	1	
					net.inet6.ip6.rfc6204w3	Accept the default router list from ICMPv6 RA messages even when packet forwarding is enabled	1	
					net.key.preferred_oldsa		0	
					net.inet.carp.senderr_demotion_factor	Send error demotion factor adjustment	0 (0)	
					net.pfsync.carp_demotion_factor	pfsync's CARP demotion factor adjustment	0 (0)	
					net.raw.recvspace	Default raw socket receive space	65536	
					net.raw.sendspace	Default raw socket send space	65536	
					net.inet.raw.recvspace	Maximum space for incoming raw IP datagrams	131072	
					net.inet.raw.maxdgram	Maximum outgoing raw IP datagram size	131072	
					kern.corefile	Process corefile name format string	/root/%N.core	

				notifications
					can tune smtp and emails slak, telegram and have push over here
					startup and shutdown sound is here

			certmanager
				certificate authoritys (cas)
				certificates
				certificate revocation

			general setting or setup
				system :
					hostname
					domain

					on appk we have :
						hostname > utm
						domain > APKGate
				
				dns server setting :
					dns server

					if unchecked dns server override, the wan dns auto detect can't be use and use internal dns if don't use this option our policies wll be on isp side and if be enable our dns will be on isp dns resolvers

					dsn server override
						use dhcp server dns ip address
							must enable it on apk gate

							if use pppoe with isp connection, lost connection or disable this >  use local dns servers in manual mode

					dns resolution behavior (or dns forwarder)
						local dns 127.0.0.1 (fall back to remote dns)(default)
						local dns 127.0.0.1 (ignore remote dns)
						use remote dns ignore local dns
							on apk gate disable this option
				
				localization (ntp)
					timezone
						asia\tehran
					time server
						0.ir.pool.ntp.org 1.ir.pool.ntp.org 2.ir.pool.ntp.org
					
					language
						can cahnge gui lang

				web configurator
					themes
					login page color
					login hostname
					interface sort 
						if selected, lists of interfaces will be sorted by description, otherwise they are listed wan,lan,optn...
					association pannel :
						available widgets
							show the available widgets panel on the dashboard
						log filters
							show the log filter panel in system logs
						manage log
							show the manage log panel in system logs
						monitoring system
							show the settings panel in status monitoring

								these options allow certain panels to be automatically hidden on page load a control is provided in the title bar to un-hide the panel

					host name in menue :
						default no name
						hostname only
						fully fqdn
					require state filter
						by default, the entire state table is displayed when entering diagnostics > states. this option requires a filter to be entered before the states are displayed. useful for systems with large state tables
					disable dragging
						disables dragging rows to allow selecting and copying row contents and avoid accidental changes.
					login hostname 
						Show hostname on login banner
					alias popups
						if selected, the details in alias popups will not be shown, just the alias description (e.g. in Firewall Rules)

						Version 2.4.3 has added cross-site request forgery (CSRF) or xsrf protection to the dashboard widgets
			
			logout

			package manager
				routing packages :
					system >  routing
					system > packages :
						routed
						quagga ospf
						open bgp
					services > roting protocols

			register

			routing
				gateways
				static routes
				gateway group
			
			setup wizard

			ha :
				synchronous protocols :
					pfsync
						pfstate
					xml-rpc
						configs

				redundancy with ha :
					loadbalancing :
						gateway
							use many links for wan
								interface :
									assignment (pfsense and client system)
									opt1
										enable
										static ip
										ip 
										gateway (must enable nat)
								system :
									routing
										gateway group
											tier
												if be same works like failover
											triggeer level
												packet lost and high latency
									advance
										miscellunous
											default gateway switching (active switch network)
						server (can works like pat or dnat to publish services)
							server side
								use pool and virtual ip address
									services :
										loadbalancing
											pools
												loadbalancing \ failover
												monitor
												port (alias)
												server ip
											virtual server
												ip address > wan ip
												virtual server pool
												port (if change all setting on default change-if don't change our server pool get detect)
									firewall
										rule
											wan
									system 
										advnace
											loadbalancing
												sticky connection (if some body use a server must use it for alltime except connection lost or close)
								status > loadbalncing
						client side
					carp
						many hosts use one ip address (one master and many slave)
							if had problem with cap must use icmp \ nat or check firewall rules
								must use 3 interface 
									internet
									lan
									pfsense link
		
								for pfsense link must set rule on firewall
		
								system 
									ha
										state synchronous setting
											sync state (enable)
											sync interface
											peer ip
										xmlrpc
											sync config to ip
											remote system user\password
								firewall
									virtual ip
										type :

										carp
											interface
												wan > ip > user\password
												lan > ip > user\password

										ip alias
											if we want use more than one ip address on each interface must use this option
											we can use in nat

										proxy arp
		
								status > carp (failover)
					
				state synchronization setting (pfsync)	
					synchronize state
					synchronize interface
						if Synchronize states is enabled this interface will be used for communication
						it is recommended to set this to an interface other than lan dedicated interface works the best
						ip must be defined on each machine participating in this failover group
						ip must be assigned to the interface on any participating sync nodes

					pssync synchronize peer ip
						setting this option will force pfsync to synchronize its state table to this IP address. The default is directed multicast

				configuration synchronous setting (xmlrpc sync)
					synchronize config to ip
						enter the ip address of the firewall to which the selected configuration sections should be synchronized

					remote system username
						enter the webconfigurator username of the system entered above for synchronizing the configuration
						do not use the synchronize config to ip and username option on backup cluster members

					remote system password
						enter the webconfigurator password of the system entered above for synchronizing the configuration
						do not use the Synchronize Config to IP and password option on backup cluster members

					synchronize admin
						admin account does not synchronize, and each node may have a different admin password
						automatically updates xmlrpc remote system password when the password is changed on the remote system username account

					select options to sync
						User manager users and groups
						Authentication servers (e.g. LDAP, RADIUS)
						Certificate Authorities, Certificates, and Certificate Revocation Lists
						Firewall rules
						Firewall schedules
						Firewall aliases
						NAT configuration
						IPsec configuration
						OpenVPN configuration (Implies CA/Cert/CRL Sync)
						DHCP Server settings
						DHCP Relay settings
						DHCPv6 Relay settings
						WoL Server settings
						Static Route configuration
						Virtual IPs
						Traffic Shaper configuration
						Traffic Shaper Limiters configuration
						DNS Forwarder and DNS Resolver configurations
						Captive Portal

			update :
				system update
					branch
						latest stable version
						latest deployement snapshot (experimental)
						previous stable version (depricated)

				update setting
					dashboard checck
						Disable the Dashboard auto-update check (disable by default)

			usermanger
				users
					add
					action 
					passwords
					group memeber
					keys
						authorized ssh key
						ipsec preshared key
				groups
				setting
				authenticate servers

		interfaces
			in apk gate we have dhcp on lan interface (ether 2), wan on ether 1 and last interface is apksupport interface

			apk support interface attributes :
				10.10.10.0/24
				no dhcp runing

			at first login to apk must use :
				http://10.10.10.1

				then use :	
					https://10.10.10.1:3444

			assignment
					interface assignment
						we can add new undefined interfaces 
						if select them can set interface mode and set ip address on them

						general configuration
							enable or disable interface
							description
							ipv4 config type
								ppp
								static ip
								dhcp 
								pptp (encryption with less security)
								pppoe
								l2tp (no encryption)

							ipv6 config type 
								dhcpv6
								6rd tunnel
								slaac
								6to4 tunnel
								static ipv6
								track interface

							mac address
							mtu
							mss
							speed and duplex

								better don't change values or set some things

						dhcp client configuration
							advance option
								protocol timming
									timeout
									retry
									select timeout
									reboot
									backoff cutoff
									initial interval

								presets 
									freebsd default
									clear
									pfsense default
									saved cfg

							configuration override

							hostname
								value in this field is sent as the dhcp client identifier and hostname when requesting a dhcp lease isps may require this (for client identification)

							alias ipv4 address
								fixed ip for dhcp client

							reject lease from
								have the dhcp client reject offers from specific dhcp servers, enter their ip addresses here (separate multiple entries with a comma) useful for rejecting leases from cable modems that offer private ip addresses when they lose upstream sync


						dhcp6 client configuration
							use ipv4 connectivity as parent interface
								request a ipv6 prefix/information through the ipv4 connectivity link

							request only an ipv6 prefix
								just prefix do not request an ipv6 address

							dhcpv6 prefix delegation size
								normally specified by the isp

							send ipv6 prefix hint
								send an ipv6 prefix hint to indicate the desired prefix size for delegation

							do not wait for a ra
								some isp need this especially those not using pppoe

							do not allow pd/address release
								dhcp6c will send a release to the ISP on exit, some ISPs then release the allocated address or prefix. This option prevents that signal ever being sent

						
						reserved networks
							block private networks and loopback addresses
							block bogon networks

					interface group
						hotspot :
							interface 
								assignment
								interface group (use group)
					
								wireless (edit)
									set mode on access point
								opt1
									enable
									static ip
									mode acceess point
									channel > 11 b\g 7 (use for 7 clients support 11)
									wpa (password can set)
									wireless qos 
									ssid
					
									https is better option for captive portal users
					
							services >	dhcp :
									set on opt1 interface
						
							services > captive portal 
								voucher
									rsa key 
										public
										private
								test voucher
								authentication
									voucher and local users
					
							better set pfsense ip address on https on captive portal
								cname must be same with pfsense ip address
					
							firewall > rules
								must set rule

					wireless

					vlans
						parent interface
						vlan tag
						vlan priority
						description

					qinq
						parent interface
						first level tag
						options
							adds interface to qinq interface groups
								rules to be written more easily

						members
						tags

					ppp
						link type
						link interface 
						username
						password

						advance
							dial on demand
								causes the interface to operate in dial-on-demand mode. don't enable if the link is to remain continuously connected
								interface is configured, but the actual connection of the link is delayed until qualifying outgoing traffic is detected

							idle timeout
							compression
								disable vjcomp(compression) (auto-negotiated by default)
								enables van jacobson tcp header compression, which saves several bytes per tcp data packet
								is almost always required compression is not effective for tcp connections with enabled modern extensions like time stamping or sack, which modify tcp options between sequential packets

							tcpmssfix
								causes mpd to adjust incoming and outgoing tcp cyn segments so that the requested maximum segment size is not greater than the amount allowed by the interface mtu
								necessary in many setups to avoid problems caused by routers that drop icmp datagram too big messages
								without these messages, the originating machine sends data, it passes the rogue router then hits a machine that has an mtu that is not big enough for the data. Because the ip don't fragment option is set, this machine sends an icmp datagram too big message back to the originator and drops the packet. The rogue router drops the icmp message and the originator never gets to discover that it must reduce the fragment size or drop the ip don't fragment option from its outgoing data

							shortseq
								 only meaningful if multi-link ppp is negotiated
								 it proscribes shorter multi-link fragment headers, saving two bytes on every frame
								 it is not necessary to disable this for connections that are not multi-link

							acfcomp
								address and control field compression only applies to asynchronous link types saves two bytes per frame

							protocomp
								saves one byte per frame for most frames

					gre
						tunnel mode use nat
						transparent mode don't use nat
				
						openvpn is powerfull and usefull
				
						ipsec :
							vpn
								ispsec
									ipv4
									wan
									remotegateway
				
									phase1
										authentication method
											mutual psk
										preshared key
										aes 256
										life time 28800
				
									phase2
										mode tunnel ipv4
										remote network (remote lan)
										esp-aes
										hash sha1
				
								status > ipsec (must set on connected)
							firewall > rule
								wan
									must give access to isakmp and ipsec on port 500 also esp on wan lan and ipsec ....
								ipsec
									give access to vpn on transfering
				
						gre tunnel :	
							interface
								gre
									parent interface > wan
									gre remote address > remote ip public
									gre tunnel local address > self gre ip address
									gre tunnel remote address > self gre ip address on remote side
				
								add gre interface and enable it
				
							firewall > rule > gre interface
				
							diagnose > status
							system > route
							
					gif
						generic tunneling interface is similar to gre; both protocols are a means to tunnel traffic between two hosts without encryption
						in addition to tunneling ipv4 or ipv6 directly may be used to tunnel ipv6 over ipv4 networks and vice versa

						parent interface
						gif remote address
						gif tunnel local address
						gif tunnel remote address
						gif tunnel subnet
						ecn friendly behavior
							ecn friendly behavior violates rcf2893 should be used in mutual agreement with the peer

							disable by default

						outer source filtering
							disable automatic filtering of the outer gif source which ensures a match with the configured remote peer when disabled, martian and inbound filtering is not performed which allows asymmetric routing of the outer traffic

						description

					bridge :
						advance
							advnace configuration
								cache size (2000 default entires)

								cache expire time
									1200 seconds is default
									0 is permanently cache

								span port
									 by interface as a span port on the bridge. Span ports transmit a copy of every frame received by the bridge
									 this is most useful for snooping a bridged network passively on another host connected to one of the span ports of the bridge

									 span interface cannot be part of the bridge member interfaces

								edge port
									connects directly to end stations and cannot create bridging loops in the network; this allows it to transition straight to forwarding

								auto edge ports
									allow interface to automatically detect edge status is the default for all interfaces added to a bridge
									this will disable the autoedge status of interfaces

								ptp ports
									set the interface as a point-to-point link. This is required for straight transitions to forwarding and should be enabled on a direct link to another rstp-capable switch

								auto ptp ports
									automatically detect the point-to-point status on interface by checking the full duplex link status is the default for interfaces added to the bridge interfaces selected here will be removed from default autoedge status

								sticky ports
									dynamically learned address entries are treated as static once entered into the cache sticky entries are never aged out of the cache or replaced, even if the address is seen on a different interface

								private ports
									private interface does not forward any traffic to any other port that is also a private interface

								enable ipv6 auto linklocal
									AUTO_LINKLOCAL flag is set on the bridge interface and cleared on every member interface is required when the bridge interface is used for stateless autoconfiguration

								enable rstp \ stp

							rstp \stp
								protocol

								stp interface
									if_bridge(4) driver has support for the IEEE 802.1D spanning tree protocol stp is used to detect and remove loops in a network topology

								valid time
									time that a spanning tree protocol configuration is valid
									20 seconds
									minimum 6 seconds
									maximum 40 seconds

								forward time
									time that must pass before an interface begins forwarding packets
									minimum 4 seconds
									maximum 30 seconds
									default is 15 seconds

								hello time
									 time in seconds between broadcasting of spanning tree protocol configuration messages
									 minimum is 1 seconds
									 maximum is 2 seconds
									 bpdu packets

								priority
									32768
									minimum 0
									maximum 61440

								hold count
									set the transmit hold count for spanning tree is the number of packets transmitted before being rate limited. The default is 6 minimum is 1 and the maximum is 10

								wlan priority
									set the spanning tree priority of interface to value default is 128 minimum is 0 and the maximum is 240 increments of 16


								lan priority
									set the spanning tree priority of interface to value default is 128 minimum is 0 and the maximum is 240 increments of 16


								wan path cost
									set the spanning tree path cost of interface to value. The default is calculated from the link speed change a previously selected path cost back to automatic, set the cost to 0 minimum is 1 and the maximum is 200000000


								lan path cost
									set the spanning tree path cost of interface to value. The default is calculated from the link speed change a previously selected path cost back to automatic, set the cost to 0 minimum is 1 and the maximum is 200000000

					lags
						parent interface
						lagg protocol
							none
								this protocol is intended to do nothing it disables any traffic without disabling the lagg interface itself
							lacp
								supports the ieee 802.3ad link aggregation control protocol (lacp) and the marker Protocol lacp will negotiate a set of aggregable links with the peer in to one or more link aggregated groups each lag is composed of ports of the same speed, set to full-duplex operation traffic will be balanced across the ports in the lag with the greatest total speed, in most cases there will only be one lag which contains all ports the event of changes in physical connectivity, link aggregation will quickly converge to a new configuration
							failover
								sends and receives traffic only through the master port the master port becomes unavailable, the next active port is used
							
							loadbalance
								balances outgoing traffic across the active ports based on hashed protocol header information and accepts incoming traffic from any active port
								is a static setup and does not negotiate aggregation with the peer or exchange frames to monitor the link hash includes the Ethernet source and destination address, and, if available, the vlan tag, and the ip source and destination address
							
							round robin
								distributes outgoing traffic using a round-robin scheduler through all active ports and accepts incoming traffic from any active port

		firewall
			aliases
				all
					port
					ip
					url
					....

			nat
				portforward (incoming - publish to web an give access to use from web)
					disable (is disable)

					has more priority than waf if need waf must disable this sections
					if need our services reachable on real time also need realtime replacement for waf and portforward must tune waf at start then disable portforward
					
					no rdp (not)
						Disable redirection for traffic matching this rule
						option is rarely needed. don't use this without thorough knowledge of the implications

					interface
					
					address family
					
					protocol
					
					source
						source
							invert matvh
								type
									lan net
									wan net
									wan address
									lan address
									l2tp client 
									pppoe client
									any
									network
									single host or alias

								address mask

						source port range
							from port
								custom
							to port
								custom

					destination
						invert match
							type
								lan net
								wan net
								wan address
								lan address
								l2tp client 
								pppoe client
								any
								network
								single host or alias

							netmask

						destination port range
							from port
								custom
							to port
								custom

					redirection target ip
						type
							single host
							wan address
							lan address

					redirection target port
						port
						custom

					no xmlrpc sync
						do not automatically sync to other carp members
						prevents the rule on Master from automatically syncing to other carp members does not prevent the rule from being overwritten on slave

					nat reflection
						use system default
						enable (nat + proxy)
						enable (pure nat)
						disable

					filter rule association
						none
						add association filter rule
						add unassociation filter rule
						pass

				1:1
					one public ip to a specific destination

					no binat (not)
						do not perform binat for the specified address
						excludes the address from a later, more general, rule

					interface

					address family

					external subnet ip
						*single host
						wan address
						lan address

					internal ip 
						type
							lan net
							wan net
							wan address
							lan address
							l2tp client 
							pppoe client
							any
							network
							*single host

					destination
							type
								lan net
								wan net
								wan address
								lan address
								l2tp client 
								pppoe client
								any
								network
								*single host

					nat reflection
						use system default
						enable
						disable

				outbound (outgoing - give access to lan for using wan)
					modes
						automatic outbound nat rule generation
							ipsec passthrough included
						
						hybrid outbound nat rule generation
							automatic outbound nat + rules below
						
						manual outbound nat rule generation (use in apk)
							aon (advanced outbound nat)
						
						disable outbound nat rule generation
							no outbound nat rules

					we have automatiic rules 
						src > 127.0.0.0/28 :: 1/128, management ip range
						des > * 
						src port > *
						dst port > 500
						nat address > wan address
						nat port > *
						static port > yes

						src > 127.0.0.0/28 :: 1/128 , management ip range 
						des > * 
						src port > *
						dst port > *
						nat address > wan address
						nat port > *
						static port > cross

					ad rules :
						do not nat
							enabling this option will disable nat for traffic matching this 

							rule and stop processing outbound nat rules in most cases this option is not required

						interface

						address family

						protocol

						source
							self firewall
							network
							any

							port or range

						destination

						not
							invert the sense of the destination match

						translation
							address

							port oor range

						misc
							no xmlrpc
								prevents the rule on Master from automatically syncing to other carp members does not prevent the rule from being overwritten on slave

				npt
					interface 
						usauly use wan

					internal ipv6 prefix
						not
							Use this option to invert the sense of the match

					address
						ula ipv6 Prefix for the network prefix translation prefix size specified for the internal ipv6 prefix will be applied to the external prefix

					destination ipv6 prefix
						not
							Use this option to invert the sense of the match
					
					address
						global unicast routable ipv6 prefix

			rule
				floating
					self device firewall
					action
						pass
						block
						reject
						match

					disable

					quick
						apply the action immediately on match
							set this option to apply this action to traffic that matches this rule immediately

					interface

					direction
						any
						in
						out

					address family

					protocol

					source
						type
							lan net
							wan net
							wan address
							lan address
							l2tp client 
							pppoe client
							any
							network
							single host or alias

						source port range
							from port
								custom
							to port
								custom

					destination
						invert match
							type
								lan net
								wan net
								wan address
								lan address
								l2tp client 
								pppoe client
								any
								network
								single host or alias

							netmask

						destination port range
							from port
								custom
							to port
								custom

					extra option
						log
							Log packets that are handled by this rule
							the firewall has limited local log space don't turn on logging for everything

						source os

						diffserv code point
							differentiated services code point is a way for applications to indicate inside the packets how they would prefer routers to treat their traffic as it gets forwarded along its path. The most common use of this is for quality of service or traffic shaping purposes. The lengthy name is often shortened or abbreviated as dscp and sometimes referred to as the tos field

						allow ip options
							allow packets with ip options to pass otherwise they are blocked by default is usually only seen with multicast traffic

						disable rely to
							disable auto generated reply-to for this rule.

						tag
							packet match this rule with this mark of tagging
							policy filter

						tagged
							invert
							match a mark placed on a packet by a different rule with the Tag option check Invert to match packets which do not contain this tag

						max state
						max src nodes
						max connection
							tcp only
						max src state
						max src conn rate
							tcp only
						max src conn rate
							tcp only

						state time out

						tcp flags

						no pfsync
							prevent states created by this rule to be sync'ed over pfsync

						state type
							keep
								works with all ip protocols
							sloppy
								works with all ip protocols
							synproxy
								proxies incoming tcp connections to help protect servers from spoofed tcp syn floods, at the cost of performance (no sack or window scaling)

						no xmlrpc sync
							carp auto enable

						vlan priority
							802.1p priority match on

							non
							background
							best effort
							excellent effort
							video 
							voice
							internetwork control
							critical application
							network control

						vlan priority set
							non
							background
							best effort
							excellent effort
							video 
							voice
							internetwork control
							critical application
							network control

						schedule
							none to leave the rule enabled all the time

						gateway

						in\ out pipe
							choose the out queue/Virtual interface only if In is also selected. 
							the out selection is applied to traffic leaving the interface where the rule is created, the in selection is applied to traffic coming into the chosen interface
							if creating a floating rule, if the direction is in then the same rules apply, if the direction is out the selections are reversed, out is for incoming and in is for outgoing.

						ackquery \ queue


				each interface rule
					interface firewall 

						same state like above

			schedules

			traffic shapper
				(qos) :
				alternative queueing (altg) (use on bsd 5 and higher)(use wizards)
				pipe and dummynet (limitations)
		
				we must set priority beetwen 0-7 (7 is highest priority)
		
				types :
					priority cased queues (pcq)
					class base queueing (cbq)
						each queue classified on whole bandwidth
						parallel queueing
						don't assign free lines to process
						has starving
		
					hierarichical fair services curve (hfsc)
						most usefull
						fair
						if line was free use for some process or assign to anothers
						no starving
		
				dummynet :
					packet lost simulation
					use ipfw on freebsd
					limit bandwidth
					use queueing and priority (no native features on freebsd)
		
				pass proto tcp to port ssh set prio 7
				pass proto {tcp,udp} to port {www, https} set prio 7
		
					can use cache if our queue get full
		
				queue name_x on interface bandwidth number [,k,m,g]
					queue name_x1 parent name_x bandwidth number [,k,m,g] default
					queue name_x2 parent name_x bandwidth number [,k,m,g] 
		
				queue main on $ext_if bandwidth 20m
					queue ftp parent main bandwidth 8m
					queue ssh parent main bandwidth 12m
						queue ssh_intractive main bandwidth 10m
						queue ssh_bulk main	bandwidth 2m
		
				set skip on {lo, $init_if}
					pass log quick on $ext_if proto tcp to port ssh
						queue (ssh_bulk, ssh_intractive) set prio (5,7)
					pass in quick on $ext_if proto tcp to port ftp queue ftp
					pass out on $ext_if proto icmp queue icmp
					pass out on $ext_if proto tcp from $localnet to port $clientout
		
				modes for shapping :
					multiple lan\wan
						mix data and protocols on each side (recommend)
		
					dedicated links		
						seperate each side
		
				firewall > traffic shaper
					wizards and multiple lan\wan is best practice
					prioritisize voip traffic set priority for qos and voip
					can set values for point to point networks and games ...
		
					has many steps on wizard (7 steps)

					we have penalty box here canchange priority of ip address
					also change p2p nodes priority here
		
				firewall > rules 
					at the end of page we have limits and queue
		
				after 3 minuts al policies can reload
		
				pftop -s 1 -v queue
		
				on captive portal make a seperate pipe and queue

			virtual ips
				type
					ip alias
					carp
					proxy arp
					other

				interface

				address type
					single address

				addresses

				virtual ip password

				vhid group
					carp vip
					Verified Handles Identifier

				advertising frequency
					0 means usually master otherwise the lowest combination of both values in the cluster determines the master

		services
			auto configuration backup
				restore
				setting
					enable acb (automatic configuration backup)
					
					backup frequency
						automatic backup on every configuration change
						automatic backup on a regular schedule

					encryption password

					hint \ identifier
						may optionally provide an identifier which will be stored in plain text along with each encrypted backup may allow the netgate support team to locate your key should you lose it

					manual backups to keep
						 may be useful to specify how many manual backups are retained on the server so that automatic backups do not overwrite them maximum of 50 retained manual backups (of the 100 total backups) is permitted

				backup now
					revision reason
					device key

			captive portal
				config
					enable
					interface
					maximum concurent connection
						on single ip address how many connections be on smae time

					idle timeout (blank > no idle time out)
					hard timeout (blank > no idle time out)

					traffic quota
						will be disconnected after exceeding this amount of traffic, inclusive of both downloads and uploads may log in again immediately, though leave this field blank for no traffic quota

					passthrough credits per mac address
						allows passing through the captive portal without authentication a limited number of times per MAC address once used up, the client can only log in with valid credentials until the waiting period specified below has expired recommended to set a hard timeout and/or idle timeout when using this for it to be effective

					waiting period to restore pass through credits
						clients will have their available pass-through credits restored to the original count after this amount of time since using the first one must be above 0 hours if pass-through credits are enabled.

					reset waiting period
						Enable waiting period reset on attempted access
						the waiting period is reset to the original duration if access is attempted when all pass-through credits have already been exhausted

					log out popup windows
						enabled, a popup window will appear when clients are allowed through the captive portal allows clients to explicitly disconnect themselves before the idle or hard timeout 
						
					pre authentication redirect url
						visitors will be redirected to this url after authentication only if the captive portal doesn't know where to redirect them. This field will be accessible through $PORTAL_REDIRURL$ variable in captiveportal's html pages

					after authentication redirection url
						 clients will be redirected to this URL instead of the one they initially tried to access after they've authenticated

					blocked mac address redirec url
						addresses will be redirected to this URL when attempting access

					preserve users database
						preserve connected users across reboot
							 connected users won't be disconnected during a pfsense reboot

					concurent users logins
						disabled: do not allow concurrent logins per username or voucher
						multiple: no restrictions to the number of logins per username or voucher will be applied
						last login: only the most recent login per username or voucher will be granted previous logins will be disconnected
						first login: only the first login per username or voucher will be granted further login attempts using the username or voucher will not be possible while an initial user is already active

					mac filtering
						if enabled no attempts will be made to ensure that the MAC address of clients stays the same while they are logged in is required when the mac address of the client cannot be determined (usually because there are routers between pfSense and the clients if this is enabled, radius mac authentication cannot be used

					pass throough mac auto entry
						 mac passthrough entry is automatically added after the user has successfully authenticated
						 users never authenticate again
						 remove the passthrough mac entry either log in and remove it manually from the mac tab or send a post from another system
						 logout window will not be shown

					per user bandwidth restriction

					use custom captive portal page
						enable to use a custom captive portal login page

				captive portal login page (editing)

				authentication
					methods
						authentication backend will force the login page to be displayed and will authenticate users using their login and password, or using vouchers
						none method will force the login page to be displayed but will accept any visitor that clicks the submit button
						radius mac authentication method will try to authenticate devices automatically with their mac address without displaying any login page

				https options
					login
						when enabled, the username and password will be transmitted over an https connection to protect against eavesdroppers server name and certificate must also be specified below


				macs
				allowed ip address
				allowed hostnames
				vouchers
					enable
					create generate and activate rolls with voucher
						public key
						
						private key
						
						character set

						# of roll bits
							reserves a range in each voucher to store the roll # it belongs to. allowed range: 1..31. sum of roll+ticket+checksum bits must be one Bit less than the rsa key size

						# of ticket bits
							reserves a range in each voucher to store the ticket# it belongs to allowed range: 1..16. Using 16 bits allows a roll to have up to 65535 vouchers bit array, stored in ram and in the config, is used to mark if a voucher has been use bit array for 65535 vouchers requires 8 kb of storage

						# of checksum 
							reserves a range in each voucher to store a simple checksum over roll# and ticket# allowed range is 0..31

						magic number
							stored in every voucher verified during voucher check size depends on how many bits are left by roll+ticket+checksum bits all bits are used, no magic number will be used and checked

						invalid voucher message
							error message displayed for invalid vouchers on captive portal error page $PORTAL_MESSAGE$

						expired voucher message
							error message displayed for expired vouchers on captive portal error page $PORTAL_MESSAGE$

				high availablity
					enable
						The xmlrpc sync provided by pfSense in high availability settings only synchronize a secondary node to its primary node checkbox enable a backward sync from the secondary to the primary, in order to have a bi-directional synchronization

						*these settings should be set on the secondary node only*
						*do not update these settings if this pfsense is the primary node*

					primary node ip
					primary node username
						backward sync
						could be any pfsense user on the primary node with system - ha node sync privileges
					primary node password

				file manager

			dhcp relay
				enable
				
				interfaces

				carp status vip
					to determine the ha master/backup status dhcp relay will be stopped when the chosen vip is in backup status, and started in master status
				
				destination server
					dhcp servver ip address on remote side

				append circuit id and agent id to requests
					this is checked, the dhcp relay will append the circuit id (pfsense interface number) and the agent id to the dhcp request

						dhcp rely :
							interface wan
							destination server (front side ip adress)
							if dhcp server was enable we can use this item

			dhcp server
				enable dhcp
				bootp
				deny unkown clients
					allow all clients
					allow known clients from any interface
					allow known clients from only this interface

				ignore denied clients
					denied clients will be ignored rather than rejected

					is not compatible with failover
					can not enable when a failover peer ip address is enable

				ignore client identifiers
					client includes a unique identifier in its dhcp request, that uid will not be recorded in its lease

					useful when a client can dual boot using different client identifiers but the same hardware mac address. Note that the resulting server behavior violates the official dhcp specification

				range

				additional pool

				servers
					wins server
					dns servers

				omapi
					object management api
					provides access to the objects (leases, hosts or groups) stored in its database

					omapi port
					omapi key
					key algorithm
						hmac sha256 current bind 9 default
						hmac md5 legacy default
						hmac sha1
						hmac sha224
						hmac sha 384
						hmac sha 512 (most secure)

				other options
					gateway
						default is to use the ip on this interface of the firewall as the gateway specify an alternate gateway here if this is not the correct gateway for the network type none for no gateway assignment

					domain name
					 	domain name of this system as the default domain name provided by dhcp alternate domain name may be specified here

					 domain search list

					 default leased time
					 	7200 seconds

					 maximum lease time
					 	86400 seconds

					 failover peer ip
					 	blank is disable
					 	machines must use carp

					 static arp
					 	persists even if dhcp server is disabled only the machines listed below will be able to communicate with the firewall on this interface

					 time format change
					 	Change dhcp display lease time from utc to local time
					 	 lease time will be displayed in local time and set to the time zone selected will be used for all dhcp interfaces lease time

					 statictics graphs
					 	enable this to add dhcp leases statistics to the rrd (round robin database) graphs disabled by default

					 ping check
					 	when enabled dhcpd sends a ping to the address being assigned, and if no response has been heard, it assigns the address. enabled by default



				mac address control
					mac allow
					mac deny

				ntp
					define servers

				tftp

				ldap
					efine ldap server url

				network booting
					enable
					next server
					default bios file name
					uefi 32 bit file name
					uefi 64 bit file name
					arm 32/64 bit file name
					uefi http boot url
					root path

				additional bootp dhcp options
					dhcp options

			dns forwarder
				enable

					by default is disable and works on unbound like dns resolver if want use dns forwarder must disable unbound

					our rules must work for lan responds not wan

				if (enable) disable dns forwarder our dns packets goes through the internet and maybe have some attacks 
				must (disable) disable dns forwarder our dns resolves like proxy				

				dhcp registration
					register dhcp leases in dns forwarder
					domain in system > general setup should also be set to the proper value

				static dhcp
					register dhcp static mappings in dns forwarder

				prefer dhcp
					resolve dhcp mapping first
						mappings will be resolved before the manual list of names below
						only affects the name given for a reverse lookup ptr

				dns query forwarding
					query dns servers sequentially
						will query the dns servers sequentially in the order specified (system > general setup - dns Servers), rather than all at once in parallel

					require domain
						will not forward a or aaaa queries for plain names, without dots or domain parts, to upstream name servers
						not known from /etc/hosts or dhcp then a not found answer is returned

					do not forward private reverse lookups
						will not forward reverse dns lookups ptr from private network to upstream
						 any entries in the domain overrides section forwarding private n.n.n.in-addr.arpa names to a specific server are still forwarded
						 not known from /etc/hosts or dhcp then a not found answer is returned

				listen port

				interface
					no wan selection on apk gate

				strict binding
					dns forwarder will only bind to the interfaces containing the ip addresses selected above
					rather than binding to all interfaces and discarding queries to other addresses
					don't work on ipv6 if use dnsmasq

				custom options

				host override options
					host
					domain
					ip address
					additional name for this host

				domain override options
					domain
					ip address
					source ip

			dns resolver
				general setting
					enable
					listen port
					enable ssl\tls services
						respond to incoming ssl/tls queries from local clients
						 answer queries from clients which also support dns over tls. activating this option disables automatic interface response routing behavior

					ssl\tls certificate

					ssl\tls listen port
						853

					network interfaces
						interface ips used by the dns Resolver for responding to queries from clients

					outgoing network interfaces
						default all interfaces are used

					strict outgoing network interface binding
						do not send recursive queries if none of the selected outgoing network Interfaces are available
						dns send requests over any available interfaces
						none of the selected outgoing network Interfaces are available option makes the dns Resolver refuse recursive queries

					system domian local zone type
						deny
						refuse
						transparent
						static
						type transparent
						redirect
						inform
						inform deny
						no default

						this option used for system > general setup > domain

					dnssec
						is enable by default

					python module

					dns query forwarding
					
					dhcp registration
					
					static dhcp
					
					openvpn clients
						register connected openvpn clients in the dns resolver
						common name (cn) of connected openvpn clients will be registered in the dns resolver
						their name can resolve
						only works on openvpn servver
						remote access ssl/tls or User auth with username as common name option
						must setup domain on system > general setup > domain
					
					custom options

					host overrides
						any individual hosts for which the resolver's standard dns lookup process should be overridden and a specific ipv4 or ipv6 address should automatically be returned by the resolver
					
					domain overrides

				advance setting
					hide identity
						id.server and hostname.bind queries are refused

					hide versions
						version.server and version.bind queries are refused

					query name minimization
						send minimum amount of qname/qtype information to upstream servers to enhance privacy

						send minimum required labels of the qname and set qtype to a when possible effort approach; full qname and original qtype will be sent when upstream replies with a rcode other than noerror, except when receiving nxdomain from a dnssec signed zone

						default is off

					strict query name minimization
						do not fall-back to sending full qname to potentially broken dns servers

						qname minimization in strict mode a significant number of domains will fail to resolve when this option in enabled use if you know what you are doin goption only has effect when query Name minimization is enabled

						default is off

					advance resolver
						prefetch support
							message cache elements are prefetched before they expire to help keep the cache up to date

								 can cause an increase of around 10% more dns traffic and load on the server, but frequently requested items will not expire from the cache (is diable)

						prefetch dns key support
							dnskeys are fetched earlier in the validation process when a delegation signer is encountered
								
								helps lower the latency of requests but does utilize a little more cpu (is disable)

						harden dnssec data
							dnssec data is required for trust-anchored zones
								such data is absent, the zone becomes bogus disabled and no dnssec data is received, then the zone is made insecure

						server expire
							serve cache records even with ttl of 0
								allows unbound to serve one query even with a ttl of 0, if ttl is 0 then new record will be requested in the background when the cache is served to ensure cache is updated without latency on service of the dns request

						aggressive nsec
							aggressive use of dnssec-validated cache
								 unbound uses the dnssec nsec chain to synthesize nxdomain and other denials, using information from previous nxdomains answers helps to reduce the query rate towards targets that get a very high nonexistent name lookup rate

						message cache size
							4mb
								size of the message cache. The message cache stores dns response codes and validation statuses resource record set (rrset) cache will automatically be set to twice this amount rrset cache contains the actual rr data  default is 4

						outgoing tcp buffer
							10
								number of outgoing tcp buffers to allocate per thread
								0 > not accepted

						incoming tcp buffers
							10 like above

						edns buffer size
							automatic value based on active interface mtus
							512 minimum ipv4
							1220 nsd ipv6 edns minimum
							1232 ipv6 minimum
							1432 1500 byte mtu
							1480 nsd ipv4 edns minimum
							4096 unbound defualt

								number of bytes size to advertise as the edns reassembly buffer size

								use udp datagrams send to peers

								auto mode sets optimal buffer size by using the smallest mtu of active interfaces and subtracting the ip header size

								fragmentation reassemble problems occur, usually seen as timeouts, then a value of 1432 should help

								512/1232 values bypasses most ip mtu path problems, but it can generate an excessive amount of tcp fallback

						number of queries per thread
							512
							1024
							2048

							queries that every thread will service simultaneously
							more queries arrive that need to be serviced, and no queries can be jostled, then these queries are dropped

						jostle timeout
							100
							200
							500
							1000

								when the server is very busy
								protects against denial of service by slow queries or high query rates
								default value is 200 milliseconds

						maximum ttl  for rrsets and messages
							86400 (or 1 day)
							in cache timing
							the internal ttl expires the cache item is expired can be configured to force the resolver to query for data more often and not trust (very large) ttl values

						minimum ttl for rrsers and message
							0
							if the minimum value kicks in, the data is cached for longer than the domain owner intended, and thus less queries are made to look up the data
							0 value ensures the data in the cache is as the domain owner intended high values can lead to trouble as the data in the cache might not match up with the actual data anymore

						ttl for host cache entires
							1,2,5,10,15 minuts

							entries in the infrastructure host cache

							infrastructure host cache contains round trip timing, lameness, and edns support information for dns servers default value is 15 minutes

						number of hosts to cache
							1k
							5k
							10k (default)
							20k
							50k
							1m
							2m

						unwanted reply threshold
							disable is default
							
							5,10,20,40,50 m

							 total number of unwanted replies is kept track of in every thread. When it reaches the threshold, a defensive action is taken and a warning is printed to the log file defensive action is to clear the rrset and message caches, hopefully flushing away any poison  default is disabled, but if enabled a value of 10 million is suggested

						log levels

						disable auto added access control
							disable the automatically-added access control entries

							networks residing on internal interfaces of this system are permitted allowed networks must be manually configured on the access Lists tab if the auto-added entries are disabled

						disable auto added host entires
							disable the automatically-added host entries

							addresses of this firewall are added as records for the system domain of this firewall as configured in system > gneral setup
							disables the auto generation of these entries

						experimental bit 0x20 support
							use 0x-20 encoded random bits in the dns query to foil spoofing attempts

						dns64 supprt
							enable dns64 (rfc 6147)

							used with an ipv6/ipv4 translator to enable client-server communication between an ipv6-only client and an ipv4-only servers

				access list
					name
					
					action
						deny: stops queries from hosts within the netblock defined below
						
						refuse: stops queries from hosts within the netblock defined below, but sends a dns rcode refused error message back to the client
						
						allow: allow queries from hosts within the netblock defined below
						
						allow snoop: allow recursive and nonrecursive access from hosts within the netblock defined below used for cache snooping and ideally should only be configured for the administrative host
						
						deny nonlocal: allow only authoritative local-data queries from hosts within the netblock defined below messages that are disallowed are dropped
						
						refuse nonlocal: allow only authoritative local-data queries from hosts within the netblock defined below sends a dns rcode refused error message back to the client for messages that are disallowed
					
					networks

			dynamic dns
				ddns client
					disable

					services type

					interface monitor
						(must be on wan)
						if the interface ip address is private the public ip address will be fetched and used instead

					hostname
						enter the complete fully qualified domain name

					mx
						dyndns service only a hostname can be used, not an ip address set this option only if a special mx record is needed not all services support this

					wildcards
					verbose logging

					username
						 is required for all types except dns made Easy

					password

				rfc2136 clients
					enable
					interface
						address of this interface will be used in the updated dns record

					hostname

					zone

					ttl (seconds)

					keyname
						 must match the setting on the dns server

					key algorithm
						hmac sha256 current bind 9 default
						hmac md5 legacy default
						hmac sha1
						hmac sha224
						hmac sha 384
						hmac sha 512 (most secure)

					key
						secret tsig (transaction signature) domain key
							is a computer networking protocol used by the domain name system (dns) as a way to authenticate updates to a dynamic dns database
					server

					protocol
						Use tcp instead of udp


					use public ip
						if the interface ip is private, attempt to fetch and use the public ip instead

					update source
						wan
						lan
						localhost
						default (use interface above)
						do not specify

						interface or address from which the firewall will send the dns update request

					update source family
						default
						ipv4
						ipv6

					record type
						a
						aaaa
						both

				check ip services


					enable registration of dhcp client names in dns

					ddns domain
	
					ddns hostnames
						force dynamic dns hostname to be the same as configured hostname for static mappings
					default registers host name option supplied by shcp client

					primary ddns address

					secondary ddns address

					dns domain key

					key algorithm
						hmac sha256 current bind 9 default
						hmac md5 legacy default
						hmac sha1
						hmac sha224
						hmac sha 384
						hmac sha 512 (most secure)

					dns domain key secret

					ddns client update
						allow
						deny
						ignor

						forward entries are handled when client indicates they wish to update dns allow prevents dhcp from updating Forward entries, Deny indicates that dhcp will do the updates and the client should not, ignore specifies that dhcp will do the update and the client can also attempt the update usually using a different domain name

						update our ip and host name on dns server when need connect to pfsense on wan

						must use on wan valid ip address
						dynamic ip
				
						must set our ip and dns on noip_free site
						each 30 days must recharge
				
						noip > dynmaic dns > host name (make a record)
				
						services > ddns
							service type > noip (free)
							interface to monitor > wan
							hostname > noip hostname made on noip site
							username and password (user and pass on noip site)
				
			igmp proxy
				interface
				type
					upstream
						upstream network interface is the outgoing interface which is responsible for communicating to available multicast data sources can only be one upstream interface

					downstream
						downstream network interfaces are the distribution interfaces to the destination networks, where multicast clients can join groups and receive multicast data one or more downstream interfaces must be configured

				threshold
					default is 1

					packets with a lower ttl than the threshold value will be ignored

				networs
				addnetwork

			ntp
				setting
					enable
					interface
						interfaces without an ip address will not be shown
						no selection means listen on all interfaces

					time servers
					add
						best results you should configure between 3 and 5 servers

					max candidate pool peers
						value should be set low enough to provide sufficient alternate sources while not contacting an excessively large number of peers

						many servers inside public pools are provided by volunteers, and a large candidate pool places unnecessary extra load on the volunteer time servers for little to no added benefit
						default: 5

					orphan mode
						allows the system clock to be used when no other clocks are available

						number here specifies the stratum reported during orphan mode and should normally be set to a number high enough to insure that any other servers available to clients are preferred over this server 

						default: 12

					minimum poll interval
					maximu poll interval

					ntp graphs
						enable rrd graphs of ntp statistics 

						default: disabled

					logging
						log peer messages 
							default: disabled

						log system messages 
							default: disabled

							status > system logs > ntp
					
					statictics logging
						these options will create persistent daily log files in /var/log/ntp

						log reference clock statistics
						log clock discipline statistics
						log ntp peer statistics 

						default: disabled

					leap seconds
						primary objectives of the iers are to serve the astronomical, geodetic and geophysical communities by providing data and standards related to Earth rotation and reference frames

					dns resolution
						default 
						ip4
						ip6

					enable ntp server authentication
						authentication allows the ntp client to confirm it is communicating with the intended server, which protects against man-in-the-middle attacks

				acls
					default access restriction
						kiss o death
						modifications
						peer association
						trap services
						
						are enable on default

						queries
						service

						are disable on default

					network

				serial gps
					gps connected via a serial port may be used as a reference clock for ntp the gps also supports PPS and is properly configured, and connected, that gps may also be used as a pulse per second clock reference

					usb gps not recommended

					gps type
						default
						custom
						generic
						garmin
						mediatek
						sirf
						u blox
						sure gps
				
				pps

			pppoe server
				enable

				interface

				total user count
					1 - 255
					the number of pppoe users allowed to connect to this server simultaneously

				user max logins
					1 - 255
					the number of times a single user may be logged in at the same time

				server address
					ip address the pppoe server should give to clients for use as their gateway typically this is set to an unused ip just outside of the client range

					this should not be set to any ip address currently in use on this firewall

				remote address range
					specify the starting address for the client ip address subnet

				subnet mask

				authentication type
					chap
					pap

				dns server
					if entered these servers will be given to all PPPoE clients, otherwise lan dns and one wan dns will go to all clients

				radius
					radius authentication
						local user database will not be used

					radius accounting

					use a backup radius authentication server
						if primary server fails all requests will be sent via backup server

				nas ip address

				radius accounting update
					 accounting update period in seconds

				radius issued ip address
					assign ip addresses to users via radius server reply attributes

				primary radius server

				primary radius server shared secret
					Enter the shared secret that will be used to authenticate to the radius server

				secondary radius server
					standard ports are 1812 (authentication) and 1813 (accounting)

				secondary radius server shared secret

				user table

			snmp daemon
				enable

				polling port
					161

				system location

				system contact

				read community string
					community string is like a password
					restricting access to querying snmp to hosts knowing the community string

				snmp traps enable

				snmp modules
					mibll
						defines the data types and access permissions for the various managed objects that can be accessed through the bea snmp agent software it also defines the event notifications that can be generated by the bea snmp agent software

					netgraph
					pf
						implements an object oriented interface to access snmp enabled network switches

					host resources
					ucd
					regex

				interface protocol
					ip4
					ip6
					ip4+6

				bind interface

					snmp :
						services > snmp
							enable
							port 161
							read community string
							bind interface (works on wich interface)

			upnp and nat pmp (port mapping protocol)
				enable

				upnp port mapping
					allow 

					this protocol is often used by microsoft-compatible systems

				nat pmp port mapping
					allow

					this protocol is often used by apple-compatible systems

				external interface
					select only the primary wan interface
					interface with the default gateway
					only one interface may be choosen

				interfaces

				download speed
					in kb

				upload speed
					in kb

				override wan address
					use an alternate wan address to accept inbound connections such as an ip alias or carp virtual ip address

				traffic shaping
					altq traffic shaping queue in which the connections should be placed

				log packets

				uptime
					use system uptime instead of upnp & nat-pmp service uptime

				default deny
					deny access to upnp & nat-pmp by default

				enable stun
					stun stands for :
						session traversal utilities for nat (network address translator; network protocol)

						simple traversal of udp through nat

						simple traversal of udp through nats (network address translators; network protocol)

					external interface must have a public ip address
					otherwise it is behind nat and port forwarding is impossible

					in some cases the external interface can be behind unrestricted nat 1:1 when all incoming traffic is forwarded and routed to the external interface without any filtering

					in these cases upnp service needs to know the public ip address and it can be learned by asking an external server via stun protocol

					the following option enables retrieving the external public ip address from a stun server and detection of the nat type


				stun server
					enable retvieving external ip address from stun server

					ip or hostname

				stun port
					3478 udp

				acl entires
					example:  [allow or deny] [ext port or range] [int ipaddr or ipaddr/cidr] [int port or range]

					allow 1024-65535 192.168.0.0/24 1024-65535

				custom presentation url
					if left blank the default value of the webgui of this firewall will be used

				custom model number
					if left blank the default value of the firmware version of pfSense will be used

			wake on lan

		vpn
			ipsec (use for most securing network an ptp connection to hq)
				tunnels
					disable
						set this option to disable this phase1 without removing it from the list

					ike endpoint configuration
						key exchange version
							ikev2
							ikev1
							auto

								internet key exchange protocol version to be used auto uses ikev2 when initiator and accepts either ikev1 or ikev2 as responder								
						internet protocol
							ip4
							ip6
							both (dual stack)

						interface
							interface for the local endpoint of this phase1 entry

						remote gateway
							public ip address or host name of the remote gateway

					phse 1 proposal (authentication)
						authentication method
							mutual psk
							mutaul cerrtificate

						my identifier
							my ip address
							ip address
							fully qualified domain name
							fully qualified domain name \ email
							asn.1 distinguished name
							key id tag
							dynamic dns
							automatic based on content

						peer identifier
							my ip address
							ip address
							fully qualified domain name
							fully qualified domain name \ email
							asn.1 distinguished name
							key id tag
							dynamic dns
							automatic based on content

						preshared key

					phase1 proposal (encryption algorithm)
						encryption algorithm
							algorithm
								aes
								aes 128 gcm 
								aes 129 gcm
								aes 256 gcm
								blowfish
								3des
								cast 128

							key length
								bits :
									128
									192
									256

							hash
								md5
								sha 1
								sha 256
								sha 384
								sha 512
								aes xcbc

							dh group
								1 - 32 

					expiration and replacement
						life time
							28800

							hard ike sa life time in seconds after which the ike sa will be expired must be larger than rekey time and reauth time
							cannot be set to the same value as rekey time or reauth time
							if left empty, defaults to 110% of whichever timer is higher (reauth or rekey)

						rekey time
							28800

							time in seconds before an ike sa establishes new keys
							this works without interruption
							cannot be set to the same value as life time only supported by ikev2 and is recommended for use with ikev2
							leave blank to use a default value of 90% life time when using ikev2 Enter a value of 0 to disable

						reauth time
							0

							time in seconds before an ike sa is torn down and recreated from scratch including authentication
							this can be disruptive unless both sides support make-before-break and overlapping ike sa entries
							cannot be set to the same value as life time. Supported by ikev1 and ikev2 leave blank to use a default value of 90% life time when using ikev1 enter a value of 0 to disable

						rand time
							28800

							A random value up to this amount will be subtracted from rekey time/reauth time to avoid simultaneous renegotiation
							if left empty defaults to 10% of life time
							enter 0 to disable randomness but be aware that simultaneous renegotiation can lead to duplicate security associations

					advance option
						child sa start action
							default
							none (responder only)
							initiate at start (vti or tunnel mode)
							initial on demand (tunnel mode only)

						child sa cloase action
							default
							close connections and clear sa
							restart \ reconnect
							close connections and reconnect demand

						nat traversal
							auto 
							force

							set this option to enable the use of nat-t (the encapsulation of esp in udp packets) if needed which can help with clients that are behind restrictive firewalls

						mobike
							enable \ disable

							ikev2 mobility and multi-homing protocol (mobike) allows the ip addresses associated with ikev2 and tunnel mode ipsec security associations (sa) to change mobile vpn client could use mobike to keep the connection with the VPN gateway active while moving from one address to another

						gateway duplicates
							(default disable)
							enable this to allow multiple phase 1 configurations with the same endpoint when enabled pfsense does not manage routing to the remote gateway and traffic will follow the default route without regard for the chosen interface static routes can override this behavior

						splite connections
							(default disable)
							enable this to split connection entries with multiple phase 2 configurations required for remote endpoints that support only a single traffic selector per child sa

						prf selection
							(default disable)
							enable manual pseudo-random Function (prf) selection

							manual prf selection is typically not required but can be useful in combination with aead encryption algorithms such as aes-gcm

						custom ike\nat-t ports
							udp port for ike on the remote gateway leave empty for default automatic behavior (500/4500)

							udp port for nat-t on the remote gateway

						dead peer detection
							(default disable)
							check the liveness of a peer by using ikev2 informational exchanges or ikev1 r_u_there messages
							active dpd checking is only enforced if no ike or esp/ah packet has been received for the configured dpd delay
						
						delay
							10
							delay between sending peer acknowledgement messages
							in ikev2 a value of 0 sends no additional messages and only standard messages (such as those to rekey) are used to detect dead peers
						
						max failures
							5

							number of consecutive failures allowed before disconnecting only applies to ikev1 in ikev2 the retransmission timeout is used instead

				mobile clients
					ike extentions
						enable ipsec mobile client support

					user authentication
						local database

					group authentication
						authenticate members of groups which have either user - vpn ipsec with dialin or webcg - all pages privileges

					radius accounting
						when enabled the ipsec daemon will attempt to send radius accounting data for all tunnels not only connections associated with mobile ipsec
						do not enable this option unless the selected radius servers are online and capable of receiving radius accounting data
						if radius accounting data is enabled and fails to send tunnels will be disconnected

					virtual address pool
						provide a virtual ip address to clients

					virtual ip6 address pool

					radius ip address piority
						ip4\ip6 address pool is used if address is not supplied by radius server

					radius advance parameters
						may only be required when using 2fa/mfa with radius or under high load

					network list
						provide a list of accessible networks to clients

					save xauth password
						allow clients to save xauth passwords (cisco vpn client only)

						with iphone clients this does not work when deployed via the iphone configuration utility only by manual entry

					dns default domain
						provice default domain name to clients

					split dns
						provide a list of split dns domain names to clients Enter a space separated list

					dns server
						dns server list to clients
							ip4-mapped ip6 addresses (ex: fd00::1.2.3.4) are not supported

					wins servers

					phse2 pfs group
						of or 1 - 32

						1,2,5,22,23,24 weak security

					login banner

				preshared keys
					identifier
					secret type
					preshared key

				advanced settings
					deamon
					sa manager
					ike sa
					ike child sa
					job processing
					configuration backend
					kernel interface
					networking
					asn encoding
					message encoding
					integrity checker
					integrity verifier
					platform trust service
					tls handler
					ipsec traffic
					strong swan lib

						logging controls
							control
							silent
							diag
							audit
							raw
							highest

					configure unique ids as
						yes (replace)
						never
						no
						keep

						whether a particular participant id should be kept unique with any new ike_sa using an id deemed to replace all old ones using that id
						participant ids normally are unique so a new ike_sa using the same id is almost invariably intended to replace an old one
						the difference between no and never is that the old ike_sa will be replaced when receiving an initial_contact notify if the option is no but will ignore these notifies if never is configured
						the daemon also accepts the value keep to reject new ike_sa setups and keep the duplicate established earlier
						
						defaults to yes

					ipsec filter mode
						filter ipsec tunnel transport and vti on ipsec tab (enc0)
						filter ipsec vti and transport on assigned interface block all tunnel mode traffics

						experimental controls how the firewall will filter ipsec traffic
						by default rules on the ipsec tab filter all ipsec traffic including tunnel mode transport mode and vti mode

						this is limited in that it does not allow for filtering on assigned vti or transport mode interfaces (gre) and it does not support features such as nat rules and reply-to for return routing

						when set to filter on assigned vti and transport interfaces all tunnel mode traffic is blocked
						do not set this option unless all ipsec tunnels are using vti or transport mode

					ikev2 retransmission parameters
						timeout parameters for ikev2 (default disable)

					ip comression
						compression of content is proposed on the connection (default disable)

					pkcs#11 support
						for Phase 1 authentication (default disable)

						that restarting the ps/sc smart card service will restart ipsec and vice versa

						versa enables the identification of users, flows, packets and applications while establishing, monitoring, and automatically adjusting security and network policies based on threats, vulnerabilities, and changes in the network environment

						versa is the modern secure network

						sase platform

					strict interface binding
						enable strongswan's interfaces_use option to bind specific interfaces only (default disable)

						option is known to break ipsec with dynamic ip interfaces

						not recommended at this time

					unecrypted payloads in ikev1 main mode
						accept unencrypted id and hash payloads in ikev1 main mode (default disable)

						some implementations send the third main mode message unencrypted, probably to find the psks for the specified id for authentication

						is very similar to aggressive mode, and has the same security implications: 
							passive attacker can sniff the negotiated Identity, and start brute forcing the psk using the hash payload

							is recommended to keep this option to no, unless the exact implications are known and compatibility is required for such devices (sonicwall boxes)

					maximum ikev1 phase 2 exchanges
						ikev1 phase 2 rekeying for one vpn gateway can be initiated in parallel

						default only 3 parallel rekeys are allowed

						undersized values can break vpn connections with many phase 2 definitions

						if unsure, set this value to match the largest number of phase 2 entries on any phase 1

					enable cisco extensions
						enable unity plugin (default disable)
					
						enable unity plugin which provides cisco extension support such as split-include, split-exclude and split-dns

					strict crl checking
						enable strict certificate revocation list checking (default disable)

						check this to require availability of a fresh crl for peer authentication based on certificate signatures to succeed

					make before break
						initiate ikev2 reauthentication with a make-before-break (default disable)

						instead of a break-before-make scheme make-before-break uses overlapping ike and child_sa during reauthentication by first recreating all new sas before deleting the old ones

						behavior can be beneficial to avoid connectivity gaps during reauthentication, but requires support for overlapping sas by the peer

					asynchronous cryptography
						use asynchronous mode to parallelize multiple cryptography jobs (default disable)

						allow crypto(9) jobs to be dispatched multi-threaded to increase performance

						jobs are handled in the order they are received so that packets will be reinjected in the correct order

					custom ports
						Local udp port for ike (efault: 500)
						Local udp port for nat-t (default: 4500)

					auto exclude lan address
						enable bypass for lan interface ip (default is enable)

						exclude traffic from lan subnet to lan ip address from ipsec

					additional ipsec bypass
						enable extra ipsec bypass rules (default disable)

			l2tp
				interface
				server address
					the ip address (l2tp server) should give to clients for use as their gateway
					must use another ranges for vpn

					if use apk gate for vpn server must set 127.0.0.1

					must not be on same range with server

					typically this is set to an unused ip just outside of the client range

					should not be set to any ip address currently in use on this firewall

				remote address range (all except server ip)
				number of l2tp users
					1 - 255

				secret (on apk is blank)
				authentication type
					chap
					pap
					mschap v2

				primary l2tp dns server
				secondary l2tp dns server
				vpn mtu
					this field is blank, the adapter's default mtu will be used 
					is typically 1500 bytes but can vary in some circumstances

				radius 
					enable
						 local user database will not be used

					accounting
					server
						if use apk gate 127.0.0.1 must set 
					secret
						admin
						admin
					radius issued ips
						issue ip Addresses via radius server

						on apk is enabled

				users :
					username
					password
					ip address

			l2tp over ipsec
				interface
				server address
					the ip address (l2tp server) should give to clients for use as their gateway
					must use another ranges for vpn

					if use apk gate for vpn server must set 127.0.0.1

					must not be on same range with server

					typically this is set to an unused ip just outside of the client range

					should not be set to any ip address currently in use on this firewall

				remote address range (all except server ip)
				number of l2tp users
					1 - 255

				secret (on apk is blank)
				authentication type
					chap
					pap
					mschap v2

				primary l2tp dns server
				secondary l2tp dns server
				vpn mtu
					this field is blank, the adapter's default mtu will be used 
					is typically 1500 bytes but can vary in some circumstances

				radius 
					enable
						 local user database will not be used

					accounting
					server
						if use apk gate 127.0.0.1 must set 
					secret
						admin
						admin
					radius issued ips
						issue ip Addresses via radius server

						on apk is enabled

				on ipsec tab must goes to :
					vpn
						ipsec
							mobile clients
								ike extentions
									enable ipsec mobile client support
			
								user authentication
									local database
			
								group authentication
									authenticate members of groups which have either user - vpn ipsec with dialin or webcg - all pages privileges

								radius accounting
									when enabled the ipsec daemon will attempt to send radius accounting data for all tunnels not only connections associated with mobile ipsec
									do not enable this option unless the selected radius servers are online and capable of receiving radius accounting data
									if radius accounting data is enabled and fails to send tunnels will be disconnected
			
								virtual address pool
									provide a virtual ip address to clients
			
								virtual ip6 address pool
			
								radius ip address piority
									ip4\ip6 address pool is used if address is not supplied by radius server
			
								radius advance parameters
									may only be required when using 2fa/mfa with radius or under high load
			
								network list
									provide a list of accessible networks to clients
			
								save xauth password
									allow clients to save xauth passwords (cisco vpn client only)
			
									with iphone clients this does not work when deployed via the iphone configuration utility only by manual entry
			
								dns default domain
									provice default domain name to clients
			
								split dns
									provide a list of split dns domain names to clients Enter a space separated list
			
								dns server
									dns server list to clients
										ip4-mapped ip6 addresses (ex: fd00::1.2.3.4) are not supported
			
								wins servers
			
								phse2 pfs group
									of or 1 - 32
			
									1,2,5,22,23,24 weak security
			
								login banner

						on ipsec menu
							phase 1 :
								authentication method
									mutual psk
	
									negotiation method
									main

							phase 2 :
								mod
									transport

								encryption algorithm
									aes128 gcm
									128bit

								hash algorithm
									sha1
							
							identifier
								all users

							preshared key

							then write firewall rules

				users :
					username
					password
					ip address

			openvpn (ptp vpn and opensource)
				servers
					disable (default is disable)

					server mode
						peer to peer
							ssl\tls
							sharedkey
						remote access
							ssl\tls
							user auth
							ssl\tls + user auth

					device mode
						tun mode carries ip4/6 (layer 3) and is the most common and compatible mode across all platforms

						tap mode is capable of carrying 802.3 (Layer 2)

					protocol
						udp ip4/6
						tcp ip4/6
						multihome

					interface
						interface or virtual ip address where openvpn will receive client connections

					local port
						1194

					tls configuration
						automatically generate a tls Key

						default is enable

						tls key enhances security of an openvpn connection by requiring both parties to have a common key before a peer can perform a tls handshake

						layer of hmac authentication allows control channel packets without the proper key to be dropped, protecting the peers from attack or unauthorized connections

						tls Key does not have any effect on tunnel data

					peer certificate authority
					peer certificate revocation list

						system > cert manager

					ocsp check
						online certificate status protocol
						default is disable

					server certificate

					dh (deffie hellman) parameter length
						1024
						2048
						3072
						4096
						6194
						8192
						ecdh only

					ecdh curve
						default
						prime 256 v1
						secp 384 r1
						secp 521 r1

						The elliptic curve to use for key exchange
						the curve from the server certificate is used by default when the server uses an ecdsa certificate
						otherwise, secp384r1 is used as a fallback

					data encryption renegotiation
						enable is default

						allows openvpn clients and servers to negotiate a compatible set of acceptable cryptographic data encryption algorithms from those selected in the data encryption algorithms list below

						disabling this feature is deprecated

					data encryption algorithms
						list is ignored in shared key mode

					fallback data encryption algorithms
						default is :
							aes 256 cbc 256 bit key 128 bit block

						fallback data encryption algorithm used for data channel packets when communicating with clients that do not support data encryption algorithm negotiation (shared key)

						algorithm is automatically included in the data encryption algorithms list

					auth digest algorithm
						default is : sha 256

						algorithm used to authenticate data channel packets, and control channel packets if a tls key is present

						when an aead encryption algorithm mode is used, such as aes-gcm, this digest is used for the control channel only, not the data channel

						server and all clients must have the same setting while sha1 is the default for openvpn, this algorithm is insecure

					hardware crypto
						intel rdrnd - engine rand
						no hardware crypto acceleration

					certificate depth
						one to five
							1 > client + server
							2 > client + intermediate + server
							3 > client + 2intermediate + server
							4 > client + 3intermediate + server
							5 > client + 4intermediate + server
						and donot check

						certificate-based client logs in, do not accept certificates below this depth

						useful for denying certificates made with intermediate cas generated from the same ca as the server

					client certificate key usauge validation
						enforce key usage

						verify that only hosts with a client certificate can connect eku : "tls web client authentication"


					ip4/6 tunnle network
						virtual network or network type alias with a single entry used for private communications between this server and client hosts expressed using cidr notation 10.0.8.0/24

						The first usable address in the network will be assigned to the server virtual interface The remaining usable addresses will be assigned to connecting clients

					redirect ip4/6 gateway
						default is disable

						force all client-generated ip4 traffic through the tunnel

					ip4/6 remote network
						ip networks that will be accessible from the remote endpoint. Expressed as a comma-separated list of one or more cidr ranges or host/network type aliases

						may be left blank if not adding a route to the local network through this tunnel on the remote machine This is generally set to the lan network

					concurent connection
						maximum number of clients allowed to concurrently connect to this server

					allow compression
						refuse any non-stub compression (most secure)
						decompress incomming do not comress outgoing (asymmetric)
						compress packet (warning potentially dangerous)

						allow compression to be used with this vpn instance
						compression can potentially increase throughput but may allow an attacker to extract secrets if they can control compressed plaintext traversing the vpn (http)

						before enabling compression, consult information about the voracle , crime, time, and breach attacks against tls to decide if the use case for this specific vpn is vulnerable to attack

						asymmetric compression allows an easier transition when connecting with older peers

					compression
						deprecated compress tunnel packets using the lzo algorithm

					push compression
						disble is default
						push the selected compression setting to connecting clients

					type of services
						disable is default
						set the tos ip header value of tunnel packets to match the encapsulated packet value

					inter client communication
						disable is default
						allow communication between clients connected to this server

					duplication connection
						disable is default
						allow multiple concurrent connections from the same user
					
						when set, the same user may connect multiple times. When unset, a new connection from a user will disconnect the previous session

						users are identified by their username or certificate properties, depending on the VPN configuration

						practice is discouraged security reasons, but may be necessary in some environments

					dynamic ip
						disable is default

						allow connected clients to retain their connections if their IP address changes.
					
					topology
						subnet one ip address per client in a common subnet
						net30 isolated / 30 network per client

						specifies the method used to supply a virtual adapter ip address to clients when using tun mode on ip4

						some clients may require this be set to subnet even for ip6, such as openvpn connect (ios/android)

						older versions of openvpn (before 2.0.9) or clients such as yealink phones may require net30

					ping setting
						interactive
						ping method
							keepalive
								helper to define ping configuration
							ping
								define ping pingexit pingrestart (can set restat or exit mode for timeout for vpn) manually

									ping = interval
									ping-restart = timeout*2
									push ping = interval
									push ping-restart = timeout

									push ping-restart/ping-exit to vpn client

						interval 10
						timeout 60

					custom option
					udp fast io
						disable is default

						use fast i\o operations with udp writes to tun/tap experimental

						optimizes the packet write event loop, improving cpu efficiency by 5% to 10%

						not compatible with all platforms, and not compatible with openvpn bandwidth limiting

					exit notify
						reconnect to this server \ retry once
						reconnect to next server \ retry twice
						disable

						Send an explicit exit notification to connected clients/peers when restarting or shutting down, so they may immediately disconnect rather than waiting for a timeout

						ssl\tls server modes, clients may be directed to reconnect or use the next server

						option is ignored in peer-to-peer shared key mode and in ssl/tls mode with a blank or /30 tunnel network as it will cause the server to exit and not restart


					send receive buffer
						default
						64 k
						128 k
						256 k
						512 k
						1 m
						2 m

						configure a send and receive buffer size for openvpn

						default buffer size can be too small in many cases, depending on hardware and network uplink speeds

						finding the best buffer size can take some experimentation

						to test the best value for a site, start at 512 k and test higher and lower values


					gateway creation
						both
						ip4
						ip6

						assign a virtual interface to this openvpn server

					verbosity level
						3 is recommended

						none: only fatal errors
						default through 4: normal usage range
						5: output r and w characters to the console for each packet read and write. Uppercase is used for tcp\udp packets and lowercase is used for tun\tap packets
						6-11: debug info range


				clients
					like server

				client specific overrides
					all are disable

					common name
						enter the x.509 common name for the client certificate, or the username for vpns utilizing password authentication

						match is case sensitive enter default to override default client behavior

					connection blocking
						block this client connection based on its common name

						prevents the client from connecting to this server

						do not use this option to permanently disable a client due to a compromised key or password

						use a crl (certificate revocation list) instead

					server lists
						select the servers that will utilize this override
						when no servers are selected, the override will apply to all servers

					ip4/6 tunnel network
					ip4/6 local networks
						if set this our clients goes to tunnel 

					ip4/6 remote networks

					redirect gateway
						all clients traffics goes to tunnel

					server definitions
						Prevent this client from receiving any server-defined client settings.

					remove server routes
						Prevent this client from receiving any server-defined routes without removing any other options.

					dns default domain
						Provide a default domain name to clients

					dns servers
						Provide a DNS server list to clients

					ntp servers
						Provide an NTP server list to clients

					netbios options
						enable netbios over tcpip
							
						if this option is not set, all netbiosover-tcpip options (including wins) will be disabled

					advance
						enter any additional options to add for this client 

						specific override, separated by a semicolon

						push "route 10.0.0.0 255.255.255.0";

				wizard

		status
			we can see states and restart them from here with their own services

			captive portal
			carp (failover)
			dashboard
			dhcp leases
			dns resolver
			filter reload
			gateways
			interfaces
			ipsec
			monitoring
			ntp
			openvpn
			queues
			services
			systmlogs
			traffic graph
			upnp and nat-pmp

		diagnostic
			arp table
			authentication
				we can test user here

			backup and restore
				backup area
					we can use area for each services to backup and restore

				skip packages (disable)

				skip rrd data (enable)
					do not backup rrd data
					rrd data can consume 4+ megabytes of config.xml space!
 
				include extra data (disable)
					captive portal - captive portal db and usedmacs db
					captive portal vouchers - used vouchers db
					dhcp server - dhcp leases db
					dhcpv6 server - dhcp6 leases db

				backup ssh keys (enable)
					otherwise clients would fail to recognize the host keys after restore

				encryption

				files are on xml

				we have seperated part for restore

				also we have config history here (snapshots)

			command prompt

			dns lookup

			edit files
				winscp

			factory defaults
			
			halt system
			
			limiter info
			
			ndp table
			
			packet capture
			
			pfinfo
				default is enable
				automatically refresh the output below
			
			ping

			reboot

			routes
				resolve names (disable)
			
			smart status
				hdd status

			sockets

			states

			staets summary

			tables
				sshgaurd
				bogon
				bogonv2
				snort2c
				virusprot

			test port
			traceroute

	some tools :
		nmap
		suricata (learn rules)
		haproxy ( loadbalance with proxy mode and ssl use)
		pftop
		tcpdump ( use -d -interface)
		tcpflow (realtime and like tcpdump)

	snort :	
		common vulnerabilities and exposures
			community (general)
			registered (must register on site has many patterns)
			subscription (shared objects has more cost need to update)

		if install on windows must install npcap

		snort.org

		after install :
		snort\etc\snort.conf
			var rule_path c:\snort\rules
			var so_rul_path c:\snort\rules
			var preproc_rule_path c:\snort\rules
			white list and black list must be on same path

			config logdir c:\snort\log

			dynamicpreprocessor c:\snort\lib\...
			dynamicengin c:\snort\lib\...sf_engine.dll

			#dynamicdetection ....

			on part 5 must disable all features with #

			preprocessor arpspoof (enable)

			process reputation :
				whitelist $white_list_path\white.list.rules
				blacklist $black_list_path\black.list.rules

				must make save as from above files to this path :
					c:\snort\rules
						(filename > white.list.rule)

			then convert all / to \

			decoder and preprocessor event rule
				active all

			preprocessor portscan:proto ...
				enable this

			snort -V
			snort -W (list of interfaces)
			snort.exe -i 1 -c c:\snort\snort.conf -T (check configs)

			we have local and manual snort rule :
				c:\snort\rule\local
					alert ismp any any -> any any (msg:"snort testing";sid:10000001)

			snort.exe -i 1 -c c:\snort\snort.conf -a console (check configs on ms console)

			site : rule.emergingthreats.net\open\snort..

			openid is an application detector

			pfsense > system > packages > available packages
				snort

			services > snort
				global setting :
					vulnerabilities research team (vrt)
						snort team name and give access to update all of them
						must buy a code
					glv2
						use internal codes for device
					emerging threats rule

				update
					don't show us update process but show success of procedure
					in higher versions we can see this
					services > snort > uodate > manage rule set log

			snort rules use these steps from version 2.8 :
				1- port number
				2- tcp \ udp \ else
				3- ip source
				4- ip destination
				5- payload

				we have rtn (root) and otn (optional)
					if have many same root process id use one rtn and theen analys rtns

				step 1 > generate hash 18 bit
				step 3 > generate hash 16 bit
				step 5 > generate hash 32 bit from combination two above hashes

			logmgmt:
				general
					remove snort logs on pkg unistall
					autolog management

			services
				snort
					passlist 
						like a whitelist

			firewall
				aliases

			services 
				snort
					snort-interfaces
						non-setting
							alert setting
								block offenders (every body generate snort log bloack them)
							detection performance setting
								search method (ac-bnfa)
							choos network snort should inspect and white list
								varuable set
						categorize
							must set some categories then start service of snort
							snort vrt ips policy selection
								connectivity
								balance
								security

	pfblock :
		system > package management > available
			pfblockng

		after version 2 add this feature
		use dns blocking

		firewall
			pfblocking
				general
					can enable feature
					kill state must be enable
					important tabs
						log
						rule
						aliases
				update
					dns blcok list url
						cron
				geoip
					ipv4
						hav list for blocking
						s3.amazonaws...
						actions must set

			after each changes must update it on manual mode in pfblock

				we can set dns filtering with pfblock package must set our dns respons on pfsens with firewall rules :
					services > dns resolver
					firewall > rules on wan :
						src > any
						des > lan
						port > 53 dns
						action > pass

						src > any
						des > any
						port > 53 dns
						action > block

					firewall 
						pfblock
							dnsblock
								easylist
									force popup blocking
								feed
									have many offers must use them
									list action
										outbound
									format > auto
									state > on
								general
									enable feature
									virtual ip to respons
									dnsblock ip firewall rule setting
										deny both

					if need use dns blocking must enable pfblock on firewall panell, also check update on manual mode

	monitring tools for pfsense :
		system > package management > available
			ntopng

		diagnose > ntopng setting
			enable
			set password

			works on port 3000

	active directory and pfsense :
		system 
			usermanager
				users
				authentication server
					server setting
						type
							ldap
								389
								protocol version 3
								hostname or ip address
								search scope > sub-tree
								base dn > original domain name
								bind credential > usename to bind ad and pfsense (our user must be on administartor group on ad)
									initial tempelate > microsoft ad
								authentication container dn > group or ou style must detect
								transport > standard tcp

							must define dns server on microsoft server
								ptr record on reverse (must use allow check box) and forward zone
								restart dns service on server and set dns ip address on interface
							radius
				setting
					domain must set here
					can test connectivity here to see details
				groups 
					if create groups on ad must add them here and assign them attributes

					assigned privileges > means set some access levels to groups or .... (like sensetive data access on user defined in mikrotik)

			general setup :
				dns and doamin of users must set

	squid :
		system > packages > squid

		we can use http and https proxy and limits on mime acl or use and set qoata and url filter

		services 
			squid
				enable squid proxy
				transparetnt http proxy 
				also enable local cache (on default mode is better)

				if need bypass proxy
					bypass proxy for these source ips

				must enable access logging to see activity of clients on raltime actions
		
					transparent proxy make some user bypass if need to force them on login must goes to :
				authentication
					required authentication
				acls
					can add range or single ip to restrict or allow access them
					or can limit download some files extentions (block mime)

					on blocking some sites in squid we can see version of squid so to hide version :
				general
					suppresses squid version string ....

				our proxy on pfsense works on 3128

						need to open https sites with proxy mode :
					
					https ssl interception
					ca
						system > cert manager > make an internal certificate authority
					set remote cert check on accept
					next box on second item

					*need squid servvice restart

						must enable local cache
						in authentication tab we have captive portal

				if need force download limitations must goes to traffic management

		lightsquid is a package for ad and logs
		squid is https and https transparent proxy
		and squidgaurd is web filter

		connecting active directory to squid
			system > package management > light squad
			status > lightsquid 
				works on 7445 and just accessable on web
			to use it must add loopback on services > squid > interfaces

			services > squid transparent proxy > authentication
				method > ldap
				authentication server > active directory server ip address
				authentication processes (after some tries bypass login)
				required authentication for unrestricted ips
				ldap version 3
				ldap server user dn > admin ou
				ldap password
				ldap base domain > domain name
				ldap username dn attribute > sAMAccountName
				ldap search filter > sAMAccountName=%s	

		squad gaurd :
			system > package management > squidgaurd
				analys https in full mode
				works on transparent
				must enable local cache on squad
				also active transparent http proxy and https ssl proxy(don't need install certificate)
					splice all (works on squid gaurd)
						ssl mtlm mode > splice all

			services > squidgaurd proxy filter
				blacklist
					shallalist.de (site)
					can add harmfull sites db tjen download them
				common acl
					block list define at above
					target rule list
				general	
					enable
					must apply after each changes
					blacklist options
						active list at above
						can add offline
				target category
					redirections
					redirection modes

				some time lock on feature must  :
					squid -k shutdown

	clamav :
		icapservice
		redirector

		need squid

		on shell must set (update clamav) :
			freshclam

		system
			packages
				available packages

		services 
			squid transparent proxy 
				general
					enable
					https ssl interception
				antivirus
					enable
					google safe search
					audio and video exclude
					database update interval
				local cache
					enable

		status
			services
				c-icap
				clamd

	NTP :
		system > general > localization

		services > ntp > setting
			0.europe.pool.ntp.org
			1.europe.pool.ntp.org
			2.europe.pool.ntp.org
			3.europe.pool.ntp.org

		on shell :
			timedatectl status
			apt install ntpdate -y
			ntpdate 192.168.57.100
			apt install ntp -y
			vim /etc/ntp.conf
				pool 0.ubuntu.pool.ntp.org iburst
				pool 1.ubuntu.pool.ntp.org iburst
				pool 2.ubuntu.pool.ntp.org iburst
				pool 3.ubuntu.pool.ntp.org iburst
			systemctl restart ntp
			ntpq -p
			timedatectl status

		status > system logs > firewall

	SynProxy:
		firewall
			rules
				wan
					advance option
						state type
							synproxy

	make delay beetwen requests :
		Postpone resource allocation until the connection is completed using a cookie
		This information is closed from the first time by the customer (SYN package) instead of stored in RAM in the data section (Payload) the second package (SYN + ACK) is stored and sent to the customer to do this . It is called cookie production, so there are two ways:
			If the sender of the first packet is an intruder, the server receives the third packet, the cookie is lost, and the server allocates its resources.
			If the sender of the first is a regular customer who needs to connect, close the second with a cookie, send the third with a cookie, the server receives the third packet and knows that it is from a customer. It is sent normally because it has a cookie with it, this server can allocate the required resources.

			system
				advance 
					system tunable
						net.inet.tcp.syncoockies

	antispoof :
		interface 
			wan
				block private networks and loopback address
				block bogon networks

				after enable these we can find 2 rules on firewall > rule > wan
					block some ips on wan interface incoming to lan :
						10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
						172.16.0.0 - 172.31.255.255 (172.16.0.0/12)
						192.168.0.0 - 192.168.255.255 (192.168.0.0/12)

						if need to set inteerval update for boggon must :
							system 
								advance 
									firewall & nat
										update frequency
										also can find maximum concurent session in same time and change it with (firewall maximum states)

	user definition on radius :
		object
			vpn
				profile
					name
					expiration
					timeset
					idle time
					reset time period
					amount of time (duration)
					up rate
					down rate
					reset traffic period
					traffic (log views)

	ips and ids on apk :
		device
			security services
				ips\ids
					interfaces
						set interface

						on submenu must select 
							iface_categories 
							iface_barnyard2(for log)
								mysql db output setting
									enable logging of alerts to a mysql instance (make enable)
									user : apk
									pass : 
							iface custom rule
								add manual snort cve and rule

	waf on apk (ned create certs) :
		device
			security services
				waf
					security policy
						options
							modsecurity protection
							 	must be enable
					general setting
						webserver (internal values)
							name > must be on owasp or site name
							type
							host
								fqdn
								port (80 / 443)
								weight
						site path
							set name and policy

						virtual servers
							listening option
								enbale (default is disable)
								mode (means blocking option)
								type > https
								some times must redirect 80 to 443 must use redirect option	protected host name > name of site

							advance
								can change ssl certificate values and redirect them
								uncheck past host header
								checked rewrite html
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	mysql :
		waf :
			mysqldump --no-create-info --complete-insert -uapk -papk-gate waff vhost > sqlBackup_waff.sql

			v4 :
				mysqldump --no-create-info --complete-insert -uapk -papk-gate radius  addresses addresses_ip app_name app_policy app_profile app_setting change_view charge_pack golestan rel_appname_appprofile rel_policy_appprofile rel_policy_users rel_policy_webfilter sms_panel users users_alias users_attr usr_per_relation webfilter config profiles radcheck radgroupcheck radgrouplimit radgroupreply radreply radprofile raduserlimit radusergroup> sqlBackup_radius.sql

		----------------------------------------------------
			v5 :
				/usr/local/bin/mysqldump -uapk -papk-gate --no-create-info --complete-insert radius --ignore-table=radius.counter --ignore-table=radius.cui --ignore-table=radius.dashboard --ignore-table=radius.data_counter_old --ignore-table=radius.data_counter_today --ignore-table=radius.error_handling --ignore-table=radius.ip2country --ignore-table=radius.nas --ignore-table=radius.payment --ignore-table=radius.paymentsetting --ignore-table=radius.proxy --ignore-table=radius.suricata_rules --ignore-table=radius.ticket --ignore-table=radius.ticket_note --ignore-table=radius.user_idle --ignore-table=radius.user_permision --ignore-table=radius.wimax --ignore-table=radius.online_user > sqlBackup_radius.sql

				/usr/local/bin/mysqldump -uapk -papk-gate --no-create-info --complete-insert radius --ignore-table=radius.counter --ignore-table=radius.cui --ignore-table=radius.dashboard --ignore-table=radius.data_counter_old --ignore-table=radius.data_counter_today --ignore-table=radius.error_handling --ignore-table=radius.ip2country --ignore-table=radius.nas --ignore-table=radius.payment --ignore-table=radius.paymentsetting --ignore-table=radius.proxy --ignore-table=radius.suricata_rules --ignore-table=radius.ticket --ignore-table=radius.ticket_note --ignore-table=radius.user_idle --ignore-table=radius.user_permision --ignore-table=radius.wimax --ignore-table=radius.online_users --ignore-table=radius.accounting_authenticate --ignore-table=radius.accounting_authenticate_log --ignore-table=radius.radacct --ignore-table=radius.acctemp > sqlBackup_radius.sql
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	IPS Thread Tuning :
		ET INFO EXE - Served Attached HTTP --> emerging-info.rules
		ET INFO Packed Executable Download --> emerging-info.rules
		ICMP Destination Unreachable Communication with Destination Host is Administratively Prohibited --> emerging-ping_flooding.rules
		ET POLICY PE EXE or DLL Windows file download HTTP --> emerging-policy.rules
		ET POLICY SSL/TLS Certificate Observed (AnyDesk Remote Desktop Software)--> emerging-policy.rules
		
		special :
			*ETPRO POLICY Observed Google DNS over HTTPS Domain (dns .google .com in TLS SNI)
			*DOS Unusually fast port any SYN packets inbound, Potential DOS
			ETPRO INFO Observed Google DNS over HTTPS Domain (dns .google in TLS SNI)

		wire config :
			boot\loader.conf
			vfs.zfs.arc_max="1G"
		
		waf and ssl proble on advance section on waf options :
			SSLProxyVerify none
			SSLProxyCheckPeerCN off
			SSLProxyCheckPeerName off
			SSLProxyCheckPeerExpire off
		
		http://192.168.1.105/update-center/v6/index.php
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
