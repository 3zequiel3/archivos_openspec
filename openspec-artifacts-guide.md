# OpenSpec: Los 4 Archivos Clave

En OpenSpec, cada cambio vive en `openspec/changes/<nombre>/` y tiene **4 artefactos principales**. Cada uno responde una pregunta diferente:

```
openspec/changes/<nombre>/
├── proposal.md      ← ¿QUÉ? ¿POR QUÉ? ¿PARA QUIÉN?
├── design.md        ← ¿CÓMO? (arquitectura técnica)
├── tasks.md         ← ¿PASO A PASO? (checklist de implementación)
└── specs/           ← ¿VERDAD? (fuente de verdad, delta specs)
```

---

## 1. **proposal.md** — El "QUÉ" y el "POR QUÉ"

### Propósito
**Define el cambio en lenguaje de negocio.** Responde:
- ¿Qué estamos construyendo?
- ¿Por qué lo hacemos?
- ¿Para quién?
- ¿Cuál es el problema que resolvemos?

### Estructura típica
```markdown
# Proposal: Agregar autenticación con JWT

## Problem
Los usuarios no pueden iniciar sesión. Usamos cookies, pero no escalan en microservicios.

## Solution
Implementar JWT (JSON Web Tokens) para autenticación stateless.

## Benefits
- Funciona en arquitecturas distribuidas
- Mejor para APIs móviles
- Más fácil de testear

## Scope
- Login/logout
- Refresh tokens
- Validación en endpoints protegidos

## Out of Scope
- OAuth/SSO (para otro cambio)
```

### Audiencia
- **Product managers** (entienden el por qué)
- **Stakeholders** (ven el valor)
- **Tú mismo** (cuando vuelves 6 meses después y preguntás "¿Por qué hicimos esto?")

### Cuándo existe
✅ Desde el inicio de un cambio (lo primero que escribís)  
✅ Se actualiza si el scope cambia  
✅ **Nunca** desaparece

---

## 2. **design.md** — El "CÓMO" Técnico

### Propósito
**Define la arquitectura y decisiones técnicas.** Es el puente entre "qué queremos" y "cómo lo codificamos".

### Estructura típica
```markdown
# Design: Autenticación con JWT

## Architecture
- Cliente: almacenar JWT en localStorage
- Servidor: validar JWT en middleware
- Refresh: endpoint separado para renovar token

## Decisions
1. **Token Storage**: localStorage (no HttpOnly porque la app es SPA)
   - Pro: Acceso desde JS
   - Con: Vulnerable a XSS (mitiga con CSP)

2. **Algoritmo**: HS256 (HMAC)
   - Pro: Simple, rápido
   - Con: Secret compartido

3. **TTL**: 15 min (access), 7 días (refresh)
   - Pro: Balance seguridad/UX
   - Con: Complejidad extra en refresh

## Data Model
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "expires_in": 900
}
```

## API Changes
- POST /auth/login → { email, password }
- POST /auth/refresh → { refresh_token }
- GET /protected → Authorization: Bearer <token>

## Error Handling
- 401 si token expirado
- 403 si token inválido
- Redirect a login en el cliente
```

### Audiencia
- **Arquitectos/seniors** (revisan decisiones)
- **Implementadores** (saben exactamente qué codificar)
- **Code reviewers** (entienden por qué se hizo así)

### Cuándo existe
✅ Después de `proposal.md` (responde cómo implementar)  
✅ Se actualiza si descubrís edge cases durante `apply`  
✅ Es la referencia durante code review

---

## 3. **tasks.md** — El "PASO A PASO"

### Propósito
**Convierte el design en un checklist concreto de trabajo.** Cada tarea es:
- Específica (no vaga)
- Verificable (sabés cuándo está done)
- De tamaño razonable (1-4 horas máximo)

### Estructura típica
```markdown
# Tasks: Autenticación con JWT

## Phase 1: Backend Middleware
- [ ] Crear función `validateJWT(token)` en `src/auth/jwt.ts`
  - Verificar firma
  - Verificar expiración
  - Retornar payload
- [ ] Crear middleware `authRequired` en `src/middleware/auth.ts`
  - Extraer token de Authorization header
  - Llamar validateJWT()
  - Pasar user a req.user
- [ ] Agregar middleware a rutas protegidas en `src/routes/protected.ts`

## Phase 2: Endpoints
- [ ] POST /auth/login en `src/routes/auth.ts`
  - Validar email/password
  - Generar access + refresh tokens
  - Retornar al cliente
- [ ] POST /auth/refresh en `src/routes/auth.ts`
  - Validar refresh token
  - Generar nuevo access token
  - Retornar al cliente

## Phase 3: Frontend
- [ ] Crear hook `useAuth()` en `src/hooks/useAuth.ts`
  - Almacenar token en localStorage
  - Proveedor de contexto
- [ ] Interceptar requests en `src/http/client.ts`
  - Adjuntar token a todos los requests
- [ ] Manejar 401 → redirect a login

