Service Provider QOS
****************************
general speaking
	bandwidth
		per seconds our transmission bits are 120 means 120kbps is bandwidth
		*it's different with link rate or access or clock rate

			a link with 100mbps capacity wanna send 1gpbs or 0/01 seconds send intervals

			normally on physical interfaces must be higher than bandwidth or be equal

			interface serial 0/0/0
			clock-rate 64000
			no shut

			show contollers serial 0/0/0

			*usually determine by isp and cir
			*eigrp and ospf use link state and access raate as main process parameters so better force use the bandwidth instead of acccess-rate

			*recommended set same values on bandwidth and access-rate 

	jitter
		differences between each delay

		delay variation (differences on delay)

		real time protocol (rtp)
			udp and forward directly to ip 

		each tool make our delay smaller make jitter too

	delay
		trip time from source to destination 
		our total delay named latency

		voice normal delay > till 150 msec
			on cisco till 200 msec

		serialization delay (sd)
			router has serial interface which information will be convert to signals this converting has delay cause calculation 

			(1500 byte * 8 bit) / 64000 bit (64kb) > 187.5 msec
			bit count divided to link speed

			is not appropriate for voice 

			must use codec for voice and manage them (convert to smaller packets cause wan links are limited)

			(60*8) / 64000 > 7.5 msec

			clock-rate 		sd 155 byte 	sd 1500 byte 
			---------------------------------------------
			100mbps 		0.0.1 			0.12 		msec
			1.5mbps 		0.65  			8 	 		msec
			512kbps 		2 				24	 		msec
			128kbps 		7.8 			93			msec
			56kbps	 		17.85 			214 		msec

			serialization delay depends on line speed and pacekt size
			sd doesn't change 
			sd use codec

			higher 1 mbps wan link is okay and sd is not challenge

		propagation delay (pd)
			publishing
			distance on meter measure / speed of publication in line

			is so small delay
			qos has no effect on pd and sd

		queueing delay (qd)
			always sense them
			by default is fifo

			useful in qos

			q1 > high priority
			q2 > fifo

		processing or forwarding delay (fd)
			completely recieved and reassembling data on peer
			wire speed forward
			always we have store and forward
			on switches we have fragment free and cut through 

			we have no forwarding delay on calculation

		shaping delay (shd)
			our access rate is on 128k
			cir is on 64k
			so better use shape and remained packets will be queue 
			on another time which has free line on bandwidth forward queued data

			our priority is voice data this voice never goes to queue just normal traffic will be goes to queue

		network delay (nd)
			inside isp delay
			on cir and contract must set this

			on sla must check this

		sd+pd+fd+qd+shd+nd > maximum must be on 150 - 200 msec

		*minimum bandwidth and sd on this state give us real sd 

		coded delay (cd)
			convert voice to special formats
			specific sample rates

		compression delay
			usually sd get shorter on hardware process also effects on qd

		more bandwidth provide less sd and less qd

	packet lost
		on fcs detect faults or noises and drop them

		ber (bit error base)
			10 pow 9

		main reason is tail drop 
		our processed packets were too much (huge queue size and packet count)

		queue pool is maximum length of queue example our pool queue is 50 if recieved 51 packet get drop

		if use too large queue our delay get trouble and lower packet loss 

		offer use 2 queue one of them has lowest dealy and smaller another one is bigger and used for data

	compression
		a tool which help us make optimized bandwidth consumption
		same part on headers will be send 
		another parts will be send without headers 
		details are not important

		compression might have delay cause our cpu must process the tasks

		we can also use queue as tool
		create 2 queue on device
			data queue 750k
			voice queue 250k
			-----------------
			1m connection link

		call admission controll (cac)
			voice to voice
			if were more than queue space doesn't store on queue

			q2 > 250 k
				packet 1
				packet 2
				packet 3

				drop new pacekt as packet 4

	link fragment and interleaving (lfi)
		we have 2 things and put another things between them 
		if recieved voice data on link cuase we changed bandwidth and priority our voice packet will be store because our last packet were too large and must join on queue
		lfi make fragment on packets so our queue will be fixed 
****************************
random early detection (red)
	some tcp applications are resource intensive
	red will drop some tcp packet so our window size will get smaller on flow controll
	our gueue get agile

	ip > 20 byte
	udp > 8 byte
		even port numbers
		16384 - 32767

		controller use odd port numbers

	rtp > 12 byte
	voip payload > even port numbers
		g711 > 160 byte
		g729 a > 20 byte

	analog telephones don't understand codecs so on routers will be process it's opposit with voip phones

					bitrate payload (kps) 		size payload 20 msec (cisco default)
	----------------------------------------------------------------------------------
	g711 pcm 				64 							160 byte
	----------------------------------------------------------------------------------
	g726 udp pcm 			32 							80 byte
	----------------------------------------------------------------------------------
	g729 					8 							20 byte
	----------------------------------------------------------------------------------
	g723.1 acelp 			5.3 						20 byte
	----------------------------------------------------------------------------------

		pcm stands for puls code mo

	digital signaling processor
		per miliseconds make samples

	codec delay + packet serialization delay + dejitter buffer delay
	----------------------------------------   ---------------------
	same time 30 msec 							initial play out delay (omit jitter store some parts of packets)

	video traffic charactereristic 
		modes
			interactive
				2way

				no delay & jitter

			non interactive
				1 way

		h323/h225 > tcp 1720
		h323/h245 > tcp 11xxx
		h323/h225ras > tcp 1719
		rtsp > tcp/udp 554
		real time transport > udp 16384 - 32767 even port numbers
		mpeg 1 > 500 - 1500 kbps
		mpeg 2 > 1.5 - 10 mbps
		mpeg 4 > 28.8 - 400 kbps
		h261 > 100 - 400 kbps

		more effect and colors also more moving need powerful codec

		flow count 		voice 				video (one signal is video another is voice)
		--------------------------------------------------------------------------
		packet size 	static (codec) 		dynamic
		--------------------------------------------------------------------------
		packet rate 	constance 			dynamic
		--------------------------------------------------------------------------
		interactive 	mission critical 	non mission critical
						1-2 sec 			3-4 sec
		--------------------------------------------------------------------------
		non interactive 

	nbar
		network base application recognition

		detect network traffic type		
		network auadit

		best effort > no qos, each device try send trffic as high priority

	intserv > integrated sevice 
		mpls
			rsvp + admission controll
			-------------------------
				per flow

	dscp > diffrentiated service code point
		computers must has trust mechanism  also has no reserved feature
		perhop behavior 

	admission controll
		routers outside will be do this 
		cops server 
