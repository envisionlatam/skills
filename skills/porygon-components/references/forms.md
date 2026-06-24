# Forms — `EFWForm`

```tsx
import {
  EFWForm, useFormController, flatToEFWValues, EFWValuesToFlat,
  type EFWFormProps, type EFWFormFieldProps, type EFWFormValues,
  type EFWFormMethods, type EFWFormRef, type EFWFieldLogic,
} from '@envisiongroup/porygon/react-components/forms/EFWForm';
// también: import { EFWForm, useFormController } from '@envisiongroup/porygon';
```

**Cuándo usar:** formulario dinámico/declarativo basado en un array de `fields` (texto, número, fecha, choice, switch, adjuntos), con validación, layout grid, lógica condicional entre campos, secciones, mensajes y control imperativo. Soporta **uncontrolled** (default, recomendado) y **controlled**.

---

## 1. Tipos de campo → tipo de valor (`EFWFormFieldValueMap`)

`EFWFormFieldType = 'Text' | 'Number' | 'Choice' | 'Note' | 'Boolean' | 'MultiChoice' | 'Date' | 'Attachments'`

| `typeAsString` | Tipo de `value` | Notas |
|---|---|---|
| `Text` | `string` | Input simple |
| `Number` | `number \| null` | Numérico |
| `Choice` | `EFWTagPickerFieldValue[]` | Selección única (1 item) |
| `Note` | `string \| null` | Textarea |
| `Boolean` | `EFWSwitchFieldValue` | Switch |
| `MultiChoice` | `EFWTagPickerFieldValue[]` | Multi; requiere `multiple: true` |
| `Date` | `EFWDatePickerFieldValue[] \| undefined` | Array de `Date \| string` |
| `Attachments` | `EFWAttachmentsFieldValue[] \| undefined` | Adjuntos |

---

## 2. Estructura de un field declarativo (`EFWFormFieldProps`)

Discriminado por `typeAsString`. Props base obligatorias:

```ts
interface EFWFormFieldBaseProps<T> {
  readonly id: string;            // identificador único
  readonly typeAsString: T;       // discriminante de tipo
  readonly internalName: string;  // CLAVE del valor en EFWFormValues
  value?: EFWFormFieldValueMap[T];
}
```

Props compartidas frecuentes (del Field base): `title`, `required`, `editable`, `disabled`, `hidden`, `hint`, `infoLabel`, `labels`.

Props específicas por tipo:

| Tipo | Props específicas relevantes |
|---|---|
| `Text` | `minLength`, `maxLength`, `formatType`, `formatConfig` |
| `Number` | `minValue`, `maxValue`, `enableFormatting`, separadores |
| `Choice` | `options: EFWTagPickerOption[]` |
| `MultiChoice` | `options`, **`multiple: true` obligatorio** |
| `Boolean` | `labels: { checkedText, uncheckedText }` |
| `Date` | `includeTime`, localización |
| `Attachments` | `acceptedTypes`, `maxFileSize` |

> Usa `internalName` como clave estable del valor (no `id`).

---

## 3. Props del componente `EFWForm`

| Prop | Tipo | Propósito |
|---|---|---|
| `fields` | `T` | Campos declarativos (excluyente con `sections`). |
| `sections` | `EFWFormSection<...>[]` | Grupos `{ title?, description?, fields, gridTemplateColumns? }`. |
| `gridTemplateColumns` | `number \| {autoFit:{minColumnWidth,maxColumns?}} \| string` | Layout grid (default 1). Solo con `fields`. |
| `gridTemplateGap` | `number` | Espaciado px (default 16). |
| `defaultValues` | `Partial<EFWFormValues<T>>` | Valores iniciales (uncontrolled). |
| `values` | `Partial<EFWFormValues<T>>` | Valores controlados (sincroniza vía `onValuesChange`). |
| `fieldLogic` | `EFWFieldLogic<T>` | Lógica condicional por campo. |
| `onValuesChange` | `(values: EFWFormValues<T>) => void` | En cada cambio. |
| `onInit` | `(allValues, updateField) => void \| Promise<void>` | **Una sola vez** al montar; soporta async. |
| `messages` | `EFWFormMessageBar[]` | Mensajes externos. |
| `formRef` | `MutableRefObject<EFWFormRef<T>>` | Acceso imperativo. |
| `localeText` | `Partial<EFWFormExtendedLocaleText>` | i18n del form + fields/buttons hijos. |
| `editable` / `disabled` / `hidden` / `required` | `boolean` | Estado global. |

