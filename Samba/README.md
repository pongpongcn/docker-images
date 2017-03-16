#Setup file permissions
* users
#set owner
chown -R root:users users

chown -R pocketpc users/pocketpc
chown -R michelle users/michelle

#set permission
#reset all
chmod -R a-rwxst users

#set 
chmod u=rwx,g=rxs users

find users -type d -exec chmod u=rwx,g=s {} \;
find users -type f -exec chmod u=rw {} \;
