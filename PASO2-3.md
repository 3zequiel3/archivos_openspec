# Por Qué OPSX es Mejor Para Proyectos Reales: Flujo Iterativo + Specs en el Repo

Voy a explicarte por qué el flujo iterativo de OPSX **no es solo mejor, es la única forma que funciona** en proyectos reales. Y por qué tener las specs en el repo **cambia el juego**.

---

## Parte 1: El Flujo Iterativo

### El Viejo Modelo (Waterfall Disfrazado)

```
Fase 1: Especificación    (2 semanas de reuniones)
           │
Fase 2: Diseño            (1 semana de PowerPoints)
           │
Fase 3: Implementación    (3 semanas de código)
           │
Fase 4: Testing           (2 semanas)
           │
Fase 5: Deploy            (1 día)
```

**El problema:** A mitad de Fase 3, descubrís que la especificación era incompleta. Volvés a Fase 1. O peor: terminas algo que NO funciona y te dicen "esto no era lo que pedí".

**Por qué falla:**
- ❌ Los requisitos cambian (siempre)
- ❌ Descubrís edge cases solo cuando codificas
- ❌ Los stakeholders no entienden specs escritas
- ❌ Cuando termina, ya no es lo que el cliente necesita

### El Modelo OPSX (Iterativo)

```
Iteración 1:
  explore  → propose → apply → verify → (feedback) ↻
     │          │        │        │
     ▼          ▼        ▼        ▼
  Pensar    Plan     Código    Validar
     │                              │
     └──────────────────────────────┘
                    ↓
            ¿Está bien? NO → Editar design.md → apply de nuevo
            ¿Está bien? SÍ → archive + siguiente cambio

Iteración 2:
  (basada en feedback de Iter 1)
  explore → propose → apply → verify → (feedback) ↻
```

**Por qué funciona:**
- ✅ El feedback llega RÁPIDO (días, no meses)
- ✅ Descubrís errores temprano (costo bajo)
- ✅ El cliente VE código funcional cada iteración
- ✅ Las specs se actualizan en tiempo real
- ✅ Si algo cambió, lo sabés antes de terminar

---

## Ejemplo Real: Sistema de Pagos

### Scenario: Waterfall (Viejo Modelo)

```
Semana 1-2: Especificación
  "Necesitamos pagos con Stripe"
  Se escriben 20 páginas de requirements
  Nadie las lee bien

Semana 3-4: Diseño
  "Vamos a usar Stripe.js en frontend, backend maneja webhooks"
  Se hace un diagrama bonito
  Nadie pregunta: ¿qué pasa si el webhook llega 2 veces?

Semana 5-7: Código
  Implementas todo
  Funcionan pagos simples ✅
  A mitad de implementar webhooks, descubrís:
    - El cliente quería refunds automáticas (¿?) 
    - La API de Stripe cambió (sí, pasó)
    - Los webhooks pueden llegar out-of-order
  Volvés a Semana 3 ❌
  Ahora son 10 semanas

Semana 10-11: Testing
  Los tests fallan porque hay edge cases
  Arreglás rápido

Semana 12: Deploy
  En producción pasó algo que NO testaste
  Downtime
```

**Costo real:** 3-4 meses, sobrecostos, frustración

### Scenario: OPSX (Iterativo)

```
Iteración 1: Pagos Básicos (1 semana)
  /opsx:explore
    → ¿Qué necesita el cliente EXACTAMENTE?
    → ¿Qué es un "pago"? ¿Refund? ¿Reembolsos?
    → ¿Qué pasa si falla?

  /opsx:propose pagos-basicos
    → proposal.md: "Pagar con tarjeta, Stripe integration"
    → design.md: "Stripe.js frontend, webhook backend"
    → tasks.md: "Phase 1: Integración Stripe básica"
    → specs/payments/spec.md documentado

  /opsx:apply
    → Implementas POST /api/payments
    → Creas webhook /api/webhooks/stripe
    → Tests: pagos básicos funcionan ✅

  /opsx:verify
    → "¿Cumple la spec?" SÍ ✅

  DEPLOY A STAGING → Cliente ve código en vivo

    Feedback: "Genial, ahora necesitamos refunds"

    ↓
Iteración 2: Refunds (3 días)
  /opsx:propose pagos-refunds
    → design.md: "POST /api/payments/:id/refund"
    → tasks.md: "Agregar endpoint + webhook handling"

  /opsx:apply
    → Codigo de refund
    → Tests de edge cases
    → ✅

  DEPLOY → Cliente ve refunds funcionando

    Feedback: "¿Podemos guardar tarjetas?" (nueva feature)

    ↓
Iteración 3: Saved Cards (2 días)
  [mismo flujo]

    ↓
Iteración 4: Admin Dashboard (3 días)
  [mismo flujo]

Total: 1 + 3 + 2 + 3 = 9 días
Cliente vio código cada 2-3 días
Cambios integrados rápido
Cero sorpresas en producción
```

