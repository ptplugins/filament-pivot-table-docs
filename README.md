<p align="center">
    <img src="logo.png" alt="Filament Pivot Table" width="200">
</p>

# Filament Pivot Table

A powerful, interactive pivot table widget for Filament. Transform your data into insightful cross-tabulations with hierarchical rows, collapsible columns, and real-time aggregations.

<p align="center">
    <img src="screenshot.png" alt="Filament Pivot Table Screenshot" width="100%">
</p>

## Demo

A live demo is available for registered users.

1. Register or log in at [plugins.premte.ch/customer](https://plugins.premte.ch/customer)
2. Open the **Demo** section in the sidebar

## Features

- **Hierarchical Rows** - Group data by multiple dimensions with expand/collapse
- **Multi-level Columns** - Nested column headers with collapse support
- **Dynamic Configuration** - Change rows, columns, and values on the fly
- **Multiple Aggregations** - Sum, Average, Count, Min, Max, Percentage
- **Row & Column Totals** - Automatic total calculations
- **Grand Total** - Overall summary row
- **Drill-Down** - Click any cell to see underlying data records
- **CSV & Excel Export** - Export visible data to CSV or Excel (configurable)
- **Dimension Reordering** - Reordering of row/column dimensions with arrows
- **Column Sorting** - Click column headers to sort data
- **URL Deep Linking** - Share specific pivot configurations via URL
- **Trend Indicators** - Show percentage change between adjacent columns
- **Heat Map** - Dynamic cell background colors based on value intensity
- **Custom Views** - Register custom views (charts, graphs) as alternatives to the table
- **Configurable Styling** - Override Tailwind CSS classes via config file
- **Dark Mode** - Full Tailwind dark mode support
- **i18n Ready** - Translation support included
- **Array Data Source** - Use raw arrays instead of Eloquent models (API, CSV, etc.)

## Installation

```bash
composer require pt-plugins/filament-pivot-table
```

Optionally publish the config file:

```bash
php artisan vendor:publish --tag=pivot-table-config
```

## Filament Resource Integration (Recommended)

The recommended way to use the pivot table is inside a Filament Resource. Extend `ListPivotRecords` to get automatic filter integration, drill-down with original records, and a consistent look within your Filament panel.

```php
<?php

namespace App\Filament\Resources\SaleResource\Pages;

use App\Filament\Resources\SaleResource;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Facades\DB;
use PtPlugins\FilamentPivotTable\Pages\ListPivotRecords;

class ListSalesPivot extends ListPivotRecords
{
    protected static string $resource = SaleResource::class;

    public function getPivotData(Builder $query): array
    {
        return $query
            ->select(
                'category',
                'region',
                'quarter',
                'month',
                DB::raw('SUM(cost) as cost'),
                DB::raw('SUM(quantity) as quantity'),
                DB::raw('COUNT(*) as count')
            )
            ->groupBy('category', 'region', 'quarter', 'month')
            ->get()
            ->toArray();
    }

    public function getDrillDownData(Builder $query, array $rowFilters, array $colFilters): array
    {
        foreach ($rowFilters as $field => $value) {
            $query->where($field, $value);
        }
        foreach ($colFilters as $field => $value) {
            $query->where($field, $value);
        }
        return $query->get()->toArray();
    }

    public function getDrillDownColumns(): array
    {
        return [
            ['name' => 'id', 'label' => 'ID', 'type' => 'numeric'],
            ['name' => 'category', 'label' => 'Category', 'type' => 'string'],
            ['name' => 'product', 'label' => 'Product', 'type' => 'string'],
            ['name' => 'region', 'label' => 'Region', 'type' => 'string'],
            ['name' => 'cost', 'label' => 'Cost', 'type' => 'numeric'],
            ['name' => 'quantity', 'label' => 'Quantity', 'type' => 'numeric'],
        ];
    }

    public function getPivotConfig(): array
    {
        return [
            'name' => 'sales-pivot',
            'availableFields' => [
                ['name' => 'category', 'label' => 'Category', 'type' => 'string'],
                ['name' => 'region', 'label' => 'Region', 'type' => 'string'],
                ['name' => 'quarter', 'label' => 'Quarter', 'type' => 'string'],
                ['name' => 'month', 'label' => 'Month', 'type' => 'string'],
                ['name' => 'cost', 'label' => 'Amount', 'type' => 'numeric'],
                ['name' => 'quantity', 'label' => 'Quantity', 'type' => 'numeric'],
                ['name' => 'count', 'label' => 'Count', 'type' => 'numeric'],
            ],
            'rowDimensions' => ['category'],
            'columnDimensions' => ['quarter', 'month'],
            'aggregationField' => 'cost',
            'aggregationType' => 'sum',
            'drillDownEnabled' => true,
        ];
    }
}
```

Then register the page in your Resource:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListSales::route('/'),
        'pivot' => Pages\ListSalesPivot::route('/pivot'),
    ];
}
```

**Key benefits:**
- Filament table filters are automatically applied to pivot data
- Drill-down shows original database records (not aggregated data)
- Filter indicators and collapsible filter panel included
- Full Resource navigation integration

## Livewire Widget

For standalone use outside of a Resource, use the Livewire widget directly in any Blade view:

```blade
@livewire('pivot-table-widget', [
    'name' => 'sales-pivot',
    'model' => \App\Models\Sale::class,
    'availableFields' => [
        ['name' => 'category', 'label' => 'Category', 'type' => 'string'],
        ['name' => 'product', 'label' => 'Product', 'type' => 'string'],
        ['name' => 'region', 'label' => 'Region', 'type' => 'string'],
        ['name' => 'quarter', 'label' => 'Quarter', 'type' => 'string'],
        ['name' => 'month', 'label' => 'Month', 'type' => 'string'],
        ['name' => 'amount', 'label' => 'Amount', 'type' => 'numeric'],
        ['name' => 'quantity', 'label' => 'Quantity', 'type' => 'numeric'],
    ],
    'rowDimensions' => ['category', 'product'],
    'columnDimensions' => ['quarter', 'month'],
    'aggregationField' => 'amount',
    'aggregationType' => 'sum',
    'valuePrefix' => '$',
    'drillDownEnabled' => true,
    'drillDownColumns' => [
        ['name' => 'id', 'label' => 'ID', 'type' => 'numeric'],
        ['name' => 'category', 'label' => 'Category', 'type' => 'string'],
        ['name' => 'amount', 'label' => 'Amount', 'type' => 'numeric'],
    ],
    'showTrends' => true,
    'trendDepth' => 1,
    'showHeatMap' => true,
    'showRowTotals' => true,
    'showGrandTotal' => true,
    'csvExportEnabled' => true,
    'xlsxExportEnabled' => true,
])
```

> This is the **full example** with all common parameters. Subsequent sections show only the **parameters that change**.

## Static Builder

For static, non-interactive pivot tables (no controls, no drill-down):

```php
use PtPlugins\FilamentPivotTable\Components\PivotTable;

