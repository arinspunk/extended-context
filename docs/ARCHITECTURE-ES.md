# Arquitectura: Contexto Extendido

Este documento explica las decisiones de diseño, fundamentos conceptuales y evolución del sistema desde v1 hasta la versión actual.

## Tabla de Contenidos
- [El Problema Fundamental](#el-problema-fundamental)
- [Conceptos Core: Prompt, Contexto, Memoria](#conceptos-core)
- [Evolución: De v1 a v2](#evolución-de-v1-a-v2)
- [Arquitectura Conocimiento/Memoria](#arquitectura-conocimientomemoria)
- [Decisiones de Diseño](#decisiones-de-diseño)
- [Limitaciones y Trade-offs](#limitaciones-y-trade-offs)
- [Aplicabilidad Universal](#aplicabilidad-universal)

---

## El Problema Fundamental

### La Ilusión del Prompt Perfecto

Los agentes de IA autónomos (Cursor, GitHub Copilot, Claude Code) crearon una expectativa ilusoria: "si escribo el prompt perfecto, el agente generará toda la feature de una tirada".

**La realidad:**
- Los LLMs tienen ventana de contexto limitada (~200K tokens)
- Sin estructura, cada sesión comienza desde cero
- El agente "olvida" decisiones previas
- Imposible rastrear por qué se rechazó arquitectura X hace dos semanas

### Vibe-Coding: Prototipado vs Producción

**Vibe-coding** (confiar en que el agente "entienda la vibe" del proyecto) funciona para prototipos rápidos, pero genera problemas en proyectos reales:

1. **Decisiones perdidas**: ¿Por qué elegimos esta arquitectura hace dos semanas?
2. **Alucinaciones constantes**: La IA inventa APIs, mezcla frameworks, ignora constraints
3. **Trabajo monolítico**: Imposible dividir features en tareas rastreables
4. **Improvisación continua**: Cada sesión empieza desde cero

**Investigación reciente** muestra que [los LLMs tienden a generar código más complejo de lo necesario](https://arxiv.org/html/2411.09916v1), reforzando la necesidad de constraints estructurados.

---

## Conceptos Core

Antes de explicar la arquitectura, necesitamos establecer tres conceptos fundamentales:

### 1. Prompt
**Definición:** La instrucción que el usuario envía al LLM en un momento específico.

**Ejemplo:**
```
"Implementa facturación automática WooCommerce"
```

**Características:**
- Efímero (solo existe en esa conversación)
- Depende del contexto disponible
- Sin memoria entre sesiones

### 2. Contexto
**Definición:** Toda la información que el LLM puede "ver" al procesar un prompt.

**Composición del contexto:**
```
Contexto Total = System Prompt + Conversación actual + Archivos adjuntos + Rules activas
```

**Ejemplo en Cursor:**
```
System Prompt:    "Eres un experto en WordPress..."
Conversación:     [Últimos 10 mensajes]
Archivos:         checkout.php, functions.php
Rules activas:    wordpress-expert.mdc, kiss.mdc, constraints-proyecto.mdc
```

**Límite físico:** ~200K tokens en modelos actuales (GPT-4, Claude Sonnet)

### 3. Memoria
**Definición:** Información persistente que sobrevive entre sesiones.

**Tipos de memoria en desarrollo asistido por IA:**
- **Memoria de corto plazo**: Conversación actual (gestionada automáticamente por la herramienta)
- **Memoria de medio plazo**: Decisiones de esta feature/sprint (manual en la mayoría de herramientas)
- **Memoria de largo plazo**: Arquitectura del proyecto, patrones establecidos (inexistente sin estructura)

**El problema:** La mayoría de herramientas solo tienen memoria de corto plazo. Contexto Extendido implementa las tres.

---

## Evolución: De v1 a la Versión Actual

### Sistema v1: Workflow Estructurado

**Propuesta original:** [Guía para desarrollar software con agentes](https://xulioze.com/blog/guia-desarrollar-software-agentes.html)

**Estructura:**
```
context/
├── 01-expert.md      # Perfil del experto
├── 02-analysis.md    # Análisis del código
├── 03-plan.md        # Plan de implementación
└── 04-backlog.md     # Tareas pendientes
```

**Workflow:**
```
Análisis → Solución → Backlog → Ejecución → Commit → Continuar
```

**Logro principal:** Convirtió vibe-coding en ingeniería rastreable.

### Limitaciones de v1

**1. Conocimiento mezclado**
`01-expert.md` combina principios generales (KISS, DRY) con constraints específicos del proyecto (stack, versiones):

```markdown
# 01-expert.md
## Principios
- KISS, DRY, SOLID
- Single Responsibility

## Stack Proyecto
- WordPress 6.3
- ACF PRO 6.2.1
- WooCommerce 8.0
```

**Problema:** Imposible reutilizar principios entre proyectos sin arrastrar configuración irrelevante.

**2. Duplicación entre proyectos**
Si trabajas en 3 proyectos WordPress, tienes "security: sanitize_*, nonces" copiado tres veces. Actualizas una best practice → sincronización manual en 3 lugares.

**3. Sin gestión de histórico**
v1 te da tres opciones, todas malas:
- Sobrescribir `02-analysis.md` → pierdes decisiones previas
- Acumular análisis en el mismo archivo → crece descontroladamente
- Crear múltiples archivos → desorden sin nomenclatura consistente

**4. Sin modularidad**
Cambiar de WordPress a React requiere reescribir `01-expert.md` completo. No puedes mantener módulos comunes (principios, patrones) y solo intercambiar el experto técnico.

**5. Ejecución manual**
Cada acción requería escribir el prompt completo:
```
"Analiza este archivo siguiendo la estructura de 01-expert.md y guarda en 02-analysis.md"
"Lee 04-backlog.md, ejecuta tareas 1-3, actualiza estado"
```

### Versión Actual: Separación Conocimiento/Memoria

La versión actual resuelve los 5 problemas mediante la separación conceptual fundamental:

**Conocimiento** (reutilizable, persiste entre proyectos) vs **Memoria** (temporal, específica del proyecto/feature)

**Implementación en Cursor:**
- `rules/` = mecanismo para inyectar conocimiento
- `memory/` = directorio para almacenar memoria
- `@` commands = sintaxis para invocar protocolos

**Problemas resueltos:**

1. **Conocimiento modular** → `rules/experts/`, `rules/guidelines/` separados
2. **Reutilización** → Un experto WordPress para N proyectos
3. **Histórico cronológico** → `memory/YYYYMMDD-VV-descripcion.md`
4. **Componibilidad** → Intercambia expertos, mantén guidelines
5. **Automatización** → Protocolos invocables (`@analysis`, `@backlog`)

**Nota importante:** Aunque esta guía usa Cursor como ejemplo, la separación Conocimiento/Memoria es un patrón universal aplicable a cualquier herramienta de IA.

---

## Arquitectura Conocimiento/Memoria

### Modelo Mental: Tu Cerebro

La arquitectura replica cómo funciona tu cerebro:

**Conocimiento permanente:**
- Idiomas que hablas
- Frameworks que dominas
- Principios de arquitectura

**Memoria episódica:**
- Lo que hiciste ayer
- Decisiones de la semana pasada
- Bugs que resolviste este mes

Pero además, tu conocimiento **no es monolítico**: tienes módulos especializados (JavaScript, WordPress, KISS) que activas según el contexto.

### Implementación en Cursor

Cursor implementa esta separación mediante `rules/` y `memory/`:

```
.cursor/
├── rules/          # Conocimiento reutilizable y modular
│   ├── experts/    # Roles técnicos (WordPress, React, Python...)
│   ├── guidelines/ # Principios y constraints (KISS, proyecto-específicos)
│   └── utils/      # Protocolos ejecutables (analysis, backlog, commit)
└── memory/         # Experiencias temporales
    └── YYYYMMDD-VV-descripcion.md
```

**Conceptos clave:**
- `rules/` = Vehículo para inyectar **conocimiento** en el contexto
- `memory/` = Almacenamiento de **memoria** del proyecto
- Esta estructura es específica de Cursor, pero el patrón Conocimiento/Memoria es universal

### Rules: Conocimiento Reutilizable

Los `rules/` en Cursor son el mecanismo para inyectar **conocimiento** estructurado. En Contexto Extendido, organizamos las rules en tres categorías:

**Experts** - Perfiles técnicos/dominio:

```markdown
# rules/experts/wordpress-classic-themes.mdc
---
alwaysApply: true
---
# WordPress Classic Expert
PHP + vanilla JS specialist. WordPress core APIs, theme dev.

## Stack
Backend: PHP 7.4+, WP Core APIs
Frontend: Vanilla JS, HTML5, CSS3

## Core Patterns
- Security: sanitize_*, esc_*, nonces
- Performance: transients (12h), conditional enqueue
- Translation: __(), _e() with text domain
```

**Guidelines** - Principios y constraints:

```markdown
# rules/guidelines/kiss.mdc
---
alwaysApply: true
---
# KISS Principle
- Direct solutions, avoid over-engineering
- Simple patterns, minimal dependencies
- Single responsibility per function

Rule: If you can't explain it in 2-3 sentences, it's too complex.
```

```markdown
# rules/guidelines/constraints-proyecto.mdc
---
alwaysApply: true
---
# Proyecto Balneario Theme
Text domain: themefront v0.1.2
Stack: Node | SASS 1.67.0 | PHP 7.4+ | jQuery 3.x
Build: npm run watch:sass (never edit compiled CSS)

Critical: ACF PRO (18 groups), WooCommerce, WPML
```

**Utils** - Protocolos invocables:

```markdown
# rules/utils/analysis.mdc
---
alwaysApply: false
---
# Analysis Protocol
Trigger: @file/@folder or "analyze"

Steps:
1. Acknowledge target
2. Analyze structure, patterns, dependencies
3. Save to .cursor/memory/YYYYMMDD-VV-analisis-*.md
4. Confirm completion
```

### Memory: Registro Histórico

Los protocolos (`rules/utils/`) generan automáticamente archivos en `memory/` siguiendo la nomenclatura `YYYYMMDD-VV-descripcion.md`:

```
memory/
├── 20250919-01-analisis-pdf-generator.md
├── 20250919-02-sistema-pdfs-emails-finalizado.md
├── 20251015-01-analisis-completo-balneario-theme.md
├── 20251015-02-solucion-facturacion-automatica-399.md
└── 20251015-03-backlog-facturacion-automatica-399.md
```

**Ventajas nomenclatura cronológica:**
- Ordenación natural por fecha
- Múltiples entradas por día (VV secuencia: 01, 02, 03...)
- Trazabilidad inmediata
- Relaciones visibles por proximidad temporal

**Clave sobre contexto:**
Los archivos en `memory/` almacenan la **memoria** del proyecto. Son **referencias en disco** que solo ocupan contexto cuando:
- El usuario los invoca: `@memory/20251015-03-backlog.md`
- Un protocolo los lee/escribe automáticamente

**Implicación práctica:** Puedes acumular cientos de archivos memory sin impacto en rendimiento. El historial completo persiste para consulta futura sin penalización de contexto.

---

## Decisiones de Diseño

### 1. Por qué `alwaysApply`

**Problema:** Con 50+ rules disponibles, ¿cuáles carga Cursor automáticamente en cada conversación?

**Solución:** Flag `alwaysApply` para control granular de qué ocupa contexto:

```markdown
---
alwaysApply: true   # Carga automática → consume contexto siempre
---
```

```markdown
---
alwaysApply: false  # Invocación bajo demanda → solo consume contexto cuando lo invocas
---
```

**Criterio de decisión:**
- `true` → Conocimiento que SIEMPRE quieres activo (experto base, principios KISS)
- `false` → Herramientas específicas que usas según necesidad (protocolos @analysis, @backlog)

**Límite práctico:** ~5-10 rules con `alwaysApply: true` para no saturar contexto.

**Nota importante:** Los archivos `memory/` **no** usan `alwaysApply` porque nunca se cargan automáticamente. Solo ocupan contexto cuando los invocas explícitamente (`@memory/archivo.md`) o cuando un protocolo los lee.

### 2. Por qué Nomenclatura YYYYMMDD-VV

**Alternativas consideradas:**

**Opción A - Descriptiva:**
```
analysis-pdf-generator.md
solution-refactor-pdfs.md
```
❌ Sin orden temporal, difícil rastrear cuándo se tomó cada decisión

**Opción B - Incremental:**
```
001-analysis.md
002-solution.md
```
❌ Números sin significado, imposible asociar a fecha

**Opción C - ISO 8601 completa:**
```
20251015T143022-analysis.md
```
❌ Precisión innecesaria, dificulta lectura humana

**Opción elegida - Cronológica con secuencia diaria:**
```
20251015-01-analisis-pdf-generator.md
20251015-02-solucion-refactor-pdfs.md
```
✅ Balance perfecto: orden cronológico + legibilidad + múltiples entradas/día

### 3. Por qué Separar Experts/Guidelines

**Alternativa rechazada:** Un solo archivo `rules/project-config.mdc`

**Problema:** Mezclar experto técnico con constraints del proyecto impide reutilización.

**Ejemplo:**

```markdown
# ❌ Monolítico (no reutilizable)
rules/wordpress-balneario.mdc
- Experto WordPress
- Principios KISS
- Constraints Balneario

# ✅ Modular (reutilizable)
rules/experts/wordpress-classic-themes.mdc    # Reutilizable en N proyectos
rules/guidelines/kiss.mdc                     # Reutilizable en cualquier stack
rules/guidelines/constraints-balneario.mdc    # Específico del proyecto
```

**Beneficio:** Trabajas en 5 proyectos WordPress → 1 experto, 1 KISS, 5 constraints.

### 4. Por qué Protocolos en utils/

**Alternativa rechazada:** Escribir prompts manualmente cada vez.

**Problema en v1:**
```
"Analiza includes/pdf-generator.php siguiendo estructura 01-expert.md, 
documenta dependencias y hooks críticos, guarda en 02-analysis.md con 
formato consistente..."
```

**Solución v2:**
```
@analysis includes/pdf-generator.php
```

**Implementación:**
```markdown
# rules/utils/analysis.mdc
---
alwaysApply: false
---
Trigger: @file/@folder or "analyze"
1. Acknowledge → 2. Analyze → 3. Save to memory/ → 4. Confirm
```

**Beneficio:** Reduce fricción cognitiva. El desarrollador piensa "necesito analizar este archivo" → invoca `@analysis` → el protocolo maneja los detalles.

### 5. Por qué Principio KISS Explícito

**Observación:** [Los LLMs tienden a generar código más complejo de lo necesario](https://arxiv.org/html/2411.09916v1).

**Sin KISS explícito:**
```python
# LLM propone
class ConfigurationManager:
    def __init__(self):
        self.strategies = {}
        self.observers = []
    
    def register_strategy(self, name, strategy):
        self.strategies[name] = strategy
    
    def notify_observers(self):
        for observer in self.observers:
            observer.update(self)
```

**Con KISS explícito:**
```python
# LLM propone
config = {
    'api_key': os.getenv('API_KEY'),
    'timeout': 30
}
```

**Resultado:** Es más fácil pedir al agente que añada complejidad a una solución simple, que perder tiempo analizando qué sobra en su propuesta.

---

## Limitaciones y Trade-offs

### Limitaciones Actuales

**1. Dependencia de Cursor (parcial)**
- Implementación usa sistema de rules nativo de Cursor
- Conceptos (rules/memory) son portables a otras herramientas
- Requiere adaptación de sintaxis (`@` commands)

**2. Ventana de contexto sigue limitada**
- Rules con `alwaysApply: true` consumen contexto permanentemente
- Con 10+ rules activas simultáneamente, puedes saturar contexto (~200K tokens)
- **Importante:** Los archivos en `memory/` **NO** consumen contexto automáticamente. Solo ocupan tokens cuando:
  - Los invocas explícitamente: `@memory/20251015-03-backlog.md`
  - Un protocolo los lee/escribe
- Puedes tener cientos de archivos en `memory/` sin impacto en contexto
- **Curación necesaria:** Desactivar rules innecesarias (`alwaysApply: false`), no archivar memory

**3. Sin sincronización automática**
- Cada desarrollador mantiene su `memory/` local
- Decisiones compartidas requieren mover manualmente a `rules/guidelines/`
- No hay tooling para "promover" memory a rule

**4. Curva de aprendizaje inicial**
- Configurar primera vez: 15-20 minutos
- Interiorizar workflow: 2-3 días de uso
- Crear expertos custom: requiere entender estructura

### Trade-offs de Diseño

**Trade-off 1: Automatización vs Control**
- ✅ Protocolos automatizan tareas repetitivas
- ❌ Menos transparencia sobre qué hace el agente
- **Decisión:** Priorizar automatización, documentar bien protocolos

**Trade-off 2: Modularidad vs Simplicidad**
- ✅ Expertos/guidelines separados → reutilización máxima
- ❌ Más archivos para mantener
- **Decisión:** Modularidad vale la pena en proyectos >1 mes

**Trade-off 3: Estructura vs Flexibilidad**
- ✅ Nomenclatura YYYYMMDD-VV → orden cronológico, historial sin límite
- ❌ No puedes usar nombres completamente custom
- **Decisión:** Consistencia > flexibilidad absoluta

**Trade-off 4: Rules activas vs Contexto disponible**
- ✅ Más rules con `alwaysApply: true` → más conocimiento siempre presente
- ❌ Menos contexto disponible para archivos del proyecto
- **Decisión:** Limitar a 5-10 rules activas, rest on-demand
- **No aplica a memory:** Archivos memory no consumen contexto hasta que los invocas

---

## Aplicabilidad Universal

### Más Allá de Cursor

La arquitectura **Conocimiento/Memoria** es agnóstica a la herramienta. El patrón fundamental es:

```
Conocimiento reutilizable + Memoria temporal = Contexto efectivo
```

**Cursor implementa esto vía:**
- `rules/` (conocimiento) + `memory/` (memoria) + `@` commands (invocación)

**Otras herramientas lo implementan de formas diferentes, pero el concepto es el mismo.**

### Adaptaciones a Otras Herramientas

### GitHub Copilot
```
# .github/copilot-instructions.md
Experto: WordPress classic themes
Principios: KISS, DRY
Constraints: [ver memory/constraints-proyecto.md]
```

**Gestión de memory:** Invoca archivos memory manualmente según necesidad, no se cargan automáticamente.

### Claude Code (CLI)
```bash
# .claude/config.json
{
  "rules": ["wordpress-expert", "kiss", "constraints-proyecto"],
  "memory_path": ".claude/memory/"
}
```

**Gestión de memory:** Archivos memory se referencian explícitamente en comandos, no consumen contexto pasivamente.

### Windsurf / Aider / Continue
Cada herramienta tiene mecanismos propios de inyectar contexto. El patrón Conocimiento/Memoria se adapta a todos:

1. **Identifica** cómo tu herramienta inyecta contexto (archivos config, system prompts, etc.)
2. **Mapea Conocimiento** a ese mecanismo (equivalente a `rules/`)
3. **Implementa Memoria** como archivos de referencia (equivalente a `memory/`)
4. **Crea shortcuts** para protocolos comunes (equivalente a `@` commands)

### Principio Universal

**Sin importar la herramienta:**
- Separa **Conocimiento** (persiste) de **Memoria** (temporal)
- Modulariza **Conocimiento** (expertos, guidelines, protocolos)
- Automatiza repetición (protocolos invocables)
- Documenta cronológicamente (YYYYMMDD-VV)

**La implementación específica (rules/, @commands, etc.) es secundaria. El patrón conceptual es lo que importa.**

---

## Conclusión

Contexto Extendido no es solo una estructura de archivos. Es una **arquitectura de conocimiento** basada en el patrón universal:

**Separación Conocimiento/Memoria**

Esta arquitectura:

1. **Extiende memoria del LLM** sin cambiar el modelo
2. **Modulariza expertise** para reutilización entre proyectos
3. **Automatiza workflows** sin perder control
4. **Documenta decisiones** cronológicamente
5. **Se adapta a cualquier herramienta** con capacidad de inyectar contexto

**Implementación en Cursor:**
- `rules/` = inyección de conocimiento
- `memory/` = almacenamiento de memoria
- `@` commands = invocación de protocolos

**El patrón es universal, la implementación es específica de cada herramienta.**

El vibe-coding funciona para prototipos. Para producción, necesitas ingeniería rastreable. Esta arquitectura proporciona el framework conceptual, independientemente de la herramienta que uses.

---

## Referencias

- [Post original sobre Contexto Extendido v1](https://xulioze.com/blog/guia-desarrollar-software-agentes.html)
- [Anthropic's prompt engineering guide](https://docs.anthropic.com/en/docs/prompt-engineering)
- [LLMs and Code Complexity (arXiv)](https://arxiv.org/html/2411.09916v1)
- [Cursor Rules Documentation](https://docs.cursor.com)
- [Knowledge Management Principles (Tiago Forte)](https://fortelabs.com/blog/para/)

---

**¿Preguntas sobre decisiones de diseño?** Abre un [GitHub Issue](link) o participa en [Discussions](link).