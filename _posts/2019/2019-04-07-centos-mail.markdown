---
layout:     post
title:      "centos 7 使用 mail 发送邮件"
date:       2019-04-07
author:     "Kingqh"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Linux
---

# centos 7 使用 mail 发送邮件

修改mail.rc配置文件，vim /etc/mail.rc

	set from=robot@126.com
	set smtp=smtps://smtp.126.com:465
	set ssl-verify=ignore
	set nss-config-dir=/root/.certs
	set smtp-auth-user=robot@126.com
	set smtp-auth-password=robot-password
	set smtp-auth=login

创建证书目录

	mkdir /root/.certs

然后，ssl 授权，执行如下命令

	echo -n | openssl s_client -connect smtp.126.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/126.crt
	certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/126.crt
	certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/126.crt
	certutil -L -d .certs
	certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d ~/.certs/ -i ~/.certs/126.crt

验证是否成功

	echo "test" | mail -v -s "hello" xx@qq.com

mail命令：

	mail -s "just a test" 收信人邮箱地址 < 要发送的邮件内容文件 -- -f 发送人邮件地址 -F 发件人姓名

效果是: 信件内容将发送给 收信人邮箱,显示的发送人为 发送人姓名<发送人邮件地址>,显示的内容为 发送的邮件内容...

包含命令执行结果的MAIL发送:

	mail -s "test" fff@aaa.com < /tmp/dd.txt -- -f cc@aaa.com -F cc
