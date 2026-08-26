MCSE
windows nt when get running :
	1: high performance action
	2: no power manger panel
	3:  run on server by default

	roles : server roles
	features : local features

Versions :
	xp > nt 2000 series
	vista > nt 2008
	7 > nt 2008 r2
	8 > nt 2012
	8.1 > nt 2012 r2
	10 (1807) > nt 2016	(1607) 16 is year and 07 is month > ltsb (every 6 month)
	10 (1809) > nt 2019	(1807) lstc

	ltsb > branch
	ltsc > channel (lont term service channel)(10 years)
	
	enterprise is the best verson for installation(suppurt direct access (vpn))

	run > winver

	in core version of windows server maybe have some conflict
	nano server support just a single rule of microsft server rules

it's better use backup and then update our systems

CMD Activation :
	slmgr /skms	kms.rayancollege.com
	slmgr /skms kms.abrim.com
	slmg /ato
 	slmgr /dli

Licences Mode:
	licence retail > 10 periods can be use
	vl > high count 8k need kms server
	oem > original installed windows on device

in hyperv for hdd parts we must set diffrence it's like linked or clone in vmware must format vhdx

SID (security id)
	cmd > cd > pstools > .\psgetsid.exe
		each windows has specific and unique sid (sid contains a compelex code of each hardware and software id)
		in clone  we have same sid solving this :
			run > sysprep
			cmd : cd  windows/system32/sysprep.exe /oob /generalize /shutdown /mode:un
			now delete vm (vm folder) then remove disk and make new vms with attaching new hard derive on it.

iteranet ip range 10.0.0.0/8 - 10.255.255.255 (some iranians networks in intranet are on this range)

