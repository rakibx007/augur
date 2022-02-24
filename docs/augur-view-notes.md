```
server {
        listen 80;
        server_name  new.augurlabs.io;

        location /api/unstable/ {
                proxy_pass http://new.augurlabs.io:5044;
                proxy_set_header Host $host;
        }

	location / {
		proxy_pass http://127.0.0.1:5201;
	}

        error_log /var/log/nginx/census.osshealth.error.log;
        access_log /var/log/nginx/census.osshealth.access.log;

}
```


```
sean@augur:~/github/augur_view$ cat config.yml
approot: /
cache_expiry: 172800
caching: static/cache/
paginationOffset: 25
reports: reports.yml
serving: http://new.augurlabs.io/api/unstable
```


```
sean@augur:~/github/augur_view$ cat wsgi.py
from augur_view import app

if __name__ == '__main__':
        app.run()
```

```
sean@augur:~/github/augur_view$ cat gunicorn.conf 
import multiprocessing

workers = multiprocessing.cpu_count() * 2 + 1
bind = 'unix:AugurView.sock'
umask = 0o007
reload = True

#logging
accesslog = 'access.log'
errorlog = 'error.log'
```

```
sean@augur:~/github/augur_view$ ls status
ls: cannot access 'status': No such file or directory
sean@augur:~/github/augur_view$ ls static
cache  css  favicon  img
sean@augur:~/github/augur_view$ 
```


```
sean@augur:~/github/augur_view$ cat /etc/systemd/system/augur_view.service
[Unit]
Description=Gunicorn instance to serve augur_view flask application
After=network.target

[Service]
User=sean
WorkingDirectory=/home/sean/github/augur_view
ExecStart=/home/sean/github/virtualenv/augur_view/bin/gunicorn -c /home/sean/github/augur_view/gunicorn.conf -b 0.0.0.0:5201 wsgi:app

[Install]
WantedBy=multi-user.target
```
