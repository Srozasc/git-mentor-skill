# Perfil `team` (reglas comunes)

> Referencia cargada por `SKILL.md` cuando el proyecto usa `profile: team`.
> **Importante**: las reglas de este archivo aplican **únicamente** al modo `team`.
> Después de leerla, cargar **solo** el flujo que aplica:
> - `flow: develop` → leer `references/flow-develop.md`
> - `flow: direct` → leer `references/flow-main.md`

## Propósito

Guiar a desarrolladores juniors paso a paso dentro de un repositorio compartido para que trabajen en ramas cortas y aisladas, mantengan una historia limpia, abran PRs fáciles de revisar y no pisen el trabajo del resto del equipo.

## Configuración por defecto del perfil `team`

| Parámetro | Valor por defecto | Descripción |
|---|---|---|
| `flow` | `develop` | `develop`: los PRs de feature van a `develop` y el release sale hacia `main`. `direct`: los PRs de feature van directo a `main`. (Definido por el usuario en la activación, D6.) |
| `feature_target` | `develop` (flujo `develop`) / `main` (flujo `direct`) | Rama a la que apuntan los PRs de feature. |
| `base_branch` | `main` | Rama de producción, donde vive lo que está desplegado. |
| `integration_branch` | `develop` (solo flujo `develop`) | Rama de integración donde se acumulan los cambios revisados. |
| `branch_prefixes` | `feat/`, `fix/`, `chore/` | Prefijos permitidos para ramas (`<prefijo><short-slug>`). |
| `merge_strategy` | `squash` | Estrategia de merge de los PRs de feature. |
| `release_strategy` | `squash` | Estrategia del release hacia `main` (solo flujo `develop`). |
| `merge_by` | `reviewer` | Mergea quien aprueba el code review, **nunca el autor junior**. |
| `commit_language` | `en` | Mensajes de commit y descripción técnica en inglés (salvo override en memoria). |

---

## Reglas innegociables para el dev junior

1. **Nunca commitear ni pushear directo a `main`**. Todo cambio entra por PR. *(En flujo `develop` tampoco se pushea directo a `develop`: siempre por PR.)*
2. **Nunca mergear tu propio PR**. El agente puede abrir el PR tras tu confirmación (D5), pero el merge lo ejecuta el reviewer después de aprobar.
3. **Una rama = una tarea**. Si mientras trabajás encontrás un bug no relacionado, anotalo y resolvelo en otra rama `fix/<slug>` nacida de la rama de destino del PR.
4. **Ramas de vida corta**. Si una rama lleva varios días abierta o acumula demasiados cambios, el agente sugiere partir la entrega.
5. **Rebase sobre la rama destino antes de abrir el PR** para llevar tu rama al día sin meter commits de merge intermedios.
6. **`git push --force-with-lease` con controles estrictos (D4)**:
   - Solo permitido sobre **tu propia rama** (`feat/`, `fix/`, `chore/`) tras un `git rebase`.
   - **Prohibido** usar `--force` a secas.
   - **Prohibido** sobre `main`, `develop` o ramas donde colabore otra persona.
   - Requiere **confirmación explícita aparte** con explicación completa del riesgo (no entra en el OK general del plan).
7. **En flujo `develop`, el release hacia `main` no es tarea del junior**: lo hace quien tenga el criterio de release, con el checklist de release.

---

## Preparar y abrir el PR (común a ambos flujos)

1. **Pre-PR checklist** (de `SKILL.md`): tests verdes, lint/type-check limpios, working tree limpio, sin archivos basura ni secretos, commits en formato Conventional Commits.
2. **Proponer título y descripción**:
   - Título: `feat(scope): ...` / `fix(scope): ...` con el scope del área tocada.
   - Descripción: qué cambia, por qué y cómo se prueba.
3. **Pedir confirmación** con `ask_user_choice` (D5) y **abrir el PR** apuntando a `feature_target`.
4. **Code review**:
   - Si piden cambios, hacer nuevos commits en la misma rama y `git push`.
   - Cuando el reviewer aprueba, **el reviewer ejecuta el merge** con la estrategia del perfil.

### Ejemplo (plan + porqué)

```text
Tarea: Commitear la validación de cancelación, subir la rama y abrir el PR
Perfil: team · Flujo: develop · Rama actual: feat/order-cancellation

Plan:
1. git add services/orders-service/src/
   → Incluye solo los archivos del servicio modificado en el commit.
2. git commit -m "feat(orders): reject cancellation when order is already dispatched"
   → Registra el cambio con formato Conventional Commits para que el historial y el squash sean claros.
3. git fetch origin && git rebase origin/develop
   → Verifica que tu rama esté al día con la rama destino antes de pedir revisión.
4. git push -u origin feat/order-cancellation
   → Publica tu rama en GitHub para poder crear el Pull Request.

¿Ejecuto el plan? (Después de subirla te muestro el título y descripción del PR para tu confirmación).
```

---

## Limpieza post-merge (común)

```bash
git switch <rama-base>                 # main en ambos flujos
git pull --ff-only origin <rama-base>
git branch -d feat/<short-slug>
```

---

## Nota sobre `team-large` (Placeholder)

El perfil `team-large` (trunk-based estricto, feature flags, CODEOWNERS multi-equipo) se mantiene como placeholder futuro y no se ofrece en la selección inicial (D3).