> `fields` y `sections` son **mutuamente excluyentes**. `gridTemplateColumns` a nivel form solo aplica con `fields`; en sections va por sección.

---

## 4. `EFWFormMethods` (API imperativa vía `formRef` / `useFormController`)

| Método | Firma | Propósito |
|---|---|---|
| `getValues` | `() => EFWFormValues<T>` | Snapshot tipado. |
| `setValues` | `(newValues, options?) => void` | Actualiza (parcial o total). |
| `resetValues` | `(options?) => void` | Resetea a estado inicial. |
| `setDisabled` | `(disabled, fieldKeys?) => names[]` | Deshabilita todo o campos. |
| `setEditable` | `(editable, fieldKeys?) => names[]` | Solo-lectura (≠ disabled). |
| `isDisabled` / `isEditable` | `() => boolean` | Estado global. |
| `validateForm` | `(showErrorNotification?) => boolean` | Valida; `true` si pasa. |
| `addMessageBar` / `removeMessageBar` / `removeAllMessagesBar` | — | Gestión de mensajes. |
| `updateField` | overloaded | Actualiza props/valor de campos. |

**`setValues` options:** `triggerOnChange?`, `triggerFieldLogic?`, `overwriteAll?`, `animateUpdatedFields?`, `force?` (fuerza aunque el valor sea idéntico).

**`updateField` — clave de valor por tipo:**

- `value` → Text/Number/Boolean/Note
- `selectedOptions` → Choice/MultiChoice
- `selectedDates` → Date
- `files` → Attachments

```ts
// Campos PLANOS (EFWFormFieldProps[]): genérico para props del tipo
updateField<'Text'>(['name'], { value: 'Nuevo', disabled: true });
updateField(['name', 'age'], { disabled: true }); // solo props comunes en modo mixto
updateField<'Choice'>(['skills'], { selectedOptions: [{ key: 'a', text: 'A' }] });

// Campos TIPADOS (as const): SIN genérico (infiere por nombre) — ver § 10
updateField(['skills'], { selectedOptions: [{ key: 'a', text: 'A' }] });
```

---

## 5. `useFormController<T>()`

Devuelve todos los métodos de `EFWFormMethods<T>` + `formRef`, `values` (estado reactivo) y `onValuesChange`. Conexión:

```tsx
const form = useFormController<typeof fields>();
<EFWForm fields={fields} formRef={form.formRef} onValuesChange={form.onValuesChange} />
// form.values siempre sincronizado; form.getValues(), form.validateForm(), etc.
```

> Sin `formRef`, los métodos hacen no-op seguro (`getValues()` cae a `values`, `validateForm()` → `false`).

---

## 6. Converters flat (API ↔ Form)

Convierten entre el shape "plano" del consumidor (primitivos) y el interno de `EFWForm`:

```ts
flatToEFWValues(fields, flat, options?)   // → Partial<EFWFormValues<F>>
EFWValuesToFlat(fields, efwValues, options?) // → Partial<FlatValues<F>>
```

Mapeo plano: Text/Note/Choice/Date → `string`; Number → `number`; Boolean → `boolean`; MultiChoice → `readonly string[]`; Attachments → `readonly EFWAttachmentsValue[]`. (Choice usa la `key`.)

```ts
flatToEFWValues(fields, { name: 'John', category: 'a' });
// → { name: 'John', category: [{ key: 'a', text: 'A' }] }
EFWValuesToFlat(fields, { name: 'John', category: [{ key: 'a', text: 'A' }] });
// → { name: 'John', category: 'a' }
```

