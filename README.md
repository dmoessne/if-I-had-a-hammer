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
 #
 # firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 27017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 27017 -j DROP \
   && firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner apache -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 0 -o lo -p tcp -m tcp --dport 28017 -m owner --uid-owner root -j ACCEPT \
   && firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP \
   && firewall-cmd --permanent --direct --add-rule ipv6 filter OUTPUT 1 -o lo -p tcp -m tcp --dport 28017 -j DROP

```

### IPA
```
 # yum install bind-utils ipa-client ipa-admintools
 # ipa-client-install --mkhomedir --enable-dns-updates 
 # ipa dnsrecord-add 10.25.172.in-addr.arpa 50 --ptr-rec=sat6.lab.example.com
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


### Enable Repositories
#### RHEL7
##### How to find Product and Name to enable Repositories?
* First search for 
```
  # hammer product list --full-results yes |less
```
~~~
----|----------------------------------------------------------------------------------|-------------|--------------|--------------|-----------
ID  | NAME                                                                             | DESCRIPTION | ORGANIZATION | REPOSITORIES | SYNC STATE
----|----------------------------------------------------------------------------------|-------------|--------------|--------------|-----------
[...]
9   | Red Hat Enterprise Linux Server                                                  |             | RHV-LAB      | 0            |           

[...]
~~~

```
# hammer product info --name 'Red Hat Enterprise Linux Server'
```
~~~
ID:           9
Name:         Red Hat Enterprise Linux Server
Label:        Red_Hat_Enterprise_Linux_Server
Description:  
Sync State:   
Sync Plan ID: 
GPG:          
    GPG Key ID: 
    GPG Key:
Organization: RHV-LAB
Readonly:     
Deletable:    
Content:      
[...]
 96)Repo Name:    Red Hat Satellite Tools 6.2 (for RHEL 7 Server) (RPMs)
    URL:          /content/dist/rhel/server/7/7Server/$basearch/sat-tools/6.2/os
    Content Type: yum
[...]
 127)Repo Name:    Red Hat Enterprise Linux 7 Server (Kickstart)
    URL:          /content/dist/rhel/server/7/$releasever/$basearch/kickstart
    Content Type: kickstart
[...]
 178)Repo Name:    Red Hat Enterprise Linux 7 Server - RH Common (RPMs)
    URL:          /content/dist/rhel/server/7/$releasever/$basearch/rh-common/os
    Content Type: yum
[...]
 212)Repo Name:    Red Hat Enterprise Linux 7 Server (RPMs)
    URL:          /content/dist/rhel/server/7/$releasever/$basearch/os
    Content Type: yum
[...]
~~~

~~~
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7Server' --name 'Red Hat Enterprise Linux 7 Server (RPMs)'
  Repository enabled
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7Server' --name 'Red Hat Enterprise Linux 7 Server - RH Common (RPMs)'
  Repository enabled
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7Server' --name 'Red Hat Enterprise Linux 7 Server (Kickstart)'
  Repository enabled
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server'  --basearch='x86_64'  --name 'Red Hat Satellite Tools 6.2 (for RHEL 7 Server) (RPMs)'  
  Repository enabled
  # 
~~~

```
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server' --releasever='7Server'
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server' --releasever='7.1'
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server' --releasever='7.2'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.1' --name 'Red Hat Enterprise Linux 7 Server (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.2' --name 'Red Hat Enterprise Linux 7 Server (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.1' --name 'Red Hat Enterprise Linux 7 Server - RH Common (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.2' --name 'Red Hat Enterprise Linux 7 Server - RH Common (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.1' --name 'Red Hat Enterprise Linux 7 Server (Kickstart)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='7.2' --name 'Red Hat Enterprise Linux 7 Server (Kickstart)'

```
#### Satellite 6

```
  # hammer repository-set enable --product 'Red Hat Satellite' --basearch='x86_64' --name 'Red Hat Satellite 6.2 (for RHEL 7 Server) (RPMs)'
  # hammer repository-set enable --product 'Red Hat Satellite Capsule' --basearch='x86_64' --name 'Red Hat Satellite Capsule 6.2 (for RHEL 7 Server) (RPMs)'
```

