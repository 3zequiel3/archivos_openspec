# PASO3-1: Diagrama de Flujo — Chatbot vs Agente con MCPs

## El Problema: Chatbot Solo vs Agente Inteligente

Cuando usás un chatbot sin herramientas (LLM puro), estás limitado a generar texto. No puede leer archivos, ejecutar código, acceder a GitHub, nada. Es como un teléfono donde solo podés HABLAR, no hacer nada.

Un **Agente con MCPs** (Model Context Protocol) es diferente. Puede ACTUAR. Puede leer, escribir, ejecutar, conectarse a sistemas. Es como tener manos además de boca.

---

## Diagrama 1: Chatbot Solo (Sin Herramientas)

```
┌─────────────────────────────────────────────────────────────┐
│                     USUARIO                                 │
│          "¿Qué hay en este archivo?"                        │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │   CHATBOT (LLM Solo)  │
         │                       │
         │  "No puedo leer el    │
         │   archivo, pero te    │
         │   cuento historias"   │
         │                       │
         └───────────────────────┘
                     │
                     ▼
    ┌────────────────────────────────────┐
    │  RESPUESTA: Adivinanza / Teoría    │
    │                                    │
    │  "Probablemente tiene código      │
    │   JavaScript que..."              │
    │                                    │
    │  (No SABE, está adivinando)       │
    └────────────────────────────────────┘

PROBLEMA:
  ❌ No puede leer archivos reales
  ❌ No sabe qué hay EN el código
  ❌ Respuestas son adivinanzas
  ❌ No puede ejecutar código
  ❌ No puede conectarse a sistemas externos
```

---

## Diagrama 2: Agente con MCPs (Con Herramientas)

```
┌─────────────────────────────────────────────────────────────┐
│                     USUARIO                                 │
│          "¿Qué hay en este archivo?"                        │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
     ┌───────────────────────────────────┐
     │   AGENTE (LLM + MCPs)             │
     │                                   │
     │  "Necesito leer ese archivo"      │
     │   → FileSystem MCP                │
     │                                   │
     │  "Necesito ver el repo"           │
     │   → GitHub MCP                    │
     │                                   │
     │  "Necesito ejecutar esto"         │
     │   → Bash/Terminal MCP             │
     └───────────────┬───────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌──────────┐
    │FileSystem│  │ GitHub  │  │ Bash/Cli │
    │   MCP    │  │  MCP    │  │   MCP    │
    │          │  │         │  │          │
    │Lee/Escr. │  │Commits, │  │Ejecuta  │
    │archivos  │  │PRs, code│  │comandos │
    └────┬─────┘  └────┬────┘  └────┬────┘
         │             │            │
         │             │            │
    ┌────▼─────────────▼────────────▼──────┐
    │         MUNDO REAL                   │
    │                                      │
    │  Archivos del disco                 │
    │  Repositorio GitHub                │
    │  Terminal del sistema              │
    │  Bases de datos                    │
    │  APIs externas                     │
    └──────────────────────────────────────┘
         │             │            │
         │ (respuestas │ reales)    │
         │             │            │
    ┌────▼─────────────▼────────────▼──────┐
    │                                      │
    │  RESPUESTA: VERDADERA                │
    │                                      │
    │  "El archivo tiene 245 líneas       │
    │   de TypeScript, importa React      │
    │   y tiene 3 componentes:            │
    │   - Button                          │
    │   - Card                            │
    │   - Modal"                          │
    │                                      │
    │  (SABE porque lo LEY)               │
    └──────────────────────────────────────┘

VENTAJA:
  ✅ Lee archivos REALES
  ✅ Ve exactamente qué hay EN el código
  ✅ Respuestas 100% precisas
  ✅ Puede ejecutar código
  ✅ Puede conectarse a cualquier sistema
  ✅ Puede hacer acciones (crear archivos, commits, etc.)
```

---

## Diagrama 3: Flujo Completo de un Agente (OPSX)