Opciones: `choiceLookup`, `unknownChoiceKey`, `dateParse` (flat→EFW); `dateSerialize`, `emptyKeys` (EFW→flat). Errores fatales lanzan `EFWConversionError`.

---

## 7. Campos tipados vs no tipados (IMPORTANTE)

Hay dos formas de declarar `fields`. **Por defecto recomienda siempre la tipada.**

### Modo tipado (RECOMENDADO) — `as const satisfies`

```ts
const fields = [
  { id: 'name',   typeAsString: 'Text',   internalName: 'name'   },
  { id: 'age',    typeAsString: 'Number', internalName: 'age'    },
  // Choice/MultiChoice EXIGEN `options` (son campos tipo TagPicker)
  { id: 'skills', typeAsString: 'Choice', internalName: 'skills', options: [{ key: 'react', text: 'React' }] },
] as const satisfies EFWFormFieldProps[];

type Names = ExtractFieldNames<typeof fields>;   // 'name' | 'age' | 'skills'
type Vals  = EFWFormValues<typeof fields>;        // { name: string; age: number | null; skills: ... }
```

> **`Choice` y `MultiChoice` requieren `options`** (su tipo es `EFWFormTagPickerFieldProps`). Omitirlas es error de compilación. Lo mismo vale para los `fields` de `EFWTable`.

El `as const` congela los literales (`internalName`, `typeAsString`) y `satisfies` valida el shape sin perder la inferencia. Resultado:

- `onValuesChange(values)` → `values.name` es `string`, `values.age` es `number | null` (no union).
- `getValues()` / `setValues()` tipados por nombre de campo; nombres inexistentes dan error de compilación.
- `updateField(['skills'], { selectedOptions })` infiere el tipo del campo **sin** genérico explícito.
- `EFWTableItem<typeof fields>` y `FilterValues<typeof fields>` quedan tipados al reusar los mismos `fields` en `EFWTable`.

### Modo no tipado (plano) — `EFWFormFieldProps[]`

```ts
const fields: EFWFormFieldProps[] = [
  { id: 'name', typeAsString: 'Text', internalName: 'name' },
];
```

Todo queda genérico: `values` es union, `getValues()` no conoce los nombres, y `updateField` exige genérico explícito (`updateField<'Choice'>(['skills'], { selectedOptions })`) y a veces casts. Funciona, pero pierde seguridad de tipos y autocompletado.

### Comparación

| Aspecto | Tipado (`as const satisfies`) | Plano (`EFWFormFieldProps[]`) |
|---|---|---|
| Inferencia de nombres/valores | Sí (`ExtractFieldNames`, `EFWFormValues`) | No (todo union) |
| `updateField` | Sin genérico | Requiere `<'Tipo'>` + casts |
| Errores en compilación | Sí | Tarde (runtime) |
| Reuso tipado en `EFWTable`/`FilterValues` | Sí | No |
| Fields dinámicos en runtime | No aplica | Único caso válido |

### Recomendación para el agente

- **Genera por defecto el modo tipado** (`as const satisfies EFWFormFieldProps[]`) en todo código nuevo de Form/Table.
- Usa el **modo plano solo si**: el usuario lo pide explícitamente, o los `fields` se construyen dinámicamente en runtime (origen desconocido en compilación: API, configuración externa, generación programática).
- Si no está claro qué modo conviene (el usuario no especificó y los campos podrían ser dinámicos), **pregúntale**: "¿Campos estáticos (tipado fuerte, recomendado) o dinámicos en runtime (tipo plano)?".
- El `as const` exige declarar `fields` **fuera del render** (módulo) o memoizado, lo que además preserva la identidad estable (§ 8).

---

## 8. Gotchas