****************************
tos, ipp, dscp
	perhop behavior cause low speed so our classification and marking (cnm) must get optimized
	on router must make classification and marking another routers just forward it by standard codes and marks ...base on marks all routers can determine and detect what should to do

	marking is on layer 3 and ip header in tos field

	tos is 8 bit
		0000 0000 > this is best effort and normal

	ip precedense (ipp)
		use 3 first bits of tos field

		000 > 0 (low priority)(routin)
		001 > 1 (priority)
		010 > 2 (immediate)
		011 > 3 (flash)
		100 > 4 (flash override)
		101 > 5 (critical)
		110 > 6 (inter-network controll like ospf ...)
		111 > 7 (high priority)(network controll)

		must set manually

		tos > 0 0 0  0 0 0 0  0
			  -----  -------  -
			  ipp 	 tos 	  mbz (must busy)
			  		  |
			  		  |
			  		  1 0 0 0 > minimum delay
			  		  0 1 0 0 > maximum through put
			  		  0 0 1 0 > minimum monetary cost
			  		  0 0 0 1 > normal service

	dscp
		tos > 0 0 0  0 0 0  0 0
			  ------------  ---
			  dscp 			tecn (explicit congestion notification)

		tos changed to dscp field

		we have 6 bit in dynamic states
		2 pow 6 means 64 modes

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

		--- 		--- 		--
		forward 	drop		ecn

		ecn has special applications with congestion avoidance target
		drop priority in congestion duration

		fd
		forward and drop
				
		xy (x is forward)(y is drop)
		af
		assued forwarding

		which one forward faster ? 
		which one drop faster ?

				forward priority 		drop priority 			af 		cs/precedense			decimal 
				***************************************************************************************
				000 (0)					00 				0 	 			cs(0) 	> 		000000		0 
				************************************************************	***********************
				cs1 	
				************************************************************	***********************
				001 (1)					00 				0 		af10 	cs(1) 	> 		001000		8
				------------------------------------------------------------	-----------------------
				001 (1) 				01  			0 		af11 			> 		001010		10
				------------------------------------------------------------	-----------------------
				001 (1) 				10 				0 		af12 			> 		001100 	 	12
				------------------------------------------------------------	-----------------------
				001 (1) 				11 				0 		af13 			> 		001110 		14 
				************************************************************	***********************
				cs2 	
				************************************************************	***********************
				010 					00 				0 		af20 	cs(2) 	> 		010000  	16
				------------------------------------------------------------	-----------------------
				010 					01 				0 		af21 			> 		010010 		18
				------------------------------------------------------------	-----------------------
				010 					10 				0 		af22 			> 		010100 		20
				------------------------------------------------------------	-----------------------
				010 					11 				0 		af23 			> 		010110 		22 
				************************************************************	***********************
				cs3 	
				************************************************************	***********************
				011 					00 				0 				cs(3)	> 		011000 		24
				------------------------------------------------------------	-----------------------
				011 					01 				0 	 	af31  			> 		011010		26
				------------------------------------------------------------	-----------------------
				011 					10 				0 		af32			> 		011100		28
				------------------------------------------------------------	-----------------------
				011 					11 				0 	 	af33 			> 		011010		30 
				************************************************************	***********************
				cs4 	
				************************************************************	***********************
				100 					00 				0 				cs(4)	> 		100000 		32
				------------------------------------------------------------	-----------------------
				100 					01 				0   	af41			> 		100010 		34
				------------------------------------------------------------	-----------------------
				100 					10 				0  		af42			> 		100100		36
				------------------------------------------------------------	-----------------------
				100 					11 				0 	 	af43  			> 		100110		38 
				************************************************************	***********************
				cs5 	
				************************************************************	***********************
				101 					00 				0 				cs(5)	> 		101000 		40
				------------------------------------------------------------	-----------------------
				101 					11 				0 		ef46 			> 		101110		46 
				************************************************************	***********************
				cs6 	
				************************************************************	***********************
				110 					00 				0 				cs(6)	> 		110000 		48 
				************************************************************	***********************
				cs7	
				************************************************************	***********************
				111 					00 				0 				cs(7)	> 		171000 		56
				------------------------------------------------------------	-----------------------

				*expedited forwarding (ef)
					fast convert and negotiate between 2 types liek dscp and precedense

				same in drop priority
					8 16 24 32 40 48 56
					10 18 26 34 
					12 20 28 36
					14 22 30 38

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

		must set same bits at 3 first bits as compatible dscp and ipp codes
		
					dscp 						ipp
		r1------------------------r2------------------------r3
				000000 00(d) 				000 00000(d)					
				000000 01(v) 				000 00001(v)

		if our switches were layer 2 and couldn't detect headers and mark what should to do?
			must inject in layer 2 tags and mark

			isl header > 26 byte
				frame type > 4 type
				isl user field > 1 byte
				cos > 3 byte

			original frame

		802.1q or p header
			destination
			source
			ethertype
			tag
				user field priority > cos 3 bit >  normally ip phones set 101
				vlan id > 12 bit

			*must be trunk 
				tagged on 0 - 7 
				offers don't use 7 and 6

			cos reach the router mapped to dscp

	*frame relay has policer help us more and higher than subscribtion, no gaurantee on using this method 
	if isp has no interferences you can send 10 instead 8 packets if has congest must drop you packets and recieved 8 packets
		de bit > discard elegibility

		clp (cell loss priority) bit in atm 
