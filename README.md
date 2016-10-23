##  Install Satellite 6.2

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
   --foreman-initial-organization "RHV-LAB" \
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
```
 # mkdir /root/.hammer
 # vi /root/.hammer/cli_config.yml
```
~~~
foreman:
  :host: 'https://localhost'
  :username: 'admin'
  :password: 'password'
~~~
```
# hammer organization list
---|---------|---------|------------
ID | NAME    | LABEL   | DESCRIPTION
---|---------|---------|------------
1  | RHV-LAB | RHV-LAB |            
---|---------|---------|------------
# 
```
 * To save a bit of times and commands, we can make use of new Satellite 6.2 `hammer defaults` feature:
~~~
# hammer defaults list
----------|------
PARAMETER | VALUE
----------|------
# 
# hammer organization list
---|---------|---------|------------
ID | NAME    | LABEL   | DESCRIPTION
---|---------|---------|------------
1  | RHV-LAB | RHV-LAB |            
---|---------|---------|------------
# 
# 
# hammer location list
---|-----
ID | NAME
---|-----
2  | LAB 
---|-----
# 
# 
# hammer defaults add --param-name organization_id --param-value 1
Added organization_id default-option with value 1.
# 
# hammer defaults add --param-name location_id --param-value 1
Added location_id default-option with value 1.
# 
# hammer defaults list
----------------|------
PARAMETER       | VALUE
----------------|------
organization_id | 1    
location_id     | 1    
----------------|------
# 
~~~

 * **NOTE: from now on everything that is dine relates to this defaults!**

### Generate and upload Manifest 

 * go to https://access.redhat.com/management/distributors?type=satellite
   1. Register a Satellite
     * fill in `Name`
     * Select Version: `6.2`
     * press `Register`
       * **NOTE:**
          * at this point we just created the satellite manifest in the Red Hat portal, actually this is not more than wrapper that needs to be filled (with subscriptions)
   2. Add subscriptions  
     * klick `Attach Subscriptions`
     * Select subscriptions _and_ appripriate ammount to be added to the manifest
       * **NOTE:**
          * If satellite is registered upsteam, no Satellite subscription needs to be added 
   3. Finally press `Download manifest` to download the manifest to your local computer, then copy it over to you Satellite 6 server

```
 $ scp manifest_12ed53a2-2ce3-2d54-6274-fe49fegaadeb.zip root@sat6:
```

 * back on the Satellite Server upload the manifest:
```
  # hammer subscription upload --file /root/manifest_12ed53a2-2ce3-2d54-6274-fe49fegaadeb.zip 

  # hammer subscription list
```
~~~
---|----------------------------------|-----------------------------------------|----------|---------|--------------|----------|----------|------------------------------|----------|---------
ID | UUID                             | NAME                                    | CONTRACT | ACCOUNT | SUPPORT      | QUANTITY | CONSUMED | END DATE                     | QUANTITY | ATTACHED
---|----------------------------------|-----------------------------------------|----------|---------|--------------|----------|----------|------------------------------|----------|---------
1  | 3d889cd345a1aa122a12f168ccc50273 | Employee SKU                            | 12345678 | 000000  | Self-Support | 99       | 0        | 1033-08-01T05:59:59.000+0000 | 99       | 0       
3  | 3d889cd345a1aa122a12f168cd9c03f5 | RHEV Employee Subscription              | 12345678 | 000000  | Self-Support | 99       | 0        | 1033-08-01T05:59:59.000+0000 | 99       | 0       
4  | 3d889cd345a1aa122a12f168cde00409 | Red Hat Satellite Employee Subscription | 12345678 | 000000  | Self-Support | 99       | 0        | 1033-08-01T05:59:59.000+0000 | 99       | 0       
---|----------------------------------|-----------------------------------------|----------|---------|--------------|----------|----------|------------------------------|----------|---------
# 
~~~
