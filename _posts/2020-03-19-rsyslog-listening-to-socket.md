---
layout: post
date: 2020-03-19
title: Rsyslog configured to monitor journald /run/systemd/journal/syslog socket
categories:
- sysadmin
---
Rsyslog (aka syslog) can pull in logs from journald, via the socket (using imuxsock module) or a specific module that taps in to a socket that is journald runs (imjournal) with very little config. imjournal specialises in logging that is structured (e.g. logs that follow json structure) and then filtering or querying logs. As a result, imjournal is more expensive.  These modules are already loaded by default inrsyslog.conf file and do not need adding in any other configuration files.

To use a socket (```imuxsock```) module instead of the ```imjournal``` module, turn off persistent logging in journald by removing the directory ```/var/log/journal``` and setting ```Storage=auto``` in ```/etc/systemd/journal.conf```. Once you restart journald, it will not write logs to disk, but instead to a virtual file location in ```/run/systemd/journal```. This runs the risk that we may lose some logs if they are not persisted. Therefore, we need to get rsysolg to moniter this socket (thereby writing them to disk and to papertrail at the same time).

Rsyslog has two ways of pulling in journald logs. One is through a module called ```imjournal``` which is good as structuring log files. However, another more lightweight method is through creating a socket ```/run/systemd/journal/syslog```. This socket is less expensive. 

To create this socket, you need to look at the systemd unit file for rsyslog. 

Two lines are usually commented out (with the character ';'). 
    
    [Unit]
    ...
    ;Requires=syslog.socket
    ...
    [Install]
    WantedBy=multi-user.target
    ;Alias=syslog.service

Remove the ; characters and restart syslog with ```systemctl restart rsyslog``` and the ```/run/systemd/journal/syslog``` file should be there.

If you are coming from a default CentOS install, rsysolg will keep a position of where it is in the journal log. This can cause problems whilst configuring the socket. By deleting the state file, ```/var/log/rsyslog/imjournal.state```, you will prevent these problems. Once you restart rsyslog, this file will be recreated with the new journal position (which is something we don’t need to worry about, but rsyslog may throw errors if you don’t force this ‘refresh’).

Once you have a fresh state, you need to create a virtual file ```/run/systemd/journal/syslog``` - once this is in place, rsyslog is ready to pull logs from journald. To create this socket/file, you need to look at the systemd unit file for rsyslog ```/lib/systemd/system/rsyslog.service```.

Remove the comment ( ; ) characters and restart syslog with systemctl restart rsyslog and the /run/systemd/journal/syslog file should be there.

Then a directive needs to be set for rsyslog to look at that socket, so this line should be included in rsyslog config (preferably /etc/rsyslog.d/##-journald.conf):
    
    # /etc/rsyslog.d/48-journal.conf
    $SystemLogSocketName /run/systemd/journal/syslog

There is also a symlinked file held by systemd that lets rsyslog manage the /run/sytemd/journal/syslog socket. This file can be symlinked with ```ln -s /lib/systemd/system/rsyslog.service /etc/systemd/system/syslog.service```.

    # /etc/systemd/system/syslog.socket
    
    [Unit]
    Description=System Logging Service
    Requires=syslog.socket
    Wants=network.target network-online.target
    After=network.target network-online.target
    Documentation=man:rsyslogd(8)
    Documentation=http://www.rsyslog.com/doc/
    
    [Service]
    Type=notify
    EnvironmentFile=-/etc/sysconfig/rsyslog
    ExecStart=/usr/sbin/rsyslogd -n $SYSLOGD_OPTIONS
    Restart=on-failure
    UMask=0066
    StandardOutput=null
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    Alias=syslog.service

The ```/etc/rsyslog.conf``` file should look like this (the change here is the ```$OmitLocalLogging off``` directive, which is usually on, but must be off for the rsyslog socket to work):

    # /etc/rsyslog.conf
    $PreserveFQDN off
    
    #################
    #### MODULES ####
    #################
    $ModLoad imuxsock # provides support for local system logging - via logger command
    $ModLoad imklog # provides kernel logging support
    $ModLoad immark # provides --MARK-- message capability
    $ModLoad imjournal
    
    
    
    ###########################
    #### GLOBAL DIRECTIVES ####
    ###########################
    
    
    #
    # Use traditional timestamp format.
    # To enable high precision timestamps, comment out the following line.
    #
    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
    
    # Filter duplicated messages
    $RepeatedMsgReduction on
    
    #
    # Set temporary directory to buffer syslog queue
    #
    $WorkDirectory /var/lib/rsyslog
    
    #
    # Set the default permissions for all log files.
    #
    $FileOwner root
    $FileGroup root
    $FileCreateMode 0600
    $DirCreateMode 0755
    $Umask 0022
    
    #
    # Set other directives
    #
    $OmitLocalLogging off
    $IMJournalStateFile imjournal.state

The journald.conf file needs one change (without this change, most logs will be logged twice - once by rsyslog itself and then also by journald sending it to rsyslog:

    # /etc/systemd/journald.conf
    ...
    ForwardToSyslog=no
    ...

After restarting both journald (```systemctl restart systemd-journald```) and rsyslog (```systemctl restart rsyslog```) you will be able to test both logs with ```journalctl -f``` and ```tail -f /var/log/messages``` - then issue the command ```logger -p daemon.warn "This is only a test."```.

This should be written to both logs at the same time.