$table = PivotTable::make('report')
    ->data($salesData)
    ->rowDimensions(['category', 'product'])
    ->columnDimensions(['quarter'])
    ->aggregations([
        ['field' => 'amount', 'type' => 'sum'],
    ])
    ->showRowTotals()
    ->showColumnTotals()
    ->showGrandTotal()
    ->decimalPlaces(2);

// Render in Blade
{!! $table->toHtml() !!}
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `name` | string | `'pivot-table'` | Unique identifier for the pivot table |
| `model` | string | * | Eloquent model class |
| `data` | array | * | Raw array data (alternative to model) |
| `availableFields` | array | `[]` | Fields available for pivot configuration |
| `rowDimensions` | array | `[]` | Default row grouping fields |
| `columnDimensions` | array | `[]` | Default column grouping fields |
| `aggregationField` | string | `''` | Field to aggregate |
| `aggregationType` | string | `'sum'` | Aggregation type (sum, avg, count, min, max, percentage, custom) |
| `showConfigPanel` | bool | `false` | Show/hide configuration controls |
| `showToolbar` | bool | `true` | Show toolbar with export buttons |
| `stickyRowHeaders` | bool | `false` | Enable sticky row dimension columns |
| `stickyColumnWidth` | int | `150` | Width of sticky columns in pixels |
| `rowHeight` | string | `null` | CSS height for uniform rows (e.g., '3.5rem') |
| `filters` | array | `[]` | Filters to apply to data query |
| `valuePrefix` | string | `''` | Prefix for formatted values (e.g., '$') |
| `valueSuffix` | string | `''` | Suffix for formatted values (e.g., '%') |
| `decimalPlaces` | int | `2` | Decimal places for values |
| `trendDecimalPlaces` | int | `1` | Decimal places for trend percentages |
| `drillDownEnabled` | bool | `false` | Enable click-to-drill-down on cells |
| `drillDownColumns` | array | `[]` | Columns for drill-down modal |
| `csvExportEnabled` | bool | `true` | Show/hide CSV export button |
| `xlsxExportEnabled` | bool | `true` | Show/hide Excel export button |
| `showTrends` | bool | `false` | Show trend indicators between columns |
| `trendDepth` | int\|null | `null` | Max column depth for trend display (null = all levels) |
| `showHeatMap` | bool | `false` | Enable heat map background colors |
| `showRowTotals` | bool | `true` | Show/hide the row totals column |
| `showColumnTotals` | bool | `true` | Show/hide column total rows |
| `showGrandTotal` | bool | `true` | Show/hide grand total row |
| `filtersView` | string | `null` | Custom Blade view for filters |
| `filtersViewData` | array | `[]` | Data for custom filters view |

