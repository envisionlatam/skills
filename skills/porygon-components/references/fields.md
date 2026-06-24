# Fields — `EFW*Field`

**Cuándo usar:** capturar un dato individual fuera de un `EFWForm` (o componer formularios a mano). Cubre texto (`EFWInputField`), número (`EFWNumberInputField`), texto multilínea (`EFWTextAreaField`), booleano (`EFWSwitchField`), selección única/múltiple (`EFWTagPickerField`), fecha/hora (`EFWDatePickerField`) y archivos adjuntos (`EFWAttachmentsField`), todos con label, validación, estados y modo solo-lectura. Si vas a renderizar varios campos juntos con validación y layout, prefiere `EFWForm` (ver forms.md).

Cada `EFW*Field` = props base de `EFWField` (sin `children`) **+** props del componente interno:
`EFWXxxFieldProps extends Omit<EFWFieldProps, 'children'>, EFWXxxProps`.

Subpaths: `@envisiongroup/porygon/react-components/fields/{Input|NumberInput|TextArea|Switch|TagPicker|DatePicker|Attachments}` (todo también desde el barrel raíz).

> **Regla clave:** `onChange` entrega `(rawValue, displayValue?)`. **Guarda el primer argumento** (raw normalizado); `displayValue` es solo presentación.

---

## 1. Props compartidas `EFWField`

Todas opcionales; cada `*Field` las hereda con `Omit<EFWFieldProps,'children'>`.

| Prop | Tipo | Default | Descripción |
|---|---|---|---|
| `title` | `string` | — | Label principal. |
| `infoLabel` | `InfoLabelProps['info']` | — | Info/tooltip junto al título. |
| `weightLabel` | `InfoLabelProps['weight']` | — | Peso visual del label. |
| `hint` | `FieldProps['hint']` | — | Texto de ayuda. |
| `required` | `boolean` | `false` | Marca obligatorio (asterisco + validación). |
| `orientation` | `'horizontal'\|'vertical'` | — | Orientación. |
| `size` | `FieldProps['size']` | `medium` | Tamaño. |
| `editable` | `boolean` | `true` | Si `false` → modo solo-lectura (usa `readOnlyRenderer`). |
| `disabled` | `boolean` | `false` | Bloquea interacción. |
| `hidden` | `boolean` | `false` | No renderiza el campo. |
| `validationState` | `'none'\|'error'\|'warning'\|'success'\|'loading'` | — | Estado de validación visual (control externo). |
| `validationMessage` | `FieldProps['validationMessage']` | — | Mensaje del estado. |
| `onValidationChange` | `(v) => void` | — | Avisa cuando cambia el estado de validación interno. |

> `value`/`defaultValue`/`onChange` **NO** viven en `EFWFieldProps`; vienen del componente interno y cambian de nombre/tipo por campo (ver cada sección).

**`editable={false}` ≠ `disabled`:** el primero es modo lectura formateado (admite `readOnlyRenderer`), el segundo bloquea con estilo deshabilitado.

---

## 2. EFWInputField (texto)

```ts
import { EFWInputField, type EFWInputFieldValue } // string | null | undefined
  from "@envisiongroup/porygon/react-components/fields/Input";
```

**Cuándo usar:** texto libre de una línea; soporta formatos (RUT, email, tarjeta, máscaras custom).

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `value` / `defaultValue` | `string \| null \| undefined` | — | Presencia de `value` → controlado. |
| `onChange` | `(value, displayValue?) => void` | — | `value: string \| null \| undefined`. |
| `type` | `'text'\|'password'\|'email'\|'number'\|'tel'` | `"text"` | |
| `formatType` | `'none'\|'creditCard'\|'chileanRut'\|'date'\|'custom'\|'email'\|'time'` | `none` | |
| `formatConfig` | `{ mask, stripPattern, validationPattern, validationTarget:'raw'\|'formatted', validate, invalidMessage }` | — | API recomendada para máscaras de negocio. |
| `formatPattern` | `string` | — | Para `custom` (placeholder `#`). |
| `minLength` / `maxLength` | `number` | `0` / `250` | |
| `contentBefore` / `contentAfter` | `string` | — | Prefijo/sufijo. |
| `validationMessages` | `EFWInputValidationMessages` | — | Mensajes por regla. |
| `readOnlyRenderer` | `(value, displayValue) => ReactNode` | — | Render en `editable={false}`. |

