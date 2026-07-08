---
name: starter-kit-commands
description: Reference for all custom Artisan commands and standalone scripts in this Laravel Starter Kit project: package:make, make:package, make:crudly, route:find-duplicates, app:install, package:seed, and extract-translations.php. Use this skill when the user asks you to run, explain, scaffold, or create any of these commands — or when they mention creating packages, CRUD modules, seeding, translation extraction, or any project scaffolding task. Also use when the user asks about the field types available for CRUD generation, package structure, or how the module/permission system works.
---

# Laravel Starter Kit — Custom Commands

## Quick Reference

| Command | Purpose |
|---------|---------|
| `package:make {name}` | Scaffold a new self-contained React package |
| `make:package {type} {name} {package}` | Add a component to an existing package |
| `make:crudly {name} [options]` | Generate full CRUD module (backend + frontend) |
| `route:find-duplicates` | Find duplicate route names with file paths |
| `app:install [--force]` | Fresh install: migrate, seed, enable all packages |
| `package:seed {packageName?}` | Seed one or all packages |
| `extract-translations.php [PackageName]` | Extract i18n strings to JSON files |

---

## `package:make {name}` — Create a new React package

Generates a complete self-contained package under `packages/{Name}/` using stubs from `stubs/react-package-stubs/`.

**Argument:**
- `name` (required) — PascalCase package name, e.g. `Blog` → `packages/Blog/`

**What it generates:**
- `composer.json` and `module.json` (package metadata)
- Service providers (`ServiceProvider`, `EventServiceProvider`)
- Database: migrations, seeders (`DatabaseSeeder`, `PermissionTableSeeder`, `MarketplaceSettingSeeder`, `DemoSeeder`)
- Controllers: `DashboardController`, `CrudController`
- Form requests: `Store{Name}ItemRequest`, `Update{Name}ItemRequest`
- Model: `{Name}Item`
- Routes: `web.php` with CRUD routes
- React pages: `Index.tsx`, `Items/Index.tsx`, `Items/Create.tsx`, `Items/Edit.tsx`, `Items/types.ts`
- Menus: `company-menu.ts`, `superadmin-menu.ts`

**Post-creation steps (always remind the user):**
1. Register the namespace in root `composer.json` under `autoload.psr-4`:
   ```json
   "Module\\Blog\\": "packages/Blog/src/"
   ```
2. Run `composer dump-autoload`

**Edge case:** Aborts if the package directory already exists.

**`module.json` structure:**
```json
{
    "name": "Blog",
    "alias": "Blog",
    "description": "",
    "priority": 10,
    "version": 1.0,
    "monthly_price": 0,
    "yearly_price": 0,
    "package_name": "blog"
}
```

---

## `make:package {type} {name} {package} [--m]` — Add a component to a package

Creates a single PHP component inside `packages/{Package}/src/`.

**Arguments:**
- `type` (required) — Component type (see table below)
- `name` (required) — PascalCase class name
- `package` (required) — Target package name (must exist)

**Options:**
- `--m` — Also create a migration (only when `type=model`)

**Supported types:**

| Type | Output Path |
|------|-------------|
| `controller` | `packages/{Package}/src/Http/Controllers/{Name}.php` |
| `model` | `packages/{Package}/src/Models/{Name}.php` |
| `migration` | `packages/{Package}/src/Database/Migrations/{timestamp}_{name}.php` |
| `middleware` | `packages/{Package}/src/Http/Middleware/{Name}.php` |
| `event` | `packages/{Package}/src/Events/{Name}.php` |
| `listener` | `packages/{Package}/src/Listeners/{Name}.php` |
| `provider` | `packages/{Package}/src/Providers/{Name}.php` |
| `seeder` | `packages/{Package}/src/Database/Seeders/{Name}DatabaseSeeder.php` |
| `request` | `packages/{Package}/src/Http/Requests/{Name}.php` |
| `trait` | `packages/{Package}/src/Traits/{Name}.php` |
| `helper` | `packages/{Package}/src/Helpers/{Name}.php` |

**Edge case:** Aborts if the target file already exists.

---

## `make:crudly {name} [options]` — Full CRUD scaffold

Generates backend and frontend for a CRUD entity. Works for the main app or inside a package.

