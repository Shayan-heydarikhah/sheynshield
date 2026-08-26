fortianalyzer:
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	general :
		it is a comprehensive monitoring system that provides new and old data and information in one unit. this system can monitor and log security threats
		filter things like user id, ip address, application, etc. have a detailed and detailed look at the data. this system can be used to check traffic such as uploads and downloads by users or watching videos on a specific site and provide you with the output information in text or graph format. it should be noted that fortiview displays information related to analytical logs and does not display archived logs.

		* analysis of ips/ids attacks
		* management of all logs at the network level
		* the possibility of storing suspicious files in quarantine
		* ability to recover quarantine files
		* receive all events, at the network level
		* presentation of various reports graphically
		* ability to customize reports
		* the possibility of reporting applications

		https://www.fortinet.com/products/product-compare
		https://www.fortinet.com/products/management/fortianalyzer
		https://help.fortinet.com/fa/vm-install/56/document/200_licenses/200_licensing.htm

		ensure you have remote serial console or virtual console access.
		ensure a local tftp server is available on a network local to the fortianalyzer.
		perform regular backups to ensure you have a recent copy of your fortianalyzer configuration.
		set up a backup schedule / on vm use snapshots
			config system backup all-settings
			set status enable
			set server 172.20.120.11
			set user admin
			set directory /usr/local/backup
			set week_days monday
			set time 13:00:00
			set protocol ftp (default is sftp also use scp)
			set password/crptpassword 123 (limited to 63 character)
			set crt test.cert
			end
			
		cli :
			execute reboot
			execute shutdown
			execute reset all-settings (factory reset)
			execute reset-sqllog-transfer (transfer again all logs to db)

			config system global
			set daylightsavetime enable
			set hostname fmg3k
			set timezone 12
			end
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	operation mode :
		analyzer mode
			default mode in fortianalyzer devices. all the features of the fortianalyzer device such as fortiview, event monitor and reports are supported in this mode. in this case, you can collect logs related to one or more devices

		collector mode
			its main task is to send the logs from the devices connected to it to an analyzer and archive the logs. instead of entering the logs in the database, it stores them in the original format, which is the binary format, for uploading. in this mode, most features of the fortianalyzer device such as fortiview, logview, event monitor and reports are disabled

		*you should allocate most of the disk space for archive logs

		better check and enable aggregation log
			cli :
				config system log-forwarding-service
				set accept-aggregation enable

		*can deploy analyzer mode and collector mode on different fortianalyzer units and make the units work together to improve the overall performance of log receiving, analysis, and reporting.
		analyzer offloads the log receiving task to the collector so that the analyzer can focus on data analysis and report generation

		might want to fetch logs from the collector to the analyzer. the collector will perform the role of the fetch server, and the analyzer will perform the role of fetch client
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	adom (administrative domain) :
		config system global
		set adom-status enable
		end

		has two modes: normal and advanced. normal mode is the default device mode. in normal mode, a fortigate unit can only be added to a single administrative domain. in advanced mode, you can assign different vdoms from the same fortigate to multiple administrative domains.
			config system global
			set admo-mode advance
			end

		each adom specifies how long to store and how much disk space to use for its logs. you can monitor disk utilization for each adom and adjust storage settings for logs as needed.

		fortianalyzer includes default adoms for specific types of devices. when you add one or more of these devices to the fortianalyzer, the devices are automatically added to the appropriate adom, and the adom becomes selectable.
		
		when a default adom contains no devices, the adom is not selectable

		can organize devices into adoms to allow you to better manage these devices. devices can be organized by whatever method
			firmware version
				group all devices with the same firmware version into an adom.
			
			geographic regions
				group all devices for a specific geographic region into an adom, and devices for a different region into another adom.

			administrative users
				group devices into separate adoms based for specific administrators responsible for the group of devices.
			
			customers
				group all devices for one customer into an adom, and devices for another customer into another adom.

		can add forti client logs for chrome books 
			forticlient logs are viewed on the fortigate device.
			when endpoints are registered to a forticlient ems, forticlient logs are viewed in the forticlient adom that the forticlient ems device is added to

			*adoms must be enabled to support forticlient ems devices.

			must set ssl certificate
				ssl certificate is required to support communication and send logs between forticlient web filter extension	and fortianalyzer

					system setting > certificate > local certificate
					system setting > admin > admin settings (https connection)
						https & web service certificate
							select certificate

			config system interface
			edit "port1"
			set allowaccess https ssh http http-logging https-logging

		device manager, fortiview, log view, event management, and reports panes are displayed per adom

		*fortigate and forticarrier devices cannot be grouped into the same adom. forticarrier devices are added to a specific default forticarrier adom.

		*he adoms feature cannot be disabled if adoms are still configured and have managed devices in them.

		how create adoms
			system setting > all adoms
				name
				type
					if you disable adom our creation has no categories
					if use adom enable status
						we can see some categories will be shown on predefine mode and add devices could be modifiable

				version
					if set fortimanager can see this
					must set device type on foritgate and forticarrier

					version can not be change
					but can set different adoms for each versions

				devices
				data policy
					keep logs for analytic
						how long to keep logs in the indexed state

						during the indexed state, logs are indexed in the sql database for the specified amount of time. information about the logs can be viewed in the fortiview, event management, and reports modules

						after the time we see data get purge automatically from sql

					keep logs for archive
						how long to keep logs in the compressed state

						 when logs are in the compressed state, information about the log messages cannot be viewed in the fortiview, event management, or reports modules

						 after specify time our logs get delete

				disk utilization
					maximum allowed
						maximum amount of fortianalyzer disk space to use for logs, and select the unit of measure

					analytic archived
						percentage of the allotted space to use for analytics and archive logs

						*analytics logs require more space than archive logs

					alert and delete when usage reaches
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	network :
		service access: 
			if you want to allow the access of fortigate update services to this port, activate the fortigate updates option

		must use specific dns here
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	system setting
		dashboard
			alert message console
				alert messages for both the fortianalyzer unit and connected devices 

			log receive monitor
				logs received can view data per device or per log type

			insert rate vs recieve rate 
				 is hidden when the fortianalyzer is operating in collector mode,and the sql database is disabled.

			log insert lag time
				is hidden when the fortianalyzer is operating in collector mode,and the sql database is disabled.
				if set interval on 0 means disable

		ntp
			for many features to work, including scheduling, logging,and ssl-dependent features,the 			 fortianalyzer system time must be accurate.

		firware upgarde mechanism
			first of all make backup then change versions
			our files are .out extentions

			execute restore image {ftp | tftp} <file path to server> 192.168.150.254 admin password!

		for backup we have .dat files also encryption mechanism
		in restore ask overwrite current ip and routing setting

		*can migrate from one analyzer to another with 
			execute migrate all-settings <ftp | scp | sftp> <server> <filepath> <user> <password> [cryptpasswd]

		*if change server location of fortigaurd servers  must reboot device
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	administartor
		admin profiles 
			system-setting
			administrative domain
			adom-switch
			device manager
			device-manager
			add/delete devices/groups
			device-op
			log view/fortiview/noc
			log-viewer
			event management
			event-management
			reports
			report-viewer
			  
			cli only settings
				device-wan-link-load-balance   
				device-ap   
				device-forticlient   
				device-fortiswitch   
				realtime-monitor

		remote authentication server
			ldap (lightweight directory access protocol)
				name
				server name \ ip
				port 
					389

				common name identifier
				distinguished name
				bind type
					simple
					anonymous
					regular

				secure connection
					protocol	
						strattls
						ldaps

					certificate

				administrative domain

			radius (remoteauthentication dial-in user)
				name
				server name/ip
				port
					1812

				server secret
				secondary server name/ip
				secondary server secret
				authentication type
					none
					pap
					chap
					msv2

			tacacs+ (terminal access controller access-control system)
				name
				server name/ip
				port
					49

				server key
				authentication type
					auto
					ascii
					pap
					chap
					mschap

		 remote authentication server groups can only be managed using the cli
		 	 config system admin group
		 	 edit <group name>
		 	 set member <server name> <server name> ...
		 	 end

		 lockout and retries attemp
		 	config system global
		 	set admin-lockout-duration 30

		 	config system global
		 	set admin-lockout-threshold 3

		we can use forti-authenticator on forti-analyzer and set 2fa
		must use local or radius users
		page 73 manage forti-authenticator
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	certificate
		pki :
			public key infrastructure (pki) authentication uses x.509 certificate authentication library that takes a list of peers, peer groups, and user groupsand returnsauthentication successful or denied notifications

			administrators only need a valid x.509 certificate for successful authentication; no username or password is necessary

			pki authentication must be enabled 
				config system global
				set clt-cert-reg enable

		*when both set clt-cert-req and set admin-https-pki-required are enabled, only pki administrators can connect to the fortianalyzer gui.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	raid :
		if the fortianalyzer device model supports raid, you can set the raid level by selecting the system settings option on the main page and then selecting the raid management option

		the raid management tree menu is only available on fortianalyzer devices that support raid

			linear: 
				all hard disks are combined and form one large virtual disk. 
				the available memory in this format is equal to the total number of used disks. by using this format of raid, a slight change is made in the efficiency and performance of the memory. if one of the drives has a problem, all the drives will be unusable and all data will be lost until the problem of the above drive is solved.

				*raid 0 is not recommended for mission critical environments as it is not fault-tolerant.

			------------------------------
			raid 0: 
				known as striping. 
				in this format, the fortianalyzer device divides the data into equal parts and stores these parts equally between the hard drives. the available memory in this case is equal to the sum of all used disks. this format does not support redundancy in data storage. if one of the drives has a problem, the data stored on the above drive cannot be recovered. in this format of raid, the efficiency increases compared to the previous mode because, as mentioned, the information is distributed between the disks. the minimum number of drives in this mode is two, and data protection is not supported.

				*one write or two reads are possible per mirrored pair. raid 1 offers redundancy of data. a re-build is not required in the event of a drive failure. this is the simplest raid storage design with the highest disk overhead.
			------------------------------
  			raid 1: 
  				known as mirroring. in this format, fortianalyzer stores information on one hard drive and copies a copy of this information to all other hard drives. 
  				the available memory in this format is only equal to the memory of one of the hard disks. this format supports redundancy in data storage. if any of the hard disks encounter problems, the backup version of the data is available on the other hard disks.
			------------------------------
			raid 1 +spare: 
				 this format is the same as the previous format, with the difference that in this raid format, the fortianalyzer device uses one of the hard disks as standby.
				 as a result, if the hard disks fail for more than a minute, the above disk will replace it. it should be noted that the replaced hard disk is considered as the original disk.
			------------------------------
			raid 5: 
				like raid0, the fortianalyzer device divides the information into equal parts and stores these parts equally between hard disks, with the difference that parity check is used in this format. 
				parity will ensure that when data is moved from one point to another in memory, errors are encountered or saved. the available memory in this case is equal to the sum of all used disks minus one disk which is considered as parity storage.
				in this format, the efficiency is increased compared to the previous mode, and if any of the hard disks encounter problems, the data will not be lost. if a drive fails, the new drive is replaced and the fortianalyzer device restores the data to the new disk via reference information in the parity volume. the minimum number of drives in this case is three, and also the act of protecting data is in the form of single-drive failure. this format is generally used more than other formats.
			------------------------------
			raid 5 +spare: 
				same as the previous format, except that in this format, the fortianalyzer device uses one of the hard disks as standby. 
				as a result, if the hard disks fail for more than a minute, the above disk will replace it. it should be noted that the replaced hard disk is considered as the original disk.
			------------------------------
			raid 6: 
				same as raid 5, except that it has an additional parity block. two parity blocks are used in this format. 
				the minimum number of drives in this case is four, and data protection is done in the form of two disk failures.
			------------------------------
			raid 6 +spare: 
				same as the previous format, with the difference that in this format, the fortianalyzer device uses one of the hard disks as standby, so if the hard disks have problems for more than a minute, the above disk replaces it. 
				it should be noted that the replaced hard disk is considered as the original disk.
			------------------------------
			raid 10: 
				known as raid 1+0. this format combines the two features of disk mirroring and disk striping to protect data. 
				the available memory in this case is equal to the sum of all used disks. in this format, at least four discs must be used.
			------------------------------
			raid 50 (recommended) : 
				known as raid 5+0, is a combination of raid 5 and raid 0. the available memory in this case is equal to the sum of all used disks minus one disk which is considered as parity storage. 
				in this format, the efficiency is increased and if a drive encounters a problem, the data is not lost. the minimum number of drives in this mode is six.

				*higher fault tolerance than raid 5 and higher efficiency than raid 0.

				raid 50 is only available on models with 9 or more disks. 
				by default, two groups are used unless otherwise configured via the cli

					diagnose system raid status (current raid)

			------------------------------
			raid 60:
				known as raid 6+0, is a combination of raid 6 and raid 0. 
				the minimum number of drives in this mode is eight.

				*high read data transaction rate, medium write data transaction rate, and slightly lower performance than raid 50

			*each changes on raid config means you lost data

			cli
				diagnose system raid status

		the disk space available for you to set log quotas depends on the raid level and the reserved space for temporary files. temporary files are needed for indexing, reporting, and file management. in your planning, include both the disk space for the original logs fortianalyzer receives (archive) and the space required to index the logs (analytics).

		fortinet recommends using the default ratio of analytics : archive for most deployments

		6.0.3 and later, you can also increase the size of an existing virtual disk. no format is required.
			execute lvm extend

		locallog disk setting :
			config system locallog disk setting
			set status enable
			set severity information
			set max-log-file-size 1000mb
			set roll-schedule daily
			set upload enable
			set uploadip 10.10.10.1
			set uploadport port 443
			set uploaduser myname2
			set uploadpass 12345
			set uploadtype event
			set uploadzip enable
			set uploadsched enable
			set upload-time 06:45
			set upload-delete-file disable
			end

		system setting > raid management
			raid level
			disk management
				disk number
				disk status
					ready
						means working in normal mode

					rebuilding
						is writing data to a newly added hard drive in order to restore the hard drive to an optimal state
						is not fully fault tolerant until rebuilding is complete

					initializing
						writing to all the hard drives in the device in order to make the array fault tolerant

					verifying
						ensuring that the parity data of a redundant drive is valid

					degraded
						 hard drive is no longer being used by the raid controller

					inoperable
						one or more drives are missing from the fortianalyzer unit
						the drive is no longer available to the operating system. data on an inoperable drive cannot be accessed

			disk model

		swapping hard disk
			if a hard disk on a fortianalyzer unit fails, it must be replaced

			devices that support hardware raid, the hard disk can be replaced while the unit is still running - known as hot swapping. on fortianalyzer units with software raid, the device must be shutdown prior to exchanging the hard disk

			*when replacing a hard disk, you need to first verify that the new disk is the same size as those supplied by fortinet and has at least the same capacity as the old one in the fortianalyzer unit. installing a smaller hard disk will affect the raid setup and may cause data loss. due to possible differences in sector layout between disks, the only way to guarantee that two disks have the same size is to use the same brand and model.

			the size provided by the hard drive manufacturer for a given disk model is only an approximation. the exact size is determined by the number of sectors present on the disk

			on devices support hardware raid we can use hot swap

		*recommends you use the same disks as those supplied by fortinet
		disks of other brands will not be supported by fortinet
			can also migrate the data to another fortianalyzer unit, if you have one. data migration reduces system down time and the risk of data loss
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	logs are in two states when logged on the fortianalyzer device. you can use the data policy to determine how long and in which state each log should be placed :
		archive logs: in this mode, the log is compressed and saved on the hard disk. in this case, the log is offline and you cannot view its details through fortiview or in the log view section. also, in this mode, you cannot generate reports about log.

		analytics logs: in this case, the log is placed in the sql database and is online, and you can view its details through fortiview or in the log view section. also in this mode you can generate reports about log.

		in the fortianalyzer device, the system reserves 5 to 25 percent of the hard disk space for system usage and unexpected overflows, and only 75 to 95 percent of the hard disk space is reserved for the devices that are added to the fortianalyzer for monitoring purposes. will be available. it should be noted that the reports are stored in the reserved space.

		when the volume of the archived logs exceeds the virtual value you have set for them in the hard disk memory, one of the following situations occurs:

			1- a warning message is generate.
			2- older archived logs are deleted

				fortianalyzer > system setting > advance > filemanager
					global automatic file deletion:
						in this method, you will specify when older archived logs, quarantined files, reports and archived files are deleted from the hard disk through file management settings. this action is performed regardless of the log storage settings. for this purpose, follow the steps below:
							data policy:
								in this method, by creating a policy, you will determine how long the logs of each device will be stored on the hard disk. after the mentioned period, the above logs will be deleted automatically.

							disk fullness automatic deletion policy:
								in this method, by creating a policy, when the memory allocated to the logs of each device is full, older logs are automatically deleted.

		logs related to devices whose connection with fortianalyzer has been deleted
			when the connection of one or more devices with fortianalyzer is removed, the log files and archived packets related to the above devices will be deleted and subsequently this event will be recorded in the event log section. of course, the logs stored in the sql database will remain unchanged and will not be deleted. as a result, the above logs and reports related to these logs will be visible in log view and fortiview. you can use the following methods to delete logs from the sql database:
				rebuild the sql database
				through the settings related to creating a policy for storing logs
				through the settings related to automatic deletion of logs

		rollover of the files that are the storage location of the logs in the fortianalyzer device
			when the monitored device sends a log to the fortianalyzer, first the log is compressed and stored in a file on the hard disk of the fortianalyzer. the size of this file has an allowed value and when the number of logs stored in this file increases and exceeds this allowed value, the above file is called roll over and a new file is created to store the next logs.
				fortianalyzer > system setting > advance > device log setting

					also here we can set some features like :
						1- uploading logs stored in the fortianalyzer device to other devices
						2- upload event logs related to the fortianalyzer device
							you can send internal logs and event logs related to the fortianalyzer device to another fortianalyzer device or to a fortimanager.
								*severity level: select the lowest severity level from the list, so all logs will be uploaded.

		*note that analytical logs need more memory than archived logs. the default ratio for these two log samples is 60 to 40 percent

		on dashbord we have :
			insert rate vs receive rate: this widget displays the rate of receiving logs sent from and entered into the fortianalyzer device based on time. if the rate of incoming logs is higher than the rate of received logs, the database will be rebuilt as a result. 

			you can change the time interval and refresh interval by clicking on the settings icon in the widget. it should be noted that this widget is not visible when the fortianalyzer device is operating in collector mode and the sql database is inactive.

				log receive rate: the number of logs received by the fortianalyzer device
				log insert rate: the number of logs entered into the database of the fortianalyzer device

			log insert lag time: shows the amount of time that the database is being processed. this widget is not visible when the fortianalyzer device is operating in collector mode and the sql database is disabled.

		the fortianalyzer device supports the following modes in log forwarding:
		system setting > log forwarding

			real-time or forwarding mode: 
				in this mode, logs are forwarded to other servers instantly and at the same moment they are found. 
				in this case, the contents of the files that include dlp files, antivirus quarantine files and ips packets capture. 

				is the default mode. the fortianalyzer device supports sending logs to other fortianalyzers, a syslog server, or a cef server in this case.

				in this case just config client side 

			aggregation (use tcp 3000):
				in this case, the fortianalyzer device first receives the logs and stores them, and then forwards them daily at a specific time. 
				in this case, the contents of the files are also forwarded. 

				in this case, the fortianalyzer only supports sending logs to other fortianalyzers. forwarding logs to a syslog server or a cef server is not supported in this case.

				log-forward and log-forward-service cli

				in this mode we must config the reciever

				Aggregation mode can onlybe configured using the CLI. Aggregation mode configurationsare not listed in the GUI  table, but still use a log forwarding ID number.
					 get system log-forward

					 config system log-forward-service
					 set accept-aggregation enable
					 set aggregation-disk-quota <quota>

					 config system log-forward (open log forward command shell)
					 edit <log forwarding ID>
					 set mode aggregation
					 set server-name <string>
					 set server-ip 192.168.150.251
					 set agg-user admin
					 set agg-password 
					 set agg-time 0 to 23 (0 means midnight)

					 config system log-forward
					 purge (delete configs)

					 check for cfg[<log forwarding ID>] svr_disp_name=<server-name>

			mixed: 
				in this mode, the logs are forwarded to other servers instantaneously and at the same moment they are found, while the content of the files are forwarded at a specific time and on a daily basis as in the aggregation mode.

				to change the log forwarding mode, enter the following commands in the cli
					config system aggregation-client
					edit [log aggregation id]
					set mode [realtime, aggregation, both, or disable]
					end

			log forwarding is enabled by default (log aggregation and forwarding on fortianalyzer)(if the fortianalyzer device is operating in collector mode, it is impossible to disable the log forwarding feature)
				config system aggregation-service
				set accept-aggregation enable
				end 

				config system admin setting
				set show-log-forwarding enable
				end 	

			enable exclusions: 
				this option is only visible on the settings page when you select one of the syslog or cef options for the remote server type field. by activating this option and using the log type, device type and exclusion list fields, you can limit the sending of logs.		

				we use 3 mode for this part of forwarding log
					syslog
					cef
					fortianalyzer

			 system settings > logging topology (see flow of log transmission)

			 in addition to forwarding logs to another unit or server, the client retains a local copy of the logs.
			 local copy of the logs is subject to the data policy settings for archived logs.

		to generate a report for a time period not covered by current analytical data:
			use log fetching (fetcher management) to fetch archived logs to generate reports.
			import log data from an external backup to generate reports.

			Log fetching is used to retrieve archived logs from one FortiAnalyzer device to another. This allows administrators to run  queries and reports against historic data, which can be useful for forensic  analysis.


			The fetching FortiAnalyzer can query the server FortiAnalyzer and retrieve the log data for a specified device and time  period, based on specified filters. 

			The retrieved data are then indexed, and can be used for data analysis and reports.
			
			Log fetching can only be done on two FortiAnalyzer devicesrunning the same firmware. A FortiAnalyzer device can be  either the fetch server or the fetching client, and it can perform both roles at the same time with different FortiAnalyzer devices.

			Only one log fetching session can be established at a time between two FortiAnalyzer devices

			*If this is the first time fetching logs with the selected profile, or if any changes have been made to the devices  and/or ADOMssince the last fetch, on the client, sync devices and ADOMswith the server

			system setting > fetcher management
				profiles
					name
					server ip
					user
					password

				The A fetch request requests archived logs from the fetch server configured in the selected fetch profile. 

				When requesting, the ADOM on the fetch server from which the report is fetched must be specified. An ADOM on the fetch client must be specified or a new ADOM can be created if needed. 

				If logs are fetched into an existing local ADOM, you must ensure that the ADOM has enough disk space for the incoming logs. 
				The data policy for the local ADOM on the client must also support retrieving logs from the specified time period. It should Keep archive and analytics logs long enough to avoid deletion in accordance with policy. 
				
				For example:
					Today, July 1, the ADOM data policy is configured to retain analytics logs for 30 days (June 1-30), and you must Download logs from the first week of May. 

					The ADOM data policy should be configured to retain analytics and archive logs For a minimum of 62 days to cover the entire period. Otherwise, the fetched logs will be automatically deleted after deletion  pick up

				after create this in profile page can see request to fetch after click on it 
					secure connection with ssl
					server adom
					local adom
					devices
						device 
						vdom

					enable filters
					time period
					index fetch logs
						fetched logs will be indexed in the SQL database of the client once theyare received. Select this option unless you want to manually index the fetched logs 

				if need synchronization between adom and devices
					system setting > fetcher management
						profiles
							right click on devices and select synch

							*If a new ADOM is created, the new ADOM will reflect the disk space and data policy of the corresponding ADOM server. If there is not enough space on the client, the client creates an ADOM with the maximum allowed disk space and a warning message. Then you can adjust the disk space allocation as needed.

				after these our server recieve request must approve them then will be reachable from session part

				* It can take a long time for the client to finish indexing the fetched logs and make the analyzed  data available. 
				A progress bar is shown in the GUI banner; for more information, click on it to  open the Rebuild Log Database dialog box.

				Log and report features will not be fully available until the rebuilding process is complete
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	event logs
		The Event Log pane provides an audit log of actions made by users on FortiAnalyzer
		view log messages that are stored in memory or on the internal hard disk drive

		Type  
		 	Event Log
		 	FDS Upload Log

		 	FDS Download Log (FDS,orFCT)
		 		All Event,Push Update,Poll Update, or Manual Update and then click Go to browse the logs.
		 T
			this optionis only available when viewing historical logs

		Thelogsub-type:
			page 202 have categories
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	task manager
		Using the task monitor ,you can view the status of the tasks you have performed

		system setting > task monitor
			Status
				Done: Completedwithsuccess.
				Error: Completed without success.
				Canceled: Usercanceled the task.
				Canceling: Useriscanceling the task.
				Aborted: TheFortiAnalyzer system stopped performing thistask.
				Aborting: The FortiAnalyzer system is stopping performing this task.
				Running: Beingprocessed. In thisstatus, a percentage bar appearsin the Status column.
				Pending
				Warning
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	snmp
		SNMP has two parts- the SNMP agent that is sending traps, and the SNMP manager that monitors those traps. The  SNMP communities on monitored FortiGate devices are hard coded and configured by the FortiAnalyzer system- they  are not user configurable

		The FortiAnalyzer SNMP implementation is read-only — SNMPv1, v2c, and v3 compliant SNMP manager applications,  such as those on your local computer, have read-only access to FortiAnalyzer system information and can receive  FortiAnalyzer system traps

		System Settings > Advanced > SNMP
			*These SNMP communities do not refer to the FortiGate devices the FortiAnalyzer system is managing.

		config system snmp sysinfo
		set trap-high-cpu-threshold <percentage value>

		set trap-cpu-high-exclude-nice-threshold <percentage value>
 		
 		set trap-low-memory-threshold <percentage value>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	device logs
		The FortiAnalyzer allows you to log system events to disk
		can control device log file size and the use of the FortiAnalyzer unit’s disk space by configuring log rolling and scheduled uploads to a server.

		FortiAnalyzer unit receives new log items, it performs the following tasks:
			Verifies whether the log file has exceeded its file size limit
			Checks to see if it is time to roll the log file if the file size is not exceeded

		When acurrent log file (tlog.log) reaches its maximum size, or reachesthe scheduled time, the FortiAnalyzer unit  rolls the active log file by renaming the file. 

		The file name will be in the form of xlog.N.log (tlog.1252929496.log), where x is a letter indicating the log type and N is a unique number corresponding to the time the first log entry was received. 

		The file modification time will match the time when the last log was received in the log file.
		Once the current log file is rolled into a numbered log file, it will not be changed. 
		New logs will be stored in the new current log called tlog.log. If log uploading is enabled, once logs are uploaded to the remote server or downloaded via the GUI, theyare in the following format:  
			FG3K6A3406600001-tlog.1252929496.log-2017-09-29-08-03-54.gz

		If you have enabled log uploading, you can choose to automatically delete the rolled log file after uploading, there by freeing the amount of disk space used by rolled log files. If the log upload fails, such as when the FTPserver is unavailable, the logs are uploaded during the next scheduled upload.
		Log rolling and uploading can be enabled and configured using the GUI or CLI.

		config system log settings
		config rolling-regular
		set upload enable\disable

		config system log settings
		config rolling-regular
		set file-size 300


		disable rolling
			config system log settings
			config rolling-regular
			set when none 

		daily rolling
			config system log settings
			config rolling-regular
			set upload enable
			set when daily
			set hour <integer>
			set min <integer>

		weekly rolling
			config system log settings
			config rolling-regular
			set when weekly
			set days {mon | tue | wed | thu | fri | sat | sun}
			set hour <integer>
			set min <integer>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	device manager :
		add devices as well as vdoms to fortianalyzer
		after adding and registering the above devices, fortianalyzer will start collecting logs from these devices. you can also send added logs to other devices by making settings.

		using the fortianalyzer device to manage the fortigate device :
			by enabling the fortimanager feature on the fortianalyzer device, you can manage fortigate devices for configuration. using this feature, you can enable all the features in the fortimanager device except the fortiguard feature on the fortianalyzer device. with the free license found on the fortianalyzer device and by enabling the fortimanager feature, you can manage two fortigate devices. by purchasing a management license, you will be able to manage up to 20 fortigate devices using the fortianalyzer device and make the desired settings on them.
				fortianalyzer > dashboard > system info > fortimanager features > then reboot
		
		right-click on the desired device that you have previously added to fortianalyzer and select the connect to device option. you will see that a new tab will open in the browser, and you can connect to the desired device by entering the authentication information.

		if fortimanager features are enabled, then this is the fortimanager device manage

		**fortianalyzer, forticache, forticlient, fortiddos, fortimail, fortimanager, fortisandbox, fortiweb, chassis, and forticarrier devices are automatically placed in their own adoms

		how add a device on fortianalyzer
			on the device manager click on add device then :
				in the ip address field, you must enter the address of the desired device that you intend to add to fortianalyzer.
				enter the serial number of the above device in the sn field.
				choose a name for the above device by the device name field.
				by entering the serial number of the above device model, it will be automatically displayed in the device model field.
				in the firmware version field, you must specify the firmware version of the above device
	
		at the same deployement concept must setup some config on fortigate or fortiweb device for log transfering and analytics tool 

		https://community.fortinet.com/t5/fortigate/technical-tip-connectivity-issue-between-fortigate-and/ta-p/205112
		https://community.fortinet.com/t5/fortianalyzer/troubleshooting-tip-fortigate-to-fortianalyzer-connectivity/ta-p/191833

		if our fortigates are on ha must specify the fortigate hagroup name when adding a fortigate cluster 
			these setting can be visible on edit tab in device manager and ha cluster

			set our log setting and send logs to forti-analayzer

		unregistered
			fortianalyzer 5.2.0 and later, the config system global set unregister-pop-upcommand is disabled by default. 
			when a device is configured to send logs to fortianalyzer, the unregistered device is displayed in the device

			manager> devices unregistered

		if have problem on fgt and faz connectivity 
			on fgt
				diag sniffer packet any 'host 192.168.150.250 and port 514' 6 0 1
			on faz
				diag sniffer packet any 'host 192.168.150.250 and port 514' 6 0 1

		fortimanager and fortianlayzer
			can add fortianalyzer devices to fortimanager and manage them
			fortimanager automatically enables fortianalyzer features
			fortianalyzer and fortimanager must be running the same osversion, at least 5.6 or later

			* in the device manager pane, a message informsyou the device is managed byfortimanagerand all changes should  be performed on fortimanager to avoid conflict. 

			after this we see lock icon on all adom panel mens managed by fortimanager
			logsare stored on the fortianalyzer device, not the fortimanager device. you configure log storage settings on the fortianalyzer device; you cannot change log storage settings using fortimanager

			central management :
				config system central-management
				set type {fortimanager}
				set allow-monitor {enable | disable}
				set authorized-manager-only {enable | disable}
				set serial-number <serial_number_string>
				set fmg <string>
				set enc-alogorithm {default | high | low}
				end

		**when adoms are enabled, can assign the device to an adom. when manually adding multiple devices at one  time, they are all added to the same adom.
		when you delete adevice or vdom from the fortianalyzer unit, its raw log files are also deleted. sql database logs are not deleted.

		adding by security fabric on fgt 
			page 80

		tac command :
			the tac report will collect useful information such as:
			serial number.
			firmware version.
			fortiguard updates state.
			memory and cpu usage.
			global configuration.
			hardware features.
			interface errors.
			traffic statistics.
			ha diagnostics.
			process crash log

			load data from fortigate

			execute tac report
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	event management : 
		the event handler displays all the events that are generated by the event handler. 

		event handler determines what information is extracted from logs and displayed in event management.
		
		with acknowledge an event, you will actually confirm the occurrence of the above event. 

		it should be noted that after the acknowledge action, the above event will be removed from the list of recent events

		event management displays events from analytics logs, not archive logs

		you can use predefined event handlers to generate events for event management. 
		there are predefined event handlers for fortigate and forticarrier devices. for other devices, you can create custom event handlers.

		event handlers define what messages to extract from logs and display in event management. you must enable an event handler to start generating events.

		you can configure event handlers to generate events for a specific device, for all devices, or for the local fortianalyzer unit. you can create event handlers for fortigate, forticarrier, forticache, fortimail, fortimanager, fortiweb, fortisandbox devices, and syslog servers. in 5.2.0 or later, event management supports local fortianalyzer event logs.

		you can configure the system to send you alerts for event handlers. you can send the alert to an email address, snmp community, or syslog server

		page 125 we can find some predefine models for fgt and forticarrier or customize them

		custom event handler
			the generic text filter uses the glibc regex library for values with operators (~,!~), using the posix standard.

			filter string syntax is parsed by fortianalyzer, and both upper and lower case characters are supported (for example "and" is the same as "and"). you must use an escape character when needed. for example, cfgpath=firewall.policy is the wrong syntax because it's missing an escape character.

			the correct syntax is cfgpath=firewall\.policy.

			we can make copy from logs and generate some custom event handler
				log view > tools > displaye raw
					the easiest method is to copy the text string you want from the raw log and paste it into the generic text filter field. ensure you insert an escape character when necessary, for example, cfgpath=firewall\.policy.

					even amnagement > event handler > custom
						in the generic text filter box, paste the text you copied or type the text you want. ensure you use the raw log field names, for example, mem (not memory) and setuprate (not setup-rate).
						for information on text format and operators, hover the cursor over the help icon. the operator ~ means contains and !~ means does not contain

		we can select reset to factory and restore our default attributes

		events
			after event handlers start generating events, you can view events and event details. 

			event management > all events 
				shows events by type and severity in a graphical format, and recent events in a tabular format. 

				event management > calendar view 
				shows events by month or week in a calendar or bar chart format.

			when rebuilding the sql database, you might not see a complete list of historical events.
			however, you can always see events in real-time logs. you can view the status of the sql rebuild by checking the rebuilding db status in the notification center.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	reports: 
		with this option, you can perform reporting, define the format of reports, schedules, and output profiles. we can use macros :
		
		reported files are stored in a reserved space on the fortianalyzer device
		when rebuilding sql database our reports are not available

		*each adom has its own reports, libraries, and advanced settings.
		*reports uses analytics logs to generate reports. archive logs are not used to generate reports
		
		fortianalyzer > report > report difinition
			all reports
				produce your desired reports directly and with minimum settings
				predefine reports are using report template with basic mode 

			templates
				report templates include charts and macros and specify the report designer
				can use directly or build upon
				a template populates the layout tab of a report that is to be created.

			charts
				select chart library. 
				this item is available to you in the design of the report or the report format that you are creating and you can use it. 
				the charts specify what data and information to extract from the logs

			macros
				select macro library.
				this item is available and you can use it in the report design or report template that you are creating. 
				macros specify what data and information to extract from logs

		*in fact, when you create a report, the datasets related to each chart and macro are extracted from the data and information in the log.

		how extract
			reports include charts and/or macros. each chart and macro is associated with a dataset. when you generate a report, the dataset associated with each chart and macro extracts data from the logs and populates the charts and macros.
			each chart requires a specific log type.

			here we can use some tools like schedule reports and run report then export them like html pdf xml csv format

			auto-cache
				when you generate a report, it can take days to assemble the required dataset and produce the report, depending on the required datasets. instead of assembling datasets at the time of report generation, you can enable the auto-cache feature for the report.

				auto-cache is a setting that tells the system to automatically generate hcache. 
				the hcache (hard cache) means that the cache stays on disk in the form of database tables instead of memory. hcache is applied to “matured” database tables. 

				when a database table rolls, it becomes “mature”, meaning the table will not grow anymore. therefore, it is unnecessary to query this database table each time for the same sql query, so hcache is used. hcache runs queries on matured database tables in advance and caches the interim results of each query. when it is time to generate the report, much of the datasets are already assembled, and the system only needs to merge the results from hcaches.

				this reduces report generation time significantly.

				the auto-cache process uses system resources to assemble and cache the datasets and it takes extra space to save the query results. you should only enable auto-cache for reports that require a long time to assemble datasets.

				fortianlyzer > reports > report definition > all reports > select edit on top  > setting > autocache 

				*group report
					reduce the number of hcache tables
					improve auto-hcache completion time
					improve report completion time

					config system report group
					edit 0
					set adom root
					config group-by (controls how cache tables are grouped)
					edit devid
					next
					edit vd
					next
					end
					set report-like security_report (report-like is report title used grouping + case sensetive)
					next
					end

					how see our report
						execute sql-report list-schedule <adom>

					initiate a rebuild of hcache tables
						diagnose sql hcache rebuild-report <start-time> <end-time>

						where <start-time> and <end-time> are in the format: <yyyy-mm-dd hh:mm:ss>

			table type: this field has regular, ranked, and drilldown options
				if you choose regular, you can add a maximum of 15 columns to the table. 
				this value is 2 or 3 columns, respectively, if you choose the ranked or drilldown options

					bundle rest into others: by activating this option, the rest of the items will be placed in a group named other. this option can be set only if the option ranked or drilldown is selected for table type.

		diagnostic logs for retrieve reports
			once you start to run a report, fortianalyzer creates a log about the report generation status and system performance.
			use this diagnostic log to troubleshoot report performance issues. 

			if your report is very slow to generate, you can use this log to check system performance and see which charts take the longest time to generate.

			reports > generated reports 
				right click on reports then select retrieve diagnostic 
				download attributes

		the types of logs available for the fortigate device are: 
			application control, intrusion prevention, content log, data leak prevention, email filter, event, traffic, virus, voip, web filter, vulnerability scan, fct event, fct traffic, fct vulnerability scan. web application firewall and gtp.

		the types of logs available for the fortimail device are: 
			email filter, event, history and virus

		the types of logs available for the fortiweb device are: 
			intrusion prevention, event and traffic

		auto generated report
			the cyber threat assessment report is automatically generated. by default, the report will run at 3:00am every monday

			*this will only affect newly installed fortianalyzer or newly created adom. upgraded adom reports, scheduling and calendar will be kept as is.

		schedule reports are available in reports > report definitions all reports must select one task then change setting

		* exporting reports only exports the report layout, charts, datasets, and images. other report  configurations are not exported.

		in templates we have some attributes for 
			fgt
				template-360-degreesecurityreview 		 		template-securityanalysis
				----------------------------------------------------------------------------
				template-adminandsystemeventsreport 	 		template-threatreport
				----------------------------------------------------------------------------
				template-applicationriskandcontrol		 		template-top20categoriesandapplications(session)
				----------------------------------------------------------------------------
				template-bandwidthandapplicationsreport  		template-top20categoryandwebsites(bandwidth)
				----------------------------------------------------------------------------
				template-clientreputation 				 		template-top20categoryandwebsites(session)
				----------------------------------------------------------------------------
				template-cyberthreatassessment 			 		template-top500sessionsbybandwidth
				----------------------------------------------------------------------------
				template-dnsreport 						 		template-topallowedandblockedwithtimestamps
				----------------------------------------------------------------------------
				template-datalosspreventiondetailedreport 		template-userdetailedbrowsinglog
				----------------------------------------------------------------------------
				template-detailedapplicationusageandrisk 		template-usersecurityanalysis
				----------------------------------------------------------------------------
				template-emailreport 							template-usertop500websitesbybandwidth
				----------------------------------------------------------------------------
				template-forticlientdefaultreport 				template-usertop500websitesbysession
				----------------------------------------------------------------------------
				template-forticlientvulnerabilityscanreport 	template-vpnreport
				----------------------------------------------------------------------------
				template-fortigateperformancestatisticsreport 	template-webusagereport
				----------------------------------------------------------------------------
				template-gtpreport 								template-what isnewreport
				----------------------------------------------------------------------------
				template-hourlywebsitehits 						template-wifinetworksummary
				----------------------------------------------------------------------------
				template-ipsreport 								template-wirelesspcicompliance
				----------------------------------------------------------------------------
				template-pci-dsscompliancereview				template-saasapplicationusagerepor
			+++++
			fwb
				template-fortiwebdefaultreport
				----------------------------------------------------------------------------
				template-fortiwebwebapplicationanalysisrepor

		chart library
			you can also create charts using the logview chart builder
			reports > report definitions > chart library

			*bundle rest into "others"
				select to bundle the rest of the results into an others category

		macro
			macros are predefined to use specific datasets and queries. 
			they are organized into categories, and can be added to, removed from, and organized in reports.

			*macros are currently supported in fortigate and forticarrier adoms only

		dataset
			reports > report definitions > datasets
			for fortigate:
				application control, intrusion prevention,content log,data leak prevention,email filter, event,traffic,virus,voip,web filter,vulnerability scan,forti client event,forti client traffic, forticlient, vulnerability scan,web application firewall,gtp,dns,and local event.

			fortimail:
				email filter,event, history,and virus

			fortiweb:
				intrusion prevention, event and traffic

			sql standard query
				root_domain(hostname) the root domain of the fqdn.an example of using this function is:
					select devid, root_domain(hostname)as website from $log
					where'user'='user01' group by devid , hostname order by hostname limit 7
				
				nullifna(expression) this is the inverse operation of coalesce that you can use to filter out n/a
					values.
					this function takes an expression as an argument.
					the actual sqlsyntax this is base on is select nullif(nullif(expression, 'n/a'), 'n/a').
					
					in the following example, if the user is n/a,the source ip is returned ,
					otherwise the username is returned.

					select coalesce(nullifna('user'),nullifna('srcip')) as user_src,coalesce(nullifna(root_domain(hostname)),'unknown') as 	domain from $log where dst port='80' group by user_src, 	domain order by user_src limit 7
				
				email_domain
				email_user
					email_domain returns the text after the @ symbol in an email address.
					email_user returns the text before the @ symbol in an email address.
					 
					select 'from' as source, email_user ('from')as e_user,email_domain('from')as e_domain from $log limit5 offset 10
				
				from_dtime
				from_itime
					from_dtime(bigint)returns the device time stamp without timezone.
					from_itime(bigint) returns fortianalyzer’s timestamp without timezone.
					
					select itime, from_itime(itime)as faz_local_time, dtime, from_dtime(dtime)as dev_local_time from $log limit 3

		output profile
			output profiles allow you to define email addresses to which generated reports are sent and provide an option to upload the reports to ftp, sftp, or scp servers.
			once created, an output profile can be specified for a report

			*must configure a mail server before you can configure an output profile

			reports > advanced> output profile

			langauge are not changeable 
				adding a new language placeholder does not create that language. 
				it only adds a placeholder for that language that contains the language name and description

		can only delete or download scheduled report sthat have a finished status. 
		can not delete scheduled reports with a pending status.
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	fortiview :
		fortiview is a comprehensive monitoring system for your network that integrates real-time and historical data into a single view. it can log and monitor threats to networks, filter data on multiple levels, keep track of administrative activity, and more

		config system fortiview setting
		set not-scanned apps include
		set resolve-ip enable
		end

		config system fortiview auto-cache
		set aggressive-fortiview enable
		set interval 300
		set status  enable 
		end

		summary
			filtered summary view or its drilldown, click the export button in the top-right and select export to pdf or export to report chart

			only log field filters are exported. device and time period filters are not exported.
		-------------------------------------------		
		threats: 
			this item provides information about the security threats that the network is facing and includes
			
			top threats: 
				lists the security threats that have occurred more than other threats. risk applications, intrusion, malicious websites, malware/botnets can be mentioned among the incidents known as threats.

				*if fortigate is running fortios 5.0.x, turn on security profiles > client reputation to view entries in top threats.
	
			threat map: 
				contains a map on which the countries that are considered traffic destinations more often are distinguished by color.

				list of threats at the bottom shows the location, threat, severity, and time of the attacks

			indicators of compromise (ioc):
				 users who are using the web incorrectly and suspiciously.

				users’ ip addresses, overall threat rating, and number of threats

				**utm logs of the connected fortigate devices must be enabled.
				the fortianalyzer must subscribe to fortiguard to keep its threat database current.

				to use this indicators of compromise summary, you must turn on the utm web filter of fortigate devices. 
				must also subscribe your fortianalyzer unit to fortiguard to keep its local threat database synchronized with the fortiguard threat database.
		-------------------------------------------
		traffic: 
			this item displays information about network traffic and includes the following:
			
			top sources: 
				 most used network traffic based on source ip address. in this list, components such as interface, device, threat score, session and volume of sending and receiving traffic based on bytes can be seen for each item.

			top destinations: 
				 most used network traffic based on destination ip address. in this list, components such as applications, sessions, and the volume of sending and receiving traffic based on bytes can be seen for each item.
		
			top countries: 
				 most used network traffic by country. in this list, components such as sessions, destination, threat score, sessions and the volume of sending and receiving traffic based on bytes can be seen for each item.

			policy hits: 
				 traffic based on policy. in this list, components such as the device to which the policy belongs, the corresponding vdom, the volume of sending and receiving traffic based on bytes, the number of times the policy has been used, and the time and date of the last time the policy was used for each policy can be seen. is.
		-------------------------------------------
		application & websites: 
			this item displays information related to the use of application and websites and includes 

			top applications: 
				applications that are used more than other applications in the network.

				application name, category, risk level, number of clients, sessions blocked and allowed, and bytes sent and received.
	
			top cloud applications: 
				cloud applications that are used more than other applications in the network.
			
			top websites: 
				authorized and unauthorized websites with the highest number of visits.
	
			top browsing users: 
				users who have searched the web the most. in this list, components such as the number of visited sites, search time, volume of sending and receiving traffic based on bytes, etc. can be seen.
		-------------------------------------------
		vpn: 
			this item provides information about the wifi network and includes 
			
			ssl & dialup ipsec: 
				users who have accessed the network through vpn and using ssl and ipsec security platforms.
	
			site-to-site ipsec: 
				names of vpns and ipsecs through which the network can be accessed.
		-------------------------------------------	
		wifi:
			this item provides information about the users using the vpn as well as the platform that secures the vpn and includes
	
			rogue aps: 	
				ssids of unauthorized access points in the network.
	
			authorized aps: 
				authorized access points in the network.
	
			authorized ssids: 
				ssids of authorized access points in the network.
	
			wifi clients: 
				name and ip address of devices connected to the wifi network.
		-------------------------------------------	
		system:
			 this item provides information about the performance and events of devices added to fortianalyzer and includes
	
			admin logins: 
				network administrators who have logged in to the device.
	
			system events: 
				events that have occurred on the device.
	
			resource usage: 
				usage of cpu, memory, hard disk and other information related to the operation of the device.
	
			failed authentication attempts: 
				failed authentication attempts to enter the device.
		-------------------------------------------
		endpoints
			all endpoints 
				endpoints registered to the fortigate device

			top vulnerabilities
				vulnerability information about the forticlient endpoints registered to specific fortigate devices. view by device or vulnerability.

				in device view, the table shows the device, source, number and severity of vulnerabilities, and category.
				in vulnerability view, select table or bubble format. the table format shows the vulnerability name, severity, category, cve id, and host count. 

				the bubble graph format shows vulnerability by severity and frequency.

			top threats 
				threats for registered forticlient endpoints, including the threat, threat level, and the number of incidents (blocked and allowed).
			
			top applications
				top applications used by registered forticlient endpoints, including the application name, risk level, sessions blocked and allowed,and bytes sent and received.
			
			top web sites 
				allowed and blocked web sites on the network

		page 104 example
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	noc
		displays both real-time monitoring and historical trends
		centralized monitoring and awareness help to effectively monitor network events, threats, and security alerts
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	logview :
		when rebuilding the sql database, log view is not available until the rebuild is complete.

		when adoms are enabled, each adom has its own information displayed in log view.

		log view displays log messages from analytics logs and archive logs:
			historical logs and real-time logs in log view are from analytics logs.
			log browse can display logs from both the current, active log file and any compressed log files.

		device type 		|		log type
		-----------------------------------------------------------------------------------------------
		fortianalyzer 		|		event
		-----------------------------------------------------------------------------------------------
		fortiauthenticator 	|		event
		-----------------------------------------------------------------------------------------------
		fortigate 					traffic
									+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
									security: antivirus, intrusion prevention, application control, web filter, dns, data leak
									+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
									prevention, email filter, web application firewall, vulnerability scan, voip, forticlient
									+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++				
									event: endpoint, ha, compliance, system, router, vpn, user, wan opt. & cache, wifi
		-----------------------------------------------------------------------------------------------		
		forticarrier 		|		traffic, event, gtp
		-----------------------------------------------------------------------------------------------
		forticache 			|		traffic, event, antivirus, web filter
		-----------------------------------------------------------------------------------------------
		forticlient 		|		traffic, event, vulnerability scan
		-----------------------------------------------------------------------------------------------
		fortiddos 			|		event, intrusion prevention
		-----------------------------------------------------------------------------------------------
		fortimail 			|		history, event, antivirus, email filter
		-----------------------------------------------------------------------------------------------
		fortimanager 		|		event
		-----------------------------------------------------------------------------------------------
		fortisandbox 		|		malware, network alerts
		-----------------------------------------------------------------------------------------------
		fortiweb 			|		event, intrusion prevention, traffic
		-----------------------------------------------------------------------------------------------
		syslog 				|		generic
		-----------------------------------------------------------------------------------------------

		traffic log
			this type of log records information about the traffic flowing through the fortigate device. since the transmission of traffic by the fortigate device requires the creation of a series of policies, this type of log is also known as firewall policy logging. these policies control all the traffic that flows between the interfaces, zones and vlan sub-interfaces of the fortigate device.

		event log
			this type of log records information related to the activities of network administrators on devices as well as system activities of fortinet devices. 
			when changes are made in the configuration and settings, or when network managers log in to the system, or events related to ha
			these logs are important because they record the system activities of fortinet devices and subsequently provide information about how the device is functioning

			fortigate event logs includes
				system, router, vpn, user, and wifi menu objects to provide you with more granularity when viewing and searching log data.

		dns log (fgt)

		security log
			this type of log is recorded only for fortigate devices. this log sample records information related to antivirus, application control, ips, dlp, vulnerability scan, email filtering, web filtering and voip activities.

		*to display the log for devices other than non-fortigate, adom must be enabled on the fortianalyzer device.

		filter logs 
			config system sql
			config ts-index-field
			edit "fgt-traffic"
			set value "app,dstip,proto,service,srcip,user,utmaction"
			next
			end

			page 117 has example for log conditions

		real time logs and historical logs are available in tools part of viewing
		also our chart builder + download our logs or ... is available in this part
			you can choose the format of the downloaded file using the log file format field. this field has text, native and csv options.
			
			by activating the compress with gzip option, the downloaded file will be compressed

		log group
			fortiview summaries, display logs, generate reports, or create handlers for a log group. log groups are virtual so they do not have sql databases or occupy additional disk space.

			in fortianalyzer 5.0.6 and earlier, you can treat log groups as a single device that has its own sql database. you cannot do this in fortianalyzer 5.2 and later

		log browse
			log file reaches its maximum size or a scheduled time, fortianalyzer rolls the active log file by renaming the file.

			the file name is in the form of xlog.n.log, where x is a letter indicating the log type, and n is a unique number corresponding to the time the first log entry was received

			importing file for log
				imported log files can be useful when restoring data or loading log data for temporary use. for example, if you have older log files from a device, you can import these logs to the fortianalyzer unit so that you can generate reports containing older data.
				
				to insert imported logs into the sql database, the config system sqlstart-time and rebuild-event-start-time must be older than the date of the logs that are imported and the storage policy for analytic data (the keep logs for analytics field) must also extend back far enough.

				config system sql
				set start-time <start-time-and-date>
				set rebuild-event-start-time <start-time-and-date>
				end
				
				*where <start-time-and-date> is in the format hh:mm yyyy/mm/dd.

				we can use logview > log setting > import
					in the device dropdown list
						select the device the imported log file belongs to or select [take from imported file] 	to read the device id from the log file.

						if you select [take from imported file], the log file must contain a device_id field in its log messages.

					don't leave page cause break operation

					**if you selected [take from imported file] and the fortianalyzer unit’s device list does not currently contain that device, a message appears after the upload. click ok to import the log file and automatically add the device to the device list.

		locallog setting :
			config system locallog setting
			set log-interval-dev-no-logging 5 (is default)
			set log-interval-disk-full 5 (is default)
			set log-interval-gbday-exceeded 1440 (is default)
			end

		locallog filter (all of them are enable) :
			config system locallog [memory | disk | fortianalyzer | fortianalyzer2 | fortianalyzer3 | syslogd | syslogd2 | syslogd3] filter
			set devcfg (device config)
			set devops (managed devices operations messages)
			set diskquota (logging fortianalyzer disk quota messages)
			set dm (deployment manager messages)
			set dvm (log device manager messages)
			set ediscovery (logging device manager messages)
			set epmgr (log endpoint manager messages )
			set event (configure log filter messages)
			set eventmgmt (logging fortianalyzer event handler messages)
			set faz (log fortianalyzer messages)
			set fazha (log fortianalyzer ha messages)
			set fazsys (logging fortianalyzer system messages)
			set fgd (log fortiguard service messages )
			set fgfm (log fortigate/fortianalyzer communication protocol messages)
			set fips (log fortigate/fortianalyzer communication protocol messages)
			set fmgws (log web service messages)
			set fmlmgr (log fortimail manager messages)
			set fmwmgr (log firmware manager messages)
			set fortiview (logging fortianalyzer fortiview messages)
			set glbcfg (log global database messages)
			set ha (log high availability activity messages)
			set hcache (logging hcache messages)
			set iolog (input/output log activity messages)
			set logd (logd messages)
			set logdb (logging fortianalyzer log db messages)
			set logdev (logging fortianalyzer log device messages)
			set logfile (logging fortianalyzer log file messages)
			set logging (logging fortianalyzer logging messages)
			set lrmgr (log log and report manager messages )
			set objcfg (log object configuration)
			set report (logging fortianalyzer report)
			set rev (log revision history messages)
			set rtmon (log real-time monitor messages)
			set scfw (log firewall objects messages )
			set scply (log policy console messages)
			set scrmgr (log script manager messages)
			set scvpn (log vpn console messages)
			set system (log system manager messages)
			set webport (log web portal messages)
			end

		locallog fortianalyzer :
			config system locallog fortianalyzer setting
			set status enable / realtime / upload (schedule)
			set severity information
			end

		locallog memory setting :
			config system locallog memory setting
			set diskfull nolog/overwrite (nolog: stop logging when disk full / overwrite: overwrites oldest log entries)
			set severity information
			set status enable
			end

		locallog syslogd setting :
			config system locallog {syslogd | syslogd2 | syslogd3} setting
			set csv {disable | enable}
			set facility {alert | audit | auth | authpriv | clock | cron | daemon | ftp | kernel | local0 | local1 | local2 | local3 | local4 | local5 | local6 | local7 | lpr | mail | news | ntp | syslog | user | uucp}
			set severity {emergency | alert | critical | error | warning | notification | information | debug}
			set status {enable | disable}
			set syslog-name <string>
			end

		log based ioc (indicators of compromise)
			config system log ioc
			set notification enable (default)
			set notification-throttle <integer>
			set status
			end

		log alert
			config system log alert
			set max-alert-count <integer>
			end

		log mail-domain
			use this command to enable restrictions on email domains. by default, this option is disabled. the logs for different email domains are stored in the same adom.
			when this option is enabled through the cli, fortianalyzer identifies the email doamins from the logs. 
			it creates a list of vdoms in the device manager based on the email domains. the vdoms are assigned to different adoms. when inserting a log to the database, fortianalyzer records the log to its corresponding adom based on the email domain information in the log. the vdom field of the log is sent to the email domain name.

				conf system log mail-domain
				edit 1
				set domain company-name.
				set code name.com
				set device all_fortimails
				next
				edit 2
				set domain network-cnet
				set code cnet.net
				set device fe00000000000001
				next
				edit 3
				set domain mail.myfortinet.com
				set code myftntmail
				set device fe00000000000002,fe00000000000003
				end

		log settings
			config system log settings
			set dns-resolve-dstip enable
			set log-file-archive-name {basic | extended}
				log file name format for archiving.
				basic: basic format for log archive file name (default), for example:
				fgt20c0000000001.tlog.1417797247.log.
				
				extended: extended format for log archive file name, for example:
				fgt20c0000000001.2014-12-05-08:34:58.tlog.1417797247.log.

			set syn-search-timeout 60 (default)
			set ha-auto-migrate enable
				automatically merging ha member's logs to ha cluster

			config rolling-regular
				del-files enable (log file deletion after uploading)
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	autodelete
		logs and files are stored on the fortianalyzer hard disks. logs are also temporarily store in the sql database.

		disk space allocation
			small disk (up to 500gb) > 20% or 50gb of disk space, whichever is smaller.

			medium disk (up to 1tb) > 15% or 100gb of disk space, whichever is smaller.

			large disk (up to 5tb) > 10% or 200gb of disk space, whichever is smaller.

			very large disk (bigger than 5tb) > 5% or 300gb of disk space, whichever is smaller.

			*the raid level you select determines the disk size and the reserved disk quota level. 
			fortianalyzer 1000c with four 1tb disks configured in raid 10 is considered a large disk, so 10%, or 200gb, of disk space is reserved.

		log and file flow
			logs are compressed and saved in a log file on the fortianalyzer disks.
			when a log file reaches a specified size, fortianalyzer rolls it over and archives it, and creates a new log file to receive incoming logs. you can specify the size at which the log file rolls over

			logs are indexed in the sql database to support analysis.
			you can specify how long to keep logs indexed using a data policy

			logs are purged from the sql database, but remain compressed in a log file on the fortianalyzer disks

			logs are deleted from the fortianalyzer disks. can specify how long to keep logs using a data policy.

			*in the compressed phase, logs are compressed and archived in fortianalyzer disks for a specified length of time for the purpose of retention. compressed, or archived, logs are considered offline, and their details cannot be immediately viewed or used to generate reports.

			in online mode or indexing part we can see our live logs on many panels

			log phase 				 location 							immediate analytic support
			------------------------------------------------------------------------------------------------------------
			indexed 				compressed in log file and 			yes. logs are available for analytic use in 
									indexed in sql database				fortiview,noc, event management, and reports.
			-------------------------------------------------------------------------------------------------------------
			compressed 				compressed in log file 				no.

		
		auto delete
			global automatic file deletion
			file management settings specify when to delete the oldest archive logs, quarantined files, reports, and archived
			files from the disks, regardless of the log storage settings.

			data policy
			data policies specify how long to store analytics and archive logs for each device. when the specified length of time expires, archive logs for the device are automatically deleted from the fortianalyzer device's disks.

			disk utilization
			disk utilization settings delete the oldest archive logs for each device when the allotted disk space is filled. the allotted disk space is defined by the log storage settings. alerts warn you when the disk space usage reaches a configured percentage.


			when you delete one or more devices from fortianalyzer, the raw log files and archive packets are deleted, and the action is recorded in the local event log. however, the logs that have been inserted into the sql database are not deleted from the sql database.
			as a result, logs for the deleted devices might display in the log view and fortiview panes, and any reports based on the logs might include results.

			the following are ways you can remove logs from the sql database for deleted devices.
			 	rebuild the sql database for the adom to which deleted devices belonged or rebuild the entire sql database.
			 
			 	configure the log storage policy. when the deleted device logs are older than the keep logs for analytics setting, they are deleted. also, when analytic logs exceed their disk quota, the sql database is trimmed starting with the oldest database tables. 

			configure global automatic file deletion settings in system settings > advanced > file management. when the deleted device logs are older than the configured setting, they are deleted. for more information,  

			*file management configures global settings that override other log storage settings and apply to all adoms.

			log storage policy affects on log and sql database of the devices associated with the log storage policy.
			reports are not affected

			if enable adom we can see storage info on system setting > storage info
				*if you change log storage settings, the new date ranges affect analytics and archive logs currently in the fortianalyzer device. 
				depending on the date change, analytics logs might be purged from the database, archive logs might be added back to the database, and archive logs outside the date range might be deleted.

			config system auto-delete
			config dlp-files-auto-deletion
			set retention {days | weeks | months}
			set runat <integer>
			set status {enable | disable}
			set value <integer>
			end
	
			config quarantine-files-auto-deletion
			set retention {days | weeks | months}
			set runat <integer>
			set status {enable | disable}
			set value <integer>
			end
	
			config log-auto-deletion
			set retention {days | weeks | months}
			set runat <integer>
			set status {enable | disable}
			set value <integer>
			end
	
			config report-auto-deletion
			set retention {days | weeks | months}
			set runat <integer>
			set status {enable | disable}
			set value <integer>
			end
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	advance setting
		adom mode
		download wsdl file
			Whenselecting Legacy Operations, no other optionscan be selected.
			Web services is a standards-based, platform independent, access method for  other hardware and software APIs. 
			The file itself defines the format of commands  the FortiAnalyzer will accept as well as the responses to expect. 
			Using the WSDL  file, third-party or custom applications can communicate with the FortiAnalyzer  unit and operate it or retrieve information
		
		task list size	
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	ha :
		Synchronize logs and data securely among multiple FortiAnalyzer units
		Provide real-time redundancy in case a FortiAnalyzer primary unit fails. If the primary unit fails, another unit in the cluster is selected as the primary unit

		FortiAnalyzer HA cluster can have a maximum of four units: one primary unit with up to three backup units. All units in the cluster must be of the same FortiAnalyzer series.
		All units are visible on the network

		All units must run in the same operation mode: Analyzer or Collector.

		the current FortiAnalyzer HA implementation is not supported by some public cloud infrastructures, such as Microsoft Azure, Google Cloud Platform, etc.

		FortiAnalyzer HA only functions under setups where VRRP is permitted

		When devices with different licenses are used to create an HA cluster, the license that allows for the smallest number of managed devices is used.

		system setting > ha 
			configure a cluster, set the Operation Mode of the primary unit to High Availability. 
				stand alone or high availability

			Then add the IP addresses and serial numbers of each backup unit to primary unit peer list. 

			*The IP address and serial number of the primary unit and all backup units must be added to each backup unit's HA configuration. 

			The primary unit and all backup units must have the same Group Name, Group ID and Password.

			can connect to the primary unit GUI to work with FortiAnalyzer. 
			Using configuration synchronization, you can configure and work with the cluster in the same way as you work with a standalone FortiAnalyzer unit

			prefered role 
				If the preferred role is Master, then this unit becomes the primary unit if it is configured first in a new HA cluster. 
				If there is an existing primary unit, then this unit becomes a backup unit.

				The default is Slave so that the unit can synchronize with the primary unit. 
				A backup unit cannot become a primary unit until it is synchronized with the current primary unit.

			log data sync
				is on by default. It provides real-time log synchronization among cluster members

		*FortiAnalyzer HA synchronizes logs in two states: 
			initial logs synchronization 
				When you add a unit to an HA cluster, the primary unit synchronizes its logs with the new unit. After initial sync is complete, the backup unit automatically reboots. 
				After the reboot, the backup unit rebuilds its log database with the synchronized logs.

				You can see the status in the Cluster Status pane Initial Logs Sync column

			real-time log synchronization
				After the initial log synchronization, the HA cluster goes into real-time log synchronization state.

				Log Data Sync is turned on by default for all units in the HA cluster.

				When Log Data Sync is turned on in the primary unit, the primary unit forwards logs in real-time to all backup units. 

				This ensures that the logs in the primary and backup units are synchronized.
				Log Data Sync is turned on by default in backup units so that if the primary unit fails, the backup unit selected to be the new primary unit will continue to synchronize logs with backup units.

				*If you want to use a FortiAnalyzer unit as a standby unit (not as a backup unit), then you don't need real-time log synchronization so you can turn off Log Data Sync.

		If the primary unit becomes unavailable, another unit in the cluster is selected as the primary unit using the following rules:
			All cluster units are assigned a priority from 80 – 120. 
			The default priority is 100. If the primary unit becomes unavailable, an available unit with the highest priority is selected as the new primary unit. 
				
				a unit with a priority of 110 is selected over a unit with a priority of 100.
		
			If multiple units have the same priority, the unit whose primary IP address has the greatest value is selected as the new primary unit. 

				123.45.67.123 is selected over 123.45.67.124

			If a new unit with a higher priority or a greater value IP address joins the cluster, the new unit does not replace (or preempt) the current primary unit.
		
			**If the FortiAnalyzer being replaced is the primary, after replacing it, use execute #fgfm reclaim-dev-tunnel to force FortiGates to connect to the new FortiAnalyzer.

		load balancing feature to make applicated performance
			Because FortiAnalyzer HA synchronizes logs among HA units, the HA cluster can balance the load and improve overall responsiveness. 
			
			Load balancing enhances the following modules:
				Reports
					When generating multiple reports, the loads are distributed to all HA cluster units in a round-robin fashion. 

					When a report is generated, the report is synchronized with other units so that the report is visible on all HA units

				SOC
					Similarly, for SOC, cluster units share some of the load when these modules generate output for their widgets.

		when we are in cluster can we upgrade firmware ?
			can upgrade the firmware of an operating FortiAnalyzer cluster in the same way as upgrading the firmware of a
			standalone FortiAnalyzer unit.

			**Upgrade the backup units first. 
			**Upgrade the primary unit last, after all backup units have been upgraded and have synchronized with the primary unit. 

			When you upgrade the primary unit, one of the backup units is automatically selected to be the primary unit following the rules you set up in If the primary unit fails 

			This allows the HA cluster to continue operating through the upgrade process with primary and backup units.

			During the upgrade, you might see messages about firmware version mismatch. 
			This is to be expected. When the upgrade is completed and all cluster members are at the same firmware version, you should not see this message.

			If firmware versions between cluster members do not match, configuration synchronization is disabled. 
			Other synchronization operations continue to function

			in tis proocdure better use telnet or ssh to control the flow and see logs and states


		config system ha
		set group-id 10
		set group-name ha-analyzer-g1
		set hb-interface
		set hb-interval 90
		set healthcheck {db | fault-test}
		set initial-sync {true | false}
		set initial-sync-threads <integer>
		set load-balance (disable | round-robin}
		set log-sync {enable | disable}
		set mode {a-p | standalone}
		set password <passwd>
		set priority <integer>
		set private-clusterid
		set private-file-quota
		set private-hb-interval
		set private-hb-lost-threshold
		set private-mode
		set private-password
		set vip <ip_address>
		set vip-interface <port>
		config peer
		edit <peer_id_int>
		set ip <peer_ip_address>
		set serial-number <string>
		set status {enable | disable}
		end

		diagnose ha status (System Settings > HA, the Cluster Status)

		--------------------------------------------------------------------------------------------------------
		config synchronization
			Configuration synchronization provides redundancy and load balancing among the cluster units. 
			A FortiAnalyzer HA cluster synchronizes the configuration of the following modules to all cluster units:
				Device Manager
				Incidents & Events
				Reports
				Most System Settings

				contains what :
					Dashboard > System Information Only Administrative Domain is synchronized. All other settings in the
					System Information widget are not synchronized.
					
					System Setting 								Configuration synchronized
					------------------------------------------------------------------------
					All ADOMs 									Yes
					------------------------------------------------------------------------
					Storage Info 								Yes
					------------------------------------------------------------------------
					Network 									No
					------------------------------------------------------------------------
					HA 											No
					------------------------------------------------------------------------
					Admin 										Yes
					------------------------------------------------------------------------
					Certificates > Local Certificates 			No
					------------------------------------------------------------------------
					Certificates > CA Certificates 				Yes
					------------------------------------------------------------------------
					Certificates > CRL 							Yes
					------------------------------------------------------------------------
					Log Forwarding 								Yes
					------------------------------------------------------------------------
					Fetcher Management 							Yes
					------------------------------------------------------------------------
					Event Log 									No
					------------------------------------------------------------------------
					Task Monitor 								Yes
					------------------------------------------------------------------------
					Advanced > SNMP 							Yes
					------------------------------------------------------------------------
					Advanced > Mail Server 						Yes
					------------------------------------------------------------------------
					Advanced > Syslog Server 					Yes
					------------------------------------------------------------------------
					Advanced > Meta Fields 						Yes
					------------------------------------------------------------------------
					Advanced > Device Log Settings 				Yes
					------------------------------------------------------------------------
					Advanced > File Management 					Yes
					------------------------------------------------------------------------
					Advanced > Advanced Settings 				Yes
					------------------------------------------------------------------------
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	fortimanager :
		page 223

		use tcp 541

		can enable FortiManager featureson FortiAnalyzer so it can manage a small number of FortiGate devices. All the
		FortiManager features can be enabled on FortiAnalyzer except FortiGuard.
		free license that comeswith your FortiAnalyzer unit enables it to manage two FortiGate devices when
		FortiManager features are enabled
		can purchase a management license to enable your FortiAnalyzer unit to manage up to 20 FortiGate devices

		upgrade license is supported only on FortiAnalyzer 2U and above devices

		system setting > dashboard > system info > fortimanager

		config system global
		set fmg-status enable

		after these our system get reboot
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	page 226 ports
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
