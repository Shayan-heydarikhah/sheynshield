VoIP Issable

pstn : general phone switches netwrok .
way of connections beween cities and big countries is long heal network.

analog phones has rj-11 sockets that use one pair of tp  tip and ring.
from pstn switches transfer -48 dc to local loop (house).

exchange area network > way of pstns in city.

chassi under earphone named hook 
if pickup earphone > hook on
else > hook off
for transfering numbers to each switch must use trunk lines.

informatiions between our phone and pstn switches are :
1 : supervisor 
	1: start signal
		local loop -48dc
	2: ground start	
		glare (sometimes we pickup earphones and hear sound of other peoples or free line sound)
		for prevention of glare tip and ring get short-circuite connection.
	3: ringging 

2 : addressing signal 
	a signal said who are u looking for? rotary or palsy 
3 : information signal
	hearable messages for customers.
		1: dial tone
		2: ring back
		3: busy signal
		4: reorder tone

private brach exchange (pbx)(feature rich)
disa is like ivr nd can set 24 disa in some models.
panasonic in tda 100/200/600/620 has memorry expansion card or mec.

aterisk > linux base /etc/aterisk
gui > free pbx
		issable >voize 
		elastix > vaak
core of oses is centos and services are :
openfire 
reporter 
hyla (fax)
postfix (email)

sterisk has backtoback arch and support 300-400 users call in same time and must not run 
skype or video conferance on it instead of asterisk can use ocano skype or webex.
for skype and .... can use proxy server like sfb or sip proxy to handle 10k.
asterisk can connect 2 diffrent codecs to each other.

sip (set initition protocol) :
a signal protocol that set or register internal code to ippbx.
all of the operations like call,brek,send,resives..... use sip.
after making connections betweenserver and user we use another protocol :
rtp (real time protocol) that transfer any data also sip control connections of server and user.

sip has 2 model : panasonic and general. 
hardware of phones is not so important compatiblity with ippbx system is more importer than.

fxs and fxo :
fxs is like ports on wall and has electerical flow (bugle) when connect analog phones to them bugle start and AD and DA
(convert analong phones to ipphones and after gateway works like voip)
it's like switch.
must set qos and has less cost instead of seperated ipphones.
it's need voip converting.

fxo doesn't have bugle , ports are on phone isntead of wall and can connect server to pstn like normal phone.
then convert pstn signals to voip and give them to elastix.
gateways :
	1: pri
	2: e1
voip translation language.
gsm gateway : 5vol can setup with pci or pci express convert gsm to voip.
directly convert to digital because use tcpip stack. angoma u200

connecting ippbx with normal pstn lines should :
	1: buy sip trunk lines 
	2: fxo gw or fxo cards
gateway is better than cards because can goes far and take effects on server loads.

converting analog to digital has resampling :
double size of pick of waves : (0 + 4k)*2 = 8k
each samples in 125 msec 
with molla method we can set 8k in 8bits and use 64kbps of line for good sound.
for better sound and low usage we should set codecs like :
g.729 (hd) 8k
g.711 (no comperessions) 64k
g.728 
ilbc
opus

sccp (sciny control protocol) like sip in cisco company.
iax2 (inter asterisk exchange 2) for asterisk and elastix
mgcp
h323

when run issable in /etc/asterisk set some values and read them from freepbx
/etc/sysconfig/network-scripts/ifconfig-eth0

in issable must set some parts for users can access their own data :
1: voice-mail
2: call-recording
3: fax-viewer
4: reports+cdr

it's better set group and then add each extentions or users to group.
if bee admin and don't have an extention can see all data of all users.

pbx > config > extention 
	can set on ipphones + fxs + softphones
	for cards can set ip phones + softphones (generic sip devices and dahdi devices)

<id =display-name + extention-number>
"karimi"<2000>
callerid

line > poshtekhat
account > inter network number

play cid in voicemail means read and play number of person send u mail.

ways can access voice mail is :
email
web and isaable ui
feature codes
	*97 (my voices)
	*98 (ask internal and then play voice mail of him or his for this must set pass)

network > network paraeters 
	can set ip static 

for change password of issable must change 3 password fields:
	/usr/in/issable-admin-password
	amportal restart

in issable we don't have persian must set it in freepbx :
unembeded issablepbx
security  >advance setting
enable direct access (on)
setting > aterisk sip settings
	language pr (persian)

