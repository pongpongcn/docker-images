Samba服务配置
====================

重置Samba服务文件夹权限
--------------------

Users文件夹

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