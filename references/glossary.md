# Glosario didáctico de Git

> Cargar cuando el usuario pregunte *"¿qué es X?"* o muestre dudas sobre un concepto. Explicar siempre con un ejemplo concreto del repositorio actual.

- **stash**: guardar cambios sin commitear en un "bolsillo" temporal. Útil cuando necesitás cambiar de rama o traer cambios sin perder tu trabajo en curso pero todavía no está listo para un commit.
- **rebase**: volver a aplicar los commits de tu rama encima de la punta más reciente de otra rama (como `origin/main`). Deja una historia lineal sin commits de merge intermedios, pero cambia los SHA de tus commits.
- **rebase sobre tu propia rama (en `team`)**: práctica estándar antes de abrir un PR para asegurarte de que tu código funciona sobre el último `main`. Si ya habías subido tu rama, después del rebase necesitás `git push --force-with-lease`.
- **force push (`--force`)**: sobreescribir la historia en el remoto sin verificar qué había ahí. Peligroso: si alguien más subió commits a esa rama, los borrás.
- **`--force-with-lease`**: versión segura del force push. Git verifica primero que nadie haya subido commits nuevos a la rama remota desde tu último `fetch`; si alguien subió algo, rechaza el push en lugar de pisarlo.
- **amend (`git commit --amend`)**: modificar el último commit (su mensaje o sus archivos). Muy útil antes de hacer push; riesgoso después de haberlo subido.
- **PR (Pull Request) / MR (Merge Request)**: propuesta formal para integrar los cambios de tu rama de trabajo hacia la rama base (`main`), permitiendo revisión de código y ejecución automática de tests (CI) antes de integrar.
- **code review**: revisión que hace otro miembro del equipo sobre tu PR antes de aprobarlo y mergearlo.
- **squash merge (`--squash`)**: estrategia donde todos los commits pequeños de tu rama de feature se compactan en un único commit limpio al entrar a `main`. Es el estándar en el perfil `team`.
- **merge commit (`--merge`)**: estrategia que preserva todos los commits individuales de la rama y agrega un commit de unión. Es el estándar en el perfil `solo`.
- **cherry-pick**: copiar un commit específico de una rama y aplicarlo en tu rama actual. Muy usado para llevar un hotfix puntual o recuperar un commit del `reflog`.
- **revert (`git revert`)**: deshacer el efecto de un commit anterior creando un commit nuevo con los cambios inversos, sin reescribir la historia publicada.
- **HEAD**: puntero que indica en qué commit o rama estás parado ahora mismo.
- **origin**: nombre por defecto del repositorio remoto principal (por ejemplo, en GitHub).
- **fast-forward (`--ff-only`)**: actualización donde una rama simplemente avanza su puntero porque no tiene commits divergentes (típico al hacer `git pull --ff-only origin main`).
- **reflog**: bitácora local donde Git anota cada movimiento de `HEAD`. Es la herramienta principal para rescatar commits o ramas borradas por accidente.
- **worktree**: carpeta de trabajo adicional conectada al mismo repositorio Git, para tener dos ramas abiertas al mismo tiempo sin usar `stash`.
- **conventional commits**: estándar para escribir mensajes de commit con el formato `<tipo>(<scope>): <descripción>` (por ejemplo, `feat(orders): add cancellation endpoint`).
- **topic / feature branch**: rama de corta duración creada para resolver un único ticket o tarea y que se elimina apenas su PR es mergeado en `main`.
- **rama de integración (`develop`)**: rama donde se acumulan los cambios ya revisados de varias features, antes de liberar una versión a `main`. Conserva la historia granular de los commits, que el squash de `main` colapsa.
- **release**: publicación de una versión desde la rama de integración hacia `main`, con squash para que `main` muestre un commit por versión.
- **back-port**: aplicar a la rama de integración un commit que ya se liberó en `main` (por ejemplo, un hotfix de producción), para que el próximo release no lo revierta.