> **Note:** Use either `model` OR `data`, not both. One is required.

### Field Definition

Each field in `availableFields`:

```php
['name' => 'field_name', 'label' => 'Display Label', 'type' => 'string']
```

Types: `string` (dimension) or `numeric` (can be aggregated).

## Array Data Source

Instead of an Eloquent model, pass raw array data. Useful for APIs, CSV files, or pre-processed data:

```php
'data' => [
    ['region' => 'North', 'product' => 'Widget A', 'quarter' => 'Q1', 'sales' => 1500],
    ['region' => 'North', 'product' => 'Widget A', 'quarter' => 'Q2', 'sales' => 1800],
    ['region' => 'South', 'product' => 'Widget B', 'quarter' => 'Q1', 'sales' => 2200],
],
'aggregationField' => 'sales',
```

When using array data:
- Filters are applied in-memory using Laravel Collections
- All filter operators work the same (`like`, `gt`, `between`, etc.)
- Drill-down functionality is fully supported

## External Filtering

Filter pivot data from parent components:

```php
'filters' => [
    'year' => 2025,
    'region' => ['North', 'South'],  // whereIn
    'status' => 'active',
],
```

### Reactive Filters from Parent Livewire Component

The `filters` property is reactive (`#[Reactive]`), so changes from a parent Livewire component will automatically refresh the pivot:

```php
// Parent component
class SalesPage extends Component
{
    public int $selectedYear = 2025;

    public function render()
    {
        return view('sales-page', [
            'pivotFilters' => ['year' => $this->selectedYear],
        ]);
    }
}
```

```blade
{{-- sales-page.blade.php --}}
<select wire:model.live="selectedYear">
    <option value="2024">2024</option>
    <option value="2025">2025</option>
</select>

@livewire('pivot-table-widget', [
    'filters' => $pivotFilters,
    // ...
])
```

## Drill-Down

Enable drill-down to allow users to click on any cell and see the underlying data records.

```php
'drillDownEnabled' => true,
'drillDownColumns' => [
    ['name' => 'order_id', 'label' => 'Order', 'type' => 'string'],
    ['name' => 'customer', 'label' => 'Customer', 'type' => 'string'],
    ['name' => 'amount', 'label' => 'Amount', 'type' => 'numeric'],
],
```

### How It Works

1. **Click on any value cell** - Cells become clickable with a hover effect
2. **Modal opens** - A Filament modal displays the filtered data
3. **Automatic filtering** - Data is filtered by the row and column dimensions of the clicked cell
4. **Drill-down filters** - Auto-generated SelectFilter dropdowns for further filtering

### Example

If your pivot table shows:

| Category | Product | North | South |
|----------|---------|-------|-------|
| Clothing |         | $500  | $300  |
|          | T-Shirt | $300  | $200  |