****************************
modular qos cli cisco (mgc)
	class map
	policy map 
	interface

	class base
		marking
		weighted fair queue
		policing
		shaping
		header compression

	class-map match-all cm-test
	!default is match all and means if match with all parameters on policy do somethin match any means if one of the parameters get match do something

	match access-group ad 1
	match source-address mac
	match destination-address mac
	match cos 2 3 4

	access-list 100 permit tcp any any eq 22
	access-list 101 permit tcp any any eq 23

	class-map t
	match access-group 100
	match dscp af31

	class-map s
	match access-group 101
	match dscp af21

	match ip precedense 2 3 
	!just ipv4 must check if not set detect on ipv4 and ipv6

	match protocol x
	!means nbar activvation

	show class-map 

	policy-map pm-test
	class cm-test
	set cos 0 1 2 3 4 5
	set precedense 
	set dscp

	show policy-map interface gig0/0 in class cm-test
	show policy-map interface gig0/0 out class cm-test

	interface gig0/0
	service-policy input pm-test

	*in normal state we have class map which works on match any on id=0

	interface gig0/0
	load-interval 30
	!works faster

	ping 8.8.8.8 -3 -p 443
	!generate traffic

	ip http secure-server
	!active on router

	for security issues recommended use ranges on clients ip and set default value on default class-map
	this mechanism help us if users use another dscp code on traffic omit them
		policy-map x
		set dscp default
		class class-default

		clear counters gig0/0

	match length is type of matching which help us set packet size and limit them
		match packet-length min 100 max 200
		!useful if set match-any and has no same conditions

	class-map match-any ipp123
	match ip precedense 3
	match ip precedense 2
	match ip precedense 1
	----------------------
	match ip precedense 1 2 3 
	!recommended use this model for effective match-any

	class-map match-any ipp123 
	match ip precedense 1
	max packet-length min 200 max 300

	access-list 100 permit ip host 2.2.2.2 host 4.4.4.4

	class-map a
	match access-group 100
	match ip precedense 5

	class-map b
	match access-group 100
	match ip dscp af21

	class-map match-any ab
	match class-map a
	match class-map b
	!nested class map
****************************
nbar
	if enable nbar on interface detect traffic on automated mode

	interface gig0/0
	ip nbar protocol-discovery

	show ip nbar protocol-discovery
	show ip nbar protocol-discovery top-n 2
	show ip nbar protocol-discovery stat packet-count
	show ip nbar protocol-discovery stat packet-count protocol

	class-map x
	match protocol telenet
	!in new generation use this model

	*doesn't work on source or destination port cause we might have random port no detection 
	or  rtp audio and rtp video use different protocols  so nbar could manage them on better state
	use 15% of cpu and inspect to layer7

	nbar has dependencies to cef
		sub port
		deep inspection
		url finder

		just support ipv4 and ipv6

	show ip nbar port-map telnet
	show ip nbar version
	show ip nbar resource

	ip nbar pdlm ftp 
	!update feature (packet description language module)
	!recommended use nbar on some intervals no always activation

	ip nbar custom http 8088 tcp 8088

	class-map test
	match protocol http url "*iran*"
	!tunnel and https and cef disble is not useful
	!cef switch traffics can works with nbar
****************************
qos pre classify
	in tunnels we have new header and old header get omit 
	in qos need old header so we have mechanism copy header to new headers

	ipheader + data (encrypted)
					-----------
					nbar couldn't work

	per classify make a copy from main data and detect data type then on new heaer inject them

	interface tunnel 0
	qos per-classify
****************************
congest management
	queueing
		ingress queue eans on interface we have process the last packet
		usually doest see on cef and routers or cpu

		on routers ram we have queue and sizing will be on pacekt or bytes without dedicated rate or capacity
		cause each interface without traffic make a queue on ram so wasted ram capacity
		we define a part as queue and force traffics use this specific part 
			use index and pointers 
			this is benefit use this model and count base no limit on size or capacity

		our qos help to make s space as processing target o egress 

		software queue and hardware queue
			our cpu might be on peak usage and our bandwidth get release so here we have trouble which our cpu did not send packt on line
			we have a harware mode pacekt frowarding in this situation
			in this situation our queue will be free and fill the line

			tx ring / tx queue is fifo mode and no control
			all procedure provide by software queue

			if hardware queue get free must fill as first queue then our software queue can be there for process on hardware queue

			here our hardware queue is free and our traffic be voice or video so need qos what should to do?
				get trouble
				recommended use software queue and change hardware queue size
				on ios change it automaticaly

				show interface gig0/0
				!show software

				show controller gig0/0 | inc tx-limited
				!show hardware

				interface gig0/0
				tx-ring-limit 100
				!1-32767 default is 256 in hardware if set qos doesn't effect and our fifo will be 0
				!wan links under 2mbps must be on wfq (weighted fair queue)
				!under 1mbps use fifo

				interface gig0/0
				priority-group 1
				!use 64 instead use 2546 and instead 0 on fifo set 1 menas behavior like another algorithem

				fair-queue
				!make our 64 to 2
				!serial links are 2 by default
				!fifo has normal tail drop
