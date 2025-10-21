# Solución: Setup Básico .cursor/

**Fecha:** 2025-10-03  
**Origen:** Análisis `20251003-01-cursor-structure.md`  
**Prioridad:** Alta

---

## PROBLEMA

Sistema `.cursor/` funcional pero sin:
1. Control de archivos sistema (`.DS_Store` en repo)
2. Documentación para uso/mantenimiento
3. Onboarding para otros devs o futuras sesiones

**Impacto:**
- Contaminación repo con archivos macOS
- Curva aprendizaje alta para entender sistema
- Riesgo de uso inconsistente de protocolos

---

## PROPUESTA (KISS)

Crear 2 archivos básicos:

### 1. `.cursor/.gitignore`
Ignorar archivos sistema operativo comunes.

### 2. `.cursor/README.md`
Documentación mínima viable:
- Qué es este sistema
- Estructura de carpetas
- Cómo usar cada protocolo (`@analysis`, `@backlog`, etc.)
- Convenciones establecidas

**No incluir:** Scripts, templates, automatizaciones (YAGNI por ahora)

---

## IMPLEMENTACIÓN

### Paso 1: Crear `.gitignore`
**Archivo:** `.cursor/.gitignore`
```gitignore
# macOS
.DS_Store
.AppleDouble
.LSOverride

# Logs
*.log

# Temporales
*.tmp
*.temp
```

### Paso 2: Crear `README.md`
**Archivo:** `.cursor/README.md`

Estructura:
```markdown
# Cursor AI Setup

## Qué es esto
Sistema de reglas personalizadas...

## Estructura
rules/
  experts/      → Perfiles especialización
  guidelines/   → Principios desarrollo
  utils/        → Protocolos trabajo
memory/         → Análisis/backlogs generados

## Uso de Protocolos

### @analysis
Analiza archivos/carpetas...

### @solution
Propone solución KISS...

### @backlog
Convierte solución en tareas...

### @execute
Ejecuta tareas del backlog...

### @commit
Genera commits convencionales...

## Convenciones

Nombres memoria: YYYYMMDD-VV-type-description.md
Estados tarea: ⏳ Pending | 🔄 In Progress | ✅ Done | ⚠️ Blocked
```

### Paso 3: Verificar
```bash
git status  # Verificar .DS_Store no aparece
```

---

## TRADE-OFFS

### ✅ Pros
- **Rápido:** 2 archivos, 10 minutos max
- **Útil inmediato:** Documenta sistema ya funcional
- **Sin overhead:** No añade complejidad
- **Git limpio:** Previene commits accidentales basura

### ⚠️ Contras
- **README manual:** Requiere update si cambian protocolos
- **Sin validación:** No garantiza que se siga documentación

### 🔮 Futuro (si es necesario)
- Templates cuando haya >10 documentos en memory/
- Índice automático cuando sea difícil buscar
- Scripts validación si hay errores frecuentes

**Principio aplicado:** YAGNI (You Aren't Gonna Need It) - implementar solo lo que se necesita ahora.

---

## DECISIONES TÉCNICAS

**Por qué no templates ahora:**
- Solo existe 1 documento en memory/ (este análisis)
- No hay evidencia de formato inconsistente aún
- KISS: resolver problema real, no hipotético

**Por qué sí README ahora:**
- Sistema ya complejo (8 reglas MDC)
- Curva aprendizaje alta sin docs
- Costo bajo, beneficio inmediato

**Scope del .gitignore:**
- Solo archivos OS comunes (macOS evidenciado)
- Sin wildcards agresivos que puedan ocultar problemas
- Extensible si aparecen nuevos casos

---

## PRÓXIMOS PASOS

1. Crear `.cursor/.gitignore` con contenido propuesto
2. Crear `.cursor/README.md` con estructura completa
3. Verificar `git status` limpio
4. Opcional: Remover `.DS_Store` existente del índice git
   ```bash
   git rm --cached .cursor/.DS_Store
   ```

**Tiempo estimado:** 15 minutos  
**Backlog necesario:** No (cambio trivial)  
**Listo para ejecutar:** Sí

---

## REFERENCIAS

- Git ignore patterns: https://git-scm.com/docs/gitignore
- Documentation best practices: Keep It Simple
- Análisis origen: `20251003-01-cursor-structure.md`