DHCP :
	its better run on server not network devices
	and it's better run on seperated vms

	dora?
	discover > send requesst in many periodic times (braodcast and give all devices has ip) process by devices  that are listennig on port 67

		src : 0.0.0.0 and destination : 255.255.255.255 source port 68 destination port 67 (dhcp protocol is the only protocol don't use random port)

	offer (server) > offer one ip if client accept that send request if not save that offer and send another one (jungel rule in dhcp)

		in offer when clients want take a ip at the first steps must garp the ip and check arp table for duplicating ip

		in some senarios we have 2 dhcp server must set delay between them
		before sending acknowledge works with 0.0.0.0

	request 
	acknowledge
	
		if before discover and offer our clients has an ip address dhcp server lease last same ip address

	time for dhcp reservation >
		8/2 = 4 +(4/2 = 2 + (2/2 = 1)) = 7 days

		if after 7 days our firs vm wasn't reachable our clients go to next dhcp server and take new pools

	reservation :
		if make reserve the ip and mac , whatever client get die our reserved ip saved in dhcp
		it's better set srv and infrastructures in this mode

	optionss in dhcp :
		like option 42 is ftp server that defined to clients when request to recieve ip address
		proxy is option 252 wpad (web proxy address)
			in android our options sets with isp

	we have many filters in dhcp :
		allow : after entirely clients join to dhcp server if set this on items no body can join to dhccp
		deny : conversely above

	in run > dhcpmgmt.msc

	in cmd :
		ipconfig /renew (same previous ip)
		ipconfig /release (new ip)

	set predefine option (develooper and server (api))

	we can set bootfiles on dhcp

	vmm > virtaul machine manager (server name)

	we can set special ip range on dhcp for example we set edari ip range and we want set ip range edari on requests:
		on ipv4 right click and  set define vendor class and add dislay value on ascii
		then goes policies > new policies (can be use above rules)
		clients cmd > 
			ipconfig /setclassid
				ethernet edari
			ipconfig /all

	.net framework 3.5 (expire)

	relay agent > convert winserver to router and define relay agent rule on it (ip helper address)
		
		on role installation :
			remoteaccess rules > router > lan routing > ipv4 > dhcp relay agent > ip dhcp server and new interface
				routing access (expire)

		after installation we can use relay agent also has a option in dhcp to set these setting

DNS :
	dns works on :
		port 53 > udp
		53 tcp use for replication
		each company can use 63 / 64 characters on host

	(fqdn)	x1 		.y2		.z3		.com
			-----------		-------------
			company
			alternatives

		ipconfig /displaydns (show cache)
		ipconfig /flushdns
		cd c/windows/system32/drivers/etc	(can change host file details and have some attacks)
		
		tld (top level dns)

		cache only server :
			applied in isps 
			many clients asked  

	dns server need authentication in zone

	a record (in making this better enable ptr and reverse)
	cname or alias (name to name)
	mx stands for email exchange in server mails and nslookup
	srv or server location works a head of mx records beacause use service and ports

	name of services define with (-)
	
	in reverse lookup for security must encrypt datas like dc1 with this ip address must encrypt data
	send ip and resolve name
	in security tasks is important
	use in infrastructure

	vlmcs > volume license use for kms and -tcp and port 1688 parameter

	in dns properties must set in advance > recursion (make it disable)(answer own zone requests not internet request)
	
	we can load balance or load sharing on servers with priority and weight

	we can change cache time by right click on server and select properties zone and soa cache time changin (use ttl)(with soa must advertise th ... doman is not exist)
	
	root hits : send updates to all tlds we have 13 root hits or tld

	how transfer records :
		root hits > tld > cache only > dns of organization

			dns server name resolution > authoraty  (same zone) > forwarder > root hits

		in client side :
			host file of clients > cache > dns 1 > dns 2

		if don't have answer of requests must answer that not exist and must cache that

	last character of fqdn is . and include all zones

	reverse lookup zone(use supernet)(also use in authentication)

	pt (resolve ip to name)

	checking email address is correct spf

		if spf and ptr didn't match we have fault in way on spf record or mail server
	
	policies > computers > admin temp > network > dnsclient > register ptr record (client name and ip address send together)

	dynamic update is better get disable in zone creating (aging 14days(firs 7 days belongs to computer could answer or not + second 7 days don't respond))
	
	we have zone transfer in dns (replication concepts)

	if we have 2 dns server it's better set 20 seconds delay between them (soa is time of replication)

	in dns we don't have @ character

	stub zone > use forward to origen server just copied ns (cache and replicate ns) use for small office

	deligation > big part of processes transfer to one partition don't replicate subsets and applied in forest (primary admin must set this)

	simple query > my zone 

	recursive query > other zone

	ns zone (name server zone) > owner f zone

	conditional forward > if see some range forward it to other (between this and stub zone we should use this beacause	stub mode need replication permissions)

	forwarder is like default route (bridge)

		right click on forward zone and add new zone (ipv4 > supernet) then on new object take properties add name server
		then foes to ripe and on our supernet subit our request (as number and ptr) 
	
	priority of dns checking steps :
		zone >	cache >	conditional > forwarder > root hints

	isps use cache only dns server also use when we must block dc internet access so make forwarder and our dc sets on cache only server

	disable recurtion :
		users just use our zone not more (it's better enable)
		not in lan just in wan is applicated

	enable bind secondary (for linux os dns server bind)

	fail on lead if bad zone data :
		if our zone db doesn't have needed data ,works or no
		for more than 1 dns server get enable

	netmask ordering :
		return each user to their networks don't realize subnet

	secure cache againest polution :
		arp aspoofing (secure dns data in ram and encrypt them)(protect cache)

	name checking (utf8)

	event loging > how many dns can monitor

	debug log

	in forward and deligating :
		from bottom to up > forward 
		from above to down > deligate
	
	in child and tree must set primary or parent dns address

	we should define each dns with conditional forward together

Active Directory :
	ldap : port 88 (lookup)
	kekrbros : 389 (authentication)(tls works with this)

	dead time is 3 hours

	if clients has ping of ad we can launch our policies 

	need computer name
	after instalation must update system
	each domain has many pc and forest (forest has same name with first or root dc)
	admin of forest is admin of all dcs
	trust relation : dcs must replicate with each other and our users can login in other domains (federation)(holdin applied)
	in same forest maybe some dcs can't make connection to each other except users

	in ad dc adding role :
		1 : one dc on same domain for redundancy
		2 : one domain add to same forest
		3 : add another forest 

	functional level (maximum services can be provide) :
		in domain we have dc for redundancy launch another one if one of them be 2012 and 2016
		type of services can be diffrente for solving that we should sync our 2016 os with 2012 os levle
		in forest if set on lower level , lowest options are provide

	active directory and dns must set and install to each other on same vm
	dns can bee set seperate only on cache only servers and public domains

	tehran provines (forest) : tehran city (dorest root domain and domain)
		has accessablity from top to down

	dsrm (directory service recovery mode) : 
		for backuping ad must stop ad service , need password, password is P@ssw0rd! (we can use it in safe mode)
	
	when our dns get crash we can login with netbios name
		upn > u1@iran.ir = netbiosname > iran/u1
	
	if had :
		u1@it.tehran.ir
		u1@it.rasht.ir

		if need to loginwith netbios name :
			it\u1 (which u1 in wich city and access level)

		on these situation our dc change usernames with not duplicated components

		after addc role installation must goes to ad users and computers change admin valuse on upn and netbios name and fill blanks
		then checked dns console and forest , domain root forest  
		active directory database saved  our dns record

		in admin login on pc must login like (domain name \ administrator) if login like (administrator) login like local user of pc

		it's better use a different user for changing clients password

		it's better change administrator user to adm and another name

		must sync clients time and ntp with addc ntp

		to resolve and checking data of dns must goes to :
			dns console and properties of domain
			name server must checked 
			and resolve ip

	log database :
		is a file can store our datas as same as in database when our db backup get faulty can retrive our data from this
		it's better use san storages.
	
	sysvol :
		apply our group policies for all pcs in our domain also executeable files will transfer on this section
		better seperate from drive c

	preferences :
		move cracked files to client paths

	adfs (active directory federation system) :
		google use our domain informations and login our users to google with it replicate with our dcs.

	in view option and advance we have more items to checked
	here have dn value starts with cn 
		cn > grooup or object
		ou > name of group or collection
		dc > name of the group or domain
	
	dns zone integrate
	general add phone organization

	at the first we use forest and the second is domain

	*must create our users fater and earlier than their joining.

	in previous of win 7 sp 3 and 2012 r2 versions must set name in one time and join domain in another time

	container (no rules)
	organnization unit (group policy) : hase rules have star

	sid = dna of users in domain (x algorythem)
	guid = unique number in forest (y algorythem generated by x algorythem)

	windows + pause +break (system properties)

	10 last users can be cache on hdd an login like addc
	like u1-u10 if u11 comes to system u1 delete and clear from cache


	if employee goes out from complex it's better disable their accounts 
	active directory manage engine :
		computer 120 days
		users 42 days 
		
		must change their passwords
		passwords must set by users

	some time we bring our devices and join to network if user be admin can do it but in normal user deny

	adsiedit (all information of active directory)(must make connect to domain on properties)

	ms ds machine cota > how many logins can be loged device with one user
	
	prestage :
		before joining to domain enter to the domain.
		on the path contain > computers
					add new (set who can access like this)
					properties of users (where his can be login)
					policies (which pcs can be accept this users)

	if  enable (protect container from additional deletion) on ou making what will be happend ?
		we can't change and edit if disable it

	store password assign revesible encryption > must disable

	role base administration > group must set for users

	we can change log on to value for users in properties to access with authentication procedure in background
	better use for our native sharing servers and services

	cmd > wmic :
		baseboard
		cpu
		memorychip

	c:\windows\ntds

Group Policy :
	in group policy just checked changed items not whole or entire the policies
	gpedit.msc > local
		system center configuration manager 
			sofware 
			windows (give policies with command executes)
			administrative template (files transfer)
	gpmc.msc > server and addc advertise policy

	policies take effects on ou groups , group policy ou (gpo)

	it's better in creating groups set default values and leave on global sec

	better disable store password using revesible encryption

	on gpmc.msc :
		computer config > windows > setting > security > account policy :
			lock out duration 30 min
			locked at threshold (better change after running active)(default is 0)
			locked counter after

		computer config > windows > setting > security > account policy :
			password policy
			default domain policy > password policy
			minimum machine account password age
			better set cached accounts on hdd on 1
			don't display last username (prefere applied on callcenters and shifts)

		windows setting > security options > local policies > user rights assignments :
			allow log on (must add administration users)
			access this computer from network (limit access to use this computer altho authenticaiton)

		computer config > admin template > system > power maager :
			sleep time setting

		computer config > admin template > system > removeable devices

		default domain policy > computers configs > admin template > system > windows time serve (ntp)

		*do this for default domain controller policy as the same path

		we should shutdown firewall on clients or make a pass through role
		shutdown -i (remote shutdown)

		we can change local admin user and in sysvol folder is visible
		preferences > control panell > local user

	default dc policy effects on ou and computer accounts
	define domain policy applied to all 

	groups dosn't mean in policy

	we use groups to set permissions

	admx is manager of applications by group policy
		for admx instalation we should download admx and adml (language)
		add the files on c:\windows\policy definition in dc computer

	new > package (computer > policy > software)

	if deploy our application policies on computers all clients can use it but deploy on user is different
	in control panel :
		on user mode :
			deploy option :
				published (notify to user and let user decide to install)
				assigned (after user login install)
				advance

			on remove mode :
				every one use it or don't delete it
				every one is use it no problem but new users can't install

	gpupdate /force (by default each 180 minuts use this option)
	w32tm

	sandbox

	policies take effect from details to whole components

		enforce is positive : 
			disable effects from details to total , effects on subsets general has more power
			after do this first enforce checked then our ous			

		if have block some times if enforce that have more priority and power from above

		block inheritance is negative:
			cut the relation but for enforce cases can not do any thing
			delete all related policies and deploy ou policy

	one ou can have more than one policy and can set priority of policies for it

	if had 2 ou in one object with same priority, to change it must goes to root ou and linked group policy object (change it)

	if need to add similar structure must :
		link to exist gpo + group policy objects

		we should add in this field our domain computers , better be together our user account and computer accounts

	here we need give special access to managers in each ou what should we do ?
		mkae a gpo
		enforce it 
		then applied policies
		add special users in security profile and filters
		add domain computers

	security filter > each object goes to authenticate user can take effect of  that, computers account also have been there
	
	wmi filter > db of all soft and hardwares that are in each ou and wmi folder (cimv2)

	how policies applied to clients (steps) :
		it policy > network policy > data center policy > virtaulization policy

		first checked virtaulization unit policies then goes to up and it then default domain policy
		if not config set our policies step by step goes to root

		the last step of policy checking in clients are local and default policy in pc

File Server :
	smb : server message block (transfer files on network system port 445)
		qnap
	
	unitesize > filecount
	unitesize < filecount

	sata > r&w half duplex 22 pin
	sas > r&w full duplex 22pin + morepins

	nvme > connectivity protocol (fastest)

	permissions :
		share permissions 			ntfs permissions

		networks						local
		(network)(nas\san)				(hdd\ssd)

		first checked network policies and permissions then localy

		in sharing files we should use five access option also security tab can use

			in security tab must see :
				creator owner
				system (os)
				administrator
				users

			permissions inheritance if be black means they are localy if be gray means are inheritance
			in advance tab on security tab we can disble inheritance
				remove
				convert

				replace all permissions ......

		better share a folder and add domain user permissions , have all access, in advance we can disable inheritance then give per user access

		owner if take his account out of system can take access 
		for admin user > advance > change > group administrator (replace owner on subs container and objejcts)(admin will be owner)

		for sharing best practice is  server manager options :
			smb (advance)
			nfs (advance)
			quick
			application (service)

			access bas on .... (if some one has access see folders)
			allow caching of share (not recomended)

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

	mbr hdd formating system supports under 2tb cap and gpt supports upper than mbr.

	//pc105-1/c$ (administrator shares)

	newfolder$ (hidden share)

	dfs (distributed file system) (shares in some lines and many fs , server replication)

	name space : instead of fs name we set domain name for redundancy

	fsrm (file resource manager) reporting , file type filter , *qoata

	server for nfs : the best way to share our files between other oses

	work folder : onedrive company (smb over ssl)443

	\\fs\share\%username% (add for each users drive in explorer)

	cmd :
		net use w : \\fs\mail
		netshare

	to create map drive:
		dsadd user cn=u1,ou=test,dc=rayan,dc=.com -upn @rayan=pwd123

		save it on .bat 

	roaming > tansfer all profiles to one folder without seperation (copy profiles to hard (sync)) create folder + this folder only  /appendfile
	folder redirection > directly works on fs and deeply (create folder + this folder only  /appendfile)

	in group policy we can set the share able folder :
		userconfig > policy > folder redirection 

	enable win 2008 moe (make loadbalance)
	staging : temp folder afte sending
	cache duration

WSUS :
	.net 3.5 must be installed on devices

	online source on wndows disk or source :
		dism /online /enable-features /featurename:netfc3 /all /source:d:\source\sxs

	we can run wsus on 2 mode:
		sql server (large companies)
		wsus db (small office)

	sqlserver network configuration > protocols for wsus database > ip > port tcp 1433 > ip(3,1,2,4) enable (yes)

		better install our large sql db on another os

	run > service.msc > sqlserver (restart)

	in computers policy :
		computer configuration > policies > administrative templates > windows components > windows update :
			specific internet microsoft update service location(enable)
				8530(port)
				https://wsus01:8530

		time : conf auto update

			better enable other option at the end page

		for norm-users notification : allow non-admin
		don't connect to any windows up internet location (if wsus was down users can't check microsoft)
		wake on lan
		auto update detection frequency (check updates internally)

		clients must connect to wsus with :
			powershell > wauaclt reportnow (client side)

		in wsus we have iis service (web base) must be always up (check ping , idle , memory)
			wsus pool \ wsus admin > advance setting
				private memory : 0 (unlimit)
				reqular time internal : 0
				ping enable : false
				idle time out : false \ 0

				with 8gig ram handle 5k clients

		catalog.update.microsoft (ie)
		cd /programfile/updateservices/tools/wsusutils

	better disable upgrade option 
	and just use english

	in wsus console must set :
		status > any \ fail or need
		wsus > computers
		approval > unapprove

		then select supresedence (version problem)

		must install no icons

		automatic approval (defender)

		groups :
			unassign
			group policy > addc > enable client side targeting

		wsus server clean up 

WDS:
	install windows over lan
	if our wds server were same range with our clients get files 
	if not must defien wds with dhcp server to clients

	must setup a windows entire apps updates .....
	before these we must make image of windows with full setting
	then sysprep that and capture it


	bootserver : wds server ip add
	
	bootfilename > file
		boot/x64/pxeboot.com

	pxe : allows users use online os installation

	bootimage : sysprep + capture image

	export  -windowsdrivers -online -destination c:/.... attend > answer files

	discovery image : if our computer don't support pxe boot must boot this on flash then till find wds

	add deriver package : add nic driver in boot file

Forest & Trust & Site :
	forest :
		forest make connection between many dc and domains
		each domain in trust are acccessable on each other
		in domain we have dc and dcs in some big sites have replication and load balancing ,if need get priority we can set weight to them
		if dns of dcs take crash our addc don't work
		
		primary dc > we can see and it is the master form
		additional dc > primaries copy 
	
		if primary dc get crash our forest get crashed
	
		relation between  domain and users are 1 : 2 or opposite on normal user level except permission access(the principle task of trust and forest is this)
	
			linked dns with conditional forward
			domain to domain
			forest to forest
	
			two way (authentication)
			one way :
				income 
				outgoing
	
			this domain only
			both domain only
	
			trust authentication level (authenticate all domain users)
			suffix name > attache domain name to users intarction
	
		if our forest root domain be A and have relation in forest with domain B, in transfering data between a to b our data transfered entirely from a to b
		if A crashed all forest get down so must set redundancy or backup for A like A'

	site :
		if our server on site a with forest A and many domains get down what we have do ?
		forest is logical concept and site is physical
		perforest define a site
		also specific used for forest
		dc users can access each others

		each client requests handel by self site domain

		eah service support active directory use site concept

	dc count = kerbros count (cause use round robin)

	like client joining we should join new dc to native dc and set dns address on native dc
	in ad installation must select role and features first option (add dc to exist domain)
	in first section we have replication, work like backup and export (offline mode)
	in dc must set dns ip address on self ip and remote dns ip address (dns + active)

	server > dns manager > properties > name server
		we shoulld see all dc ip address if not exist must add them
		for forest and domain
		for clients better set 2 dns address

	replication for microsoft consoles are 15 seconds

	for relation of forest and domai we have models :
		trust - trust
			better see each other 
			many domains of forest a can see forest b

		domain - domain 

	we have authentication :
		2way
		one way income (remote side)
		one way outgoing (remote side)
		this domain only
		both domain (lazy mode of configuration) (need password)

	trust authentication level

	activedirectory site and services (ip and services)

	each site must have ip address if don't set organize them iin default path

	why use site :
		1: users distribute
		2: site replication control
			for same site use real time replication but for other zones use x15
				interasite (internal)
				intersite (external)

	for change time ,site link must be create (include schedule and cost)


	replication action just sync new data and need many links (ip site link)

		*if we have 2 server that need replication and one says do it another says not each site has more weigth than other has periority
	
	must use mesh topology

	for hub and spoke senarios must set specific site ip link for customization
		if have many domain controller in forest our site data get replicate

	replication has kcc (wich server has light process weigth) option use faster way to replication routing
		for changing :
			active directory site and services > site > subnet > server 

	forcing replication (cmd) :
		repadmin
		rep admin /syncall /a /e

		active has many parts for databse:
			1: forest dns zone
			2: domain dns zone
			3: schema
			4: domain
			5: configs

	domain dns zone :
		dsn + ad (if install together in same time we find in this folder)

	zone dns zone  :
		server zone (more secure and easier like domain - in replication we replicate this)

	schema :
		forest dc
		en=schem,cn........(object)
		configure in domain get replicate

	configuration :
		information of related applications with ad

	in each forest we have one unique dns record that replicate doman partition

		repadmin /syncall /e (external foriegn dcs + domain partition) 
		repadmin /syncall /a (internal in same forest sync all dcs and databases) 
		repadmin /syncall /e /a (internal + external)

		repadmin /showrepl (what data are transfering)
		repadmin /replsummary (untill this moment what's the status)

	globalcatalog > ntdss > properties :
		it's not a master roll but it's like db and when we are looking to some ranges we can use fast index searching
		like summary of objects
		at minimum must set one per forest

if didn't set ip for each sites take them to default site (site0) that all of the our dcs are there

RODC (read only domain controller) :
	replicate every thing except passwords
	user for small city that use less than a huge amount of users
	for authenticaiton send requests to another dc
	save them in small size and in ram
	it's good install global catalog in rodc

	deligate admin user (access to admin for that specific part not whole ad)

	in next part must set allowed users for replication :
		except admin groups we must set every thing

	adding password in rodc :
		dsa.msc > domain controller > rodc1 (properties > password replication poicies > add (allow/deny))

			just do this on rodc
			just send encrypted password of specific users in replicated time not more

	ntdsutil > set dsrm password

	group scope :
		domain local :
			1: in self domain range access
			2: joining range on forest range

		global :
			1: access in forest range 
			2: joining in domain range

		universal :
			use 2 above options but screwed up global catalog

		after create can change categories but first moves to universal then delete some datas
		group type > security > policy + permissions
		distribute > ha..... (some applications for organization inside security)

Master Rolls : 
	fsmo role (better be on one dc )
	counted 5 and seperate each other
		master (means played on one primary dc) :
			schema (per forest)
				kernel on active directory db
				no console
				advance view

			domain naming (per forest)
				name structure of forest
		
		additional (can accessable with 2 dc players)(active directory computer & users > right click on domain (operation masters)) :
			rid (per domain)
				use for sid making
			
			infra (per domain)
				domains agent in forest said from this domain to another domain what permissions are valid
			
			pdc (per domain) 
				time setting 
				
				computer browsing maybe in same broadcast domain or diffrent vlans 
					services.msc > disable
					by explorer > network
				
				each dc must checked replication timming and informate it to others this action use pdc
				
				password manager for domains 
					replication problem in delay between transfering each changing must advetise to dc, before reject must checked dc

		regsvr32 schmmngnt.dll

		for master rolls changing must opened our inputs and outputs 

		if master rols owner get crash : seizing must be happend and never give back to last system

		netdom /quer fsmo

		ntdsautil (seizing)
			:active instace ntds
			:roles
			:connections:connect to server dc2
			:q
			:seizing infra master,pdc,naming master, ridmaster,schema master

		netdo \query fsmo 
			:meta data clean up
			:select operation target
			:list domains
			:select domin 0 
			:list site 
			:select site 0
			:list server in site 
			:select server 0
			:q
			:remove selected server

		now clean our dc
		for inject another dc must install again new windows

		dsa.msc
		
		msds-password setting container 
		password policy objects:
			adsi.edit
		activedirectory administrative center :
			system > password setting container

	Upgrade 2012 > 2019
		inplace upgrade:
			master rolls take transfer to ne dc
			ntdsutil :
				transfer online dc
				seizing > offline dc
				then unistall roles and features then depromot on 2012

CA :
	msa (heart bleed > backdoor in sla)
	ssl stands for secure socket layer(ssl get expire)

	tls
		https = tls + http

	rsa > support all formats (authentication)(hardware locks)

	certs > it is a certificate that used by tls (id card)

	without changing of their content encrypt data and other protocols 

	steps of certificate checking :
		1: is it same typing text with google or web content
		2: who is the publisher
		3: valid date
		4: issued

		cmd > certlm.msc > list of certs

		crl (if our certificate get hacked publisher transfer our cert to invalid certs)(offline list that checked online)
	
		it's better install on seperated server but on ad has more options

	instalation ca server :
		root ca 
		roles service (certficate authority)
		certificate authority web envirement (online cert)

		roles that must install on edge and another server (os) :
			1: certificate envirement web service
			2: certificate envirement policy web service

		these act like proxy

		it's better run enterprise certificate server must join in domain ,no admin local just forest admin can do this

		beacuase in enterprise after users joining their certs automacally get install and have template

		in same forest and diffrent dcs must run root ca to easly access

		we use enterprise mode beacuase have tree structure and install certs on clients in joining progress

		in manually mode or standalone we have limited categories and no template also need approve from admin 

		in progress of instalation if we checked allow admin interaction when the private key is access ....
			can use our certs in old windows agian in new windows

			*this option need protection

		revoke certs (invalid and expired ca)

		issued (valid and usable certs)
		
		pending 
		
		templates

		in web ca we have unique ca for each users

		ways to take ca :
			certlm.msc (os and computers)(alot certs)
			certmgr.msc (users)(afew certs)
	
				each of them has presharedkey and private key

				os deny private key export

		for web certificates we use iis to make more secure transfer and our networks accessablity
		must setup in another way because can add infected items

				they are same but has many diffrente attributes
				cert name is not important but maybe have conflict
				for dns must set cname in iis

		common name : web.test.com
		url : web.test.com (it's better use this instead of dns)

		after runing web cert must run iis server

		in iis > right click and select edit binding > 
													1: cert user
													2: cert computer

													we can use one of these it hasnt differences between them.
													web > has edit option
													computer > hast edit option

		if our certs for web did't match with our certificate take an error must select proceed....
		we have version for erts and hase options on each versions
		for cert joining to users :
			group policies domain > selected users > computers > windows > security > pulicies > automatic certs

		in web mode must work with https  443 (turn off firewall)
		in web mode we can take certficates :
			https://192.168.1.100/certsrv

		lightweight ad > empty db don't need domain for light works

Cluster : 
	for special service runs on many servers ,has 2 mode:
		1: load balance
		2: failover

	failover :
		behinde of this type has san storage and shared storage (don't have loadbalance)
		inside of server we have vpc and works in 7 (application layer) this vpc data saved to shared storage use ip and mac of server

	loadbalance : 
		nlb : have 2 server that use one id like same mac and ip but didn't support replication 
		don't need ad ,can run in work group
		we can add our selfs to nlb
		cluster ip is virtual ip
		we have priority
		can use 32servers
		vip (convert ip to hexadecimal and make mac add)

	we can use in features

	if unicast happend , goes to switch , advertise to all systems
	if mutlticast happend , goes to switch , advertise to one systems

		affinity > 
			1: none (loadbalance (weight) + failover)
			2: single (failover)
				we can set value if didn't second server goes up fast

		after apply can accessable configuration

		on second server add host and if make dns record or differente data with single mode detect

		iscasi (san in application mode)

		witness > on share > iscasi initiator

		failover give ownership to owner of process

		in fail over :
			1: ip
			2: mini windows or computer name
			3: disk

			each service need mini windows

		right click on cluster :
			more action > config cluster .... > witness (select hdd for witness)
			
			move to next one > moreaction > move cluster resource

			evic on more action means delete

		for distroy cluster must shut down cluster

Hyper-V:
	has cluster prerequisites
	must run virtual machine on cluster

	must select expose to enable hyperv mode

	instal all vms  on same iscasi

	allow manage .....don't use econd card must unchecked the option

	what is the diffrente between cluster in vm and role :
		in vm we have little down time but don't have dependencies to services
		in role just have mili seconds

		we can add cluster with failover custer manager and ad together (roles > virtual machine)

		we must create for each vm a single and seperated disk

	csv : cluster shared volume
		we can set shared storage and witness for each vms and use per vm hdd but use csv
		ntfs makes our hard accessable with one admin or user but csv makes it wide access
		create each item in csv is like hdd

		hyper-v use movation vms without downtime or turning off option (live migrate)

			live migrate has one ping lost and crashing host take 1 minute lost (unmonitord)

		if before clustering had vms must set move to folder and start roel virtal machine

		priority (lower is better)
		
		cluster > properties > balancer (devide our resources)

	broker : use block of data and transfer it faster than other options

		roles > config role > hyper-v > replication broker
		
		enable replication > broker add + server + https cert

			5 min eriod
			have compress option
			one port or many ports
			has failover backups
			replicate must set on brooker

			must enable replicate on vms
	
	for disaster systems admin must do it manually (failover replicate)
	if get hangged must create site
	port 3260
	services.msc > microsoft iscasi....(restart)

	if first site crashed and down mut create new replicate (our vms are active in each site)
	in site 1 must make them stop and remove replicate then on site 2 must reverse replicate to transfer all data from 2 to 1

RDS (remote desktop service) :
	snapin (run > mmc , snappin is management mode of mmc)

	in server click on features and :
		remote server administration tools (most of the required options are inside it)
	
		group policies are above of this option
	
		some snapins don't ask wich scopes must connect them to

		for clients must install rsat

		admin local can do this

	system manager > remote setting (allow)

	in firewall has some roles but can't access on internet or other vlans ....
	need certs

	rds > port 3389 tcp udp and nat , use clipboard option and vpn

	windows admin center (wac) > port 4443 > firewall turns off (diffrent 80,443)
		we should set gateway server on this port 

	wac > extention > mmc > snapin

	each profile in remote desktop role installation has a profile and seperated roaming space in hdd like virtualization
	it's diffrente with vdi
	license for banks are cheaper

	after 1200 day need license

	rules :
		1: rdsh (session host) :
			desktop and programs run on this and need source alot, we have many servers with broker make loadbalance for sessions

		2: rdcb (connection broker)

		3: rdwa (web access) :
			clients connect to access points and connects to wellcome screen or panel and tell to users wwhat can use

		4: rdg (gateway) :
			access us from internet and play proxy roll to transfer datas , has udp session and ssl encryption , use in dmz to hide our lan 

			*we can install all above services on single server except rdg that need seperated server

		in ip public using we use 2 ip address :
			1: ip > rdg
			2: ip > rdwa

		installation :
			remote desktop services :
				1: standard
					distributed mode > server manage > add another servers

				2: quick
					1: session base (rds) 
						we can't use linux oses in this mode

					2: virtual base (vdi)

			between our dns must set forward zone and conditional forward

			must set certs per hosts

				remote desktop services > task > deployment (properties) > for license must install feature rule

			in normal mode somebody connect to server can use our service :
				overview > collection > quick ses....collection > list of programs

			rds.test.com (space) /admin

		in controll panel we should set rdweb address
		
		must install ca :
			ca server > web server (enroll permission)

		rds works on server manager and must set pfx , in ca (make export..) with admin access
		now export ca and import it to (overview > task > deployment) and apply it to all servers

		in clients > 192.168.1.100 \certsrv (install certs)

		this remote desktop access user to use our sources with out exporting or seizing data (dlp)

		file association (we can open known type files in rds) :
			in server > file association > client setting (drivers + clip board)
			if our file share and san didn't work we can use server processor and save it localy in our hdd.

		on server (session broker) :
			install-module -name powershellget -force
			install-module -name rdwebclientmanagemet
			install -rdwebclientpackage
			import -rdwebclientbrokercert (cert address)
			publish -rdwebclientpackage -type production -latest
			reboot
			
		public key > certs for broker
			certlm.msc (export it)

		we can run our rds web on html5 :
			https://rds.../rdweb/webclient/index.html

		ip 1 > 443 to > rdweb
		ip 2 > 443 + 3392 (stream udp) > rdg

		after adding rdg server must set cert and make A record in dns.

		*scom*
		performance > user define > new > data collection set > monitor network microsoft. 
