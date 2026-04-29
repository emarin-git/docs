# Mejores Prácticas, Políticas y Anti-patterns

La verdad acumulada sobre cómo ejecutar bien el SDLC SDD-First — a cualquier escala, en cualquier topología. Este archivo es el currículo contra el que se incorpora a los nuevos squads y la referencia para las retrospectivas.

---

## Tabla de Contenidos

1. [Principios transversales](#principios-transversales)
2. [Mejores prácticas por etapa](#mejores-prácticas-por-etapa)
3. [Catálogo de anti-patterns](#catálogo-de-anti-patterns)
4. [Políticas (valores por defecto recomendados)](#políticas-valores-por-defecto-recomendados)
5. [Guía de escalado](#guía-de-escalado)
6. [Multi-repo / microservices](#multi-repo--microservices)
7. [Flujos de trabajo asistidos por IA](#flujos-de-trabajo-asistidos-por-ia)
8. [Adopción y rollout](#adopción-y-rollout)

---

## Principios transversales

Estos aplican en cada etapa:

- **El spec gana.** Cuando el código, las pruebas o el comportamiento no coinciden con el spec, la plataforma genera un defecto de drift — y la resolución es actualizar el spec o corregir el código, nunca dejar que diverjan en silencio.
- **Hacer las políticas ejecutables.** Una política que no se aplica es una vibra. Si una regla importa, corre en CI, en el policy engine o en el deploy pipeline.
- **Pequeño, frecuente, reversible.** Specs pequeños, PRs pequeños, rollouts pequeños. La reversibilidad es una propiedad del sistema, no una esperanza.
- **Agentes de IA dentro de límites, no con correa.** Los agentes funcionan bien dentro de restricciones explícitas; fallan gravemente bajo supervisión vaga. Define las restricciones, automatiza la auditoría, luego *confía* en el comportamiento acotado.
- **Optimizar el inner loop.** El tiempo del inner loop predice la adopción mejor que cualquier política. Si el inner loop es lento, tu TDD colapsará y tus specs se pudrirán.

## Mejores prácticas por etapa

### Constitution

**Prácticas.** Mantenla corta y aplicable. Cada cláusula tiene una respuesta a "¿cómo notaría si esto se viola?" Las enmiendas se agrupan mensualmente con justificación y planes de migración. La constitution se carga como contexto del agente — escríbela como si el agente la fuera a leer (porque lo hará).

**Políticas.**

- Todas las enmiendas requieren ratificación del Architecture Council, registrada en el historial de versiones.
- El archivo de políticas legible por máquina debe pasar el lint antes de que se apruebe la enmienda de constitution.
- Revisión trimestral de waivers — los patrones de waivers indican que la regla es incorrecta, no el equipo.

### Specification

**Prácticas.** Co-autorar con el agente de borrador IA; ajustar con humanos. Escribir primero la sección de *fuera del alcance*. Usar IDs estables en todos lados. Mantener los specs en ≤ ~5 páginas de contenido de autoría (excluyendo generado/enlazado); los specs más grandes se dividen en hijos enlazados.

**Políticas.**

- Los lints automatizados de G1 deben pasar para que cualquier spec entre en Clarification.
- Los specs que tocan contratos públicos requieren aprobación del Architect.
- Los specs que tocan comportamiento de cara al usuario requieren aprobación de UX; los specs que manejan datos sensibles requieren aprobación de Security.
- Los specs sin ACs testables fallan G1.

### Clarification

**Prácticas.** Limitar a ≤3 días hábiles. Ejecutar el clarify-agent para completitud; humanos para el juicio. Registrar decisiones, no discusiones. Si una decisión implica un cambio en el spec, editar el spec — no dejarlo solo en el log.

**Políticas.**

- Cero marcadores `MUST-CLARIFY` abiertos requeridos para G2.
- El clarification log es append-only y auditable.
- Los agentes no pueden enmendar specs de forma autónoma.

### Design

**Prácticas.** Diseñar contratos primero; implementación después. Hacer tareas de ≤1 día. Ejecutar el critique agent antes de la revisión del Architect. Siempre documentar las alternativas consideradas. Planificar migración y rollback antes de hacer merge del diseño.

**Políticas.**

- Cada AC debe estar cubierto por ≥1 tarea.
- Cada tarea debe referenciar su spec.
- Los cambios DDL requieren un plan de rollback explícito.
- Los diseños cross-servicio requieren revisión de diff de contratos.
- `agent_assignable: false` es requerido para tareas de seguridad crítica, migración irreversible y cripto/auth.

### Implementation

**Prácticas.** TDD por defecto. PRs de ≤300 líneas. El trabajo de autoría de agentes revisado por un humano *diferente* al del prompter. Emparejar humanos con agentes en trabajo novedoso. Pruebas basadas en propiedades para invariantes en NFRs. Runbooks actualizados junto con el código, no después.

**Políticas.**

- Los agentes no pueden hacer merge a main.
- Delta de cobertura en líneas modificadas ≥80% (o umbral establecido en la constitution).
- Las tareas exentas de TDD requieren justificación registrada.
- Cada PR cita el ID de tarea, ID de spec y cláusulas constitucionales.
- Los lints arquitectónicos, las verificaciones de contratos y los escaneos de seguridad bloquean el merge.

### Validation

**Prácticas.** Automatizar el reporte; nunca escribirlo a mano. Tratar el drift como un defecto con severidad. Ejecutar la validación NFR tan pronto como el diseño produzca suficiente superficie para instrumentar. Hacer el validation report parte de las release notes.

**Políticas.**

- Estado del drift report `clean` o waiver explícito requerido para G5.
- Cada AC en el spec debe tener ≥1 prueba que pase.
- Los cambios que rompen contratos requieren un waiver aprobado antes de que se cierre G5.
- Los incumplimientos de NFR fallan G5 a menos que una excepción sea aprobada por el rol relevante (SRE para rendimiento, Security para seguridad, etc.).

### Deployment

**Prácticas.** Progresivo por defecto. Desacoplar deployment de release con flags. Rollback con un botón. Provisionar dashboards en código. Ejecutar ejercicios de rollback trimestralmente por servicio.

**Políticas.**

- Todos los releases usan progressive delivery a menos que se documente una excepción.
- Versiones fijadas en los deploy specs; sin `:latest`.
- Los SBOMs y las firmas de imágenes son obligatorios.
- Cada release tiene un runbook con criterios de reversión explícitos.
- Los releases de producción del viernes por la tarde requieren aprobación a nivel de Director (política cultural; adaptar para tu zona horaria/topología).

### Operations

**Prácticas.** Tratar el spec como documentación viva. Las acciones del postmortem clasificadas en spec/constitution/runbook/platform — nunca "ser más cuidadosos". La señal de usuario fluye continuamente hacia el backlog del spec. La salud del spec se revisa trimestralmente por servicio.

**Políticas.**

- Las acciones del postmortem con más de 60 días son en sí mismas un incidente.
- Cada servicio tiene un propietario documentado; los servicios huérfanos se bloquean en deploy.
- Los incumplimientos de SLO que exceden el error budget pausan los cambios no esenciales al servicio afectado hasta que la tasa de consumo se normalice.

## Catálogo de anti-patterns

Una lista no exhaustiva de las formas más comunes en que este marco falla cuando se aplica descuidadamente.

### Anti-patterns de spec

El spec "implementación disfrazada" — prescribe nombres de clases y SQL; ya no es un contrato, solo código en prosa.
El spec "lista de deseos" — múltiples deseos apenas relacionados; imposible de validar.
El spec "congelado" — nunca enmendado después de la aprobación; el drift se acumula en silencio.
El spec "retro-spec" — escrito después de que el código llega; describe lo que se construyó, no lo que se quería.
La "constitution en las sombras" — spec contradice la constitution pero se cuela por la fatiga del revisor.

### Anti-patterns de proceso

**Inflación de gates.** Cada problema se convierte en un nuevo gate. Eventualmente nadie puede entregar; los equipos eluden los gates.
**Teatro de gates.** Los gates existen pero se eluden rutinariamente; el elusión se convierte en la norma.
**Abuso de la ruta patch.** Los cambios de comportamiento se envían como patches porque la ruta Standard es demasiado lenta — el síntoma es una ruta Standard demasiado lenta, no engineers perezosos.
**Clarification por aplazamiento.** "Lo resolveremos en el diseño" — el mismo problema ahora contamina una etapa más costosa.

### Anti-patterns de agentes

**Agentes haciendo merge de su propio trabajo.** Elimina completamente el gate de juicio humano.
**Agentes escribiendo tanto pruebas como el código que las satisface en una sesión.** Riesgo de colusión; las pruebas pasan mientras el spec no se cumple.
**Ediciones ocultas de agentes.** Cambios de autoría de agentes presentados como PRs humanos sin procedencia — destruye la auditabilidad.
**Output genérico de agentes.** Seguir el esfuerzo por líneas de código generadas en lugar de specs satisfechos — puro cargo cult.

### Anti-patterns organizacionales

**Equipos de componentes.** Las mitades frontend y backend de una feature en equipos diferentes. Los specs se convierten en cebo para traspasos.
**Servicios compartidos sin propietario.** Propiedad ambigua = specs obsoletos, on-call ambiguo, respuesta lenta a incidentes.
**Architects que no construyen.** Deriva entre principio y práctica; la constitution se vuelve irrelevante.
**Integrados permanentemente temporales.** Engineers de Security/UX permanentemente integrados en un squad se convierten en puntos únicos de falla.

### Anti-patterns de plataforma

**Construir herramientas que nadie usa.** Platform medido por conteo de entregas en lugar de adopción.
**Agent runners opacos.** Sin auditoría, sin trazabilidad — destruye la propuesta de valor del SDD.
**CI de una sola pasada.** Pipelines que corren de extremo a extremo en cada PR pero no cachean nada — el tiempo del inner loop explota.
**"Advertencias" de drift sin consecuencias.** Si el drift nunca bloquea, nunca se corrige.

## Políticas (valores por defecto recomendados)

Un conjunto de inicio que puedes adoptar y enmendar. Codifícalos en la constitution y respáldalos con reglas del policy engine.

| Política | Valor por defecto | Notas |
|---|---|---|
| Lint de calidad del spec | Requerido para G1 | Todos los FR/NFR/AC deben tener IDs estables |
| Delta de cobertura | ≥ 80% en líneas modificadas | Por repo; subir para librerías, bajar para prototipos |
| Tamaño del PR | ≤ 300 líneas diff | Guía suave, límite duro en 800 con justificación |
| Revisor requerido | ≥1 humano, no el prompter | Separación de funciones para trabajo de autoría de agentes |
| Compatibilidad de contratos | Compatible hacia atrás por defecto | Los cambios que rompen requieren waiver |
| Estrategia de release | Progresivo (canary → ramp) | Excepciones documentadas |
| Política de imágenes | Fijadas, firmadas, SBOM | Aplicado en deploy |
| Seguimiento de postmortem | Cerrado dentro de 60 días | Tracked como métrica de plataforma |
| Revisión de salud del spec | Trimestral por servicio | Tracked en el catálogo de servicios |
| Cadencia de enmiendas a la constitution | Lote mensual | Ruta de emergencia para incidentes |
| Permisos de agentes | Default-deny | La constitution enumera explícitamente las herramientas permitidas por rol |

## Guía de escalado

**Pequeño (≤30 engineers).** Adoptar el 80% del marco. Constitution ligera; un platform engineer compartido; agentes de IA como flota compartida; RACI manual; drift detector mínimo viable. Omitir tribus; todos son un squad-de-squads.

**Mediano (30–150 engineers).** Marco completo. Equipo de Platform dedicado. Tribus con rotaciones de on-call compartidas y constitutions parciales. Agent runner multi-tenant con presupuestos de costo por tribu. Architecture Council formalizado; se reúne quincenalmente.

**Grande (150+ engineers).** Federar. La constitution a nivel de org se mantiene pequeña y absoluta (seguridad, compliance, contratos). Las constitutions a nivel de tribu la extienden. Múltiples equipos de platform (platform de org + platforms de tribu). Gestión de specs explícitamente cross-repo con grafo de trazabilidad global. Flotas de agentes de IA por tribu compartiendo herramientas e infraestructura de auditoría. Las auditorías trimestrales de límites son no negociables; los squads mal alineados son reorganizados.

Un fallo de escalado común: copiar los hábitos de un squad exitoso e intentar aplicarlos centralmente. Lo que funcionó suele ser la *autonomía*, no los hábitos específicos.

## Multi-repo / microservices

El marco trata los specs y el código como **co-ubicados**: el spec para una feature vive junto al código que lo implementa. Para estates multi-repo / microservicio:

- **Los specs cross-repo son de primera clase.** El spec vive en el repo *impulsor* (generalmente el de cara al usuario), y enlaza hacia los schemas, registries y servicios contribuyentes que toca.
- **El schema registry es global.** Los schemas por-repo no escalan; tendrás drift entre consumidores y productores en semanas.
- **El grafo de trazabilidad es global.** Los specs, diseños, tareas, PRs y deploys deben ser consultables entre repos — ese es el punto central.
- **El ownership.yaml es global.** Cada servicio, cada schema público, cada contrato tiene exactamente un squad propietario. Auditar trimestralmente.
- **Las contract tests corren cross-repo.** El consumer-driven contract testing (ej., Pact) es una línea base, no opcional, en estates de microservicios.

Un modo de falla común en microservicios es tratar la etapa de spec como una preocupación por-repo. Los specs que tocan múltiples servicios necesitan un autor propietario único y aprobaciones explícitas de squads consultados en G3. La plataforma debe hacer de esto un comportamiento por defecto, no papeleo.

## Flujos de trabajo asistidos por IA

Los agentes amplifican cualquier proceso que tengas. Las prácticas clave:

**Tratar a los agentes como nuevos empleados con superpoderes y sin juicio.** Límites explícitos, trabajo revisado, sin autoridad de merge, audit trail obligatorio.

**Usar múltiples agentes separados para roles adversariales.** No tengas el mismo agente redactando un diseño *y* criticándolo. Diferentes prompts, diferentes proveedores de modelos cuando sea posible.

**Dejar que los agentes hagan el trabajo pesado.** Scaffolds, migraciones de schema, instrumentación de telemetría, refactors con buena cobertura de pruebas, traducciones mecánicas. Son excelentes aquí.

**No dejar que los agentes posean trabajo novedoso transversal.** Trabajo de rendimiento sin un profiler en el ciclo, rutas de seguridad crítica, elecciones arquitectónicas novedosas — los humanos impulsan, los agentes asisten.

**Citar todo.** Cada artefacto de autoría de agentes referencia su ID de spec fuente y cláusulas constitucionales. Esto es no negociable; es lo que hace funcionar el audit trail.

**Medir lo que importa.** Seguir los specs satisfechos por ciclo, defectos de drift por release, PRs de autoría de agentes revertidos, hallazgos de seguridad de autoría de agentes capturados en revisión. No seguir las líneas de código generadas.

**Evaluar continuamente el fitness del agente.** Cuando lleguen nuevas versiones de modelos, ejecutarlas en un benchmark de tareas recientes antes de promoverlas en el runner. Las regresiones de calidad en los flujos de trabajo de producción son reales y costosas.

## Adopción y rollout

Adoptar este marco en una org existente es en sí mismo un proyecto. Fases recomendadas:

**Fase 0 — Constitution (semana 1–2).** Autorar una constitution v0.1 desde tus estándares existentes. Obtener el signoff de Architecture y CTO. Hacerlo el contexto cargado para cualquier herramienta de IA ya en uso.

**Fase 1 — Un squad, una feature (semanas 3–6).** Elegir un squad dispuesto y ejecutar una sola feature a través del ciclo de vida completo. Construir el spec linter mínimo, el drift checker y la integración de agentes para apoyarlo. Mantener manual donde la automatización sería lenta de construir.

**Fase 2 — Fundamentos de plataforma (semanas 4–12, en paralelo).** Construir la fuente de verdad del spec, el detector de lint/drift, el agent runner y el policy engine en una plataforma creíble. Emparejar con el squad piloto.

**Fase 3 — Expandir a una tribu (meses 3–6).** Incorporar 3–5 squads más. Refinar la constitution basándose en lo que reveló el piloto. Migrar el trabajo en curso existente al marco solo en límites naturales (siguiente feature principal), nunca a mitad de una feature.

**Fase 4 — Adopción a nivel de org (meses 6–18).** RACI completo, gates aplicados, grafo de trazabilidad completo. Ejecutar auditorías de límites trimestrales. Seguir la adopción con métricas: % de features a través de la ruta Standard, defectos de drift por release, tiempo del ciclo postmortem-a-enmienda.

El fallo más común es exigir el marco antes de que la plataforma lo soporte — los equipos imitan el proceso sin la automatización, y el trabajo de políticas consume su capacidad. Construir la plataforma en paralelo, en lockstep.

---

[← Overview](../overview-sp.md) · [Architecture ↑](../architecture/sdd-platform-sp.md) · [Lifecycle ↑](../sdlc/lifecycle-sp.md)
