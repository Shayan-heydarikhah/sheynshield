#linux  
tips :
ctrl+shift+f1 (real linux)
case sensitive
tab key ---> helper key
apt-get install openssh-client openssh-server
execute for directories means cd.

sudo passwd (username) (set password for users)

ls (list view) :
	ls -l (short view)
	ls -li (show inode)
	ls -a (hidden files view)
	ls *.txt (find and list all .txt file extentions)
	ls file ??.txt (finde and list all files with 2word .txt files)

cd (change directories):
	cd ~ (home folder)

cat (execute file):
	cat text1.txt

echo (print some lines or scripts):
	echo "hello"
	a=123
	b=456
	echo $a
	echo $a;echo $b (print in seperated lines)

get help with these :
	cd --help
	which cd
	man cd
	man -k cd (shows every where we can use cd )

less (open and show zip files):
	less a.zip

pwd (path work directory)

files handelling :
	-i (interactive mode , firs of all ask then process)
	file 	(show info about something)
	touch 	(creat a file and modify)
	mv 		(lice cut in windows and rename)
	cp 		(copy)
		copy -r (folders can be copy)(recersive)
	rmdir	(remove directory)
	mkdir	(make directory)
	rm 		(remove file)
		rm -rf (remove folder)

creat zip files :
for folders and contents archive and zip :
 	zip -r file.zp

files archiving :
	tar -xvf a.tar
	tar -tvf a.tar (show content without extracting)

	tar (just archive):
		tar -cf a.tar (creat tar file)
		tar -xf a.tar (extract tar file)
	zip (compress and archive):
		zip a.zip
		unzip a.zip
	gzip (good compression and delete comp file):
		tar -cvzf a.gzip (creat gzip file)
		gzip a.gzip
		tar -zxvf a.gzip (guzip file)
		gunzip a.gzip
	bzip2 (better than gzip):
		tar -cvjf a.bzip2 (creat bzip2 file)
		bzip2 a.bzip2
		tar -jxvf a.bzip2 ( bunzip2 file)
		bunzip2 a.bzip2

