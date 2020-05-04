```
server {
  listen 80;
  include /etc/nginx/conf/le-combiworks.conf;
  server_name dletta.rig.airfaas.com;
  return 301 https://dletta.rig.airfaas.com$request_uri$args;
}
server {
   listen 443 ssl http2;
   include /etc/nginx/conf/le-airfaas.conf; # certificate configs
   include /etc/nginx/conf/le-combiworks.conf; # certificate renewal passthrus
   server_name dletta.rig.airfaas.com;
   root /jail/home/dletta/test/node_modules/gun/examples/;
        location /gun {
          rewrite /gun/(.*) /$1 break;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $host;
          proxy_set_header X-Rewrite-To $PROJECT;
          proxy_set_header X-NginX-Proxy true;
          proxy_pass_header  Set-Cookie;
          proxy_pass_header  P3P;
          proxy_pass_header X-Boxer-Signature;
          proxy_pass_header boxerignature;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
          proxy_redirect off;
          proxy_pass http://localhost:8765;
        }

}
```

Please [contribute to this](https://github.com/amark/gun/wiki/nginx/_edit).