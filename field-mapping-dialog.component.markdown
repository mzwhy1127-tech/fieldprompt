# FieldMappingDialog Component Documentation

## Overview
The `FieldMappingDialog` component is an Angular dialog component designed for configuring field mappings. It supports configuring three target fields (`question`, `answer`, `correct answer`) with their respective field types, source field separators, and source fields. The component includes functionality for adding/deleting source fields and provides save/cancel operations.

## Technical Stack
- **Angular**: Version 17+
- **Angular Material**: For UI components and dialog
- **TypeScript**: For component logic
- **SCSS**: For styling

## Component Structure
- **`FieldMappingDialogComponent`**: Main dialog component managing the overall layout and configuration data.
- **`FieldMappingRowComponent`**: Sub-component for rendering each row of the field mapping configuration.
- **`SourceFieldItemComponent`**: Sub-component for managing individual source field items.

## Inputs and Outputs
```typescript
// Inputs
@Input() configData: FieldMappingConfig[];

// Outputs
@Output() save = new EventEmitter<FieldMappingConfig[]>();
@Output() cancel = new EventEmitter<void>();
```

## Data Structures
```typescript
export interface FieldMappingConfig {
  targetField: 'question' | 'answer' | 'correct answer';
  fieldType: 'string' | 'array';
  separator?: string; // Required for 'array' type
  sourceFields: string[];
}

export interface SourceFieldItem {
  value: string;
  selected: boolean;
}
```

## Component Details

### 1. FieldMappingDialogComponent
- **Purpose**: Manages the dialog's display, layout, and configuration data.
- **Features**:
  - Displays dialog title and description.
  - Renders a table of field mappings using `FieldMappingRowComponent`.
  - Handles save and cancel actions.
- **Template**: `field-mapping-dialog.component.html`
- **Styles**: `field-mapping-dialog.component.scss`

### 2. FieldMappingRowComponent
- **Purpose**: Renders a single row of field mapping configuration.
- **Features**:
  - Displays read-only target field and field type.
  - Shows a separator input for `array` type fields.
  - Manages a list of source fields.
  - Includes buttons for adding/removing source fields.
- **Template**: `field-mapping-row.component.html`
- **Styles**: `field-mapping-row.component.scss`

### 3. SourceFieldItemComponent
- **Purpose**: Manages individual source field items.
- **Features**:
  - Displays an editable text input for the source field value.
  - Shows a status indicator (`check_circle` for the first field, `cancel` for others).
  - Provides a delete button for removing the field.
- **Template**: `source-field-item.component.html`
- **Styles**: `source-field-item.component.scss`

## Implementation

### FieldMappingDialogComponent
```typescript
// field-mapping-dialog.component.ts
import { Component, EventEmitter, Inject, Output } from '@angular/core';
import { MAT_DIALOG_DATA } from '@angular/material/dialog';
import { FieldMappingConfig } from './field-mapping.model';

@Component({
  selector: 'app-field-mapping-dialog',
  templateUrl: './field-mapping-dialog.component.html',
  styleUrls: ['./field-mapping-dialog.component.scss']
})
export class FieldMappingDialogComponent {
  constructor(@Inject(MAT_DIALOG_DATA) public configData: FieldMappingConfig[]) {}

  @Output() save = new EventEmitter<FieldMappingConfig[]>();
  @Output() cancel = new EventEmitter<void>();

  updateConfig(index: number, config: FieldMappingConfig) {
    this.configData[index] = config;
  }

  addSourceField(index: number) {
    this.configData[index].sourceFields.push('');
  }

  removeSourceField(index: number, fieldIndex: number) {
    this.configData[index].sourceFields.splice(fieldIndex, 1);
  }

  onSave() {
    this.save.emit(this.configData);
  }

  onCancel() {
    this.cancel.emit();
  }
}
```

```html
<!-- field-mapping-dialog.component.html -->
<div class="dialog-container">
  <div class="dialog-header">
    <h2>格式转换</h2>
    <h3>字段配置</h3>
    <p>仅支持string与array类型的字段进行处理</p>
  </div>

  <div class="mapping-table">
    <div class="table-header">
      <div class="header-cell">目标字段</div>
      <div class="header-cell">字段类型</div>
      <div class="header-cell">来源字段分隔符</div>
      <div class="header-cell">来源字段</div>
      <div class="header-cell">操作</div>
    </div>

    <app-field-mapping-row 
      *ngFor="let config of configData; let i = index" 
      [config]="config" 
      (configChange)="updateConfig(i, $event)"
      (removeField)="removeSourceField(i, $event)"
      (addField)="addSourceField(i)">
    </app-field-mapping-row>
  </div>

  <div class="dialog-footer">
    <button mat-raised-button color="primary" (click)="onSave()">确定</button>
    <button mat-button (click)="onCancel()">取消</button>
  </div>
</div>
```

