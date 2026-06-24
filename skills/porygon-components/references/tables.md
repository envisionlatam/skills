# Tables — `EFWTable`

```tsx
import {
  EFWTable, useTableController,
  type EFWTableProps, type EFWTableItem, type EFWTableRef,
  type EFWTableMethods, type FilterValues, type EFWTableSortingState,
} from '@envisiongroup/porygon/react-components/tables/EFWTable';
import { type EFWFormFieldProps } from '@envisiongroup/porygon/react-components/forms/EFWForm';
```

**Cuándo usar:** mostrar/editar colecciones derivando columnas de los mismos `fields` que define un `EFWForm`. Trae de serie: sorting, filtering (drawer), selección, command bar CRUD (drawers con formularios), overlays (loading/error/empty), virtualización, scroll infinito y animación de filas. Es la "tabla de negocio" completa, no un data-grid de bajo nivel.

---

## 1. Columnas derivadas desde `fields`

`EFWTable` **no** define columnas manualmente: recibe `fields: T` (el mismo array de `EFWFormFieldProps[]` de `EFWForm`) y crea una columna por field:

- `id`/`accessorKey` = `field.internalName`
- `header` = `field.title || field.internalName`
- render y `sortingFn` según `field.typeAsString`
- los drawers CRUD reusan esos fields para renderizar un `EFWForm` interno

`EFWTableItem<T>` es el shape de cada fila (inferido desde fields) + `_rowId?: string`:

```ts
// con fields `as const satisfies readonly EFWFormFieldProps[]`:
EFWTableItem<typeof fields> = { firstName: string; experience: number | null; skills: EFWTagPickerFieldValue[]; _rowId?: string }
```

`_rowId` es la **identidad estable** de fila. Es opcional al pasar items: si falta, la tabla lo autogenera.

---

## 2. Props principales

`EFWTableProps<T extends readonly EFWFormFieldProps[]>`. Las más usadas:

| Prop | Tipo | Req. | Descripción |
|---|---|---|---|
| `fields` | `T` | ✅ | Campos. De aquí salen columnas y formularios CRUD. |
| `items` | `EFWTableItem<T>[]` | – | Modo **controlled** (actualiza vía `onItemsChange`). |
| `defaultItems` | `EFWTableItem<T>[]` | – | Modo **uncontrolled**. |
| `onItemsChange` | `(newItems) => void` | – | Notifica cambios. |
| `height` | `number` | – | Altura máx px → scroll interno. **Requerido si `useVirtualization`**. |
| `enableSelection` | `boolean` | – | Checkboxes de selección. |
| `enableCommandBar` | `boolean` | – | Muestra command bar (CRUD/filtro). |
| `dense` | `boolean` | – | Densidad compacta. |
| `tableStyle` | `'row' \| 'grid'` | – | Estilo visual. |
| `resizableColumns` / `autoSizeColumns` | `boolean` | – | Columnas. |
| `columnConfig` | `Record<string, {...}>` | – | Config por columna (ver abajo). |
| `useVirtualization` | `boolean` | – | Virtualiza filas (requiere `height`). |
| `defaultSorting` | `EFWTableSortingState` | – | Ej. `[{ id:'firstName', desc:false }]`. |
| `onSortChange` | `(state, tableMethods) => void` | – | Provisto → **sorting server-side** (usa `tableMethods.replaceAllItems`). |
| `onFilterChange` | `(filters, tableMethods) => void` | – | Provisto → **filtering server-side**. |
| `defaultSelectedItems` | `Set<number>` | – | Selección inicial (por índice). |
| `onSelectionChange` | `(selected: Set<number>) => void` | – | Cambios de selección. |
| `onRowClick` / `onRowDoubleClick` | `(item, index) => void` | – | Clicks de fila. |
| `tableRef` | `MutableRefObject<EFWTableRef<T>>` | – | API imperativa (§4). |
| `beforeAddItems` / `beforeUpdateItems` / `beforeDeleteItems` | callbacks async | – | Interceptores CRUD (§7). |
| `addButtonConfig` / `updateButtonConfig` / `deleteButtonConfig` | `{ enabled?, form?, ... }` | – | Config/desactivación de botones CRUD por defecto. |
| `filterButtonConfig` | `{...}` | – | Config del botón de filtro. |
| `columnRenderers` | `ColumnRenderers` | – | Renderers de celda por tipo/campo. |
| `rowAnimationConfig` | `{ addRows?, updateRows? }` | – | Animación de filas. |
| `onScrollEnd` / `loadingMore` / `scrollEndThreshold` | – | – | Infinite scroll. |
| `overlayState` / `defaultOverlayState` / `onOverlayStateChange` | `EFWTableOverlayConfig` | – | Overlays. |
| `localeText` | `Partial<EFWTableExtendedLocaleText>` | – | i18n tabla + form/fields/buttons de drawers. |

