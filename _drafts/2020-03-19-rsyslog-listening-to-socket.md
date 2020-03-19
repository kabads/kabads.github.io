---
layout: post
date: 2020-03-19
title: Rsyslog configured to create /run/sytemd/journal/sysolg
---

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
