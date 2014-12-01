You can check the sap-ccms on ABAP/JAVA/TREX-Systems (other systems not tested).
This check don�t use a login to the sap-system. I use the sapcontrol binarie to check the ccms output from sap.
The sapcontrol is installed on every sap-system with a new sap-kernel.

You can find the tool in the following path:
/usr/sap/hostctrl/exe

You can use the sapcontrol for many things. Start,stop, monitoring and so on.....
The check can monitor 
	# sap-ccms 
	# utilization of the abap-processes ( Dialog, Batch, Spool, Update, Update2, Enqueu ).

I have to methodes to use this check.

	1. You must install the hostctrl on the icinga-system.
	   You can use the sapcontrol-binarie directly from icinga-system without ssh and sudo.
	   That is the preferred way!!
	
	2. You can use check_by_ssh and use the sapcontrol on the remote-machine
	   The binary use the account <root> to connect to the system.
	   This is sap-standard!
	   You must configure the sudo command on the remote-machine.
	   The check_plugin should only use with your nagios-user and check per ssh the remote-machine.
	
	   This is the command ->
		sudo su - root -c /usr/sap/hostctrl/exe/sapcontrol

	   The sudo-file on the remote-machine have this entry:
	   %nagios  ALL=(ALL)       NOPASSWD: /bin/su - root -c /usr/sap/hostctrl/exe/sapcontrol -nr * -function GetAlertTree*

	   Adv:		no software installation on the icinga-system
	   DisAdv:	all sapchecks use a ssh-call








Usage:
	check_host_ctrl.pl -host <HOSTNAME only with local sapcontrol> -sysnr <SAP-SYS-NR> -meth <sap|nag|ls|cons> -<obj|function> <MONITORING-OBJEKT> -backend <abap|java|trex|multi> -w <WARNING-LEVEL> -c <CRITICAL-LEVEL> -t <TIME_IN_SEC> -sudo <0|1>
			
