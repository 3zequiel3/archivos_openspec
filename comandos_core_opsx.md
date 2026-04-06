# Comandos Core de OPSX — Explicados con Mis Palabras

OPSX tiene **4 comandos principales** que te llevan desde la idea hasta el archivo. Cada uno hace una cosa clara. Sin fases rígidas, sin puertas de entrada. Flujo líquido.

---

## 1. `/opsx:explore` — "Necesito Pensar Primero"

### Qué hace
**Te pone a pensar ANTES de comprometerte.** Es el "meeting de diseño" pero sin gente innecesaria. Investigamos, hacemos preguntas, miramos el código, descubrimos qué está pasando.

### Cuándo lo usás
- ❓ "¿Cómo resolvemos esto?"
- ❓ "No me cierra cómo abordarlo"
- ❓ "Necesito entender primero qué está roto"
- ❓ "Hay 3 formas de hacerlo, ¿cuál es mejor?"

### Qué pasa adentro
1. Nos sentamos juntos (sin código todavía)
2. Leemos el problema
3. Investigamos el código existente
4. Hacemos preguntas técnicas
5. Descubrimos constraints y oportunidades
6. Salimos con **claridad**

### Qué NO es
❌ No escribe código  
❌ No crea archivos  
❌ No es debate teórico sin evidencia  

### Qué sí es
✅ Pensamiento compartido  
✅ Basado en el código real  
✅ Salimos sabiendo QUÉ hacer  

### Ejemplo real
```
Vos: "El login está lento, ¿cómo optimizamos?"
    ↓
/opsx:explore login-optimization
    ↓
Investigamos:
  - ¿Dónde está el cuello de botella? (DB? Crypto? Red?)
  - ¿Qué traffic tenemos?
  - ¿Qué cambios rompen la seguridad?
  - ¿Hay trade-offs?
    ↓
Salimos sabiendo:
  "Usar bcrypt con workFactor=10, caché tokens en Redis, 
   refresh cada 1 hora"
    ↓
Ahora sí, podés usar /opsx:propose
```

---

## 2. `/opsx:propose` — "Voy a Hacer Esto"

### Qué hace
**Convierte tu idea en un PLAN COMPLETO.** Escribís qué querés en lenguaje natural. Yo genero los 4 artefactos listos para trabajar:
- `proposal.md` (QUÉ + POR QUÉ)
- `design.md` (CÓMO técnico)
- `tasks.md` (PASO A PASO)
- `specs/` (FUENTE DE VERDAD)

### Cuándo lo usás
- ✅ "Quiero agregar autenticación con JWT"
- ✅ "Necesito refactorizar el store de Redux"
- ✅ "Voy a crear una nueva tabla de usuarios"
- ✅ Después de `/opsx:explore` (ya sabés qué hacer)

### Qué pasa adentro
1. Me describís la idea (1-2 párrafos alcanza)
2. Yo analizo el codebase
3. Genero `proposal.md`: problema, beneficio, scope
4. Genero `design.md`: arquitectura, decisiones técnicas
5. Genero `tasks.md`: checklist de trabajo (Phase 1, 2, 3...)
6. Genero `specs/`: especificación formal

### Qué NO es
❌ No escribe el código implementado  
❌ No compila ni testa  

### Qué sí es
✅ El MAPA completo antes de trabajar  
✅ Todos los artefactos listos para `/opsx:apply`  
✅ Decisiones técnicas explicadas  

### Ejemplo real
```
Vos: "Necesito agregar un dashboard de analytics"
    ↓
/opsx:propose analytics-dashboard
    ↓
Yo genero:
  proposal.md →
    "Problem: Users no ven sus métricas
     Solution: Dashboard con gráficos interactivos
     Scope: Usuarios premium, últimos 30 días"

  design.md →
    "Frontend: React + Recharts
     Backend: API /api/analytics con agregación
     DB: Query materializada en tabla analytics_summary
     Decision: Server-side paging, no todo en memoria"

  tasks.md →
    "Phase 1: Backend
      - [ ] Crear tabla analytics_summary
      - [ ] Crear stored procedure de agregación
      - [ ] Crear endpoint GET /api/analytics
    Phase 2: Frontend
      - [ ] Componente DashboardView
      - [ ] Gráficos con Recharts
      - [ ] Filtro por rango de fechas"

  specs/analytics/spec.md →
    "Endpoint GET /api/analytics
     Query params: start_date, end_date, metric
     Response: { data: [...], total_rows: N }
     Error 400 si rango > 90 días"
    ↓
Ready para /opsx:apply ✅
```

---

## 3. `/opsx:apply` — "Dale, Implementá"

### Qué hace
**Toma las `tasks.md` y ESCRIBE EL CÓDIGO.** Fase a fase, tarea a tarea. Yo implemento, explico cada decisión, y vos sos el copiloto.

### Cuándo lo usás
- ✅ "Hacé el código" (después de `/opsx:propose`)
- ✅ "Continúa donde paramos"
- ✅ Necesitás el código ya
- ✅ `tasks.md` está listo

### Qué pasa adentro
1. Leo `design.md` para entender la arquitectura
2. Leo `tasks.md` y empiezo Phase 1
3. Escribo código, archivo por archivo
4. Explico cada decisión
5. Agrego tests (unitarios, integración, e2e)
6. Si descubro tareas faltantes → las agrego a `tasks.md`
7. Repito hasta que todas las tasks están ✅

### Qué NO es
❌ No propone, no diseña  
❌ No te deja sin entender qué pasó  
❌ No salta tareas  

