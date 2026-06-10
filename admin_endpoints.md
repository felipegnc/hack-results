# 🚨 ZARION ADMIN ENDPOINTS DESCOBERTOS

**Fonte:** Análise dos bundles JavaScript (ESPECTRO)

## Admin Applications
```
GET    /api/admin/applications?page=&limit=&search=&status=&planId=&expired=&blocked=
GET    /api/admin/applications/stats
GET    /api/admin/applications/:id
GET    /api/admin/applications/audit/logs
POST   /api/admin/applications/:id/block|unblock|restart|stop|start|restore
PUT    /api/admin/applications/:id
PUT    /api/admin/applications/:id/ram
DELETE /api/admin/applications/:id?deleteFrom=&permanent=
```

## Admin Users
```
GET    /api/admin/users?page=&limit=&search=&admin=&from=&to=&sort=
PUT    /api/admin/users/:id/admin ← PROMOVE/DESPROMOVE ADMIN!
```

## Admin Coupons
```
GET    /api/admin/coupons
POST   /api/admin/coupons
PUT    /api/admin/coupons/:name
DELETE /api/admin/coupons/:name
POST   /api/admin/coupons/:name/archive|restore
GET    /api/admin/coupons/audit
```

## Admin Plans
```
GET    /api/admin/plans/list
POST   /api/admin/plans/create
PUT    /api/admin/plans/update/:id
DELETE /api/admin/plans/remove/:id
POST   /api/admin/plans/upload/:id/zip
GET    /api/admin/plans/download/:id/zip
```

## Admin Payments
```
GET    /api/admin/payments?page=&limit=&planId=&coupon=&user=
GET    /api/admin/payments/stats?period=
```

## Status: Acesso negado (403)
Precisa de JWT com flag `admin: true`. Tentativas de forgery em andamento.
