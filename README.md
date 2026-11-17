# Skills de Claude de Corestack

Respaldo de las skills personales de Claude Code (`~/.claude/skills`).

## Qué incluye

- `skills/`: 98 skills propias, cada una con su `SKILL.md`.

## Qué no incluye

- Las skills instaladas desde repositorios públicos (carpetas que son enlaces a `~/.agents/skills`). Se reinstalan con `~/.agents/.skill-lock.json`, que guarda el repositorio de origen de cada una.
- `notebooklm`: es una copia de `github.com/PleasePrompto/notebooklm-skill` (480 MB).
- `synced`: skills que sincroniza la aplicación de Claude por su cuenta.

## Restaurar

```bash
cp -r skills/* ~/.claude/skills/
```

## Actualizar el respaldo

```bash
cd ~/.claude/skills
for d in */; do d=${d%/}; [ -L "$d" ] || [ "$d" = notebooklm ] || [ "$d" = synced ] || cp -r "$d" ~/claude-skills/skills/; done
cd ~/claude-skills && git add -A && git commit -m "Actualiza skills" && git push
```

## Licencias

Este repositorio es público. Algunas skills no son originales de Corestack y conservan su licencia:

- `frontend-design`: Apache 2.0 (ver `skills/frontend-design/LICENSE.txt`).
- `prompt-engineer`: MIT (ver `skills/prompt-engineer/LICENSE`).
- `database-schema-designer`: MIT + Commons Clause (no se puede vender).

Las demás no declaran licencia en su `SKILL.md`. Si eres autor de alguna y quieres que se retire, abre un issue.

No guardar llaves ni contraseñas dentro de las skills: el historial de git es permanente.
