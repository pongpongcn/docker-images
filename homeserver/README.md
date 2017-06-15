Samba服务配置
====================

重置Samba服务文件夹权限
--------------------

### Users文件夹

设置基本所有权，属于users组。
`chown -R root:users Users`

清空权限，为后续权限设置确立基准。
`chmod -R a-rwxst Users`

设置Users文件夹权限，允许users组读取。
`chmod u=rwx,g=rxs Users`

设置用户私有文件夹权限，以Users/pocketpc文件夹为例。
```
chown -R pocketpc Users/pocketpc
find Users/pocketpc -type d -exec chmod u=rwx,g=s {} \;
find Users/pocketpc -type f -exec chmod u=rw {} \;
```

设置组共享文件夹权限。
```
chmod u=rwx,g=rwxs,o=t Users/shared
find Users/shared/* -type d -exec chmod u=rwx,g=rwxs,o=t {} \;
find Users/shared/* -type f -exec chmod u=rw,g=r {} \;
```

### Public文件夹

设置基本所有权，仅属于root。
`chown -R root:root Public`

清空权限，为后续权限设置确立基准。
`chmod -R a-rwxst Public`

设置主要权限，开放读权限，非管理员不能写。
```
find Public -type d -exec chmod u=rwx,g=rxs,o=rx {} \;
find Public -type f -exec chmod u=rw,g=r,o=r {} \;
```

设置Public/Uploads文件夹组和权限。
```
chgrp -R users Public/Uploads
find Public/Uploads -type d -exec chmod u=rwx,g=rwxs,o=rxt {} \;
find Public/Uploads -type f -exec chmod u=rw,g=r,o=r {} \;
```

-p 139:139 -p 445:445 -p 137:137/udp -p 138:138/udp

-v /DataVolume/testdata/Public/Downloads/transmission:/var/lib/transmission-daemon \


/tmp/DataVolume/testdata/samba/users
/var/lib/samba/usershares/users

[Unit]
Description=Samba container
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker run -h Samba -p 139:139 -p 445:445 -p 137:137/udp -p 138:138/udp \
        -v /DataVolume/testdata/samba/smbpasswd:/smbpasswd \
        -v /DataVolume/testdata/Public:/share/Public \
        -v /DataVolume/testdata/Users:/share/Users \
        --name samba dperson/samba -n \
        -i "/smbpasswd" \
        -s "Public;/share/Public;yes;yes;yes;all;pocketpc;pocketpc" \
        -s "Uploads;/share/Public/Uploads;yes;no;yes" \
        -s "Pocketpc;/share/Users/pocketpc;yes;no;no;pocketpc" \
        -s "Michelle;/share/Users/michelle;yes;no;no;michelle"
ExecStop=/usr/bin/docker stop -t 2 samba
ExecStopPost=/usr/bin/docker rm -f samba

[Install]
WantedBy=default.target

		
docker build -t pongpongcn/samba .

/usr/bin/docker run -d --rm --net=host \
        -v /DataVolume/appdata/samba/smb.conf:/etc/samba/smb.conf:ro \
        -v /DataVolume/appdata/samba/smbpasswd:/etc/samba/smbpasswd:ro \
        -v /DataVolume/userdata/users:/srv/samba/users \
        -v /DataVolume/userdata/public:/srv/samba/public \
        --name samba pongpongcn/samba \
        -i "/etc/samba/smbpasswd"

docker logs -f samba





设置策略路由
https://www.thomas-krenn.com/en/wiki/Two_Default_Gateways_on_One_System

/etc/iproute2/rt_tables
101     docker

/etc/sysconfig/network-scripts/route-ens224
/etc/sysconfig/network-scripts/rule-ens224

sudo ip route add default via 10.0.0.1 dev ens224 table untrusted
sudo ip route add 172.18.0.0/16 dev docker-dmz src 172.18.0.1 table untrusted

sudo ip rule add from 172.18.0.0/16 table untrusted
sudo ip route flush cache

设置桥接
https://wiki.archlinux.org/index.php/Network_bridge_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E9.80.9A.E8.BF.87_iproute2

