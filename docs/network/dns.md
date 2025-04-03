BIND DNS

```bash
$TTL 1W
@	IN	SOA	ns1.example.com.	root (
			2019070700	; serial
			3H		; refresh (3 hours)
			30M		; retry (30 minutes)
			2W		; expiry (2 weeks)
			1W )		; minimum (1 week)
	IN	NS	ns1.example.com.
	IN	MX 10	smtp.example.com.
;
;
ns1.example.com.		IN	A	192.168.1.5
smtp.example.com.		IN	A	192.168.1.5
;
helper.example.com.		IN	A	192.168.1.5
helper.ocp4.example.com.	IN	A	192.168.1.5
;
api.ocp4.example.com.		IN	A	192.168.1.5 
api-int.ocp4.example.com.	IN	A	192.168.1.5 
;
*.apps.ocp4.example.com.	IN	A	192.168.1.5 
;
bootstrap.ocp4.example.com.	IN	A	192.168.1.96 
;
control-plane0.ocp4.example.com.	IN	A	192.168.1.97 
control-plane1.ocp4.example.com.	IN	A	192.168.1.98 
control-plane2.ocp4.example.com.	IN	A	192.168.1.99 
;
compute0.ocp4.example.com.	IN	A	192.168.1.11 
compute1.ocp4.example.com.	IN	A	192.168.1.7 
;
;EOF
```

该文件中，DNS record 可以用简单的host名代替 to use just `helper` instead of `helper.example.com.`, but it depends on how your DNS resolution is configured.

Example of a /etc/resolv.conf file:
```bash
search example.com
nameserver 192.168.1.5
```
这种情况下可以只用 `helper`  比如：
```bash
helper		IN	A	192.168.1.5
```