**`columnConfig[internalName]`:** `{ maxWidth?, width?, minWidth?, sticky?, hidden?, sortable?=true, filterable?=true, cellLineClamp? }`. (Attachments siempre excluido del drawer de filtros.)

---

## 3. `EFWTableMethods` (API imperativa)

CRUD (async, devuelven `OperationResult<T>`):

```ts
addItem(item, options?)        addItems(items, options?)
updateItem(index, item, opts?) updateItems(items, opts?)
deleteItem(index)              deleteItems(indexes)
```

Consultas / manipulación:

```ts
getItem(id)            // por _rowId
getItemByIndex(index)  replaceAllItems(newItems)  clearItems()
getItemIndex(id)       itemExists(id)  getItemsCount()  getItems()
```

Selección:

```ts
getSelectedRows()      // { index, item }[]
getSelectedItems()     // { items, indexes }
deselectAllRows()
```

Overlay:

```ts
setLoading(active, options?)        setError(active, { onRetry, retryButtonText })
setPartialLoading(active, options?) setEmpty(active, options?)
clearOverlay()                      getOverlayState()
```

Filtrado / sorting:

```ts
getColumnFilters()  setColumnFilters(filters)  clearColumnFilters()  hasActiveFilters()
getSortingState()   setSortingState(state)     clearSorting()        hasActiveSorting()
```

Options: `EFWTableAddOptions { position?: 'start'|'end'; animateRows? }`, `EFWTableUpdateOptions { animateRows? }`.

---

## 4. `useTableController<T>()`

Crea el `tableRef` y expone **todos** los `EFWTableMethods<T>` ya envueltos (con fallbacks seguros si la tabla no está montada) + `setFieldsConfig`, `isTableReady`:

```tsx
const { tableRef, addItem, updateItem, deleteItems, getSelectedItems,
        replaceAllItems, setLoading } = useTableController();
<EFWTable tableRef={tableRef} fields={fields} defaultItems={[]} />
```

---

## 5. `FilterValues<T>`

Mapa `{ [internalName]: filterValue | null | undefined }`. Con fields `as const`, claves y valores son tipados. Se obtiene/establece con `getColumnFilters()` / `setColumnFilters(filters)` / `clearColumnFilters()`. `onFilterChange(filters, tableMethods)` recibe este shape; proveerlo activa filtrado server-side (trae datos y llama `tableMethods.replaceAllItems(data)`).

---

## 6. Callbacks CRUD interceptores

```ts
beforeAddItems?:    (items, tableMethods) => Promise<OperationResult<Item> | void>;
beforeUpdateItems?: (items, tableMethods) => Promise<BeforeUpdateItemsResult<Item> | void>;
beforeDeleteItems?: (items, tableMethods) => Promise<BeforeDeleteItemsResult<Item> | void>;
onBeforeDelete?:    (items, indexes) => number[] | Promise<number[]>; // filtra índices a borrar
```

Reglas de retorno:

- **`void`** → continúa la operación normal (usa `throw new Error(...)` para abortar todo).
- **Objeto result** `{ successful, failed }` → control granular: solo se aplican los `successful`; los `failed` se reportan con su `error`.
- Son `async`: la tabla espera la promesa antes de mutar.

