HPE SAN Essential
	hpe (enterprise) : server ....
	msa : entry level storage (small - medium office)
	eva (enterprise level)

	3par \tripar > hpe buy this product license
		can't find these series from 6 years ago

	ibm and fujitso , netapp produce allflash storages or purestorage

	price per performance (ppp)

	datacenter switching > mellanox

	in irans market :
		hp msa
		hp 3par
		emc vnx
		emc unity

	das :
		single os access data and storages, not share
	nas :
		storage pool handel by raid controlle and switch and tcpip stack 
	san :
		use fabric channel protocol and dcoe option (das biger concept)

	media :
		odd
		hdd
		ssd
		tape

		tape :	
			lto (means generatioin)
			bigger means more density
			lto7 > 15TB
			rw > sequential (cold data)
			backward compatibility

		hdd :
			small form factor (sff)
			2.5 inches
			faster
			lesss capacity from lff
			smaller than lff
			high speed in raid

			large form factor (lff)
			3.5 inches
			more capacity
			normal speed
			need more power
			in raid we have more storage space

			round per minute (rpm)
				home : 
					5400
					7200
				server :
					7200
					10000
					15000

			has 64MB and 128MB cache

		flash or ssd :
			random i\o 
			1mb in flash on badest situation is 15x faster than hdd

	hgst : hitachi was purchased by western

	in storage we have 2 section on pins smaller side used r&w and bigger side use power

	when data comes to storage our data divided to our nand chips count with controller chip (cpu arm 700 mhrz use aes encryption)

	our nanads are raid together so lost one nanad means lost raid

	we have dram chip (cache) use to read or write faster
		write back model
			write (dram \ cache)
			read (controller)

		writh through model
			write (dram)
			read (controller \ cache)

			consumer
				in opposite of enterprise
			enterprise
				power less protect (must write every thing on dram in old version had battery)

	nand types :
		slc (singel)
		mlc (multiple)
		tlc (triple)
		qlc (quad)

		single to quad :
			expensive > cheaper
			faster > slower
			more tbw > less tbw 
			in quad we have more capacity

		total byte write (tbw) if get expire our nand will be read only 
		dwpd (drive write per day) : fill and empty memory

	v-nand : vertical nand
		3d computing
		more ssd cpacity
		faster
		more life cycle
		more cache capacity

	over provisioning :
		enable on some nand and not enable on some else
		save 10 percent of capacity if one nand works badly can replace some nands , better enable on server

	in raid controllers we have power lost in raid mechanism we divided data on nand count and send acknowledge to os actually raid controller write data on cache not disk, write procedure on disk take at the another time in this time if we lost power get fucked so must provide battery

	interface :
		sas 
			enterprise and fullduplex
			sas cable 15cm - 25m
			disc enclousure d2700 \ d3700
			version :
				1 > 3 gbs
				2 > 6 gbs
				3 > 12 gbs
		sata 
			home and normal usage
			normal capacity
			normal bandwidth
			halfduplex
			versions :
				1 > 1.5 gbs
				2 > 3 gbs
				3 > 6 gbs

		nvme
			logical concept in transmission
			ssd need more space
			use pcie interface 
			m2 and u2 are formfactors
			m and u are logic transmission protocol
			we have sata and sas m2 formfactor
			m2 has :
				mkey
				bkey (obsolete)

			dl380 g8 and g9 don't support this module

		msata was version before m2sata

			u2 is 2.5 inches formfactor and support 70 nvme with maximum io

			intel samsung kingstone

			hhhl use pcie x4

			hp g10 servers use 4 u2

	i\ops performance:
		sequential rw : must use fix speed 
		random rw : alocate size
		raid
		thread : power on vms count has many threads and copies 
		queue : same time transfer thread

		in iops peformance thread and queue if be higher our delay will be more altho better performance

	crystal disk mark > performance test

	p200g3 just works with sas and problem with san storage

	cady on g 10 is different with g9+8 and g6+7

	ssdbazar.com

	depends on media we have san\das  \   nas\das

	raid controller aggregate disk or storage pool together but jbod define all hdd in seperated drives  to os
	on jbod disk management is windows base

	in g8 to higher versions we can convert raid controller to jbod

	in nas we have shared hdd or storage pool between many os multiple access
	freenas 
	qnap is file sharing concept
	in nas actually use das
	not suitable bank
	single point failor
	use tcpip maybe use more bandwidth and load
	file level concept + tcpip  > ftp , nfs ....

	san is different story use fabric channel over ethernet protocol
	use 5 layer in protocol
		l0 > physical (lc fiber)
		l1 > data link (produce signals and led pulses)
		l2 > network (control signals i\o)
		l3 > common services (raid and encryption)
		l4 > protocol mapping (rw standard on os like iscasi)

	fchba card use in topology
	fabric switch

	we have type of ports :
		node > client
		fabric > fabric connected port
		expansion > trunk

	we have routing in fc

	fcoe behave like fc on layer 2 after that can mixed up mac iscasi and tcpi
	need set jumbo packets in ncpa.cpl

	in esxi must set mtu on 9000
	if need 4 switches must set 4 vm kernel
	esxi storgae need software iscasi and config iscasi

	must enable multipathing in vcenter to use fc

	iscasi use tcpip and wwn addressing on 64 bit like ip address class
		connect to port not sfp
		world wide name

		we have name zone
		name zone works like vlan in ethernet
		some users need to use storges
		need ntfs permission in this part we have lan mapping

	raid :
		redundancy
		fast speed
		storage aggregation

		different from disaster and backup concept

		in raid we have :
			file base :
				speed x1
				division our data to hdd count
				not applicated
			block base :
				more speed 
				allocate unite size
				divided hdd on some cells 


	raid :
		1: hard
		2: soft

			model			cap			fault telorance			writesp			readsp		maxcounthard		mincounthard

			rai0 			 n 				no 					 ncount 		ncount 		depends on 				2 (crazy raid)
																							controller
		
			raid1 			1/2 			yes 				  1/2 			  1 			2 					2 (mirror)1
		
			raid5 			n-1 			1 disk 				  n-1 			 n-1 		   doc 				    3 (parity rotary) cache
		
			raid6 			n-2 			2 disk 				  n-2 			 n-2 		   doc 					4 (2 layer 5 raid)	
		
			raid10 			n/2 			1 					  n/2 			  n 		   doc  				4

		raid 0 use for cache and places need more speed and less security
		raid 1 is mirror use for os 
		raid 6 use for security and redundancy also need more storage space 
		raid 10 use for databases first step we raid on 1 and then raid on 0 use l1 and l2 terms
			if lost odd or even hdd can still work but lost a array can't work

	raid controller cache :
		battery back write cache
			write on disk
			read on cache
			don't need battery
			write through

		flash back write cache 
			use nands with highest speed
			write back method use this
			need battery
			write on cache
			read from disk

	the best way to use backup > sharing blocked dass (iscasi)

	in msa2050 must use same hdd to make raid 
	optane dc persistent memory > 1.5 TB ram use if in power lost save cache
	storage tiering > aggregate ssd and hdd together and seperate traffics by patterns
		in minimum need 2 ssd and need fast cache
		fast tier > 10% hdd , 25%ssd
		emc use this
		p200 don't support this
		fast cache works like fbwc , higher is better, enterprise level need bcuase tbw


	software define :
		capacity : ssd
		fast tier: nvme
		cache : intel persist memory

	in raid controllers we have chunk concept
	actually we use chunk in raid
	per chunk we use mirror raid or other types

	we can write roles on chunks if lost disk enclouser, you can recover enclousure
		enclousure evernese resevlianse
	
	msa 2040 to higher level block sata

	data deduplication :
		big blocks converted to smaller blocks with unique unit and link repeated data to each other with chunk store
		then clear original data

		dedup help us to read data faster

		online
			need resources like cpu and ram more than offline
			1TB need 35 gig ram
			save life cycle of ssd or storage
			microsoft in this mode has better performance


		offline 
			after write dedup data
			need more storage

		in 3par we say reductior

		in servermmanger we cant right click on hdd
		config dedup :
			normal file
			vdisk
			backup

		powershell > start-dedupjob -volumeE: -type.optimized -memory 100 -cores 100

			bydefault is 50%

			optimize > if not busy start
			throughput > fuck disk

	in disaster recovery must make copy from data and use asn to reachability

	ater buy carteridge must buy sserial
	must be on same version our drive and carteridge

	hci comes in clustering seperated hdds and servers, backplane sas, distibute raid
	in next layer set hypervisor

	storage space direct (s2d)
		minimum :
			2-3 cache drive ssd
			4 capacity drive hdd

	vsan use in vmware

	sdn has bgp and router option
	datacenter sdn in win srv 2019 not support

	g9-10 win srv 2019
	g7-10 win srv 2016