for iax2 also do this :
pbx > tools > asteriskcli
	reload

recording :
in recording option on each extention have 4 type of recording  type :
1: inbound
	1 internal
	2 external
1: outbound
	1 internal
	2 external

don't care > when takes effect that on demand record get active. (by defualt dont record exept times enter feature codes)
*1 > enable and disable recording.
if we don't enable recording mode > never get enable.
don't care use cases in organizations and less data storing. 
record priority policy is a number that can change conditions in recording and policies.

for join smartphones or external organization devices to our voip system must set some nat in our edge device:
1: destination port 5060 tcp to issable 5060
2: destination port 5060 udp to issable 5060
3: destination port 10000-20000 to issable 10000-20000
then:
pbx config > unembeded issable pbx > setting > asterisk setting 
add ips of lan and public.
then:
set in extention nat option.

2fxo + 2811 is more econamical.

for tshooting issable codec conflict :
asterisk -r vvvv

for activation of codecs :
unembeded issable pbx > setting > asterisk sip setting

to see registered extention:
pbx > tools > asteriskcli
	sip show peers

call transfer :
phone or operating panel
when we are talking to some body we want transfer it to admin.
if it has analog phoone or fxs :
1 attended
2 blind

## wait for seconds to insert new number and ends u if u don't enter call back to u
*2 wait for seconds to insert new number and music on hold plays to customer and bugle for us ,we connect to admin if want answer we hook off and take down earphone and customer connect to admin.
*43 echo test
*69 calltrace

ways connecting our internals to pstn :
1: fo and issable
2: issable connect to gw
3: internet and sip trun lines 

digiam cards has driver zaptel 
sangoma wanpinpe
lspci
atcom
sometimes don't detect asterisk
wancfg-zaptel 
	yum -y update
it's better don't checked dahdi option in manual must set for replacing.
issable > system > hardware detector	

for looking recorded files in linux issable should go to monitor/xmonitor and voicemail.
cd /var/spool/asterisk

for calling out of issable area must set :
1: fxo and pci card
2: gw
3: sip trunk lines

how config the defined card in isaable :
1: trunk 
	trunk set all of the connectivities to foreign places.
	types of trunk:
		1: dahdi	(card)
		2: sip trunk(gw / other issable)
		3: iax2 	(other issable)

2: outbound routes
	tell to issable which numbers must set for destination

3: dial patterns
	detect and  find wich replicated numbers mean.
	like 110 in inter network or 110 police number.

dial pattern used in outbound routes and we can set 511 (tabriz city code number) at the start of 110 and when we want call 1100 (police) must call 511110 with dial pattern we hide or clear 511 and our dial number goes out of issable network and call 110 (police) no 110 internal.

4: inbound route
	cid (caller id) / did (direct input dial)(number that we dialed)
	first of all must set inbound routes to define issable how route it and connect to which destination.
	1: cid
	2: did
	3: destination

