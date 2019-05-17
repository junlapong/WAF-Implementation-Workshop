# Web Application Firewall Implementation Workshop

By: [SOSECURE](https://www.facebook.com/s0secure)

## Presentation

--- WAIT WEBINAR VIDEO AND SLIDE ---

### Theory

![](https://lh3.googleusercontent.com/proxy/254YZFei7cCxFzjRx-RYtYnQKu3FfsYwkl8jHVS4jV4N7glCFq7npMrl4OQISvJ2fQECynoJ1LJszS8H1bMxknkGOsmKfqFOFHpXceChJ5orQrGRmNLJo1iNa43BGf13l-sYBqu3sHjKpgSYtrUUjmhi_qRCvWoS8y1zKK95YvOYf5dkIY8WsJKcTAAfy6eypB-RNUxbbOG3ialcxmuX1Y8=s0-d)

- [OSI MODEL 7 Layers ทำหน้าที่อะไรบ้าง และแต่ละ Layers มีหน้าที่อย่างไร?](https://sites.google.com/site/worawanfies18/phakh-reiyn-thi-2-60/osi-model-7-layers-tha-hnathi-xari-bang-laea-taela-layers-mihna-thi-xyangri)
- [How To Secure Nginx with NAXSI on Ubuntu 16.0](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-naxsi-on-ubuntu-16-04)

## Lab Environment

### VMWare Fusion for macOS

```
brew cask install vmware-fusion
```

### VMWare Workstation for Windows

- [Download](https://sosecure-my.sharepoint.com/:u:/g/personal/watcharaphon_wo_sosecure_co_th/EbE1I-naVXpKuhrnTnVcc00BpkokNlYncQfF6foxfVhGmg?e=QiVywd)

- [VDO Installation](https://sosecure-my.sharepoint.com/:u:/g/personal/watcharaphon_wo_sosecure_co_th/EaHpH9vjGTxCt_AWO5R_3J8BHKsgyAN0AdXtEPKxHkm2WQ?e=z0Lc7y)


### Virtual Image

- [Downoad](https://sosecure-my.sharepoint.com/:u:/g/personal/watcharaphon_wo_sosecure_co_th/EX_H50eX4PhJnrNjilV4Ik0BiIbBaNJ6-tHYMP26Y5s9KQ?e=un16JU)

## Workshop

- [Lab Notes](https://docs.google.com/document/d/1KhOSFTzQpkgkJ2v-ScpWE8sjRmJbJBCGDma1gOtzfxc/edit)

- Vuln Web: http://122.154.75.206:6789

### Login to Ubuntu
- username: root
- password: TqwrZE56E55a

### NAXSI Installation

```
root# apt-get update
root# apt-get install nginx -y
root# apt-get install g++ gcc libpcre3-dev libssl-dev zlib1g-dev libxml2-dev libxslt1-dev libgd-dev libgeoip-dev -y

--- TODO ---

root# wget https://nginx.org/download/nginx-1.14.2.tar.gz
root# wget https://github.com/nbs-system/naxsi/archive/0.56.tar.gz -O naxsi
root# tar -xvpzf nginx-1.14.2.tar.gz
root# tar -xvf naxsi

root# cd nginx-1.14.2
root# ./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --add-module=../naxsi-0.56/naxsi_src/ --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module

root# make
root# sudo make install
root# cp ~/naxsi-0.56/naxsi_config/naxsi_core.rules /etc/nginx/

```


### Customize Rules

```
root# nano /etc/nginx/naxsi.rules

SecRulesEnabled;
DeniedUrl "/waf/error.html";

## Check for all the rules
CheckRule "$SQL >= 8" BLOCK;
CheckRule "$RFI >= 8" BLOCK;
CheckRule "$TRAVERSAL >= 4" BLOCK;
CheckRule "$EVADE >= 4" BLOCK;
CheckRule "$XSS >= 8" BLOCK;
CheckRule "$UPLOAD >= 8" BLOCK;

root# mkdir /var/www/html/waf
root# nano /var/www/html/waf/error.html

<html>
  <head>
    <title>Blocked By NAXSI</title>
  </head>
  <body>
    <div style="text-align: center">
      <h1>Malicious Request</h1>
      <hr>
      <p>This Request Has Been Blocked By NAXSI.</p>
    </div>
  </body>
</html>

root# nano /etc/nginx/nginx.conf
include /etc/nginx/naxsi_core.rules;

include /etc/nginx/mime.type;
default_type application/octet-stream;
include /etc/nginx/naxi_core.rules;

root# rm -rf /etc/nginx/sites-enabled/default
root# nano /etc/nginx/conf.d/waf.conf

server {
	listen 90;
	listen [::[:80;
	server_name www.local.net;
	server_tokens off;
	
	location /waf {
		root /var/www/html;
	}
	
	location / {
		include /etc/nginx/naxi.rules;
		proxy_pass http://122.154.75.206:6789;
	}
}

root# mkdir -p /var/lib/nginx/body
root# nginx -t
root# systemctl restart nginx
root# systemctl enable nginx
```

### Test Web Attack Payload

#### Direct to Web Server
- http://122.154.75.206:6789/xss/example1.php?name=<script>alert(123)</script>
- http://122.154.75.206:6789/xss/example2.php?name=%3CSCRIPT%3Ealert(123)%3C/SCRIPT%3E
- http://122.154.75.206:6789/fileincl/example1.php?page=/etc/passwd

#### Via WAF
- xxx

### Additional Information
- https://www.owasp.org/index.php/OWASP_Broken_Web_Applications_Project
- https://pentesterlab.com