设置macvlan
https://docs.docker.com/engine/userguide/networking/get-started-macvlan
docker network create -d macvlan \
    --subnet=10.43.12.0/24 \
    --ip-range=10.43.12.128/25 \
    --gateway=10.43.12.1  \
    -o parent=ens256 untrusted

设置Container固定IP
http://stackoverflow.com/questions/39493490/provide-static-ip-to-docker-containers-via-docker-compose


docker network create \
-o "com.docker.network.bridge.name"="untrusted" \
untrusted


trusted
samba
rygel

dmz
mysql
owncloud
httpd

untrusted
transmission


   usershare allow guests = yes

   security = user
   create mask = 0664
   force create mode = 0664
   directory mask = 0775
   force directory mode = 0775
   force user = smbuser
   force group = users
   load printers = no
   printing = bsd
   printcap name = /dev/null
   disable spoolss = yes
   socket options = TCP_NODELAY
   
打印机配置
====================
```
sudo yum install cups hplip foomatic
hp-plugin -i
#Make sure cupsd running.
sudo hp-setup -i
```

HP Printer upload firmware when connected
--------------------
There should be file /usr/lib/udev/rules.d/56-hpmud.rules  
Create file /etc/udev/rules.d/57-hp-firmware.rules  
```
ACTION!="add", GOTO="hp_firmware_rules_end"

LABEL="hp_firmware_rules"
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", ATTR{idVendor}=="03f0", ATTRS{idProduct}=="4817", RUN+="/usr/bin/hp-firmware -n"

LABEL="hp_firmware_rules_end"
```

CUPS Discover
-------------------
```
sudo yum install avahi
```

Air Printer
-------------------
https://wiki.archlinux.org/index.php/avahi#Airprint_from_Mobile_Devices  
Create new mime files - This is needed for iOS 6 to recognize CUPS print shares!  
```
echo "image/urf urf string(0,UNIRAST<00>)" > /usr/share/cups/mime/airprint.types
echo "image/urf application/pdf 100 pdftoraster" > /usr/share/cups/mime/airprint.convs
```
Avahi along with CUPS also provides the capability to print to just about any printer from airprint compatible mobile devices. In order to enable print capability from your device, simply create an Avahi service file for your printer in /etc/avahi/services/. An example of a generic services file for an HP-Laserjet printer would be similar to the following with the name, rp, ty, adminurl and note fields changed.
```
/etc/avahi/services/airprint.service
```
```
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name>yourPrnterName</name>
  <service>
    <type>_ipp._tcp</type>
    <subtype>_universal._sub._ipp._tcp</subtype>
    <port>631</port>
    <txt-record>txtver=1</txt-record>
    <txt-record>qtotal=1</txt-record>
    <txt-record>rp=printers/yourPrnterName</txt-record>
    <txt-record>ty=yourPrnterName</txt-record>
    <txt-record>adminurl=http://198.168.7.15:631/printers/yourPrnterName</txt-record>
    <txt-record>note=Office Laserjet 4100n</txt-record>
    <txt-record>priority=0</txt-record>
    <txt-record>product=(GPL Ghostscript)</txt-record>
    <txt-record>printer-state=3</txt-record>
    <txt-record>printer-type=0x801046</txt-record>
    <txt-record>Transparent=T</txt-record>
    <txt-record>Binary=T</txt-record>
    <txt-record>Fax=F</txt-record>
    <txt-record>Color=T</txt-record>
    <txt-record>Duplex=T</txt-record>
    <txt-record>Staple=F</txt-record>
    <txt-record>Copies=T</txt-record>
    <txt-record>Collate=F</txt-record>
    <txt-record>Punch=F</txt-record>
    <txt-record>Bind=F</txt-record>
    <txt-record>Sort=F</txt-record>
    <txt-record>Scan=F</txt-record>
    <txt-record>pdl=application/octet-stream,application/pdf,application/postscript,image/jpeg,image/png,image/urf</txt-record>
    <txt-record>URF=W8,SRGB24,CP1,RS600</txt-record>
  </service>
</service-group>
```
Alternatively, https://raw.github.com/tjfontaine/airprint-generate/master/airprint-generate.py can be used to generate Avahi service files. It depends on python2 and python2-pycups. The script can be run using:
```
python2 airprint-generate.py -d /etc/avahi/services
```