```scss
/* field-mapping-dialog.component.scss */
.dialog-container {
  width: 800px;
  padding: 24px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}

.dialog-header {
  margin-bottom: 20px;
  
  h2 {
    font-size: 20px;
    font-weight: 600;
    color: #333;
    margin-bottom: 8px;
  }
  
  h3 {
    font-size: 16px;
    color: #666;
    margin-bottom: 4px;
  }
  
  p {
    font-size: 14px;
    color: #999;
    margin-bottom: 0;
  }
}

.mapping-table {
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 24px;
}

.table-header {
  display: flex;
  background-color: #f5f7fa;
  border-bottom: 1px solid #e0e0e0;
  
  .header-cell {
    padding: 12px 16px;
    font-weight: 600;
    font-size: 14px;
    color: #333;
    
    &:nth-child(1) { width: 15%; }
    &:nth-child(2) { width: 15%; }
    &:nth-child(3) { width: 20%; }
    &:nth-child(4) { width: 40%; }
    &:nth-child(5) { width: 10%; }
  }
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  
  button {
    min-width: 100px;
  }
}
```

### FieldMappingRowComponent
```typescript
// field-mapping-row.component.ts
import { Component, EventEmitter, Input, Output } from '@angular/core';
import { FieldMappingConfig } from './field-mapping.model';

@Component({
  selector: 'app-field-mapping-row',
  templateUrl: './field-mapping-row.component.html',
  styleUrls: ['./field-mapping-row.component.scss']
})
export class FieldMappingRowComponent {
  @Input() config!: FieldMappingConfig;
  @Output() configChange = new EventEmitter<FieldMappingConfig>();
  @Output() removeField = new EventEmitter<number>();
  @Output() addField = new EventEmitter<void>();

  onSeparatorChange(value: string) {
    this.config.separator = value;
    this.configChange.emit(this.config);
  }

  onSourceFieldChange(index: number, value: string) {
    this.config.sourceFields[index] = value;
    this.configChange.emit(this.config);
  }
}
```

```html
<!-- field-mapping-row.component.html -->
<div class="mapping-row">
  <div class="row-cell target-field">
    {{ config.targetField }}
  </div>
  
  <div class="row-cell field-type">
    {{ config.fieldType }}
  </div>
  
  <div class="row-cell separator">
    <mat-form-field *ngIf="config.fieldType === 'array'">
      <input matInput 
             [value]="config.separator || ''" 
             placeholder="请输入"
             (change)="onSeparatorChange($event.target.value)">
    </mat-form-field>
    <span *ngIf="config.fieldType !== 'array'">-</span>
  </div>
  
  <div class="row-cell source-fields">
    <div class="source-field-list">
      <ng-container *ngFor="let field of config.sourceFields; let i = index">
        <app-source-field-item 
          [value]="field" 
          [index]="i"
          (valueChange)="onSourceFieldChange(i, $event)"
          (remove)="removeField.emit(i)">
        </app-source-field-item>
      </ng-container>
    </div>
  </div>
  
  <div class="row-cell actions">
    <button mat-icon-button (click)="addField.emit()">
      <mat-icon>add</mat-icon>
    </button>
  </div>
</div>
```

```scss
/* field-mapping-row.component.scss */
.mapping-row {
  display: flex;
  border-bottom: 1px solid #e0e0e0;
  
  .row-cell {
    padding: 12px 16px;
    font-size: 14px;
    color: #333;
    
    &.target-field { width: 15%; }
    &.field-type { width: 15%; }
    &.separator { width: 20%; }
    &.source-fields { width: 40%; }
    &.actions { width: 10%; }
  }
  
  .source-field-list {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
}
```

### SourceFieldItemComponent
```typescript
// source-field-item.component.ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-source-field-item',
  templateUrl: './source-field-item.component.html',
  styleUrls: ['./source-field-item.component.scss']
})
export class SourceFieldItemComponent {
  @Input() value!: string;
  @Input() index!: number;
  @Output() valueChange = new EventEmitter<string>();
  @Output() remove = new EventEmitter<void>();

  onValueChange(event: Event) {
    this.valueChange.emit((event.target as HTMLInputElement).value);
  }
}
```

