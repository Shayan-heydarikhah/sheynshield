Devnet
	iaas
		a computer with os on cloud (cpu ram hdd network ...)
		turning on vms (sping out instationtion)

	saas
		some software tasks based on iaas and give online service
	
	paas
		ide + iaas
			design a platform on cloud

	cloud service router
		router ios that give features like router without router device

	data plane (forwarding plane)(use asiic) :
		decapsulation + recapsulation in datalink
		add\remove dot1q tjing header
		match ethernet frame destination mac on mac address table
		match ip on ip routing table
		nat
		vpn
		acl
		port security

	data detect from control plane then outgoing data detect from data plane

	control plane 
		destination ip > routing table \ mac address table
						-----------------------------------
									control plane (ospf , eigrp , stp , bgp , arp table ,....)

		use cpu and background tasks

	management plane
		ssh telnet snmp syslog

	traditional network design and problems :
		application specific id 
		ceph is asic  >>> data plane
		application search engine
		useful section of mac address use
			ternary content addressable memory (tcam)(useful section are here)

	software define network (sdn)
		some applications get trouble with sdn (openflow)

		south bound interface
			above all devices we have controller and all manageable devices are bottom 
				(outgoing manager interfaces to clients)
			api is an interface between some devices
				api give us a function
				api + functions on application

			oplex(openflex)
			openflow
			cli ssh telnet (apic-em cisco)
			cli ssh telnet (netconf cisco sdn aci)

		north bound interface
			controller must have list of all devices (information of device state, features, hardware information)
			must known topology of network
			what config is running on device

		*controller is a repository

		nbi > application (provide information for app and api from some queries)
		sbi > device

		controller (program) <-nbi-> api (java base) <-> application java

		rest api (representational state transfer)
			json (java script objject notation) and xml use for language

			application >>>>>>> http get		<<<<<<<<< controller
				pc 		>>>>>>> http request	<<<<<<<<< server

	open foundation network (onf) design sdn
	opendaylight (openflow)
		opendsn controller
		open source and use open flow
		each newest devices have osc (cisco opensdn controller)
			internet base network (ibn)

	datacenter aci
		each leaf and spine connect to each like this :
			leaf <> spine
			leaf X leaf \ spine X spine

			endpoint <> leaf

			aci works on performance :
				vms > endpoint
				policy > data goes to apic

			web application
				web server
					on ibn base give endpoints policy
				database server
					apic prepare a virtualization define epg

			express the needs + define epg > automatic config

	apic enterprise module (apicem)
		digital network architect use apicem

		in sdn must use newest switches or newest ios but have cost so use apicem to manage old versions

		control plane in apicem is on local device in other mode are centrall

		control plane of openflow get evolved

		aci\apic em > cisco
		openflow > onf

		sdwan (aci and dna in wan)
		sda (aci and dna in lan)
			use different ip range for lan and wan 

		dna :
			underlay (infrastructure)
			overlay (services result (use vxlan (use asiic)))
				before changes must check which device type can be capable to handle
					csico.com\go\sda
				between 2 fabric edge we have vxlan tunnel in auto mode
				works in data plane

		south bound interface
			sda-fabric
				overlay
				underlay

		fabric edge node > a place on access switch that epg outgoing data recieve by sda-fabric 
		fabric border > sda connect to wan or dc
		fabric controller > use lisp (special usuage)

		in sda and sdn must design our dc with mls in access layer, better run isis drp and routed mode design + layer 3
		we have lisp (locator id seperation protocol) on sda (auto mode tunnel but no route mode)

		like gre make tunnels on sda but use lisp and detect destination

		overlay control plane > endpoint id + routing locator > mac address 
			eid connectivity faric edge ip address

		we use 3 parameter  :
			lisp
			eid
			rloc

			base on these we can run lisp map server
				contains eid and self rloc ip transport to server

			each connection first goes to server then check data does exist  ? if exist > destination send ack to source and make tunnels

		we can manage old devices with dna 
			dnac use seperated policies with sgt (scaleable group tag)
				use many object and groups if be allow > make tunnel

			in older versions use cisco prime on ucs cisco

			benefits :
				single pane of glass
				convergence
				swim (software image management)
				life cycle management (zeroday - day1 - dayn)
				discover inventory topology
				entire enterprise
				application visibility
				network time travel
				devices health
				devices and clients count 360
				sda support
				encryption traffic analys
				path trace
				easy qos

		acl is not flexiable so use group or scaleable groups
			ace > access controll entry (acls parameters or elementss)
			each group have a tag

		applications can run on local and not in central controller so use api 
		application programing interface (api)
			inquir application like database
			for controll base networks need rest api
				non local applications
				http \ https

			client side authentication
			clear statement of cacheable \ uncacheable
			stateless connection \ operation
			uniform interface
			layered
			code on demand

			often works with http to use refresh rate and time frame

			api variable :
				simple
				list or directories

			crud : create	 update 	read 	delete
			http : post 	path + put 	get 	delete

			for each value need too get from api must set uri

				*https://dnna.example.com\dna\internet\api\v1\network-devices

			between api and dna and restapi and application must use authentication
			
			cisco api call (postman)			

			data serialization 
				focus on input and output querier
				values connect to simple text

				rest server <-----------------> rest api <------------------> rest client(python)
									query						query
									
								convert to 					convert to 						
								normal text					normal text
									form						form
		
									json						json <> content

				data modeling
				xml > rest api + markkup language
				yaml > rest api + ansible
				json > rest api

			configuration drift
			
			change management system
				save histories and manage logs

			centralize config files and verion controll
			config managemnt and monitoring & enforcment
				make a copy from range then compare between running and ideal then deploye changes

			yaml :
				config provisioning
					config deployement and checking (config monitoring)
	
				playbooks
					text files define actions 
					monitoring
	
				inventory
					hostnames 
					subset of devices
	
				agent less (push configs)
					ssh \ telnet
					netconf

			puppet :
				use agent + api
				some cisco devices and iioss does'nt support this
				use proxy agent (after connect to server use ssh to manage device)

				manifest (read)
					pull

			chef :
				like puppet
				chef zero (standalone)
				server and client
				now does'nt use agent or proxy

				task
					cookbook
					recepie
					replist
					resource

			ansible				puppet				chef
			playbook 			manifest			recepe-runlist
			ssh-netconf			rest (http)			rest (http)
			agent less 			agent 				agent
			push 				pull 				pull

