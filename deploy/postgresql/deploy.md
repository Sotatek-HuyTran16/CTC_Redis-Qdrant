# 1. Admin user create 

## Edit values.yaml file

```bash
users:
  - name: admin
    grantPublicSchemaAccess: true
    options: "SUPERUSER"
    password:
      type: ASCII
    secretName: "admin-credentials"
```