Clicking **$300** (T-Shirt / North) opens a modal with all sales records where `category = 'Clothing'`, `product = 'T-Shirt'`, `region = 'North'`.

## Trend Indicators

Show percentage change between adjacent columns with colored arrows.

```php
'showTrends' => true,
```

- **▲ +15.3%** (green) - Value increased vs previous column
- **▼ -8.1%** (red) - Value decreased vs previous column
- **—** (gray) - No previous column to compare

### Trend Depth (Multi-level Columns)

When using multi-level columns (e.g., Quarter > Month), `trendDepth` controls how many column levels display trend indicators:

```php
'showTrends' => true,
'columnDimensions' => ['quarter', 'month'],
'trendDepth' => 1,  // Trends only on Quarter level, not on Month
```

| trendDepth | Behavior |
|------------|----------|
| `null` (default) | Trends on all column levels (backward compatible) |
| `1` | Trends only on the first column level (e.g., Quarter) |
| `2` | Trends on first two levels (e.g., Quarter + Month) |

When a column group is collapsed, the collapsed cell shows the parent-level trend regardless of `trendDepth`, as long as `showTrends` is enabled.

### Custom Trend Calculation

```php
use PtPlugins\FilamentPivotTable\Components\PivotTableWidget;

PivotTableWidget::trendUsing(function (float $current, float $previous): ?float {
    return $current - $previous; // Absolute difference instead of percentage
});
```

### Previous Period Value (First Column)

Supply a custom closure for the first column's comparison value:

```php
PivotTableWidget::previousPeriodValueUsing(
    function (string $columnKey, array $rowFilters, string $field): ?float {
        return Sale::query()
            ->where($rowFilters)
            ->where('quarter', '2024-Q4')
            ->sum($field);
    }
);
```

## Heat Map

Color data cells with dynamic background intensity based on their value.

```php
'showHeatMap' => true,
```

- Min/max values are calculated automatically across all cells
- **Light mode**: green tones (opacity 0–0.35)
- **Dark mode**: lighter green tones (opacity 0–0.35)
- Total cells are not colored

### Custom Heat Map Colors

```php
PivotTableWidget::heatMapColorUsing(function (float $value, float $min, float $max): ?string {
    $intensity = ($value - $min) / ($max - $min);
    return "rgba(59, 130, 246, " . ($intensity * 0.4) . ")"; // Blue theme
});
```

### Combining Trends and Heat Map

```php
'showTrends' => true,
'showHeatMap' => true,
```

Cells get a heat-colored background with trend arrows below each value.

## Styling

The pivot table uses Tailwind CSS classes that can be fully customized via the config file.

Publish the config:

```bash
php artisan vendor:publish --tag=pivot-table-config
```

Then override any class in `config/pivot-table.php`:

```php
'classes' => [
    'container' => 'bg-white dark:bg-gray-900 rounded-lg shadow-sm border border-gray-200 dark:border-gray-700 overflow-hidden',
    'thead' => 'bg-gray-100 dark:bg-gray-800 text-gray-700 dark:text-gray-200 border-b-2 border-gray-300 dark:border-gray-600',
    'row' => 'hover:bg-gray-50 dark:hover:bg-gray-800 transition-colors',
    'cell' => 'px-4 py-2 text-right tabular-nums',
    'cell_drilldown' => 'px-4 py-2 text-right tabular-nums cursor-pointer hover:bg-primary-50 dark:hover:bg-primary-900/20',
    'grand_total_row' => 'bg-gray-100 dark:bg-gray-800 font-bold',
    'expand_button' => 'w-5 h-5 flex-shrink-0 flex items-center justify-center text-gray-500 hover:text-gray-700 rounded hover:bg-gray-200',
    // ... see config file for all available keys
],
```

## Preset Actions

Add toolbar buttons that apply predefined configurations or toggle visual options. Override `presetActions()` in your `ListPivotRecords` page:

