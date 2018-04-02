## 申请 Let's Encrypt 数字证书

1. 从 [github][1] 上下载 certbot

2. 运行 certbot (-d 参数指定对应域名)
	- 如果知道软件的类型，如Apache, nginx等，运行
	`./certbot-auto --apache -d example.com -d www.example.com -d other.example.net`
	- 如果不知道就运行
	`./certbot-auto certonly --standalone --email admin@example.com -d example.com -d www.example.com -d other.example.net`
	
成功后会有以下输出：
```
	IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.suikp.cn/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.suikp.cn/privkey.pem
   Your cert will expire on 2018-03-05. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
上面标出了证书(fullchain.pem)和私钥(privkey.pem)的路径

[1]:https://github.com/certbot/certbot


3. 更新证书: `./certbot-auto renew` 