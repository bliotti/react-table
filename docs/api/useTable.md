# `useTable`

- Required

`useTable` is the root hook for React Table. To use it, pass it with an options object with at least a `columns` and `data` value, followed by any React Table compatible hooks you want to use.

### Table Options

The following options are supported via the main options object passed to `useTable(options)`

- `columns: Array<Column>`
  - Required
  - Must be **memoized**
  - The core columns configuration object for the entire table.
  - Supports nested `columns` arrays via the column's `columns` key, eg. `[{ Header: 'My Group', columns: [...] }]`
- `data: Array<any>`
  - Required
  - Must be **memoized**
  - The data array that you want to display on the table.
- `initialState: Object`
  - Optional
  - The initial state object for the table.
  - Upon table initialization, this object is **merged over the table's `defaultState` object** (eg. `{...defaultState, ...initialState}`) that React Table and its hooks use to register default state to produce the final initial state object passed to the `React.useState` hook internally.
- `initialState.hiddenColumns: Array<ColumnId: String>`
  - Optional
  - The initial state object for hidden columns
  - If a column's ID is contained in this array, it will be hidden
- `reducer: Function(newState, action, prevState) => newState`
  - Optional
  - With every action that is dispatched to the table's internal `React.useReducer` instance, this reducer is called and is allowed to modify the final state object for updating.
  - It is passed the `newState`, `action`, and `prevState` and is expected to either return the `newState` or a modified version of the `newState`
  - May also be used to "control" the state of the table, by overriding certain pieces of state regardless of the action.