****************************
states
	classification
	drop decision
	maximum number of queue
	maximum queue length
		interface gig0/0
		hold-queue 30 in
		!change queue size

	schedule inside queue
	schedule logic
		outside queue
	-----------------------

	priority queue (depricated)
		above ios 15 omit this feature

		in this model we have 4 queue
			1 > high
			2 > medium
			3 > normal
			4 > low

			*each line get check and forward packet, round robin behavior, base on delay set different priority

		distribute and seperate traffics on priority level inject on these queues
		if high priority qeueue or line 1 get fill another queues and lines get fill
		then our high line could be on tx-ring so another lines get starvation

		access-list 100 permit host 192.68.1.10 host 192.168.2.10

		prioty-list 1 protocol ssh high tcp
		prioty-list 1 protocol ssh high udp
		prioty-list 1 protocol ssh high list 100

		priority-list 1interface gig0/0 normal list 101

		with acl or object get match on normal
			priority-list 1 default high
			priority-list 1 queue-limit 20 40 60 80
			!high, med, normal, low

			interface g0/0
			priority-group 1

			show queue gig0/0
			show queue priority
			!show us queue-limit values changed

		our classification might be interface base and  tail drop with use 4 qeueu
		each queue has fifo method inside
		our schedule is high to low

		not useful on nodays networks

	custom queue (depricated)
		omited from ios 15 above
		sueful on bandwidth
		16 custom queues

		each queue has byte count
		each queue get fill must use round robin method
		each queue fill goes to next

		has no starvation
		has no priority for voice traffics
		not so accurate
			our 900 k and 1500 k reqeust will be on same queue

		q1 > 1000 b > 10%
		q2 > 2000 b > 20%
		q3 > 3000 b > 30%
		q4 > 4000 b > 40%

		till queue counter did not fill did not change the queue

		queue-list 1 queue1 limit 1000
		queue-list 1 queue1 byte-count 1000
		queue-list 2 queue1 byte-count 2000

		queue-list 1 protocol ip 1 list 100
		queue-list 1 protocol ip 2 list 101
		queue-list 1 protocol ip 3 list 102
		!ip 1 means queue-list 1

		queue-list 1 default 1
		!by default use which queue

		interface gig0/0
		custom-queue-list 1
		show queue custom

		like befor has same attributes
		queue length is 80 but changeable

	mdrr (modified deficit round robin)
		custom queue get better performance
		but works on gsr 12k (internet routers)

		use now days
		change priorityand accurate

		bit count > quantum value (qv)
			q1 > 1500 qv 33.3% 
			q2 > 3000 qv 66.6% 

			each rr send 4500 

			packets on queue has 1000 byte 
				q1 > 1500 - 1000 > 500 
					-------------------
							p1

					1500 - 1000 > 500
					------------------
						p2

					500+500 > 100 deficit

					p3
						1000 - 1000 > 0 (last part use too much here use them) in rr

				q2
					3000 - 3000 > 0 (pc 1 2 3)

		has 6 queeu one of them has priority
		0 - 5
		0 1 2 3 4 5 p
			if p get fill recieved services and has starvation

			strict priority mode

		first of all each queue must service on qv seted 
		then goes to queue priority

	weighted fair queue (wfq)
		under 2 mbps use wfq
		above 2 mbps use fifo

		has minimal config and default state
		for ip precedense must consider space on wfq
		for each level need space
		bigger ipp means priority will be higher for recieved volume o space

		volume or space is capacity of pacekts with higher priority
		made smaller volume for sensetive packets

		first of all works on trafffic flow automaticaly
			flow
				source to destination (bytes, tos, peers, ip )

		sequence number or fresh time 
			on interface detect many flows
			before adding to egress queue router set sequence number on it 
			base this sequence number make them forward

		*base on ipp and volume set sequence number

		older sequence number + (weight * new packet length)

		weight > 32384 / (ipp+1) 

		ipp 	weight
		0 		32384
		1 		16192
		2 		10794
		3 		8096
		4 		6476
		5 		5397
		6 		4626
		7 		4048

		can make 4096 quue on flow 
		normaly use 256

	schedule on queue
		modified tail drop
			many queue which has more pacekt than qlength
			make drop in this condition
			here set size called congest discard threshold (cdt)
			each queue check this on seperated process
			if whole queue has volume or space but cdt is full, check another sequence numbers of other cdts
			if your queue sequence number were smaller than others replaced with another sequence numbers cdt
			cuase we need forward

			use 75% of bandwidth but after this use 92%

		dynamic queue and reserved queue
			use 8  queue as routing and sensetive

			show queue fair
****************************
class base wfq (weighted fair queue)
	ranking 
		nbar
		class base marking

	drop decision 
		wred
		tail drop as default

	schedule in queue
		fifo + wfq

	has 64 queue and each queue size 64

	llq set priority and without them can distribute bandwidth

	1 mbps link  can divided to 200k and set  gaurantee bandwidth for telenet

	class-map telnet-reserve
	match protocol telnet

	policy-map bandwidth-reserve-telnet
	class telnet-reserve
	bandwidth 200
	class class-default
	bandwidth 800
	!here seperate the bandwidth usage
	!----------------
	bandwidth percent 30
	fair queue

	*if bandwidth were free just assigned to telnet
	but busy times use 200 k at the least

	*here behave as wfq and fifo

	interface gig0/0
	service-policy out bandwidth-reserve-telnet

	class-map rtp-test
	match ip  rtp 16384 16383
	!from 16384 to 16383 collect even port numbers
	!----------------------------------------------
	class-map audio
	match protocol rtp audio

	class-map video
	match protocol rtp video

	class-map match-any meeting
	match protocol rtp payload-type 4
	match protocol rtp payload-type 34

	*before these days just use wfq on interface in 7500 cisco series
****************************
cbwfq + llq
	priority and bandwidth distribution

	class-map telnet-reserve
	priority 10
	!--------------
	priority percent 10

	class-map ping-reserve
	priority 30
	!--------------
	priority percent 30

	*here calculate telenet at the first then calculate  telnet

	llq has quota and above this rate get policing
	if violate them discard them and forward every thing else except llq

	class-map pc1
	match access-group 180

	policy-map a
	class pc1
	bandwidth 200

	*offer leave little free percents on bandwidth

	interface fif0/0
	service-policy out a

	*if had more than one llq must service them and just has 2 priority queue at the maximum

	if had one llq and other queues need get config a=on percentage
		policy-map a
		bandwidth remaining percent 5