---

## 7. Identidad estable (`_rowId`) y selección

- Al montar, cada item recibe `_rowId: item._rowId || generateId()`.
- Búsquedas O(1) por `_rowId` (`getItem`/`getItemIndex`/`itemExists`). En update/replace se conserva el `_rowId`.
- La **selección** se expone por **índice** (`Set<number>`), pero la identidad de fila para render/animación se basa en `_rowId`.
- Pasa `_rowId` propio si la selección/animación debe sobrevivir a refetch/reordenamientos.

---

## 8. Gotchas

- **Caches/maps:** la tabla cachea índices por `_rowId`; mutar items in-place sin cambiar referencia puede no reflejarse — usa los métodos (`updateItem`, `replaceAllItems`) o `onItemsChange`.
- **Controlled vs uncontrolled:** `items` **requiere** `onItemsChange`; `defaultItems` lo maneja la tabla. No mezclar para los mismos datos.
- **Virtualización:** `useVirtualization` exige `height`.
- **Server-side:** `onSortChange`/`onFilterChange` desactivan el procesamiento interno; eres responsable de traer datos y llamar `replaceAllItems`.
- **Tipado fuerte:** usa `as const satisfies readonly EFWFormFieldProps[]` para items tipados.
- **Filtros:** `Attachments` nunca aparece en el drawer de filtros; usa `columnConfig[col].filterable: false` para ocultar otros.
- **Botones por defecto:** desactívalos con `addButtonConfig={{ enabled: false }}` (idem update/delete) si manejas el CRUD tú mismo.

---

## 9. Ejemplos

### A) Básico (uncontrolled, tipado fuerte)

```tsx
const fields = [
  { id: 'firstName', title: 'Nombre', typeAsString: 'Text', internalName: 'firstName', required: true },
  { id: 'experience', title: 'Experiencia (años)', typeAsString: 'Number', internalName: 'experience' },
  { id: 'skills', title: 'Habilidades', typeAsString: 'Choice', multiple: true, internalName: 'skills',
    options: [{ key: 'piloting', text: 'Pilotaje' }, { key: 'nav', text: 'Navegación' }] },
  { id: 'acceptRisks', title: 'Acepta riesgos', typeAsString: 'Boolean', internalName: 'acceptRisks' },
] as const satisfies readonly EFWFormFieldProps[];

const items = [
  { firstName: 'Samus', experience: 25, skills: [{ key: 'piloting', text: 'Pilotaje' }], acceptRisks: true },
  { firstName: 'Link', experience: 10, skills: [{ key: 'nav', text: 'Navegación' }], acceptRisks: false },
];

<EFWTable
  fields={fields}
  defaultItems={items}
  height={500}
  enableSelection
  enableCommandBar
  resizableColumns
  defaultSorting={[{ id: 'firstName', desc: false }]}
  onRowDoubleClick={(item, index) => console.log(index, item.firstName)}
  onSelectionChange={(sel) => console.log('selected', [...sel])}
/>
```

### B) CRUD con `useTableController` + interceptores async

