Name
====
openwaf_reqstat-openwaf的统计模块，对访问信息、安全信息、转发信息等进行统计

Table of Contents
=================
* [Name](#name)
* [Synopsis](#synopsis)
* [Statistical information](#statistical-information)
* [Prerequisite modules](#prerequisite-modules)
* [Configuration Directives](#configuration-directives)
* [Directives](#directives)

Synopsis
========
```lua
    -- app/openwaf_init.lua
    local twaf_reqstat_m = require "lib.twaf.twaf_reqstat"
    twaf_reqstat = twaf_reqstat_m:new(config, policy_uuids)
    
    -- app/openwaf_init_worker.lua
    twaf_reqstat:init_worker()
    
    -- app/api.lua
    api["get"]["reqstat"] = function() twaf_reqstat:reqstat_show_handler(policy_uuid) end
    api["delete"]["reqstat"] = function() twaf_reqstat:reqstat_clear() end
    
    -- app/openwaf_log.lua
    twaf_reqstat:reqstat_log_handler(events, policy_uuid)
```
[Back to TOC](#table-of-contents)


[Back to TOC](#table-of-contents)


Configuration Directives
========================
```txt
{
    "twaf_access_rule": [
        "rules": [                                 -- 注意先后顺序
            {                                      
                "client_ssl": false,               -- 客户端认证的开关，与ngx_ssl组成双向认证
                "client_ssl_cert": "path",         -- 客户端认证所需公钥地址
                "ngx_ssl": false,                  -- nginx认证的开关
                "ngx_ssl_cert": "path",            -- nginx认证所需公钥地址
                "ngx_ssl_key": "path",             -- nginx认证所需私钥地址
                "host": "^1\\.1\\.1\\.1$",         -- 域名，支持字符串、正则
                "path": "\/",                      -- 路径，支持字符串、正则
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

接入规则，顺序执行

###client_ssl
**syntax:** *"client_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

客户端认证开关，与ngx_ssl组成双向认证，默认false

###client_ssl_cert
**syntax:** *"client_ssl_cert": "path"*

**default:** *none*

**context:** *twaf_access_rule*

客户端认证所需公钥地址

###ngx_ssl
**syntax:** *"ngx_ssl": true|false*

**default:** *false*

**context:** *twaf_access_rule*

服务器端(nginx)认证开关，与client_ssl组成双向认证，默认关闭

###ngx_ssl_cert
**syntax:** *"ngx_ssl_cert": "path"*

**default:** *none*

**context:** *twaf_access_rule*

服务器端(nginx)认证所需公钥地址

###ngx_ssl_key
**syntax:** *"ngx_ssl_key": "path"*

**default:** *none*

**context:** *twaf_access_rule*

服务器端(nginx)认证所需私钥地址

###host
**syntax:** *"host": "ip|domain name string|regex"*

**default:** *none*

**context:** *twaf_access_rule*

域名，支持正则

例如:
```
    "host": "^1\\.1\\.1\\.1$"
    "host": "test\\.com"
    "host": "^.*\\.com$"
```

###path
**syntax:** *"path": "string|regex"*

**default:** *none*

**context:** *twaf_access_rule*

路径，支持字符串及正则

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

OpenWAF向后端服务器连接的ssl开关

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
**syntax:** *"forward": "string"*

**default:** *none*

**context:** *twaf_access_rule*

forward表示后端服务器的uuid即upstream的名称

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

forward_addr表示后端服务器的ip地址（TODO：支持域名）
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

forward_port表示后端服务器端口号，默认80

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

接入规则的唯一标识

###policy
**syntax:** *"policy": "policy_uuid"*

**default:** *none*

**context:** *twaf_access_rule*

满足此接入规则的请求，所使用安全策略的ID