****************************
congest avoidance
	decrease packet lost
		omit noise and tail drop

	window sizing is system feature

	congest windowing (cwnd)
		router could recieved 4k packets but on path saw 2k volumes
		here consider owest window sizing and congest windowing
		step by step increase pacekt size till get drop then set dedicated windows size 

		window size > reciever window / advertise window

		slow start threshold > is half of congest windowing
		congest avoidance will be calculate and between 2 above parameters will be selected 
		on udp our pacekt size are small

		slow start threshold congest avoidance > from half of congest windowing increase transmission size and leave free as little bytes to don't make flat

		 no acknowlege on tcp means loss
		 and after this set a sequnce on cw (congest windowing) then our sst (slow start threshold) be half of cw
		 again start increasing to cw 
		 after this increasing  with  sstcwnd make congest avoidance and prevent flat bandwidth

		 *somoe time queue is ful and our source tcp change window size so get troubl eon link and queue cuase one seconds is fast another is slow

		 solution is global synch with dropping some traffics with low priority
		 	random early detectcion (red)
		 	just works on tcp

		 tcp starvation
		 	if drop to much tcp packets our udp packets fill the line and doesn't access to tcp make connection
		 	another name is flow aware
		 		detect udp packets on flow

		 wred (weighted random early detectcion)
		 	on ipp like 0 or 1 omit or drop many packets on random mode

		 	h0
		 		full drop
		 			random drop
		 			------------
		 				maximum threshold > 30 (depends on queeu size or volume in this part)(lower ipp get drop)

		 				no drop
		 				--------
		 					minimum threshold > 5

		 red worked on average queue depth and traffic behavior which collect live capture with last 1 second capture then make average	on it
		 on this average drop 2 packets
		 predit traffic flow and prevent peaking or flatting

		 	new average = ( (old average * (1 - (2 pow -n) ) ) + (curent queue depth * (2 pow -n ) ) )

		 		n > exponential weighting constrant
		 			how does it effect on average this last or old state? 

		 		n = 9 is too much high 

		 ipp | minimum threshold | maximum threshold | mark proability dennominator (mpd) | calculated maximum percent discard
		 ----------------------------------------------------------------------------------------------------------------------
		 0 		20 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 1 		22 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 2 		24 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 3 		26 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 4 		28 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 5 		31 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 6 		33 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 7 		35 					40 					10 									10%
		 ----------------------------------------------------------------------------------------------------------------------
		 rsvp	37 					40 					10 									10%

		 *between minimum threshold and maximum threshold cisco drop ipp 0 or ipp 1 > inclined to ipp 0 dopping
		 so here on ipp 0 we have most drop state an will never pass best solution is mpd
		 	mpd help us if 10 packet were on ipp0 one of them get drop not all of them


								| minimum threshold | maximum threshold | mpd | calculated maximum percent discard
		 -----------------------------------------------------------------------------------------------------------
		 af 11, 21, 31, 41 		  33 					40 				  10 		10% 	
		 -----------------------------------------------------------------------------------------------------------
		 af 12, 22, 32, 42 		  28 					40 				  10 		10%
		 -----------------------------------------------------------------------------------------------------------
		 af 13, 23, 33, 43 		  24 					40 				  10 		10%
		 -----------------------------------------------------------------------------------------------------------
		 ef 				 	  37 					40 				  10 		10%

		 wred just consider drop not forward

	scenario

		acls
			a > af 13 pc1---- .51 --|
									|
			a > af 22 pc2---- .52 --|------------- sw ------------------ r1 ------------------ r2 ------------------ srv .50
									| 					192.168.1.0/24  	 192.168.12.0/30	    192.168.2.0/24
			a > af 31 pc3---- .53 --|

			r1
				access-list 101 permit ip  host 192.168.1.51 host 192.168.2.50
				access-list 102 permit ip  host 192.168.1.52 host 192.168.2.50
				access-list 103 permit ip  host 192.168.1.53 host 192.168.2.50

				class-map a
				match access-group 101

				class-map b
				match access-group 102

				class-map c
				match access-group 103

				policy-map mark
				class a
				set ip dscp af 13

				class b
				set ip dscp af 22

				class c
				set ip dscp af 31

				interface gig 0/0
				service-policy input mark
				random-detect
				!normally works on ipp
				--------------------------
				random-detect dscp-base

				*wred doesn't work in newest ios version on interfaces, doesn't have red on cisco if write red term on cisco points wred

				show queueing random-detect

				*normal mechanism on fifo and wred on it

				interface gig0/0
				random-detect dscp af13 35 40 30
				random-detect dscp 5 35 40 30
				!dscp group, min threshold, max threshold, mpd

				random-detect ipp 5 35 40 30

				random-detect exponential-weight-cons 5
				!same n on formula

				random-detect ecn

				*normally on active interfaces doesn't set this
					class-map af13
					match ip dscp af13

					class-map af22
					match ip dscp af22

					class-map af31
					match ip dscp af31

					policy-map bandwidth-red
					class af13
					bandwidth 100
					random-detect dscp-base

					class af22
					priority 200

					class af31
					bandwidth 500
					random-detect dscp-base
					!on this state just show only this af but setting on ipp show all conditions

					class class-default
					fair-queue

					**don't set wred on voip and udp

					ecn (explicit congestion notification)
						we can use ecn as mechanism to underestand the origin that send less pacekts instead oof use wred and drop mechanism on queue packets 

						ecn notify clients as congest state and force them send less than before

						ecn can detect if were not 00
						01 means you have congest

						has 2 bit
							ecn copable transport(ect) 		congest expriented (ce)
								0 							0 						> doesn't underestand
								0 							1 						> 
								1 							0 						> underestand ecn
								1 							1 						> tcp sender send less packets and apply wred

								*wred if detect full bandwidth or random drop scope use ce bit as congest avoidance mechanism

						we have this bits on ip header and tos or dscp field
						trouble is on tcpip v4 doesn't have these ecn bits

						tcp flags
							ece (explicit congestion exprienced) help us and set a tcp falg on tcp header  so change window size and less transmission

						usually ecen works on 00 and get trouble for wred 

						interface gig0/0
						----------------
						policy-map wred

						random-detect ecn
						bandwidth 800
