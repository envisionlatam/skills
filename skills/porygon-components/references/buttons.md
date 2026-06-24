# Buttons — `EFWButton`, `EFWDrawerButton`, `EFWGroupButton`

**Cuándo usar:** acciones de usuario sobre Fluent UI con comportamiento enriquecido. `EFWButton` para acciones (normalmente async) que reflejan estados loading/success/error; `EFWDrawerButton` para un botón que abre un panel lateral con contenido/footer dinámicos; `EFWGroupButton` para renderizar grupos de botones heterogéneos desde un array declarativo con inyección de parámetros. Para toggles/checkbox usa campos (`EFWSwitchField`), no botones.

Los 3 componentes y `useEFWButton` se exportan desde la raíz `@envisiongroup/porygon` y también vía subpath `@envisiongroup/porygon/react-components/buttons/*`.

---

## EFWButton

```tsx
import {
  EFWButton, useEFWButton,
  type EFWButtonProps, type EFWButtonActionResult,
  type EFWButtonHandle, type UseEFWButtonOptions,
} from '@envisiongroup/porygon';
```

**Cuándo usar:** botón sobre Fluent `Button` con estados automáticos `loading / success / error` para acciones (normalmente async). Úsalo cuando el `onClick` ejecuta una operación cuyo resultado debe reflejarse visualmente.

### Props (extiende `Omit<ButtonProps, "onClick">`)

| Prop | Tipo | Default | Descripción |
|---|---|---|---|
| `onClick` | `(event) => Promise<EFWButtonActionResult> \| EFWButtonActionResult` | — | Acción al click; su retorno controla el estado final. |
| `appearance` | `'primary'\|'secondary'\|'outline'\|'subtle'\|'transparent'` | `"secondary"` | Apariencia Fluent. |
| `icon` | `ButtonProps['icon']` | — | Icono Fluent. Necesario para `collapseLabel`. |
| `disabled` | `boolean` | — | Deshabilita el botón. |
| `disableOnLoading` | `boolean` | `true` | Deshabilita durante `loading` (evita doble click). |
| `autoLoading` | `boolean` | `false` | Si `true`, un `onClick` async entra a `loading` y procesa el resultado al resolver. **Con `false`, un onClick async NO muestra loading/success/error.** |
| `successConfig` | `{ onSuccess?, timeout?=2000, text? }` | `{}` | Config del estado de éxito. |
| `errorConfig` | `{ onError?(e), timeout?, text? }` | `{}` | Config del estado de error. |
| `tooltips` | `{ idle?, loading?, success?, error? }` (strings) | — | Tooltips por estado. |
| `showStatusIcons` | `boolean` | `true` | Spinner/Check/ErrorCircle según estado (reemplaza `icon`). |
| `autoReset` | `boolean` | `true` | Vuelve a `idle` tras success/error según timeout. |
| `collapseLabel` | `boolean \| "xs"\|"sm"\|"md"\|"lg"` | — | Oculta el label (solo icono) vía Container Query. Requiere `icon` y un ancestro con `container-type: inline-size`. |
| `labels` | `EFWButtonLabels` | i18n | `loadingText`, `defaultErrorMessage`, `operationFailedMessage`, `unknownErrorMessage`, `parseErrorMessage`. |

```ts
type LoadingState = "idle" | "loading" | "success" | "error";
type EFWButtonActionResult = { success: boolean; error?: string | Error } | void;
```

### Patrón async (qué retornar en `onClick`)

- `{ success: true }` → estado `success` (muestra `successConfig.text`).
- `{ success: false, error: 'msg' | Error }` → estado `error`.
- `undefined` / `void` → solo detiene `loading`.
- `throw` → se captura y pasa a `error`.

### Hook `useEFWButton(options?): EFWButtonHandle`

Para controlar el estado del botón desde fuera o con UI propia.

`EFWButtonHandle`: `currentState`, `isLoading`, `error`, `reset()`, `setLoading()`, `stopLoading()`, `setSuccess()`, `setError(err)`, y `executeAsync(fn)` que envuelve la operación (maneja loading/success/error y **re-lanza** el error).

### Gotchas

