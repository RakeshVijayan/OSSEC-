# Install ossec Local on ubuntu 
```
1)sudo apt install -y wget unzip make gcc build-essential inotify-tools
2)export VER="3.1.0"
3)wget https://github.com/ossec/ossec-hids/archive/${VER}.tar.gz
4) tar -xvzf ${VER}.tar.gz
5) cd ossec-hids-${VER}
6) sudo sh install.sh
      a) (en/br/cn/de/el/es/fr/hu/it/jp/nl/pl/ru/sr/tr) [en]: en
      b) Press Enter 
      c) What kind of installation do you want (server, agent, local, hybrid or help)? local
      d)  Give the rest of the answer as you wish and restart ossec with following 
                sudo /var/ossec/bin/ossec-control restart
```
Ossec Install Web UI 

```
1) git clone https://github.com/ossec/ossec-wui.git
2) sudo mv  ossec-wui /srv
3) cd /srv/ossec-wui
4) sudo ./setup.sh
      Set Dashboard admin username/password and web server username
5) sudo nano  /etc/apache2/sites-enabled/ossec-wui.conf
      
        <VirtualHost *:80>
     DocumentRoot /srv/ossec-wui/
     ServerName ossec.example.com
     ServerAlias www.ossec.example.com
     ServerAdmin admin@example.com
 
     <Directory /srv/ossec-wui/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>

     ErrorLog /var/log/apache2/modsec-error.log
     CustomLog /var/log/apache2/modsec-access.log combined
</VirtualHost>

```
Must Read and Applied this to get alert 
https://blog.wpsec.com/using-ossec-to-monitor-directory-and-file-changes-in-wordpress/

Setup alerts if any file changes or new files added in given into the  directory which we are going to monitor.
 Also ensure we can send mail from the linux box by install sendmail or smtp 

```
nano  /var/ossec/etc/ossec.conf

Add line as follows

  <global>
    <email_notification>yes</email_notification>
    <email_to>rakesh.v@stratagile.com</email_to>
    <smtp_server>alt1.aspmx.l.google.com.</smtp_server>
    <email_from>ossecm@rakesh</email_from>
  </global>

```

Configure the directory we need to mointor with ossec. Here I am going to check emample.como web directory in /var/www

```
root@rakesh:/home/rakesh# cd /var/www/
example.como/ html/         
root@rakesh:/home/rakesh# cd /var/www/
```
Edit the main conf file of ossec 


> nano  /var/ossec/etc/ossec.conf

On syscheck section update as follow to monitor the direcotries we needed 

```
<syscheck>
    <!-- Frequency that syscheck is executed - default to every 22 hours -->
    <frequency>180</frequency>
    <alert_new_files>yes</alert_new_files>
    <auto_ignore>no</auto_ignore>
    <!-- Directories to check  (perform all possible verifications) -->
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin,/var/www</directories>
<!--    <directories check_all="yes" realtime="yes" report_changes="yes">/var/www/example.como/www</directories> -->
    <directories report_changes="yes" realtime="yes" restrict=".php|.js|.py|.sh|.html" check_all="yes">/var/www/example.como/www</directories>
    <directories check_all="yes">/bin,/sbin,/boot</directories>
```
> Change freequency to alert to 7200 to 180 that meens 3 min Alert. 

After all changes we will get the update in web gui to evaluate. file chages in 

integrety checking option. See the picture 

[Why aren’t new files creating an alert?¶
https://www.ossec.net/docs/faq/syscheck.html


By default OSSEC does not alert on new files. To enable this functionality, <alert_new_files> must be set to yes inside the <syscheck> section of the manager’s ossec.conf. Also, the rule to alert on new files (rule 554) is set to level 0 by default. The alert level will need to be raised in order to see the alert. Alerting on new files does not work in realtime, a full scan will be necessary to detect them.

Add the following to local_rules.xml:
```
<rule id="554" level="10" overwrite="yes">
  <category>ossec</category>
  <decoded_as>syscheck_new_entry</decoded_as>
  <description>File added to the system.</description>
  <group>syscheck,</group>
</rule>

The <alert_new_files> entry should look something like this:

<syscheck>
  <frequency>7200</frequency>
  <alert_new_files>yes</alert_new_files>
  <directories check_all="yes">/etc,/bin,/sbin</directories>
</syscheck>

```
### To Remove Ossec from the host do the following steps 
```
/var/ossec/bin/ossec-control stop 
service ossec-hids stop 
chkconfig ossec-hids --del 
rm -rf /var/ossec 
rm -f /etc/ossec-init.conf 
rm -f /etc/init.d/*ossec* 
userdel ossecr 
userdel ossec  
groupdel ossec
```
# Adding new host to centralised server will update ...