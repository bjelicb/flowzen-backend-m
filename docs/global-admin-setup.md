# Global Admin Setup

## Superadmin Bootstrap
- Endpoint: `POST /admin/setup/superadmin`
- Body fields: `protectToken`, `superadmin` (`email`, `password`), optional `scopes`
- Use protect token `superadminSetup2025`
- Response returns per-step status plus the plain-text password for confirmation

### Example Request
```
POST /admin/setup/superadmin
{
  "protectToken": "superadminSetup2025",
  "superadmin": {
    "email": "global.admin@flowzen.io",
    "password": "superadmin"
  },
  "scopes": [
    "global.tenants:read",
    "global.tenants:create",
    "global.tenants:update",
    "global.tenants:suspend",
    "global.tenants:activate",
    "global.users:read",
    "global.users:create",
    "global.users:update",
    "global.users:delete",
    "global.scopes:*",
    "global.audit:*",
    "global.settings:*"
  ]
}
```

### What the Setup Does
- Validates the protect token
- Upserts the supplied global scopes (`category: 'global'`)
- Upserts the `superadmin` role (`tenant = null`, `type = 'global'`) and seeds the provided scopes
- Upserts the superadmin user (`tenant = null`) with the hashed password
- Normalises all tenants (`status = 'active'`, clears suspension fields)
- Migrates existing tenant scopes (`category: 'tenant'`) and roles (`type: 'tenant'`)
- Writes an `auditLogs` entry with action `superadmin-setup`
- Returns a detailed log of each operation together with plaintext password confirmation

## Rollback
- Endpoint: `POST /admin/setup/superadmin/rollback`
- Body fields: `protectToken`, `superadminEmail`, `scopesPrefix`
- Runs in a transaction and returns per-step results

### Example Request
```
POST /admin/setup/superadmin/rollback
{
  "protectToken": "superadminSetup2025",
  "superadminEmail": "global.admin@flowzen.io",
  "scopesPrefix": "global."
}
```

### What the Rollback Does
- Validates the protect token
- Removes scopes matching the prefix
- Deletes the `superadmin` role if it is no longer referenced
- Deletes the specified superadmin user
- Writes an `auditLogs` entry with action `superadmin-rollback`
- Returns a detailed per-step log describing deleted or skipped entities

## Default Credentials
- Email: `global.admin@flowzen.io`
- Password: `superadmin`
- Change the password immediately after setup via the regular user management flows

## Postman & Verification
- Import the backend Postman collection and run the `Superadmin Setup` and `Rollback` requests
- Suggested smoke flow: setup → login as superadmin → call `/admin/tenants` (200) → optionally suspend/activate a tenant → verify audit log → run rollback

## Notes
- No environment variables are required; these endpoints handle seeding directly
- After running setup, global scopes are ready for the Angular admin shell
- Tenant users are blocked from `/admin/*` routes by `SuperAdminGuard` and `GlobalScopesGuard`