#### RHEL6

```
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='6Server' --name 'Red Hat Enterprise Linux 6 Server (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='6Server' --name 'Red Hat Enterprise Linux 6 Server - RH Common (RPMs)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --releasever='6Server' --name 'Red Hat Enterprise Linux 6 Server (Kickstart)'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server'  --basearch='x86_64'  --name 'Red Hat Satellite Tools 6.2 (for RHEL 6 Server) (RPMs)'
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server' --releasever='6Server'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (RPMs)' --releasever='6.8'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - RH Common (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - RH Common (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - RH Common (RPMs)' --releasever='6.8'
  
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (Kickstart)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (Kickstart)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server (Kickstart)' --releasever='6.8'


  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Software Collections for RHEL Server' --basearch='x86_64' --name 'Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server' --releasever='6.8'
  

```

#### RHEV
```
  # hammer product list --full-results yes |grep JBoss
  # hammer product info --name 'JBoss Enterprise Application Platform'
  # hammer product info --name 'JBoss Enterprise Application Platform'|grep "(RHEL 6 Server) (RPMs)"
  # hammer repository-set enable --product 'JBoss Enterprise Application Platform' --basearch='x86_64' --releasever='6Server' --name 'JBoss Enterprise Application Platform 6 (RHEL 6 Server) (RPMs)'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Optional (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Optional (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Optional (RPMs)' --releasever='6.8'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Optional (RPMs)' --releasever='6Server'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Supplementary (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Supplementary (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Supplementary (RPMs)' --releasever='6.8'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 6 Server - Supplementary (RPMs)' --releasever='6Server'

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor (RPMs)' --releasever='6.8'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor (RPMs)' --releasever='6Server'

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Manager 3.5 (RPMs)' --releasever='6.6'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Manager 3.5 (RPMs)' --releasever='6.7'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Manager 3.5 (RPMs)' --releasever='6.8'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Manager 3.5 (RPMs)' --releasever='6Server'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Agents for RHEL 6 Server (RPMs)' --releasever="6.6"
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Agents for RHEL 6 Server (RPMs)' --releasever="6.7"
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Agents for RHEL 6 Server (RPMs)' --releasever="6.8"
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Agents for RHEL 6 Server (RPMs)' --releasever="6Server"

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Manager 3.6 (RPMs)' 

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Management Agents for RHEL 7 (RPMs)' --releasever='7.1'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Management Agents for RHEL 7 (RPMs)' --releasever='7.2'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Management Agents for RHEL 7 (RPMs)' --releasever='7Server'

  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 7 Server - Supplementary (RPMs)' --releasever='7.1'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 7 Server - Supplementary (RPMs)' --releasever='7.2'
  # hammer repository-set enable --product 'Red Hat Enterprise Linux Server' --basearch='x86_64' --name 'Red Hat Enterprise Linux 7 Server - Supplementary (RPMs)' --releasever='7Server'
```

#### RHV
```
  # hammer repository-set enable --product 'Red Hat Virtualization Host' --basearch='x86_64' --name 'Red Hat Virtualization Host 7 (RPMs)'

  # hammer repository-set enable --product 'JBoss Enterprise Application Platform' --basearch='x86_64' --name 'JBoss Enterprise Application Platform 7 (RHEL 7 Server) (RPMs)' --releasever='7.2'
  # hammer repository-set enable --product 'JBoss Enterprise Application Platform' --basearch='x86_64' --name 'JBoss Enterprise Application Platform 7 (RHEL 7 Server) (RPMs)' --releasever='7Server'

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor 7 (RPMs)' --releasever='7.1'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor 7 (RPMs)' --releasever='7.2'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Enterprise Virtualization Hypervisor 7 (RPMs)' --releasever='7Server'

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Virtualization Manager 4.0 (RHEL 7 Server) (RPMs)'

  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Virtualization 4 Management Agents for RHEL 7 (RPMs)' --releasever='7.2'
  # hammer repository-set enable --product 'Red Hat Virtualization' --basearch='x86_64' --name 'Red Hat Virtualization 4 Management Agents for RHEL 7 (RPMs)' --releasever='7Server'
```

