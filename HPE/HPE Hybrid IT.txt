HPE Hybrid IT
	hp :
		nic > home product
		enterprise > datacenter

	hpe :
		proliant
			4 type :
				stand alone (sl)
				maximum expanion (ml)
				blade (bl)
				density line (dl)
					need dc cooling
					dl 355 > ryzen
					dl 350 > intel

					dl 580 g9 is the bestseller

				microserver (1 cpu socket and define after g8 like desktop)
				easy connect (like micro server, many m2 expansion, no cooling dc)
		blade
			compact series
			16 serve and 8 switches

		appolo
			use many hdd
			40k and 45k
			hp search the company bought devices 
			use cloud centers

		sgi
		integrity non stop
		integrity (superdom)
		carrie grade

	we have onboard administration (oa) > a board like modular option can mirror left and right  + whole server management like ilo has frameware
		active ip 1 + ip 2
		or we can set ip 1 to active passive

		we can add some modules on blades :
			ethernet sw
			san sw
			fex
			passthrough

		down link is a link connection between server to switch
	fiber channel director (modular)
	fiber channel switch

	store once :
		works like hdd speed and behave like tape

	bay is oa box that hdd and dl can connect to eac other

	sl is a version of ordered vip  product

	ryzer is a part of server in back of cpu that access to add module or hdd or external sas
	aroc is chip for raid works with intel and on motherboard

	sequential workload > write in random cells after a time i sort them can do this on read or just write and just work on hp

	kibibyte, mebiibyte, gibibyte, tibibyte

	hp hasn't cache	

	solid state card (nvme m2)
	solid state drive

	ssd has blocking in 3*3 \ 9*9
		windows fill with 1
		linux fill with 0

		if need edit a block of data must clear block and write again

		flash write amplification make editions without rewrite

		wear leveling :
			some blocks after many useages get crash
			with this option use whole drive in balance mode and distributed
			screw up the raid
			if all blocks get truble convert to read-only

		remapping write :
			if a cell in block get destroy with this link again

	p440ar (ar means on motherboard chip)

	raid 0 has less raid penalty
	raid 5 in minimum need 3 hdd maximum hdd 16 in 1 hdd has failover 5(4 storage space +1 data)
		200iops *4 - raid penalty

	raid 6 has 2 failover

	hotspare :
		froce to born hdds in each raid group with hdd pool after rebuild

		rebuild > if hdd borned and replace it with another one, cluster get crash and make raid again ffrom start

		when use spare ? have many raid pack

	red button means hotplug
	gray means power off
	blue means cold plug

	red and triangle or hand shape > raid 0 (no failover)
	blue on hdd > borned detection
	green ring :
		cycling > rw
		freez > not raid
	mid light in hdd :
		yellow > readonly
		red > fail

	hdd battery must checked in 6 months

	smart array :
		p408iasr
		p section :
			p > performance (highest)
			e > essential
			s > software (in microservers)
		4 section :
			series :
				200
				400
				800
		08 section :
			sas line
		i section :
			i > internal
			e > external
		a section :
			a > modular type a
			b > blade
			c > modular type c
			m > mezzanine blade
			p > pcie
		sr section :
			sr > smart array
			mg > mega raid (32hdd)

	jbod :
		fc mode
		hba mode

	in part number we must see b21, this code means our devices is fresh in health

	cpu :
		risc
			reduce instruction set computer
		cisc
			complex instruction set computer

		es (engineer sample) > test model 18 month, cpu-z detect these

	if u need order server u can :
		bto > bundel to order (all things are in)
		cto > config to order (customization)
		ato > assemble to order (we set or orders to hp and hp bundel our orders)

	hpe warranty check \ diagnostic test
	more than 80 hours means used

	ram chips contain 32 bit or 64 bit
	64 bit means single rank
	2*64 bit means dual rank

	ecc (error correction code) > like parity

	hp has nvdimm > also use ram parameters have a nand like ram with whole ram module capacity on board module
		also has power manager and battery to save data inside ram on this chip (hibernate)

	white slots are master and black is slave

	hp don't test slvaes

	load reduce > has different buffer and newest but not like before powerfull

	flr > flexiable lan slot 

	some graphic cards works on ryzer 2

	inteligent lights out (ilo) :
		remote access to server without edge public access
		we can see boot steps
		has port and ip
		mini computer on server to manage
		don't backup this section
		altho power off server we can run server to on mode if has power
		ilo5 is the latest version 

		user : administrator
		pass : seral of ilo server \ server serial

		new updates checked with hash md5
		frameware with digital sign on md5

		in port we have 5 steps must pass them
		we have some backups to recover them

	keyboard moouse video (kvm):
		we don't need to buy many prepherals for systems 

	arrangement of devices :
		centralize :
			a rack of full switches
			many cables
			cost and volume of switches be economical

		end of row :
			in each row of racks the last rack contains switch
			cable volume reduce

		middle of row : like above
		top of rack :
			in each rack we have switch and the uplink to another devices

		fex is centralize and tor mixed up
		fex is not a switch 

		passthrough is like cable that bypass and connect to server

		half > 2*10gig ethernet
			embeded 1,2
		full > 4*ethernet
			connect on 1 , 2

		3,4 > mez 1
		5678 > mez 2

	synergy > integrity option
		6 interconnect
		12 blade for servers
		a giant
		manage by a port

		composer
			for raid has template to apply on blades and other options ...

		image streamer
			on many blades we have many esxi in installation we can advertise ios as derive

		framelink module
			manage normal links tha connect to another device

	virtaul conect > fc + ethernet > fcoe

	converge network adaptor (cna) > use for fcoe

	mez 3 > type c
	mez 1,2 > c\d


			mez 	mez
	mez1	1 		4
	mez2 	2 		5
	mez3 	3 		6

	satelite switches are like fex but aggregate 4 chasis together for 2 master an 2 else for satelite
	support aci

	superdom has switch and merge the servers

	world wide name 
		64 bit
		128 bit
		x1x2:x3x4:x5x6:x7x8:x9x10:x11x12:x13x14:x15x16

		x2-x7 > company id
		x8-x16 > vendor specific info

	ech port on wwn card has pwwn :
		aaa and aab

	fcid 
		24 bit > 8 bit domain id + 8 bit port id + 8 bit area id

	fabric switching 
		more than a gig above has supervisor 
		have many board and use many codes

	cross switching : fullduplex and has a board have spaning tree

	spaning tree in wwn use lower value to set master

	domain id 1-239

	subordinated switch :
		connectivity 2 domains and force collisions in area

	area and domain use 0 as default value

	flogi > fabric logi
		in fcid recieve our wwn comes in flogi table
		like arp
		sns db is a replicate
		rscn send these to specific destination

	plogi > port logi
		enter  new device advertise to secific port with rscn

	prli > after advertise all items link together directly

	fabric short path first (fspf)

	register state chane notfication (rscn)
		swrscn
		rscn

	in san we have word 32 bit like bit in lun

	word > frame > sequence > exchange
		each exchange contains 3 sequence

	buffer in buffer credit / flow control:
		if our servers need transmite heavy data must checked buffers and capacity with exchange
		they don't check recieve acknowledge 
		in each uffer unite say spaces
		we don't have sniff in san because don't have flood

	end to end flow control :
		if middle switches turn off ?
		with this option we can detect capacity of middle switches and destination
		we can add many switches with mirror buffers

	virtual fabric or vsan in cisco
		zoning 
			work with specific wwn + more security
			divided targets

		each 15 minuts send message with rscn on lan and reply specific clientss
		for each target we have a zone
		sserver side must detect tagsand zones

		dataplane is seperate like vlan

		wwn zoning is soft
		port zoning is hard

		it's not ok to connect a link or cable without zone between 2 switches but some imes need replicate zone db like stack 
		manage plane is seperate

	inter switch link \ extended inter switch link :
		use for trunk expansion port

	zoning :
		alias
		zone
		zone config
			turn on principle

			perhost zoning > connect to one by one host or storage

			in zoning we have seperated data plan 

			vsan seperate all (data plane + control plane + management plane) > virtual context like vlan
			each port of vsan joined to one vsan like vlan 4096 in cisco

			in each vendor we have default after definition will clear and replace

			for vsan connectivity between each switcch must connect seperate cable from each switch to another switch
				so easier way is trunk > isl trunk license

			our buffers distribute between vsans so be carefull

	cisco commands on mds zoning :
		conf t
		zone broadcast enable vsan 10
		zone clone orig zone clone zone vsan 45
		zone comact vsan 1 (create one)
		zone convert smart-zone vsan 1 (convert)
		zone default-zone permit vsan 2
		show zone set name zs1
		fcalias name x vsan 10
		number pwwn ........
		exit
		zone name x2 vsan 10
		member y2
		exit
		show zone active

	jre 7 up 8
	firefox 21
	in java must set security on ip > trust

	backup :
		recover point object (rpo)
		recover time object (rto)

			closer is better
			less value is better

		full
			every day have full backup
			in weekend have total backup
			rto > faster than differential 
			during week lost a many various data

		incremental
			changes from backup day till today
			all changes from last backup
			rto > worse than differential
			rpo > better than differential

		differential
			changes from total backup +full backup
			rpo > better than full

		synthetic
			backup > incremental > rpo
			restore > full backup > rto

		make some partitions and convert to offline
		vtl > virtual temporary link (tape)
			convert to readonly

		rcout > 0

		recovery time reduce

		vpower 
			nfs > ethernet
			fc > fcoe

	to config san switch
		cofnig tab > admin zone
			enable all zones

	storage replication :
		lun transmission to another storage

		synchronize > traffic from server transfer to lun 1 then to lun 2 and send acknowledge 
			good bandwidth
		asynchronize > traffic at the dirst step set on lun 1 and srv then send acknowledge after these lun 1 and lun 2 negotiate again send acknowledge
			bad bandwidth

		active-active
		active-passive\standby

		one to many replication > directional \ birectional
