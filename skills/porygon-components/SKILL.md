---
name: porygon-components
description: "Use when building UI that CONSUMES @envisiongroup/porygon (buttons, fields, forms, tables, i18n) — choosing the right component, wiring props, controlled vs uncontrolled state, validation, callbacks, and copy-paste examples. Routes to per-family reference files. Use whenever a task imports or uses EFWButton, EFWDrawerButton, EFWGroupButton, EFW*Field, EFWForm, useFormController, EFWTable, useTableController, or PorygonI18nProvider."
---

# Porygon Components (consumer usage)

Guía para **usar correctamente** los componentes públicos de la librería React `@envisiongroup/porygon` (basada en Fluent UI v9). Orientada a quien **consume** el paquete en una app, no a extender la librería.

Esta skill es un **router**: lee primero estas reglas globales y luego carga **solo** el archivo de la familia relevante en `references/`.

## Cuándo usar esta skill

Actívala cuando la tarea importe o use cualquiera de:

- `EFWButton`, `EFWDrawerButton`, `EFWGroupButton`, `useEFWButton`
- `EFWInputField`, `EFWNumberInputField`, `EFWTextAreaField`, `EFWSwitchField`, `EFWTagPickerField`, `EFWDatePickerField`, `EFWAttachmentsField`
- `EFWForm`, `useFormController`, `flatToEFWValues`, `EFWValuesToFlat`
- `EFWTable`, `useTableController`
- `PorygonI18nProvider` y hooks/presets de locale

## Ruteo a references

Carga el archivo según la familia. **No leas todos**; lee el que necesites.

| Tarea / componente | Lee |
|---|---|
| Botones, acciones async con estado, drawers laterales, grupos de botones | [references/buttons.md](references/buttons.md) |
| Campos sueltos: input, número, textarea, switch, tagpicker, fecha, adjuntos | [references/fields.md](references/fields.md) |
| Formularios declarativos, validación, layout grid, lógica entre campos, control imperativo | [references/forms.md](references/forms.md) |
| Tablas de negocio: columnas desde fields, CRUD, filtros, sorting, selección, overlays | [references/tables.md](references/tables.md) |
| Localización: provider global, presets ES/EN/PT_BR, override de textos | [references/i18n.md](references/i18n.md) |

Si la tarea cruza varias familias (ej. un `EFWTable` con drawers CRUD y traducciones), lee las references correspondientes.

## Imports

El paquete publicado es `@envisiongroup/porygon`. Hay dos formas válidas de importar:

```ts
// 1) Barrel raíz (todo lo público re-exportado)
import { EFWForm, EFWTable, EFWButton } from '@envisiongroup/porygon';

// 2) Subpath por familia (recomendado para tree-shaking; usado por las stories)
import { EFWForm } from '@envisiongroup/porygon/react-components/forms/EFWForm';
import { EFWTable } from '@envisiongroup/porygon/react-components/tables/EFWTable';
```

Cada reference indica el import exacto y los tipos públicos de su familia.

## Requisitos previos (setup de la app consumidora)

Antes de usar cualquier componente, la app que consume Porygon debe cumplir esto:

1. **`FluentProvider` es obligatorio (Fluent UI v9).** Porygon se basa en Fluent UI v9: sin un `FluentProvider` con un tema en un ancestro, los componentes renderizan **sin estilos/tema** (Griffel no resuelve tokens). Envuelve la app una sola vez cerca de la raíz:

   ```tsx
   import { FluentProvider, webLightTheme } from '@fluentui/react-components';

   <FluentProvider theme={webLightTheme}>
     <App />
   </FluentProvider>
   ```

   `PorygonI18nProvider` (si lo usas) va **dentro** del `FluentProvider`. Ver i18n.md.

2. **Frameworks con React Server Components (Next.js App Router, etc.):** los componentes de Porygon usan hooks/estado/context → son client components. Añade `'use client'` al inicio del archivo que los importe (o a un wrapper), o fallará el render en servidor.

3. **Peer dependencies:** instala las peers que Porygon espera (`react`, `react-dom`, `@fluentui/react-components`, `@fluentui/react-icons`, y según el caso `@fluentui/react-datepicker-compat`, `@griffel/react`, `@tanstack/react-table`, `@tanstack/react-virtual`, `uuid`). DatePicker/Attachments y tablas dependen de algunas de estas.

