# git-mentor

Skill didáctica de Git para agentes de IA. Antes de tocar el repositorio, pregunta si trabajas **solo** o **en equipo**, muestra un **plan numerado con el porqué de cada paso** y bloquea las acciones destructivas hasta recibir tu confirmación explícita.

## Qué es

`git-mentor` es una skill (`SKILL.md` + referencias bajo demanda) para agentes de código compatibles con el formato de skills, como Antigravity, Claude Code o Gemini CLI. Convierte al agente en un mentor de Git: no ejecuta comandos a ciegas, sino que explica qué va a hacer y por qué, y recuerda las convenciones de cada proyecto.

Se activa con palabras como `git`, `commit`, `branch`, `PR`, `merge`, `rebase`, `stash`, `status`, `log` o `diff`.

## Qué hace

- **Selección de perfil obligatoria**: antes de cualquier operación que modifique el repositorio, confirma el perfil.
  - `solo`: rama de trabajo de larga vida (`dev`), un PR grande hacia `main` con merge commit y disciplina anti-huérfanos.
  - `team`: guía para devs juniors, con ramas cortas por tarea (`feat/`, `fix/`, `chore/`), rebase, PRs pequeños y review obligatoria. Admite flujo con rama `develop` o directo a `main`.
- **Plan + porqué**: cada tarea se presenta como una lista de comandos exactos, cada uno con una sola frase de justificación. Se aprueba el plan completo con un solo OK.
- **Protección contra acciones destructivas**: `push --force`, `reset --hard`, `branch -D`, push directo a producción y reescritura de historia compartida requieren una confirmación aparte. En `team` solo se permite `--force-with-lease` sobre la propia rama.
- **Conventional Commits**: valida los mensajes, bloquea subjects vagos (`WIP`, `update`, `fix`) y nunca agrega `Co-Authored-By`.
- **Guardia determinista de `.gitignore`**: en `git status` y antes de cada commit, audita el repo con comandos nativos de Git (`git ls-files` y `git check-ignore`) contra un diccionario estático. El resultado no depende del modelo que ejecute la skill.
  - Si no existe `.gitignore`, propone uno a partir del stack detectado (Node, Python, .NET, Go, Rust).
  - Si ya existe, agrega solo los patrones que faltan.
  - Si hay archivos ya trackeados que no deberían estarlo, incluye `git rm --cached` en el mismo plan. Si el archivo es un secreto que ya entró al historial, avisa que hay que rotar las credenciales.
- **Memoria por proyecto** (opcional): si hay [Engram](https://github.com/Gentleman-Programming/engram) disponible, guarda el perfil y las convenciones de cada repositorio para no volver a preguntar.
- **Carga bajo demanda**: solo se lee la referencia del perfil activo, para gastar menos tokens.

## Estructura

```text
git-mentor/
├── SKILL.md                 # núcleo: perfil, plan + porqué, reglas, acciones bloqueantes
├── DESIGN.md                # decisiones de diseño (D1-D7)
└── references/
    ├── profile-solo.md      # perfil solo
    ├── profile-team.md      # perfil team (reglas comunes)
    ├── flow-develop.md      # team con rama develop
    ├── flow-main.md         # team directo a main
    ├── gitignore.md         # guía de .gitignore
    ├── audit.gitignore      # diccionario de patrones auditado por Git
    ├── recovery.md          # recuperación de commits y ramas perdidas
    └── glossary.md          # glosario didáctico
```

## Requisitos

- [Git](https://git-scm.com/) instalado y disponible en el `PATH`. Es la única dependencia: la guardia de `.gitignore` no necesita Bash, Node ni Python, así que funciona igual en Windows, Linux y macOS.
- Un agente que soporte skills en formato `SKILL.md`.
- Opcional: [GitHub CLI](https://cli.github.com/) para que el agente abra PRs, y Engram para la memoria por proyecto.

## Instalación

Una skill es una carpeta con un `SKILL.md`. Instalarla consiste en clonar este repositorio dentro del directorio de skills de tu agente.

1. Clona el repositorio con el nombre de carpeta `git-mentor`, porque debe coincidir con el campo `name` de `SKILL.md`:

   **Windows (PowerShell)**

   ```powershell
   git clone https://github.com/Srozasc/git-mentor-skill.git "$HOME\.agents\skills\git-mentor"
   ```

   **Linux / macOS**

   ```bash
   git clone https://github.com/Srozasc/git-mentor-skill.git ~/.agents/skills/git-mentor
   ```

2. Si tu agente usa otro directorio de skills, cambia la ruta de destino:

   | Agente | Directorio de skills habitual |
   |---|---|
   | Antigravity / skills compartidas | `~/.agents/skills/` |
   | Claude Code | `~/.claude/skills/` |
   | Gemini CLI | `~/.gemini/skills/` |

   Para instalarla solo en un proyecto, usa la carpeta `skills/` equivalente dentro de ese repositorio.

3. Reinicia el agente o abre una sesión nueva para que detecte la skill.

### Actualizar

```bash
git -C <ruta-a-la-skill> pull
```

### Verificar la instalación

Abre un repositorio git y pide algo como *"quiero hacer un commit"*. El agente debe preguntar primero si trabajas en modo `solo` o `team`.

## Uso rápido

```text
Tú:     Quiero guardar mis cambios del login.
Agente: ¿Trabajas solo o en equipo?
Tú:     Solo.

Agente: Tarea: guardar el fix de validación y subirlo
        Perfil: solo · Rama actual: dev

        Plan:
        1. git add src/login/
           → Prepara solo los archivos del cambio para no mezclar otros.
        2. git commit -m "fix(auth): handle empty password with explicit error"
           → Deja un commit atómico con scope claro.
        3. git push origin dev
           → Respalda el avance sin tocar producción.

        ¿Ejecuto el plan?
```

## Documentación

- [SKILL.md](SKILL.md): instrucciones completas que lee el agente.
- [DESIGN.md](DESIGN.md): diseño, diagramas y decisiones cerradas.

## Licencia

Apache-2.0.
