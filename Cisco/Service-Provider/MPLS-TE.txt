MPLS-TE (Traffic Engineering) :
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
general :	
	in mpls some times we have specific path to arrive on network so ospf could use this path also ospf can use anothe path if our path get break
	in ip routing we have routin protocol mechanisms and use link attributes to reachablity f network like > bw + reliability + delay + hopcount + load
	when choose a best path between all paths we could'nt use another path cause we just use best and winners
	in mpls te we use path in effecient mode
	if our path contian these values like 500k and 1.5m in mpls te task and duty must use links on best inwaste mode to optimize link bw usuage

	here we have some criterions or standards  like destination ip checking and perhop behavior
	dependes on router topology and data knowlege

	maybe some bodies use static routes for this concept but static route forward whole traffic and data on interface

	also pbr is another one soloution but i not scaleable

	mpls is our soloution but in mpls we use ldp in mpls te use rsvp
		resource reservation protocol
		or rsvp-te

	advertise labels and igp parameters like ospf leanr and recognize what attribute need and exist on rsvp

	rip and eigrp detect and select best paths but ospf and isis advertise link attributes unlike eigrp and rip

	te extiontion added to isis and ospf 
	have attributes to detect all information of links
		ospf use lsa type 9 10 11
		lsa 10 is te mpls

	smart mechanism to detect link

	constract spf (cspf)(limited on spf)
	path calculation based on cspf

	must use on router cspf instead of perhup behavior and calculation
	another routers just make label switching and forwarding
	for reservation of bw we can use rsvp and rsvp path

	in before las hop we pop labels and unset reserved or standby links to use them
	then mpls labels advertised and forward 

	path message (bw reserve + label)
	head end
	tail end

	in mpls we use downstream on demand (dod) on te mode of mpls 

		ip cef
		mpls label range 100 199
		mpls traffic-engineering tunnels (active mpls te)

	must set specific values on links
		int fast 0/0
		bandwidth 1000 (means 1mbps)

	if didn't apply on interface our ospf doesn't work on constrain
	also rsvp reserve works on this command

		int fast 0/0
		mpls traffic-engineering tunnels
		ip rsvp bandwidth 800 (rsvp get enable)(how much of bw should be used from all bw capacity)
		ip rsvp bandwidth 800 400 (800k  from 1m shold be used and 400k should be used for each flow of packets)

		roouter ospf 1
		mpls traffic-engineering area 1
		mpls traffic-engineering router-id loop0 (works with what rid in mpls te)

		sh ip ospf database (ust show us type 1 of lsa)
		sh ip rsvp interface (which interfaces will be on rspv game)
			in ios use 75% of links could e change

	rsvp doesn't have neighborship
	in lsa 10 of ospf show us details of link

	gre tunnel > [ip [ [ip] , [data] ] ]

	in mps-te select all traffics requirements in container on name of tunnel

		int tun 100
		ip unnumbered loop0

		*we don't use this interface ip any were but must be on rib and way of injection were that
		unnumbered were applicable on device
		just leack reservation on segment part will be usefull

		tun des 192.168.254.4
		tunnel mode mpls traffic-engineering
		tunnel mpls traffic-engineering bandwidth 600 k (means this not reserved)
		tunnel mpls traffic-engineering path-option 1 dynamic (how choose tunnel)

		*after these we see tunnel up log on cli

		show mpls traffic-engineering tunnels
			*explicit route (say which path)

	cspf contain some vlues in egress and export like : nexthop dependes on rsvp + ero (explicit route object) make standby and advertise it to another router

		sh ip rsvp interface (see reserved paths that were standby interfaces)(say hw many get allocate)

	with te we use many links to reach all networks on all link capacities not igp mode and making flat on one bw link

	here we can use pbr

	traceroute 192.168.254.4 numeric

	lsa type 9 - 11 used for opaque or extentions
		9 > scope is on link between 2 router
		10 > scope for area not suitable on foriegn routes
		11 > scope is as and domain like lsa5

		sh ip ospf opaque-link self-orginate (9)
		sh ip ospf opaque-area (10)
		sh ip ospf opaque-as (11)

	used tlv for these extentions
	lsa10 > rid of mpls te and lsa counts + size of path 
		part 1 > mpls te rid
		part 2 > sub tlv  and link details

		sh ip ospf database opaque-are 1.0.0.0
			opaque id and type contains number of lsa and mpls te type (cisco just use this)
			each item has code
			all items are about router

			id is 24 bit (rid)
			type is 8 bit

		sh ip ospf opaque-area self-originate
			lsa age > grows and increment
			option (flags)
			ls type > 10 opaque area link
			opaque type = 1
			opaque-id = 0
			advertise router 192.168.254.1
			ls sequence number 8 000 008
			checksum
			length
			fragment number
			mpls te router id 192.168.254.1

			sub tlv :
				1 > link type > 8bit > ptp / multiaccess
				2 > ink id > 32bit > rid (ptp) / multiaccess (dr address)
				3 > local interface ip address > 32bit
				4 > remote interface ip address > 32bit
				5 > te metric > 32bit
				6 > max bandwidth > 32bit
				7 > max reservation bandwidth > 32bit
				8 > unreserverd bandwidth > 25bit (the main cause of bandwidth assignment)
				9 > administrative group > 32bit

			affinity bit
			igmp metric 100

		sh ip ospf
			area has rrr enable (routing with resource reservation)(another te name)

	for resservation use rsvp (protocol number 46)
		path + label request
			explicit route object (ero)
				nexthop on cspf routing processor (every one recieve this findout what way is the best)
				label request + tunnel specification
					set standby mode if confirmed on frontside router to recieved them and convert them to reserved
		reserve
			after router confirmation and reserver converting our labels transfer then pop labels
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te explicit tunnel ospf :
	router ospf 1
	router-id 192.168.254.1
	int range fas 0/0
	ip ospf 1 area 0
	ip ospf network point-to-point
	int loop0
	ip ospf 1 area 0

	int ether 0/0
	bandwidth 500
	int ether 0/1
	bandwidth 200
	
	sh ip rsvp inteface (still not active)

	sh mpls traffic-engineering tunnels
	sh mpls traffic-engineering tunnels summary
	sh mpls traffic-engineering

	ip cef
	mpls label range 100 199
	mpls traffic-engineering tunnels

	int fast 0/0
	mpls traffic-engineering tunnels
	ip rsvp bandwidth 500 500

	int fast 0/1
	mpls traffic-engineering tunnels
	ip rsvp bandwidth 200 200

	sh ip rsvp interface

	sh ip ospf database
	sh ip ospf

	router ospf 1
	mpls traffic-engineering router-id loop0 (enable our cspf)
	mpls traffic-engineering area 0 (send lsa 10 on this area 0)(fo this reason we use one area in isp for mpls)

	sh ip ospf database
	sh ip ospf database opaque-area
	sh ip ospf database opaque-link
	sh ip ospf database opaque-as

	sh ip cspf (rscp and opaque shown)

	mainly doesn't use mpls ip cause we have runing ldp in network mpls ip is applicated on mpls vpn not in te

	sh mpls traffic-engineering tunnels summary
		*after this our rsvp get enable

	we need launch a tunnel transfer 150k from below way and launch another 1m tunnel to user above path
		int tun 0
		ip unnumbered loop0
		tunnel des 192.168.254.5
		tun mode mpls traffic-engineering
		tun mpls traffic-engineering bandwidth 150 (tunnel bandwidth)
		tun mode traffic-engineering path option 1 dynamic (in dynamic use smart finder)
			path option
				dynamic > find on routing on automatic mode learn with rsvp
				explicit > set administartivly
				semidynamic

		ip explicit-path identifier 10 (here can use number or name)
		next-address 192.168.13.3
		next-address 192.168.34.4
		next-address 192.168.45.5

		*here cspf result transfer to these next address with manual mode
		*ospf just check bw in these path on link attributes

		tunnel mode traffic-engineering path-option 1 explicit id 10

		*till detect path our signaling and rsvp stop and tunnel is down

		sh ip explicit path 
			*compelete means up and we have strict and loose

			if use more cost value for tunnel like 300k our tunnel will be down always

		here in rib we couldn't see mmpls te tunnel cause we didn't set encapsulation for mpls te
		static route and pbr ...

		ip route 192.168.254.5 255.255.255.255 tun 0
		sh ip route static

		traceroute 192.168.254.5 numeric

		sh mpls traffic-engineering tunnels (say what label or tag will be select)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te with isis :
	router isis
	net 49.0000.1111.1111.1111.00
	is-type level-2-only
	log-adjencency-change all
	metric-style wide (ipv6 and te)

	sh ip isis database
	*must use same are for all isp routers like 49.0000

	isis use newest lsp compositions like 
		tlv 22 (subtlv)
		tlv 137 (rid)
		tlv 135 (link path metric)(instead of wide metric with tlv 128 and 130)

		length 		type 	
		4 			3 admin group (tlv22)
		4 			6 ipv4 interface address
		4 			8 ipv4 neighbor address
		4 			9 maximum link badwidth
		4 			10 reserverd link bandwidth
		32 			11 unreserverd link bandwidth

	ip cef
	mpls label range 100 199
	mpls traffic-engineering tunnels
	mpls traffic-engineering router-id loop0
	mpls traffic-engineering level-2 (in ospf use area0 here use level 2)

	sh isis database verbose (still not recieve cause we didn't active on interface)

	int serial 0/0/0
	mpls traffic-engineering tunnels
	ip rsvp bandwidth 1000 1000

	int serial 0/0/1
	mpls traffic-engineering tunnels
	ip rsvp bandwidth 500 500

	sh isis database (lsp)
	sh isis database details
	sh isis database verbose (tlv)

	sh isis protocol (sh topology and constrain)

	int tun 0
	ip unnumbered loop0
	tun destination 192.168.254.4
	tun mode mpls traffic-engineering
	tun mpls traffic-engineering bandwidth 600
	tun mpls traffic-engineering path-option 1 dynamic

	sh mpls traffic-engineering tunnels

	*in isis we don't care about bandwidth

	sh run int tun 0
		here we have priority 
		like 7 7 > first 7 means setup second 7 means hold

	router isis 
	net 49.0000.1111.1111.1111.00
	is-type level-2-only
	metric-style wide
	log-adgencency-change all

	int fas 0/0
	ip router isis
	isis network point-to-point

	int loop0
	ip router isis

	mpls labe range 100 199
	mpls traffic-engineering tunnels
	int fast 0/0
	ip rsvp bandwidth (if enter here means all bandwidth will use by config also flow)
	mpls traffic-engineering tunnel

	router isis
	mpls traffic-engineering router-id loop0
	mpls traffic-engineering level2

	*some scenarios have fewer hop count and isis use fewer hop count path so

	int tun0
	ip unnumbered loop0
	tun mode mpls traffic-engineering
	tunnel mpls traffic-engineering bandwidth 1000
	tun destination 192.168.254.5
	tun mpls traffic-engineering path-option 1 dynamic

	ip explicit path name r1345
	next-address 192.168.13.3
	index 3	next address 192.168.12.2
	next-address 192.168.34.4
	next-address 192.168.45.5
	append-after next-address 192.168.45.5

	int tun 0
	mpls traffic-engineering path-option 1 explicit name r1345

	ip route 192.168.254.4 255.255.255.255 tun0

	*tun get up when has path attribute
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
multiple explicit tunne on isis :
	if some haed ends need to transfer traffic to tail ends
	could set flow paths with explicit tunnels

	we config our isis like before and set all network on 49.0000
	wide metric on mettric style and set interfaces isis routing then set ptp mode on physicals

	sh isis neighbor
	sh ip route isis

	without te we forward traffic in straight path but in te use explicit path

	we must enable mpls label range and mpls te tunnels
	on interface we have rsvp bandwidth and mpls te tunnels

	on head end :
		int tun 0
		ip unnumbered loop 0
		tun destination 192.168.254.7
		tun mode mpls traffic-engineering
		tun mpls traffic-engineering bandwidth 10000
		tun mpls traffic-engineering path-option 1 explicit name r1237

		ip explicit-path name r1237
		next-address 192.168.18.1
		next-address 192.168.12.2
		.....

		*must write full path

		ip route 192.168.254.7 255.255.255.255 tun 0

		sh mpls traffic-engineering tunnels brief
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls ospf with pbr on eplicit tunnels
	all configs are same

	mpls label range 100 199
	mpls traffic-engineering tunnels

	router ospf 1
	router-id 192.168.254.1
	int range ehter 0/0-1/1
	ip ospf 1 area 0
	ip ospf network point-to-point

	int loop0
	ip ospf 1 area 0	

	int range ehter 0/0-1/1
	mpls traffic-engineering tunnels
	ip rsvp bandwidth

	*with pbr can use rib to direct traffic without cahnges

	ip explicit-path name r1237
	next-address 192.168.12.2/23.3/37.7

	ip explicit-path name r1457
	next-address 192.168.14.4/45.5/57.7

	int tun0
	ip unnumbered loop0
	tun mode mpls traffic-engineering
	tunnel destination 192.168.254.7
	tu mpls traffic-engineering bandwidth 100000
	tu mpls traffic-engineering path option 1 explicit name	r1237

	*make another tunnel with explicit-path r1457

	access-list 100 permit ip host 192.168.1.8 any
	access-list 101 permit ip host 192.168.1.9 any

	route-map mypolicy permit 10
	match ip address 100
	set interface tun 0

	route-map mypolicy permit 20
	match ip address 101
	set interface tun 1

	sh route-map 

	interface ether 1/3 (incoming interface from headend)
	ip policy route-map mypolicy

	debug ip policy (which policy get match)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te dynamic tunnel ospf
	all configs are same except path option

	ospf and ip addresses are same also we added interface in ospf with ptp mode for physical interfaces
	rid get set
	all setting are on area 0
	ospf pid is 1

	mpls label range get set
	then use mpls te tunnels in global also on interfaces we set rsvp and bw with mpls te tunnels config

	in ospf we have mpls te tunnels with id and use area 0 activation

	next enable tunnels on destination 192.168.254.5 with mpls te on te mode
	int tun 0 must be unnumbered and bw shoulld define

	ospf select cost that dependent on interface bandwidth

	in some scenarios we have fewer hopcount and less bandwidth than anothers or have another way use more hopcount with more bandwidth

	tunnel mpls traffic-engineering path-option 1 dynamic

	* here ospf cost doesn't matter so use allocate to apply bandwidth
	ospf use fewer hopcount and cspf use more hopcount with more bandwidth

	in path option we have dynamic mode with 3 model :
		lock down
		bandwidth
		attributes

	ip route 192.168.254.5 255.255.255.255 tun 0 (direct traffic and use rib and make changes on rib)

	sh mpls traffic-engineering tunnels
		in/out label
		signaling 

	sh mpls traffic-engineering tunnels up / summary / brief / statictis / accounting / topology (details)(mpls te id + rid + system id)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te semiynamic tunnel eclude address
	administartivly set our path on explicit

	semidynamic say sometimes need use dymaic path with excluded path and use all routers except some lsr and routers and totaly use dynamic model

	router isis 
	net 49.0000.1111.1111.1111.00
	log-adgencency-change all
	is-type level-2-only
	metric-style wide
	mpls traffic-engineering level2
	mpls traffic-engineering router-id loop0
	mpls label range 100 199

	interface range fast 0/0-1-1
	ip router isis
	ip rsvp bandwidth 500 500
	isis network point-to-point
	mpls traffic-engineering tunnels

	int loop0
	ip router isis
	mpls traffic-engineering tunels

	sh isis neighbor
	sh ip route isis

	int tun 0
	ip unnumbered loop0
	tun mode mpls traffic-engineering
	tun mpls traffic-engineering band 400
	tun destination 192.168.254.7

	tun mpls traffic-engineering path-option 1 dynamic (if use this our policy didn't use)

	mpls traffic-engineering reoptimize tunnel 0 (make changes faster)

	now use semidynamic
		ip explicit-path name exclude-r6
		exclude-address 192.168.16.6

		int tun0
		tun mpls traffic-engineering path-option 1 explicit name exclude-r6

		*here use all path dynamic mode except r6
		in background use routes to reach destination

		ip route 192.168.254.7 255.255.255.255 tun 0

	another method of semidynamic
		if has 2 way on some scenarios and exclude one path maybe get in trouble
		in last option we were using some dynamic model here we use explicit model
		use loop back ip address instead of interface ip address (cause use redundant interfaces)

		sh ip explicit-path (all of them use strict mode means just use these ip address no other)

		ip explicit-path
		next-address loos 192.168.254.4 (here use loopback ip address)

		*cause use loopback ip address we could find then set on tunnel interface
		we config transfering traffic but need config reply path or recieving path
		msut config like this router on another router
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
automatic bandwidth
	help us on allocate bandwidth

	in our topologies we doesn't have real bandwidth and need manage on it
	we have self tunning tunnel on auto bandwidth
	each 300 seconds get analyze each tunnel interface to find what up down happend
	each 5 minutes use average value for bandwidth

	adjustment > 1 day
	sample interval > 5 minutes
	max 1000
	min 400
	no peak

	depends on real load of interfaces
	in one day analyze all peaks and up downs to detect real output and set in output

	must config te before these

	in global
		mpls traffic-engineering auto-bw times frequency 300 (default is 300 could change)

		int tun 0
		tun mpls traffic-engineering auto-bw (active on interface tun)

		tun mpls traffic-engineering auto-bw collect (means just see don't set any thing)

		tun mpls traffic-engineering auto-bw 800 1500 (min and max)

	mpls traffic-engineering tunnels 
	mpls label range 100 199
	mpls traffic-engineering auto-bw timers 200

	router ospf 1
	router-id 192.168.254.1
	
	int fas 0/0
	ip ospf 1 area 0
	ip ospf network point-to-point
	ip rsvp bandwidth
	mpls traffic-engineering tunnels

	int loop0
	ip ospf 1 area 0

	int tun 0
	ip unnumbered loop0
	tun mpls traffic-engineering
	tun mpls traffic-engineering bandwidth 1000
	tun mpls traffic-engineering path-option 1 dynamic

	*here auto-bw analyze all link to find how much you need

	in global
		mpls traffic-engineering auto-bw timers frequency 5 (each interface use auto-bw will be capture)

		int tun 0
		tun mpls traffic-engineering auto-bw frequency 300
		tun mpls traffic-engineering auto-bw max-bw 1000
		tun mpls traffic-engineering auto-bw min-bw 300

		*if minimum detected bandwidth finded by auto-bw our manual config will be use
		after this if our traffic be less than 10m our auto-bw start procedure and change bandwidth

	clear mpls traffic-engineering auto-bw
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
affinity bit attribute flag
	each tunnel has many parameters and use specific values to make connection till here we used bandwidth now use another attributes
	some situations in network use same bandwidth but use different media like cable and wireless
	in cale we have less latency
	one soloution is semidynamic and exclude address (not so scaleable)
	another way is resource class attribute use resouce class attribute bit
		32bit
		from 0000 0000
		to ffff ffff

		can set 0x00000001 and 0x00000002 on eah interface to make isolation
		here set affinity bit and values on wireless and cable to make diffrences

		define coding

		in tunnels connection we use bandwidth and affinity bit to detect better path
		on lsa we send these

	on head ends
		ip cef
		mpls ip
		mpls label range 100 199
		mpls traffic-engineering tunnels
		mpls traffic-engineering auto-bw
	
		router ospf1
		router-id 192.168.254.1
		mpls traffic-engineering router-id loop 0
		mpls traffic-engineering area 0
	
		int range fas0/0-1/1
		ip ospf 1 area 0
		ip ospf network point-to-point
		mpls traffic-engineering tunnels
		ip rsvp band
	
		int loop0
		ip ospf 1 area 0
	
		int tun 0
		ip unnumbered loop0
		tun mode mpls traffic-engineering
		tun mpls traffic-engineering band 1000
		tun des 192.168.254.4

		tun mpls traffic-engineering path-option 1 dynmaic 

		*if set affinity value on 0 our tun doesn't show us vakue

		sh mpls traffic-engineering tunnels
			affinity > 0x0
			mask > 0xffff (like ip and subnet mask) (which bit is variable)
			0is not important
			1is important (emphasis on the equivalence of link value)

			attribute flag :
				0x 0000 ffff
					p1   p2
	
				p1 > 0000 0000 0000 0000
				p2 > 1111 1111 1111 1111

			or
				0x 0000 0001
					p1   p2

				p1 > 0000 0000 0000 0000
				p2 > 0000 0000 0000 0001

			admin setting
				0x00000000
				or
				0x0000ffff

			we check incoming values 
				0x00000000 not equal 0x00000001

	on head end set this on outgoing interface
		int fas 0/0
		mpls traffic-engineering attribute-flag 0x00000001

	on isp 
		int range fast 0/0-0/1
		mpls traffic-engineering attribute-flag 0x00000001

		int range fast 0/2-0/3
		mpls traffic-engineering attribute-flag 0x00000002

	inside isp routers 
		int range fast 0/2-0/3
		mpls traffic-engineering attribute-flag 0x00000002

	sh ip ospf database opaque-area self-org

	on head end
		int tun 0
		tun mpls traffic-engineering affinity 0x00000001 mask 0xffffffff
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
admin weight
	we have 2 type metric on mpls te
		igp metric and te metric
		between some values select some specifics on ip routing and rib
		in te we above tips and use additional options
		in mpls te we use igp metric and te metric inlike igp protocols like ospf and isis just use igp metric and cost

		some scenarios use fewer cost to set path
		if had voice and data in our network should make different policy to controll them 
		should set te metric on lower value then add it to mpls te and admin weight

		al configs are same

	on isp pe end and connected routers to it also set on tail end router
		int range fast 0/0-01
		ip ospf cost 100

	on head end  
		int tun0
		ip unnumbered loop0
		tun destination 192.168.254.4
		tun mode mpls traffic-engineering
		tun mpls traffic-engineering tunnels
		tun mpls traffic-engineering bandwidth 1000
		tun mpls traffic-engineering path-option 1 dynamic

		sh mpls traffic-engineering tunnels (find metric type)

		another headend must usse igp metric and change path 
			int tun 0
			tun mpls traffic-engineering path-selection metric igp

	on tail end
		int fas 0/0
		mpls traffic-engineering admin-weight 150
		int fast 0/1
		mpls traffic-engineering admin-weight 100

		*if set cost on interface isis or ospf change te metric automatic
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te  hold priority isis
	int tun 0
	tun mpls traffic-engineering priority 7 7

	on lsa 10 lsp has some constraint informations like unreserved bandwidth

	first 7 means setup
	second 7 means hold

	0 is highest priority
	7 is lowest priority

	in example :
		if use link between devices and be on 1m 
		then setup a tunnel on these head end and tail end routers 
		all bandwidth assigned to this tunne after these we need launch another tunnel but never get up cause no bandwidth to be consumed
		with priority can setup and change launchig order

		tun 0 > voice > setup priority 2 hold priority 2
		tun 1 > data > setup priority 7 hold priority 7

		if need 1m in tun1 and launch it our device make it standby and behave like preemtive
		standby tunnel use another way to get up or get down
		here we have hold priority
		if didn't config that our first tunnel hold value and secondary tunne setup value will be compared

		if our setup value be lower than hold priority our lower setup priority tunnel get up and start working

		hold priority means preemtive
		setup means in launching time which of them must start faster

		some ios never access you change hold and make it worst than setup value

		in tunnel running we use bandwidth and see bandwidth on unreserverd bandwidth part

		tun 0 > 400k from 1m > 600k free > setup priority 7 hold 7
		tun 1 > 700k from 600k > drop > setup priority 3 hold 3

		we make replace in device and make tun 1 up then make tnnel 0 up if be same on priority have error in order of turning up

		we use levels to advertise available bandwidth on network in lsa
				use 	avl
			0	0 		1m
			1	0 		1m
			2	0 		1m
			3	0 		1m
			4	0 		1m
			5	0 		1m
			6	0 		1m
			7	400k 	600k

		all configs in sceanrio is same like isis

		sh isis database r2.00-00 verbose
			here see level 7 use bw
			if level 4 comes could set these band

		int tun0
		tun mpls traffic-engineering priority 3 3 (here shut tun 1 and make connection on tun 0 then make sh on isis like above say level 012 could use all bandwidth)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
reoptimization
	like clearing
	in automatic mode
	happen in 3 times
		periodic
			1 hour	
			te checks paths to select best path 
			las tunnel get down and run new
			need more time
			bw release
		
		event driven
			one link get up and down then attributes get changed > triger link down

		manual
			mpls traffic-engineering reoptimization
			
			reoptimization effects on all tunnels or one tunnel can reoptimize
			reoptimization works on dynamic mode of path-option not explicit also has lockdown option to make no effect on it

			applicated on fix scenarios

			mpls traffic-engineering reoptimize events linkup (in default is disable when turn up trigger and reoptimization happen, don't wait to periodic)
			*if make lock down and make reoptimize our way and path didn't change

			mpls traffic-engineering reoptimize timers frequency 30 
			mpls traffic-engineering reoptimize timers delay 10
			mpls traffic-engineering reoptimize timers dealy clceanup\installation
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
auto route
	before autoroute we used static route and pbr for to forward traffic
	our frowarding get automatic and dynamic
	our tunnel interfaces will be like connected interface on router and rib
	use metric 
	didn't inject into the igp calculations
	for after these networks will be useful

	head end 					tailend0 								tailend1		
	r1------------r2------------r3------------r4------------r5------------r6------------r7
								tun0									tun1

	all configs are on ospf and same as before

	dynamic bandwidth on tunnels are 10m

	in sh ip route ospf (see all network will be visible on from 2 part one part igp and one part tun)

	int tun 0
	tun mpls traffic-engineering autoroute announce (make our feature activate then make connected mode on rib)

	after r3 will be visible from tun0 and after r6 will be find on tun1

	used igp metric for show
	tunnels metric get better than ospf metric

	int tun 0
	tun mpls traffic-engineering autoroute metric 3 (when for one router in rib we used smae metric we have loadbalancing)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
forwarding adjencency 
	autoroute hs some limitation
	just source have access to use these shortcuts
	with forwarding adjencency we can earn our rib to all routers in area
	forwarding adjencency make a real interface with tunnel
	used in isis and ospf
	advertise in lsa
	must make a  tunnel on tail end to detect this states
	applicated in unequal load balancing
	in isis and ospf on cost reason doesn't use many links

	in some scenarios we make tunnels and our path get shorter than before

	*if need load balancing on isis and ospf must use mpls te

	make real interface from our virtual interfaces 
	altho all path get change like device crash or turning off use another way or path

	head end
		int tun 0
		tun mode mpls traffic-engineering
		tun destination 192.168.254.3
		ip unnumbered loop0
		tun mpls traffic-engineering bandwidth
		tun mpls traffic-engineering path-option 1 dynamic
		tun mpls traffic-engineering forwarding-adjencency (set hold time)

	head end router has 2 way to reach front side router one way is igp another is tunnel
	if use same metric could set loadbalancing

	we set head end config for tail end config with destination of head end

	our main routers and paths get hide from another routers

	sh isis database r4-00.00
		say our routers are using metric 10 and main routers get hide not computed

	sh mpls traffic-engineering tunnels
		say we were using forwarding adjencency

	sh isis neighbor
		no neighborship happen because on interface tunnel need encapsulation
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
class base tunnel selection
	cbts
	works with qos
	after tunnel config our traffic didn't forward automatic on it
	cbts is one of the forwarding way

	mpls label
		32 bit
			20bit > acctual label
			3bit > exp (exprimental)
			1bit > ottom of stack (bos)
			8bit > ttl

			exprimental is qos field and cbts part

	ipp in tos field on ip header
		tos > 8bit
			ipp
				000 00000 > first part were used values like 1 to 7 (7 is highest)
			dscp
				000000 00

	lsr use exp part of mpls label to forward
	with exp we can set outgoing te tunnel and make destination reachable
	mqc > modular quality of service cli
	class base > class map > policy map

	example
		all configs are same
		head end
			int tun0
			ip unnumbered loop 0
			tun mode mpls traffic-engineering
			tun mpls traffic-engineering tunnels
			tun des 192.168.254.5
			tun mpls traffic-engineering bandwidth 1000
			tun mpls traffic-engineering path-option 1 explicit name r125

			ip explicit-path name r125
			next-address 192.168.12.2/25.5

			*make tun1 and tun2 with another explicit-path values to forwarding

			int tun0
			tun mpls traffic-engineering exp 0 1 2 4

			int tun1
			tun mpls traffic-engineering exp 3 5

			int tun2
			tun mpls traffic-engineering exp 6 7

		here our ios didn't effect from our policy
		must inject to one bundel then process them

		make master tunnel then all another tunnels joined them

		after making master part with forwarding adjencency or ... forward them

		head end
			int tun 100
			ip unnumbered loop0
			tun destination 192.168.254.5
			tun mode mpls traffic-engineering 
			tun mpls traffic-engineering tunnels
			tun mpls traffic-engineering ex-bundel master
			tun mpls traffic-engineering ex-bundel memeber tun 0
			tun mpls traffic-engineering ex-bundel memeber tun 1
			tun mpls traffic-engineering ex-bundel memeber tun 2
			tun mpls traffic-engineering autoroute announce (inject on rib)

			sh mpls traffic-engineering tunnels (show exprimental and master memeber)
			sh ip route (all networks after head end get tunnel)
			
			none of the traffics has no exprimental label must enable ldp on mpls to add them

			router ospf 1
			mpls ldp auto-config

			if ommit exp from tun2 make tun0 on default mode and forward on it

			int tun2
			tun mpls traffic-engineering exp 6 default (use 6 and another)

		ipp
			value 		bit 		
			0 			0 		000 00000
			1 			32 		001 00000
 			2 			64 		010 00000
			3 			96 		011 00000
			4 			128 	100 00000
			5 			160   	101 00000
			6 			192     110 00000
			7 			224 	111 00000
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
atom
 	any transport over mpls
 	on pe routers we launch mpls layer 3 vpn and make vrf with rt and rd also setup vpnv4
 	use for customers need layer3 vpn
 	some users need layer 2 vpn

 	on pe
 		psudowire-class cu1
 		encapsulation mpls
 		int fas 0/2
 		xconnect 192.168.254.1 1 encapsulation mpls pw-class cu1

 		sh mpls l2transport vc 1 details

 		*here need config tunnel on pe routers to transfer laer2 vpns

 		int tun 0
 		ip unnumbered loop0
 		tun destination 192.168.254.3
 		tun mode mpls traffic-engineering
 		tun mpls traffic-engineering bandwidth 1000
 		tun mpls traffic-engineering path-option 1 explicit name r1453

 		ip explicit-path name r1453
 		next-address 192.168.14.4/45.5/53.3

 		psudowire-class cu1
 		prefered-path interface tun0
 			we must do this actio on another pe side with returning path of nextaddress on explicit path

 			if get trouble on responding from pe to pe use another path ecept use disable-fallback at the end of command

 		sh mpls traffic-engineering tunnels accounting

 		int tun 0
 		load-interval 30
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mpls te cost calculation in igp
 	all configs are same like before on ospf
 	cost igp in ospf perlink is value 1

 	example 1
 		r1--------------r2
 		|				|
 		|				|
 		r3--------------r4
 		|
 		|
 		r5

 		*tunnel 0 used r135 

 		head end
 			ip explicit-path name r135
 			next-address 192.168.13.3/35.5

 			int tun 0
 			ip unnumbered loop0 
 			tun des 192.168.254.5
 			tun mode mpls traffic-engineering
 			tun mpls traffic-engineering band 1000
 			tun mpls traffic-engineering path-option 1 explicit name r135
 			tun mols traffic-engineering autoroute announce

 			sh mpls traffic-engineering tunnels

 			in this model scenarios if see 2 path on tail end for head end reachablity must use tunnel instead of igp

 			sh ip route 192.168.254.5 (say tunnel will be used)

 			int loop 1
 			ip address 5.5.5.5 255.255.255.255
 			ip ospf 1 area 0

 			if change our tunnel destination to 192.168.254.3 our r4 address and .. will be discovered behind tail end
 			but some routes acccessiabilies are on igp and tunnels maybe use same metric here can use load alancing
 			also if we are using 2 tunnel on r3 could use loadbalancing
 			in tunnels we have path metric that is equal bby lowest metric on igp
 			we used shortest path

 			totaly we have less priority on igp except our way or path be on behind of main path in this situation could use loadsharing 

 	example 2
 		r1--------------r2
 		|				|
 		|				|
 		r3--------------r4
 		|
 		|
 		r5

 		*tunnel 0 used r124 
 		we use this path on explicit-path all configs are same but have littel changes
 		here r1 can use 2 path for r4 destination reachablity
 		here we see cost 2 on tunnel and cost 3 on igp
 		tunnel count shortest path of igp
 		here we have problem 
 			if need to access r5 from r1 should select which of them r3 or r4
 			one use cost 3 and indirectly access
 			another use cost 2 and directly acccess 			


 	example 3
 		r1--------------r2
 		|				|
 		|				|
 		r3--------------r4
 		|
 		|
 		r5

 		*tunnel 0 used r134 
 		r1 make tunnel to r4 and use path like abpve but see r4 in 2 way 


 	example 4
 		r1--------------r2
 		|				|
 		|				|
 		r3--------------r4
 		|
 		|
 		r5

 		*tunnel 0 used r1243 
 		explicit path is same like this
 		make tunnel from r1 to r3 use cost 3
 		cause r3 reachablity from r1 we have 2 way one is tun another is igp
 		in this situation doesn't use loadbalancing
 		tun 0 has more priority
 		if need loadbalance  must make a another tunnel on r1 to r3 then use loadbalance

 		int tun 0
 		tun mpls traffic-engineering load-share 1 (set ratio)

 		sh ip cef exact-route 192.168.254.1 192.168.254.4
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
adjust cost calculation
	in normal we use shortest path in igp for tunnels

	r1--------r2--------r3--------r4--------r5--------r6--------r7
	 |--------tun0-------|
	 |------------tun1-------------|
	 |------------------tun2----------------|
	 |------------------------tun3--------------------|

	 we have same config and concepts like before

	 r1
	 	ip explicit-path name r123
	 	next-address 192.168.12.2/23.3

	 	int tun0
	 	ip unnumbered loop0
	 	tun destination 192.168.254.3
	 	tun mode mpls traffic-engineering
	 	tun mpls traffic-engineering bandwidth 1000
	 	tun mpls traffic-engineering path-option 1 explicit name r123
	 	tun mpls traffic-engineering autoroute announce

	 	we can see path weight 2 

	 	in tun 0 and behinde it all routers like r7 can visible on tun 0 also in igp ospf are exist

	 	for tun 1 we have same condition
	 	with explicit path 12.2 23.3 34.4

	 	in last line of config 
	 		tun mpls traffic-engineering autoroute announce
	 		tun mpls traffic-engineering autoroute metric 1 (applicted in igp and loadsharing)

	 	sh mpls traffic-engineering tunnels (path weigh 3)

	 	here we use 34.4 access on igp cause use metric 3 and tunnel 3
	 	in normal use tunnel but for loop backs use igp  and ospf access 

	 	for tun2 and r5 we have explicit path 12.2/23.3/34.4/45.5 + autoroute
	 		tun mpls traffic-engineering autoroute metric absoloute 1 

	 		*all networks before tunnel or behinde the tail tunnel will be shown with same metric
	 			12.2/23.3/34.4 igp metric
	 			another networks shows with fixed metric

	 	for tun3 we se relative command
	 		tun mpls traffic-engineering autoroute metric relative -1

	 		*path weight 5
	 		just 192.168.56.0 will be on tunnel also ospf see 56.0 in network so use load sharing
	 		till r6 we have metric 5 but in calclation and relative -1 we change value to 4 so igp and tunnel are same 
	 		also see r5 loop0 on igp
	 		r7 and r6 are visible from tunnel
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
rsvp
	in qos define this tool
	int serv to reserve bandwidth use path message on application media and make reservation replied on resv message to source
	it's not recommended  in these situation if each router set rsvp all bandwidth get fuck
	difserve is perhop behavior and detect bandwidth reservation on live requests

	rsvp te is an extention use label to make reservation on mpls

	tunnel requirement
		bandwidth
		setup priority
		hold priority
		affinity

		*head end use igp attributes to calculate these items

	headend didn't behave like befor and normal mode on ip 
	rsvp transfer all path with labels also reserver bandwidth

	ero or explicit route object say where should go and detect this on next address

	with rsvp make standby all path after recieve path message on tailend convert all standby to reserved mode also send reserved message to all nodes
	in this step use  pop label

	this mechanism happened till recieved by headend

	rsvp wors on the validity of the igp data 

	label 0 > explicit null > mpls qos (just pop value of labels and use eprimental als save exp bit)
	label 1 > implicit null php > ommit all label data and part
		also delete all qos ...label and data ...

	in mpls te just ommit label 0 with 3 just need ommit label 0 so 
		mpls traffic-engineering signaling interpret explicit-null verbatim
		mpls traffic-engineering signaling advertise implicit-null acl (here just use label3)

		sh ip rsvp reservation details

	we have cell mode behavior must happen dod (dial on dimand)

	in rsvp we have shared explicit style on reserved message
	in te we have make brefore break 

	path tear

	rro > record route object like ero
		in default is disable

		int tun0
		tun mpls traffic-engineering record-route

		*ero say which path is good and use it

		scenario is like this if use rectangel model project and our first model were triangle
		r123 were working 
		then r4 added after make rectangel model
		in this model need reoptimization 
		first off all make r4 ready to handel tunnel then change tunnel path from r123 to r124

		here we have se style
			in newest path and old path we have shared point or same points and don't like get trouble on bandwidth

			in se style say new path with bandwidth of tunnel get assigned
			if didn't assign gives to the one who has the most need then signals measured all path after this tear down last used tunnel 

			debug ip rsvp damp-message (must use in lowest load of router)

		rsvp message :
				path (on headend side to tailend side)
					error
						say te tunnel has problem
					tear 
						after path error or admin dicision happen

				reserve (on tailend side to headend side)
					error

					tear (rarely see this)
						means cancel reservation

	link manager
		in mpls engine we have link admission controll
		like rsvp say bandwidth assignment must use link manager
		link manager do all tasks

		house keeping
			keep track of bandwidth
			preemption (base on tunnel priority)
			trigger igp to flood ls information

		in te when we use ls and advertise or generate it

		ospf and  isis works on intecremental mode means each change on internal update  shold checked in osppf 30min and isis 15 min overall

		if r1 has 1m bandwidth and tun 0 on it use 500k of this must trigger a notification also link manager could use theshold to detect this

		threshold for tear down and turning on could change and generated

		each 3 minutes we have threshold checked and flood ls

			mpls traffic-engineering link-manager timers periodic-flood 192

			*up means we have more bandwidth
			*down means less bandwidth

			sh mpls traffic-engineering link-manager bandwidth-allocate

			15 30 45 60 75 80 85 90 95 96 97 98 99 100 (interval of notifications)

			int tun0
			tun mpls traffic-engineering flooding threshold up 80 90 100 (just on these percents notify)

			flood by igp
				la chang
				config change
				periodic flood
					ospf 30
					isis 15
					te 3
				change reserved bandwidth
				after tunnel setup fail

				router osdp 
				tears pacing lsa-group 15

				router isis
				lsp refresh interval 15

	show
		sh mpls traffic-engineering link-manage admission-control fast 0/0 (show us house keeping)
		sh mpls traffic-engineering link-manage advertise ment (information)
		sh mpls traffic-engineering link-manage bandwidth-allocate
		sh mpls traffic-engineering link-manage igp-neighbor (te sets on which igp neighbor altho turn off te show us agian but no lsa advertise)
		sh mpls traffic-engineering link-manage interface
		sh mpls traffic-engineering link-manage statictis
		sh mpls traffic-engineering link-manage summary

	debug
		debug mpls traffic-engineering link-manage admission-controll 1-99 1300-2699 details
		debug mpls traffic-engineering link-manage  advertise
		debug mpls traffic-engineering link-manage bandwidth-allocate
		debug mpls traffic-engineering link-manage error\events\link\preemption\routing
		debug mpls traffic-engineering link-manage igp-neighbor
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
frr (mpls fast reroute)
	if one link get trouble what happend
	headend send path error to network and use igp to find newest path 
	igp rsvp try find new path
	need time to make newe path more than igp recalculation
	in isp we have this time challenge

	in tunnels we have reroute mechanism that setup and signal handle big part of control plane

	for main link of isp that is transit part of many tunnels must set backup link
	means wee config rsvp and ... in full mode to make redundancy

	nexthop bbypass tunnel (nhop)
	ppoint of label repair (plr)(headend)
	merge point (mp)(tailend)

	we can make layer1 connectivity to make them fast forward
		automatic protection switching (aps)

		in miliseconds get replace

		it's not recommended and isn't applicated for links cause for each line must set backup

	fast reroute has node and link protection means we have backup link on another router
	no reserve bandwidth just works on explicit mode
	or make exclude a currepted router

	protect works on local router (plr)

	on headend we manage that on trouble use backuplink if physical get fuck

	if our path get change must change labels

	r1-------X-------r3
	\ 				/
	 \ 			   /
	  \ 		  /
	   \	r2	 /

	  in this situation we use bakcup link on r1 
	  r2 i backup here we have problem that r2 recieve data and transfer t ro r3 with diffrent label
	  here r3 use pop lable and send to r2
	  r2 pop the labels send data to r3 and r3 doesn't know what should to do
	  here r1 could do something send many labels to r2 like label r2 and cached label of r3 that got break
	  r2 pop itself label then use r3 label to manage transmissions

	  label spacing
	  	per platform > framed mode (mpls vpn and te works on it)
	  		recieve on every interface
	  	per interface > cell mode
	  		just recieve on specify interface

	  does work our backup link on optimize path?
	  	our headend use this temporary not permanent search for stable and best path 
	  	and make befre break after these

	another name frr is nhop

	if headend say to tail end on protected mode we have path error what happend ?
		instead of path tear down use signalling and setup new tunnel

	temporary send 2 label to backup router after find best router and path make tempoary link tear down then decide if temp path get main pth send one label instead of 2 label

	we can detect faster than befor if use bfd and rsvp hello
		int tun0
		tun mpls traffic-engineering fast-reroute (here if headend see path error search for newest path)

		ip explicit-path name fast-123
		next-address 192.168.12.2/23.3

		int tun 100
		ip unnumbered loop0
		tun des 192.168.254.3
		tun mode mpls traffic-engineering 
		tun mpls traffic-engineering path-option 1 explicit name fast-123

		* aggregation of tunnels and bandwidth must be equal physical interface bandwidth

		an interface we are worried about being cut off 
			int fast 0/0
			mpls traffic-engineering backup-path tunnel 100 (just do this not more or another mpls te configs)

			sh mpls traffic-engineering fast-reroute database
			sh mpls traffic-engineering tunnels backup

	frr node protection
		we protect a lsr
		we have no nethop label

		we use next nethop
	
	r4---------r1------r6-------r3-----r5
				\ 				/
				 \ 			   /
				  \ 		  /
				   \	r2	 /		

				   r6 is protected node
				   r1 is plr
				   r3 is mp

	r6 has nexthop labelreached from r3
	r1 just use r6 label not more to transfer labels and data to r2 in this state we cann't launch backup so just use label recording soloution

	in setup tunnel use r3 > r2 path in not stable sate if we have stablity must use normal path like r4 existed on reserved path and recorded and will be find and visible on r1
	r3 has something in databse and recieved by r6 
	r3 didn't send label to r1 directly

	head ned or r4
		int tun 0
		tun mpls traffic-engineering fast-reroute node protection

			*by path learning advertise another paths
			session attribute 0x10

	must set some attributes like before on r1

	sh mpls traffic-engineering tunnels backup
	sh ip rsvp fast-reroute
	sh mpls traffic-engineering tunnels tun100 protection

	multiple backuplink
		nnhop
				   /    r7   \
 				  / 		  \
				 / 			   \
			    /  			    \
	r4---------r1------r6-------r3-----r5
				\ 				/
				 \ 			   /
				  \ 		  /
				   \	r2	 /

				   r6 is prtected node
				   r1 > plr
				   r3 > mp	

	if r6 get trouble our bakcups could use loadshare 
	or r7 after r6 get fuck could manage links ?

	r1 config like before but use 2 tunnel and 2 explicit path
		ip explicit-path name fast123
		next-address 192.168.12.2/23.3

		ip explicit-path name fast173
		next-address 192.168.17.7/37.3

		set tunnel 100 on fast123
		and tunnel 101 on fast173

		backup path set on 100 and 101

		on plr if has many backup and works on protected mode make load share
		means not next nexthop

		between nexthop tunnel and nextnethop tunnel our nnhp has most priority
		protect one node and save all nodes that connected to this

				   /    r7   \
 				  / 		  \
				 / 			   \
			    /  			    \
	r4---------r1------r6-------r3-----r5
				\ 		|		/
				 \ 		|	   /
				  \ 	|	  /
				   \	r2	 /

				   r6 is prtected node
				   r1 > plr
				   r3 > mp	

			launch tunnel 100 and 101 with 2 diffrent explicit-path 

			ip explicit-path name fast126
			next-address 192.168.12.2/26.2

			on r4 use node protected for 2 tunnels on r5 destination
			altho didn't set node protection we use nnhop

			if bakcup tunnel get shut use next hop

		srlg or shared risk link group
			when we have link and protected tunnel
			protection will be apply with backup

						   /    r7   \
		 				  / 		  \
						srlg1 		   \
					    /  			    \
			r4--------r1-srlg1---r6---r3-----r5
						\ 				/
						srlg2 		   /
						  \ 		  /
						   \	r2	 /
		
						   r6 is prtected node
						   r1 > plr
						   r3 > mp	

			if r4 make tunnel to r5 from r163 on physical media but trouble on fiber optic links or .. maybe happen
			srlg detect risks
			we have 2 model of implemetation
				add tunnels in shared group
				another is The probability of occurrence of this problem which is used if it is healthy will change if it has a disorder

			r1
				int range fast 0/0
				mpls traffic-engineering	srlg1

				int fast 1/0
				mpls traffic-engineering srlg2 (number is 2^32)

			just work with autotunnel and semidynamic

			int tun 100
			ip unnumbered loop0
			tunnel destination 192.168.254.3
			tun mod mpls traffic-engineering
			tun mpls traffic-engineering path-option 1 explicit name r1tor3

			ip explicit-path r1tor3 
			exclude-addres 192.168.254.6

			here bypass r6 and change path if use	srlg our path used r7 way not another ways

			must enable	srlg in global in 2 mode 
				force > just use owned	srlg
				prefered > if see diffrent	srlg use don't use same

				mpls traffic-engineering auto-tunnel backup
				mpls traffic-engineering auto-tunnel backup	srlg exclude prefered

				sh mpls traffic-engineering topology srlg
				sh mpls traffic-engineering topology brief

				sh ip explicit-path
					afte autotunnel generate some tunnels and set some explicit path to complete the conditions we want

				sh mpls traffic-engineering tunnels tunnel 6537

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
backup and promotion bandwidth

				   /    r2   \
 				  / 		  \
				 / 			   \
			    /  			    \
	r4---------r1------r6-------r3-----r5
				\ 				/
				 \ 			   /
				  \ 		  /
				   \	r7	 /

				   tun 0 use 1m
				   tun 1 use 500k

	we use this concept in link and node
	in nnho destination doesn't matter

	on r123 we use 500k
	and on r173m use 1m
	we have backup if tun0 get tear down could use another path?
	no because we have 500k for bandwidth
	another path has more bandwidth must add them into the fast reroute to make scan and analyze them

	create 2 tunnel on r4 and r5 with node protection + autoorute + dynamic use main path like r163
	for r2 and r7 must set cost
	on r1 set 2 explicit path like r123 and r173
	set tunnel 100 and 101 with these explicit path
	also set destination on 192.168.254.3

	int tun 100
	mpls traffic-engineering backup-bandwidth 1000 (say to frr i have this)
	tun mpls traffic-engineering bandwidth 1000 (must set this bandwidth on interface)

	int tun 101
	mpls traffic-engineering backup-bandwidth 600
	tun mpls traffic-engineering bandwidth 600

	sh mpls traffic-engineering tunnels summary
		promotion has a backup and use a bandwidth if get chnage msu stay 300 seconds to rechecked 

	mpls traffic-engineering fast-reroute timers punt 30 (if set 0 get disable)
	mpls traffic-engineering fast-reroute promotion (now check the status)

	*reoptimization used for tunnels
	frr promotion used for backups

	when set limits on backup links must set limited bandwidth on 500k and 1m and priority tunnel need 500k use a tunnel that has the least waste

	for unlimited and limited have same behave
	main tunnel if doesn't have bandwidth , for backup use link with fewer lsp
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
path protection
	protect one path from headend to tailend
	not related to frr
	works on explicit
	orginally we setup 2 tunnel in ready state in this method we have overhead

	r4----------r1---------r3-----------r5
				 \        /
				  \ 	 /	
				   \ r2 /

				   r2 used for backup

	int tun 0
	ip unnumbered loop0
	tun destination 192.168254.5
	tun mode mpls traffic-engineering
	tun mpls traffic-engineering band 1000
	tun mpls traffic-engineering path-option 1 explicit name r4135
	tun mpls traffic-engineering autoroute announce

	ip explicit-path name r4135
	next-address 192.168.14.1/13.3/35.5

	int tun 0
	tun  mpls traffic-engineering protect 1 name r41235
	ip explicit-path name r41235
	next-address 192.168.14.1/12.2/23.3/35.5

	sh mpls traffic-engineering tunnels 
	sh mpls traffic-engineering tunnels protection
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
mls te and mpls vpn
	pe to pe tunnel

	on pe we create vrf for customers with rt and rd use ptp mode connection
	has loop0 with 192.168.254.x/32 also set isis on them
	we assigned ip to internal interfaces of isp oruters like 192.168.xy.w/24
	set mpls te on on them set rsvp bandwidth 
	enable mpls ldp autoconfig
	mpls label range
	enable level 2 on isis and mpls te
	set wide metric style
	and log adjencency

	create vpnv4 and bgp on r1 to r4 without ipv4
	redistribute all static routes on customers redistibution
	set static routes for each customer behind network
	set update source loop on r1 and r4
	on r2 and r3 set mpls (te and label + isis and ip addressing)
	change costs on path set them higher to change traffic flow
	if use label on te we don't need ldp at all on isp infrastructure

	first label is te label second is vrf vpn label

	must set tun on r1 to r4 also r4 to r1

	pe r1
		int tun 0
		ip unnumbered loop0
		tun destination 192.168.254.4
		tun mode mpls traffic-engineering
		tun mpls traffic-engineering bandwidth 1000
		tun mpls traffic-engineering path-option 1 explicit name r134

		ip explicit-path name r134
		next-address 192.168.13.3/34.4

		*must define another path to select it when get trouble

		*set configs like these for pe r4

		return path is msut config on 2way so on r1 and r4 should define it

		when can use fromm tunnel? autoroute , static route , pbr
			r1 > ip route 192.168.254.4 255.255.255.0 tun 0
			on r4 is same with .1 on tun 0

			all pe traffic transfer to another pe

	cuase ldp is disable on isp must set explicit path
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
vrf to pe tunnel
	set isis bgp mpls on isp
	this methid is not scaleable
	for each transmission must create one vrf and config them

	ldp is not so applicable for us in these type scanrios 

	we have same config like above scanrio

	r1 pe
		int tun 124
		ip unnumbered loop0
		tun destination 192.168.254.4
		tun mode mpls traffic-engineering 
		tun mpls traffic-engineering bandwidth 1000
		tun mpls traffic-engineering path-option 1 explicit name r124

		ip explicit-path name r124
		next-address 192.168.12.2/24.4

		create another tunnel 134 with explicit path 13.3/34.4

		here need forward traffic from vrf to pe and use explicit-path also need special tunnel 
			int loop 100
			ip address 192.168.254.100 255.255.255.255
			ip router isis
			int loop 101
			ip address 192.168.254.101 255.255.255.255
			ip router isis

			rd 1:1 > 192.168.254.7/32 > nexthop 192.168.254.4
			rd 1:2 > 192.168.254.8/32 > nexthop 192.168.254.4

			vrf define cus1
			address-family ipv4
			bgp next-hop loop 100 (every packet transfer to cus1 used nexthop on loop100)

			for each customer have seperated vrf

			better use a bullshit nexthop instead of real loopback ip nethop
			r1 use static route with 2 diffrent value on tunnels normally 

		int loop 90
		ip address 192.168.254.90 255.255.255.255
		ip router isis

		vrf define cus1-1
		address-family ipv4
		bgp next-hop loop 90

		ip route 192.168.254.100 255.255.255.255 tun 124
		ip route 192.168.254.101 255.255.255.255 tun 134

		set static route on loop ip address 192.168.254.90 in r4 to make reachablity

		clear bgp vpnv4 unicast * soft
		sh ip route vrf cus1-1
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
pe to pe tunnel
	we have bgp isis mpls vpn and labels in isp

	we have proble on p router isp
	on pe router we set destination on tunnel so must set and learn labels

	if p router recieve packet pop labels and on pe customer side need to see labels
	must add tunnel tail labels on packets

	make tunnel on pe to p routers on te infrastructure with explicit path

	labels :
		bgp 409 , ldp 900 , rsvp 200

	int tun 0
	ip unnumbered loop0
	tun destination 192.168.254.9
	tun mode mpls traffic-engineering
	tun mpls traffic-engineering bandwidth 1000
	tun mpls traffic-engineering path-option 1 explicit name r129

	ip explicit-path name r129
	next-address 192.168.12.2/29.9

	*make a runnel for r9 to r1 with 29.2/12.1

	ip route 192.168.254.4 255.255.255.255 tun 0

	cause r9 has no ldp so must set these to r1
		int tun 0
		mpls ip

		*must set ldp activation on each tunnel side

		remote ldp targeted
			doesn't need activate on each side 
			one ldp targeted between p router and pe

			pe
				ip route 192.168.254.4 255.255.255.255 tun 0 (same above config)

			on pe and p router
				mpls ldp neighbor 192.168.254.9 targeted

				*here add 2 label and just add mpls ip on int tun0 in r1 or pe router

				use 3 label for tasks

	controll plane resserved bandwidth
