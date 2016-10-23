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
 # firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP \
   && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd  --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP \
   && firewall-cmd  --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP
```

### IPA
```
 # yum install bind-utils ipa-client ipa-admintools
 # ipa-client-install --mkhomedir --enable-dns-updates 
 # host sat6
 # host 192.168.10.50
```

### For RHV
```
 # subscription-manager repos --enable rhel-7-server-rh-common-rpms
 # yum install rhevm-guest-agent-common -y
 # systemctl start ovirt-guest-agent.service
 # systemctl enable ovirt-guest-agent.service

```

### Initial Installation 
```
 # yum install -y satellite
 # (ipa config to add)
 # satellite-installer --scenario satellite \
   --foreman-admin-username admin  \
   --foreman-admin-password password \
   --foreman-initial-organization "MY-LAB" \
   --foreman-initial-location "LAB" \
   --foreman-proxy-dhcp true \
   --foreman-proxy-dhcp-interface eth0 \
   --foreman-proxy-dhcp-range "192.168.100.1 192.168.100.254" \
   --foreman-proxy-dhcp-gateway 192.168.0.1 \
   --foreman-proxy-dhcp-nameservers 192.168.10.10 \
   --foreman-proxy-tftp true \
   --foreman-proxy-tftp-servername 192.168.10.50 
 # (ipa config to add)
```

### Configuring Satellite
#### Hammer w/o passord
 # mkdir /root/.hammer

 # vi /root/.hammer/cli_config.yml
~~~
foreman:
  :host: 'https://localhost'
  :username: 'admin'
  :password: 'password'
~~~

### Generate and upload Manifest 