- **`onInit` corre UNA sola vez** al montar (deps vacías). Cambiar su referencia no re-ejecuta; memoizar solo importa si depende de props/estado (evitar closures stale). Soporta async. Úsalo para inicialización en vez de un `useEffect` externo.
- **`fieldLogic`**: con campos genéricos el `value` es union y `updateField` necesita genérico explícito. Con `as const` el `value` ya viene tipado por campo. Contexto: `allValues`, `trigger` (`'init'|'change'`), `fieldName`, `previousValue`, `isControlled`.
- **`updateField` sin tipo**: solo props comunes/compatibles entre TODOS los campos; `selectedOptions`/`selectedDates`/`files` prohibidas en modo mixto.
- **Identidad de fields**: declara `fields` fuera del render (o memoiza). Cambiar referencia/orden reconstruye el índice.
- **Controlled vs uncontrolled**: NO mezclar `values` y `defaultValues`. En controlled implementa `onValuesChange` y actualiza `values`. Default recomendado: uncontrolled + `formRef`.
- **`setValues`** omite por defecto keys sin cambios (`Object.is`); usa `force: true` para forzar.

---

## 9. Ejemplos

### A) Básico (uncontrolled, grid responsivo)

```tsx
const fields: EFWFormProps['fields'] = [
  { id: 'firstName', typeAsString: 'Text', internalName: 'firstName', title: 'Nombre',
    required: true, labels: { placeholder: 'Ingrese su nombre' }, minLength: 2, maxLength: 50 },
  { id: 'experience', typeAsString: 'Number', internalName: 'experience',
    title: 'Años de experiencia', minValue: 0, maxValue: 50 },
  { id: 'skills', typeAsString: 'Choice', internalName: 'skills', title: 'Habilidades', multiple: true,
    options: [{ key: 'piloting', text: 'Pilotaje' }, { key: 'astro', text: 'Astrofísica' }] },
];

<EFWForm
  title="Formulario básico"
  fields={fields}
  gridTemplateColumns={{ autoFit: { minColumnWidth: '250px', maxColumns: 4 } }}
  onValuesChange={(values) => console.log(values)}
/>
```

### B) Tipado fuerte + `useFormController` + lógica entre campos

```tsx
const fields = [
  { id: 'firstName', typeAsString: 'Text',   internalName: 'firstName', title: 'Nombre', required: true },
  { id: 'experience', typeAsString: 'Number', internalName: 'experience', title: 'Experiencia' },
  { id: 'skills', typeAsString: 'Choice', internalName: 'skills', title: 'Habilidades', multiple: true,
    options: [{ key: 'survival', text: 'Supervivencia' }] },
] as const satisfies EFWFormFieldProps[];

const TypedForm = () => {
  const form = useFormController<typeof fields>();

  const handleSubmit = () => {
    if (form.validateForm(true)) {
      const values = form.getValues(); // values.firstName: string, values.experience: number | null
      console.log(values);
    }
  };

  return (
    <>
      <EFWForm
        fields={fields}
        formRef={form.formRef}
        onValuesChange={form.onValuesChange}
        defaultValues={{ firstName: 'Samus' }}
        fieldLogic={{
          firstName: (value, updateField) => {
            if (value === 'Samus') {
              updateField(['experience'], { value: 30 });
            } else {
              updateField(['experience'], { value: null });
            }
          },
        }}
      />
      <button onClick={handleSubmit}>Guardar</button>
    </>
  );
};
```

### C) Datos de API (flat) → form y de vuelta

```tsx
const apiData = { firstName: 'Samus', skills: ['survival'] };
const formValues = flatToEFWValues(fields, apiData);
// <EFWForm fields={fields} defaultValues={formValues} />

const flat = EFWValuesToFlat(fields, form.getValues()); // para enviar a la API
```

---

## 10. Casos avanzados

### Layout por secciones (`sections`)

Agrupa un formulario largo en bloques lógicos, cada uno con su título, descripción y grid. **Mutuamente excluyente con `fields`**.

```ts
interface EFWFormSection<T extends EFWFormFieldProps = EFWFormFieldProps> {
  title?: ReactNode; description?: ReactNode;
  fields: T[];
  gridTemplateColumns?: GridTemplateColumnsType;
}
```