```tsx
const CrudTable = () => {
  const { tableRef, addItem, updateItem, deleteItems, getSelectedItems,
          replaceAllItems, setLoading } = useTableController();
  const [selected, setSelected] = useState<Set<number>>(new Set());

  // Validación: throw aborta todo; void continúa.
  const beforeAddItems: EFWTableProps['beforeAddItems'] = async (items) => {
    for (const item of items) if (!item.firstName) throw new Error('El nombre es obligatorio');
  };

  // Control granular: bloquea borrado de registros protegidos.
  const beforeDeleteItems: EFWTableProps['beforeDeleteItems'] = async (items) => {
    const successful: { index: number }[] = [];
    const failed: { index: number; error?: Error }[] = [];
    for (const { index, item } of items) {
      if (item.acceptRisks === false) failed.push({ index, error: new Error('Protegido') });
      else successful.push({ index });
    }
    return { successful, failed };
  };

  const refetch = async () => {
    setLoading(true);
    const data = await fetch('/api/astronauts').then(r => r.json());
    replaceAllItems(data);
    setLoading(false);
  };

  return (
    <>
      <button onClick={() => addItem({ firstName: 'Nuevo', experience: 0 })}>Agregar</button>
      <button disabled={selected.size < 1}
        onClick={() => deleteItems(getSelectedItems().indexes)}>Eliminar</button>
      <button onClick={refetch}>Refrescar</button>

      <EFWTable
        tableRef={tableRef}
        fields={astronautFields}
        defaultItems={[]}
        height={600}
        enableSelection
        enableCommandBar
        beforeAddItems={beforeAddItems}
        beforeDeleteItems={beforeDeleteItems}
        addButtonConfig={{ enabled: false }}
        updateButtonConfig={{ enabled: false }}
        deleteButtonConfig={{ enabled: false }}
        onSelectionChange={(s) => setSelected(new Set(s))}
      />
    </>
  );
};
```

> Para usar los **drawers CRUD por defecto** (alta/edición/borrado con formulario), no desactives los botones y opcionalmente configúralos con `addButtonConfig.form` / `updateButtonConfig.form`.

---

## 10. Casos avanzados

Patrones avanzados de `EFWTable`. Cada caso incluye el shape de tipos real y un snippet mínimo basado en las stories.

### Botones de Command Bar personalizados

Para añadir acciones propias además (o en lugar) de los botones CRUD por defecto. Los 4 arrays controlan en qué escenario de selección aparece cada botón. `enabled` puede ser booleano o función `(selectedRows, tableMethods) => boolean`. Cada botón sigue el shape de `EFWGroupButtonConfig` (`kind` + `props`) y su `onClick` recibe `(event, { selectedRows, tableMethods })`.

```ts
alwaysButtons?: EFWTableCommandBarButtonInput<T>[];          // siempre visibles
noSelectionButtons?: EFWTableCommandBarButtonInput<T>[];     // 0 filas seleccionadas
singleSelectionButtons?: EFWTableCommandBarButtonInput<T>[]; // exactamente 1
multiSelectionButtons?: EFWTableCommandBarButtonInput<T>[];  // más de 1

type EFWTableCommandBarButton<T> = EFWGroupButtonConfig<CommandBarOnClickParams<T>> & {
  enabled?: boolean | ((selectedRows: EFWTableRow<EFWTableItem<T>>[], tableMethods: EFWTableMethods<T>) => boolean);
};
interface CommandBarOnClickParams<T> {
  selectedRows: EFWTableRow<EFWTableItem<T>>[]; // { index, item }
  tableMethods: EFWTableMethods<T>;
}
```

> **2 reglas para que compile (verificadas):**
>
> 1. Con `fields` **tipado** (`as const`), declara cada array con tipo explícito `EFWTableCommandBarButton<typeof fields>[]`. Inline en el JSX, `EFWTableCommandBarButtonInput<T>` es una **unión** y el `onClick` cae en implicit-any.
> 2. Con `kind: 'EFWButton'`/`'EFWDrawerButton'`, el `onClick` espera retornar `void`/`EFWButtonActionResult` (estados async del botón). Usa **cuerpo de bloque** `=> { ... }`: si retornas directo `tableMethods.addItem(...)` (que devuelve `OperationResult`) no compila.

