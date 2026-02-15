# JWT Authentication Implementation Summary

## Tasks Completed

### Task 1: App Backend JWT Authentication Infrastructure ✅
- **Added dependencies**: PyJWT, cryptography, django-cors-headers
- **Fixed missing import**: Added `import os` to settings.py
- **Created JWT auth backend**: `config/auth.py` with JWTAuthenticationBackend class
- **Updated Django settings**:
  - Added corsheaders to INSTALLED_APPS
  - Updated MIDDLEWARE (added CorsMiddleware, removed CsrfViewMiddleware)
  - Added AUTHENTICATION_BACKENDS with JWT backend
  - Added JWT configuration (public key path, algorithm, CORS settings)
- **Updated API endpoints**: `domain/api.py` now uses JWTAuthenticationBackend instead of x_session_token_auth
- **Updated tests**: `domain/tests.py` now generates and uses JWT tokens for API testing
- **Added JWT public key**: Copied from ID service to `config/keys/jwt_public_key.pem`

### Task 2: ID Service CORS Update ✅
- **Updated CORS configuration**: Added `https://app.digidex.bio` to `CORS_ALLOWED_ORIGINS` in ID service
- **Supports token refresh**: App frontend can now make cross-origin requests to ID service token endpoints

## Docker Configuration

### Compose File Updates

#### Production (compose.yaml)
```yaml
app-backend:
  secrets:
    - jwt_public_key
  environment:
    - JWT_PUBLIC_KEY_PATH=/run/secrets/jwt_public_key
```

**How it works in production:**
1. Docker secret `jwt_public_key` is defined from `../secrets/public_key.pem`
2. Secret is mounted at `/run/secrets/jwt_public_key` inside container
3. `JWT_PUBLIC_KEY_PATH` env var points to the mounted secret
4. Settings.py reads the key at startup

#### Development (compose.override.yaml)
```yaml
app-backend:
  environment:
    - JWT_PUBLIC_KEY_PATH=/home/web/code/backend/config/keys/jwt_public_key.pem
  volumes:
    - ../id/backend/config/keys:/home/web/code/backend/config/keys:ro
```

**How it works in development:**
1. Volume mount shares ID service's keys directory (read-only)
2. `JWT_PUBLIC_KEY_PATH` points to the mounted volume
3. Allows local testing without Docker secrets

### Environment Variables

#### .env.dev
```
JWT_PUBLIC_KEY_PATH=/home/web/code/backend/config/keys/jwt_public_key.pem
# Optional JWT claim validation
# JWT_ISSUER=https://id.digidex.bio
# JWT_AUDIENCE=app-backend
```

#### .env.prod
```
JWT_PUBLIC_KEY_PATH=/run/secrets/jwt_public_key
# Optional JWT claim validation
# JWT_ISSUER=https://id.digidex.bio
# JWT_AUDIENCE=app-backend
```

## Key Features

### JWT Auth Backend (`config/auth.py`)
- Validates JWT tokens using ID service's RS256 public key
- Handles token expiration, invalid signature, wrong algorithm
- Auto-creates Django User from JWT claims if not found
- Sets `request.user` for downstream authentication checks

### CORS Configuration
Allows API requests from:
- Production: `https://app.digidex.bio`
- Development: `http://localhost:3000`, `http://localhost:3001`

### Test JWT Generation
`domain/tests.py` includes `create_test_jwt_token()` helper that:
1. Loads ID service's RS256 private key
2. Creates a valid JWT with user claims
3. Returns token for API testing

## Deployment Notes

### What's in Docker Container
- `backend/config/keys/jwt_public_key.pem` is copied to container via COPY in Dockerfile
- Settings.py loads public key from `JWT_PUBLIC_KEY_PATH` at startup

### Production Secret Mount
The app-backend service expects:
1. Docker secret named `jwt_public_key` to exist
2. Secret to contain the ID service's RS256 public key
3. Secret mounted at `/run/secrets/jwt_public_key` in container

### Key File Locations

| Environment | Path | Type |
|-------------|------|------|
| Dev (local) | `app/backend/config/keys/jwt_public_key.pem` | File in repo |
| Dev (Docker) | `/home/web/code/backend/config/keys/jwt_public_key.pem` | Volume mount from host |
| Prod | `/run/secrets/jwt_public_key` | Docker secret mount |

## Configuration Summary

| Component | Value |
|-----------|-------|
| JWT Algorithm | RS256 (asymmetric) |
| Public Key Source | ID service (`id/backend/config/keys/jwt_public_key.pem`) |
| Token Format | Bearer token in Authorization header |
| Session Handling | Signed cookies (no database sessions) |
| CSRF Protection | Removed (JWT is inherently CSRF-safe) |

## Next Steps

1. **Task 3**: Implement app frontend auth module (token storage, auth context)
2. **Task 4**: Add ID service login redirect flow support
3. **Task 5**: DevOps configuration (handled by devops-engineer)
4. **Task 6**: Integration testing across services

## Testing the Implementation

### Local Docker Testing
```bash
cd /path/to/digidex/app
docker compose -f compose.yaml -f compose.override.yaml up app-backend

# Container should start successfully with JWT public key loaded
# Check logs for any errors loading the key
```

### Test JWT Token Generation
```bash
pytest domain/tests.py -v
# Tests generate JWT tokens and make API calls with Bearer tokens
```

### Verify CORS
```bash
curl -X OPTIONS http://localhost:8003/api/nfctags \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: GET" -v
# Should return 200 with CORS headers
```
