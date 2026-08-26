#linux essential 
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
	groupdel		(delete group)
	chage 			(change age or limit time for users)
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