```html
<!-- source-field-item.component.html -->
<div class="source-field-item">
  <div class="field-value">
    <input type="text" [value]="value" (change)="onValueChange($event)">
  </div>
  
  <div class="field-status">
    <mat-icon *ngIf="index === 0" class="selected">check_circle</mat-icon>
    <mat-icon *ngIf="index !== 0" class="not-selected">cancel</mat-icon>
  </div>
  
  <button mat-icon-button class="remove-btn" (click)="remove.emit()">
    <mat-icon>close</mat-icon>
  </button>
</div>
```

```scss
/* source-field-item.component.scss */
.source-field-item {
  display: flex;
  align-items: center;
  gap: 8px;
  
  .field-value {
    flex: 1;
    
    input {
      width: 100%;
      padding: 8px;
      border: 1px solid #e0e0e0;
      border-radius: 4px;
      font-size: 14px;
    }
  }
  
  .field-status {
    .selected { color: #4caf50; }
    .not-selected { color: #f44336; }
  }
  
  .remove-btn {
    color: #f44336;
  }
}
```

## Module Declaration
```typescript
// field-mapping.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatDialogModule } from '@angular/material/dialog';
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { FormsModule } from '@angular/forms';
import { FieldMappingDialogComponent } from './field-mapping-dialog.component';
import { FieldMappingRowComponent } from './field-mapping-row.component';
import { SourceFieldItemComponent } from './source-field-item.component';

@NgModule({
  declarations: [
    FieldMappingDialogComponent,
    FieldMappingRowComponent,
    SourceFieldItemComponent
  ],
  imports: [
    CommonModule,
    MatDialogModule,
    MatInputModule,
    MatButtonModule,
    MatIconModule,
    FormsModule
  ],
  exports: [FieldMappingDialogComponent]
})
export class FieldMappingModule { }
```

## Usage Example
```typescript
import { Component } from '@angular/core';
import { MatDialog } from '@angular/material/dialog';
import { FieldMappingDialogComponent } from './field-mapping-dialog.component';
import { FieldMappingConfig } from './field-mapping.model';

@Component({
  selector: 'app-root',
  template: `<button mat-raised-button (click)="openMappingDialog()">Open Mapping Dialog</button>`
})
export class AppComponent {
  constructor(private dialog: MatDialog) {}

  openMappingDialog(): void {
    const dialogRef = this.dialog.open(FieldMappingDialogComponent, {
      width: '850px',
      data: [
        {
          targetField: 'question',
          fieldType: 'string',
          sourceFields: ['选项A', '选项B', '选项C']
        },
        {
          targetField: 'answer',
          fieldType: 'string',
          sourceFields: ['input']
        },
        {
          targetField: 'correct answer',
          fieldType: 'array',
          separator: ',',
          sourceFields: ['input']
        }
      ]
    });

    dialogRef.afterClosed().subscribe(result => {
      if (result) {
        console.log('Saved configuration:', result);
      }
    });
  }
}
```

## Validation
- **Separator**: Required for `array` type fields; displays an error if empty when saving.
- **Source Fields**: Must not be empty; displays an error if any source field is empty when saving.

## Development with Cursor
Use the following prompts for step-by-step development in Cursor:

1. **Main Component**: "Create an Angular Material dialog component named FieldMappingDialogComponent with a title '格式转换' and description '仅支持string与array类型的字段进行处理'."
2. **Row Component**: "Create a FieldMappingRowComponent to display target field, field type, separator input (visible only for 'array' type), and a list of source fields."
3. **Source Field Item**: "Develop a SourceFieldItemComponent with a text input, status indicator (check_circle for first field, cancel for others), and a delete button."
4. **Data Binding**: "Implement two-way data binding for configuration data, updating the model when source fields or separators are modified."
5. **Layout**: "Use Flexbox for a table layout with column widths: target field 15%, field type 15%, separator 20%, source fields 40%, actions 10%."
6. **Interactivity**: "Add functionality to add/remove source fields using '+' button for adding and '×' button for deleting."
7. **Styling**: "Apply SCSS styles for the dialog with borders, shadows, and consistent spacing for a clear visual hierarchy."
8. **Validation**: "Add validation to ensure array-type fields have a separator and all source fields are non-empty."

This implementation provides a modular, reusable, and maintainable field mapping dialog for Angular applications.