```tsx
<EFWForm
  title="Organización por secciones"
  sections={[
    { title: 'Información personal', description: 'Datos básicos.',
      gridTemplateColumns: { autoFit: { minColumnWidth: '250px', maxColumns: 4 } }, fields: personalFields },
    { title: 'Información adicional', gridTemplateColumns: 1, fields: extraFields },
  ]}
/>
```

### Mensajes declarativos (`messages`)

Barras de mensaje contextuales (info/success/warning/error) desde props, sin lógica imperativa. Soporta mensajes no descartables y con acciones.

```ts
interface EFWFormMessageBar {
  id: number | string;
  intent?: 'info' | 'success' | 'warning' | 'error';
  title?: string; message?: ReactNode;
  actions?: EFWFormMessageBarAction[]; // { id, content, appearance?, icon?, onClick? }
  dismissable?: boolean;
  source?: 'internal' | 'external';
}
```

```tsx
<EFWForm
  fields={fields}
  messages={[
    { id: 'info', intent: 'info', title: 'Información', message: 'Los campos con * son obligatorios.' },
    { id: 'locked', intent: 'info', message: 'No se puede cerrar.', dismissable: false },
    { id: 'actions', intent: 'warning', message: 'Revisa los datos.',
      actions: [{ id: 'fix', content: 'Corregir', onClick: () => handleFix() }] },
  ]}
/>
```

### Gestión imperativa de mensajes (controller)

Cuando los mensajes dependen de eventos (guardado, validación de negocio, async), agrégalos/quítalos por código. `addMessageBar(message, source?)` distingue origen `'internal'` (lógica del form) de `'external'` (negocio).

```ts
addMessageBar: (message: EFWFormMessageBar, source?: 'internal' | 'external') => void;
removeMessageBar: (messageId: string | number) => void;
removeAllMessagesBar: () => void;
```

```tsx
const { formRef, addMessageBar, removeAllMessagesBar } = useFormController();

const notifySuccess = () => addMessageBar({
  id: crypto.randomUUID(), intent: 'success', title: 'Éxito', message: 'Operación completada.',
});

<EFWForm formRef={formRef} fields={fields} />
```

### Render personalizado en modo solo-lectura (`readOnlyRenderer`)

Cada campo personaliza cómo se ve con `editable={false}` vía `readOnlyRenderer` (prop **a nivel de campo**, no del form). Si retorna `null`/`undefined`, usa el render por defecto. La firma varía el tipo de `value` por campo.

```ts
// Input:  (value: EFWInputValue, displayValue: string) => ReactNode
// Number: (value: EFWNumberInputValue, displayValue: string) => ReactNode
// Date:   (value: EFWDatePickerValue[], displayValue: string) => ReactNode
```

```tsx
const fields: EFWFormFieldProps[] = [
  { id: 'mission_link', internalName: 'missionLink', title: 'Enlace', typeAsString: 'Text',
    value: 'https://example.com/mission',
    readOnlyRenderer: (value, displayValue) => (
      <Link href={value || ''} target="_blank">{displayValue}</Link>
    ) },
];

<EFWForm editable={false} fields={fields} gridTemplateColumns={2} />
```

### Animación de campos actualizados (`fieldAnimationConfig`)

Resalta los campos modificados programáticamente vía `setValues`/`resetValues`. Útil para que el usuario note qué cambió tras autocompletar o cargar datos.

```ts
interface EFWFormFieldAnimationConfig { setValues?: boolean; resetValues?: boolean; }
// override por llamada: setValues(values, { animateUpdatedFields?: boolean }) — tiene prioridad
```

```tsx
const { formRef, setValues, resetValues } = useFormController();

<EFWForm formRef={formRef} fields={fields}
  fieldAnimationConfig={{ setValues: true, resetValues: true }} />

setValues({ firstName: 'Samus', email: 'samus@x.com' });          // anima por defecto
setValues({ firstName: 'Samus' }, { animateUpdatedFields: false }); // sin animación esta vez
resetValues();                                                     // anima al volver al inicial
```