Devnet :
	version controll
	abstraction (no vendor base)
	automation

	all of the network can be on a code (infrastructure as code) 
	can use roll back 

	mib (management information base) have version 1 and 2 use some object-id (oid) on snmp and make some configs and manages or monitor session and values
	
	concept model :
		transport
		data statement (yang)
		data encoding (xml , yaml , json)
	
	yang is the newest mib model
		yang use yang explorer\p-yang

	oid contain name space and yang model

	netconf use port 830 and work on ssh (use roll back feature + remote procedure call)
	restconf use http like grpc 

	github\yangmodel\yang

	container means we have a list of special and unique things base on oid name in mib (here we call leaf)

	we have seperatet sessions for manage and state

	pyang -f tree ietf-interfaces.yang
		(list <-*)
		(? means optional)

		ssh rayka@192.168.1.95 -p830 -s netconf
		ssh rayka@192.168.1.95 -p830 -s netconf | grep ietf-interfaces

	netconf :
		use ssh and remote procedure call
		have roll back feature
		some other features :
			commit , delete config , copy config , edit config , get config
			rpc (get) , lock , unlock , close session , kill session

		code with xml

		developer.cisco.com
		github.com\ciscodevnet\netprog.basic (csr machine)
		devnetsandboxcisco.com

		ncclient > netconf library	
			xmltodictionary (python container)
			xmldomminidom (pretty view)

		all of the equipment must have datastore like rang (means config)
		if need change some datastore values must use filter

		netconf_reply = m.get_config (source='rang',flter=netconf.filter)

		netconf_fiter=
			<filter>
				<interface xmlns="urn : ietf : params : xml : ns :yang : ietf_interfaces">
					<interfaces>
				<interface>
			<filter>

		manager.connect {
			host, password , username , port , hostkey_verify(this feature use for ssl and enable showing or preventing to show public key) 
		}as m;

		print(xml.do.minidom.parsestring(netconf_reply.xml).toprettyxml())

		netconf_data=xmltodict.pase(netconf_reply.xml)["rpc_reply"]["data"]
		interfaces=netconf_data["interfaces"]["interface"]

		impport ncclient import manager

		for interface in interfaces :
			print (".........".formatc)
				interface ["name"]
				interface ["enabled"]

		python program :
			datacenter={"host": "" , "username" : "" , "password" : "" }
			if envirment_in_use == "sandbox" :
				dna_center={"host":"sandboxdnac.cisco.com" , "port":443 , "user":"rayka" , "password":"rayka" }
				ios_xe_1{"host":"192.168.1.95" , "host":"lab.exam" , "username":"rayka" , "password":"rayka" , "netconf": 830 , "restconf":443 , "ssh":23}

				*after run this script we can see xml type data depends on yang from exporting data from device

		if need changevalues \ config device must sset config instead of filter :
			<config>
				<interface xmlns="urn : ietf : params : xml : ns :yang : ietf_interfaces">
					<interfaces>
						<name> [interface managers] </name>
						<type xmlns="urn : ietf : params : xml : ns :yang : ietf_interfaces">
							new_loopback=[]
							new_loopback=["name"]="loopback"+input("plz enter interface name : ")
							new_loopback["type"]=ietf_interface_type["loopback"]
				<interface>
			<config>

			netconf_data=netconf_interface_template.format(name=new_loopback["name"],type=new_loopback["type"],......)
			netconf_reply = m.edit_config(netconf_data,target="runnig")

			cisco just use startup config and must use yang structure (cisco.ia)
				netconf_reply=m.dispath(xml_oto_ele(save_body))
				save_body=<cisco_ia:save_config xmlns=cisco_ia="https://cisco.com\yang\cisco_ia"/>

		restconf address formating and content :
			http:/<address>/<root>/data/<[yang modle :]container>/<leaf>[<options>]

			accept > what content in packets must be used
			content_type > contents must be yang or xml

		curl (use for troubleshooting)
			curl_vk (verbose- non secure)
			(user)-a rayka:rayka
			(header) -h "accept:application\yang+data+xml (or use yang+data+json)"\
			https://192.168.1.95/restconf/data/ietf_interfaces(yang_statement):interface(container/interface=gig1 (leaf)
		
		postman (use for rest api and making api)
			in setting > general > ssl cert verification (off)
			if use self sign cert maybe have trouble

		python

		if have problem in certificates in python and post man :
			import requets
			requests.packages.urllib3.disable_warning()
			url="https://....:443/restconf/data/ietf_yang_library.module_state"
			payload.()
			header=('content_type':'application\yang+data+json','accept':'applicatiion\yang+data+json','authentication':'basic...','user':'password','authorization':'basic...')
			response=requests.request("get",url,headers=header,data=payload,verify_false)
			print(response.text.encode('utf8'))

	nxapi
		access to shell linux
		python scripting
		agent (puppet - chef)
		docker

		#feature bash-shell

		can be programable

			---------------------------nxapi---------------------------
					nxapi cli 						nxapi rest
					json prc 						data management engine (dme)
					ins-api (cisco)					management information tree (mit)
						xml + json						all nodes are object

		nxapi cli (use cli for every thing (manage and mmonitor))
			must use \ins and post method

		ins -api use bash

		nxapirest (dme)
		restconf (yang)

		cli and cli ascii have differences like ascii can be human readable

		if set message format on xml in command type can use bash shell

		input textbox : vlan 30 
			request 
				[
					{
						"jsonrpc":"2.0", "method":"cli", "paramas":{"cmd":"vlan30", "verion":1}, "id" : 1, "rollback": "rollback-on-error"
					}
				]

		nx-api-rest
			use dme and mit
			these oop provides by http

			nx-api-rest on sandbox cisco > methods > dme
			model browser
			visore interface
				data structure dme on nexus devices
				root file (sys<>)

			with url we can find path if exist mo (means special object from class)
			after authentication we use a token that use in various taasks

	jinja2 templates
		raw commands (netmiko (python module))
		a library with python condition loop and... + connect data and config
		with this can deploye every thing on every device type
		if found yang data structure and xml\json and name spaces can config

		host file
			list of equipments ip

		data.yaml
			list structure and dictionary each subset of list is disctionary
			shows with (-)

		router ospf{{process-id}}
		router-id{{router-id}}
		network{{{network1}} {{wildcardmask1}} area {{area1}} }

		ospf:
			- process-id=1
			  router-id=1.1.1.1
			  networks= - network 192.168.1.0
			  			  wildcardmask= /24
			  			  area0

		jinja2 templates :
			router ospf {{process-id}}
			router-id {{router-id}}
			{%for ospf in ospf.network%}
			network{ {{ospf.network}} {{ospf.wildcardmask}} area{{ospf.area}} }
			{%endfor%}

		python code with jinjja2 :
			ncclient import manager
			import xmltodict
			import xml_doom_minidom
			import yaml , safeload (make list and dictionary with this)
			import jinjja2, envirment, filesystemloader

			with open{f"data.yaml","r"}as handler;
				ospf-data.safe-loader(handler)

			j2_enr=envirment(loader=filesystemloader{"."},trim_blocks(means no space)=true,aeoscape=true (enable regex))
			template=j2_enr.get_template(f"ospf_config.j2")
			ospf_config=template.render(data=ospf_dta["ospf"])

			manager.connect {
				host, password , username , port , hostkey_verify(this feature use for ssl and enable showing or preventing to show public key) 
				}as m;

			netconf_reply = m.edit_config(ospf_config,target="runnig")
			print(xml.do.minidom.parsestring(netconf_reply.xml).toprettyxml())

			our configs get replace
				config > native > router > router osp > ospf
				<ospf operation="replace">

	cisco aci :
		aci toolkit (use objects without url)
		internal datacenter sdn feature
		layer 2 media with no layer 2 limitation

		tenant (datacenter isolation)(parent class)
			network
				vrf
					bridge domain (like vlan)
						subnet
			application profile
				epg
					contract

		------------------------------------------------

		tenant (pearent class)
			bridge domain
				subnet
			vrf
			filter
			contract
				contract subjects
			layer3 external
			filter
			application
				epg
					clients (end point ip)

		cisco cobra and rest api use for apic programming

	GIT
		version controll
		developer collaboration

		cmd :
			cd c:/projects/
			git init

			define a path to git community and set project paths

			git status (make sync)
				check all files version

			git add x.x (goes to stage area)
			git add *
			git add -a

			git commit -m"date" (must set versions with text)

			git restore -stage x.x (remove from stage)

			git log --oneline (show versions)
			git log

			git checkout (back where) a number with 7 digit ^
			git checkout master
				with this we can make branch

			git commit --amend -m"date" (replace the last commit with out no midification)

			git revert (7 digit) -m " date "

			git reset (clear history)

		git collaboration	
			we have team a and team b
			for fetch and update some things must push and pull
			if team a pull some date then modify data and push it to server during this procedure team b change values 
			server deny push
			first pull newest team b data to team a then change versions and update them then push modified data to server

		branch
			team a make a copy from master 
			chage the values on local servers
			master projet also make some changes
			for problem solving and manage versions must update them like above

		github use for online and public soloutions 
			
			git clone....
			git remote .... (manual source definition)

			git branch team-a
			git checkout (seperate us from master)

			git merge a.a
			git push

			cobrasdk
				a library for aci progrming
				all relations are not 1:1
				have a shared object like fv or vz

			git clone https://github.com/ciscodevnet/aci-learning-labs-code-sample

	DNA
		central managing
		openflow (float table)
		enterprise controller
		not like cisco prime or network management system

		inted (application infrastructure)
			policies define based on bussiness and applications

		use vxlan
		we have manage data and control planes in central and local
		automations define tassks to devices directly but new versions define to controller

	cisco sandbox :
		user : Demo
		password : Demo1234!

		apic em is like dnac but newer than it

		dna console
			desif > sites
			assurance > monitoring
			platform > automation
			provisioning > fabric > intent > vxlan

			platform > developer toolkit

	Network Services Orchestration (nso)
		ned (must download needed drivers)
		multivendors
		in background use netconf and sometimes convert cli and snmp

		developer.cisco.com\nso

		sync from make a copy from ruuning config and take to nso

		we have a nde in packages on view 

		set devices device ce2 config ios:snmp-server community com ro
		commit

		each roll back make versions
			selective (a device)
			comulative (a cluster)

		commit dry run

	ASA With RestAPI
		copy ftp: flash:
		rest-api image disk0:/file.spa
		show rest-api agent
		rest-api agent
		http server enable
		http ip mask mgmt0
		aaa authentication http console local
		user admin secret 123 prov 15

		each hop interface get enable must push on it

		chrome 
			http://ip/api/objects/networkobjects
			https://ip/docs
				show apis

	NSO Devices
		cli
		multidevice config
			ncs-cli -c (cisco -j . jonus) -a admin
			show runing-config devices device config | de-select config (this part means just information)
			show device list (device name ios driver)
			show running device device-group
			device connect
			device sync-from
			device check-sync
			devices ddevice-group x
			device-name r1 
			device-name r2
			devices device-group all
			device-group all
			top
			show config (till not deploy)
			commit

		single device config
			devices device x
			config
			pwd
			do sh vlan brief
			show config | displa xml\json
			commit dry run

		rollback
			show config rollback change <cr>(last backup) \ number
			revert (rollback)
			rollback config x.x.x

		template
			device template set-device-dns
			ned-id csico
			config
			ip name server server 8.8.8.8
			show config devices template 
			devices device-group all apply-template
			template-name set-dns-server
			commit

		opeerational state;
			show devices dist-sw platform version
			show devices live-state ip route
			show devices dis-sw live-state exec