**Argument:**
- `name` (required) — Entity name in PascalCase (multi-word like `Priority Stage` is OK — converted via `Str::studly()`)

**Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--package` | (none) | Target package. Omit for main app. |
| `--fields` | `name:textbox,description:textarea,is_active:select` | Field definitions (`name:type:validation`) |
| `--searchable` | `name` | Comma-separated searchable fields |
| `--filterable` | `is_active` | Comma-separated filterable fields |
| `--icon` | `Tag` | Lucide icon name |
| `--relationships` | (none) | `relation:type:Model:foreign_key` format |
| `--table-relationships` | (none) | `relation.field` columns shown in the table |
| `--table-fields` | (none) | Extra table columns |
| `--view` | (none) | `modal` or `page` view mode |
| `--system-setup` | (flag) | Generate as system setup module (package only) |
| `--system-setup-icon` | `Settings` | Icon for system setup entry |

### Field format

```
name:type:validation_rules
```

- Validation rules use Laravel's pipe-delimited format, e.g. `required|string|max:255`
- `nullable` makes the field optional; `required` makes it mandatory

### Field types

| Type | `--fields` value | DB column | React component |
|------|-----------------|-----------|-----------------|
| Textbox | `name:textbox` | `string` | `<Input>` |
| Email | `email:email` | `string` | `<Input type="email">` |
| Textarea | `desc:textarea` | `longText` | `<Textarea>` |
| Select | `status:select` | `boolean` | `<Select>` |
| Radio Button | `option:radiobutton` | `string` | `<RadioGroup>` |
| Checkbox | `agree:checkbox` | `boolean` | `<Checkbox>` |
| Checkbox Group | `interests:checkboxgroup` | `json` | `<CheckboxGroup>` |
| Date Picker | `date:datepicker` | `date` | `<DatePicker>` |
| Time Picker | `time:timepicker` | `time` | `<TimePicker>` |
| Date Range | `range:daterangepicker` | `string` | `<DateRangePicker>` |
| Number | `qty:number` | `integer` | `<NumberInput>` |
| Currency | `price:currency` | `decimal(10,2)` | `<CurrencyInput>` |
| Rich Text | `body:richtext` | `longText` | `<RichTextEditor>` |
| Phone | `phone:phone` | `string(20)` | `<PhoneInputComponent>` |
| Slider | `range:slider` | `integer` | `<Slider>` |
| Switch | `active:switch` | `boolean` | `<Switch>` |
| Rating | `rate:rating` | `integer` | `<Rating>` |
| Media | `image:media` | `string` | `<MediaPicker>` |
| Multi Select | `tags:multiselect` | `json` | `<MultiSelectEnhanced>` |
| Color | `bg:color` | `string(7)` | `<ColorInput>` |
| Date Time | `start:datetime` | `timestamp` | `<DateTimeRangePicker>` |
| Date Time Range | `slot:datetimerange` | `string` | `<DateTimeRangePicker>` |
| Tags Input | `keywords:tagsinput` | `json` | `<TagsInput>` |

### Static options (for select / radiobutton / checkboxgroup / multiselect)

```
status:select:required:Active|Inactive|Pending
color:radiobutton:nullable:Red|Green|Blue
interests:checkboxgroup:nullable|array:Option A|Option B|Option C
```

### Dynamic options from DB (uses `@Model.field` syntax)

```
category_id:select:required@Category.name
assigned_to:select:nullable@User.name
```

### Special field modifiers (append to validation rules)

- `showPreview` — shows a preview for media/image fields
- `multiple` — enables multi-upload for media fields
- `searchable` — enables search/filter inside select dropdowns

**Important:** Use `multiselect` for multiple-select options (e.g. categories). `select` is only for single active/inactive-style selection.

### Relationships

Format: `relation:type:Model:foreign_key`

```
--relationships="author:belongsTo:User:user_id,category:belongsTo:Category:category_id"
```

What happens with relationships:
- Foreign key column added to the migration
- `belongsTo` relationship added to the generated model
- Reverse `hasMany` added to the parent model (e.g. `User` gets `posts()`)
- Parent model data loaded in controller for form dropdowns
- Dependent dropdown route and controller method auto-generated (order matters — first relationship is parent of second)

### What `make:crudly` creates vs modifies

**Creates (new files):**
- Migration, Model, Controller, Form Requests, Seeder
- Frontend pages: `index.tsx`, `create.tsx`, `edit.tsx`, `types.ts`
- With `--view=modal`: `View.tsx`; with `--view=page`: `Show.tsx`

**Modifies (existing files):**
- `database/seeders/PermissionRoleSeeder.php` — adds 7 permissions per entity
- `resources/js/utils/menus/company-menu.ts` — adds menu item + icon import
- `routes/web.php` — adds resource route + controller import
- `database/seeders/DatabaseSeeder.php` — adds seeder call
- Parent models with reverse `hasMany` relationships

**Package mode** modifies the package-local equivalents of these files instead.

### Permissions generated (per entity)

| Permission | Description |
|------------|-------------|
| `manage-{name-kebab}` | General manage permission |
| `manage-any-{name-kebab}` | Manage any record |
| `manage-own-{name-kebab}` | Manage own records only |
| `view-{name-kebab}` | View records |
| `create-{name-kebab}` | Create records |
| `edit-{name-kebab}` | Edit records |
| `delete-{name-kebab}` | Delete records |

### System Setup mode

With `--system-setup` and `--package`:
- Pages go under `SystemSetup/{PluralName}/` instead of the normal path
- Generates/updates `SystemSetupSidebar.tsx`
- Uses different controller and frontend stubs
- Adds a parent "System Setup" menu item instead of individual items
- `--system-setup-icon` controls the sidebar icon (default: `Settings`)

### Examples

```bash
# Simple CRUD in main app
php artisan make:crudly Category