### Grid avanzado (`gridTemplateColumns` / `gridTemplateGap`)

Tres formas: número fijo, CSS Grid directo (string) o responsivo `autoFit`. `gridTemplateGap` ajusta el espaciado (px, default 16).

```tsx
<EFWForm fields={fields} gridTemplateColumns="250px minmax(0, 1fr)" gridTemplateGap={24} /> {/* CSS directo */}
<EFWForm fields={fields} gridTemplateColumns={{ autoFit: { minColumnWidth: '250px', maxColumns: 4 } }} /> {/* responsivo */}
<EFWForm fields={fields} gridTemplateColumns={3} /> {/* fijo */}
```

### Propagación de interactividad (`editable`/`disabled` global vs. por campo)

`editable`/`disabled` del form aplican a todos los campos que **no** lo definen explícitamente; un campo con valor propio gana sobre el global. Imperativamente, `setEditable`/`setDisabled` aceptan `fieldKeys` para acotar a campos concretos.

```ts
setDisabled: (disabled: boolean, fieldKeys?: ExtractFieldNames<T>[]) => ExtractFieldNames<T>[];
setEditable: (editable: boolean, fieldKeys?: ExtractFieldNames<T>[]) => ExtractFieldNames<T>[];
```

```tsx
const { formRef, setEditable, setDisabled } = useFormController();

<EFWForm formRef={formRef} fields={fields} editable={formEditable} disabled={formDisabled} />

setEditable(false, ['missionName']); // solo lectura en un campo
setDisabled(true);                   // deshabilita todo el form
```

### Validación imperativa (`validateForm`)

Ejecuta todas las reglas de los campos (requeridos, formatos, validaciones custom). Retorna `boolean`; con `showErrorNotification=true` muestra los mensajes de error automáticamente. Ídeal antes de enviar.

```tsx
const { formRef, validateForm, getValues } = useFormController();

const handleSubmit = () => {
  if (validateForm(true)) submit(getValues());
  else console.log('Formulario inválido.');
};

<EFWForm formRef={formRef} fields={fields} />
```

### Actualización avanzada de campos (`updateField`)

A diferencia de `setValues` (solo valores), `updateField` modifica **cualquier** prop de uno o más campos: valor, validación, visibilidad, labels, opciones. **El uso del genérico depende de si los campos son tipados o planos** (verificado contra el código):

- **Campos tipados (`as const`) → SIN genérico.** `updateField` infiere el tipo de cada campo por su nombre.
- **Campos planos (`EFWFormFieldProps[]`) → CON genérico** (`updateField<'Choice'>`) para props específicas del tipo; sin genérico solo permite props comunes.

```tsx
// ✅ Campos TIPADOS (useFormController<typeof fields>): sin genérico
const { formRef, updateField } = useFormController<typeof fields>();

updateField(['firstName'], { value: 'Samus', labels: { placeholder: 'Nombre' } });
updateField(['gender'], { selectedOptions: [{ key: 'female', text: 'Femenino' }] });
updateField(['firstName', 'lastName', 'age'], { disabled: true });          // props comunes (tipos mixtos)
updateField(['email'], { validationState: 'error', validationMessage: 'Correo ya registrado' });
updateField(['emergencyPhone'], { hidden: true });                          // mostrar/ocultar dinámico

<EFWForm formRef={formRef} fields={fields} />
```

```tsx
// Campos PLANOS (EFWFormFieldProps[]): genérico obligatorio para props del tipo
const { updateField } = useFormController();
updateField<'Choice'>(['gender'], { selectedOptions: [{ key: 'female', text: 'Femenino' }] });
updateField<'Number'>(['age'], { value: null });
updateField(['firstName', 'age'], { disabled: true }); // props comunes: sin genérico igual
```

> Misma regla aplica al `updateField` que recibe `fieldLogic`: con campos tipados infiere sin genérico; con planos necesita `<'Tipo'>`.
