# Guardia de `.gitignore` (carga bajo demanda)

> Referencia cargada por `SKILL.md` **únicamente** cuando la auditoría determinista detecta archivos expuestos/trackeados que deberían ignorarse, o cuando el repositorio no tiene `.gitignore`.

## 1. Interpretación de la auditoría determinista

1. **Untracked sospechosos**:
   - `git ls-files --others --ignored --exclude-from="<skill>/references/audit.gitignore" --directory`
   - Si el repo **no tiene** `.gitignore`, todas las rutas devueltas están expuestas.
   - Si el repo **sí tiene** `.gitignore`, pasar esas rutas a `git check-ignore -n -v -- <rutas...>`. Solo las líneas que empiezan con `::` están **realmente expuestas** (las que muestran `.gitignore:...` ya están cubiertas o exceptuadas con `!`).
2. **Ya trackeados por error**:
   - `git ls-files --cached --ignored --exclude-standard --exclude-from="<skill>/references/audit.gitignore"`
   - Toda ruta devuelta está en el índice de Git y necesita `git rm --cached`.

---

## 2. Catálogo de "porqués" (una frase por categoría)

Usar la frase que corresponde al presentar el paso en el plan:

| Categoría | Patrones típicos | Porqué (una línea) |
|---|---|---|
| `[SECRETS]` | `.env`, `.env.*`, `*.pem`, `*.key`, `credentials*.json` | Evita exponer credenciales, claves privadas o variables locales en el historial del repositorio. |
| `[DEPS]` | `node_modules/`, `.venv/`, `venv/`, `__pycache__/`, `.pnpm-store/` | Evita subir dependencias pesadas que cada persona reinstala desde el archivo de lock. |
| `[BUILD]` | `dist/`, `build/`, `out/`, `target/`, `obj/`, `.next/`, `coverage/` | Evita ensuciar el historial con artefactos compilados o generados que se reconstruyen en cada build. |
| `[OS_IDE]` | `.DS_Store`, `Thumbs.db`, `nul`, `.idea/`, `.vscode/*` | Evita subir basura del sistema operativo o configuraciones locales de tu editor. |
| `[LOGS_TMP]` | `*.log`, `npm-debug.log*`, `*.tmp`, `*.bak`, `*.tar.gz` | Evita versionar logs de ejecución o respaldos temporales sin valor para el código. |

---

## 3. Estrategia híbrida: crear vs. editar `.gitignore`

### Caso A — NO existe `.gitignore` en la raíz del repo

Armar el archivo nuevo uniendo **únicamente**:
1. El **bloque base universal**, más
2. Los **bloques literales de los stacks detectados** por archivos marcadores presentes en la raíz, más
3. Cualquier patrón puntual reportado por la auditoría que no estuviera en esos bloques.

**Bloque base universal (siempre presente):**
```gitignore
# OS & Editor
.DS_Store
Thumbs.db
desktop.ini
nul
.idea/
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json

# Secrets
.env
.env.*
!.env.example
!.env.sample
!.env.template
*.pem
*.key

# Logs & Temp
*.log
*.tmp
*.bak
```

**Tabla cerrada de detección de stack (no inventar reglas de memoria):**

| Archivo marcador en el repo | Bloque literal a agregar |
|---|---|
| `package.json` | `node_modules/`<br>`.pnpm-store/`<br>`dist/`<br>`build/`<br>`coverage/`<br>`*.tsbuildinfo`<br>`npm-debug.log*`<br>`yarn-error.log*`<br>`pnpm-debug.log*` |
| `next.config.*` | `.next/`<br>`out/` |
| `nuxt.config.*` | `.nuxt/`<br>`.output/` |
| `turbo.json` | `.turbo/` |
| `pyproject.toml` o `requirements.txt` | `.venv/`<br>`venv/`<br>`__pycache__/`<br>`*.pyc`<br>`.pytest_cache/`<br>`.mypy_cache/`<br>`.ruff_cache/` |
| `*.csproj` o `*.sln` | `bin/`<br>`obj/`<br>`.vs/` |
| `go.mod` | `bin/`<br>`coverage.out` |
| `Cargo.toml` | `target/` |

### Caso B — YA existe `.gitignore` en el repo (modo quirúrgico)

- **NO** reescribir ni reordenar el `.gitignore` existente.
- Agregar al final **únicamente** los patrones necesarios para cubrir las rutas expuestas (`::`).
- Si se agrega `.env*`, incluir siempre `!.env.example` debajo para no ocultar plantillas compartidas.

---

## 4. Archivos ya trackeados (`--cached`) y protocolo de secretos

Si `git ls-files --cached --ignored ...` devolvió rutas:

1. Agregar al plan el paso previo:
   - Archivo suelto: `git rm --cached <ruta>`
   - Carpeta completa: `git rm -r --cached <carpeta>/`
2. **Verificación obligatoria si el archivo es de `[SECRETS]`**:
   Ejecutar en solo lectura:
   ```bash
   git log -1 -- <ruta>
   ```
   - **Si no devuelve nada**: el archivo solo estaba en staging; `git rm --cached` lo limpia por completo.
   - **Si devuelve un commit**: el archivo **ya entró al historial**. Mantener `git rm --cached` en el plan y mostrar esta alerta explícita antes de pedir OK:
     > ⚠️ **Atención**: `git rm --cached` saca `<ruta>` de los próximos commits, pero **su contenido sigue grabado en el historial de Git**. Si ya hiciste `push`, **rotá o revocá esas credenciales ahora mismo**. Si todavía no hiciste `push`, avisame y lo sacamos del historial local antes de subir.

---

## 5. Falsos positivos (cuando el usuario SÍ quiere versionar el archivo)

Si el usuario confirma que un archivo detectado debe quedar en el repo (por ejemplo, un fixture `sample.log` o un certificado de prueba):
- Proponer agregar la excepción `!<ruta-exacta>` al `.gitignore` del proyecto. Como los comandos de auditoría respetan `--exclude-standard`, no vuelve a disparar alertas en futuras sesiones.

---

## 6. Ejemplo de Plan + porqué integrado

```text
Tarea: Commitear el endpoint de pagos
Perfil: team · Flujo: develop · Rama actual: feat/payment-webhook

Detecté que `.env.local` está sin ignorar y que `dist/` quedó trackeado en el índice. Los corrijo dentro del plan:

Plan:
1. Agregar `.env`, `.env.*` (con `!.env.example`) y `dist/` a `.gitignore`
   → Evita exponer credenciales locales y subir artefactos compilados al repositorio.
2. git rm -r --cached dist/
   → Saca la carpeta compilada del seguimiento de Git sin borrarla de tu disco local.
3. git add .gitignore src/payments/ && git commit -m "feat(payments): add stripe webhook handler"
   → Registra tu feature junto con las reglas de exclusión para que el repo quede protegido.

¿Ejecuto el plan?
```
