---
layout: post
title: "How to stream with RTMP on NginX, Part Ⅰ"
description: "How to stream with RTMP on NginX"
category: 
tags: [NginX, RTMP]
---
### Preface

 Recently, Streaming Service is contemporary and open but high-level performance so that is rugged to low API designing and costly. On ad-hoc network - only streaming in some like Pigg Channel, the most important things is short-term development to implement it with low cost and high API. 

### Contents

    Preface
    Contents
    Abstract
    Design

        0. Prerequisite
        1. Create RTMP module with C based on Unix programming.
        2. Then rebuild NginX with it.
        3. Configure .conf file
        4. Access the site and record on live

    Conclusion
    References

### Abstract

 NginX is a well-known web server asynchronous each request, RTMP is protocol developed by Adobe (Macromedia) for streaming on flash player the specification has been published in 2009. It is a goal to implement RTMP service running on NginX as media server. There are two parts record a stream and publish it on live. In this part, I want to clear how to record a live stream to a file using RTMP, next to how to publishing about those. See what I try out below.

 0. Prerequisite

 1. Create RTMP module with C based on Unix programming.

 2. Then rebuild NginX with it.

 3. Configure .conf file

 4. Access the site and record on live

### Design

## 0. Prerequisite

 NginX 1.3.4
 nginx-rtmp-module 3.0.1 (use RTMP port 1935)
 gcc 4.1.2
 Make 3.81
 you can download library whenever get errors or warnings.

 Here are environments,

 CentOS release 5.4

 Linux 2.6.18-164.el5 #1 SMP Thu Sep 3 03:28:30 EDT 2009 x86_64 x86_64 x86_64 GNU/Linux

## 1. Create RTMP module with C based on Unix programming.

 There are a lot of open source group for that, in this case I tried out nginx-rtmp-module※1 have participated as watcher in github. 

 repository : git clone https://github.com/arut/nginx-rtmp-module.git nginx-rtmp-module
 or
 download as zip : curl -L -O -k https://github.com/arut/nginx-rtmp-module/tarball/master

 Prior to make and build let me skim those sources in nginx-rtmp-module.
 As you can see, it is simple category - conduct with NginX, control packet, send and receive each event and command (record, so on)
 Most heady files are blow.

 - control RTMP from request on NginX referenced their modules
 ngx_rtmp.h

 - metadata 
 ngx_rtmp_core_moduel.h

 You should know about NginX module how to add modules and customize it but not no need to fall down that hole.

## 2. Then rebuild NginX with it.

 Here are files downloaded - nginx-rtmp-module-v0.3.0-1-g8293c42.tar.gz and nginx-1.3.4.tar.gz

 $ cd ﻿﻿INSTALL_PATH  $ tar zxvf nginx-rtmp-module-v0.3.0-1-g8293c42.tar.gz  
 $ cd nginx-1.3.4/  
 $ ./configure --add-module=/usr/local/nginx-rtmp-module --with-debug --without-http_rewrite_module 
 $ make 
 $ make install

 run it, nginx/sbin/nginx

 see nothing to error nginx/logs/error.log

## 3. Configure .conf file

 vim where is .conf file in NginX - nginx/conf/nginx.conf

 (I do not want to explain the others, fastcgi.conf and so on, get a chance to know about NginX web server and join to the study group if you want , attach our group link)


 stop and start the web server, you may stop master and worker processes both of them.

 Note, here is FFmpeg is open source framework, able to decode, encode, transcode, stream... to url on RTMP, for instance, /usr/local/ffmpeg-0.11.1/ffmpeg -loglevel verbose -re -i /tmp/FILE_NAME  -f flv rtmp://YOUR_SERVER_IP/my_app/YOUR_FILE_NAME

## 4. Access the site and record on live

 make javascript in a html to record stream on camera

 each .js and .swf are included test/www directory in nginx-rtmp-module archived.

 record.html
   
 

 load the html on browser and check the path whether .flv stream file exists where you want to save it have written down on .conf.
 
 
 
 $ ll /tmp/*flv 
 
 rw-rr- 1
 
 nobody nobody 992722 8月 29 11:54 /tmp/mystream.flv

### Conclusion

 As you have read or done, it is simple you aim't install any produce licensed like Flash Media Server... 
 Of course someone wonder what if have a number of performance cpu, memory and so on such a situation -  on a big site and have many many users.. also need to test performance continuously but you do remember, as I mention agenda above, the most importance is the purpose what situation you want to use, service and publish to users. At least I do convince to satisfy that on ad-hoc service.
 In this part, let me show you some utilities..

 ffmpeg - http://ffmpeg.org/index.html

 ffplay - http://ffmpeg.org/documentation.html

 rtmpdump - http://all-streaming-media.com/record-video-stream/rtmpdump-freeware-console-RTMP-downloading-application.htm

### References

 ※1:github open source group, https://github.com/arut/nginx-rtmp-module

 RTMP - http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol

 NginX - www.nginx.org, http://en.wikipedia.org/wiki/Nginx

 Network - The Linux Programming Interface : A Unix and Linux System Programming Handbook by Michael Kerrisk 