- **`autoLoading` por defecto es `false`.** Para ver loading/success/error con `onClick` async, pásalo explícito (`EFWDrawerButton` lo activa solo cuando hay `onClick`).
- Con `onClick` **síncrono** el resultado se procesa siempre.
- `collapseLabel` no hace nada sin `icon` ni sin ancestro `container-type: inline-size`.
- `showStatusIcons=true` reemplaza tu `icon` durante los estados.
- `timeout: 0` desactiva el auto-reset de ese estado.

### Ejemplos

```tsx
// Acción async con resultado tipado
const handleSave = async (): Promise<EFWButtonActionResult> => {
  try {
    await api.save();
    return { success: true };
  } catch {
    return { success: false, error: 'Error al guardar los datos' };
  }
};

<EFWButton
  appearance="primary"
  autoLoading
  onClick={handleSave}
  successConfig={{ text: '¡Guardado!' }}
  errorConfig={{ text: 'Error' }}
  tooltips={{ idle: 'Haz clic para guardar' }}
>
  Guardar
</EFWButton>
```

```tsx
// Control manual con useEFWButton
const btn = useEFWButton({ successTimeout: 3000, errorTimeout: 4000 });
const run = () =>
  btn.executeAsync(async () => {
    await api.process();
    return { success: true };
  });
// btn.currentState / btn.isLoading / btn.error / btn.reset()
```

---

## EFWDrawerButton

```tsx
import {
  EFWDrawerButton,
  type EFWDrawerButtonProps, type EFWDrawerButtonClickHandler,
} from '@envisiongroup/porygon';
```

**Cuándo usar:** `EFWButton` + `useDrawer`: un botón que abre un panel lateral con contenido/footer dinámicos (detalles, formularios, flujos contextuales).

### Props (extiende `Omit<EFWButtonProps, 'onClick'>`)

Hereda **todas** las props de `EFWButton` (appearance, icon, disabled, successConfig, errorConfig, tooltips, labels, collapseLabel, etc.) excepto `onClick`, más:

| Prop | Tipo | Default | Descripción |
|---|---|---|---|
| `onClick` | `(event, openDrawer) => Promise<EFWButtonActionResult> \| EFWButtonActionResult` | — | Recibe `openDrawer` para abrir/configurar el drawer. Mismo contrato de retorno que `EFWButton`. |
| `drawerConfig` | `UseDrawerProps` | `{}` | Config inicial del drawer (sobrescribible al llamar `openDrawer(options)`). |
| `autoOpenDrawer` | `boolean` | `true` | Si NO hay `onClick`, abre el drawer automáticamente al click. |

**`DrawerOpenOptions` (config del drawer):**

| Campo | Tipo | Default | Descripción |
|---|---|---|---|
| `title` | `ReactNode` | `''` | Título. |
| `content` | `ReactNode \| ((closeDrawer) => ReactNode)` | — | Cuerpo. Si es función recibe `closeDrawer`. |
| `buttons` | `EFWGroupButtonConfig<DrawerCloseHandler>[]` | `[]` | Footer. **Su param inyectado es `closeDrawer`** → `onClick: (event, closeDrawer) => ...`. |
| `size` | `'small'\|'medium'\|'large'\|'full'` | `'small'` | Tamaño. |
| `position` | `'start'\|'end'\|'bottom'` | `'end'` | Posición. |
| `modalType` | `'modal'\|'non-modal'\|'alert'` | `'modal'` | Modalidad. |
| `deferContentRendering` | `boolean` | `false` | Difiere render (muestra loader). |
| `drawerOnOpen` / `drawerOnClose` | `() => void \| Promise<void>` | — | Callbacks. |

`openDrawer` además expone `.updateDrawer(updates, { force? })` para mutar el drawer abierto sin reabrir.

### Notas y gotchas

- `EFWDrawerButton` activa `autoLoading` automáticamente cuando hay `onClick` (`autoLoading ?? !!onClick`).
- Sin `onClick`, el drawer se abre con `drawerConfig` (por `autoOpenDrawer`). Con `onClick`, **tú** debes llamar `openDrawer()`; si no, no se abre.
- Los `buttons` del footer reciben `closeDrawer` como 2º argumento: `onClick: (_, closeDrawer) => closeDrawer()`.
- El drawer se renderiza en portal (`document.body`).

### Ejemplos

```tsx
// Drawer estático automático (sin onClick)
<EFWDrawerButton
  appearance="primary"
  drawerConfig={{ title: 'Detalle', content: 'Contenido del panel…' }}
>
  Abrir panel
</EFWDrawerButton>
```