```tsx
import {
  EFWTable,
  type EFWTableCommandBarButton,
} from '@envisiongroup/porygon/react-components/tables/EFWTable';

const alwaysButtons: EFWTableCommandBarButton<typeof fields>[] = [{
  enabled: true,
  kind: 'EFWButton',
  props: {
    appearance: 'primary',
    children: 'Agregar',
    onClick: (_e, { tableMethods }) => { tableMethods.addItem({ firstName: 'Nuevo' }); },
  },
}];

const singleSelectionButtons: EFWTableCommandBarButton<typeof fields>[] = [{
  enabled: true,
  kind: 'EFWButton',
  props: {
    children: 'Editar selección',
    onClick: (_e, { selectedRows, tableMethods }) => {
      tableMethods.updateItems(selectedRows.map(r => ({
        index: r.index, item: { firstName: `${r.item.firstName} (editado)` },
      })));
    },
  },
}];

const multiSelectionButtons: EFWTableCommandBarButton<typeof fields>[] = [{
  enabled: true,
  kind: 'EFWButton',
  props: {
    children: 'Eliminar selección',
    onClick: (_e, { selectedRows, tableMethods }) => {
      tableMethods.deleteItems(selectedRows.map(r => r.index));
    },
  },
}];

<EFWTable
  fields={fields}
  defaultItems={items}
  enableSelection
  enableCommandBar
  alwaysButtons={alwaysButtons}
  singleSelectionButtons={singleSelectionButtons}
  multiSelectionButtons={multiSelectionButtons}
/>
```

### Renderers de celda personalizados (`columnRenderers`)

Customiza el render de una celda por tipo de campo (`default`) o por campo concreto (`fields[internalName]`). Si el renderer retorna `null`/`undefined` se usa el renderer por defecto (salvo `useDefaultRendererAsFallback={false}`). Recibe `{ field, value, item, rowIndex, columnId }` tipado por tipo.

```ts
columnRenderers?: ColumnRenderers;
useDefaultRendererAsFallback?: boolean; // default true

type ColumnRenderers = {
  [K in EFWFormFieldType]?: {
    default?: CellRenderer<K>;                          // por tipo
    fields?: { [fieldName: string]: CellRenderer<K> };  // por campo concreto
  };
};
type CellRenderer<T> = (ctx: { field; value: EFWFormFieldValue<T>; item; rowIndex; columnId }) => React.ReactNode;
```

```tsx
<EFWTable
  fields={fields}
  defaultItems={items}
  columnRenderers={{
    Text: {
      default: ({ value }) => <span>{value}</span>,
      fields: {
        email: ({ value }) => <Link href={`mailto:${value}`}>{value}</Link>,
      },
    },
    Boolean: {
      default: ({ value }) => (
        <Badge appearance="tint" color={value ? 'success' : 'danger'}>{value ? 'Sí' : 'No'}</Badge>
      ),
    },
  }}
/>
```

### Configuración de columnas (`columnConfig`) e iconos (`getFieldIcon`)

Ajusta ancho/sticky/visibilidad/orden/filtro/clamp por columna sin tocar `fields`. La clave es el `internalName`. `hidden: true` oculta la columna en la tabla pero la conserva en los formularios CRUD. `getFieldIcon` añade un icono dinámico por celda según el item.

```ts
columnConfig?: Record<string, {
  maxWidth?: number; width?: number; minWidth?: number;
  sticky?: boolean;        // fija la columna en scroll horizontal
  hidden?: boolean;        // oculta en tabla, visible en drawers CRUD
  sortable?: boolean;      // default true
  filterable?: boolean;    // default true; false = excluye del drawer de filtros
  cellLineClamp?: number;  // máx líneas (Text, Note, Choice, MultiChoice, Attachments)
}>;
getFieldIcon?: (internalName: string, item: EFWTableItem<T>) => React.ReactNode;
```

```tsx
<EFWTable
  fields={fields}
  defaultItems={items}
  resizableColumns
  autoSizeColumns
  columnConfig={{
    firstName: { width: 120, minWidth: 150, maxWidth: 350, sticky: true },
    rut:       { hidden: true },            // oculta en tabla, presente en CRUD
    bio:       { minWidth: 450, cellLineClamp: 2 },
  }}
  getFieldIcon={(name, item) =>
    name === 'firstName' && item.acceptRisks ? <CheckmarkRegular /> : null}
/>
```

### Configurar drawers CRUD por defecto (`addButtonConfig` / `updateButtonConfig` / `deleteButtonConfig` / `filterButtonConfig`)

