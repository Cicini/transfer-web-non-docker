## About

This repository is a non-docker version of [transfer.zip](https://github.com/robinkarlberg/transfer.zip-web).

To run this repository you will also need to download [transfer-server](https://github.com/Cicini/transfer-server).

This repository may not update the official version, but you can manually change the official version through the README in this repository.

## Transfer

Transfer is a web application that allows you to easily transfer files between two devices. At the moment, transfer.zip is the easiest and most secure way to share files on the web.  There is no signup, no wait and no bullshit, and the files can be as large as you want. 

It uses [WebRTC](http://www.webrtc.org/) for peer-to-peer data transfer, meaning the files are streamed directly between peers and not stored anywhere in the process, not even on transfer.zip servers. To let peers initially discover each other, a signaling server is implemented in NodeJS using WebSockets, which importantly no sensitive data is sent through. In addition, the file data is end-to-end encrypted using [AES-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode) with a client-side 256 bit generated key, meaning if someone could impersonate a peer or capture the traffic, they would not be able to decrypt the file without knowing the key. Because the file is streamed directly between peers, there are **no file size or bandwidth limitations**. 

The easiest way to transfer a file is to scan the QR code containing the file link and encryption key. It is also possible to copy the link and share it to the receiving end over what medium you prefer the most. 

## Known Problems

Because of how peer-to-peer works, some network firewalls may not allow direct connections between devices. A solution for this is to use a [TURN server](https://webrtc.org/getting-started/turn-server), effectively relaying all file data, although encrypted, through a third party server. That is however against the whole purpose of this service, which is to be as secure as possible.

On some Safari browsers, the file download will not work because of bugs on Apple's part.

## Local Development

### Nginx Config

```nginx
server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name example.com;
        ssl_certificate     /path/to/your/cert;
        ssl_certificate_key /path/to/your/certkey;
        location / {
	      root /path/to/your/webroot;
              index index.html;
	}
        location /ws {
              proxy_pass http://127.0.0.1:8001;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection $connection_upgrade;
              proxy_set_header Host $host;
        }

        access_log off;
}
```

> **Note**
> How to Fix Unknown "connection_upgrade" Variable

put in `nginx.con` `http`

```nginx
map $http_upgrade $connection_upgrade {  
    default upgrade;
    ''      close;
}
````

Example:

```nginx
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    http2 on;
    map $http_upgrade $connection_upgrade {  
        default upgrade;
        ''      close;
    }
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
````