search and information extracting :
export results to file :
	cat -c3- a.txt > b.txt (creat new file)
	cat -c3- a.txt >> b.txt (append to file)	

	head 	(what append in 10 lines of beginning of files) :
			head -n 2 a.txt (2 lines of file will be show)
	tail	what append in 10 lines of ending of files)
			tail -f /var/log/message (shows every update)
	less	(like zip command but have scroll option (it's not editor))
	locate (find every thing that looking for and update own db by /etc/updatedb.conf use indexess)
	find 	(like windows search just for files not contents) :
			find / (like windows tree cmd command)
			find . -type (f or d) (shows files in same directory)
			find . -name "uni*"
			find . -type f | grep abc
			find . -size +3m (show files bigger than 3 mb)
			find . -atime +2 (show touched files in 2 days ago)
			find . -ctime +5 (show touched files in change mode)
			find . -mtime +10 (show touched files in modified mode)
	grep 	(globaly search a regullar expression snd print)
			(like find but have context and index search option) :
			grep will file.txt
			grep is *
			grep -n oo a.txt
			grep -i oo a.txt (no case sensetive)
			grep ^o a.txt (first of file contains o show us)
	egrep	(extended grep) :
			egrep '^(o|g)' a.txt (if a.txt start with o or g at the first of file show me)
			egrep '(o|g)|(G|O)' a.txt 
	fgreap 	(fast grep) :
			fgrep @ a.txt (search @ in a.txt in fast mode)
	sort 	(is like cat)
			sort -r file.txt (reverse)
			sort -R file.txt (random)
	cat 	(find and fetch column and row)
			cat -c3,5 a.txt (just 5 and 3)
			cat -c 3-5 (between 3 and 5)
			cat -d" " -f 2 a.txt (-d = delimiter , -f = field
	wc 		(word count)
	nl 		(number of lines)
		    like : cat -n
	^		(beginning of files)
	$		(ending of files)
	sed 	(replace of chars or words)
			sed -e 's/bisim/walkytaki/' a.txt
			sed -re 's/^(a-z)/u' a.txt
	cut 	(seperate part of file)
			cut -c 4,5,6,7,8 a.txt
	paste 	(append part of file)
	od 		(convert file to octed file)
			od -c a.txt (shows no-printable characters)
	fmt 	(change view format)
			fmt -w 7 a.txt (lines with)
	expand 	(convert tabs > spaces)
			expand -a a.txt
	unexpand (convert spaces > tabs)
			unexpand -a a.txt
	join 	(join two files in one with shared cel)
	pr 		(printable version of file)
	uniq 	(just don't show duplicated lines)
			uniq -c a.txt (count of repeated lines)
	splite 	(splite file)
			splite -l 2 a.txt (splite 2 lines of large file to separated file )
	tr 		(translate)
			echo "shayan" | tr a-z salam
			echo "shayan" | tr -s a (omit repeated words)

		grep bb$ a.txt 	(every where have bb end of file)
		grep ^aa a.txt 	(every where have aa begin of file)
		grep a.b a.txt 	(every where  have aa begin and b at the end of file)
		grep ^... a.txt (every where have 3 char whole of file)

convert commands to shell script :
normal form of shell definantion :
#!/bin/bash/
#!/bin/sh/
$1 = first parameter
$? = exit code

for execute code :
chmode -x script1

etc-log-lib-process :
	etc (linux configuration of all linux applications and softwares)
	lib (dlls and documentations)
	bin (all of the commands and apps)
	log and \var\log\ (notifications and warning)

process (create , kill , monitor)(like windows task manager) :
	creat:
		sleep 444 (goes to sleep for 444 secs and ctrl+c (break) , ctrl+z(stop),(forground))
		sleep 444 & (background)
	kill :
		kill 864684 (kill process with 864684 pid)
		killall sleep (kill all sleep process for users , not root user)
	
	jobs (show running ps)
	bg 2 (take ps number 2 and convert it to background ps)
	fg 2 (take ps number 2 and convert it to foreground ps)
	nohup sleep 400& (persistance(with logout don't kill process))

	proces levels :
		nice level :
			nice sleep 666 (creat new ps with priority 10,if exicst kill and creat again)
			nice -n -20 sleep 666 (priority -20 (high))
		renice :
			renice -n -20 -p 666 (change and set priority -20 (high) to pid 666 (sleep) with out kill and creat again)
			renice -n 17 -u unity (set priority 17 to unity user)	

		-ps 	not live and running process
			ps -aux (all users process)
			ps -alx (long view process)
			ps -aux | grep fire
			pstree
			ps -eo pid,user,nice,comm,pcpu
		-top	live and running process
			top -u unity
			15(tree)
			9(have zombies)
		-free	free ram space
		-uptime
		=		avg usage
		z 		running(different color)
		c 		absolute path
		d 		delay
		m 		memory
		shift+p cpu usage
		k 		kill pid

fail syslog (execute syslog)

\etc\resolve.conf 	(dns server info)
dig 				(dns information record)
ifconfig -a 		(show all interfaces although not active or off)

users information :
	\etc\passwd 	(users info like id naame ... used to save pass but not now)
	\etc\group		(wich id is in wich group)
	\etc\shadow		(sha512 of all useers password)

convert user to root or some others :
su root

if some users use su command at the same time ,make root pasword posibel to see.
sudo have syslog info in file. 

user handeling :
	useradd		(add user)
		-d 	(speific home directory path)
		-m 	(create home directory right now)
		-s 	(shell type and path of shell type)
		-g 	(group number)
		-G 	(group name for join in many groups)
		-u 	(user id)
		-c 	(comments)
	groupadd	(add group)
		groupadd -g barobax
	id 			(like id in user information)
	last		(all of the log like console and terminal)
	passwd 		(set password)
	usermode	(set users level access mode)
		usermode -L sheyn	(lock users)
		usermode -U sheyn	(unlock users)
		usermode -aG barobax shayan	(add to group)
		usermode -u -G a b (add some users to group)
		usermode -F (just delete the name although have data on disk)
		usermode -h (delete all users data)
		usermode -r (if doesn't have ownership cant delete)
	userdel		(delete user)
		userdel -r unity
	groupdel	(delete group)
	chage 		(change age or limit time for users)
		chage -L (list of users aging)
		chage -E (expriration date)
			chage -E 2020-10-10 unity 
	usermod -s /bin/sh -c "itpro" mimo (change shell to sh and write comment on mimo user)

access levels on files and folders :
(- (file) or d (directory))---(r (read) , w (write) , x(execute))---(r (read) , w (write) , x(execute))---(r (read) , w (write) , x(execute))
											users 								groups 									others
									4		  2			1				4		  2				1				4		  2			1
chmod	(change modification)
	chmod u+w a.txt (for users we can allow\set write permission on a.txt)
	chmod u-w a.txt (for users we can deny write permission on a.txt)
	chmod ugo+w a.txt (for users+groups+others we can allow\set write permission on a.txt)
	chmod ugo=rwx a.txt (for users+groups+others we can allow\set write+read+execute permission on a.txt)
	chmod 721 a.txt (for users(write + read + execute) group(write) others(execute) permission on a.txt)
	chmod 777 folder1 -R (folder1 and their container have full access)
	chmod +x * (for ugo and all files in this directory enable executive mode)
chown	(change ownership)
	chown root folder1 -R (folder1 and their container have root access)
	chown root:root  folder1 -R
chgrp	(change group)

symbolic link and shortcuts :
	ln-s file location (it's soft link type)
		ln-s a.txt b.txt
	ln file location (it's hard link type)
		a.txt b.txt

stiky bit :
chmod o+t folder1 -R
chmod 0777 folder

save these info at umas file.
	/etc/profile

Application	----|	
 |				|
dbus			|	(layer between software and hardware)
 |				|
hal-------|		|	(don't have direct access to Applications)
 |		  |		|
udev	  |		|	(devices information)(hals database)(hotplug and play on kernel 2.6)
 |		  |		|
kernel----|-----|	(os)
\dev is directory independent of \udev that is place of getting updates from \sys .

\proc or \procfs ===>>> runnning process on boot starting and RAM ,you could find it in virtual directory.(after reset get change)(manipulate of this must be root)
\sys or \sysfs ===>>> about CPU , RAM ,HDD , ODD , SSD
what is different \proc and \sys ? \sys is newversion of \proc

\dev (list of hardwares) 
	cpu info

changing data in \proc and \sys is temporary, to permanently change :
\etc\sysctl.conf

df -h (show devices)

modules installation and emitting :
\lsmod (kernels module list)
	rmmod vmxnet		(delete device)
	insmod vmxnet		(just install device)
	modprob	vmxnet		(install devices with their own dependecies and no error like insmod)
\lsusb	
\lshw
\lshal (prepherals with their own names)

for auto modul installation :
	nano \etc\module  (like insmod)

	it's bettwr use this : nano \etc\modprobe.d (like modprobe)

cd \proc
cat mounts

cat \var\log\dmesg
cat \var\log\messages (other application logs)

pstree
cd \boot
cd \boot\grub

run level >>>> is like pool of scripts by using them we can do some thing. 
rl or rc files have no content and they are linked to \etc\init.d direction that's place of all scripts.
inittab 
init.d
reboot
shutdown
upstart
systemd
run levels or running codes :
rl / rc 0 :	shutdown				(power off)
rl / rc 1 :	singel user				(rescue or recover mode)
rl / rc 2 :	multiuser no netwrok	(debian - textmode)
rl / rc 3 :	multiuser with netwrok 	(redhat and opensuse) 
rl / rc 4 :	not used
rl / rc 5 :	graphic					(default redhat and opensuse)	
rl / rc 6 :	restart					(reboot)

systemctl get-default
ls -al \lib\systemd\system\runlevel (list of rls/rcs)

.\netwrok-manger start
.\netwrok-manger stop
.\netwrok-manger status

boot directory size =100-200 mb (kernel os)
root folder don't go to home folder like other users because it's network user.
in linux for each device , concept , .... have folder.(mount points)
virual memory is kind of binding hdd to ram.(swap space)

simple design in linux hdd partition (local):
	\boot - \root partition - \swap
		(in local system we should not raid on boot drive (collision))

netwrok design in linux hdd partition:
	\boot - \root partition - \home - \swap

server design in linux hdd partition:
	\boot - \root partition - \home - \var - \swap

! type of raid is important.

to change bootmanager setting in these type of files :
lilo
grub :
	v1 : legacy
	v2 : grub
backup mbr

lilo :
	\etc\lilo.config

grub :
	v1 (legacy) :
		\boot\grub\menu.lst
		grub-install \dev\sda
		grub-update (don't need)

	v2(grub):
		\etc\default\grub (1- for change values)
		\etc\grub.d 	  (linked to 1)
		grub2-mkconfig -o \boot\grub2\grub.config (write changed values to \boot\grub2\grub.config)
		grub2-install

ldd \bin\ping (ping command contains which libraries)

\etc\
	ld.so.conf (dont have body but contains all libraries link. this folder execute \etc\ld.so.conf.d\*conf)

ldconfig (in linux some commands have dependecies so make links libraries to execute codes,linking of links have cost and must spend time for fetch and compile and searching but with this command we make binary file of links and make procedure faster than before - it is a static file).

export (like envirement variables)
	export ld-library-path=\home\unity\lib:\home\hasan\lib:.....

source.lst (list of all apps in repository pn linux service)
repository is like link mirroring.

installing package on linux : 
dpkg (without install dependecies):
	-i(--install)
	-l(--list)
	-L(--listfiles)
	-p(--print_avail)
	-P(--purge)
	-r(--remove)
	-s(--search)

apt (with install dependecies) :
	apt-get update (update source.lst links of repositories)
	apt-get upgrade (update all installed local apps)
	apt-get dist-upgrade (like above command)
	apt-get install tor (install package)
	apt-get auto-remove (remove unused pakcages)
	apt-get remove

apt-cache (local commands) :
	apt-cache search
	apt-cache depends (this file need or depedence wich files)
	apt-cache rdepends (which filles need or dependece on yhis file)
	apt-cache show

aptitude :
	aptitude install

nano \etc\apt\source.lst (show and change value of mirror links)

rpm (like dpkg and stands for redgat package management)(don't check dependecies):
	rpm -v *.rpm (verbose mode - show detailes)
	rpm -qi firefox.rpm (all info about pack)
	rpm -qd firefox.rpm (all documents about pack)
	rpm -qa (all info for all apps)
	install :
		rpm -i firefox.rpm
		rpm -i -nodeps firefox.rpm
		rpm -i *.rpm
	rpm remove :
		rpm -e firefox (erase)
	rpm signiture *
		rpm -k firefox
	rpm verify *
		rpm -V firefox.rpm
	rpm query *
	rpm rpm2cpio (extract rpm files or packages) :
		rom2cpio w.rpm > w.cpio
	rpm alien (convert .deb files to rpm)

yum (yellowdog package management)(check dependencies ,it's better than aptitudde and apt) :
	yum install
	yum remove
	yum upgrade
	\etc\yum.conf
	\etc\yum.repos.d
	yumdownloader
		yumdownloader firefox (without dependencies)
		yumdownloader --resolve firefox (with dependencies)

for create a ideal command :
	cat > jimbo
	ifconfig
	chmod +x jimbo
	
	to execute our jimbo code must go to absolute address :
		\ducs\jimbo

	for execute this code in universal mode must go to envirement variables and set this :
		path=$path:\ducs\jimbo:.....
	(this add to envirement var path our ideal code and destination)
	it's temporary . set permanently must go to :
		\etc\bashrc

	env  (shows envirement variables in linux)

for set var in global mode :
	unified="salam shayan"
	echo $unified
	export unified
	now we can see unified in env and way to remove :
		unset unified

globbing and wildcards :
	* 		(every thing)
	! 		(not)
	? 		(one char)
	[ac]	(a or c)
	[a-c]	(a to c)

	ls ???.??? 		(fetch every thing with 3 char at first part and every thing with 3 char at second part)
	ls [ac]* 		(fetch every thing start with a or c )
 	
 	ls | cpio -o > .\acpio.cpio (export ls and put it to acpio.cpio)

make image :
	dd if=\dev\sdb  of=.\usb-storage.img (creat image file)
	dd if=.\usb-storage.img  of=\dev\sdb (extract image file)

pipe and redirection :
	stdin 		(standard input)
	stdout 		(standard output)
	stderr		(standard error)
	pipe		|
	xargs		like pipe 
	tee 		(1 input and 2 output)

	ls 2> list.txt (stdout and stderr get together in 1 file)
	ls 2>> error.txt (append errors to file)
	ls *.mp>> mp4res.txt 2>> error.txt

	redirection :
	ls *mp4 >out.txt 2>&1 (&1 send to one args or out.txt)

	pipe vs xargs vs tee:
		ls *.txt | echo 	  (don't show any thing)
		ls *.txt | xargs echo (this show and print with echo ,send first parameter (ls) to second parameter (echo))
		ls *.txt | tee unity.txt (first show ls and print it to unity.txt ,send first parameter (ls) to terminal then second parameter execute)

vi editor :
it's built-in editor.
vi a.txt

/ and n 	(forward search)
? and N 	(backward search)
i 			(insert mode at the current character)
a 			(insert mode after the current character)
o 			(insert mode in new line)
esc 		(return to command mode)
c 			(change letter)
cw 			(change word)
d 			(cut or delete line or letter)
dd			(cut or delete whole line)
dw			(cut or delete whole line)
dl			(cut or delete letter)
p 			(paste)
yy			(copy or yank whole line)
yw			(copy or yank whole word)
yl			(copy or yank whole character)
:e! 		(reverting to original file)
:wq 		(write changes and quit)
:ZZ 		(write changes and quit)
:w! 		(write to new file or read-only file)
:q! 		(quit without saving)

fdisk (partitions) :
	fdisk -l (management disk)
	fdisk /dev/sdb
	t 1 L
	swapon -s  (paging size)
	fsck (fs checker fo use must unmount)
		unmount /dev/sdb
		fsck /dev/sdb 
		fsck -t ext2 /dev/sdb1
	xfs_check
		xfs_check /dev/sdb
	xfs_repair
		xfs_repair /dev/sdb
	xfs_metadump (analyse xfs ,fs data dump)
	xfs_info
	debugfs (repair and recovery fs for use this we must have inode id)
		mount -t ext /dev/sdb1 /dev/ffs1
		debugfs -w /dev/sdb1
		undel <12> t.txt
		quit
	dumpe2fs (information fs)
	tune2fs (like dumpe2fs + editing)
		tune2fs -o has_journal /deev/sdb1 (add)
		tune2fs -o ^has_journal /deev/sdb1 (remove)
	mkswap /dev/sdb1 (just make swap space)
	swapon /dev/sdb1 (active our swap space)
	mkfs -t ext2 /dev/sdb5 (make filesystem or change it)
	mkfs.xfs -t /dev/sdb5 (or) mkfs -t exf /dev/sdb1
	df (disk free (in mount or fs))
		df -ih
	du (disk usuage (in directory mode))
		du --summarize
	mount (shows and connect devices to system)

filessystems : ext (2 (centos) ,3 (ubuntu) ,4) - xfs (dynamic node) - reisefs - vfat (dos) - ntfs (windows)
swap spacce : mkswap , swapon
we can create 4 primary partitions and rest in extended partitions.
for using fs :
make partitions
format partitions
make directories (mkdir)
mount device to folder

mkdir ffs1
mount -t ext /dev/sdb1 /dev/ffs1

auto mount device :
vi /etc/fstab
/dev/sdb1	/external	ext3	RO,user unity,noexec {default}	

disk quota :
	vim /etc/fstab
	/dev/sdb1	/home/unity/quota 	ext3 	usrqouta,grpquota
	umount -a
	mount-a
	quotaoff -a (don't effect on fs)
	quotacheck -cug /dev/sdb1
	edquota -u unity
		set hard and soft limit
	edquota -t
		set grace period
	quotaon -a
	repquota -a


logins mode :
interactive mode (without user and pass)
shell login (with user nad pass)

after login linux execute some code or scripts .
/etc/profile (global mode) :
	for all users and general mode :
	/etc/bash.bashrc
	/etc/bashrc
	
	place of scripts :
	/etc/profile.d

	specific persons or users :
	/home/users/.bash_profile
	/.bash _login
	/.profile

	(any code for any users goes to profile.d)(individualy functions and aliases)
	/home/user/.bashrc

	in linux ,rc means execute code or scripts.aliases is name that replaced by org commands.

some times we need place some files on our users desktops :
	cd /etc/skel

functions and scripts :
#!/bin/bash
function shabake(){
	echo "tanzimate shabake !";
	ifconfig;
}

if :
	#!/bin/bash
	echo "are you an itpro ?"
	read isname
	if[isname = "yes"];
	then	
		echo"allright , wellcome !!" | mail -s "congrats" root
		echo"allright , wellcome !!" | mail -s "congrats" root < a.txt
	else 
		echo "goback."
	fi

for :
	for x in 1 2 3 4 5 (or `seq 1 10`)
	do 
		echo "number is $x"
	done

	lis=`ls -al`
	for x in lss
	do
		echo $x
	done

while :
	x=1
	while[$x -ne5]
	do 
		echo "number is $x"
		x=$(($x+1))
	done

for set aliases (to set that we must reset our terminal) :
/etc/.bashrc
	alias shayan=`ping 127.0.0.1`
	:wq

open new terminal :
	gnome-terminal

sql (mysql):
	apt-get install mysql-server
	service mysql start
	mysql
	mysql -u root
	SHOW DATABASES;
	USE sqldb;
	SHOW TABLES;
	CREATE DATABASE shayan;
	USE shayan;
	CREATE TABLE A (id INT(10),name CHAR(20),video CHAR(20));
	CREATE TABLE B (user CHAR(10),record CHAR(20));
	SELECT * FROM A;
	SELECT ID FROM A;
	SELECT * FROM A WHERE id=hasan;
	SELECT * FROM A WHERE id!=hasan;
	SELCET * FROM A LIKE '%ED%';
	SELECT * FROM A ORDERBY id;
	SELECT * FROM A JOIN B ON A.id=B.user;
	SELECT * FROM A JOIN B;
	SELECT COUNT(*),record FROM A GROUP BY record;
	UPDATE user SET user="unity" WHERE user="uity";
	DELETE FROM A WHERE video LIKE '%ux' LIMIT 1;

x11 :
x ===> graphic modes
xorg.conf
/etc/x11/xorg
x -configure
when we want go to x11 :
	/etc/default/grub
	nano grub.d and set text mode instead of splash.
	update-grub
	startx
	xorg.conf (nano)
	xhost (make our system like x11 server)
	display
	ctrl+alt+f7

display managers :
gdm (dnome)
kdm (kde)
xdm (x11)

echo $desktop-session
list of display mangers :
	ls -l /usr/shared/xsessions

install graphic :
	yum groupinstall 'x windows system' 'kde'
	apt-get install kdm gdm xdm

switch beetwen graphic and text display manager  :
gnome :
	graphic:
		systemctl get-default
			graphic.target (or runlevel 5)

	text:
		systemctl list-units --type=target
			multi-user.target (or runlevel 3)
		systemctl set-default multiuser-user.target

debian :
	graphic:
		startx (start graphic mode)
	text:
		in grub file must change value of quitsplash screen :
		/ect/default/grub (nano and change to text)
		update-grub
		reboot

centos :
	cd /etc/sysconfig/desktop (nano desktop)
		display-manager="kde"
	echo $desktop-session
or		cd /etc/x11/prfdm

set default display manager :
	/etc/x11/defualt-display-manager (nano and write our favariot dm)
	reboot
or	dpkg-reconfigure 
		gdm

show message on login page :
	cd /etc/gdm/custom.conf (nano)
		[deamon] : greeter=/usr/libexec/gdmlogin
				   logingreeter =/usr/libexec/gdmlogin
		[greeter]: defaultwellcom=false
				   wellcome="hi"
				   remotewellcome="hi"
		[xdmp]: enable=true
		reboot

	on kdm :
		cd /usr/shared/config/kdm
		nano kdmrc

schedule and tasks :
	/etc/crontab 								(define new job and it's file by itself)
	/etc/cron.d 								(a file that define cron jobs and execute by schedules and link to bottom)
	/etc/cron.(daily,weekly,hourly,monthly)		(body of scripts that execute by linking in cron.d file)
	./var/spool/cron 							(each users tasks)

	minuts	hours 	dayofmonth 	month 	dayofweek 		username
	0-59 	0-12 	1-31 		1-12 	0and7=sunday
							  (jun-jul)

	crontab -e a (then nano a file)
	10 		*		*			*		*		root cat /home/unity/hi.txt | mail -s "<3 u" nasiri@itpro.ir

	anacron = 0hourly = cron.d
	anacron check the tasks that maynot execute or crashed and replay them.
	place of storing our cron data :
		 cd /var/spool/cron 					  

some times we need shortest schedules or 2 or 3 repeated in period :
	at (define new task)
	at 22:00
		> /bin/ping
	atq (show queue of tasks)
	atrm (remove from queue tasks)

font and encoding :
$lang (env variable and langs encode , lang details)
lang = c (by default encode for scripts)
locale (detailes for $lang)
LC_TIME=EN_US.UTF8
export LC_TIME
LC_ALL=EN_US.UTF8
export LC_ALL
iconv -f utf8 -t ascii /path......
iconv --list (show all supports)

timezone :
tzconfig
tzselect (help for tzconfig)
nano /etc/timezone	(/usr/shared/zoneinfo)
nano /etc/localtime (etc/timezone)(copy of aboe file)
date (set date)

in env we can set date :
	TZ='ASIA/tehran';
	export TZ

hardware clock : set in bios battery 
software clock : show uptime (loading kernel time , read data from hardware clock)
utc :standard format to showing time.
hwdclock --set --date=".........." 
hwdclock
reboot

set hardware clock from utc :
hwdclock -w
hwdclock -u -w
hwdclock --localtime -w

ntp :
ntpdate auheoahoef.afalifia.ajfai
yum install ntp
service ntp start
systemctl ntpd start
ntpdate "......"	(use when u are ntp client)
nano /etc/ntp.conf
systemctl restart ntpd

syslog :
versions :
	syslog (base and base of others)(udp - clear text - no authentication)
	rsyslog (better than syslog in some os don't have syslog use this can copy content and use likke syslog)(tcp + udp - authentication)
	syslog-ng

	facilities (sectors and grouping)
	priority (levels of logs)
	action (what to do after catch log)

	*.* 	/var/logs/messages 		(write all facilities and priority in this path and file)
	*.info 	/var/logs/messages		(write all facilities and above information priority in this path and file)
	ftp.crit 	/var/logs/messages  (write ftp facilities and critical and above it priority in this path and file)
	*.*		* 						(send all notifications for all users)
	*.* 	-/var/logs/messages		(use cache for this action and send it faster)
	*.=crit  /var/logs/messages		(catch all facilities and just critical priority in this path and file)

	we can set exceptions :
		nano /etc/syslog.conf
			*.info ; mail.none ; news.none ; 	-/var/logs/messages
			*.info ; mail.none ; news.none ; 	@172.16.100.11 (udp)

	to set a system become listener (works just on syslog) :
		in debian we can copy syslog content to rsyslog :
			nano /etc/rsyslog.conf

		nano /etc/sysconfig/syslog
			SYSLOG-OPTION="-m 0 -r" (must set remote syslog)
			$MODLOAD IMUDP
			$UDPSERVERRUN 514
			$MODLOAD IMTCP
			$INPUTTCPSERVERRUN 514
			service rsyslog restart	

	to set a system syslog client :
		in the end of file nano /etc/syslog.conf:
			*.*  	@@172.16.100.11:514
			*.*  	@172.16.100.11:514
		service rsyslog restart

	we can send log by ourselves :
		logger -p user.info #hi dude

how mail in linux :
	apt-get install mail
	
	send :
		mail 	nasiri 
		subject	:	ahuifah
		aihfahfiaeflafahf
		efhahfalheflahefa
		falheflahflahef.
		ccdd	:	ausalhdas			
		ctrl+d
	read :
		mail
		1
		q(quit)

	forwarding mail root and others :
		root :
			nano /etc/aliases
			unity
		others 	:
			nano /home/usr/.forward
				unit
			newaliases


printers managements :
cups (common unix printing system) :
	works on port 631 (listen) or 9100.
	/etc/cups/cupsd.conf 		(services,confogs,acs.options)
		nano /etc/cups/cupsd.conf 
			listen 631
			browsing on
			browsingaddress	@local
	/etc/cups/printers.conf 	(configs for evry printers)
		we can set uri in this file.

	cups use specia mechanisme like uri. fro use it in ubuntu we should use nonroot users.cups is appsocket program.must not be in vlan or behinde firewall.use simple name.

	ppd = portscript print device
	cd ppd/ 	(shows installed drivers)

	spooler data forlder :
		cd /etc/spool/cups

	lpq 		(show all printers on default printer)
		lpq -a 	(shows for all printers)
	lprm 		(remove from queue)
		lprm - (remove all print jobs)
	lpc 		(status of printers)
	lpmove 		(move prints a to other printers if have problem on a printer)
		lpmove job 5 printer-number-2
		lpmove printer-number-1 printer-number-2

network configuration :
	/etc/hostname (name of my computer)
	/etc/hosts
		nano /etc/hosts
			172.16.1.10 	route.itprolocal
	/etc/switch.conf 	(set dnss)(read above file and set order to exec them)
		nano /etc/switch.conf 
			hosts=dns files
	/etc/resolve.conf 	(order of name servers and checking priority)
		nanon /etc/resolve.conf
			nameserver 8.8.8.8
	in redhat :
		/etc/sysconfig/network.scripts/ifconfig.etch0
		/etc/sysconfig/network

	in debian :
		/etc/network/interfaces
			nano /etc/network/interfaces
				auto eth0
				iface eth0 inet static
				address 172.16.1.101
				netmask 255.255.0.0
				network 172.16.0.0
				gateway 172.16.1.1
				dns-nameserver 8.8.8.8
			reboot

	ifconfig (status and information of interfaces)
		ifconfig eth0 ip 172.16.1.101
		ifconfig eth0 netmask 255.255.0.0
		ifconfig eth0 gateway 172.16.1.1
		ifconfig eth0 network 172.16.0.0		
	ifup
	ifdown
	route (set default gateway)
		route add defaultgateway 172.16.1.1
	netstat
		netstat -tuna -nao -na 
		lsof -i
	dig (shows info and details)
		dig google.com
		dig @8.8.8.8 google.com

simple security and management duties :
	find / -perm -u+s (show places that user id set an have permisions)
	find / -perm -u+s,g+s (users and groups)
	find / -perm /u+s,g+s (users or groups)

	ulimit (limit users access in session after close session get reset)
		ulimit -u 	(for users)
		ulimit @foriegn-groups
		ulimit -f 0 (can create file with 0 capacity)
	to set permanently changes :
		nano /etc/security/limits.conf

	see open ports :
		nmap 127.0.0.1

	sudoers (get permirmisions to users like bee root):
	 nano /etc/suduers (or visudo)
	 	unity 	ALL 	 (	ALL 	: 	ALL 	) 	ALL

	 		 	all 		all 		all 		all
	 		 	hosts	  commands   	groups 		commands
	 		 				can 
	 		 			   execute
	 		 			   like root	

	 	%group-mali all(all:all)all		
	 	shayan nopasswd:all (don't ask password)

	 hardening can help us to close nousefull ports that are open.
	 in redhat :	
	 	cd /etc/init.d/httpd
	 	/etc/init.d/httpd stop
	*	these setting can be reset on reboot but we can change to permanently off :
		chkconfig httpd off
		/sbin/chkconfig off
		/sbin/chkconfig --list httpd (check the status of activate modes)
		chkconfig --level 5 httpd on (just run httpd service on rl 5)
	in debian :
		/etc/init.d/apache2 stop (temp)
		update-rc.d  -f apache2 remove (perm)

	 /etc/nologin (if creat this file just root can access system not others)

cryptography :
	rsa = cryptograpyhy + digital signing
	dsa = digital signing
	ecdsa = newest version of dsa
	passphrase is like master password.
	on client :
		gpg --gen-key
		cd .gnapg/
			gpg --lis-key
			gpg --export a>b.pub
			scp b.pub root@172.16.1.10 :/home/desktop
			gpg --out a.txt --decrypt d.txt
	on server :
		gpg --list-key
		gpg --import b.pub
		touch c
		nano c.txt
			salam chetory
		gpg --out d.txt  --reciption "shayanheydarikhah" --encrypt c.txt
		(now we can not execute the file just shayan can exec that)
		scp d.txt root@172.16.1.10 :/home/desktop

for remote access with no pass and user :
	nano /etc/ssh/sshdconfig
		permit-root-login = yes
	cd .ssh/
	ssh-keygen -t rsa (create rsa key)
	ssh-keygen -t dsa (create dsa key)
now we must send public key to server :
	ssh-copy-id -i .ssh/id-rsa.pub root 172.16.1.169
now set passphrase.
	ssh-agent bash (shell in shell)
	ssh-add .ssh/id-rsa

ssh -X 172.16.1.154 (get image of system)

port forwarding :
	ssh 172.168.1.10 -N -L 8888 : 172.168.1.11:88

send and recive file to server and client :
	on server :
		scp vid.mp4 root@172.168.1.10:/home/videos
