Name
====

access_rule是[openwaf](https://github.com/titansec/openwaf)项目的接入规则模块

通过分析请求的域名和路径等信息，将请求转发至指定的后端服务器

Table of Contents
=================
* [Name](#name)
* [Synopsis](#synopsis)
* [TODO](#todo)
* [Modules Configuration Directives](#modules-configuration-directives)
* [Nginx Variables](#nginx-variables)

Synopsis
========
```txt
#nginx.conf

upstream server_1 {
    server 0.0.0.1; #just an invalid address as a place holder
    balancer_by_lua_file twaf_balancer.lua;
}

server {
    listen 443 ssl;
    server_name _;
    
    set $twaf_https 1;
    set $twaf_upstream_server "";
    
    ssl_certificate nginx.crt;
    ssl_certificate_key nginx.key;
    ssl_certificate_by_lua_file twaf_ssl_cert.lua;
    
    location / {
        proxy_pass $twaf_upstream_server;
    }
}

server {
    listen 80;
    server_name _;
    
    set $twaf_upstream_server "";
    
    location / {
        proxy_pass $twaf_upstream_server;
    }
}

#Configuration
{
    "twaf_access_rule": [
        "rules": [
            {   #单向认证
                "ngx_ssl": true,
                "ngx_ssl_cert": "ngx_a.crt",
                "ngx_ssl_key": "ngx_a.key",
                "host": "test_a.com", 
                "path": "/", 
                "server_ssl": false,
                "forward": "server_1", 
                "forward_addr": "1.1.1.1",
                "forward_port": "8080",
                "uuid": "access_567b067ff2060",
                "policy": "policy_1"
            },
            {   #双向认证
                "client_ssl": true,
                "client_ssl_cert": "client_a.cert",
                "ngx_ssl": true,
                "ngx_ssl_cert": "ngx_a.crt",
                "ngx_ssl_key": "ngx_a.key",
                "host": "test_b.com", 
                "path": "/", 
                "server_ssl": false,
                "forward": "server_1", 
                "forward_addr": "1.1.1.2",
                "forward_port": "8090",
                "uuid": "access_567b067ff2061",
                "policy": "policy_2"
            },
            {   
                "host": "test_c.com", 
                "path": "/", 
                "server_ssl": true,
                "forward": "server_1", 
                "forward_addr": "1.1.1.3",
                "uuid": "access_567b067ff2062",
                "policy": "policy_default"
            }
        ]
    }
}
```
[Back to TOC](#table-of-contents)

TODO
====
* proxy ssl 客户端认证
* proxy 支持域名

[Back to TOC](#table-of-contents)

Modules Configuration Directives
================================

```txt
{
    "twaf_access_rule": [
        "rules": [                                 -- 注意先后顺序
            {                                      
                "client_ssl": false,               -- 客户端认证的开关，与ngx_ssl组成双向认证
                "client_ssl_cert": "path",         -- 客户端认证所需PEM证书地址
                "ngx_ssl": false,                  -- nginx认证的开关
                "ngx_ssl_cert": "path",            -- nginx认证所需PEM证书地址
                "ngx_ssl_key": "path",             -- nginx认证所需PEM私钥地址
                "host": "^1\\.1\\.1\\.1$",         -- 域名，正则匹配
                "path": "\/",                      -- 路径，正则匹配
                "server_ssl": false,               -- 后端服务器ssl开关
                "forward": "server_5",             -- 后端服务器upstream名称
                "forward_addr": "1.1.1.2",         -- 后端服务器ip地址
                "forward_port": "8080",            -- 后端服务器端口号（缺省80）
                "uuid": "access_567b067ff2060",    -- 用来标记此规则的uuid
                "policy": "policy_uuid"            -- 安全策略ID
            }
        ]
    }
}
```
###rules
**syntax:** *"rules": table*

**default:** *none*

**context:** *twaf_access_rule*

table类型，接入规则，顺序匹配

###client_ssl
**syntax:** *"client_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

客户端认证开关，与ngx_ssl组成双向认证，默认false

###client_ssl_cert
**syntax:** *"client_ssl_cert": "path"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，客户端认证所需PEM证书地址，目前仅支持绝对地址

###ngx_ssl
**syntax:** *"ngx_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

boolean类型，服务器端(nginx)认证开关，与client_ssl组成双向认证，默认关闭

###ngx_ssl_cert
**syntax:** *"ngx_ssl_cert": "path"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，服务器端(nginx)认证所需PEM证书地址，目前仅支持绝对地址

###ngx_ssl_key
**syntax:** *"ngx_ssl_key": "path"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，服务器端(nginx)认证所需PEM私钥地址，目前仅支持绝对地址

###host
**syntax:** *"host": "ip|domain name regex"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，域名，正则匹配

例如:
```
    "host": "^1\\.1\\.1\\.1$"
    "host": "test\\.com"
    "host": "^.*\\.com$"
    "host": "www.baidu.com"
```

###path
**syntax:** *"path": "regex"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，路径，正则匹配

例如:
```
    "path": "/"
    "path": "/images"
    "path": "/[a|b]test"
```

###server_ssl
**syntax:** *"server_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

boolean类型，OpenWAF向后端服务器连接的ssl开关

例如:
```
    upstream test {
    	server 1.1.1.1;
    }
    
    http {
    	server {
    	    listen 80;
    	    server_name _;
    	    
    	    location / {
    	        #server_ssl为true，相当于proxy_pass后为https
    	    	proxy_pass https://test;
    	        #server_ssl为false，相当于proxy_pass后为http
    	    	#proxy_pass http://test;
    	    }
    	}
    }
```

###forward
**syntax:** *"forward": "upstream_uuid"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，forward表示后端服务器的uuid，即upstream的名称

```
    #如：forward值为test
    upstream test {
        server 1.1.1.1;
    }
```

###forward_addr
**syntax:** *"forward_addr": "ip"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，forward_addr表示后端服务器的ip地址（TODO：支持域名）

```
    upstream test {
        #如：forward_addr值为1.1.1.1
    	server 1.1.1.1;
    }
```

###forward_port
**syntax:** *"forward_port": port*

**default:** *80*

**context:** *twaf_access_rule*

number类型，forward_port表示后端服务器端口号，默认80

```
    upstream test {
    	#如：forward_port值为50001
    	server 1.1.1.1:50001;
    }
```

###uuid
**syntax:** *"uuid": "string"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，接入规则的唯一标识

###policy
**syntax:** *"policy": "policy_uuid"*

**default:** *none*

**context:** *twaf_access_rule*

string类型，满足此接入规则的请求，所使用安全策略的uuid

[Back to TOC](#table-of-contents)

Nginx Variables
===============

###$twaf_https
**syntax:** *set $twaf_https 0|1*

**default:** *0*

**context:** *server*

用于标记请求是否通过ssl加密

"set $twaf_https 1"，则表示请求通过ssl加密

"set $twaf_https 1"，则表示请求未通过ssl加密

```
server {
    listen 443 ssl;
    set $twaf_https 1;
    ...
}

server {
    listen 80;
    set $twaf_https 0;
    ...
}
```

###$twaf_upstream_server
**syntax:** *set $twaf_upstream_server ""*

**default:** *none*

**context:** *server*

用于指定后端服务器地址，只需初始化为空字符串即可，其值由"server_ssl"和"forward"确定

```
upstream server_1 {
    ...
}

upstream server_2 {
    ...
}

server {
    ...
    
    set $twaf_upstream_server "";
    location / {
        ...
        proxy_pass $twaf_upstream_server;
    }
}

若"server_ssl"值为true, "forward"值为"server_1"
等价于proxy_pass https://server_1;

若"server_ssl"值为false, "forward"值为"server_2"
等价于proxy_pass http://server_2;
```

[Back to TOC](#table-of-contents)
