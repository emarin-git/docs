# Roles y Responsabilidades

Cómo el SDLC SDD-First se mapea en los roles. El marco asume una organización de producto moderna con squads cross-funcionales respaldados por plataformas horizontales.

---

## Tabla de Contenidos

1. [Catálogo de roles](#catálogo-de-roles)
2. [Product](#product)
3. [Engineering](#engineering)
4. [QA](#qa)
5. [Architecture](#architecture)
6. [Platform / DevOps](#platform--devops)
7. [CX / UX](#cx--ux)
8. [Los agentes de IA como un "rol"](#los-agentes-de-ia-como-un-rol)
9. [Contribuciones de roles por etapa](#contribuciones-de-roles-por-etapa)

---

## Catálogo de roles

| Rol | Etapa de mayor influencia | Resumen en una línea |
|---|---|---|
| Product Manager | Spec, Clarification, Operations | Dueño del *por qué* y el *qué*; acepta el trabajo entregado contra el spec. |
| Tech Lead | Design, Implementation | Dueño de la corrección técnica para las entregas del squad. |
| Engineer | Implementation, Validation | Autor de código y pruebas; colabora con agentes. |
| QA Engineer | Validation | Dueño de los pipelines de aceptación, drift checks y el nivel de calidad. |
| Architect | Constitution, Design | Administra las preocupaciones transversales y la constitution. |
| Platform / DevOps | Constitution, Deployment, Operations | Construye y opera la plataforma que hace posible el SDLC. |
| SRE | Deployment, Operations | Dueño de la confiabilidad; se asocia con los squads en los SLOs. |
| CX/UX Designer | Spec, Clarification, Validation | Dueño de la experiencia de usuario y el comportamiento; co-autor de specs de cara al usuario. |
| CX/UX Researcher | Spec, Operations | Aporta señal del usuario; cierra el ciclo de feedback. |
| Security Engineer | Constitution, Design, Validation | Dueño de la postura de seguridad; controla contratos que tocan datos sensibles. |
| Engineering Manager | All | Dueño del flujo, la capacidad y la salud del squad. |

## Product

Los Product Managers son responsables del spec de extremo a extremo. Impulsan Specification y Clarification, colaboran con Engineering en la factibilidad durante Design, y aceptan el trabajo entregado en Validation contra el mismo spec que escribieron.

En una org SDD-First, los PMs escriben *mejores* specs que bajo el Agile canónico porque el spec es estructurado, linteable y reutilizado. Dedican menos tiempo a los tickets y más tiempo al planteamiento del problema.

Responsabilidades específicas del PM:

- Autoría del spec; co-autoría con el agente de borrador IA y ajuste fino con el Tech Lead y CX/UX.
- Resolución de ítems `MUST-CLARIFY` con decisiones nombradas e impacto registrado.
- Definición de KPIs de negocio en el plan de telemetría.
- Aceptación o rechazo final en Validation.
- Transmisión de señal de usuario (en colaboración con CX) hacia Operations.

## Engineering

Los Engineers (y Tech Leads) son dueños del ciclo de vida técnico desde Design hasta Operations.

Los Tech Leads específicamente:

- Co-autoran el documento de diseño con el squad.
- Son dueños de G3 (design review) para los specs del squad.
- Asignan tareas entre humanos y agentes de IA, incluyendo el marcado de `agent_assignable`.
- Mantienen la salud técnica — presupuestos de refactoring, higiene de dependencias, nivel de observabilidad.
- Mentorizan; aseguran que la disciplina TDD se mantenga.

Los Engineers:

- Implementan tareas bajo TDD.
- Autoran y revisan PRs (incluidos los PRs de autoría de agentes — aplica separación de funciones).
- Son dueños de los runbooks para el código que autoran.
- Participan en el on-call para los servicios de su squad.

## QA

El rol de QA cambia en una org SDD-First. QA **no** es principalmente una función de pruebas manuales; es una función de **quality engineering** que:

- Es dueña de la disciplina de mapeo spec-a-prueba (cada AC tiene una prueba, cada prueba tiene un spec).
- Construye y mantiene el framework de acceptance tests (el helper `spec()`, el validation report, el drift check).
- Ejecuta y supervisa las suites de validación NFR (carga, chaos, accesibilidad).
- Opera G5 — el validation gate.
- Realiza exploratory testing focalizado en nuevas superficies — de alto impacto, orientado a hipótesis, no persiguiendo regresiones.

QA colabora con los engineers, no los bloquea. La plataforma es dueña del regression suite; QA es dueño de la estrategia.

## Architecture

Los Architects son administradores de las preocupaciones transversales. *No* son una torre de marfil — pasan la mayor parte de su tiempo en design reviews, en crítica y enmendando la constitution a medida que la realidad enseña.

Los Architects:

- Mantienen la constitution. Ratifican enmiendas. Autoran principios transversales.
- Aprueban specs de alto impacto y cualquier diseño cross-servicio o que rompa contratos.
- Dirigen el Architecture Council (un foro permanente, no un comité de control).
- Mantienen arquitecturas de referencia y el registro de schemas/eventos.
- Mentorizan a los Tech Leads. El objetivo es que con el tiempo se necesiten menos aprobaciones del Architect, no más.

Los Architects deben construir, aunque sea en pequeña dosis, para mantenerse calibrados.

## Platform / DevOps

El equipo Platform construye y opera la **plataforma SDD** — el sistema de gestión de specs, la capa de agentes de IA, el pipeline de validación, el pipeline de deploy, el policy engine. Ver [SDD Platform Architecture](../architecture/sdd-platform-sp.md) para la vista del sistema; esta sección cubre las expectativas del *rol*.

Platform/DevOps:

- Es dueño de la experiencia del developer. El tiempo del inner loop, la tasa de flakiness en CI, el lead time de deploy son *sus* métricas.
- Implementa la aplicación de políticas para la constitution.
- Opera el agent runner de IA, incluyendo los audit trails.
- Proporciona templates de camino pavimentado: scaffolds de servicios, templates de dashboards, templates de runbooks.
- Se asocia con los squads para incorporarlos al framework.

Un modo de falla común es que Platform construya herramientas que nadie usa. El éxito de Platform se mide por la adopción del squad y la reducción del dolor reportado por el squad, no por el conteo de features entregadas.

## CX / UX

CX/UX es un **participante de primera clase** en el SDLC, no un paso de traspaso.

Los UX Designers:

- Co-autoran los specs de cara al usuario desde el inicio.
- Resuelven los ítems `MUST-CLARIFY` relevantes para la usabilidad.
- Producen artefactos de diseño (flujos, prototipos) referenciados desde el spec.
- Aprueban en Validation para usabilidad/accesibilidad.
- Participan en el ciclo de Operations para features de cara al usuario.

Los CX Researchers:

- Transmiten señal de usuario hacia la etapa de Specification (problemas, evidencia, puntos de dolor).
- Realizan investigación de usabilidad en la etapa de prototipo de diseño en specs de alto impacto.
- Cierran el ciclo en Operations: ¿la feature entregada realmente resolvió el problema?

Los specs de cara al usuario que se entregan sin participación de CX/UX son marcados por la plataforma; esto es política, no consejo.

## Los agentes de IA como un "rol"

Los agentes no son empleados, pero son **participantes responsables** en el SDLC con límites explícitos.

Los agentes reciben tareas de la misma manera que los humanos. Cada rol de agente tiene un fragmento de system prompt y un conjunto de herramientas permitidas:

- **Spec-draft agent** — redacta specs a partir de planteamientos de problemas; no puede hacer merge.
- **Clarification agent** — detecta brechas y contradicciones; no puede decidir.
- **Design-draft agent** — redacta diseños técnicos y task graphs.
- **Critique agent** — revisa de forma independiente specs y diseños en busca de brechas.
- **Implementation agent** — escribe código y pruebas bajo TDD; abre PRs.
- **Reviewer agent** — asiste a los revisores humanos con verificaciones específicas (seguridad, accesibilidad, drift); no aprueba.

Cada rol de agente está documentado como una descripción de trabajo, con herramientas permitidas, herramientas prohibidas, disparadores de escalada y expectativas de auditoría.

## Contribuciones de roles por etapa

Un resumen; la [matriz RACI](raci-sp.md) es la versión formal.

| Etapa | Impulsa | Co-autora | Revisa / Aprueba | Informado |
|---|---|---|---|---|
| Constitution | Architect | Tech Leads, Platform | Architecture Council, CTO | Todos |
| Specification | PM | Tech Lead, UX | Architect (cuando aplica), Security (cuando aplica) | Squad, CX |
| Clarification | PM | Tech Lead, UX, Architect | Todos los co-autores | Squad |
| Design | Tech Lead | Engineers, UX (para UI) | Architect, Platform/SRE, Security | PM |
| Implementation | Engineers | Agentes IA | Tech Lead, peer engineers | PM, QA |
| Validation | QA | Squad | PM, UX, SRE, Architect (si es cross-servicio) | Platform |
| Deployment | Platform/SRE | Squad | Squad on-call | PM, CX |
| Operations | SRE + Squad | Todos | — (continuo) | Todos |

---

[← Overview](../overview-sp.md) · [Squads →](squads-sp.md) · [RACI →](raci-sp.md)
