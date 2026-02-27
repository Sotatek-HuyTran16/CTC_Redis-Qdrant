# 0. Ensure using following resources

- Ingress nginx + MetalLB
- Enable ingress in helm chart

# 1. Caddy file

```bash

{
    auto_https off
    order dify_audit before reverse_proxy
    order dify_credential_patcher before reverse_proxy
}
http://console.dify.local:3080 {
header {
    -Server
    -X-Powered-By
    Strict-Transport-Security "max-age=31536000;"
    Content-Security-Policy "frame-ancestors 'none';"
    Access-Control-Allow-Origin  http://console.dify.local
    Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS"
    Access-Control-Allow-Headers "Authorization, Content-Type, Accept, Origin, X-Requested-With"
    Access-Control-Allow-Credentials true
    Access-Control-Max-Age 86400
}
    route {
dify_audit "redis+sentinel://:sotatek@redis-node-0.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-1.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-2.redis-headless.dev-redis.svc.cluster.local:26379/0?sentinelService=mymaster" 7

        handle /console/api/workspaces/current/tool-provider/builtin/* {
dify_credential_patcher "redis+sentinel://:sotatek@redis-node-0.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-1.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-2.redis-headless.dev-redis.svc.cluster.local:26379/0?sentinelService=mymaster"
            reverse_proxy http://dify-api-svc:5001
        }
        handle /console/api/workspaces/current/model-providers* {
dify_credential_patcher "redis+sentinel://:sotatek@redis-node-0.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-1.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-2.redis-headless.dev-redis.svc.cluster.local:26379/0?sentinelService=mymaster"
            reverse_proxy http://dify-api-svc:5001
        }
        handle /console/api/enterprise/* {
            reverse_proxy http://dify-enterprise-svc:8082
        }
        handle /console/api/account/* {
            reverse_proxy http://dify-api-svc:5001
        }
        handle /console/api/* {
            reverse_proxy http://dify-api-svc:5001
        }
        handle /v1/* {
            reverse_proxy http://dify-api-svc:5001
        }
        handle /api/enterprise/* {
            reverse_proxy http://dify-enterprise-svc:8082
        }
        handle /api/* {
            reverse_proxy http://dify-api-svc:5001
        }
        handle /e/* {
            reverse_proxy http://dify-plugin-daemon-svc:5002 {
                header_up Dify-Hook-Url {http.request.scheme}://{host}{uri}
            }
        }
        handle {
            reverse_proxy http://dify-enterprise-frontend-svc:3000
        }
        handle_path /healthz {
            respond "OK"
        }
    }
    encode zstd gzip
reverse_proxy {
    transport http {
        dial_timeout 10s
        keepalive 30s
        keepalive_interval 30s
    }
}
}

http://enterprise.dify.local:3080 {
header {
    -Server
    -X-Powered-By
    Strict-Transport-Security "max-age=31536000;"
    Content-Security-Policy "frame-ancestors 'none';"
    Access-Control-Allow-Origin  http://console.dify.local
    Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS"
    Access-Control-Allow-Headers "Authorization, Content-Type, Accept, Origin, X-Requested-With"
    Access-Control-Allow-Credentials true
    Access-Control-Max-Age 86400
}
    route {
dify_audit "redis+sentinel://:sotatek@redis-node-0.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-1.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-2.redis-headless.dev-redis.svc.cluster.local:26379/0?sentinelService=mymaster" 7
        handle /v1/audit/* {
            reverse_proxy http://dify-enterprise-audit-svc:8083
        }
        handle /admin-api/* {
            reverse_proxy http://dify-enterprise-svc:8082
        }
        handle /scim/* {
            reverse_proxy http://dify-enterprise-svc:8082
        }
        handle /v1/plugin-manager/* {
            reverse_proxy http://dify-plugin-manager-svc:8084
        }
        handle /v1/* {
            reverse_proxy http://dify-enterprise-svc:8082
        }
        handle /* {
            reverse_proxy http://dify-enterprise-frontend-svc:3000
        }
    }
    encode zstd gzip
    respond /healthz 200 {
        body "OK"
    }
reverse_proxy {
    transport http {
        dial_timeout 10s
        keepalive 30s
        keepalive_interval 30s
    }
}
}
http://app.dify.local:3080 {
    handle /api/enterprise/* {
        reverse_proxy http://dify-enterprise-svc:8082
    }
    handle /api/* {
dify_audit "redis+sentinel://:sotatek@redis-node-0.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-1.redis-headless.dev-redis.svc.cluster.local:26379,redis-node-2.redis-headless.dev-redis.svc.cluster.local:26379/0?sentinelService=mymaster" 7
        reverse_proxy http://dify-api-svc:5001
    }
    handle {
        reverse_proxy http://dify-enterprise-frontend-svc:3000
    }
}

http://upload.dify.local:3080 {
    handle /files/* {
        reverse_proxy http://dify-api-svc:5001
    }
}

http://api.dify.local:3080 {
    handle /mcp/* {
        reverse_proxy http://dify-api-svc:5001
    }
    handle /v1/* {
        reverse_proxy http://dify-api-svc:5001
    }
}
```

# 2. Enterprise frontend config

```bash
API_URL = http://enterprise.dify.local
APP_API_URL = http://app.dify.local
CONSOLE_API_URL = http://console.dify.local
```