**Guardar:** el `value` (raw normalizado, `""` si vacío). Con `mask`+`stripPattern`, entradas inválidas se emiten **tal cual** (con decoración) y se marcan error.

```tsx
// Teléfono móvil chileno con máscara
<EFWInputField
  title="Teléfono móvil Chile"
  type="tel"
  formatType="custom"
  formatConfig={{
    mask: "+56 # #### ####",
    stripPattern: /\D/g,
    validationPattern: /^569\d{8}$/,
    validationTarget: "raw",
    invalidMessage: "Ingrese un teléfono móvil chileno válido.",
  }}
  onChange={(value) => setTel(value ?? "")} // raw: 56912345678
/>
```

---

## 3. EFWNumberInputField (numérico)

```ts
import { EFWNumberInputField, type EFWNumberInputFieldValue } // number | null
  from "@envisiongroup/porygon/react-components/fields/NumberInput";
```

**Cuándo usar:** montos, cantidades, porcentajes; con separadores y rangos.

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `value` / `defaultValue` | `number \| null` | — | |
| `onChange` | `(value: number \| null, displayValue?) => void` | — | |
| `minValue` / `maxValue` | `number` | `0` / `0` | Valida límites. |
| `enableFormatting` | `boolean` | `true` | Separadores miles/decimales. |
| `thousandSeparator` / `decimalSeparator` | `string` | `","` / `"."` | |
| `contentBefore` / `contentAfter` | `string` | — | Ej. `$`, `%`, `kg`. |
| `validationMessages` | `{ invalidNumber?, minValue?(min,fmt), maxValue?(max,fmt) }` | — | |

**Guardar:** el `number` (o `null` si vacío). ⚠️ `maxValue` default `0`: si no lo defines, puede limitar inesperadamente — pásalo explícito.

```tsx
<EFWNumberInputField
  title="Edad (0-120)"
  minValue={0}
  maxValue={120}
  contentAfter="años"
  onChange={(value) => setEdad(value)} // number | null
/>
```

---

## 4. EFWTextAreaField (multilínea)

```ts
import { EFWTextAreaField, type EFWTextAreaFieldValue } // string | null
  from "@envisiongroup/porygon/react-components/fields/TextArea";
```

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `value` / `defaultValue` | `string \| null` | — | |
| `onChange` | `(value: string \| null, displayValue?) => void` | — | |
| `onKeyDown` | `(e, currentValue: string) => void` | — | |
| `maxLength` | `number` | `250` | |
| `autoResize` | `boolean` | `true` | Crece con el contenido. |

**Guardar:** el `string`.

```tsx
<EFWTextAreaField title="Comentarios" maxLength={500} autoResize
  onChange={(value) => setComentario(value ?? "")} />
```

---

## 5. EFWSwitchField (booleano)

```ts
import { EFWSwitchField, type EFWSwitchFieldValue } // boolean | undefined
  from "@envisiongroup/porygon/react-components/fields/Switch";
```

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `value` / `defaultValue` | `boolean` | — | |
| `onChange` | `(value: boolean, displayValue?) => void` | — | |
| `labels` | `{ checkedText?, uncheckedText?, emptyState? }` | — | Texto por estado. |

**Guardar:** el `boolean`. (`*FieldValue` incluye `undefined` por compat con EFWForm, pero en runtime es `boolean`.)

```tsx
<EFWSwitchField title="Notificaciones" defaultValue={true}
  labels={{ checkedText: "Activadas", uncheckedText: "Desactivadas" }}
  onChange={(value) => setNotif(value)} />
```

---

## 6. EFWTagPickerField (choice / multichoice)

```ts
import { EFWTagPickerField, type EFWTagPickerFieldOption, type EFWTagPickerFieldValue }
  from "@envisiongroup/porygon/react-components/fields/TagPicker";
// EFWTagPickerOption = { key: string; text: string; secondaryContent?: string; item?: IItem }
```