# CRUD in a package with custom fields
php artisan make:crudly Product \
  --package=Shop \
  --fields="title:textbox:required,price:number:required|numeric|min:0,description:textarea" \
  --icon=ShoppingBag

# CRUD with relationships
php artisan make:crudly Post \
  --fields="title:textbox:required,content:textarea:required" \
  --relationships="author:belongsTo:User:user_id,category:belongsTo:Category:category_id" \
  --table-relationships="author.name,category.name" \
  --icon=FileText

# CRUD with modal view
php artisan make:crudly Task \
  --fields="title:textbox:required,due_date:datepicker,status:select:required:Pending|Done|Cancelled" \
  --view=modal

# System setup module in a package
php artisan make:crudly PaymentMethod \
  --package=Billing \
  --fields="name:textbox:required,icon:media" \
  --system-setup \
  --system-setup-icon=CreditCard
```

**Post-generation steps:**
- **Main app:** `php artisan migrate:fresh --seed`
- **Package:** `php artisan migrate`

---

## `route:find-duplicates` — Find duplicate route names

Scans all registered routes, groups them by name, and reports duplicates with file paths.

```bash
php artisan route:find-duplicates
```

No arguments or options.

---

## `app:install [--force]` — Install the application

Full installer: generates app key, runs `migrate:fresh --force`, seeds the database, seeds all packages (reads `packages/*/module.json`, calls `package:seed` for each), registers packages as `AddOn` records, and creates `storage/installed` as a marker file.

```bash
php artisan app:install
php artisan app:install --force    # reinstall even if already installed
```

---

## `package:seed {packageName?}` — Seed a package

Runs the database seeder for a specific package (or all packages).

```bash
php artisan package:seed Blog      # seed Blog package only
php artisan package:seed           # seed all packages
```

Looks for `Module\{PackageName}\Database\Seeders\{PackageName}DatabaseSeeder`. Shows an error but continues if a package's seeder doesn't exist.

---

## `extract-translations.php [PackageName]` — Extract translation keys

Standalone PHP script. Scans `.tsx`, `.jsx`, `.ts`, `.php`, and `.blade.php` files for `t("...")` and `__("...")` patterns, writes results to JSON translation files.

```bash
php extract-translations.php           # scan main app + all packages
php extract-translations.php Blog      # scan Blog package only
```

**Main app directories scanned:** `resources/js/pages`, `resources/js/components`, `resources/js/layouts`, `resources/views`, `app`

**Output:** Main app → `resources/lang/en.json`; each package → `packages/{Name}/src/Resources/lang/en.json`

Merges with existing translations (doesn't overwrite keys), sorts alphabetically, outputs pretty-printed JSON.
