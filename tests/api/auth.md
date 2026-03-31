# Authentication

All API requests require a Bearer token in the `Authorization` header.

## Token Format
```
Authorization: Bearer <token>
```

## Obtaining a Token
POST to `/auth/login` with `username` and `password`.