****************************
shaping and policy
	if recieved traffic from clients check recieved range is on cir (commited information rate) scope or not

			64k
	r1 --------------- isp

	traffic |				 	*******
			|   		*********
			| 		***
			| 	****
			--------------------------
						time

	on peak level must make queue on traffic then on bandwidth free times send remained traffic 
	on shaping tried send traffic on smooth line
	on policing drop every thing
		opposite the shape cut the peak traffic

	some engineers say use shape on all conditions but policing has special target

	our links has higher transmission capability but use policing as droping mechanism to prevent more clients usage and comply cir

	on some states on one  second just has 0.5 transmit and another 0.5 second just wait

	policer
		effect on incoming traffic
		if had huge traffic on incoming interfaces must drop them
		more than cir must drop

		use token bucket on different way

		2 color
			confirm (transmit) , exceed (drop)

			single rate
				( (curent packet arrival - previous packet arrival time ) * policy rate ) / 8

				a container which has token values must used for tasks
				in policing we have byte 
				each toke has 1 byte
				each pacekt recieved check is it token size or not
				commited bucket (bc) is volume or size of container 

			2 color
				policy-map p1
				class class-default
				police 64000 confirm-action transmit exceed-action drop

				interface gig0/0
				service-policy input p1

				show policy-map
				show interface gig0/0

		3 color
			confirm (transmit) , exceed (remark means change dscp and ipp) , violation (drop)

			* higher rate cause remarking

			we have 2 container or bc if one of them get more bc put on next bc
			first bc is size and second is be (exceed)
			we have single rate and dual rate

			dual rate means we have 2 buckets and 2 rates
			we have 2 token generator one of them is cir base another is peak information rate (pir) base

			policer 
				cir bc > bit
				pir cbe > byte

				*confirm > decrease from cir and pir
				*exceed > just decrease from pir
				*violation > drop them and decrease nothing
					we have remark or mark down on 3 color

			on previous mode if were free give access to customers get exceed
			
			3 color single rate
				policy-map p2
				class class-default
				police 64000 4000 4000 confirm-action transmit exceed-action set-dscp-transmit af13 violation-action drop 
				!on each transmission has 4000 byte on bc, on each transmission has 4000 exceed rate
				!of don't set second 4000 rate and fix violation actioin detect automaticaly our numbers

				interface gig0/0
				service-policy input p2

				show policy-map interface gig0/0

			3 color dual rate
				policy-map p3
				class class-default
				police cir 96000 pir 128000

				interface gig0/0
				service-policy input p2

				show policy-map interface gig0/0

		bc > cir / 32 
		be > pir / 32

		dual rate 
			bc > cir / 32
			be > pir / 32

		single rate
			2 color 
				be > cir / 32

			3 color
				bc > cir / 32
				be > bc

	shaping just apply on outgoing traffic
		if you don't have delay or loss sensetive traffic don't use shaping and isp must policing them

		if just had traffic on loss sensetive mode recommended use shaping and isp by pass policing
		*we have no traffic which has no delay sensetivityand loss sensetivity

		egress bocking . our 4 way path converted to 2 way path (congest on outgoing)

		speed mismatch > our branches works on 1 mbps and office works on 3 mbps

		how does it work

			access  |	64k	
			rate	|------------ 0.5 second 	
			128k	|///////////|
					|///////////|
					--------------------------1 second
								time

			on some intervals has transmission so our voice has no quality


			access  |					   each 62.5 msec send traffic	 
			rate	|--	--	--	--	--	-- 	
			128k	|//	//	//	//	//	/|
					|//	//	//	//	//	/|
					--------------------------1 second
								time

			125 kbps * 8 kbps > 64 kbps
			 
			125 kbps / 2 (intervals sed and wait) > 62.5 is time that one cycle has transmission and one cycle wait

			62.5 msec * 8 (send intervals) >  500 msec like befor but our voice packet did not wait 500 msec just wait 62.5 msec and it is faster than before

			shape average
				64000 bps

				burst commited > 64000 bc cause had high delay

				burst commited > 8000 bc means send it 8 times and each time 8000 bps cause low delay and fast transmission

				bc > tocken bucket * cir

				bc > tocken bucket * shape rate

				token bucket
					a container which has token generator on each second
					10000 token mena saccess you send 10000 packet

					works on token bucket or bits

					imagine we use lower tokens on each portio, fill the container as it is 10000 token

					spillage waste

					if on time intervals our token buckets get empty and our links will be free so optimized model need 2 container and fill 10000 tokens between them
					limited storing

					our shape use these tokens on each interval and our bandwidth consumption will be optimized

		generic traffic shaper > wfq (old and not class base)

		class base shaping (mqc) > good role fifo, wfq, cbwfq, llq

		distributed traffic shaping (router 7500) > fifo, wfq, cbwfq, llq
		
		framerelay traffic shaping > fifo, wfq, cbwfq, llq, pq, cq (depricated)

		do we have packets in shape queue ? 
			if yes 
				put on one of the high low medium.. queues
				then process by software queue
				after this goes to hardware queue

			if no 
				use token bucket queue must goes to shape queue
				if shape queue were empty checks software queue
				then checks hardware queue 
				next another queues get check if were empty and free transfer on them

				traffic-shaper rate 64000 8000 8000 
				!rate in this state contains all traffics, 64000 is isp rate, 8000 is burst commited (bc), 8000 is be (bucket exceed)

				traffic-shaper group 1 64000 8000 8000
				!access-list number is grou 1

				*generic traffic shaper (gts) use whole interface or seperated traffic type by access-list

				access-list 1 permit host 192.68.1.2 host 192.168.2.3

				show traffic-shaper gig0/0
				show traffic-shaper statistics gig0/0

				ping -t -i 0 192.168.1.1 -w 0

		 	shape queue is on fifo 
		 	software queue is on wfq


		 	scenario
		 									    192.168.1.0/24       192.168.2.0/24
		 		192.168.1.100 - pc1 ----- sw1 ------------------ r1 ----------------- isp -------------- r3 --------- sw2 ----------- r4
		 														 101 																  301
		 														 102  																  302

		 		policy-map shape-all
		 		class class-default
		 		shape average 64000

		 		interface gig0/0
		 		service-policy out shape-all

		 		show queue gig0/0
		 		show policy-map

		 		*if bandwidth were under 320 kbps set commited burst (bc) on 8000, if were above 320 kbps set tc over  25 msec on old versions

		 		prioritise voip traffic and normal traffic
		 			ip cef
		 			!use nbar

		 			class-map match-all voip-rtp
		 			match ip rtp 16384 16383

		 			policy-map queue-voip
		 			class voip-rtp 
		 			priority 32
		 			!g729, 32 kbps

		 			class class-default
		 			fair-queue
		 			!another traffic will work on fair mode in queue process and no shape 

		 			policy-map shape-all
		 			class class-default 
		 			shape average 96000 960
		 			service-policy queue-voip
		 			!now make shape and queue

		 			interface gig 0/0
		 			service-policy out shape-all

		 			*recommended use tc as 10 msec for voip depends on cir
		 			on shape use class base wfq + llq and on interface need software queue
		 			inisde policy-map we set these doesn't need set software queue on policy-map

		2 buckets which are be and bc will charge on intervals, also transmission portion will be more
		depends on cir and some times get remark or mark down
			policy-map x
			shape peak 64000

			shape rate = (configured rate * ((1 + be) / bc ) ) > (64000 * ( ( 1 + 8000) / 8000 ) ) > 128

			show policy-map interface gig0/0

		on signals detect shape
			adaptive shape
			
			shape adaptive 300
			!rarely use this

			when isp get busy need increase bandwidth normally send 25% less than before so need more shape process

			mir stands for minimum information rate
				couldn't send under this rate and maximum rate that isp recieved on busy time

			(bc + be) / 16 
				if didn't recieved again, add this and rescue the bandwidth usage

			framerelay
				backward ecn (back route)(becn)
				forward ecn (fecn)

				framerelay switches on this bits say i have congest

		change fifo queue length
			shape max-buffer 50
			shape adaptive 30
			shape fecn-adapt
			!apply on outgoing way

		startacom foresight message bougth by cisco and produced framerelay switches then switches advertise this congest instead of set bit