Personaliza el formulario y el drawer de los botones CRUD por defecto sin reimplementarlos. `form` acepta toda la `EFWFormProps` (sin `formRef`), incluyendo `gridTemplateColumns` y `fieldLogic`. `props.drawerConfig` controla `title`/`size`/`position`. `enabled` desactiva o condiciona el botón. `filterButtonConfig={{}}` habilita el botón de filtro con defaults.

```ts
addButtonConfig?: DefaultDrawerButtonProps<T>;
updateButtonConfig?: DefaultDrawerButtonProps<T>;
deleteButtonConfig?: DefaultDrawerButtonProps<T>;
filterButtonConfig?: DefaultFilterButtonProps<T>;

type DefaultDrawerButtonProps<T> = {
  enabled?: boolean | ((selectedRows, tableMethods) => boolean);
  insertPosition?: 'start' | 'end';
  form?: Omit<EFWFormProps<T>, 'formRef'> & { fields?: T };
  props?: EFWDrawerButtonProps & { drawerConfig?: Omit<EFWDrawerButtonProps['drawerConfig'], 'buttons'> };
};
```

```tsx
<EFWTable
  fields={fields}
  defaultItems={items}
  enableCommandBar
  addButtonConfig={{
    props: { children: 'Nuevo astronauta', drawerConfig: { size: 'medium', title: 'Nuevo elemento' } },
    form: {
      gridTemplateColumns: { autoFit: { minColumnWidth: '250px', maxColumns: 4 } },
      fieldLogic: {
        // form hereda el tipado de los `fields` de la tabla -> updateField SIN genérico
        firstName: (value, updateField, ctx) => {
          const lastName = ctx.allValues.lastName as string;
          updateField(['displayName'], { value: value ? `${value} ${lastName ?? ''}` : lastName });
        },
      },
    },
  }}
  updateButtonConfig={{ props: { children: 'Editar', drawerConfig: { size: 'full', position: 'start' } } }}
  deleteButtonConfig={{ enabled: (rows) => rows.length <= 3 }} // bloquea borrar si hay >3
  filterButtonConfig={{}}
/>
```

### Estados de overlay imperativos

Controla loading/error/empty/partial al cargar datos async. Obtén los métodos desde `useTableController()`. `setError` recibe `onRetry`. `setPartialLoading` superpone sobre datos existentes. `overlayCustomization` define textos por estado; `defaultOverlayState` arranca en un estado.

```ts
setLoading: (active: boolean, options?: { title?; description?; spinnerSize? }) => void;
setError: (active: boolean, options?: { title?; description?; onRetry?; retryButtonText? }) => void;
setPartialLoading: (active: boolean, options?: { opacity?; showSpinner? }) => void;
setEmpty: (active: boolean, options?: { title?; description?; imageUrl? }) => void;
clearOverlay: () => void;
overlayCustomization?: EFWTableOverlayCustomization;
defaultOverlayState?: EFWTableOverlayConfig; // ej. { state: 'loading' }
```

```tsx
const { tableRef, setLoading, setError, setPartialLoading, clearOverlay, replaceAllItems } =
  useTableController();

const loadData = async () => {
  setLoading(true, { title: 'Cargando astronautas...' });
  try {
    const data = await api.getItems();
    replaceAllItems(data);
    clearOverlay();
  } catch {
    setError(true, { title: 'Error de conexión', onRetry: loadData });
  }
};

<EFWTable
  fields={fields}
  tableRef={tableRef}
  defaultItems={[]}
  height={400}
  overlayCustomization={{
    empty:   { title: 'No hay registros', description: 'Presiona "Cargar"' },
    loading: { title: 'Por favor espere...', spinnerSize: 'large' },
    error:   { title: 'Ocurrió un problema', retryButtonText: 'Volver a intentar' },
  }}
/>
```

### Virtualización e infinite scroll

Virtualización para datasets grandes (miles de filas): requiere `height` (scroll interno) y se combina con modo controlled. Infinite scroll: `onScrollEnd` se dispara a `scrollEndThreshold` px del final; `loadingMore` muestra un indicador sin contar como fila. Protege contra disparos repetidos con un flag.

