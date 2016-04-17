CentOS 7 | FreePBX 13 | Asterisk 13 | Webmin | Fail2ban | OpenVPN

##### Outline
* SSHd
  * Add admin user, allow sudoer
  * Add / Create SSH Key
  * Add auth keys
  * Change / disable root password

* Enable EPEL Repo
  * Install epel repo
* ZSH / Oh-My-Zsh
  * Install / Verify ZSH
  * Install Oh-My-Zsh
  * Pull custom my.zsh source

* MISC
  * htop

### SSHd
##### Add Admin Sudoer
`# adduser <username>`

`# passwd <username>`

`# gpasswd -a <username> wheel`

Now `$ sudo -i` should grant root shell when ran from user acct

(logout and back in if already logged into `<username>` acct)

##### Create SSH Key / Copy local ssh pub key
(key auth setup for admin user created above, never root)

`$ ssh-keygen`

`local(mac)$ cat ~/.ssh/id_rsa.pub | pbcopy`

`$ vi ~/.ssh/authorized_keys` (Paste in key, and SHIFT+ZQ)

`$ chmod 600 ~/.ssh/authorized_keys`

Disable root login:

`# vi /etc/ssh/sshd_config`

Replace `#PermitRootLogin yes` -> `PermitRootLogin no`

Reload SSHd:

`# systemctl reload sshd`


### Enable EPEL Repo
`$ yum install -y epel-release`

Disable epel repo as needed:

`$ vi /etc/yum.repos.d/epel.repo`

Replace `enabled=1` -> `enabled=0`




### ZSH / Oh-My-Zsh
`$ yum install -y zsh wget git`

`$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

### Install FreePBX 13
Ref: [Installing FreePBX 13 on CentOS 7](http://wiki.freepbx.org/display/FOP/Installing+FreePBX+13+on+CentOS+7)

`# sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux`

`# sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config`

Restart and verify: `$ sestatus` results in `SELinux status: disabled`

`# yum -y update`

`# yum -y groupinstall core base "Development Tools"`

`# yum -y install lynx mariadb-server mariadb php php-mysql php-mbstring tftp-server \
  httpd ncurses-devel sendmail sendmail-cf sox newt-devel libxml2-devel libtiff-devel \
  audiofile-devel gtk2-devel subversion kernel-devel git php-process crontabs cronie \
  cronie-anacron wget vim php-xml uuid-devel sqlite-devel net-tools gnutls-devel php-pear`

**Install Legacy Pear requirements**

`# pear install Console_Getopt`

##### Firewall Rules

`# firewall-cmd --zone=public --add-port=80/tcp --permanent`

`# firewall-cmd --reload`

##### Enable / Secure MariaDB
`# systemctl enable mariadb.service`

`# systemctl start mariadb`

`# mysql_secure_installation`

##### Enable / Start Apache
`# systemctl enable httpd.service`
`# systemctl start httpd.service`

##### Google Voice Dependencies
```
# cd /usr/src
# wget https://iksemel.googlecode.com/files/iksemel-1.4.tar.gz
# tar xf iksemel-*.tar.gz
# rm -f iksemel-1.4.tar.gz
# cd iksemel-*
# ./configure
# make
# make install
```

##### Add Asterisk User
`# adduser asterisk -M -c "Asterisk User"`

##### Install / Config Asterisk
```
cd /usr/src
wget http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-current.tar.gz
wget http://downloads.asterisk.org/pub/telephony/libpri/libpri-1.4-current.tar.gz
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
wget -O jansson.tar.gz https://github.com/akheron/jansson/archive/v2.7.tar.gz
wget http://www.pjsip.org/release/2.4/pjproject-2.4.tar.bz2
```

##### Compile / Install pjproject
```
cd /usr/src
tar -xjvf pjproject-2.4.tar.bz2
rm -f pjproject-2.4.tar.bz2
cd pjproject-2.4
CFLAGS='-DPJ_HAS_IPV6=1' ./configure --prefix=/usr --enable-shared --disable-sound\
  --disable-resample --disable-video --disable-opencore-amr --libdir=/usr/lib64
make dep
make
make install
```

##### Compile / Install jansson
```
cd /usr/src
tar vxfz jansson.tar.gz
rm -f jansson.tar.gz
cd jansson-*
autoreconf -i
./configure --libdir=/usr/lib64
make
make install
```

##### Compile / Install Asterisk
```
cd /usr/src
tar xvfz asterisk-13-current.tar.gz
rm -f asterisk-13-current.tar.gz
cd asterisk-*
contrib/scripts/install_prereq install
./configure --libdir=/usr/lib64
contrib/scripts/get_mp3_source.sh
make menuselect
```

_You will be prompted at the point to pick which modules to build. Most of them will already be enabled, but if you want to have MP3 support (eg, for Music on Hold), you need to manually turn on_ `format_mp3` _on the first page._

Save & exit, then continue:
```
make
make install
make config
ldconfig
chkconfig asterisk off
```

##### Install Asterisk Soundfiles
```
cd /var/lib/asterisk/sounds
wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-core-sounds-en-wav-current.tar.gz
wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-extra-sounds-en-wav-current.tar.gz
tar xvf asterisk-core-sounds-en-wav-current.tar.gz
rm -f asterisk-core-sounds-en-wav-current.tar.gz
tar xfz asterisk-extra-sounds-en-wav-current.tar.gz
rm -f asterisk-extra-sounds-en-wav-current.tar.gz
# Wideband Audio download
wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-core-sounds-en-g722-current.tar.gz
wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-extra-sounds-en-g722-current.tar.gz
tar xfz asterisk-extra-sounds-en-g722-current.tar.gz
rm -f asterisk-extra-sounds-en-g722-current.tar.gz
tar xfz asterisk-core-sounds-en-g722-current.tar.gz
rm -f asterisk-core-sounds-en-g722-current.tar.gz
```

##### Set Asterisk ownership
```
chown asterisk. /var/run/asterisk
chown -R asterisk. /etc/asterisk
chown -R asterisk. /var/{lib,log,spool}/asterisk
chown -R asterisk. /usr/lib64/asterisk
chown -R asterisk. /var/www/
```

##### Install / Configure FreePBX
**Apache modifications:**
```
sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php.ini
sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/httpd/conf/httpd.conf
sed -i 's/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
systemctl restart httpd.service
```
**Download and Install FreePBX:**
```
cd /usr/src
wget http://mirror.freepbx.org/modules/packages/freepbx/freepbx-13.0-latest.tgz
tar xfz freepbx-13.0-latest.tgz
rm -f freepbx-13.0-latest.tgz
cd freepbx
./start_asterisk start
./install -n
```
##### Post-Install Start-up Script
ref: [Example systemd startup script](http://wiki.freepbx.org/display/FOP/Example+systemd+startup+script+for+FreePBX)
```
[Unit]
Description=FreePBX VoIP Server
After=mariadb.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/fwconsole start
ExecStop=/usr/sbin/fwconsole stop

[Install]
WantedBy=multi-user.target
```

### MISC
##### htop
`$ yum install -y htop`