****************************
link efficienty
	use more links capabilities
	on compression we don't send again same parts
	must be enable on source and destination
	has delay but effects o bandwidth consumption
	our serialization and queueing delay get decrease
	if use specific hardware for compression might help us on delay issue

	payload compression (mppc-stalker (cpu))(predictor (ram) is resource intensive)
	headers

	compression advance integration module (aim)

	3660
	3620
	7200
	7300
	7400
	7500

	ip > 20 byte
	tcp > 20 byte
	-------------
	total 1440

	dl
	payload compression
		headers compression
			ip 
			tcp

		payload
	dl

	ip and tcp will be 3-5 byte
	data 1400 byte
	------------------
	total 1405

	ratior 1.02

	if traffic were telnet and 40 byte our compression will be more
	if our data were normal recommended use header compression
	if were huge recommended use payload compression

	in crowded traffic places must consider delay and jitter

	tcp > tcp + ip
	rtp > udp + ip + rtp

	software compression is not good
	our forward delay get increase

	in classes better use compression for tcp and udp

	class-map match-all telnet-only
	match protocol telnet

	class-map match-all voice-only
	match protocol rtp audio
	

	policy-map test-comp
	class vioce-only
	priority 30
	compress header ip rtp

	class telnet-only
	bandwidth 300
	compress header ip tcp

	interface gig0/0
	service-policy out test-comp

	show policy-map interface gig0/0

	iphc > cisco compression on header
	ration > effeciency improvement factor
****************************
link fragmentation and inter leaving (lfi)
	sometimes we have trouble on class base model

	class-map voip
	priority 100
	!or 32

	class-map other
	bandwidth 900
	!or 64

	in some condition our voip queue is free and normal data line  or queue requested 1500 byte so our serialization will be too long 
	cisco offer 10-15 msec on serialization 
	after 1500 byte transmission device recieved voip traffic with 60 byte and need 7 msec on serialization
	in this time we are on data transmission time till get end must wait then voip will be forward
	we have delay here but above command set priority for voip
	what is the solution ?
		if recieved 1500 byte set 5 * 300 byte and make queue delay on 1/5 
		between each segment send voip and data without delay

		*cause use fragmentation must process them

		*if had high bandwidth don't need lfi

		under 768 kbps must use lfi

		don't support mqc

		detect important delay and calculate them, then set fragmentation
		tell required delay and calculate fragments

		useful on ppp or multiple ppp (aggregate many ppp)
		make fragmentation and send on many links

		unlike ether channel don't generate many pacekts and send over many links
		first collect all pacekts then send or distribute over links
		lfi is subset of mlp

		interface gig0/0
		bandwidth 128
		clock-rate 128000
		encapsulation ppp
		!normaly works on hdlc

		load-interval 30
		ppp multilink
		multilink-group 9

		interface multilink 9
		bandwidth 128
		ip address 192.168.1.2 255.255.255.0
		multilink-group 9
		ppp multilink
		ppp multilink fragment-delay 10
		ppp multilink interleave

		ip rtp priority 16384 16383 65
		!65kb

		show interface multilink 9

		*old llq config
****************************
ipv6 qos
	tos
		traffic class 8 bit
		flow label 20 bit

		if sense some traffics have same flow or attributes must set same flow label 
****************************
lan qos
	on 100 mbps we have sd, fd, qd > 0
	on higher bandwidth still have trouble

	recommended set priority on first device like switch ...

	2950 weighted rr
	3750 and 3560 (different qos) shaped rr 

	standard images  > less qos options
	enhanced images > more qos options

	features 				enhanced images (ei) 			standard images (si)
	--------------------------------------------------------------------------------
	classification 			l2-l4 							l2 only
	--------------------------------------------------------------------------------
	marking 				l2 and l3 						l2 only
	--------------------------------------------------------------------------------
	priority queue 			yes 							yes
	--------------------------------------------------------------------------------
	wrr 					yes 							yes
	--------------------------------------------------------------------------------
	policy 					yes 							no
	--------------------------------------------------------------------------------
	auto qos 				yes 							no