- `useControlledState: HookFunction(state) => controlledState`
  - Optional
  - If you need to control part of the table state, this is the place to do it.
  - This function is run on every single render, just like a hook and allows you to alter the final state of the table if necessary.
  - You can use hooks inside of this function, but most of the time, we just suggest using `React.useMemo` to memoize your state overrides.
  - See the FAQ ["How can I manually control the table state?"](./faq.md#how-can-i-manually-control-the-table-state) for a an example.
- `defaultColumn: Object`
  - Optional
  - Defaults to `{}`
  - The default column object for every column passed to React Table.
  - Column-specific properties will override the properties in this object, eg. `{ ...defaultColumn, ...userColumn }`
  - This is particularly useful for adding global column properties. For instance, when using the `useFilters` plugin hook, add a default `Filter` renderer for every column, eg.`{ Filter: MyDefaultFilterComponent }`
- `initialRowStateKey: String`
  - Optional
  - Defaults to `initialState`
  - This key is used to look for the initial state of a row when initializing the `rowState` for a`data` array.
  - If the value located at `row[initialRowStateKey]` is falsey, `{}` will be used instead.
- `getSubRows: Function(row, relativeIndex) => Rows[]`
  - Optional
  - Must be **memoized**
  - Defaults to `(row) => row.subRows || []`
  - Use this function to change how React Table detects subrows. You could even use this function to generate sub rows if you want.
  - By default, it will attempt to return the `subRows` property on the row, or an empty array if that is not found.
- `getRowId: Function(row, relativeIndex) => string`
  - Use this function to change how React Table detects unique rows and also how it constructs each row's underlying `path` property.
  - Optional
  - Must be **memoized**
  - Defaults to `(row, relativeIndex) => relativeIndex`
  - You may want to change this function if
  - By default, it will use the `index` of the row within it's original array.
- `debug: Bool`
  - Optional
  - A flag to turn on debug mode.
  - Defaults to `false`

### Column Options

The following options are supported on any column object you can pass to `columns`.

- `accessor: String | Function`
  - **Required**
  - This string/function is used to build the data model for your column.
  - The data returned by an accessor should be **primitive** and sortable.
  - If a string is passed, the column's value will be looked up on the original row via that key, eg. If your column's accessor is `firstName` then its value would be read from `row['firstName']`. You can also specify deeply nested values with accessors like `info.hobbies` or even `address[0].street`
  - If a function is passed, the column's value will be looked up on the original row using this accessor function, eg. If your column's accessor is `row => row.firstName`, then its value would be determined by passing the row to this function and using the resulting value.
- `id: String`
  - **Required if `accessor` is a function**
  - This is the unique ID for the column. It is used by reference in things like sorting, grouping, filtering etc.
  - If a **string** accessor is used, it defaults as the column ID, but can be overridden if necessary.
- `columns: Array<Column>`
  - Optional
  - A nested array of columns.
  - If defined, the column will act as a header group. Columns can be recursively nested as much as needed.
- `Header: String | Function | React.Component => JSX`
  - Optional
  - Defaults to `() => null`
  - Receives the table instance and column model as props
  - Must either be a **string or return valid JSX**
  - If a function/component is passed, it will be used for formatting the header value, eg. You can use a `Header` function to dynamically format the header using any table or column state.
- `Cell: Function | React.Component => JSX`
  - Optional
  - Defaults to `({ cell: { value } }) => String(value)`
  - Receives the table instance and cell model as props
  - Must return valid JSX
  - This function (or component) is primarily used for formatting the column value, eg. If your column accessor returns a date object, you can use a `Cell` function to format that date to a readable format.
- `width: Int`
  - Optional
  - Defaults to `150`
  - Specifies the width for the column (when using non-table-element layouts)
- `minWidth: Int`
  - Optional
  - Defaults to `0`
  - Specifies the minimum width for the column (when using non-table-element layouts)
  - Specifically useful when using plugin hooks that allow the user to resize column widths
- `maxWidth: Int`
  - Optional
  - Defaults to `0`
  - Specifies the maximum width for the column (when using non-table-element layouts)
  - Specifically useful when using plugin hooks that allow the user to resize column widths

### Instance Properties

The following properties are available on the table instance returned from `useTable`

- `state: Object`
  - **Memoized** - This object reference will not change unless the internal table state is modified.
  - This is the final state object of the table, which is the product of the `initialState`, internal table reducer and (optionally) a custom `reducer` supplied by the user.
- `dispatch: Function({ type: Actions[type], ...payload }) => void`
  - This function is used both internally by React Table, and optionally by you (the developer) to update the table state programmatically.
  - `type: Actions[type] | String`
    - The action type corresponding to what action being taken against the state.
  - `...payload`
    - Any other action data that is associated with the action
- `columns: Array<Column>`
  - A **nested** array of final column objects, **similar in structure to the original columns configuration option**.
  - See [Column Properties](#column-properties) for more information
- `flatColumns: Array<Column>`
  - A **flat** array of all final column objects.
  - See [Column Properties](#column-properties) for more information
- `headerGroups: Array<HeaderGroup>`
  - An array of normalized header groups, each containing a flattened array of final column objects for that row.
  - **Some of these headers may be materialized as placeholders**
  - See [HeaderGroup Properties](#headergroup-properties) for more information
- `footerGroups: Array<HeaderGroup>`
  - An array of normalized header groups, but in reverse order, each containing a flattened array of final column objects for that row.
  - **Some of these headers may be materialized as placeholders**
  - See [HeaderGroup Properties](#headergroup-properties) for more information
- `headers: Array<Column>`
  - A **nested** array of final header objects, **similar in structure to the original columns configuration option, but rebuilt for ordering**
  - Each contains the headers that are displayed underneath it.
  - **Some of these headers may be materialized as placeholders**
  - See [Column Properties](#column-properties) for more information
- `flatHeaders[] Array<Column>`
  - A **flat** array of final header objects found in each header group.
  - **Some of these headers may be materialized as placeholders**
  - See [Column Properties](#column-properties) for more information
- `rows: Array<Row>`
  - An array of **materialized row objects** from the original `data` array and `columns` passed into the table options
  - See [Row Properties](#row-properties) for more information
- `getTableProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for your table wrapper.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `getTableBodyProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for your table body wrapper.
  - Custom props may be passed. **NOTE: Custom props will override built-in table body props, so be careful!**
- `prepareRow: Function(Row)`
  - **Required**
  - This function is responsible for lazily preparing a row for rendering. Any row that you intend to render in your table needs to be passed to this function **before every render**.
  - **Why?** Since table data could potentially be very large, it can become very expensive to compute all of the necessary state for every row to be rendered regardless if it actually is rendered or not (for example if you are paginating or virtualizing the rows, you may only have a few rows visible at any given moment). This function allows only the rows you intend to display to be computed and prepped with the correct state.
- `flatRows: Array<Row>`
  - An array of all rows, including subRows which have been flattened into the order in which they were detected (depth first)
  - This can be helpful in calculating total row counts that must include subRows
- `totalColumnsWidth: Int`
  - This is the total width of all visible columns (only available when using non-table-element layouts)
- `toggleHideColumn: Function(columnId: String, ?value: Boolean) => void`
  - This function can be used to toggle or set a column's hidden state
  - Passing a `value` is optional. If passed, the hidden state will be set to that Boolean value.
  - If a `value` is not passed, the visibility for this column will be toggled.
- `setHiddenColumns: Function(Array<ColumnId: String> | Function(oldHiddenColumns) => Array<ColumnId: String>) => void`
  - This function can be used to set the `hiddenColumns` state for the entire table.
  - If a value is passed, `hiddenColumns` will be set to that value
  - If a function is passed, it will received the previous `hiddenColumns` value and will be expected to return the new value.
- `toggleHideAllColumns: Function(?value: Boolean) => void`
  - This function can be used to toggle or set the visibility for all columns to `true` or `false`
  - If a value is not passed, the visibility for all columns will be toggled back and forth from `true` to `false`
  - If `true` is passed, all columns will be hidden
  - If a `false` is passed, all columns will be visible
- `getToggleHideAllColumnsProps: Function(userProps) => props`
  - This function can be used to retrieve all necessary props to be placed on an `<input type='checkbox'>` component that will control the visibility of all columns

### HeaderGroup Properties

The following additional properties are available on every `headerGroup` object returned by the table instance.

- `headers: Array<Column>`
  - **Required**
  - The columns in this header group.
- `getHeaderGroupProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this header group's row.
  - You can use the `getHeaderGroupProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `getFooterGroupProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this header group's footer row.
  - You can use the `getFooterGroupProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**

### Column Properties

The following properties are available on every `Column` object returned by the table instance.

- `id: String`
  - The resolved column ID from either the column's `accessor` or the column's hard-coded `id` property
- `isVisible: Boolean`
  - Whether the column should be currently visible or not.
  - Columns that are not visible are still used for sorting, filtering, etc.
- `render: Function(type: String | Function | Component, ?props)`
  - This function is used to render content with the added context of a column.
  - The entire table `instance` will be passed to the renderer with the addition of a `column` property, containing a reference to the column
  - If `type` is a string, will render using the `column[type]` renderer. React Table ships with default `Header` and `Footer` renderers. Other renderers like `Filter` and `Aggregated` are available via plugin hooks.
  - If a function or component is passed instead of a string, it will be be passed the table instance and column model as props and is expected to return any valid JSX.
- `totalLeft: Int`
  - This is the total width in pixels of all columns to the left of this column
  - Specifically useful when using plugin hooks that allow the user to resize column widths
- `totalWidth: Int`
  - This is the total width in pixels for this column (if it is a leaf-column) or or all of it's sub-columns (if it is a column group)
  - Specifically useful when using plugin hooks that allow the user to resize column widths
- `getHeaderProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this column's header cell.
  - You can use the `getHeaderProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `getFooterProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this column's footer cell.
  - You can use the `getFooterProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `toggleHidden: Function(?hidden: Boolean) => void`
  - This function can be used to hide or show this column.
  - If no value is passed, the visibility of this column will be toggled.
  - Optionally pass a value to set the visible.
- `getToggleHiddenProps: Function(userProps) => props`
  - This function can be used to retrieve all necessary props to be placed on an `<input type='checkbox'>` component that will control the visibility of this column.

### Row Properties

The following additional properties are available on every `row` object returned by the table instance.

- `cells: Array<Cell>`
  - An array of `Cell` objects containing properties and functions specific to the row and column it belongs to.
  - See [Cell Properties](#cell-properties) for more information
- `values: Object<columnId: any>`
  - A map of this row's **resolved** values by columnId, eg. `{ firstName: 'Tanner', lastName: 'Linsley' }`
- `getRowProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this row.
  - You can use the `getRowProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `index: Int`
  - The index of the original row in the `data` array that was passed to `useTable`. If this row is a subRow, it is the original index within the parent row's subRows array
- `original: Object`
  - The original row object from the `data` array that was used to materialize this row.
- `path: Array<string>`
  - This array is the sequential path of indices one could use to navigate to it, eg. a row path of `[3, 1, 0]` would mean that it is the **first** subRow of a parent that is the **second** subRow of a parent that is the **fourth** row in the original `data` array.
  - This array is used with plugin hooks like `useExpanded` and `useGroupBy` to compute expanded states for individual rows.
- `subRows: Array<Row>`
  - If subRows were detect on the original data object, this will be an array of those materialized row objects.
- `state: Object`

  - The current state of the row. It's lifespan is attached to that of the original `data` array. When the raw `data` is changed, this state value is reset to the row's initial value (using the `initialRowStateKey` option).
  - Can be updated via `instance.setRowState` or the row's `setState` function.

### Cell Properties

The following additional properties are available on every `Cell` object returned in an array of `cells` on every row object.

- `column: Column`
  - The corresponding column object for this cell
- `row: Row`
  - The corresponding row object for this cell
- `value: any`
  - The **resolved** value for this cell.
  - By default, this value is displayed on the table via the default `Cell` renderer. To override the way a cell displays
- `getCellProps: Function(?props)`
  - **Required**
  - This function is used to resolve any props needed for this cell.
  - You can use the `getCellProps` hook to extend its functionality.
  - Custom props may be passed. **NOTE: Custom props will override built-in table props, so be careful!**
- `render: Function(type: String | Function | Component, ?props)`
  - **Required**
  - This function is used to render content with the added context of a cell.
  - The entire table `instance` will be passed to the renderer with the addition of `column`, `row` and `cell` properties, containing a reference to each respective item.
  - If `type` is a string, will render using the `column[type]` renderer. React Table ships with a default `Cell` renderer. Other renderers like `Aggregated` are available via hooks like `useFilters`.
  - If a function or component is passed instead of a string, it will be be passed the table instance and cell model as props and is expected to return any valid JSX.

### Example

- [Source](https://github.com/tannerlinsley/react-table/tree/master/examples/basic)
- [Open in CodeSandbox](https://codesandbox.io/s/github/tannerlinsley/react-table/tree/master/examples/basic)