### Enabled Repos

```
  # hammer repository list
```
~~~
---|----------------------------------------------------------------------------------|----------------------------------------------|--------------|---------------------------------------------------------------------------------
ID | NAME                                                                             | PRODUCT                                      | CONTENT TYPE | URL                                                                             
---|----------------------------------------------------------------------------------|----------------------------------------------|--------------|---------------------------------------------------------------------------------
67 | Red Hat Virtualization Manager 4.0 RHEL 7 Server RPMs x86_64                     | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhv/4.0/os     
33 | Red Hat Virtualization Host 7 RPMs x86_64                                        | Red Hat Virtualization Host                  | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhvh/4/os      
69 | Red Hat Virtualization 4 Management Agents for RHEL 7 RPMs x86_64 7Server        | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhv-mgmt-age...
68 | Red Hat Virtualization 4 Management Agents for RHEL 7 RPMs x86_64 7.2            | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/rhv-mgmt-agent/4/os
5  | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhscl/1/os     
14 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/rhscl/1/os         
13 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/rhscl/1/os         
20 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rhscl/1/os     
29 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/rhscl/1/os         
28 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/rhscl/1/os         
27 | Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server x86_6... | Red Hat Software Collections for RHEL Server | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/rhscl/1/os         
4  | Red Hat Satellite Tools 6.2 for RHEL 7 Server RPMs x86_64                        | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/sat-tools/6....
19 | Red Hat Satellite Tools 6.2 for RHEL 6 Server RPMs x86_64                        | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/sat-tools/6....
6  | Red Hat Satellite Capsule 6.2 for RHEL 7 Server RPMs x86_64                      | Red Hat Satellite Capsule                    | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/sat-capsule/...
15 | Red Hat Satellite 6.2 for RHEL 7 Server RPMs x86_64                              | Red Hat Satellite                            | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/satellite/6....
51 | Red Hat Enterprise Virtualization Manager 3.6 RPMs x86_64                        | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rhevm/3.6/os   
50 | Red Hat Enterprise Virtualization Manager 3.5 RPMs x86_64 6Server                | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rhevm/3.5/os   
49 | Red Hat Enterprise Virtualization Manager 3.5 RPMs x86_64 6.8                    | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/rhevm/3.5/os       
48 | Red Hat Enterprise Virtualization Manager 3.5 RPMs x86_64 6.7                    | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/rhevm/3.5/os       
47 | Red Hat Enterprise Virtualization Manager 3.5 RPMs x86_64 6.6                    | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/rhevm/3.5/os       
54 | Red Hat Enterprise Virtualization Management Agents for RHEL 7 RPMs x86_64 7S... | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhev-mgmt-ag...
53 | Red Hat Enterprise Virtualization Management Agents for RHEL 7 RPMs x86_64 7.2   | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/rhev-mgmt-agent/...
52 | Red Hat Enterprise Virtualization Management Agents for RHEL 7 RPMs x86_64 7.1   | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/rhev-mgmt-agent/...
46 | Red Hat Enterprise Virtualization Hypervisor RPMs x86_64 6Server                 | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rhevh/os       
45 | Red Hat Enterprise Virtualization Hypervisor RPMs x86_64 6.8                     | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/rhevh/os           
44 | Red Hat Enterprise Virtualization Hypervisor RPMs x86_64 6.7                     | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/rhevh/os           
43 | Red Hat Enterprise Virtualization Hypervisor RPMs x86_64 6.6                     | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/rhevh/os           
66 | Red Hat Enterprise Virtualization Hypervisor 7 RPMs x86_64 7Server               | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rhevh/os       
65 | Red Hat Enterprise Virtualization Hypervisor 7 RPMs x86_64 7.2                   | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/rhevh/os           
64 | Red Hat Enterprise Virtualization Hypervisor 7 RPMs x86_64 7.1                   | Red Hat Virtualization                       | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/rhevh/os           
61 | Red Hat Enterprise Virtualization Agents for RHEL 6 Server RPMs x86_64 6Server   | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rhev-agent/3/os
60 | Red Hat Enterprise Virtualization Agents for RHEL 6 Server RPMs x86_64 6.8       | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/rhev-agent/3/os    
59 | Red Hat Enterprise Virtualization Agents for RHEL 6 Server RPMs x86_64 6.7       | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/rhev-agent/3/os    
58 | Red Hat Enterprise Virtualization Agents for RHEL 6 Server RPMs x86_64 6.6       | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/rhev-agent/3/os    
57 | Red Hat Enterprise Linux 7 Server - Supplementary RPMs x86_64 7Server            | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/supplementar...
56 | Red Hat Enterprise Linux 7 Server - Supplementary RPMs x86_64 7.2                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/supplementary/os   
55 | Red Hat Enterprise Linux 7 Server - Supplementary RPMs x86_64 7.1                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/supplementary/os   
1  | Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server                            | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/os             
8  | Red Hat Enterprise Linux 7 Server RPMs x86_64 7.2                                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/os                 
7  | Red Hat Enterprise Linux 7 Server RPMs x86_64 7.1                                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/os                 
2  | Red Hat Enterprise Linux 7 Server - RH Common RPMs x86_64 7Server                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/rh-common/os   
10 | Red Hat Enterprise Linux 7 Server - RH Common RPMs x86_64 7.2                    | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/rh-common/os       
9  | Red Hat Enterprise Linux 7 Server - RH Common RPMs x86_64 7.1                    | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/rh-common/os       
3  | Red Hat Enterprise Linux 7 Server Kickstart x86_64 7Server                       | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/kickstart      
12 | Red Hat Enterprise Linux 7 Server Kickstart x86_64 7.2                           | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/kickstart          
11 | Red Hat Enterprise Linux 7 Server Kickstart x86_64 7.1                           | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.1/x86_64/kickstart          
42 | Red Hat Enterprise Linux 6 Server - Supplementary RPMs x86_64 6Server            | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/supplementar...
41 | Red Hat Enterprise Linux 6 Server - Supplementary RPMs x86_64 6.8                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/supplementary/os   
40 | Red Hat Enterprise Linux 6 Server - Supplementary RPMs x86_64 6.7                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/supplementary/os   
39 | Red Hat Enterprise Linux 6 Server - Supplementary RPMs x86_64 6.6                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/supplementary/os   
16 | Red Hat Enterprise Linux 6 Server RPMs x86_64 6Server                            | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/os             
23 | Red Hat Enterprise Linux 6 Server RPMs x86_64 6.8                                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/os                 
22 | Red Hat Enterprise Linux 6 Server RPMs x86_64 6.7                                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/os                 
21 | Red Hat Enterprise Linux 6 Server RPMs x86_64 6.6                                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/os                 
17 | Red Hat Enterprise Linux 6 Server - RH Common RPMs x86_64 6Server                | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/rh-common/os   
26 | Red Hat Enterprise Linux 6 Server - RH Common RPMs x86_64 6.8                    | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/rh-common/os       
25 | Red Hat Enterprise Linux 6 Server - RH Common RPMs x86_64 6.7                    | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/rh-common/os       
24 | Red Hat Enterprise Linux 6 Server - RH Common RPMs x86_64 6.6                    | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/rh-common/os       
38 | Red Hat Enterprise Linux 6 Server - Optional RPMs x86_64 6Server                 | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/optional/os    
37 | Red Hat Enterprise Linux 6 Server - Optional RPMs x86_64 6.8                     | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/optional/os        
36 | Red Hat Enterprise Linux 6 Server - Optional RPMs x86_64 6.7                     | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/optional/os        
35 | Red Hat Enterprise Linux 6 Server - Optional RPMs x86_64 6.6                     | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/optional/os        
18 | Red Hat Enterprise Linux 6 Server Kickstart x86_64 6Server                       | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/kickstart      
32 | Red Hat Enterprise Linux 6 Server Kickstart x86_64 6.8                           | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.8/x86_64/kickstart          
31 | Red Hat Enterprise Linux 6 Server Kickstart x86_64 6.7                           | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.7/x86_64/kickstart          
30 | Red Hat Enterprise Linux 6 Server Kickstart x86_64 6.6                           | Red Hat Enterprise Linux Server              | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6.6/x86_64/kickstart          
62 | JBoss Enterprise Application Platform 7 RHEL 7 Server RPMs x86_64 7Server        | JBoss Enterprise Application Platform        | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/jbeap/7/os     
63 | JBoss Enterprise Application Platform 7 RHEL 7 Server RPMs x86_64 7.2            | JBoss Enterprise Application Platform        | yum          | https://cdn.redhat.com/content/dist/rhel/server/7/7.2/x86_64/jbeap/7/os         
34 | JBoss Enterprise Application Platform 6 RHEL 6 Server RPMs x86_64 6Server        | JBoss Enterprise Application Platform        | yum          | https://cdn.redhat.com/content/dist/rhel/server/6/6Server/x86_64/jbeap/6/os     
---|----------------------------------------------------------------------------------|----------------------------------------------|--------------|---------------------------------------------------------------------------------
# 
~~~


