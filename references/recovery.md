# Recuperación de commits y ramas (Referencia común)

> Guía orientativa para recuperar trabajo perdido en cualquier perfil (`solo` o `team`).
> La recuperación depende de cada caso: inspeccionar siempre antes de ejecutar y recordar que cualquier `git reset --hard` requiere confirmación bloqueante.

## 1. Si perdiste un commit local (después de `reset --hard` o cambio de rama)

```bash
git reflog                                 # 1. Ver el historial local de movimientos de HEAD
git show <sha>                             # 2. Confirmar que ese SHA tiene los cambios buscados
git cherry-pick <sha>                      # 3. Traer ese commit a la rama actual
# Alternativa si solo necesitás un archivo:
git checkout <sha> -- <path/to/file>
```

## 2. Si hiciste `git reset --hard` por error y querés volver exactamente atrás

```bash
git reflog                                 # 1. Ubicar el SHA inmediatamente anterior al reset
git reset --hard <sha>                     # 2. Restaurar la rama a ese SHA (requiere OK explícito)
```
*(Nota: funciona mientras los objetos no hayan sido eliminados por `git gc`).*

## 3. Si una rama quedó abandonada o se borró localmente

```bash
git branch -a                              # Ver ramas locales y remotas
git reflog                                 # Si se borró localmente, buscar el último SHA de esa rama
git switch -c <rama-recuperada> <sha>      # Recrear la rama apuntando a ese SHA
```

## 4. Si un `force push` pisó commits en el remoto

- **Si tenés los commits en tu `reflog` local**: podés recuperarlos con `git cherry-pick <sha>` o volver a subir la referencia correcta con `--force-with-lease` (con OK explícito).
- **Si los commits eran de otro compañero y no los tenés localmente**: pedile a esa persona que revise su `git log` / `git reflog` local antes de hacer `pull`.

## Regla general

La mejor defensa contra pérdidas es hacer **commits atómicos y frecuentes** con mensajes claros. Todo lo que llegó a ser un commit queda registrado en el `reflog` local durante semanas; lo que nunca se commiteó ni stasheó y se pisa con `reset --hard` o `checkout -- .` no se puede recuperar con Git.