**Diferencia:**
| Metric | Waterfall | OPSX |
|--------|-----------|------|
| **Tiempo Total** | 3-4 meses | 9 días |
| **Cambios de requisitos** | 🔴 Catastróficos | 🟢 Un día de iteración |
| **Cliente ve código** | Día 84 | Día 2, 5, 8, 11... |
| **Edge cases encontrados** | Semana 11 (caro) | Iteración 1 (barato) |
| **Surpresas en prod** | Muchas | Casi ninguna |

---

## Parte 2: Specs en el Repositorio

### Viejo Modelo: Specs "En Algún Lado"

```
Especificación del Sistema
├── Word en OneDrive
├── PDF en Confluence
├── Diapositivas en Google Drive
├── Emails con feedback
└── Post-its en la pared

Código
├── GitHub
├── git history
└── ??? (no sabe que hay una spec nueva)
```

**El problema:**
- ❌ Spec y código se desincroniza
- ❌ Nadie sabe cuál es la "version de verdad" de la spec
- ❌ Cuando revisas código en GitHub, la spec está en otro lado
- ❌ Cambios en código sin actualizar spec
- ❌ Nuevos developers no encuentran la spec
- ❌ En 6 meses: "¿Por qué se hizo así?" — nadie recuerda

Ejemplo real:
```
Código en src/auth/jwt.ts:
  const TTL = 15 * 60 * 1000  // 15 minutos

¿POR QUÉ 15 minutos?
  → Spec está en Confluence (acceso de 2 personas)
  → Nadie actualizó Confluence cuando cambió
  → 6 meses después, developer pregunta "¿Por qué 15?"
  → Nadie sabe, lo cambias a 1 hora sin pensar
  → Breaks security en producción
```

### Modelo OPSX: Specs en el Repo (AKA "Source of Truth")

```
Repository
├── src/
│   ├── auth/
│   ├── payments/
│   └── ...
├── openspec/
│   ├── specs/
│   │   ├── auth/
│   │   │   └── spec.md          ← Fuente de verdad, junto al código
│   │   ├── payments/
│   │   │   └── spec.md
│   │   └── ...
│   └── changes/
│       ├── auth-jwt-15min/
│       │   ├── proposal.md      ← Por qué lo hacemos
│       │   ├── design.md        ← Cómo lo implementamos
│       │   ├── tasks.md         ← Checklist
│       │   └── specs/           ← Delta specs
│       └── ...
└── README.md (con link a openspec/specs)
```

**Por qué es genial:**

#### 1. **Spec + Código = Un Mismo Commit**

```bash
git log --oneline

abc1234 Implementar JWT auth con TTL 15 minutos
  → src/auth/jwt.ts (el código)
  → openspec/specs/auth/spec.md (por qué TTL=15)
  → openspec/changes/auth-jwt/design.md (la decisión técnica)

6 meses después:
  Developer: "¿Por qué TTL=15?"
  →git show abc1234
  → Ve el código Y la spec Y el design
  → Entiende al toque
```

#### 2. **Pull Request = Código + Spec Juntos**

```
PR: "Add JWT authentication"

Cambios:
  src/auth/jwt.ts        ← Código
  openspec/specs/auth/spec.md  ← Spec actualizada
  openspec/changes/.../design.md ← Decisión documentada

Reviewer ve:
  ✅ Código implementa lo que dice la spec
  ✅ Spec explica el por qué
  ✅ Design documenta tradeoffs
  → Approval fácil, confiado
```