inbound route works on did forms.
for connecting to pstn llines :
make trunk (dahdi) by default has 1 trunk and must not delete that 
pbx > config > trunk
	dahdi identifier > g0 (group 0)
	in connectivity withdahdi interfaces we have 2 files:
	dahdi_channels.conf  seperated config of ports on card
		signaling fxs-ks (fxo) / fxo-ks (fxs)
		group 0/1/2....
		g0 > all of the ports on card is group 0 if change get conflict ,don't use round roubin for free lines if use 4 ports and person number 5 comes online can't access the system we should define another group like g1 and connect it to issable with dial pattern:
		dahdi_identifier=g1 
		import one line to g1 and :
		dahdi_identifier=r0 (it's g0 with round rubin algorythem)

trunk name doesn't any matter but outbound caller id is important.
outbound caller id > "rayancollege"<4821>

	chan_dahdi.conf 	general config 
		busydetect yes (busy is signal that clear our lines if get 5 clear line)
		busycount 5
		rxgain (incoming from pstn)
		txgain (outgoing from issable)

		must go to asteriskcli and reload it.

private numbers :
when we are going through trunk to outgoing line our cid take change pstn get access on sip and e1 lines to change this option.
cid option :
by default is allow cid and transfer our cid through trunk line except private numbers.
force trunk id (private number)

call counting  in sip lines depends on bandwidth.
edit dahdi trunk
cid option > allow any cid
in issable we have rounte no sip line.
if cid from foreign of system and wants make call to admin in the way with trunk moodd  shows our cid  if block foreigncid check and set outbound caller id shows itself id.

hide our internal codes and make call with organization general number.

continue if busy
	check the a.....
	send calls to next line
trunk sequence 
	
dial pattern > dial number  manipulated place
outbound rounte is the best place for writing.
issable :
	using card > trunk
	define routes > outbound routes

	1: city 
	2: between cities
	3: mobile or phones
	4: international
	5: with passwords
	6: vip
	7: all

adding patterns comes with adding route, 
	prepend + prefix | mactch pattern /cid

we have regex.
x > 0 to 9
z > 1 to 9
n > 2 to 9
. > any numbers 0 to 9
[] > each specific numbers in []
| > delete any left side

in the end of list we have some trunks.
if we have two same number just called internal.for solving use prefix.
it's like acl .

we can set pass for foreign calls access.
	1: pbx > pbxconfig > pinsets
		define them to outbound routes.

	2: in outbound routes (match pattern) set 090 and null in trunk ,optional destination and anouncement plays sound.
		if admin wants make call can set pinless dialing.

set a special line to vip calling:
	outbound routes
		caller id  200x
		match pattern 912.....
		prefix 9 (foreign numbers)
		prepend 0 (add this)

		this values make this call to this path enable and accessable.

outbound routes recording values :
	call recording :
		1: allow (extention properties)
		2: never (never)
		3: record on answer (record after answer)
		4: record emidiatly (record after hook on)

when inbound routes does'nt know how handle our calling :
with did (fxo and general organization number) + cid > if get null means any
set dest and pick ivr .
e1 and sip has did option.
hotline bind :
	dahdi channel did
	channel 2 (in hardware)
	did 4444 (virtual)
	now set in inbound routes whom comes with 4444 take it to ivr....

	beacause in isaable we don't have an did for fxo lines so :
		dahdi_channels.conf
			channel=2
			context=from pstn > context = from analog

backup and restore :
database
config files
menues and permissions

or all

except :
voicemail and monitors			

ampportal restart

for reset pass :
restart and enter "e"
linux 16
ro > rw
rw init=/sysroot/bin/sh
chroot /sysroot
password root
123
exit
init 6

for admin > /usr/bin/issable admin password change

call transfer :
	while talking a person transfer it to another one
	if we don't answer and transfer it it's mean forwarding call.
pbx > config > feature code
	call forward all (transfer in any condition)
	call forward busy (if busy transfer)
	call forward no answer (if don't answer)
	*720 disable
	*740 cancel all forwarding

cos > class of service
pbx > config > class of service

if we have voice email and call forward wich one has priorship?
forward
optional destination is option like forward in extentions.

555 chanspy > we have isaable and no card random spy * goes next
888 zapburg > when we have card and firs ask our channel.

spoof special >
extentions_costam.conf
tools > asterisk file editor
add these commands :
[chaspy]
	exten => 555 , 1 authenticate (1234)
	exten => 555 , 2 read (spynam,extention)
	exten => 555 , 3 chamspy (sip/#{spynam},q)

trunk to card / another issable / gw :
trunk to gw is same but from gw to issable is diffrent
first is web console 
second is command executed result

outgoing :
user ="ali"
secret=123
host =(front view ip)
type =peer
allow g.728

incoming :
usercontext="ali"
type=user
secret=123
type=friend (without user and pass)

context is tag for issable to set how behave oour calls.
1: internal
2: trunk 
3: pstn 
4: analog
except internal all of them goes to inbound routes.

in some gw we if we have sip server or voip server ,can set inbound and outbound routes.
configure of gw :
	1: network 
	2: sip :
		1: register server
		2: proxy server
			voip provider like asiatech,shatel,pstn
			like firewall and bar counting
			they have 2 server to handle 
			have 2 ip 
			in routing part :
				ip [192.168.1.200] route fxo 1-4
				outbound route ^
			in each fxo we have id for inbound route.
			trunk > feature
			trunk > advance 
				gain to ip (issable)
				gain to pstn (tci)
			registeration check
			inbound handle > binding
			now must set inbound route on issable

ivr :
	issable > voice uploadcenter
	pbx > config > system recordings
		wav format 8khrz 16x pcm encode

		1: ivr
		2: customers
		3: queue
*77 record 
# end recording
*99 listen

announcement does aplicate with ivr together
pbx > config > announcement

return to ivr 
	turns back to ivr and dont go to destination use when have many ivr and shared announcement

allow skip 
	if dont enable must listen till end of voice

dontanswerchannel > disable

time out > till 10 sec insert number
direct dial > if knows internal numbers insert it and dont listen to ivr or announcement must set in exten
misc destination > out range number of organization can be use for destination (hide menue)
	invalid :
		1: retrive > comes back to ivr and many times is valid
		2: retry recording > a sound can be selected and say goodbye
		3: return on invalid > if enterd invalid number and comes from another ivr , get back on that ivr

	time out > like invalid options

	append announcement on invalid > play invalid sound and start ivr again
	invalid recording + destination > if invalid options take a frequented play a soun and say connecting to cleck.
	return to ivr after voice mail > usaully dont enable it but after ivr goes to vm

	in front of ivr we have return option can set and cames to start of ivr no destination 

transfer + forward + follow me:
they are same. on forward we cant forward again but we can set follow me (advance forward) and use ring topology.
initial ring time > after how many rings transfer it to another one =0 means treansfer it on list  at the end of list we can set # and trunk
ring strategy :
	hunt > sequence
	ring all > ring all of clients
	destination no answer > destination
	memory hunt > first 10sec + (first 20sec + second 10sec ) + (first 30sec + second 20sec + third 10sec )
	first available > check all clients diffrent with hunt (goes till answer)
	*huntprime > has priority if origin number was busy dont go to another one play announcement and say reasons.

	cid name prefix > say where are comming our call from
	alert info > specific alert

	*32 > block call

	call flow control :
		1: income
		2: outgoing
			1: default
			2: manual
		has fetaure code > *280

	time group > time conditional ivr and announcement
	
ring group > ring all strategy
	cid name prefix > where is call from
	ignore call forward setting > if checked call forwarding to some body gets ignore
	skip busy agent > for comunicated person disable line (poshtekhati) sound
	enable call group > our roomate can access and answer it
	change extention cid config > ci for calls
	fix cid value > variable

	ring group use for small office andd doesn't have application in large size

	queue is the best soloution for large offices.

queue:
	callers > customers 
	agent > clercks 
		1: permanently
		2: temporary
			we have pass for entering to queue.
		call center is place to manage our temp clercks
		generate device hint > blf 
		statics define on phones
		wait time prefix > how long time they spend in queue
		agent restriction > set how manage queue if we have follow me and forward select follow me option.
		no follow me and forward > if we have 200 and select 201 isaable forward it to 200
		ring strategy >
			1: least recents > low frequency activaty
			2: fewes call > not compelete
			3: weight random > random mode

			queue weight use for extention that joined in 2 queue and we want set priority for  it.
			make call answered
			maximum wait time > maximum time to answer
			maximum wait time mode >
				lose > if queue get full what should we do
				agent time out > how long clercks phone ring
				retry > how long wait and goes next clercks
				agent time oout restart > on which counting rings get reset
				wrap up time  > in break time dont connect any calls and transfer it to next one
				member delay 
				report hold time > report how long wait in queue
				max caller > maximum capacity of queue
				join empty 
				leave empty
				penalty > if clercks doesn't answer issable give weight on it

install module on issable :
	yum -y install issable-callcenter

	on issable :
		callcenter > config 
		current status > start (if change value must restart)
		define agents :
			agent option 
				agents
		agent where goes > queue
		ingoing calls > queue
			for using our agents in queue msut makes them static
			a9000
		call back extention > s9000

disa :
	in panasonic is ivr in others is free line 
	in issable if we have this number and make call to it ours get free.
	callback > caller if take break and call agian transfer to call agent
		soloution is vpn and zoiper
		for admin :
			call back > number of admin
			delay befor call back > 15sec transfer to disa need some seconds
			destination > disa
			response time out > how long wait system to get access
			disa pin 
			caller id > which number replaced 
			digit time out > delay of inserting number
			allow hangup > if finished our call break it or no?
			misc destination > 000000#
			misc application > feature code 788 
