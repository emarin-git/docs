# Etapa 0 — Constitution

La constitution es un **documento residente en el repositorio y versionado** que define los principios, políticas y restricciones que cada spec, diseño y agente de IA debe respetar. No es una página de wiki. Está checada en el repo, peer-reviewed, y cargada como contexto persistente para los agentes de IA al inicio de cada sesión.

---

## Tabla de Contenidos

1. [Propósito](#propósito)
2. [Inputs y outputs](#inputs-y-outputs)
3. [Qué va en la constitution](#qué-va-en-la-constitution)
4. [Template de autoría](#template-de-autoría)
5. [Gobernanza y enmiendas](#gobernanza-y-enmiendas)
6. [Reglas de agentes de IA](#reglas-de-agentes-de-ia)
7. [Quality gate G0](#quality-gate-g0)
8. [Mejores prácticas y anti-patterns](#mejores-prácticas-y-anti-patterns)

---

## Propósito

La constitution existe para:

- Dar a los agentes de IA y a los humanos un contexto compartido y duradero sobre *cómo se construye este producto*.
- Prevenir que cada spec re-litigue las mismas decisiones arquitectónicas.
- Hacer los principios auditables y aplicables en CI, no solo aspiracionales.
- Hacer a prueba de decaimiento el conocimiento institucional (vive junto al código, no en la cabeza de alguien).

Si un principio no es aplicable o referenciable, no pertenece a la constitution.

## Inputs y outputs

| | |
|---|---|
| **Inputs** | Estándares de ingeniería de la organización, requisitos de seguridad/compliance, estrategia de producto, postmortems previos, restricciones regulatorias |
| **Outputs** | `constitution.md` (versionado), `policies/*.yaml` (reglas legibles por máquina), fragmentos del system prompt para agentes de IA |
| **Dueños** | Architecture Council (ratificación); CTO/VP Eng (aprobación final); equipo de Platform (aplicación) |
| **Cadencia** | Documento permanente; enmiendas ratificadas se agrupan mensualmente |

## Qué va en la constitution

Siete secciones, cada una obligatoria:

1. **Principios de producto** — valores no-negociables del producto (p. ej., "nunca perdemos silenciosamente datos del usuario").
2. **Principios arquitectónicos** — fronteras, reglas de capas, dependencias permitidas (p. ej., "la capa de dominio no tiene I/O").
3. **Línea base tecnológica** — versiones de lenguajes, elecciones de frameworks, datastore por defecto, el message bus estándar.
4. **Seguridad y compliance** — clasificación de datos, manejo de secretos, reglas para datos regulados.
5. **Calidad** — umbrales de cobertura, SLOs de latencia/disponibilidad, mínimos de accesibilidad.
6. **Reglas operativas para agentes de IA** — qué pueden hacer los agentes sin supervisión, qué requiere aprobación humana (ver abajo).
7. **Reglas de proceso** — umbrales de revisión, definiciones de gates, criterios de ruta ligera.

Cada sección termina con **enforcement**: cómo se verifica la regla (CI lint, policy engine, revisión humana) y **excepciones**: cómo solicitar un waiver.

## Template de autoría

```markdown
# Constitution — <product / repo name>
Version: 2026.04.01  •  Last ratified: 2026-04-15  •  Owner: Architecture Council

## 1. Product principles
- P1. We never silently lose user data.
  - Enforcement: any write path without durable acknowledgement requires CR-DATA waiver.
- P2. ...

## 2. Architectural principles
- A1. Domain layer has no I/O. Side effects live in adapters.
  - Enforcement: `arch-lint` rule `no-io-in-domain`. CI fails on violation.
- A2. Services communicate via versioned, schema-registered events.
  - Enforcement: schema registry CI check.

## 3. Technology baseline
- TypeScript 5.x, Node 22 LTS, Postgres 16, Kafka 3.x, OpenTelemetry.
- Default web framework: Hono. Default ORM: Drizzle. Default test runner: Vitest.

## 4. Security & compliance
- S1. PII is tagged at schema level; queries against PII tables require `pii-reviewer` approval.
- S2. Secrets only via the secret broker; no plaintext in env files in repos.

## 5. Quality bar
- Q1. Unit coverage ≥ 80% on changed lines (delta coverage).
- Q2. p99 API latency ≤ 300ms unless waived in spec NFRs.
- Q3. WCAG 2.2 AA for all user-facing surfaces.

## 6. AI agent operating rules
- AI1. Agents may write code, tests, and specs. Agents may not merge to main.
- AI2. Agents may run read-only tools without approval. Write tools require human-in-the-loop.
- AI3. Agents must cite the spec section ID for every code change.

## 7. Process rules
- R1. Standard path is default. Patch path requires explicit declaration.
- R2. Spec changes affecting public contracts require Architect approval.
```

La constitution es Markdown legible por humanos más un compañero legible por máquina (`policies/*.yaml`) para que CI y la spec platform puedan consumirla directamente.

## Gobernanza y enmiendas

Las enmiendas siguen la misma ruta que una spec: PR, revisión, ratificación por el Architecture Council, version bump. Las enmiendas deben incluir:

- Justificación y el problema que la regla actual no maneja.
- Plan de migración para specs y código existentes (o una cláusula de grandfather explícita).
- Fecha efectiva.

Las enmiendas de emergencia (p. ej., durante un incidente de seguridad) pueden ratificarse asíncronamente por dos miembros cualquiera del Architecture Council y confirmarse en la siguiente reunión programada.

## Reglas de agentes de IA

Esta sección merece su propio énfasis porque los agentes de IA la leerán antes de cada sesión.

La constitution define explícitamente:

- **Permisos** — qué herramientas pueden invocar los agentes, y cuáles requieren aprobación humana.
- **Fronteras** — paths, repos o entornos prohibidos (p. ej., secretos de producción, datos del cliente).
- **Citas obligatorias** — cada artefacto producido por un agente debe referenciar su spec ID de origen y la cláusula constitucional.
- **Casos de rechazo** — cuándo un agente debe detenerse y preguntar (spec ambigua, contexto faltante, conflicto con la constitution).
- **Hooks de auditoría** — cada acción del agente se registra con prompt, herramientas llamadas, archivos tocados, y el aprobador humano si lo hay.

Estas reglas se distribuyen como un fragmento de system prompt cargado por el agent runner. No son consultivas — el runner las hace cumplir.

## Quality gate G0

La constitution debe pasar G0 antes de que el repo se desbloquee para trabajo de spec:

- Las siete secciones presentes y no vacías.
- Cada principio tiene enforcement explícito.
- El archivo de políticas legible por máquina pasa el lint limpio.
- Dos miembros del Architecture Council han firmado.

El drift constitucional (el Markdown dice una cosa, el YAML dice otra, CI hace cumplir una tercera) es en sí mismo un incidente.

## Mejores prácticas y anti-patterns

**Mejores prácticas.**

La constitution es corta. Apunta a un documento que un nuevo ingeniero senior pueda leer el primer día. Si la tuya supera las ~30 páginas, tienes policy bloat — divide los estándares que cambian raramente en sub-docs enlazados. Cada principio debe responder "¿cómo notaría si esto se violara?" Si no puedes responder, es una declaración de valor, no un principio, y no pertenece aquí. Mantén ejemplos y justificación en línea con cada principio para que el agente que carga este archivo como contexto vea el *por qué* junto al *qué*.

**Anti-patterns.**

Una constitution que es solo una lista de preferencias ("nos gusta el código limpio"). Una constitution que se contradice a sí misma a través de las secciones. Una constitution que nunca se enmienda (sugiere que no se está leyendo). Una constitution que se enmienda semanalmente (sugiere que es un basurero de backlog). Meter decisiones tácticas ("usar lodash 4.x") en la constitution en lugar de en los manifiestos de dependencias.

---

[← Lifecycle](../lifecycle-sp.md) · [Siguiente: Specification →](specification-sp.md)