```
┌──────────────────────────────────────────────────────────────┐
│                    USUARIO                                  │
│     "Quiero agregar autenticación con JWT"                 │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ▼
     ┌───────────────────────────────────────┐
     │  AGENTE CON MCPs (OPSX Orchestrator)  │
     │                                       │
     │  "Necesito entender el proyecto"      │
     └───────────────┬───────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
   ┌─────────┐  ┌─────────┐  ┌──────────┐
   │ File    │  │ GitHub  │  │  Bash    │
   │ System  │  │  MCP    │  │   MCP    │
   └────┬────┘  └────┬────┘  └────┬─────┘
        │            │            │
        │            │            │
    Lee archivos  Lee commits   Corre tests
    existentes    + issues      + npm
        │            │            │
        └────────────┼────────────┘
                     ▼
     ┌──────────────────────────────────────┐
     │  AGENTE ENTIENDE EL PROYECTO         │
     │                                      │
     │  "OK, veo que usan Express,          │
     │   TypeScript, JWT en otro lado,      │
     │   tests con Jest"                    │
     └────────────┬───────────────────────────┘
                  │
                  ▼
     ┌──────────────────────────────────────┐
     │  /opsx:explore (si es necesario)    │
     │  o directo a /opsx:propose           │
     │                                      │
     │  Genera:                             │
     │  - proposal.md (QUÉ + POR QUÉ)      │
     │  - design.md (CÓMO)                  │
     │  - tasks.md (CHECKLIST)              │
     │  - specs/auth/spec.md                │
     └────────────┬───────────────────────────┘
                  │
                  ▼
     ┌──────────────────────────────────────┐
     │  /opsx:apply (IMPLEMENTACIÓN)        │
     │                                      │
     │  Usa FileSystem MCP para:            │
     │    - Crear src/auth/jwt.ts           │
     │    - Crear src/middleware/auth.ts    │
     │    - Actualizar src/index.ts         │
     │                                      │
     │  Usa Bash MCP para:                  │
     │    - npm install jsonwebtoken        │
     │    - npm run test                    │
     │    - npm run build                   │
     │                                      │
     │  Genera código + tests               │
     └────────────┬───────────────────────────┘
                  │
                  ▼
     ┌──────────────────────────────────────┐
     │  /opsx:verify (VALIDACIÓN)           │
     │                                      │
     │  Lee specs y verifica:               │
     │    ✅ Código cumple spec             │
     │    ✅ Tests pasan                    │
     │    ✅ Design se respetó              │
     └────────────┬───────────────────────────┘
                  │
                  ▼
     ┌──────────────────────────────────────┐
     │  /opsx:archive (CIERRE)              │
     │                                      │
     │  Usa GitHub MCP para:                │
     │    - Crear PR                        │
     │    - Actualizar specs en repo        │
     │                                      │
     │  Usa FileSystem MCP para:            │
     │    - Mover a archive/                │
     │    - Limpiar artifacts               │
     │                                      │
     │  ✅ DONE                             │
     └──────────────────────────────────────┘

TODO INTEGRADO:
  ✅ Lee el código actual
  ✅ Entiende arquitectura existente
  ✅ Genera plan (proposal + design + tasks)
  ✅ Escribe código nuevo
  ✅ Ejecuta tests
  ✅ Valida contra specs
  ✅ Maneja git/GitHub
  ✅ Cierra el cambio
```

---

## Diagrama 4: Comparación Side-by-Side

```
╔════════════════════════════════════╦════════════════════════════════════╗
║      CHATBOT SOLO (Sin MCPs)       ║    AGENTE CON MCPs (Conectado)     ║
╠════════════════════════════════════╬════════════════════════════════════╣
║                                    ║                                    ║
║  Input:  Pregunta en texto         ║  Input:  Pregunta en texto         ║
║          │                         ║          │                         ║
║          ▼                         ║          ▼                         ║
║  LLM:    Procesa + Genera texto    ║  Agent:  Planifica acciones        ║
║                                    ║          │                         ║
║          ▼                         ║          ├─ FileSystem MCP         ║
║  Output: Solo texto               ║          ├─ GitHub MCP             ║
║          (adivinanza)              ║          ├─ Bash MCP               ║
║                                    ║          └─ ...más                 ║
║  LÍMITES:                          ║          ▼                         ║
║    ❌ No lee archivos              ║  Exec:   Ejecuta acciones reales  ║
║    ❌ No conecta a sistemas        ║          ▼                         ║
║    ❌ No ejecuta código            ║  Output: Resultado real             ║
║    ❌ Solo genera hipótesis        ║          (README actualizado,      ║
║    ❌ Baja precisión               ║           código escrito,          ║
║    ❌ No puede iterar              ║           tests pasando)           ║
║                                    ║                                    ║
║  CASO DE USO:                      ║  VENTAJAS:                         ║
║    - Explicaciones                 ║    ✅ Lee archivos reales          ║
║    - Brainstorming                 ║    ✅ Conecta a sistemas          ║
║    - Preguntas generales           ║    ✅ Ejecuta código               ║
║                                    ║    ✅ Alta precisión               ║
║                                    ║    ✅ Puede iterar                 ║
║                                    ║    ✅ Automatiza workflows         ║
║                                    ║    ✅ Mantiene estado              ║
║                                    ║                                    ║
║                                    ║  CASO DE USO:                      ║
║                                    ║    - Implementar features          ║
║                                    ║    - Escribir código real          ║
║                                    ║    - Administrar repos             ║
║                                    ║    - Orquestar workflows           ║
║                                    ║    - Hacer cambios concretos       ║
║                                    ║                                    ║
╚════════════════════════════════════╩════════════════════════════════════╝
```

