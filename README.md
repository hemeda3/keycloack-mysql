# keycloack-mysql

username: ```admin```
password: ```admin```
normal user of SudaCoin realm:
username: ```user1```
password: ```password```
enable SSL

root@emulator1:~/keycloack-mysql# ```docker exec -it keycloak2 bash```

* bash-4.4$ ```cd opt/jboss/keycloak/bin/```
* bash-4.4$ ```./kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin```
* bash-4.4$ ```./kcadm.sh update realms/master -s sslRequired=NONE```
* bash-4.4$ ```./kcadm.sh update realms/SudaCoinRealm -s sslRequired=NONE```
