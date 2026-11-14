# Perfil `solo` (antes `solo-freelancer`)

> Referencia cargada por `SKILL.md` cuando el proyecto usa `profile: solo` (o `solo-freelancer` en memorias anteriores).
> **Importante**: las reglas de este archivo aplican **únicamente** al modo `solo`.

## Configuración y reglas del perfil

Pensado para trabajo individual: freelancer, asesor, proyectos personales.

- **work_branch**: `dev` (default, configurable por proyecto).
- **production_branch**: `main` (default, configurable por proyecto).
- **commit_strategy**: todos los commits juntos en un solo PR al final del paquete de mejoras.
- **merge_strategy**: `--merge` (merge commit, preserva la historia de commits conventional).
- **Restricciones**:
  - **NO** crear ramas adicionales durante el trabajo en una feature.
  - Los **bugs no relacionados** encontrados durante una feature se arreglan en la rama de trabajo actual (`dev`).
  - La rama de trabajo **no cambia** hasta cerrarse con PR a producción (`main`).

**Caso típico**: `main` tiene la versión en producción. El cliente pide un paquete de mejoras. Esas mejoras se desarrollan en `dev`. Cuando todo está verde, se abre un PR de `dev` a `main` con merge commit.

---

## Ejemplo de Plan + porqué en modo `solo`

```text
Tarea: Guardar el fix de validación en la rama de trabajo y subirlo
Perfil: solo · Rama actual: dev

Plan:
1. git add src/fdh/validator.ts
   → Prepara únicamente el archivo del fix para no mezclar cambios ajenos.
2. git commit -m "fix(fdh): handle empty sigla column with explicit BadRequestException"
   → Deja un commit atómico con scope claro por si hay que auditarlo o revertirlo antes del PR.
3. git push origin dev
   → Respalda el avance en el remoto sin tocar producción.

¿Ejecuto el plan?
```

---

## Anti-huérfanos: disciplina de commits en `solo`

Con ramas de larga vida (`dev`), los commits se acumulan antes de ir a producción. Esta disciplina evita que un commit pierda contexto.

### Las cuatro preguntas pre-commit

Antes de ejecutar un commit, validar:

1. **¿Es una unidad lógica completa?** No `WIP`, `fix typo`, `more changes`, `actualizar`.
2. **¿Tiene scope claro?** `feat(fdh)`, `fix(bimestre)`, `chore(deps)`.
3. **¿El subject explica el qué por sí solo?** 50 chars ideal, 72 máximo. El **por qué** va en el body.
4. **¿Si quedara huérfano sería entendible?** Si sin contexto no se entiende qué hace, agregar body o esperar a completar la unidad lógica.

### Cuerpo del commit (cuando el subject no alcanza)

```text
<subject>

<por qué del cambio, no el qué — el qué ya está en el subject>

Refs: <contexto adicional, ticket, conversación>
Refs-SP: <stored procedure afectado, si aplica>
```

Ejemplo:

```text
feat(fdh): add upload endpoint with ExcelJS streaming

Replaces xlsx with ExcelJS for memory efficiency on large
uploads (>5MB). The previous in-memory buffer approach was
crashing on production with 200+ row spreadsheets.

Refs: discovered during Q3 client load test
Refs-SP: sp_reporte_fdh
```

**`Refs:` y `Refs-SP:` NO son `Co-Authored-By`** — son metadata de auditoría.

### Tags de hito

Sugerir tags cuando se cierra un bloque importante o antes de auditar el PR:

- `pre-pr-audit` — antes de empezar el pre-PR audit.
- `ready-for-review` — después del pre-PR audit, listo para abrir PR.
- `vX.Y-wip` — hito de versión en desarrollo.
- `checkpoint-<descripcion>` — punto de recuperación manual.

```bash
git tag pre-pr-audit dev
git push origin pre-pr-audit
```

### Pre-PR audit automático

Antes de abrir el PR de `dev` a `main`, ejecutar:

```bash
git log --oneline main..dev
```

Revisar commit por commit con el usuario:
- ¿El subject sigue siendo relevante?
- ¿El scope es claro?
- ¿El cambio todavía aplica al estado actual del proyecto?

Si algún commit **no debe entrar al PR**, excluirlo con `git revert` (nunca con `git reset`):

```bash
git revert <commit-sha>
```

### Anti-patrones explícitos en `solo`

En el perfil `solo` **NUNCA** hacer:

- **Squash de N commits en 1**: pierde granularidad de todo el paquete de trabajo en `dev`.
- **`git reset --hard HEAD~N` sin stash previo**: borra commits locales.
- **Force push** sin OK explícito.
- **Amend de commits ya pusheados**.
- **Crear rama extra para un bug no relacionado**: en `solo`, `dev` lo absorbe.

*(Si hace falta recuperar un commit perdido, leer `references/recovery.md`).*

---

## PR Workflow en `solo`

1. Ejecutar el **Pre-PR audit** (`git log --oneline main..dev`) y la **Pre-PR checklist** de `SKILL.md`.
2. Proponer título y descripción agrupando los commits por scope:
   ```markdown
   ## Cambios incluidos

   ### feat(fdh)
   - upload endpoint + 6 specs
   - frontend page + components

   ### fix(bimestre)
   - SP column-name corrections
   ```
3. Pedir confirmación del título y descripción con `ask_user_choice`.
4. Abrir el PR tras el OK (D5) y, cuando el usuario pida mergear, usar `--merge` (merge commit) tras confirmación.