****************************
mapping tos and cos to dscp
	mls qos
	mls qos map cos-dscp 0 af13 af12 ...
	!here 0 menas cos0 af11 means cos 1
	!recommended use ef as cos5 and af31 as cs3

	show mls qos map cos-dscp

	*auto qos correct mapping , nowdays switches has correct and accurate qos and cos
	like 2950
		show mls qos map dscp-cos

	trust boundray
		first point entry and after this point managed by admin
		it is not clients border

		show mls qos interface gig0/0
		!trust state > not trust
		!trust mode > not trust

		*even if clients set ef on packets we change to cs0

		interface gig/0
		mls qos trust cos
		mls  qos trust cos passing-through
		!trust to cos and don't mark dscp
		!on trunk links we have cos coul be converted over dscp

		interface gig0/0
		mls qos trust device cisco-phone
		!if were cisco phones detect by dscp and if were not cisco ip phones tell us it is not trusted
		!find on above show commands

		interface gig0/0
		switchport priority extend cos 0
		!if ip phones recieved packets from pc set this cos on traffic

		switchport priority extend trust
		!each mark on pc will be accepted by ip phone

		mls qos cos 5
		!each mark from pc convert to cs5

		mls qos cos override
		!every thing will be replaced, above command has priority and must set a cos level then set this command except these has no effect

		scenario

				pc1 ---- ip phone1 ---------|
											|sw1 (2950) --------- r1
						pc2-----------------|
											|
									r2------|

			sw1
				interface gig0/0
				switchport mode trunk
				mls qos trust dscp
				!router side will be trunk and trust

				interface gig0/1
				switchport mode trunk
				mls qos trust cos

				interface gig0/2
				switchport voice vlan 100
				mls qos trust device cisco-phone
				!automaticaly set ef and signals on af31 and cos 3
			-----------------------------------------------------------

				pc1 ---- ip phone1 ---------|
											|sw1 (2950) --------- r1
						pc2 ----------------|
											|
									srv ----|
											|
					ip phone2 ------l2------|

			sw1
				access-list 1 permit 10.2.1.1
				class-map video 
				match access-group 1
				policy-map video
				class video
				set ip dscp 4

				interface gig0/0
				service-policy input video
				!if recieved mls qos cos 4 on interface map on sdcp 4

				each interface has 1 queue and4 queue at egress base on maac forwarding

				for congest management > strikt priority, wrr, srikt and wrr

				strikt priority is like priority queues with high mid low nom queues, might has starvation

				wrr
					wrr-queue cos-map 1 0 1 2
					wrr-queue cos-map 2 3 4 5
					wrr-queue cos-map 3 6
					wrr-queue cos-map 4 7
					!map cos types on q1 q2 q3 q4			

					wrr just distribute cq
					has 4 queues without priority just redistribute bandwidth
					weight is packets which must be forwarded
					has no starvation

					wrr-queue cos-map 1 0 1 2
					!strikt 

					wrr-queue bandwidth 250 350 450 550
					!wrr q1 q2 q3 q4

					show wrr-queue cos-map
					show wrr-queeu bandwidth

				strikt + wrr
					like class base queue and llq

					wrr-queue cos-map 1 0 1 2
					!strikt 

					wrr-queue bandwidth 250 350 450 0
					!q4 0 means new type and priority if set other 0 means real 0 bandwidth

					show wrr-queue cos-map
					show wrr-queeu bandwidth

			2950 policing
				in profile / confirm
				out of profile or non-confirm exceed

				access-list 1 permit 10.2.1.2
				class-map ftp 
				match access-group 1
				policy-map police 
				class ftp
				police 50000 8000 exceed-action drop

				interface gig0/0
				service-policy input police
		

			3560 police
				has no qos bydefault

				mls qos
				!active qos feature or option
				!use asiic

				show mls qos

				*ingress qos config > mapping
				cos 3 bit
				if had cosframe convert to dscp

				if wre untrust omit cos  > cos and dscp > 0

				trunk and trus means make mapping on ingress

				ingress queue (map table)
					1 > other 
					2 > voice

				classification
					qos by interface (port base)
					qos by acl and class

				on access ports and l3 use dscp

				on traffic use cos (convert tos to dscp) and dscp

				lan qos
					signal on cos 3 and payload cos 5 with native vlan
					map to  every where every thing and every thing if were trust

				policing
					shaped rr (round robin) replaced by weighted rr
						queue input
							egress has shared base on cos (limit on minimum)

						queue output
							shapped limit maximum and minimum

					shared rr will be dedicated to many queueif were freee distribute them

				show mls qos maps-cos-input-q
				show mls qos maps-dscp-input-q

				q1 > dscp 0 - 42 , 48 - 63 | other cos 
				q2 > dscp 46 | ef or cos 5

				show mls qos input-queue
				!same bandwidth threshold

				buffer 90% on q1 other traffics and 10% on q2 ef or voip traffic
				priority 0 for q1 and q2 and 90% dedicated traffic by mix mode of normal traffic and voip 
				dedicated priority 10 for distributes voip

				shared rr
					mls qos srr-queue input cos-map queue 1 0 1
					!1 is queue 1 manage cos 0 and 1

					mls qos srr-queue input cos-map queue 2 2 3 4 5 6 7
					!2 is queue 2 and manage cos 2 - 7

					*must set cos and dscp on same to make mapping

					show mls qos map cos-input-q

					mls qos srr-queue input buffer 67 33
					mls qos srr-queue input bandwidth 90 10
					mls qos srr-queue input priority-queu 2 bandwidth 20
					!queue 2 on bandwidth 20 if set 0 just redistribute 

					mls qos srr-queue input cos-map queue1 threshold 2 1
					!2 is level and 1 is cos

					mls qos srr-queue input cos-map queue 1 threshold 3 0
					mls qos srr-queue input cos-map queue 2 threshold 1 2
					mls qos srr-queue input cos-map queue 2 threshold 2 4 6 7
					mls qos srr-queue input cos-map queue 2 threshold 3 3 5

					*usually works on 100 and threshold 3 no changeable
					for dscp use same
						mls qos srr-queue input dscp-map queue2 threshold 1 2

					mls qos srr-queue input threshold 1 8 16
					mls qos srr-queue input threshold 2 34 66
					!from 8 percent to 16 percent

					egress queue
						q1  
						q2
						q3
						q4

						*each one has 3 threshold 3rd threshold is not changeable

						same commands at above with out-queue
							mls qos srr-queue output cos-map queue 1 threshold 3 0
							mls qos srr-queue output cos-map queue 2 threshold 1 2
							mls qos srr-queue output cos-map queue 2 threshold 2 4 6 7
							mls qos srr-queue output cos-map queue 2 threshold 3 3 5

							mls qos srr-queue output dscp-map queue 1 threshold 3 0
							mls qos srr-queue output dscp-map queue 2 threshold 1 2
							mls qos srr-queue output dscp-map queue 2 threshold 2 4 6 7
							mls qos srr-queue output dscp-map queue 2 threshold 3 3 5

						*each interface has 2 queue set one template is forwarding naother is config
							mls qos queeu-set out 1 buffers 10 20 30 40
							!q1 q2 q3 q4

							mls qos queeu-set out 1 threshold 50 60 100

							interface gig0/0
							queue-set 1

							*queue 1 might be priority queue and srr make them serviceif has no packets check another packets

							interface gig0/0
							srr-queue bandwidth limit 500
							!how much is our bandwidth

							srr-queue bandwidth shape 10 20 30 40
							srr-queue bandwidth share 10 20 30 40

							*shape with empty queue will be distribute

							priority queue out
							!till queue has items must forward them on queue 1
							!if don't set this our forwarding will be rr if set queue 1 will be on higher priority

							show mls qos queue-set 1
							show mls qos interface gig0/0 queue
****************************
auto qos
	cisco best practice
		autoqos
			voip
				voip
				video

				auto qos voip trust
				!set trust on interfaces which are on  higher designed layers 
				!on router and switches
				!if don't use trust command here make map and convert

			enterprise
				voip 
				video
				wan

				just on routers
					auto discovery qos

			voip interfaces
				auto qos voip cisco-phone

			after these commands on priority queues
				voice and video control traffic
				realtime video traffic 
				voice traffic
				routing protocol
				spanning tree bpdu traffic

			*all tasks and commands will be automated
			don't set auto qos on access links

			*has remark on non trust auto qos voip

			show auto qos

			*must set bandwidth on corrected values

			* snmp trap voip must be enable 
