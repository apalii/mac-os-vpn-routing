# Mac OS VPN routing to AWS

### Fetch and parse AWS ips range

I assume that VPN is already configured and we need to route traffic through VPN only for needed resources instead of absolute all traffic. 
Open your `Network Settings => Choose you VPN connection => Advanced`
and disable `Send all traffic over VPN`

Also, I assume that you are familiar with Python programming language which will be used further. 
Let's install an appropriate Python version from https://www.python.org/downloads/
At this moment 3.7.3 is the latest version.
```bash
$ python3 -V
Python 3.7.3
```


Create a virtual environment:
```bash
python3 -m venv ipython
cd ipython/
```
Activate it and install all the needed packeges :
   ```
   . bin/activate
   pip install --upgrade pip
   pip install requests
   pip install ipython
   ```
Open the ipython terminal using `ipython` command. Here is an example how to fetch IP networks renges for AWS us-east-1 data centre :

```python

import requests as r
ips = r.get('https://ip-ranges.amazonaws.com/ip-ranges.json').json()['prefixes']

# Here we filter IP ranges only for us-east-1
list_ips_us_east_1 = [ i['ip_prefix'] for i in ips if i['region'] == 'us-east-1']
list_ips_us_east_1
len(list_ips_us_east_1)

# And here we filter CIDRs which start with '54.' '4.' and '3.' for my individual needs.
# Feel free to skip or change this step
needed_list = [i for i in list_ips_us_east_1 if i.startswith('54') or i.startswith('4.') or i.startswith('3.')]

# Let's create all the needed route commands
for i in needed_list:
    print(f'sudo /sbin/route add -net {i} -interface $1')
```

Let's cteate ip-up file and put all the route commands in it:
```bash
sudo nano /etc/ppp/ip-up
```
File structure:
```
#!/bin/sh
/sbin/route add 192.168.0.0/24 -interface $1
...
```
Set permissions:
```bash
sudo chmod +x /etc/ppp/ip-up
```
That's it. Now your traffic to AWS us-east-1 will be routed over VPN, respectively all other traffic over standart connection.
In order to check your routing table execute the following command:
```bash
netstat -nr 
```

Alternatively you can create alias in your ~/.profile for particular host
```
alias host-route-add='sudo /sbin/route add -host 192.168.1.1 -interface ppp0'
```
Or particular network
```
alias net-route-add='sudo /sbin/route add -net 192.168.1.1/27 -interface ppp0'
```
To remove the needed route:
```
sudo /sbin/route delete 192.168.1.1/27 -interface ppp0
```