## Phase 4: Testing
- [ ] Tests unitarios para validateJWT()
- [ ] Tests de integración para endpoints
- [ ] Tests e2e de flow login → protected → logout
```

### Audiencia
- **Implementadores** (esto es el mapa)
- **Project managers** (ven el progreso)
- **Tú mismo** (hoy no terminas, mañana sabés dónde estabas)

### Cuándo existe
✅ Después de `design.md` (cuando ya saben QUÉ codificar)  
✅ Se actualiza durante `apply` si descubrís nuevas tareas  
✅ Finales cuando archivas el cambio

---

## 4. **specs/** — La "FUENTE DE VERDAD"

### Propósito
**Son los specs formales del sistema.** Viven en `openspec/specs/<capability>/spec.md` y documentan el comportamiento esperado permanentemente.

En cada cambio, pones **delta specs** en `openspec/changes/<nombre>/specs/` que después se mergean al canonical.

### Estructura típica
```
openspec/specs/
├── auth/
│   └── spec.md          ← La fuente de verdad sobre auth
├── users/
│   └── spec.md          ← La fuente de verdad sobre users
```

### Contenido de un spec
```markdown
# Auth Capability Spec

## JWT Token Structure
- Format: header.payload.signature
- Header: { "alg": "HS256", "typ": "JWT" }
- Payload: { "sub", "iat", "exp", "email" }
- Signed with HS256

## Login Endpoint
- Method: POST
- Path: /auth/login
- Request: { "email": string, "password": string }
- Response (200):
  ```json
  {
    "access_token": "string (JWT)",
    "refresh_token": "string (JWT)",
    "expires_in": 900 (segundos)
  }
  ```
- Response (401): Invalid credentials
- Response (400): Missing fields

## Authorization
- Clients attach token: Authorization: Bearer <token>
- Server validates en CADA request protegido
- Si inválido/expirado: 401 Unauthorized
```

### Audiencia
- **Arquitectos** (la fuente de verdad)
- **Implementadores** (qué deben cumplir)
- **QA** (qué testear)
- **Futuro tú** (documentación permanente)

### Cuándo existe
✅ Desde el inicio (se crea o se actualiza en cada cambio)  
✅ Es el "después": cuando termina un cambio, el spec se actualiza en canonical  
✅ **Nunca** desaparece — es historia del sistema

---

## Cómo Conectan Los 4 Archivos

```
       proposal.md
       (QUÉ + POR QUÉ)
            │
            ▼
       design.md
      (CÓMO técnico)
            │
            ├─────────────────┐
            ▼                 ▼
         tasks.md          specs/
    (CHECKLIST)        (FUENTE DE VERDAD)
         │                   │
         └───┬───────────────┘
             ▼
         IMPLEMENTACIÓN
      (Durante "apply")
             │
             ▼
       VERIFICACIÓN
      (Durante "verify")
             │
             ▼
         ARCHIVO
      (specs canonicales
       se actualizan)
```

---

## Flujo Real: Un Ejemplo

### Día 1: Proponer
```
User: "Necesito agregar JWT auth"
      ↓
1. Escribo proposal.md: problema, beneficio, scope
2. Escribo design.md: arquitectura, decisiones técnicas
3. Escribo tasks.md: checklist de trabajo
4. Creo specs/auth/spec.md: formato de token, endpoints
      ↓
Ready para implementar ✅
```

### Día 2-3: Implementar
```
apply-change:
  - Leo tasks.md
  - Voy tarea por tarea (Phase 1, 2, 3...)
  - Escribo código
  - Si descubro algo faltante → agrego a tasks.md
      ↓
Código listo ✅
```

### Día 4: Verificar
```
verify:
  - ¿El código cumple proposal.md? Sí ✅
  - ¿El código cumple design.md? Sí ✅
  - ¿Todas las tasks están done? Sí ✅
  - ¿Los specs están al día? Sí ✅
      ↓
Listo para producción ✅
```

### Día 5: Archivar
```
archive:
  - Sincronizo specs/ → openspec/specs/auth/spec.md (canonical)
  - Muevo change a openspec/changes/archive/2026-04-06-jwt-auth/
  - Done ✅
```

---

## Cheat Sheet: Cuándo Escribo Qué

| Artefacto | Quién | Cuándo | Propósito |
|-----------|-------|--------|----------|
| **proposal.md** | Product + Tú | Inicio | Alinearse en QUÉ y POR QUÉ |
| **design.md** | Tú (arquitecto) | Después proposal | Definir arquitectura + decisiones |
| **specs/** | Tú | Después design | Documentar formal + permanente |
| **tasks.md** | Tú | Después design | Convertir en checklist implementable |

---

## Key Point

**Cada archivo responde una pregunta diferente. NO son lo mismo.**

- `proposal.md` = "¿POR QUÉ lo hacemos?"
- `design.md` = "¿CÓMO lo implementamos?"
- `tasks.md` = "¿QUÉ tareas, EN QUÉ ORDEN?"
- `specs/` = "¿QUÉ debe CUMPLIR esto? (forever)"

Si te salteás uno, después va a costar más. No hay atajos acá, hermano.

---

Eso. ¿Se entiende? ¿Alguna pregunta?