#### 3. **Bisect + Blame Muestran La Verdad Completa**

```bash
git blame src/auth/jwt.ts | grep TTL
  abc1234 | const TTL = 15 * 60 * 1000

git show abc1234
  → Ve la spec, el design, todo
  → NO solo el código

git bisect
  → Cuando algo se rompió, ves TODAS las decisiones
    que se tomaron en ese commit
```

#### 4. **Onboarding Nuevo Developer**

```
Viejo modelo:
  "Oye, ¿dónde está la spec de auth?"
  "Creo que está en Confluence... o era en Drive?"
  "Ah, y también hay un email con cambios"
  → Developer abandona y reverting sin entender

Modelo OPSX:
  "¿Dónde está la spec de auth?"
  → openspec/specs/auth/spec.md
  → También está el design en openspec/changes/auth-jwt/design.md
  → También está el por qué en la proposal
  → Siguió el git history
  → OK, entiendo
```

#### 5. **CI/CD Puede Validar Specs**

```bash
# CI Pipeline
1. Código compila
2. Tests pasan
3. Specs están updated
   → Si editaste código, ¿actualizaste la spec?
   → Si actualizaste spec, ¿el código la cumple?
4. Design.md está sincronizado
5. Deploy
```

---

## Parte 3: Ventajas Combinadas (Iterativo + Specs en Repo)

### Ventaja 1: Feedback Loop Corto

```
Día 1: Código en staging + spec actualizada
Cliente: "Esto está bien, pero..."
Día 2: Cambio incorporado + spec actualizada
Día 3: De nuevo en staging
```

Sin "¿dónde está la spec?" — está en el repo, todo junto.

### Ventaja 2: Decisiones Documentadas (Forever)

```
Código: WHY implementamos así?
Spec:   LA RAZÓN (vinculada al commit)

6 meses después: "¿Por qué JWT no HttpOnly?"
→ git show commit
→ spec.md dice: "localStorage porque es SPA sin backend separation"
→ design.md documenta: "Trade-off: XSS risk vs acceso desde JS"
→ Entiende la razón completa, puede tomar decisión informada
```

Sin specs en repo: "Alguien lo hizo así, no sé por qué"

### Ventaja 3: Refactoring Seguro

```
Quiero cambiar auth de JWT a OAuth:

Viejo modelo:
  ❌ Editar código
  ❌ ¿Dónde está la spec?
  ❌ ¿Qué otros sistemas dependen de JWT?
  ❌ Deploy
  ❌ Sorpresa: "Esto rompió X"

OPSX:
  ✅ Leo openspec/specs/auth/spec.md
     → Veo TODO lo que JWT hace
  ✅ Busco en el repo: "grep -r JWT"
     → Veo qué depende de esto
  ✅ Propongo oauth-migration
     → design.md documenta cómo cambiar
     → specs/ muestra la diferencia
  ✅ Apply cambio iterativamente
     → Cada fase se despliega
     → Feedback inmediato
  ✅ Nada se rompe
```

### Ventaja 4: Auditoria & Compliance

```
"¿Por qué guardamos password en SHA256 y no bcrypt?"

Viejo modelo:
  Buscar en Confluence... no está
  Email del 2019... desaparecido
  Stack Overflow post... no relevante
  🤷 No sabes

OPSX:
  git log --grep="password"
  → Ves exactamente cuándo, quién, y por qué
  → Spec documenta los requerimientos de seguridad
  → design.md explica el trade-off
  → Audit trail completo
```

### Ventaja 5: Reuso de Patrones

```
"¿Cómo hizo auth otro proyecto?"

Viejo modelo:
  Buscar otro proyecto en GitHub
  Leer código
  Esperar que sea reciente
  Esperar que esté bien documentado
  😭 Normalmente no lo está

OPSX:
  otro-proyecto/openspec/specs/auth/spec.md
  → Ves exactamente cómo lo hicieron
  → Ves las decisiones
  → Ves los trade-offs
  → Copiás el patrón (o lo adaptas)
  → En 15 minutos, no 2 horas
```

---

## Comparación: Matrices de Decisión

