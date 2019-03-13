# 脚本修改和优化之处

1）去除Registrar字段输出，防止某些域名此字段长度过长，影响对齐；
2）增加Status字段，如果剩余日期小于设置的域名过期告警时间，则显示"Expiring"，否则显示“Valid”；
3）增加对cn域名的支持（在EXPIRE_STRINGS增加"Expiration Time:"）；
4）添加邮件发送的SMTP设置（增加用户名和密码设置项和EHLO命令）；
5）经过使用发现，发现whois命令有概率会卡住，因此增加whois命令的超时时间设置（添加timeout 15）。

# 执行相关

脚本输出结果示例如下：
Domain Name                Expiration Date                 Days Left   Threshold   Status    
abc.com                    2020-02-06 00:37:35             329         30          Valid
xyz.com                    2019-07-21 14:27:36             130         150         Expiring
sample.com                 2019-09-07 21:48:29             178         30          Valid

计划任务配置如下（结合cronmon）：
40 10 * * * cd /path/to/dns-domain-expiration-checker && python dns-domain-expiration-checker.py --interactive --domainfile myDomainList --sleeptime 3 --email --smtpserver smtp.mail.com --smtpto "receiver@apowo.com" --smtpfrom "sender@apowo.com" > dns-domain-expiration-checker.py.cron.log 2>&1 && curl -kfsS --retry 3 --connect-timeout 10 --ipv4 https://cronmon.abc.com/api/monlink/dd250236-1efb-11e9-b39f-001jk272e686 >> dns-domain-expiration-checker.py.cron.log

# Checking DNS Domain Expiration

If you are here you may have had a domain expire and dealt with the annoyances that go with reclaiming it. It's no fun is it? To prevent yourself from dealing with this again you can install and run dns-domain-expiration-checker.py to monitor your domains. The script is easy to install and will send you an e-mail if your domain is set to expire in the near future. You can also use the script to view the registrars and expiration dates for your domains. Now to some examples.

To interactively view the expiration dates and registrars for a list of domains run the script with the "--interactive
" option:
<pre>
$ dns-domain-expiration-checker.py --interactive --domainfile domains
Domain Name                Registrar                       Expiration Date       Days Left
prefetch.net               DNC HOLDINGS, INC.              2020-06-23 00:00:00   1064
spotch.com                 GANDI SAS                       2017-12-03 00:00:00   131 
google.com                 MARKMONITOR INC.                2020-09-14 00:00:00   1147
</pre>

To generate an e-mail when a domain is about to expire you can pass a domain and threshold to the script:

<pre>
$ dns-domain-expiration-checker.py --domainname prefetch.net --email --expiredays 90
</pre>

This will generate an e-mail if the domain prefetch.net is set to expire in the next 90-days. You can also add several domains and expiration intervals to a file and pass that as an argument:

<pre>
$ cat domains
spotch.com 60
yahoo.com 60
prefetch.net 2000
google.com 80

$ dns-domain-expiration-checker.py --domainfile domains --email --smtpserver smtp.mydomain --smtpto "biff" --smtpfrom "Root"
</pre>

# Installation

The dns-domain-expiration-checker.py script relies on the dateutil PiPY pacakge to normalize dates. Here are the steps to get this script working:

1. Create a virtualenv to run dns-domain-expiration-checker in:
<pre>
$ mkproject dns-domain-expiration-checker
</pre>
2. Surf over to that new environment:
<pre>
$ workon dns-domain-expiration-checker
</pre>
3. Pull down dateutils with pip:
<pre>
$ pip install python-dateutil
</pre>
4. Clone this repo:
<pre>
$ git clone https://github.com/Matty9191/dns-domain-expiration-checker.git
</pre>
5. Run the script against the domains you want to check (this assume you are in the root of your virtualenv):
<pre>
$ dns-domain-expiration-checker/ns-domain-expiration-checker.py ....
</pre>
6. Automate the domain checking process.




