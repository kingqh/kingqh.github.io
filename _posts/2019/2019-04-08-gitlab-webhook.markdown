---
layout:     post
title:      "使用webhook触发gitlab发送邮件"
date:       2019-04-08
author:     "Kingqh"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Linux
---

# 使用webhook触发gitlab发送邮件

环境：

- 腾讯云虚拟机一台
- gitlab仓库

使用github上的开源webhook项目，安装webhook到腾讯云主机上。

	go get github.com/adnanh/webhook

添加配置文件hooks.json，修改如下：

	
	[
		{
			"id": "sdn-webhook",
			"execute-command": "/root/hooks/sdn.sh",
			"command-working-directory": "/root/hooks",
			"pass-arguments-to-command": [
				{
					"source": "entire-payload"
				}
			]
		}
	]


创建 /root/hooks/sdn.sh ， 编辑如下：


	#!/bin/bash
	mails={"xxx1@qq.com", "xxx2@qq.com"}
	time=`date +"%Y-%m-%d %H:%m:%S"`
	for i in ${mails[@]};do
	echo $1 | mail -v -s "sdn-git-robot "$time $i
	done

其中，mails是需要发送的邮件列表地址，$1是由webhook传入的gitlab的触发后的payload数据。

最后，在gitlab代码仓库上设置webhook如下：

	http://xxx:9000/hooks/sdn-webhook

xxx为腾讯云主机的公网地址。

目前对于gitlab返回的数据没有处理，需要进一步开发参考如下资料：

[https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/user/project/integrations/webhooks.md](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/user/project/integrations/webhooks.md)