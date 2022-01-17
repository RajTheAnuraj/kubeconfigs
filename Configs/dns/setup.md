Install dnsmasq
```
apt install dnsmasq
```

the config is stored in /etc/dnsmasq.conf

Uncomment the below lines for security. 
What it does is in the file as comments

```
#domain-needed
#bogus-priv
```

Add the upstream servers as below towards the end of the file
```
server=8.8.8.8
server=4.4.4.4
```

Adjust the cache size. The default is 150
```
cache-size=1000
```

Add all the mappings you need in /etc/hosts file


Restart the service
```
sudo service dnsmasq restart
```

Check the status
```
sudo service dnsmasq status
```