```php
public function presetActions(): array
{
    return [
        'by-category' => [
            'label' => 'By Category',
            'icon' => 'grid',           // grid, chart, or layers
            'pivotConfig' => [
                'rows' => ['category', 'region'],
                'columns' => ['quarter'],
                'value' => 'cost',
                'aggregation' => 'sum',
            ],
        ],
        'by-region' => [
            'label' => 'By Region',
            'icon' => 'layers',
            'pivotConfig' => [
                'rows' => ['region'],
                'columns' => ['month'],
                'value' => 'quantity',
                'aggregation' => 'sum',
            ],
        ],
        'toggle-trends' => [
            'label' => 'Toggle Trends',
            'icon' => 'chart',
            'toggle' => 'showTrends',   // toggles the option on/off
        ],
        'toggle-heatmap' => [
            'label' => 'Toggle Heatmap',
            'icon' => 'chart',
            'toggle' => 'showHeatMap',
        ],
    ];
}
```

### Preset Action Types

**Preset** (`pivotConfig`) — Sets the full pivot configuration (rows, columns, value, aggregation). Can also set `showTrends` and `showHeatMap`.

**Toggle** (`toggle`) — Toggles a single boolean option on/off. Supported options: `showTrends`, `showHeatMap`, `showRowTotals`, `showColumnTotals`, `showGrandTotal`.

### Available Icons

| Icon | Description |
|------|-------------|
| `grid` | Grid/layout icon (good for presets) |
| `chart` | Trend/chart icon (good for toggles) |
| `layers` | Stacked layers icon (good for grouped presets) |

## Programmatic Configuration API

Control the pivot table configuration from parent components using Livewire events.

### Setting Configuration

```php
$this->dispatch('set-pivot-configuration', [
    'rows' => ['category', 'region'],
    'columns' => ['quarter'],
    'value' => 'cost',
    'aggregation' => 'sum',
    'showTrends' => true,
    'showHeatMap' => true,
    'trendDepth' => 1,
    'showRowTotals' => false,
    'showGrandTotal' => false,
]);
```

### Toggling Options

```php
$this->dispatch('toggle-pivot-option', 'showTrends');
$this->dispatch('toggle-pivot-option', 'showHeatMap');
$this->dispatch('toggle-pivot-option', 'showRowTotals');
$this->dispatch('toggle-pivot-option', 'showColumnTotals');
$this->dispatch('toggle-pivot-option', 'showGrandTotal');
```

### Getting Current Configuration

```php
protected $listeners = [
    'pivot-configuration-ready' => 'handlePivotConfiguration',
];

public function requestConfiguration(): void
{
    $this->dispatch('request-pivot-configuration',
        context: ['action' => 'save', 'saveName' => 'my-config']
    );
}

public function handlePivotConfiguration(array $configuration, array $context = []): void
{
    // $configuration = ['rows' => [...], 'columns' => [...], 'value' => '...', 'aggregation' => '...']
    session(["pivot_config_{$context['saveName']}" => $configuration]);
}
```

## URL Deep Linking

The pivot table automatically syncs state to URL parameters:

```
/admin/sales-pivot?rows[0]=category&rows[1]=product&cols[0]=quarter&value=amount&agg=sum
```

Users can share specific configurations, bookmark views, and use browser back/forward.

## Custom Views

Register alternative views (charts, graphs) that users can switch to from the Controls panel.

### Registering a Custom View

```php
// In AppServiceProvider::boot()
use PtPlugins\FilamentPivotTable\Components\PivotTableWidget;

PivotTableWidget::registerView('pie', 'Pie Chart', 'components.pivot-views.pie-chart');
PivotTableWidget::registerView('bar', 'Bar Chart', 'components.pivot-views.bar-chart');
```

### Available Variables in Custom Views

| Variable | Type | Description |
|----------|------|-------------|
| `$pivotData` | array | Full pivot data (rows, columns, totals) |
| `$rawData` | array\|null | Raw data array (if using array source) |
| `$modelClass` | string | Eloquent model class (if using model) |
| `$filters` | array | Active filters |
| `$selectedRowDimensions` | array | Current row dimensions |
| `$selectedColumnDimensions` | array | Current column dimensions |
| `$selectedAggregationField` | string | Aggregation field |
| `$selectedAggregationType` | string | Aggregation type |
| `$valuePrefix` | string | Value prefix |
| `$valueSuffix` | string | Value suffix |
| `$name` | string | Pivot table name |