**Cuándo usar:** selección desde opciones (única o múltiple), estáticas o async.

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `options` | `EFWTagPickerOption[] \| (query, {signal}) => Promise<Option[]>` | `[]` | **Requerida.** Soporta loader async. |
| `selectedOptions` | `EFWTagPickerOption[]` | — | Presencia → controlado. |
| `defaultSelectedOptions` | `EFWTagPickerOption[]` | — | Uncontrolled inicial. |
| `onChange` | `(selectedOptions: EFWTagPickerOption[], displayValue?) => void` | — | |
| `multiple` | `boolean` | `true` | `false` = selección única. |
| `minQueryLength` | `number` | `0` | Caracteres antes de búsqueda async. |
| `tagShape` / `tagAppearance` / `tagSize` | — | — | Estilo de tags. |
| `renderOptionMedia` | `(option) => ReactNode` | — | Media custom. |

**Guardar:** el **array de opciones completas** (`{key,text,...}`), no solo los keys. Aun con `multiple={false}` entrega array. ⚠️ La prop de selección es `selectedOptions`/`defaultSelectedOptions` (NO `value`).

```tsx
const loadSkills = async (q: string) =>
  skillOptions.filter(s => s.text.toLowerCase().includes(q.toLowerCase()));

<EFWTagPickerField title="Habilidades" options={loadSkills}
  onChange={(selectedOptions) => setSkills(selectedOptions)} />
```

---

## 7. EFWDatePickerField (fecha / fecha+hora)

```ts
import { EFWDatePickerField, type EFWDatePickerFieldValue } // Date | string
  from "@envisiongroup/porygon/react-components/fields/DatePicker";
```

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `selectedDates` | `(Date\|string)[]` | — | Presencia → controlado. Vacío = `[]`. |
| `defaultSelectedDates` | `(Date\|string)[]` | `[]` | Uncontrolled inicial. |
| `onChange` | `(value: (Date\|string)[]) => void` | — | **Array, sin displayValue.** |
| `minDate` / `maxDate` | `Date` | — | Restricción de rango. |
| `restrictedDates` | `Date[]` | `[]` | Fechas bloqueadas. |
| `friendlyDateFormat` | `boolean` | `false` | `"Enero 1, 2025"` vs `01/01/2025`. |
| `localizedStrings` | `CalendarStrings` | ES | i18n del calendario. |
| `includeTime` *(Field)* | `boolean` | `false` | Agrega input HH:MM. |
| `timeValue` / `defaultTimeValue` *(Field)* | `string` (4 dígitos, `"1430"`) | — | Hora. |

**Guardar:** normalmente `value[0]` (un `Date`). Con `includeTime`, el `Date` combina fecha + hora. ⚠️ Selección vía `selectedDates`/`defaultSelectedDates` (NO `value`), siempre array.

```tsx
const [dates, setDates] = useState<EFWDatePickerFieldValue[]>([new Date(2025,0,15,14,30)]);
<EFWDatePickerField title="Fecha y hora" includeTime
  selectedDates={dates}
  onChange={(newDates) => setDates(newDates)} /> // newDates[0] = Date con hora
```

---

## 8. EFWAttachmentsField (adjuntos)

```ts
import { EFWAttachmentsField, type EFWAttachmentsFieldValue }
  from "@envisiongroup/porygon/react-components/fields/Attachments";
// Value = { id; name; size?; type?; lastModified? } & ({ file: File; serverRelativeUrl? } | { file?: File; serverRelativeUrl: string })
```

| Prop | Tipo | Default | Notas |
|---|---|---|---|
| `files` | `EFWAttachmentsValue[]` | — | Presencia → controlado. |
| `defaultFiles` | `EFWAttachmentsValue[]` | — | Uncontrolled inicial. |
| `onChange` | `(values: EFWAttachmentsValue[], displayValue?) => void` | — | |
| `onRemoveAttachment` | `(attachment) => Promise<boolean>` | — | Confirma eliminación (resolver `true`). |
| `multiple` | `boolean` | `true` | |
| `acceptedTypes` | `string` (MIME / extensión) | — | Ej. `"image/*"`, `".pdf"`. |
| `maxFileSize` | `number` (bytes) | `104857600` (100MB) | |

