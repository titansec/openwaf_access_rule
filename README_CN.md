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