Py4Net :
	use interepter and compile each line on time give results
	highlevel lang
	use in various genere 

	controll plane 
		mac adertise
		mac address table
		stp
		arp table
		ospf
		eigrp
		bgp

	data plane
		encapsulation and decapsulation packets
		trunk add and remove
		match mac and forwaridng packets
		match ip on routing table
		nat table
		acls

	management plane
		telnet
		ssh
		console
		snmp

		box by box management
			each device manage on local and on site

		remote management
			ssh ...

		centrall management
			sdn and aci
				distributed controll plane
				centrall controll plane

	application centric infrastructure (aci and sdn (software define network))
	aci use appication policy infrastructure controller for network controller

	north bound interface
		application (api)
			gui
			java and python
			application

	south bound interface
		opflex
		snmp
		netconf
		cli

	api is an application or interface between applicaion and sdn
		rest api
			http 80
			get 	post 	pull 	put 	delete

		data format xml and json

	we can run python on nexus

	cmd
		py de:/1/a.py (executeable python files)

	case sensetive and must use _ or name at the begingn of variables

	in python if use help can see with type of variables be use

	need make some comments :
		""" comments and write multiple line if insert variable use in string  """
		on ide :
			# this is comment

	x= 5
	id(x)

	if set y = 5 > y points x value on ram

	x="life is good"
	define my_func()
		print (x)

	if have a function like this :
		x="life is good"
		define my_func()
			x= 1
			print (x)

		which x use ?
		in these situatios must use global
		global make a update or replace on values

	tuple ()
	list []
	dictionary {}

	text
	numeric
	sequence > list , tuple , rang
	mappin > dictionary
	sets > set , frozen set
	bolean > bool
	binary > bytes , bytes array , memory view

	x=1j
	type(x)
		*result > compelex*

	5pow3 > 5e3 = 5 * 10 pow 3 (float type)

	compelex > 3j > can't convert to something 

	import random
	print (random.random(1,25))

	a="salam 2nya"
		print (a[1]) > a
		print (a[-3]) > n
		print (a[0,5]) > 0-4

		print (a.strip()) > ignore spaces

	a="$cisco$"
	print (a.strip("$"))
		or
	a=a.strip("$")
	print (a)

	a="salam , 2nya!"
		c=a.replace("2","do")
		print (c) > salam , donya!
	
		b=a.split(",")
		print(b)

	txt = "the train in spain stays mainly in the plain"
	x = "ain" in txt \ x="ain" not in txt
	print (x)

	txt="my age is"
	age=36
	print(txt.age) > my age is 36

	txt="my age is {}"
	age = 50
	print(txt.format(age)) > my age is 36

		in {} place inject values

	val_1 = 1
	val_2 = 2
	val_3 = 3
	txt="first is {2} second is {3} third is {1}"
	print (txt.format(val_1,val_2,val_3))

	txt="salam \"ahvalet"\ khobi "
		\\
		\"
		\n > enter 
		\r > befor enter ommit all chars

	a="python for everbody"
	x=a.index("o")
	y=a.count("o")
	z=a.find("every")
	w=a.find("cisco")
	q="_".join(a) > between each character add _
		p_y_t_h_......

	a=1
	b=2
	if b>a :
		print ("b is bigger than a")
	else :
		print("a is bigger")


	false :
		bool(false) , (none) , (0) , ("") , (()) , ({}) , ([])

		x=200 
		print (is instance(x,int)) > tru

	// submultiple
	/ divide
	% divide remaining

	x&=3
	x|=3
	x^=3
	x>>#

	x=5
	x>>3
	print x > 40

		00000101 > 5
		00101000 > >>3 (shift)

	list have order and changeable values + index and printable
	thislist["x","y"]
	thislist.append("z") (at the end of list inject values)
	thislist.insert(1,"z") (injext in specific index)

	use remove to ommit items from end of list
	use pop to ommit some values  with index
	use delete to purge data from ram
	use clear to make our list items ommit and make empt list

	thislist=["x","y"]
	mylist=thislist.copy()
	print(mylist)

	have different id

	thislist=["x","y"]
	mylist=list(thislist)
	print (mylist)

	list1=[1,2,3]
	list2=[4,5,6]

	list3=list1+list2
		or

	for xin list2
		list1.append(x)

		or

	list1.extend(list2)

	thislist.reverse()
	thislist.sort()

	def myfunc(e):
		return len(e)

	cars=["ford",....]
	cars.sort(key.myfunc) \ (reverse=true \ key= func)
	print(cars)

	tup=("apple")
	print (tup)

	tupple is noncountable 

	set have no order no repeated data no cahngeable if need can add items
	add one item and update use for many items
	discard remove items if not exist give no error
	remove use for delete items and if not exist has error
	pop in set have more details like wich item be ommiting

	dictionary is changeable and has order + use index like key=value
	thisdict{"brand":"ford" , "model" : "mustang" , "year" : "1998"}
	x=thisdict["model"]
	x=thisdict.get("model")
	thisdict["year"]=2018

	print keys :
		for x in thisdict :	
			print (x)

	print values :
		for x in thisdict :
			print (thisdict[x])

		for x in thisdict.values():
			print (x)

	key and values :
		for x,y in thisdict.items():
			print(x,y)

	in dictionary we have update in exist values
	also have pop option if set pop value send confirm if don't set value by default use silent ommiting from end of the frames

	in earlier versions delete from random values

	if need use ifelse conditions with no parameters must :
		a= 1
		b=2
		if a>b :
			pass

	for loops use for dictionary and set tupple list strings
		for x in range(6)
			print(x)

		for x in range(2,10,3) (each 3 item)

	define myfunc():
		print("salam")

	define myname(name="ali"):
		print("i am " + name) > i am ali

	define myname() > ali

	define myname("hassan") > hassan

	c:/modules/modules.py
		def greeting(name) :
			print ("hi " + name )
	c:/modules/hi.py
		import modules
		modules.greeting("hassan")

		import modules as mamad_module (rename)

		x=dir(platform)
		print(x)
		x=datetime.datetime()
		print(x.startoftime("%b"))

	import json
	x='{"name":"json","age":10}'
	y=json.loads(x) > loads : python \ dumps : json (we can use indents)
	print(y["age"])
	print(json.dumps(x,indent=4, seperator(".","="), sort_key))

	pip (python package manager)
	if not exist on built-in packages can add
	cmd :
		pip list
		pip install camelcase
		import camelcase
		c=camelcase.camelcase()
		x="hello"
		print(c.hump(x))

	try :
		print(x)
	exept :
		print("x is ....")
	finally :

	pip install paramiko (use for ssh)
		must set priviledge 15
		also enable ssh
		c:/users/ssh/known.host (ssh keys problem)

	pip install netmiko
	pip install napalm (netmiko requirements)

	route add -p ..... (-p means permanent)

	pip install pyntc
	pip install colorama

	geeks of geeks

	kali and python
		chmod
		shbag

	convert pyv2 to pyv3
		on python path tools scope 