**Guardar:** el **array** de objetos adjunto. Cada item trae `file: File` (local) y/o `serverRelativeUrl` (servidor). ⚠️ Prop de valor: `files`/`defaultFiles` (NO `value`).

```tsx
const handleRemove = async (att: EFWAttachmentsFieldValue) =>
  window.confirm(`¿Eliminar "${att.name}"?`);

<EFWAttachmentsField title="Solo imágenes" acceptedTypes="image/*"
  maxFileSize={2 * 1024 * 1024} multiple={false}
  onChange={(files) => setFiles(files)} onRemoveAttachment={handleRemove} />
```

---

## 9. Normalización de valores (`commons/valueNormalization`)

Reglas de "vacío" compartidas por Form y campos:

- **`isSemanticallyEmptyValue`** → `true` si: `undefined`/`null`, string con `.trim() === ''`, o array `length === 0`.
- **Texto** (Input/TextArea): vacío = `''`. `onChange` siempre emite `string`.
- **Arrays** (TagPicker `selectedOptions`, DatePicker `selectedDates`, Attachments `files`): vacío = `[]`, nunca `null`/`undefined`.
- **Booleano** (Switch): siempre `boolean` en runtime.
- **Número** (NumberInput): vacío = `null`.
- En campos de texto: `undefined` = prop ausente (uncontrolled); `null` = vacío explícito externo; `string` = valor efectivo.

---

## 10. Gotchas por campo

- **Generales:** prop de valor presente → controlado (actualiza estado en `onChange`). Guarda el primer arg de `onChange`. `editable={false}` ≠ `disabled`.
- **Input:** con `mask`+`stripPattern`, inválidos se emiten tal cual con error. `maxLength` default 250.
- **NumberInput:** value `number | null`; `maxValue` default `0` puede limitar — pásalo explícito. Separadores solo con `enableFormatting`.
- **TextArea:** `formatPattern` no implementado; `onKeyDown` entrega `(event, currentValue)`.
- **Switch:** `*FieldValue` incluye `undefined` por compat, runtime siempre `boolean`.
- **TagPicker:** `options` requerida; selección vía `selectedOptions`; entrega array de opciones completas (no keys), incluso con `multiple={false}`.
- **DatePicker:** selección vía `selectedDates` (array); `onChange` sin `displayValue`; `includeTime`/`timeValue` son props del Field.
- **Attachments:** valor vía `files`; `Value` exige `file` o `serverRelativeUrl`; `onRemoveAttachment` debe resolver `Promise<boolean>`; `maxFileSize` en bytes.

> **Tipado:** los `EFW*Field` sueltos ya entregan valores concretos por tipo (`string`, `number | null`, `EFWAttachmentsValue[]`, etc.) — no necesitan `as const`. El tipado fuerte con `as const satisfies EFWFormFieldProps[]` aplica cuando declaras un **array de `fields`** para `EFWForm`/`EFWTable` (ver forms.md § 7).

---

## 11. Casos avanzados

Patrones avanzados de los campos sueltos. Cada caso trae el shape de tipos real y un snippet mínimo basado en las stories.

### TagPicker — carga asíncrona de opciones

Cuando las opciones vienen del backend y se filtran al escribir: pasa una función a `options` en vez de un array.

```ts
export type AsyncOptionsLoader = (query: string, options?: { signal: AbortSignal }) => Promise<EFWTagPickerOption[]>;
// options: EFWTagPickerOption[] | AsyncOptionsLoader
// minQueryLength?: number  // mínimo de caracteres antes de buscar
```

```tsx
const loadSkills = async (query: string): Promise<EFWTagPickerFieldOption[]> => {
  const res = await fetch(`/api/skills?q=${query}`);
  return res.json();
};

<EFWTagPickerField title="Habilidades" options={loadSkills} minQueryLength={2}
  onChange={(selectedOptions) => console.log(selectedOptions)} />
```