### Qué sí es
✅ CÓDIGO REAL, file by file  
✅ Cada cambio explicado  
✅ Tests incluidos  
✅ Verificamos contra `design.md`  

### Ejemplo real
```
/opsx:apply analytics-dashboard
    ↓
Phase 1: Backend
  - Creo src/db/migrations/001_create_analytics_summary.sql
  - Creo src/db/procedures/aggregate_analytics.sql
  - Creo src/api/routes/analytics.ts
    - Valido query params
    - Ejecuto query
    - Retorno con paginación
  - Creo src/api/routes/analytics.test.ts
    - Test: 200 OK con datos válidos
    - Test: 400 si rango > 90 días
    - Test: Paginación funciona
    ↓
Phase 1: ✅

Phase 2: Frontend
  - Creo src/components/DashboardView.tsx
    - Hook useAnalytics() que llama endpoint
    - Manejo de loading/error
  - Creo src/components/AnalyticsChart.tsx
    - Gráfico con Recharts
    - Formateo de datos
  - Creo src/hooks/useAnalytics.ts
    - Fetch desde endpoint
    - Caching con React Query
  - Tests de componentes
    ↓
Phase 2: ✅

Código listo, tests pasan ✅
```

---

## 4. `/opsx:archive` — "Terminamos, Cerrá Esto"

### Qué hace
**Finaliza el cambio y lo mueve al historial.** Sincroniza los `specs/` con los specs canonicales, archiva todo, y marca como "DONE".

### Cuándo lo usás
- ✅ "Terminamos, archivá"
- ✅ Después de `/opsx:apply`
- ✅ Código testeado, todo funciona
- ✅ Listo para producción

### Qué pasa adentro
1. Valida que todas las `tasks.md` estén ✅
2. Sincroniza `specs/` → `openspec/specs/<capability>/spec.md` (canonical)
3. Copia el cambio a `openspec/changes/archive/YYYY-MM-DD-<nombre>/`
4. Cierra el cambio (ya no está en "active")
5. Genera resumen de qué cambió

### Qué NO es
❌ No sigue aceptando cambios  
❌ No permite re-aplicar tareas  

### Qué sí es
✅ Documentación final + actualizada  
✅ Historial permanente  
✅ Specs canonicales al día  
✅ "DONE" de verdad  

### Ejemplo real
```
/opsx:archive analytics-dashboard
    ↓
1. Valida tasks.md
   "Phase 1: ✅ ✅ ✅
    Phase 2: ✅ ✅ ✅
    Todo done"
    ↓
2. Sincroniza specs/
   "openspec/specs/analytics/spec.md actualizado
    - Endpoint GET /api/analytics
    - Respuesta con paginación
    - Errores documentados"
    ↓
3. Archiva el cambio
   "Movido a:
    openspec/changes/archive/2026-04-06-analytics-dashboard/"
    ↓
4. Resumen final
   "Cambio completado:
    - 3 archivos backend
    - 3 archivos frontend
    - 8 tests pasando
    - Specs actualizados"
    ↓
DONE ✅ Listo para producción
```

---

## El Flujo Completo (Real)

```
TÚ: "Necesito agregar autenticación"
    │
    ├─ ¿Necesitás explorar primero?
    │  SI  → /opsx:explore
    │       (investigamos, pensamos)
    │       → Salimos con claridad
    │
    └─ NO → /opsx:propose
            (genero plan completo)
            → proposal.md + design.md + tasks.md + specs/
            │
            ▼
        /opsx:apply
        (escribo el código)
        → Fase a fase, task a task
        → Tests incluidos
        │
        ▼
        /opsx:archive
        (cierro el cambio)
        → Specs actualizados
        → Listo para producción
        │
        ▼
        ✅ DONE
```

---

## Cheat Sheet: Cuál Comando Usar

| Situación | Comando | Resultado |
|-----------|---------|-----------|
| "¿Cómo resolvemos esto?" | `/opsx:explore` | Claridad, decisiones técnicas |
| "Tengo una idea, hacé el plan" | `/opsx:propose` | 4 artefactos listos |
| "Dale, a codificar" | `/opsx:apply` | Código + tests |
| "Terminamos, archivá" | `/opsx:archive` | Specs actualizados, DONE |
| "Necesito cambiar algo del plan" | Edita `design.md` o `tasks.md` | Re-applica con cambios |
| "Descubrí un bug en prod" | `/opsx:propose` (nuevo) | Nuevo cambio para bugfix |

---

## Key Rules

1. **No podés hacer `/opsx:apply` sin `/opsx:propose`** — primero el plan, después el código
2. **Podés explorar sin proponer** — `/opsx:explore` → `/opsx:archive` es "nada" (solo pensaste)
3. **Podés volver atrás** — edita `design.md`, corre `/opsx:apply` de nuevo
4. **Podés proposer múltiples cambios en paralelo** — cada uno es independiente
5. **Archivá cuando esté DONE** — no es "cuando estés cansado"

---

## La Magia de OPSX

No hay fases rígidas. No hay "espera a que termine Phase 1". **Vos mandás.**

- ¿Necesitás pensar? → `/opsx:explore`
- ¿Ya sabés qué hacer? → `/opsx:propose` directo
- ¿Necesitás cambiar el plan? → Editá `design.md` y re-aplicá
- ¿Descubriste algo a mitad de implementación? → Agrega a `tasks.md`

**OpenSpec es una herramienta. Vos sos quien la dirige.**

---

Eso. ¿Se entiende? ¿Cuál comando necesitás ahora?