Optionen:
	
	-host: HOSTNAME
	
	-sysnr: SAP-System-NR
	
			
	-meth: <sap|nag|ls|cons>
		sap:	The alarmlevels are used from sap-ccms-methode ta:rz20. The SAP-LEVEL are GREEN, YELLOW, RED, GRAY.
		nag:	The alarmlevel warning or critical are used from nagios. This options are -w (warning) and -c (critical).
		ls:	List the monitoring tree with all objects.
		cons:	You can use other objects without ccms. For more information use sapcontrol -h.
		
	-obj: <MONITORING-OBJECT>
		abap ->
			Operating-System:
				Free Memory 			-> Free Memory OS
				Paging\\Page_In			-> PageIn
				Paging\\Page_Out			-> PageOut
				Swap_Space\\Percentage_Used	-> SWAP Space usage %
		
			SAP:
				Utilisation Granule Entries 	-> DE: Sperrtabelle
				Gateway_CommAdmEnty	-> DE: SAP Gateway Verbindungen
				PerformanceU1\\Utilisation		-> Performance UPD1
				OS_Collector\\State		-> OS Collector State
				CacheHits			-> CacheHits %
				CacheHitsMem			-> CacheHitsMem %
				ResponseTime			-> Dialog Response Time msec
				FrontendResponseTime		-> Frontend Response Time msec
				UsersLoggedIn			-> Number of Users logged in
				LDAP_RFC-01\\Status		-> LDAP Connector 01 State			
				EM Used				-> Extended Memory Usage %
				R3RollUsed			-> Roll area usage %
				Shortdumps			-> ABAP Shortdumps st22. You should use with -obj: sap
				Shortdumps Frequency		-> ABAP Shutdump frequenze
				deadlocks			-> deadlocks
				HostspoolListUsed		-> Used Spool Numbers %
				SyslogFreq			-> Syslogfrequency %
				R3Syslog\\Security		-> Syslog analysis scope: security messages
				R3Syslog\\CCMS			-> Syslog analysis scope: ccms messages
			
			DB:
				DBRequestTime			-> RequestTime
				SqlError			-> SQL-Error
			
			java ->
				HTTPConnectionsCount		-> dispatcher http connections
				Average response time		-> msec. DE: durchschnittliche Antwortzeit
				UsedMemoryRate			-> Memory Usage
				UnsuccessfulLogonAttemptsCount	-> Unsuccessful Logons
				CurrentHttpSessions		-> act. http sessions
			
			trex ->
				Free Memory			-> Free Memory \%
				IndexServer Memory		-> Index Server Memory
				QueueServer Memory		-> Queue Server Memory
				RfcServer Memory		-> RFC Server Memory
				Build				-> TREX Version
				Search Time			-> Performance: Search Time
				Search Count			-> Performance: Request per minute
				RFC Check			-> RFC Connections to backend systems
				Index Status			-> Index State
				Build				-> TREX Version
			
	-function: use this with -meth: cons
		ABAPGetWPTable -> Processtable of sap-system ( dia-proc, btc-proc, upd-proc, spo-proc, up2-proc )
			
	-backend: Type of sap-backend system
		abap: abap-backend-system
		java: java-backend-system
		trex: trex-backend-system
		multi: multiline output for meth: cons
			
	-w: warning-level
	
	-c: Critical Level
			
	-t: plugin timeout, default: 30 sec.
			
	-sudo: 1
		This parameter is optional. You can use it if you start the sapcontrol check off remote host not on icinga-system.
		If you set the parameter to 1 the icinga-system check with ssh and sudo command on remote-site
			
	Help:
		Error:
			GetAlertTree FAIL: NIECONN_REFUSED (Connection refused), NiRawConnect failed in plugin_fopen -> The sap-system-nr is incorrect.
	
			FAIL: HTTP error, HTTP/1.1 401 Unauthorized -> You use a new sap-kernel binarie. In this new version the sapstartsrv service have security features enabled.
			Set the paramter service/protectedwebmethods=NONE in the sap-instance profile. NONE->sapstartsrv-security-features disabled
			For more information have a look at SAP-Note 927637. 
			
	Others:
					
			
			
		Monitoring-Objects:
			If you want other monitoring objects from ccms, please use the command:
			/usr/sap/hostctrl/exe/sapcontrol -nr <SYSNR> -function GetAlertTree
			or
			check_host_ctrl.pl -host <HOSTNAME> -nr <SYSNR> -meth ls
			to investigate which objects are available.
			
	Version: check_host_ctrl.pl -v
			
	Syntax: check_host_ctrl.pl -host <hostname> -sysnr <SAP-SYS-NR> -meth <sap|nag|ls|cons> -<obj|function> <OBJEKT> -backend <abap|java|trex|multi> -w <WARNING-LEVEL> -c <CRITICAL-LEVEL> -t <TIME_IN_SEC>
			
		Examples:
		
			check_host_ctrl.pl -host <host> -sysnr 00 -meth ls
				List all montoring-objects from the backend
			
			check_host_ctrl.pl -host <host> -sysnr 00 -meth ls -sudo 1
				List all monitoring-objects from backend, but use the sapcontrol binarie from backend-system with ssh connection and sudo command
			
			check_host_ctrl.pl -host <host> -sysnr 00 -meth sap -obj CacheHits -backend abap
				Output of sap-abap cachehits
			
			check_host_ctrl.pl -host <host> -sysnr 00 -meth nag -obj CacheHits -backend abap -w 60 -c 80
				Output of sap-abap cachehits with alarmlevel from nagios-system
			
			check_host_ctrl.pl -host <host> -sysnr 00 -meth cons -function ABAPGetWPTable -backend multi -w 80 -c 90
				Output of sap-processes, DIA-Usage, BTC-Usage, SPO-Usage, UPD-Usage, UP2-Usage
			






Roadmap:
	# ABAPReadSyslog -> Syslog (TA:sm21)
	# J2EEGetProcessList -> J2EE-Processes running
	# an so on... 
