## Convert2rhel process for CentOS 7.7 to RHEL 7.7 ##

### Prereqs ###
1.Take a backup /snapshot of the VM.  
2.if using RHV , go to RHV dashboard for VM and add click on ... next to migrate and click on 'Change CD' and add the RHEL ISO.  
3.mount the  /dev/sr0 /dvd  

### Convert2rhel  ###
1.yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm  
2.yum install convert2rhel  

### Add latest RHEL Repo ###
cp /dvd/media.repo  /etc/yum.repos.d/rhel.repo  
update /etc/yum.repos.d/rhel.repo as below  
```
[yum.repos.d]# cat rhel.repo
[rhel]
name=Red Hat Enterprise Linux 7.7
mediaid=1563892373.442998
metadata_expire=-1
gpgcheck=0
cost=500
baseurl=file:///dvd
```
### create and copy package , convert2rhel need this ###

```mkdir -p /usr/share/convert2rhel/redhat-release/Server/  
rpm --import /dvd/RPM-GPG-KEY-redhat-release  
cp /dvd/Packages/redhat-release-server-7.7-10.el7.x86_64.rpm  
```
#Run conversion
``` convert2rhel --disable-submgr --disablerepo "*" --enablerepo rhel  --restart -y ```

``` log ---Failed First time -----
Error as below
Running transaction check
ERROR with transaction check vs depsolve:
kmod-kvdo conflicts with kmod-kvdo-6.1.2.41-5.el7.x86_64
kmod-kvdo conflicts with (installed) kmod-kvdo-6.1.2.41-5.el7.x86_64
** Found 4 pre-existing rpmdb problem(s), 'yum check' output follows:
GeoIP-1.5.0-14.el7.x86_64 has missing requires of geoipupdate
abrt-cli-2.1.11-55.el7.centos.x86_64 has missing requires of libreport-centos >= ('0', '2.1.11', '43')
abrt-cli-2.1.11-55.el7.centos.x86_64 has missing requires of libreport-plugin-mantisbt >= ('0', '2.1.11', '43')
plymouth-0.8.9-0.32.20140113.el7.centos.x86_64 has missing requires of system-logos
Your transaction was saved, rerun it with:
 yum load-transaction /tmp/yum_save_tx.2020-06-29.22-59.AsjJ7H.yumtx
Received return code: 1
```


##### DEBUG one by one #####
GeoIP-1.5.0-14.el7.x86_64 has missing requires of geoipupdate  
As per https://access.redhat.com/discussions/4385901 it an issue with  net-snmp-libs and net-snmp-utils  
``` yum -y erase net-snmp-utils; yum -y erase net-snmp-libs ```

abrt-cli-2.1.11-55.el7.centos.x86_64 has missing requires of libreport-centos >= ('0', '2.1.11', '43')
abrt-cli-2.1.11-55.el7.centos.x86_64 has missing requires of libreport-plugin-mantisbt >= ('0', '2.1.11', '43')
Followed https://access.redhat.com/discussions/4385901

``` yum remove libreport centos-indexhtml ```





**kmod-kvdo issue  
details in https://access.redhat.com/solutions/3829661  

added kmod-kvdo  to blacklist in yum.conf  
Following line was added in yum.conf  
```exclude=kmod-kvdo```

**Plymouth package is CentOS package  
removing it  
``` yum remove -y centos-logos epel-release ```

Re-run the installer and it will reboot the OS if no error .  


#https://www.redhat.com/en/blog/converting-centos-rhel-convert2rhel-and-satellite
#https://www.redhat.com/en/blog/convert2rhel-how-update-rhel-systems-place-subscribe-rhel





