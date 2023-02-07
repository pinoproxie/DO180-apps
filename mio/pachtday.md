# PATCHDAY Middleware TODO



**Beispiel STUFE SNDX**

## WLS PSU Version (Kann im Vorfeld gemacht werden)



#### Verteilung

###### Vorbereitung 12er Release

```bash
#rausfinden welche instanzen mit welcher wls version laufen
WLSSTATUS | grep -i sndx | grep '12.2' | awk '{ print $1}' # Relevante Instanzen mit WLS 12.2
```

###### Definitionen

```bash
DOMAINS12="MTXX01"
WLSDEPLOYVERSION12="wlserver12.2.1.4.230117.tar"   # Neue Version wird installiert
WLSOLDVERSION12="wlserver12.2.1.4.220719"          # Diese läuft aktuel, wird inaktiviert (env ausgetragen) 
WLSVERSION2REMOVE12="wlserver12.2.1.4.220418"      # Diese wird weggelöscht
```



###### Vorbereitung 14er Release

```bash
#rausfinden welche instanzen mit welcher wls version laufen
WLSSTATUS | grep -i sndx | grep '14.1' | awk '{ print $1}'  # Relevante Instanzen mit WLS 14.1
```

###### Definitionen


 ```bash
 
DOMAINS14="ASEX01 ASEX02 ASEX40 ASEX80 ASEX90 ASEX91 ASEX93 NESX01"    #Relevante Instanzen
WLSDEPLOYVERSION14="wlserver14.1.1.0.230117.tar"
WLSOLDVERSION14="wlserver14.1.1.0.220727" # mittels ps -ef auf Targetmaschine ausgefiltert
WLSVERSION2REMOVE14="wlserver14.1.1.0.220418"
 ```



###### Verteilung 12 er

```bash
for DOMAIN in $DOMAINS12
do
gh $DOMAIN
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
cd /nfsmnt/shared/ifsmnt/midsrc/linux/wls/product
DODHOST "cat ${WLSDEPLOYVERSION12} | ssh -o BatchMode=yes weblogic@\$DHOST \"cd /app/lib/wls/product && tar -xvf - \" "
done
```

##### 

###### Verteilung 14er

```bash
for DOMAIN in $DOMAINS14
do
gh $DOMAIN
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
cd /nfsmnt/shared/ifsmnt/midsrc/linux/wls/product
DODHOST "cat $WLSDEPLOYVERSION14 | ssh -o BatchMode=yes weblogic@\$DHOST \"cd /app/lib/wls/product && tar -xvf - \" "
done

## don't forget the admin server  # we did already a few lines above
#cd /nfsmnt/shared/ifsmnt/midsrc/linux/wls/product
#DHOSTS=h50l180
#DODHOST "cat $WLSDEPLOYVERSION12 | ssh -o BatchMode=yes weblogic@\$DHOST \"cd /app/lib/wls/product && tar -xvf - \" "
#DODHOST "cat $WLSDEPLOYVERSION14 | ssh -o BatchMode=yes weblogic@\$DHOST \"cd /app/lib/wls/product && tar -xvf - \" "
```

##### 



#### Entfernen alte Releases

###### 12 er Version

```bash
WLSVERSION2REMOVE="$WLSVERSION2REMOVE12"
DOMAINS2CLEANUPWLS="$DOMAINS12"

for DOMAIN in $DOMAINS2CLEANUPWLS
do
gh $DOMAIN
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
DSSH "ps -ef | grep ${WLSVERSION2REMOVE} | grep -v grep"
#Da wo nichts zurückkomt DSSHRCE da kann die Version entfernt werden
#->
DHOSTS=$DSSHRCE
DSSH -u weblogic "rm -rf /app/lib/wls/product/${WLSVERSION2REMOVE}*"
done

```

###### 14 er Version

```bash
WLSVERSION2REMOVE="$WLSVERSION2REMOVE14"
DOMAINS2CLEANUPWLS="$DOMAINS14"

for DOMAIN in $DOMAINS2CLEANUPWLS
do
gh $DOMAIN
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
DSSH "ps -ef | grep ${WLSVERSION2REMOVE} | grep -v grep"
#Da wo nichts zurückkomt DSSHRCE da kann die Version entfernt werden
#->
DHOSTS=$DSSHRCE
DSSH -u weblogic "rm -rf /app/lib/wls/product/${WLSVERSION2REMOVE}*"
done

```



#### Alle betroffenen ENV's Umstellen



