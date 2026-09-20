---
description: Crea un git worktree en .worktrees/<nombre>
agent: build
---

Contexto recibido: $ARGUMENTS

Genera <nombre-del-worktree> y ejecuta un único comando. Nada más.

Reglas:

1. Convierte `$ARGUMENTS` a kebab-case seguro: minúsculas, espacios y `_` a `-`, elimina todo excepto `a-z 0-9 -`, colapsa `-` repetidos, recorta `-` inicial/final. Ejemplo: `Mi Nueva Feature` -> `mi-nueva-feature`.
2. Si los argumentos son muy largos, simplificalos a un nombre significativo.
3. Si el contexto viene vacío, pide el nombre y detente sin ejecutar nada.
4. Ejecuta exactamente este único comando con bash, sin cambiar de directorio:
   `git worktree add ".worktrees/<nombre-generado>"`

Prohibido: `cd`, `mkdir`, `git status`, `git branch`, `git fetch`, listar/leer/crear archivos, o cualquier segundo comando.