## Column Sorting

Click any column header to sort pivot rows by that column's values.

1. **First click** - Sort descending (highest first)
2. **Second click** - Sort ascending (lowest first)
3. **Third click** - Reset to original order

Works with single-level columns, multi-level child headers, and the Total column.

## Expand/Collapse

### Rows
Click the arrow icon next to any parent row to collapse/expand its children.

### Columns
Click a parent column header (e.g., "Q1") to collapse all child columns into a single aggregated column showing the sum.

## Export Configuration

Export is **client-side** (browser-only). CSV uses pure JavaScript, Excel uses SheetJS from CDN. No data sent to server.

```php
'csvExportEnabled' => false,   // Hide CSV export
'xlsxExportEnabled' => true,   // Show Excel export
```

Export settings also apply to the drill-down modal. The SheetJS CDN URL can be customized in `config/pivot-table.php`.

## Dimension Reordering

1. Open the configuration panel (click "Show Controls")
2. Each dimension badge shows arrow buttons: **↑** (up) / **↓** (down)
3. The pivot table re-renders immediately with the new order

## Translations

```bash
php artisan vendor:publish --tag=pivot-table-translations
```

Available locales: `en`, `sr`

## Configuration Quick Reference

### Financial Report

```php
'rowDimensions' => ['department', 'account'],
'columnDimensions' => ['quarter', 'month'],
'aggregationField' => 'amount',
'aggregationType' => 'sum',
'valuePrefix' => '$',
'showTrends' => true,
'showHeatMap' => true,
'drillDownEnabled' => true,
```

### Inventory Dashboard

```php
'rowDimensions' => ['warehouse', 'product'],
'columnDimensions' => ['month'],
'aggregationField' => 'stock_level',
'aggregationType' => 'avg',
'showHeatMap' => true,
'stickyRowHeaders' => true,
```

### Customer Analytics

```php
'rowDimensions' => ['segment', 'channel'],
'columnDimensions' => ['year', 'quarter'],
'aggregationField' => 'customer_id',
'aggregationType' => 'count',
'showTrends' => true,
```

### Sales Performance

```php
'rowDimensions' => ['region', 'salesperson'],
'columnDimensions' => ['quarter'],
'aggregationField' => 'revenue',
'aggregationType' => 'sum',
'valuePrefix' => '$',
'drillDownEnabled' => true,
'csvExportEnabled' => true,
'xlsxExportEnabled' => true,
```

## Custom Aggregation

You can define your own aggregation logic using the `custom` aggregation type.

### Aggregation Type Constants

Use the `AggregationType` class for cleaner code:

```php
use PtPlugins\FilamentPivotTable\Enums\AggregationType;

'aggregationType' => AggregationType::SUM,      // 'sum'
'aggregationType' => AggregationType::AVG,      // 'avg'
'aggregationType' => AggregationType::COUNT,    // 'count'
'aggregationType' => AggregationType::MIN,      // 'min'
'aggregationType' => AggregationType::MAX,      // 'max'
'aggregationType' => AggregationType::PERCENTAGE, // 'percentage'
'aggregationType' => AggregationType::CUSTOM,   // 'custom'
```

### Custom Aggregation Callback

Register a custom aggregation function in your `AppServiceProvider`:

```php
use Illuminate\Support\Collection;
use PtPlugins\FilamentPivotTable\Components\PivotTableWidget;

public function boot(): void
{
    // Weighted average example
    PivotTableWidget::customAggregationUsing(
        function (Collection $data, string $field, float $grandTotal): float|int {
            $totalWeight = $data->sum('quantity');
            if ($totalWeight === 0) {
                return 0;
            }

            return $data->sum(fn ($row) => $row[$field] * $row['quantity']) / $totalWeight;
        }
    );
}
```

Then use it in your widget:

```php
@livewire('pivot-table-widget', [
    'aggregationType' => 'custom',
    // ...
])
```

## Requirements

- PHP 8.1+
- Laravel 10+
- Filament 3.x / 4.x / 5.x
- Livewire 3.x

## License

MIT License. See [LICENSE](LICENSE) for details.

## Support

- Email: plugins@premte.ch
