QOS
	rtp (real time protocol) works on udp and codec g 711
	nbar (network base application recognization)
		base on nbar can define and detect traffic types then make acls (base on application signitures)

	tos (type of service)
		8 bit
			1 2 3 	4 5 6 7 8 
			* * * 	* * * * *
			used	unused

			ip precedence

	dscp (diffrentiated service code point)
		8 bit
			1 2 3 4 5 6 	7 8 
			* * * * * * 	* *
			used			unused
							ecn (explicit congestion notification)

		layer 3 header is the best layer for marking

	in 8 bit mode we have class selectors like 
		cs 0 >  0 (use for normal data)
		cs 1 >  8
		cs 2 >  16
		cs 3 >  24
		cs 4 >  32
		cs 5 >  40
		cs 6 >  48
		cs 7 >  56 (use for voice)

		also voice use dscp 46 or tos 46

		video conferences > af 4x
		stream video > af 3x
		low latency data > af 2x
		standard data > cs0

	qos tools use classifications with acls and nbars (marking use for layer 3 and tos headers use for layer 2)
	
	on cisco devices :
		policy map mark
		class data
		set dscp default
		class voip
		set dscp cs5

		class-map h
		match protocol http
		policy-map a 
		class h
		set dscp af 46

	if need more manipulation on data qos must use queue and tail drop data to make our queue faster (for drop just attention to drop part)
		dscp
			* * * 			* * *
			forwarder		drop

		ipp
			* *  			* * * *
			forwarder		drop

		per hub behavior

		000 00 0 > 0 (cs 0)
		001 00 0 > 8 (cs 1)
		001 01 0 > 	forward = 1, drop = 1 	10 dscp dscp with smaller drop value and have less priority for drop 
		001 10 0 >	forward = 1, drop = 2	12 dscp
		001 11 0 >	forward = 1, drop = 3	14 dscp with bigger drop value and have more priority for drop

		instead of use dscp use assured forwarding phb (afxy)

		cs 1 > af 11 , 12 , 13 (11 less drop value , 13 more drop value)

	dot1q :
		2byte tpid
		2byte tag control information
			contain cos in management
				priority code point

		ip phones set cs5 on data
		use trust binding :
			first hub can manage traffics to make them more compressed is our pcs or system 
			to avoid them to convert their traffics higher cs is on local system then send traffics to ip phones and ip phnes bind traffics to their cs

	queue :
		bandwidth 
		delay
		jitter
		loss

		the most slowness reason dependes on outgoing parameters

		have a system :
			classifier
			queues
			scheduler

				round robin methods
				priority and weighted round robin method
					cbwfg (class based weighted for queue)

		low latency queue (llq)
			voip use this
			a seperated queue with limitation to optimal usauge and prevent starving
			better use llq for videos

		cac > call advision controll
			check bw limitations and check traffics if be more than bw drop traffics

	shape and policy :
		in edge we set this
		if be more than traffic rate must discard traffics
		policy rate = 1 mbps >= discards
			this 1mbps is cir (commited information rate)

		policy check traffic rate thn drop\discard
		deploy on inbound isps shape deploy on outbound isps

		isps remark traffics some times we set af41 to traffics then isps get full and busy so convert it to af 43 and queue it or discard if not busy trasfer it by af 41

		on client side must shaped traffics if have 1g must convert to 200mbps and make queue on

		must set burst tim e interval on minimum values

		congestion avoidance is a mechanism use windowing system or flow control
			before queue get full drop one packet to make window size 1/2 and then trasfer

			random arly detection (tcp)
			weighted red (udp + tcp)
				if get drop our queue get worse


Example :
	e.g. (note syntax will be off)
	ip access-list extended user1
	permit ip host x.x.x.x any
	ip access-list extended user2	
	permit ip host x.x.x.x any
	class-map user1
	match access-list user1
	class-map user2
	match access-list user1
	policy-map traffic-control
	class user1
	police 8000
	class user2
	police 10000
	serial x
	service policy traffic-control out