---

## Diagrama 5: MCPs Disponibles (Ejemplos)

```
┌────────────────────────────────────────────────────────────┐
│                  AGENTE                                   │
│  (Claude + herramientas de verdad)                       │
└────────────────────────────────────────────────────────────┘
            │
            ├─────────┬──────────┬──────────┬────────────┐
            ▼         ▼          ▼          ▼            ▼
        ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
        │FileS.  │ │GitHub  │ │ Bash   │ │  HTTP  │ │ Notion │
        │  MCP   │ │  MCP   │ │  MCP   │ │  MCP   │ │  MCP   │
        │        │ │        │ │        │ │        │ │        │
        │ Read   │ │Commits │ │Execute │ │ Call   │ │ Read   │
        │ Write  │ │ PRs    │ │Scripts │ │ APIs   │ │ Write  │
        │ Rename │ │Issues  │ │ Tests  │ │        │ │ Update │
        └────────┘ └────────┘ └────────┘ └────────┘ └────────┘
            │         │          │          │            │
            ▼         ▼          ▼          ▼            ▼
        Archivos    Repos      Terminal   APIs    Project Mgmt
        del PC      GitHub     del Sistema Externas  Datos

Cada MCP es como una "extensión" que da al agente nuevas MANOS.
```

---

## Diagrama 6: Flujo de Decisión

```
┌──────────────────────────────┐
│  NECESITO HACER ALGO REAL    │
│  (cambiar código, crear PR,  │
│   ejecutar tests, etc.)      │
└──────────────┬───────────────┘
               │
               ▼
        ¿Es solo texto/info?
         /         \
       NO/           \YES
       /               \
      ▼                 ▼
   ┌─────────┐    ┌──────────────┐
   │ Chatbot │    │ Chatbot o    │
   │  SOLO   │    │ Agente       │
   │         │    │ (ambos van)  │
   │ ❌      │    │ ✅           │
   │ No puede│    │ Puede        │
   │ hacer   │    │ responder    │
   │ nada    │    │ bien         │
   └─────────┘    └──────────────┘
      ↓
     SAD           
   (Necesitás      
    otro sistema)  

               ¿Necesitas conectar con sistemas reales?
                      /           \
                    NO/             \YES
                    /                 \
                   ▼                   ▼
            ┌──────────────┐    ┌──────────────────┐
            │ Chatbot está │    │  AGENTE CON MCPs │
            │ bien         │    │  ✅              │
            │              │    │  Lee archivos    │
            └──────────────┘    │  Conecta repos   │
                                 │  Ejecuta código  │
                                 │  Hace cambios    │
                                 │  Automatiza      │
                                 └──────────────────┘
                                        ↓
                                      ✅ HACE EL TRABAJO
```

---

## Resumen Ejecutivo

### Chatbot Solo
- **Es:** Un LLM con boca (genera texto)
- **Hace:** Explica, aconseja, teoriza
- **NO hace:** Acciones concretas

### Agente con MCPs
- **Es:** Un LLM con manos (lee, escribe, ejecuta)
- **Hace:** Implementa features, maneja repos, automatiza workflows
- **Puede:** Conectarse a cualquier sistema

### OPSX en Particular
- **Usa:** Múltiples MCPs (FileSystem, GitHub, Bash, HTTP, etc.)
- **Hace:** Orquestra el flujo completo (explore → propose → apply → archive)
- **Resultado:** Features implementadas + tests + documentación + specs, todo integrado

---

## La Analogía Final

```
CHATBOT SOLO:
  Es como un colega que solo te PUEDE HABLAR por teléfono.
  Te da consejos, ideas, explicaciones.
  Pero no puede HACER nada en el mundo real.

AGENTE CON MCPs:
  Es como un colega que no solo te HABLA, sino que:
    - Lee los archivos (FileSystem)
    - Ve el repo (GitHub)
    - Ejecuta código (Bash)
    - Llama APIs (HTTP)
    - Gestiona tareas (Notion, etc.)
  
  Puede ACTUAR en el mundo real.
```

---

Eso. Ahora entendés la diferencia. 

**Pregunta:** ¿Querés que profundice en algún MCP específico, o pasamos a otra cosa?