```bash
DOMAINS2EDIT="$DOMAINS12"
WLSOLDVERSION="$WLSOLDVERSION12"
WLSDEPLOYVERSION="$WLSDEPLOYVERSION12"
#wichtig nur die nummer ersetzten nicht wlserver einsetzten
cd /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc
for DOMAIN in $(echo ${DOMAINS2EDIT} | tr 'A-Z' 'a-z')
do
# do it by vim
#vim ${DOMAIN}.env
echo ${DOMAIN}.env
grep $WLSOLDVERSION /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
# do it by sed
WLSOLDVERSIONNUMBER=$(echo $WLSOLDVERSION | sed 's/wlserver//')
WLSDEPLOYVERSIONNUMBER=$(echo $WLSDEPLOYVERSION | sed 's/wlserver//')
sed -i "s/$WLSOLDVERSIONNUMBER/$WLSDEPLOYVERSIONNUMBER/" /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
grep $WLSDEPLOYVERSION /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
done
```

```bash
DOMAINS2EDIT="$DOMAINS14"
WLSOLDVERSION="$WLSOLDVERSION14"
WLSDEPLOYVERSION="$WLSDEPLOYVERSION14"
#wichtig nur die nummer ersetzten nicht wlserver einsetzten
cd /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc
for DOMAIN in $(echo ${DOMAINS2EDIT} | tr 'A-Z' 'a-z')
do
# it by vim
#vim ${DOMAIN}.env
echo ${DOMAIN}.env
grep $WLSOLDVERSION /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
# do it by sed
WLSOLDVERSIONNUMBER=$(echo $WLSOLDVERSION | sed 's/wlserver//')
WLSDEPLOYVERSIONNUMBER=$(echo $WLSDEPLOYVERSION | sed 's/wlserver//')
sed -i "s/$WLSOLDVERSIONNUMBER/$WLSDEPLOYVERSIONNUMBER/" /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
grep $WLSDEPLOYVERSION /nfsmnt/shared/ifsmnt/midsrc/linux/app/lib/etc/${DOMAIN}.env
done
```



## Tag X

### Zuerst Middleware Stufe X stoppen

```bash
midinit -f stop -s test -d now -w                -v   #sofort
midinit -f stop -s test -d 03.02.2023 -t 11:25   -v   #11:25
```

-> Task schieben auf Board https://jira.suvanet.ch/secure/RapidBoard.jspa?rapidView=1312

warten bis Unix Basis fertig ist

### env von kompletter Stufe rauskopieren

```bash
copyoutdomainenv4stufenletter x
```



### JAVAVERSION

Zuerst Check welche Domain hat welche Java Version http://h50l140.suvanet.ch/wls/wlslinux.html?stufe=sndx

-> Alle ase Versionen und NESX01 haben 11.0.6 

```bash
 DOMAINSJAVA11="ASEX01 ASEX02 ASEX40 ASEX80 ASEX90 ASEX91 ASEX93 NESX01"
```

-> MTXX01 NESX01 haben 1.8

```bash
DOMAINSJAVA8="MTXX01"
```

#### Check welche Version neu  verteilt werden

```
hpc@h50lbb0:/nfsmnt/shared/ifsmnt/midsrc/linux/java/java_ind> ls -l VERSIONS
..
lrwxrwxrwx. 1 hpc unixadm    9 Jan 20 11:07 current -> 1.8.0_361
lrwxrwxrwx. 1 hpc unixadm    7 Jan 20 11:08 current11 -> 11.0.18
```

#### Verteilung Java11

```bash
cd /nfsmnt/shared/ifsmnt/midsrc/linux/java/java_ind
for DOMAIN in $(echo $DOMAINSJAVA11)
do
gh $DOMAIN
DODHOST "./java_ind11_setup  -f -v current11 -z \$DHOST "
done
# warten
dhc
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
DODHOST "./java_ind11_setup  -f -v current11 -z \$DHOST "
```



##### Verteilung java8

```bash
cd /nfsmnt/shared/ifsmnt/midsrc/linux/java/java_ind
for DOMAIN in $(echo $DOMAINSJAVA8)
do
gh $DOMAIN
DODHOST "./java_ind_setup -f -v current -z \$DHOST "
done
# warten
dhc
STUFENKUERZEL=$(echo $DOMAIN | cut -c4 | tr 'A-Z' 'a-z')
dha${STUFENKUERZEL}
DODHOST "./java_ind_setup -f -v current -z \$DHOST "
```



## Anpassung shared key

asex40 copy Files to Adminserver

./target_domain_home/ase_jee_auth.conf
./target_ase_home/conf/bc_auth.conf
./target_ase_home/conf/utc_auth.conf

copy->modify Files to Domainhome (deploy_ase -f swonly -r 3.12.04.00-00-M5 -z asex40 )



### Middleware start 

```bash
midinit -f start -s test -d now -w               -v   #sofort
midinit -f stop -s test -d 03.02.2023 -t 11:25   -v   #11:25
```

-> Task schieben auf Board https://jira.suvanet.ch/secure/RapidBoard.jspa?rapidView=1312