```tsx
// Carga async + footer con cierre
<EFWDrawerButton
  appearance="primary"
  successConfig={{ text: '¡Perfil cargado!' }}
  errorConfig={{ text: 'Error al cargar' }}
  onClick={async (_event, openDrawer) => {
    try {
      const userData = await fetchUserData('123');
      openDrawer({
        title: `Perfil de ${userData.name}`,
        content: <UserProfileDisplay data={userData} />,
        buttons: [
          { kind: 'Button', props: {
            children: 'Cerrar', appearance: 'secondary',
            onClick: (_, closeDrawer) => closeDrawer(),
          } },
        ],
      });
      return { success: true };
    } catch (error) {
      return { success: false, error: (error as Error).message };
    }
  }}
>
  Cargar perfil
</EFWDrawerButton>
```

---

## EFWGroupButton

```tsx
import {
  EFWGroupButton,
  type EFWGroupButtonProps, type EFWGroupButtonConfig,
} from '@envisiongroup/porygon';
```

**Cuándo usar:** renderiza un grupo de botones heterogéneos (Fluent + EFW) desde un array declarativo, con inyección automática de parámetros contextuales (`onClickParams`).

### Props (genérico `EFWGroupButton<T extends Record<string, any>>`)

| Prop | Tipo | Req. | Default | Descripción |
|---|---|---|---|---|
| `buttons` | `EFWGroupButtonConfig<T>[]` | **Sí** | — | Configuración declarativa de cada botón. |
| `onClickParams` | `T` | No | — | Objeto inyectado como último argumento a los handlers que lo declaren. |
| `gap` | `string \| number` | No | `'8px'` | Espaciado. |
| `style` / `className` | — | No | — | Contenedor (`display:flex; width:100%`). Para wrap: `flexWrap:'wrap'`. |

**`EFWGroupButtonConfig<T>`** — unión discriminada por `kind`:

| `kind` | `props` | Handler |
|---|---|---|
| `'Button'` | `Omit<ButtonProps,'onClick'>` | `onClick: NormalClickHandler<T>` |
| `'CompoundButton'` | `Omit<CompoundButtonProps,'onClick'>` | `NormalClickHandler<T>` |
| `'MenuButton'` | `Omit<MenuButtonProps,'onClick'>` + `menuTriggers?` | sin onClick propio; usa `menuTriggers[].onClick` |
| `'SplitButton'` | `Omit<SplitButtonProps,'onClick'>` + `menuTriggers?` | `onClick` (primaria) + `menuTriggers[].onClick` |
| `'EFWButton'` | `Omit<EFWButtonProps,'onClick'>` | `onClick: EFWClickHandler<T>` (retorna `EFWButtonActionResult`) |
| `'EFWDrawerButton'` | `Omit<EFWDrawerButtonProps,'onClick'>` | `onClick: (event, openDrawer, params) => ...` |

### Inyección de parámetros

El componente inspecciona la **aridad** (`fn.length`) del `onClick`: si declara ≥2 args (≥3 para `EFWDrawerButton`, por `openDrawer`) **y** `onClickParams` está definido → invoca `(event, params)` / `(event, openDrawer, params)`.

### Gotchas

- La inyección depende de la **firma declarada**: usar rest/default params (`(...args)`) rompe la detección. Declara explícitamente `(event, params)`.
- `MenuButton` no tiene `onClick` propio; usa `menuTriggers`.
- `kind` inválido renderiza `labels.invalidButtonText`.
- `EFWDrawerButton` se carga con `React.lazy` internamente.

### Ejemplo

```tsx
interface ActionParams { userId: string; userName: string }

<EFWGroupButton<ActionParams>
  gap="12px"
  style={{ flexWrap: 'wrap' }}
  onClickParams={{ userId: '1986', userName: 'Samus Aran' }}
  buttons={[
    { kind: 'Button', props: {
      children: 'Ver', appearance: 'secondary',
      onClick: (_event, params) => console.log(params.userId),
    } },
    { kind: 'EFWButton', props: {
      children: 'Guardar',
      onClick: (_event, params) => { /* ... */ return { success: true }; },
    } },
    { kind: 'EFWDrawerButton', props: {
      children: 'Detalle',
      onClick: (_event, openDrawer, params) =>
        openDrawer({ title: params.userName, content: 'Panel' }),
    } },
  ]}
/>
```
