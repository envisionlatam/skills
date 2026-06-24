# envisionlatam / skills

Catálogo de [Agent Skills](https://agentskills.io/specification) de Envision.
Compatibles con **pi**, **Claude Code**, **Codex**, **opencode**, **Cursor** y
otras herramientas que soporten el formato `SKILL.md`.

> Repositorio generado/actualizado automáticamente desde el monorepo Porygon.
> No edites manualmente; los cambios se publican con `pnpm run publish:skill`.

## Skills disponibles

| Skill | Descripción | Versión |
|---|---|---|
| [`porygon-components`](skills/porygon-components/SKILL.md) | Uso correcto de los componentes de `@envisiongroup/porygon` (buttons, fields, forms, tables, i18n). | `1.0.3` |

## Instalación

### Con skills.sh (cualquier herramienta)

```bash
# última versión (rama main)
npx skills add envisionlatam/skills --skill porygon-components

# fijar una versión concreta
npx skills add envisionlatam/skills#porygon-components-v1.0.3 --skill porygon-components

# a herramientas específicas
npx skills add envisionlatam/skills --skill porygon-components -a claude-code -a codex -a opencode
```

### Con pi

```bash
pi install git:github.com/envisionlatam/skills           # usuario
pi install -l git:github.com/envisionlatam/skills        # proyecto (equipo)
```

## Versionado

Cada release crea un tag `porygon-components-vX.Y.Z`. Usa `#<tag>` para fijar versión.