```ts
useVirtualization?: boolean; height?: number; itemSize?: number; // height REQUERIDO si virtualizas
onScrollEnd?: () => void; scrollEndThreshold?: number /*500*/; loadingMore?: boolean;
```

```tsx
// Virtualización (dataset grande como estado inicial, sin useEffect)
const [items, setItems] = useState<EFWTableItem[]>(() => astronautsData10000Rows);
<EFWTable fields={fields} useVirtualization height={600} itemSize={44}
  items={items} onItemsChange={setItems} />
```

```tsx
// Infinite scroll
const [items, setItems] = useState(initialData);
const [loadingMore, setLoadingMore] = useState(false);

const handleScrollEnd = useCallback(async () => {
  if (loadingMore) return;
  setLoadingMore(true);
  const next = await api.getPage();
  setItems(prev => [...prev, ...next]);
  setLoadingMore(false);
}, [loadingMore]);

<EFWTable fields={fields} items={items} onItemsChange={setItems}
  height={500} onScrollEnd={handleScrollEnd} scrollEndThreshold={500} loadingMore={loadingMore} />
```

### Sorting y filtering server-side

Por defecto sorting y filtering son **client-side** (TanStack procesa localmente). Si pasas `onSortChange`/`onFilterChange`, EFWTable activa el modo manual: conserva los íconos/UI pero delega el procesamiento. Pide los datos al backend y llama `tableMethods.replaceAllItems()`.

```ts
onSortChange?: (sortingState: EFWTableSortingState, tableMethods: EFWTableMethods<T>) => void;
onFilterChange?: (filters: FilterValues<T>, tableMethods: EFWTableMethods<T>) => void;
// columnConfig[col].sortable/filterable = false desactiva por columna
```

```tsx
const handleSortChange = async (sorting: EFWTableSortingState, tableMethods: EFWTableMethods) => {
  if (sorting.length === 0) { tableMethods.replaceAllItems(originalData); return; }
  const { id, desc } = sorting[0];
  tableMethods.setPartialLoading(true, { showSpinner: true });
  try {
    tableMethods.replaceAllItems(await api.getItems({ sortBy: id, sortDir: desc ? 'desc' : 'asc' }));
  } finally { tableMethods.setPartialLoading(false); }
};

const handleFilterChange = async (filters: FilterValues<typeof fields>, tableMethods: EFWTableMethods<typeof fields>) => {
  tableMethods.setPartialLoading(true, { showSpinner: true });
  try { tableMethods.replaceAllItems(await api.getItems({ filters })); }
  finally { tableMethods.setPartialLoading(false); }
};

<EFWTable fields={fields} defaultItems={items} height={350}
  filterButtonConfig={{}} onSortChange={handleSortChange} onFilterChange={handleFilterChange} />
```

> Valores en `FilterValues<T>` por tipo: Text/Note → `string`; Number → `number`; Boolean → `boolean`; Choice/MultiChoice/Date → `{ key, text }[]`.

### Animación de filas (`rowAnimationConfig` + `animateRows`)

Resalta filas agregadas/actualizadas vía métodos. `rowAnimationConfig` activa la animación global; la opción por llamada (`{ animateRows }`) tiene prioridad. `addItem` acepta `position: 'start' | 'end'`.

```ts
rowAnimationConfig?: { addRows?: boolean; updateRows?: boolean };
addItem(item, { position?: 'start' | 'end'; animateRows?: boolean });
updateItem(index, item, { animateRows?: boolean });
```

```tsx
const { tableRef, addItem, updateItem } = useTableController();
addItem(newItem, { position: 'start' });             // anima (config global)
addItem(newItem, { position: 'end', animateRows: false }); // override: sin animación

<EFWTable tableRef={tableRef} fields={fields} defaultItems={items} height={420}
  rowAnimationConfig={{ addRows: true, updateRows: true }} />
```
