---
layout: single
title:  "Create data grid in angular with primeng"
date:   2024-10-29 07:14:38 -0600
categories: angular primeng editable table datagrid grid
---

### Define Column Types and Cell Types

```typescript
enum TableCellType {
  Input = 'input',
  InputNumber = 'inputNumber',
  Checkbox = 'checkbox',
  Dropdown = 'dropdown',
  Icon = 'icon',
  Date = 'date',
  Link = 'link',
}
```

```typescript
type TableColumn = {
  header: string;header.
  field: string;
  cellType?: TableCellType;
  width?: string;
  editable?: boolean;
  align?: 'left' | 'right' | 'center';
  iconClass?: string;
  iconOnClick?: AnyFunc;
};
```

* header: `string`: Represents the column's display name in the table header.
* field: `string`: Specifies the key in the data source that the column will use.

### Define Columns

```typescript
cols: TableColumn[] = [
    {
      header: 'Code',
      field: 'code',
      editable: true,
      cellType: TableCellType.Input,
      width: '100px',
    },
    // More column definitions...
  ];
```

### Define Form Group

To handle the data updates, I use a FormGroup that keeps track of each row’s data:

```typescript
formGroup = new FormGroup({
  id: new FormControl(),
  code: new FormControl(),
  name: new FormControl(),
  active: new FormControl(),
  price: new FormControl(),
  category: new FormControl(),
  quantity: new FormControl(),
});
```

Since only one row can be edited at a time, the row's data is updated into the form group during editing. To listen for editing events, I use `onEditInit` and `onEditComplete`

```typescript
onEditInit(event: TableEditInitEvent) {
  const entity = this.products[event.index];
  this.formGroup.setValue(
    pick(entity, ['id', 'code', 'name', 'active', 'price', 'category', 'quantity']),
    { emitEvent: false }
  );
}
```

```typescript
onEditComplete(event: TableEditCompleteEvent) {
  if (this.formGroup.dirty && event.index) {
    this.products = [
      ...this.products.slice(0, event.index),
      this.formGroup.value,
      ...this.products.slice(event.index + 1),
    ];
    this.formGroup.markAsPristine();
  }
}
```

### Initialize PrimeNG Table

The table is initialized in the HTML as follows:

```html
<p-table
  [value]="products"
  dataKey="id"
  [tableStyle]="{ 'min-width': '50rem' }"
  selectionMode="single"
  [columns]="cols"
  (onEditInit)="onEditInit($event)"
  (onEditComplete)="onEditComplete($event)"
  >
</p-table>
```

### Initialize the Table Colgroup

To control the column widths:

```html
<ng-template pTemplate="colgroup" let-columns>
  <colgroup>
    <col
      *ngFor="let col of columns"
      span="1"
      [style.width]="col.width ? col.width : 'auto'"
    />
  </colgroup>
</ng-template>
```

### Initialize the Table Header

The header is rendered as follows:

```html
<ng-template pTemplate="header" let-columns>
  <tr>
    <th
      *ngFor="let col of columns"
      [class.text-center]="col.align === 'center'"
      [class.text-right]="col.align === 'right'"
    >
      {% raw %}{{ col.header }}{% endraw %}
    </th>
  </tr>
</ng-template>
```

### Initialize the Table Body

For rendering the table body dynamically and handling inline editing:

```html
<ng-template
  pTemplate="body"
  let-rowData
  let-columns="columns"
  let-i="rowIndex"
>
  <tr>
    <td
      *ngFor="let col of columns"
      [class.text-center]="col.align === 'center'"
      [class.text-right]="col.align === 'right'"
      [pEditableColumn]="rowData[col.field]"
      [pEditableColumnField]="col.field"
      [pEditableColumnRowIndex]="i"
      [pEditableColumnDisabled]="!col.editable"
    >
      <p-cellEditor>
        <ng-template pTemplate="input">
          @switch (col.cellType) {
            @case (cellType.InputNumber) {
              <form [formGroup]="formGroup">
                <p-inputNumber [formControlName]="col.field" inputStyleClass="w-full text-right" />
              </form>
            }
            @default {
              <form [formGroup]="formGroup">
                <input [formControlName]="col.field" pInputText class="w-full" />
              </form>
            }
          }
        </ng-template>
        <ng-template pTemplate="output">
          @switch (col.cellType) {
            @case (cellType.Dropdown) {
              <p-dropdown 
                [options]="categories" 
                optionLabel="label"
                optionValue="label"
                appendTo="body"
                [ngModel]="rowData[col.field]"
                placeholder="Select a category" />
            }
            @case (cellType.Checkbox) {
              <p-checkbox [binary]="true"  [ngModel]="rowData[col.field]"/>
            }
            @case (cellType.Icon) {
              <i
                class="{% raw %}{{ col.iconClass }}{% endraw %}"
                [ngStyle]="{ cursor: 'pointer' }"
                (click)="
                  col.iconOnClick &&
                    col.iconOnClick(rowData[col.field], i, $event)
                "
              ></i>
            }
            @default  {
              {% raw %}{{ rowData[col.field] }}{% endraw %}
            }
          }
        </ng-template>
      </p-cellEditor>
    </td>
  </tr>
</ng-template>
```

The complete source code can be found on [Stackblitz](https://stackblitz.com/~/github.com/haku-d/ng-editable-table).


## Summary

You may have noticed that the left and right arrow keys trigger a focus event on the table, which should ideally be disabled while editing a cell. To achieve this, you can create a custom directive and apply it to each cell to override this behavior.

Additionally, you can enhance the grid by adding a feature that selects the cell's data automatically when it receives focus, making editing more convenient.

Just drop me a message if you have any question.

Thanks for reading!

---