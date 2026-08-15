BVI (Bridge Virtual Interface) :


bridge concept :
	
	is like repeater
	used for segmentation
	save src  mac to ram
	works in datalink layer (osi) 
	combine each media type together
	use routing tables not like a router
		if have des address in routing table and be in same segment with src address the bridge don't look at data package

		if does'nt have des address , the bridge send data package to all segments except src segment

		if has a des address in routing table and be in different segments of src address  transfer data package to des address.

	lan connectivity to bridge :
		1 - transparent
				just used for ethernets support
				self training

		2 - spanning tree
				just used and create for improving first option

		3 - source routing
				used in ring topology


why we use bridge ?
	security
	bandwith
	reliablity
	translate or nat

for connectivity beetwen each bridge use bpdu option
	(BPDU: Bridge Protocol Data Unit)


enable bvi option :
	bridge irb

add interfaces to bvi zones :
	interface gig 0/0
	bridge-group 1
	interface gig 0/1
	bridge-group 1
	exit

enable spannng tree like switch :
	bridge 1 protocol ieee

like svi in switch we can set ip and mac to our bridge virtual interface :
	interface bvi 1
	no sh
	ip address 10.10.10.1 255.255.255.252
	mac-address 0000.0000.0001

for use layer 3 options must set this command :
	bridge 1 router ip


for test make some loopbacks on device

routing protocols can be run on bvi