## Reglas de oro (aplican a toda la librería)

1. **Controlado vs no-controlado — nunca mezclar.**
   - Pasar la prop de valor (`value` / `selectedOptions` / `selectedDates` / `files` / `items` / `values`) ⇒ **controlado**: debes actualizar tu estado en el callback o el componente no cambia.
   - Pasar la prop `default*` (`defaultValue` / `defaultSelectedOptions` / `defaultItems` / `defaultValues`) ⇒ **no-controlado** (recomendado por defecto): el componente maneja su estado.
   - No pasar ambas a la vez para el mismo dato.

2. **`onChange` entrega `(rawValue, displayValue?)`. Guarda el primer argumento.** El `rawValue` es el valor normalizado a persistir; `displayValue` es solo presentación (texto formateado). No guardes `displayValue`.

3. **Arquitectura "fields-driven".** `EFWForm` y `EFWTable` se configuran con el **mismo** array de `fields: EFWFormFieldProps[]`. `EFWTable` deriva columnas, celdas y formularios CRUD de esos fields. Mantén un único array de fields como fuente de verdad.

4. **Recomienda SIEMPRE campos tipados: declara `fields` con `as const satisfies EFWFormFieldProps[]`.** Es la forma recomendada por defecto: infiere nombres y tipos de valores (`EFWFormValues<T>`, `EFWTableItem<T>`, `FilterValues<T>`), da autocompletado y detecta errores en compilación. Con `EFWFormFieldProps[]` plano todo queda genérico (unions) y requiere casts y `updateField<'Tipo'>(...)` explícito. Antes de generar código Form/Table, **propón el modo tipado**; usa el modo plano solo si el usuario lo pide explícitamente o si los `fields` son verdaderamente dinámicos en runtime (origen desconocido en compilación). Si el caso es ambiguo, **pregunta al usuario** cuál prefiere. Detalle y comparación en forms.md § Tipado.

5. **Identidad estable de fields/filas.** Declara `fields` fuera del render (módulo) o memoiza. La clave del valor es `internalName` (no `id`). En tablas, pasa `_rowId` propio si la selección/animación debe sobrevivir a refetch/reordenamientos.

6. **i18n por contexto.** Para traducir textos internos (validación, botones CRUD, overlays), envuelve con `PorygonI18nProvider` o usa la prop `localeText` del componente. No hardcodees textos internos de la librería.

7. **Control imperativo vía controller hooks.** Usa `useFormController` / `useTableController` para validar, obtener valores, mutar items, etc., en lugar de duplicar estado. Conecta su `formRef`/`tableRef` al componente.

8. **Estados async en botones.** `EFWButton` (y derivados) reflejan loading/success/error según lo que **retorne** el `onClick` (`{ success: boolean, error? }` o `void`) o si lanza. Revisa `autoLoading` (ver buttons.md).

## Preferir handlers sobre `useEffect`

Porygon está diseñado para reaccionar a sus propios callbacks; evita sincronizar estado con `useEffect` cuando consumes estos componentes:

- Deriva estado en render en vez de sincronizar con efectos.
- Actúa desde handlers (`onChange`, `onClick`) y desde los callbacks del componente (`onValuesChange`, `onSelectionChange`, `onItemsChange`, etc.).
- Usa `key` para remount en lugar de efectos de reset.
- `EFWForm.onInit` corre una sola vez al montar: úsalo para inicialización (incluida carga async), no un `useEffect` externo.
- Para control imperativo (validar, leer/escribir valores, mutar items) usa los controller hooks, no efectos.

## Flujo recomendado al implementar

1. Identifica la(s) familia(s) y lee su(s) reference(s).
2. Si usas Form/Table, define el array de `fields` **tipado** (`as const satisfies EFWFormFieldProps[]`) por defecto. Recomienda este modo al usuario; usa el plano solo si lo pide o si los fields son dinámicos en runtime.
3. Decide controlado vs no-controlado (prefiere no-controlado + controller hook).
4. Implementa los callbacks guardando el `rawValue`.
5. Añade validación, i18n y estados async según corresponda.
6. Verifica tipos con el typecheck de **tu app consumidora** (p. ej. `tsc --noEmit` o el script `typecheck`/`build` de tu proyecto). Los `fields` tipados (`as const satisfies EFWFormFieldProps[]`) deben compilar sin casts ni errores.