### Cuándo Los Specs Cambian

| Situación | Waterfall | OPSX |
|-----------|-----------|------|
| Cliente pide cambio | "Espera a Phase 2" (5 sem) | Iteración 1-2 (2-3 días) |
| Descubrís edge case | Fase 4 (caro) | Fase 1-2 (barato) |
| Stakeholders piden clarificación | Mail thread caótico | openspec/specs/X/spec.md (claro) |
| Necesitás refactorizar | "¿La spec dice algo?" (busca en 5 lugares) | `cat openspec/specs/...` (1 comando) |
| Nuevo developer pregunta "¿por qué?" | Nadie sabe | `git show <commit>` (todo ahí) |

### ROI: Iterativo + Specs en Repo

| Métrica | Sin OPSX | Con OPSX |
|---------|----------|----------|
| **Tiempo a primera versión** | 3-4 meses | 1-2 semanas |
| **Tiempo a "production ready"** | 4-6 meses | 3-5 semanas |
| **Cambios después de deploy** | Frecuentes, caros | Raros, baratos |
| **Bugs en producción por "no sabía la spec"** | 5-10 por mes | 0-1 por mes |
| **Onboarding nuevo developer** | 2-3 semanas | 3-5 días |
| **Refactoring sin sorpresas** | 70% success | 95% success |
| **Documentación actualizada** | 30% de casos | 95% de casos |

---

## Parte 4: Por Qué La Mayoría Aún No Lo Hace

### Razón 1: "La Gente No Lee Specs"
**Verdad a medias.** La gente NO lee especificaciones largas en PDF. Pero SÍ lee:
- Spec en el README del repo
- Link en el PR: "Ver la spec"
- Spec en Git cuando hace blame

```bash
# La gente LEE esto:
git show abc1234 | grep -A 20 "spec.md"

# La gente NO lee:
"Especificacion_v2_final_REAL_2024.pdf" en OneDrive
```

### Razón 2: "Lleva Tiempo Escribir Specs"
**Verdad a medias.** Lleva tiempo al principio. Ahorra 10x después.

```
Tiempo invertido en specs: 4 horas
  (proposal, design, formal spec)

Tiempo ahorrado después:
  - Onboarding: -5 horas
  - Refactoring: -10 horas
  - Bugs by misunderstanding: -20 horas
  - Comunicación con stakeholders: -8 horas

ROI: 4 hours → 40+ horas ahorradas
```

### Razón 3: "Ya Tenemos Documentación"
**Sí, pero está desincronizada.**

```
Problema:
  1. Escriben spec
  2. Implementan código
  3. Código tiene bugs, lo arreglan
  4. No actualizan spec
  5. Spec es incorrecta
  6. 6 meses después: confiúsión

Solución OPSX:
  1. Escriben spec en openspec/specs/
  2. Implementan código
  3. Código tiene bugs, lo arreglan
  4. Actualizan spec en el MISMO commit
  5. Spec = código, siempre sincronizados
```

---

## El Punto Clave

**Waterfall espera "tenerlo todo claro" antes de empezar.**
No funcionan en proyectos reales porque **nunca tenés todo claro hasta que no codificas.**

**OPSX asume que las cosas van a cambiar.** 
Y hace que los cambios sean BARATOS y RÁPIDOS.

```
Waterfall:  Diseño perfectamente → Cambio = catástrofe
OPSX:       Diseño "suficiente" → Cambio = 1-2 días de iteración
```

**Y tener las specs en el repo** hace que todos estemos hablando de la MISMA VERDAD. No hay "Yo creía que la spec decía...", no hay "¿Dónde está eso?", no hay "Nadie sabe".

**Es así de simple.**

---

## Conclusión

Si comparás OPSX con Waterfall:

| Factor | Impacto |
|--------|---------|
| **Feedback rápido** | 10x más rápido |
| **Costos de cambio** | 5x más barato |
| **Bugs en prod** | 10x menos |
| **Developer happiness** | Infinitamente mejor |
| **Documentación actualizada** | Casi siempre (95%) |
| **Time to market** | 3-4x más rápido |

**¿Por qué no usarlo? Preguntale a tu PMO.**

Dale, eso es todo. ¿Questions?