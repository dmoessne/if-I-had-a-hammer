##  Install Satellite 

### install packages
```
 # subscription-manager register
 # subscription-manager list --available |less
 # subscription-manager attach --pool <pool-id>
 # subscription-manager repos --disable \*
 # subscription-manager repos --enable=rhel-7-server-rpms --enable=rhel-server-rhscl-7-rpms --enable=rhel-7-server-satellite-6.2-rpms
 # yum update -y && reboot
```

### Firewall
``` 
 # firewall-cmd --add-service=RH-Satellite-6
 # firewall-cmd --permanent --add-service=RH-Satellite-6
 # firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP
```
### IPA
```
**
 # yum install bind-utils ipa-client ipa-admintools
 # 
 
```
