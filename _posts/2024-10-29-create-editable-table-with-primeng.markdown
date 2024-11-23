---
layout: single
title:  "Create editable table with angular and primeng"
date:   2024-10-29 07:14:38 -0600
categories: angular primeng editable table
---

PrimeNG has provided a very clear instruction on how to create a table that can be directly edited by cell or row. But that is just the most basic guide to getting started with PrimeNG table.

In this post, I will show you how I customize PrimeNG by changing the default theme, rendering columns dynamically, collecting edited data with reactive forms.

### I will start with defining type for a column and its cell type

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
};
```

* header: string
  * Represents the name of the column as it will appear in the table header.

* field: string
  * Specifies the key in the data source that this column should reference.

### Define columns

```typescript
cols: TableColumn[] = [
    {
      header: 'Code',
      field: 'code',
      editable: true,
      cellType: TableCellType.Input,
      width: '100px',
    },
    ...
  ];
```

### Define form group

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

* Suppose, at a time only one row is editing. Once any column is editing, the data of the corresponding row will be updated to the form group. So to listen for edit events on the table, I use `onEditInit` and `onEditComplete`.

```typescript
onEditInit(event: TableEditInitEvent) {
  const entity = this.products[event.index];
  this.formGroup.setValue(
    pick(entity, [
      'id',
      'code',
      'name',
      'active',
      'price',
      'category',
      'quantity',
    ]),
    {
      emitEvent: false,
    }
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

### Initialize prime table

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

### Init table colgroup

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

### Init table header

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

### Init the table body template

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
                <p-inputNumber [formControlName]="col.field" inputStyleClass="w-full" />
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

The complete source code can be found on [Stackblitz](https://stackblitz.com/~/github.com/haku-d/ng-editable-table)

---