### Sync Repositories
```
  # for repo in $(hammer --csv --csv-separator ';' repository list --organization-id 1 |grep -v Id |awk -F ';' '{print $1}'); do echo $repo; hammer repository synchronize --async --id $repo;done
```
~~~
67
Repository is being synchronized in task 2ef86e82-d608-4dbd-8a44-c3a2198dabbc
33
Repository is being synchronized in task a2b409cc-d6c7-40d4-8712-e276c48272b6
69
Repository is being synchronized in task 0edf77e8-ea79-43f2-ba94-80eb4eaeb971
68
Repository is being synchronized in task 37da4085-fc1a-4716-945a-4b3aa886af1f
5
Repository is being synchronized in task cb77a370-2091-422c-b3ae-a7335702eac6
[...]
~~~
```
  # hammer task list |head -10
```
~~~
-------------------------------------|------|-------------------|---------------------|---------------------|---------|---------|----------------------------|---------------------------------------------------------------------------------
ID                                   | NAME | OWNER             | STARTED AT          | ENDED AT            | STATE   | RESULT  | TASK ACTION                | TASK ERRORS                                                                     
-------------------------------------|------|-------------------|---------------------|---------------------|---------|---------|----------------------------|---------------------------------------------------------------------------------
ffac2b67-ba8f-4d56-bb9e-47938d073c87 |      | admin             | 2016/10/24 08:29:54 |                     | running | pending | Synchronize                |                                                                                 
94b865ac-6289-4b26-b0b8-52e49fad4d46 |      | admin             | 2016/10/24 08:29:47 |                     | running | pending | Synchronize                |                                                                                 
eb4c445a-891d-4b8b-b38e-e240b7415a54 |      | admin             | 2016/10/24 08:29:45 |                     | running | pending | Synchronize                |                                                                                 
74b172ca-b149-49f9-aa97-3fad28c0ceae |      | admin             | 2016/10/24 08:29:42 |                     | running | pending | Synchronize                |                                                                                 
41422a3d-2d47-4e2b-b58d-a380e8700ab1 |      | admin             | 2016/10/24 08:29:39 |                     | running | pending | Synchronize                |                                                                                 
dc1d9cd7-8c00-413a-b891-ab02a915b1c6 |      | admin             | 2016/10/24 08:29:35 |                     | running | pending | Synchronize                |                                                                                 
72dfa33f-ee38-4f08-82ff-b886fa01e39e |      | admin             | 2016/10/24 08:29:33 |                     | running | pending | Synchronize                |                                                                                 
~~~

~~~
  # hammer task progress --id 83ee90c2-7962-4e87-9fb2-4d7930d4001e
  [......................................................................................................................................................................................................................................................................] [100%]
  # hammer task progress --id 790b93cf-c54c-4c88-b29c-78b910f95af8
  [...................................................................................................................................                                                                                                                                    ] [50%]
~~~


### Sync Plans