### TagPicker — choice vs multichoice

`multiple={false}` limita a selección única (por defecto es múltiple). En ambos casos `onChange` entrega un **array** de opciones completas.

```tsx
<EFWTagPickerField title="Departamento" options={departmentOptions} multiple={false}
  onChange={(opts) => console.log(opts[0])} />
```

### TagPicker — media/avatar en opciones y tags

`renderOptionMedia` personaliza el media de cada opción (si se omite, usa Avatar por defecto desde `option.text`; `() => null` lo desactiva). `showOptionMediaInSelectedTags` muestra el media también dentro de cada tag seleccionado.

```ts
// renderOptionMedia?: (option: EFWTagPickerOption) => ReactNode;
// showOptionMediaInSelectedTags?: boolean;  // @default false
```

```tsx
import { Avatar } from '@fluentui/react-components';

const renderMedia = (o: EFWTagPickerFieldOption) =>
  <Avatar shape="square" name={o.text} image={{ src: imagesByKey[o.key] }} />;

<EFWTagPickerField title="Responsable" options={employeeOptions}
  renderOptionMedia={renderMedia} showOptionMediaInSelectedTags
  onChange={(opts) => console.log(opts)} />
```

> No existe freeform/creación de tags: las opciones siempre vienen de `options` (array o loader async).

### DatePicker — fecha + hora

`includeTime` agrega selección de hora; `timeValue`/`defaultTimeValue` son raw de 4 dígitos (`"1430"` = 14:30).

```ts
// includeTime?: boolean; timeValue?: string; defaultTimeValue?: string; timePlaceholder?: string;
```

```tsx
const [dates, setDates] = useState<EFWDatePickerFieldValue[]>([new Date(2025, 0, 15, 14, 30)]);
<EFWDatePickerField title="Fecha y hora" includeTime selectedDates={dates} onChange={setDates} />
```

### DatePicker — rango y fechas restringidas

`allowRangeSelection` / `daysToSelectInDayView` para rango; `minDate`/`maxDate`/`restrictedDates` para limitar.

```ts
// minDate?: Date; maxDate?: Date; restrictedDates?: Date[];
// allowRangeSelection?: boolean; daysToSelectInDayView?: number;
```

```tsx
<EFWDatePickerField title="Agendamiento" minDate={new Date()} maxDate={maxDate}
  restrictedDates={[feriado1, feriado2]} infoLabel="Días festivos no disponibles"
  onChange={(values) => console.log(values)} />

<EFWDatePickerField title="Rango" allowRangeSelection daysToSelectInDayView={7}
  onChange={(values) => console.log(values)} />
```

### DatePicker — localización y formato

`localizedStrings` reemplaza textos del calendario (default `esLocalizedStrings`); presets `esLocalizedStrings`/`enLocalizedStrings`/`ptLocalizedStrings`. `friendlyDateFormat` muestra "Enero 15, 2025"; `onFormatDate` permite formato custom.

```tsx
import { enLocalizedStrings } from '@envisiongroup/porygon/react-components/fields/DatePicker';
<EFWDatePickerField title="Date" localizedStrings={enLocalizedStrings} friendlyDateFormat
  onChange={(values) => console.log(values)} />
```

### Input — formatos incluidos (`formatType`)

Formato + validación listos. `onChange` entrega `(rawValue, displayValue)`: **guarda el raw**, muestra el display.

```ts
type FormatType = 'none' | 'creditCard' | 'chileanRut' | 'date' | 'custom' | 'email' | 'time';
```

```tsx
{/* raw 16602228-2 -> display 16.602.228-2, valida dígito verificador */}
<EFWInputField title="RUT" formatType="chileanRut" onChange={handleChange} />
<EFWInputField title="Correo" formatType="email" onChange={handleChange} />
<EFWInputField title="Tarjeta" formatType="creditCard" onChange={handleChange} />
<EFWInputField title="Hora" formatType="time" labels={{ placeholder: 'HH:MM' }} onChange={handleChange} />
```

### Input — formato propio con `formatConfig` (máscara + validación)

