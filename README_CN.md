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