API recomendada para formatos de negocio: máscara visual, `stripPattern` para obtener el raw y validación por regex o función. Úsalo con `formatType="custom"`. Entradas inválidas se marcan como error (no se descartan en silencio).

```ts
interface EFWInputFormatConfig {
  mask?: string;                          // '#' = carácter del raw; el resto es decoración
  stripPattern?: RegExp | string;         // quita decoración para el raw
  validationPattern?: RegExp | string;
  validationTarget?: 'raw' | 'formatted'; // default 'raw'
  validate?: (value: string, ctx: { rawValue: string; formattedValue: string }) => boolean | string;
  invalidMessage?: string;
}
```

```tsx
{/* Teléfono móvil chileno: 569 + 8 dígitos */}
<EFWInputField title="Móvil" type="tel" formatType="custom"
  formatConfig={{
    mask: '+56 # #### ####', stripPattern: /\D/g,
    validationPattern: /^569\d{8}$/, validationTarget: 'raw',
    invalidMessage: 'Ingrese un móvil chileno válido.',
  }}
  onChange={handleChange} />

{/* validar el texto visible + función custom (si retorna string, es el mensaje de error) */}
<EFWInputField title="Serial" formatConfig={{
  validationPattern: /^NV-\d{4}-\d{3}$/, validationTarget: 'formatted', invalidMessage: 'Use NV-AAAA-NNN.',
}} onChange={handleChange} />
```

### Input — estado de validación controlado externamente

Para mostrar resultados de validaciones del servidor/async sin alterar el valor. Props heredadas de `EFWField`.

```ts
type ValidationState = 'error' | 'warning' | 'success' | 'none' | 'loading' | string;
// validationState?, validationMessage?: ReactNode, validationMessageIcon?: ReactNode
```

```tsx
<EFWInputField title="Correo" validationState="error" validationMessage="Correo ya registrado"
  onChange={handleChange} />
```

### NumberInput — formato regional y rango

`thousandSeparator`/`decimalSeparator` cambian el formato visible sin perder el número crudo (`onChange` entrega `number | null`). `minValue`/`maxValue` validan; `contentBefore`/`contentAfter` agregan prefijo/sufijo. No hay `step` ni `currency` dedicados.

```ts
// enableFormatting?: boolean /*true*/; thousandSeparator?: string /*','*/; decimalSeparator?: string /*'.'*/;
// minValue?: number; maxValue?: number; contentBefore?: string; contentAfter?: string;
```

```tsx
{/* Europeo: 10000.56 -> 10.000,56 */}
<EFWNumberInputField title="Monto" value={10000.56} thousandSeparator="." decimalSeparator=","
  contentBefore="$" onChange={handleChange} />

<EFWNumberInputField title="Descuento" minValue={0} maxValue={50} contentAfter="%" onChange={handleChange} />
```

### Attachments — restricciones y confirmación de borrado

`acceptedTypes` (MIME o extensión), `maxFileSize` (bytes, default 100MB), `multiple`. `onRemoveAttachment` retorna `Promise<boolean>`: si resuelve `false`, no se elimina (ideal para confirmar). El archivo cargado llega por `onChange` como `EFWAttachmentsValue` con `file: File`; los del servidor se referencian con `serverRelativeUrl`. No hay `onUpload`/`onDownload` (la subida/descarga es interna).

```ts
type EFWAttachmentsValue = { id: string; name: string; size?: number; type?: string; lastModified?: number }
  & ({ file: File; serverRelativeUrl?: string } | { file?: File; serverRelativeUrl: string });
// acceptedTypes?: string; maxFileSize?: number; multiple?: boolean;
// onRemoveAttachment?: (a: EFWAttachmentsValue) => Promise<boolean>;
```

```tsx
const handleRemove = async (att: EFWAttachmentsFieldValue) => window.confirm(`¿Eliminar "${att.name}"?`);

<EFWAttachmentsField title="Comprobante (PDF, máx 2MB)" acceptedTypes=".pdf"
  maxFileSize={2 * 1024 * 1024} multiple={false}
  onChange={(files) => console.log(files)} onRemoveAttachment={handleRemove} />
```
