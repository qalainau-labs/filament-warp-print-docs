# Warp Print

![The Warp Print designer: fields on the left, an invoice in pounds on an A4 page in the middle, the item table's settings on the right](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/designer.png)

**Let your admins design invoices, delivery notes, price tags and address labels inside Filament — and print them from any record exactly as previewed. Long item tables continue over pages with carried totals, PDFs come from the same page, the Excel forms your office already uses are filled in place, and official PDF and paper forms can be filled in on screen — by your staff or, through a link, by customers without an account.**

<a href="https://demo.warpui.com/print"><img src="https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/live-demo.png" alt="Try the live demo: demo.warpui.com/print" width="592"></a>

Print an invoice of four pages, change its template in the designer, print price tags with barcodes and fill an Excel invoice at **[demo.warpui.com/print](https://demo.warpui.com/print)**. No sign-up needed; the data is reset every hour.

Most Filament PDF plugins render a Blade view with dompdf: someone has to edit the view for every change of layout, item tables do not continue cleanly onto a second page, and fonts and Japanese text are a struggle. Warp Print turns this around. You, the developer, write a small `PrintSource` class that says **which fields** a template may use. Your admins **design the page** in the panel: drag fields, a logo and the item table onto an A4 sheet, publish, done. When someone presses Print on an invoice, they see the real pages in a preview, and the preview is what prints — the designer, the print preview and the server PDF are drawn by the same renderer.

```php
use Qalainau\FilamentWarpPrint\Actions\PrintAction;

public static function table(Table $table): Table
{
    return $table
        ->columns([/* ... */])
        ->recordActions([
            PrintAction::make(), // a "Print" button on every invoice
        ]);
}
```

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
  - [AI coding assistants (Laravel Boost)](#ai-coding-assistants-laravel-boost)
- [Getting started](#getting-started)
- [Sources and fields](#sources-and-fields)
- [Document language](#document-language)
- [Print buttons](#print-buttons)
- [The designer](#the-designer)
- [Starter templates](#starter-templates)
- [Item tables over several pages](#item-tables-over-several-pages)
- [Photos over several pages](#photos-over-several-pages)
- [Text, fields and formats](#text-fields-and-formats)
- [Calculated fields and conditional styles](#calculated-fields-and-conditional-styles)
- [Copies and watermarks](#copies-and-watermarks)
- [Text blocks](#text-blocks)
- [Postcards, envelopes and stamps](#postcards-envelopes-and-stamps)
- [Barcodes](#barcodes)
- [Charts](#charts)
- [Labels](#labels)
- [Excel forms](#excel-forms)
- [Word documents](#word-documents)
- [PDF and paper forms](#pdf-and-paper-forms)
- [Printing a list](#printing-a-list)
- [PDF files on the server](#pdf-files-on-the-server)
- [Sending by email](#sending-by-email)
- [Large batches in the background](#large-batches-in-the-background)
- [Issued documents (electronic bookkeeping)](#issued-documents-electronic-bookkeeping)
- [Print log and reprints](#print-log-and-reprints)
- [Using it from code](#using-it-from-code)
- [Moving templates between environments](#moving-templates-between-environments)
- [Approvals with Warp Flow](#approvals-with-warp-flow)
- [Multi-tenancy](#multi-tenancy)
- [Remote images and SSRF](#remote-images-and-ssrf)
- [Fonts](#fonts)
- [Configuration](#configuration)
- [Security](#security)
- [Translations](#translations)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)

## Features

- **A designer in the panel.** Text, fields, the item table, images (logo, company seal), lines, boxes, [bar, line and pie charts](#charts), [nine kinds of barcode](#barcodes) (Code 128, GS1-128, EAN-13 / JAN, UPC-A, Code 39, ITF-14, NW-7, DataMatrix, the Japan Post customer barcode) and QR codes, placed in millimetres on the real paper size. Rulers, grid, snapping to other elements with guides, alignment, undo and redo, keyboard nudging.
- **Drafts and versions.** Every change is saved as a draft right away; printing uses the published version until an admin publishes the draft. Earlier versions can be opened again. Two admins editing the same template are warned instead of overwriting each other.
- **Item tables over many pages.** The heading row repeats on every page, the total can be carried to the next page (*Carried forward / Brought forward*), each page can have a subtotal, empty ruled rows can fill the form, and the last page can be shorter so the table ends above the totals. Elements can appear on the first page only, on the last page, on every page but the first, and so on, or move up under the last row.
- **What you see is what prints.** The print window shows the actual pages. Printing uses the browser's own print dialog (which also saves PDFs), so nothing has to be installed on the server.
- **Server PDFs when you need them** — to attach to an email or to save many at once — through Chrome ([Browsershot](https://github.com/spatie/browsershot)) or [Gotenberg](https://gotenberg.dev), drawing exactly the same page.
- **Your Excel forms.** Upload the `.xlsx` invoice your office already has and write `{{customer.name}}` in its cells. The item row is repeated for every item — first into the empty ruled rows below it, then by adding rows — and merged cells, formulas such as `SUM()`, images, conditional formats and the print area move along. Formatting is never touched.
- **Your PDF and paper forms.** Upload a PDF — or photos and scans of a paper form — and place fields on it: boxes on scans are found for you, fillable PDF fields are read in. Staff fill it in on the page with the record's values already written, sign it and submit; what they wrote can be saved back to the record. Values are written on top of the original page, so it stays the official form. [More](#pdf-and-paper-forms)
- **Forms filled in by people without an account.** *Send to fill in* emails a link; customers, suppliers or applicants fill in and sign in their browser, on a phone too. The link expires and closes once the form is submitted, and the result waits for you to check it.
- **Labels.** Address labels, price tags and name badges on any sheet (A4 24, A4 21, Letter 30, … or your own sizes). Skip the labels already used on a sheet and print several copies of each.
- **Print the list itself.** `PrintTableAction` prints a Filament table as it is filtered, searched and sorted, with the heading on every page.
- **Made for real forms.** Japanese era dates (令和8年10月22日), currencies in their own decimals, tax per rate, vertical text, company seals, "shrink to fit" for long names, and a *Reprint* / 再発行 mark that appears from the second print of the same record.
- **Start from a ready-made template.** Invoice, quote, delivery note and receipt (in Japanese and English), address labels, price tags, a postcard and a Naga 3 envelope. Their fields are matched to your data when they are created.
- **Send by email.** `EmailPrintAction` attaches the PDF (or the Excel file) and sends it to the record's own recipients, with a subject and message that the template fills in. `EmailPrintBulkAction` sends a month of invoices, each to its own customer, in the background.
- **Copies and watermarks.** Print an *original* and a *copy* (控え) of every record in one go, and put *COPY*, *DRAFT* or *PAID* across the page — always, from the second copy, on reprints, on test prints or when a field matches.
- **Calculated fields and conditional styles.** Admins write small formulas (`total - paid`, `sum(items.amount)`, `addDays(issued_on, 30)`) without touching your code, and make a value red when it is negative or bold when it is over a limit.
- **Text blocks.** Terms and conditions with headings, bold text and lists, continuing onto extra pages when they are long.
- **Postcards and envelopes.** Postal codes in the printed boxes of a Japanese postcard, vertical addresses with kanji numbers, and red name and date stamps for approval boxes.
- **Large batches in the background.** More records than fit in the print window are made by a queued job — one PDF, one PDF per record or Excel files in a ZIP — and the user gets a notification with the download.
- **Issued documents kept unchanged.** Every PDF the server makes for an invoice can be stored with its hash, searchable by date, amount and counterparty, as Japan's electronic bookkeeping law (電子帳簿保存法) asks; printing the same invoice again gives exactly the same PDF.
- **Templates move between environments.** Export them as JSON and import them in another panel, tenant or server (`php artisan warp-print:import` in your deploy).
- **Approvals from [Warp Flow](https://warpui.com/flow).** Show who approved a purchase order and when, as red stamps, and print only approved records.
- **Safe for admins to edit.** Templates can only show the fields you list in a `PrintSource`. They cannot run code, read other columns or follow other relations.

## Requirements

- PHP 8.3+ with the `zip` and `dom` extensions
- Laravel 12 or 13
- Filament 5.8.3 or later
- SQLite, MySQL 8.0+, MariaDB 10.11+ or PostgreSQL 14+
- For PDF files made on the server (optional): Node and Chrome for Browsershot, or a Gotenberg container
- For filled PDF forms made on the server (optional): Node.js 18 or later (no Chrome)

## Installation

> **Warp Print is a commercial plugin.** A license is required. After purchase you receive credentials for the private Composer repository.

Add the repository and authenticate:

```bash
composer config repositories.qalainau composer https://qalainau.privato.pub/composer
composer config --auth http-basic.qalainau.privato.pub "your-email" "your-license-key"
```

This writes your license key to `auth.json` in the project. Add `auth.json` to `.gitignore`, and on CI or servers pass the credentials in the `COMPOSER_AUTH` environment variable instead.

Install the package, run the migration and publish Filament's assets:

```bash
composer require qalainau/filament-warp-print
php artisan migrate
php artisan filament:assets
```

The migration creates these tables: `warp_print_templates`, `warp_print_template_versions` (published versions), `warp_print_logs` (who printed which record, when, and how), `warp_print_exports` (background jobs), `warp_print_documents` (issued documents, when kept) and `warp_print_entries` (filled PDF and paper forms). Your tables are not changed. To edit the migration, publish it with `php artisan vendor:publish --tag=filament-warp-print-migrations`. Publish it before you run `php artisan migrate` for the first time: Warp Print then runs your copy instead of its own. If the tables already exist, change them with a migration of your own instead.

> Run `php artisan filament:assets` again after every update of the package.

### AI coding assistants (Laravel Boost)

Warp Print ships instructions for AI coding assistants such as Claude Code, Cursor and Codex. If your app uses
[Laravel Boost](https://laravel.com/docs/boost), the assistant learns how to use Warp Print from them: how to
register sources and print actions, design invoices and labels, fill Excel and Word forms, save PDFs and send email,
and handle permissions and document language.

- **Guidelines** (`resources/boost/guidelines/core.md`): a short summary that Boost adds to `CLAUDE.md`,
  `AGENTS.md` and the other agent files. It also points the assistant to this README in `vendor/`, so it reads the
  documentation of the version you have installed instead of guessing.
- **Skill** `warp-print-development` (`resources/boost/skills/`): the steps for sources, print buttons, item tables,
  labels, Office forms, server PDFs, email, permissions and document language. The assistant loads it when it works
  on printing.

Boost does not add guidelines of other vendors' packages on its own. When `php artisan boost:install` asks
"Which third-party AI guidelines/skills would you like to install?", select `qalainau/filament-warp-print`. To do
it without the question, add the package to the existing `packages` array in `boost.json` and run the command again:

```json
{
    "packages": ["qalainau/filament-warp-print"]
}
```

```bash
php artisan boost:install --guidelines --skills -n
```

## Getting started

**1. Describe what can be printed.** A source names a model and the fields a template may show:

```php
namespace App\Print;

use App\Models\Invoice;
use App\Models\InvoiceItem;
use Qalainau\FilamentWarpPrint\Sources\Field;
use Qalainau\FilamentWarpPrint\Sources\PrintSource;

class InvoiceSource extends PrintSource
{
    protected static string $model = Invoice::class;

    public function fields(): array
    {
        return [
            Field::text('number')->label('Invoice no.'),
            Field::date('issued_on'),
            Field::date('due_on'),
            Field::text('customer.name')->label('Customer'),
            Field::text('customer.address')->label('Address'),
            Field::image('company.logo')->state(fn () => 'logos/company.png'), // a path on the "public" disk
            Field::money('total', 'JPY')->state(fn (Invoice $invoice) => $invoice->total()),
            Field::table('items', [
                Field::text('name')->label('Item'),
                Field::number('quantity'),
                Field::money('unit_price', 'JPY'),
                Field::money('amount', 'JPY')->state(fn (InvoiceItem $item) => $item->quantity * $item->unit_price),
            ])->label('Items'),
        ];
    }

    public function title(\Illuminate\Database\Eloquent\Model $record): string
    {
        return 'Invoice '.$record->number; // used for file names
    }
}
```

**2. Register the plugin** with your sources. It adds the **Print templates** page to the panel:

```php
use App\Print\InvoiceSource;
use Qalainau\FilamentWarpPrint\WarpPrintPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(WarpPrintPlugin::make()->sources([InvoiceSource::class]));
}
```

**3. Add a print button** to the invoices (see [Print buttons](#print-buttons)):

```php
->recordActions([PrintAction::make()])
```

**4. Design and publish.** Open *Print templates* → *New*, give it a name, choose *Invoice* as the data and *Designed layout* as the kind, and pick the paper. The designer opens. Click *Items* in the field list to add the item table, add the customer's name and the total, then press **Publish**. The Print button on an invoice now offers this template.

## Sources and fields

A source is a plain class extending `PrintSource`. Everything a template can show is in `fields()`.

| Field | Shows | Default format |
| --- | --- | --- |
| `Field::text('path')` | Text. Enums show their label (`HasLabel`) | as is |
| `Field::number('path', decimals: 2)` | A number | thousands separator |
| `Field::money('path', 'EUR')` | An amount. Without a currency it uses [the record's currency](#sources-and-fields) (`PrintSource::currency()`) or `config('filament-warp-print.currency')` | currency symbol and its own decimals (¥1,234 / €1,234.50) |
| `Field::date('path')` / `Field::datetime('path')` | A date or a date and time | `YYYY/MM/DD` (`HH:mm`) |
| `Field::boolean('path')` | Yes / no | ✓ or empty |
| `Field::image('path')` | A logo, a seal or a product photo: a path on `->disk('public')` (default `config('filament-warp-print.image_disk')`), a `data:image/...` URI, or an http(s) URL when [remote images](#remote-images-and-ssrf) are allowed. `->directory('teams/1')` keeps paths inside one folder | — |
| `Field::table('path', [...])` | Item rows: a relation (`hasMany`, `belongsToMany`, …) or anything iterable from `state()`. The columns are fields read from each item | — |

**Paths** follow relations with dots (`customer.name`, and in a table `product.code`). The relations a source uses are eager loaded for you, also when many records are printed at once.

**Every field can be computed** with `->state(fn (Model $record) => ...)`; in a table column the closure receives the item. Use it for totals, tax per rate, your own company's details or anything that is not a column.

**Other options:** `->label('…')` is the name in the designer, `->group('Customer')` groups fields in the designer's list, `->sample('INV-0001')` is shown in the designer when there is no record yet.

**Dates and times** are passed to the page as the wall-clock time of the panel's timezone (`FilamentTimezone`, which defaults to `app.timezone`), so a browser in another timezone never prints the day before.

**Money in each record's currency.** When your invoices are in different currencies, return the record's currency from `currency()`. Every `Field::money()` without a currency of its own — also in item tables, totals and page subtotals — then uses it, in the designer's sample, the print window, server PDFs, Excel forms and email text:

```php
public function currency(Model $record): ?string
{
    return $record->currency; // 'GBP' → £562.00, 'JPY' → ¥3,431,094
}
```

- The currency is an ISO 4217 code (`GBP`, `usd` is read as `USD`). `null`, an empty string or anything that is not three letters falls back to `config('filament-warp-print.currency')`.
- A field with its own currency keeps it: `Field::money('fee', 'JPY')` is always in yen.
- A format chosen in the designer or in text wins over both: `{{total|money:EUR}}` is always in euros.
- Calculated fields of the `money` type made in the designer follow the record's currency too.
- Excel cells that hold only one field get the number itself, so the cell's own number format in your .xlsx decides how it looks.

A source can also limit what is printed and by whom:

```php
public function query(): Builder
{
    return parent::query()->where('status', '!=', 'cancelled'); // records outside this query are not printed
}

public function canPrint(Model $record, ?Authenticatable $user): bool
{
    return $user?->can('view', $record) ?? false; // the default checks the model policy's "view", if there is a policy
}

public function sampleRecord(): ?Model
{
    return $this->scopedQuery()->has('items', '>=', 3)->latest()->first(); // the designer's default sample
}
```

`query()` is where you add your own conditions. When Warp Print reads records it always uses `scopedQuery()`: your `query()` plus the current tenant (see [Multi-tenancy](#multi-tenancy)). Use `scopedQuery()` too when you override `sampleRecord()`, so the designer never shows another tenant's record.

Every template also has the **print** fields: *Printed on* (`print.date`), *Printed at* (`print.datetime`), *Printed by* (`print.user`), *Reprint* (`print.reprint`, true from the second print of the record with this template) and *Times printed* (`print.count`).

## Document language

By default a document is made in the language of the person printing it (`app()->getLocale()`). When documents go to customers, that is often wrong: someone using the panel in English still sends a Japanese invoice to a Japanese customer. Choose the document's language separately, per record:

```php
// In a source: only this kind of document
public function documentLocale(Model $record): ?string
{
    return $record->customer?->locale; // 'ja', 'en', 'de', … or null for the screen language
}
```

```php
// In the panel: every document of every source
WarpPrintPlugin::make()
    ->sources([InvoiceSource::class, QuoteSource::class])
    ->documentLocale(fn (Model $record, PrintSource $source): ?string => $record->customer?->locale)
```

**Which one wins:** the source's `documentLocale()` first. When it returns `null`, the plugin's `documentLocale()` (a language code or a closure; the closure receives `$record` and `$source`). When that is `null` or not set too, the screen language.

**What uses it:** while Warp Print builds one record's document, it switches the app locale, Carbon's locale and `Number`'s locale to the document language, and puts all three back afterwards — also when something throws. So in that language you get:

- what your `state()` closures return (`__('Total')`, `$date->translatedFormat('F')`, `Number::currency(...)`),
- numbers, money and dates on the page (`1,234.50` or `1.234,50`, `¥700` or `￥700`, month and weekday names),
- the email subject and message (also the default message), the mail's own view and the attachment's file name,
- the issued document kept in the archive,
- Excel forms and PDF / paper forms.

It is the same in the print window's preview, the designer's preview with a chosen record, PDF downloads, `WarpPrint::pdf()` / `html()` / `xlsx()` / `send()`, and background exports and bulk emails.

**Several records at once:** each record gets its own language. A bulk print for one Japanese and one German customer makes one PDF with a Japanese page and a German page; a bulk email sends each customer their own language.

**The screen stays in the screen language.** The print window, the designer, buttons, notifications ("Your PDF is ready") and the name of a ZIP file follow the person using the panel. A background export remembers that person's language for its notification and makes each document in its own language.

Notes:

- Use a language your app has translations for (`lang/ja/...`); Warp Print's own words (the default email message, "continued" on a form) are included in 10 languages.
- Month and weekday names, and AM / PM, written by a date format (`{{issued_on|date:MMMM}}`) follow the document language everywhere: on the page, in email text, and in Excel and Word cells. The words and the order in a pattern are yours; to keep both in one language, see *A template's own language* below.
- The labels of fields in the designer (`->label()`) are not printed, so they stay in the screen language.
- The words you type into a template stay as they are. Numbers and money follow the document language, so pick patterns that suit it. Keep one template per language (an English *Invoice* and a Japanese 請求書) when the words differ too, and give each one its own language (next section).

### A template's own language

A date pattern such as `MMM D, YYYY` has an English order, so for a Japanese document it used to print `Visited 11月 18, 2026`: English words and order with a Japanese month. A report should be in one language. In the template's settings choose **Document language** (the `locale` column of `warp_print_templates`):

- **Match the screen or the customer** (the default, `null`): month and weekday names (`MMM`, `MMMM`, `ddd`, `dddd`) and AM / PM follow the document language above, as before. Use it for a template whose words you let the customer's language decide, such as `M月D日(ddd)`.
- **A language** (English, 日本語, Deutsch, Français, Español, Italiano, Português (Brasil), 한국어, 简体中文, 繁體中文): those names are always in that language, whatever the screen or the customer uses. An English template set to English prints `Visited Nov 18, 2026` for every document.

Only the names in dates use it. Numbers and money keep following the document language. It applies to the page, PDFs, email subjects and bodies, and Excel, Word and form templates, and it is exported and imported with the template. A template made from a starter gets the starter's language (an English starter is English, a Japanese one is Japanese).

Date patterns that are only numbers (`YYYY/MM/DD`, `YYYY年M月D日`) print the same in every language.

## Print buttons

### One record

```php
use Qalainau\FilamentWarpPrint\Actions\PrintAction;

// In a table row, or in the header of a view or edit page
PrintAction::make()

// Only one template, or a choice of some, in this order
PrintAction::make()->template('invoice')
PrintAction::make()->templates(['invoice', 'delivery-note'])

// When two sources share a model, say which one
PrintAction::make()->source(InvoiceSource::class)
```

The source is found from the record's model among the registered sources. The print window lists the **published** templates of that source (or the ones you name), shows the preview with the page count, and has **Print** (the browser's print dialog, which can also save a PDF), **Download PDF** (when a [PDF driver](#pdf-files-on-the-server) is set) and, for Excel templates, **Download Excel**.

**Which template is selected first:** the templates are listed by their order (the `sort` column), and the window starts on the first one that can be made in the document's language — the record's [document language](#document-language) when one record is printed, the screen language for several. A template without a *Document language* follows the document's language, so it always counts; a template set to another language is skipped. So on an English screen an English customer's invoice opens with your English invoice even when the Japanese one is listed first. When none fits, the first template is selected. Templates you name with `->templates([...])` keep your order and the first one is selected.

It is an ordinary Filament action: `->label()`, `->icon()`, `->visible()`, `->authorize()` and the rest work as usual.

### Several records

```php
use Qalainau\FilamentWarpPrint\Actions\PrintBulkAction;

->toolbarActions([
    PrintBulkAction::make(),                                       // invoices: one document, every invoice from a new page
    PrintBulkAction::make()->template('address-labels')->label('Print labels'),
])
```

The records are printed in the order they were selected, each starting on a new page (labels fill the sheet). Excel templates download one file per record in a ZIP. Up to `batch_limit` records (default 200) are shown and printed in the window; with more, the window offers to [make them in the background](#large-batches-in-the-background) instead.

## The designer

Open **Print templates** in the panel. *New* asks for:

- **Name** and **key** — the key is what `->template('invoice')` refers to.
- **Data** — one of the registered sources. It decides the fields.
- **Kind** — *Designed layout* (invoices, delivery notes, quotes, designed in the panel), *Labels* (many records on a sheet) or *Excel form*.
- **Paper** and orientation (documents) or the label sheet (labels). Both can be changed later.

![The print window: page 1 ends with the total carried forward, page 2 starts with it](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/pages.png)

**Adding.** Click an element in *Add* to put it in the middle of the page, or drag it to a place. Click a field to add it as a field element (an image field becomes an image; *Items* becomes the item table), or drag it.

**Moving and sizing.** Drag elements; drag the handles to resize. Elements snap to the grid and to the edges and centres of the paper, the margins and other elements (the pink guides show it); hold **Alt** to move freely. Arrow keys move by 0.5 mm, with **Shift** by 5 mm. Select several with **Shift**-click or by dragging a box on an empty part of the page, then align or space them evenly.

**Keys.** **Ctrl/⌘ Z** undo, **Ctrl/⌘ Shift Z** or **Ctrl/⌘ Y** redo, **Ctrl/⌘ D** duplicate, **Ctrl/⌘ C / V** copy and paste, **Delete** remove, **Ctrl/⌘ A** select all, **Esc** deselect, **Ctrl/⌘ S** save now. Double-click a text to edit it.

**Settings.** With nothing selected, the panel on the right shows the paper, the margins (guide lines only), the template's default font and size, the grid and the published versions. With one element selected it shows its position and size in millimetres and everything about it: text and format, font, size, bold, alignment, colours, padding, borders (per side), rounded corners, vertical text, *shrink to fit*, the pages it is shown on and a condition.

**Showing.** *Show only when* hides an element unless a field has a value, is empty, is yes / no or equals a value. The sample *Reprint* mark is a text with *Show only when: Reprint — is yes*.

**Sample.** The designer shows real values from the newest record (choose another in *Sample*), or made-up values when there are none. **Preview** opens exactly the pages that would print, with the draft.

**Saving and publishing.** Changes are saved as a draft a moment after you make them ("Draft saved"). Printing keeps using the published version, so half-finished changes never reach your users. **Publish** makes the draft the version that prints; you can add a note ("New bank account"). *Published versions* lists them; *Open in draft* loads an earlier one into the draft (publish it to use it again).

If someone else saved the template after you opened it, the designer says so and lets you load their version or keep yours, instead of silently overwriting.

**Images.** Uploaded images (logos, seals) are stored in the template, scaled down to 1,600 px. Images that differ per record come from image fields. When [remote images](#remote-images-and-ssrf) are allowed, the inspector also has an *image URL* box: the URL is checked when you press **Use**, and a template with an image URL that is not allowed is not saved.

## Starter templates

Admins rarely want to start from a blank page. **From a starter** on the Print templates page offers ready-made templates:

| Starter | What is on it |
| --- | --- |
| 請求書 (invoice, Japanese) | A4. The qualified invoice registration number, tax per rate (10 % / 8 %), items that carry their total over pages, the company seal |
| 見積書 (quote, Japanese) | A4. The quote total, valid until, delivery and payment terms |
| 納品書 (delivery note, Japanese) | A4. No prices; the slip number as a barcode and a box for the receiver's stamp |
| 領収書 (receipt, Japanese) | A5 landscape. 但し書き, the amount before tax and the tax, a revenue stamp box, and an original plus a copy marked 控 |
| Invoice, Quote, Receipt (English) | A4 / A5. Logo, amount due, totals right under the items; terms on the quote continue onto a new page when they are long; a *PAID* watermark on the receipt |
| 発注書, Purchase order | A4. The supplier, the order total, deliver by and deliver to, items that continue on more pages |
| 取引明細書, Statement of account | A4. Opening balance, charges, payments and closing balance; transactions by date with the running balance, charges and payments carried over pages |
| 給与明細書, Payslip | A4. Employee and pay period, days worked and overtime, earnings and deductions as two tables side by side, gross pay, total deductions and net pay |
| 送り状, Shipping label | 100 × 150 mm (4 × 6 in). Ship to and from, contents, ship date, deliver by, pieces, and the tracking number as a Code 128 barcode |
| Address labels, Price tags | A4 24 and A4 21 sheets; price with an EAN-13 / JAN barcode |
| はがき (postcard), 長形 3 号 (Naga 3 envelope) | Postal code in the printed boxes, vertical address with kanji numbers, the name with 様 / 御中, the sender |

Choose a starter and the data. The starter's fields (`number`, `issued_on`, `customer.name`, `items.amount`, …) are **matched to your source's fields**: first by the same path, then by common names (`invoice_number`, `client.name`, `lines`, `qty`, …) and only with a fitting type (a date for `issued_on`, money for `total`). The template is created as a **draft** and opens in the designer. If some fields were not found, a notice lists them; the elements that use them are marked *Fields that are not in the data*, so the admin can pick the right ones before publishing.

When your field names differ, tell Warp Print how they match:

```php
class InvoiceSource extends PrintSource
{
    public function starterFields(): array
    {
        return [
            'number' => 'code',
            'issued_on' => 'invoice_date',
            'customer.name' => 'client.company_name',
            'items' => 'lines',              // the item table
            'items.amount' => 'line_total',  // a column of it
        ];
    }
}
```

From code: `WarpPrint::fromStarter('invoice-ja', InvoiceSource::class, name: '請求書', key: 'invoice')` returns the template and the fields that were not found.

The starters and the fields they look for (a field with another name is found through the common names, or through `starterFields()`):

| Key | Fields | Tables and their columns |
| --- | --- | --- |
| `invoice-ja`, `invoice-en`, `quote-ja`, `quote-en`, `receipt-ja`, `receipt-en`, `delivery-note-ja` | `number`, `subject`, `issued_on`, `due_on` / `valid_until`, `note`, `subtotal`, `tax`, `total` (Japanese: `base_10`, `tax_10`, `base_8`, `tax_8`), `customer.*`, `company.*` | `items`: `name`, `quantity`, `unit`, `unit_price`, `amount` (delivery note: `product.code`) |
| `purchase-order-ja`, `purchase-order-en` | As an invoice, with `delivery_on` and `delivery_place`. `customer.*` is the supplier (`supplier.name` and `vendor.name` are found too) | `items`, as above |
| `statement-ja`, `statement-en` | `number`, `issued_on`, `period_from`, `period_to`, `opening_balance`, `charges`, `payments`, `closing_balance`, `note`, `customer.*`, `company.*` | `transactions`: `date`, `reference`, `name`, `charge`, `payment`, `balance` |
| `payslip-ja`, `payslip-en` | `period`, `paid_on`, `employee.name`, `employee.code`, `employee.department`, `work_days`, `absent_days`, `paid_leave_days`, `overtime_hours`, `gross`, `deductions_total`, `net`, `note`, `company.name`, `company.logo` | `earnings` and `deductions`: `name`, `amount` |
| `shipping-label-ja`, `shipping-label-en` | `number`, `tracking_number`, `subject`, `issued_on`, `delivery_on`, `package_count`, `note`, `customer.*`, `company.*` | — |
| `address-labels`, `postcard-ja`, `envelope-naga3-ja` | `name`, `contact`, `postal_code`, `address`, `address2`, `company.*` | — |
| `price-tags` | `name`, `code`, `price`, `jan` | — |

- A payslip's two tables stay on one page (ten rows each); a longer list of earnings or deductions needs a taller table in the designer. The amounts are yours: the starter adds nothing up, so `gross`, `deductions_total` and `net` come from your source.
- A statement prints the balances your source gives it. The table carries the *Charges* and *Payments* columns over pages; the running `balance` of each row is a field of the row.
- The shipping label is a layout for your own label printer or plain paper. It is not a carrier's official waybill: carriers that require their own form or barcode format need that form (see [PDF and paper forms](#pdf-and-paper-forms)).

## Item tables over several pages

The first table on a page (with *Rows that do not fit continue on the next page*) is the **flowing table**. When its rows do not fit:

- the rows continue on the next page, under the heading row again;
- with **Carry the total to the next page**, the columns marked *Add up* are totalled at the bottom of the page (*Carried forward*) and at the top of the next one (*Brought forward*);
- with **Subtotal on every page**, every page ends with the total of its own rows;
- with **Fill with empty rows**, the rest of the table is ruled with empty rows, as on a printed form;
- **Rows per page** stops a page at a fixed number of rows.

Use **Showing** at the top of the designer to place things per page:

| Showing | What you set there |
| --- | --- |
| **First page** | Everything, and the table's place on the first page |
| **Next pages** | Where the table is on page 2 onwards (usually higher and longer, under a small continuation header) |
| **Last page** | How tall the table may be on the last page, so it ends above the totals |

If the remaining rows fit on a page but not above the totals, Warp Print adds one more page for the last rows and the totals. If they all fit on the first page, the first page is the last page.

Each element has **Show on**: every page, the first page only, the last page only, every page but the first (for a continuation header such as "Invoice 1234 (continued)"), or every page but the last. **Move up under the table** puts an element (a totals box, a note) right under the last row on the last page instead of at a fixed place.

### Heading rows for groups of items

A monthly invoice usually lists its deliveries one after another, each under a heading with the delivery number and subject. In the table's **Rows** settings, choose a field of the items in **Heading row when this changes**. A row across the whole table is then added before the first item and wherever that field's value changes from the item above it.

- **Heading text** is the text of that row, with `{{fields}}` of the first item of the group: `Delivery {{sale_number}} — {{sale_subject}}`. Left empty, it shows the field itself. The fields are the item's own, as in a column's text (`{{sale_subject}}`, not `{{items.sale_subject}}`).
- The heading is bold on a light background; **Heading background** changes the colour.
- Items are not sorted: give the table its rows in group order (`->orderBy('sale_number')` on the relation, or sort in `state()`). A value that comes back later starts a new heading.
- A heading is never left alone at the bottom of a page: it moves to the next page with its first item. The heading row of the table is repeated on every page as usual; a group that continues on the next page carries on without a new heading.
- Headings are not items: row numbers (`#`), page subtotals and carried totals count the items only.
- They work in flowing tables, other tables on the page and tables on labels. Excel forms are filled row by row and have no heading rows.

```php
Field::table('items', [
    Field::text('sale_number'),
    Field::text('sale_subject'),
    Field::text('name'),
    Field::money('amount'),
])->state(fn (Invoice $invoice) => $invoice->items->sortBy(['sale_number', 'position'])->values()),
```

### A subtotal under each group

A statement for a month of deliveries needs the amount of each delivery, not only the total of the page. Once a heading field is chosen, tick **Subtotal under each group** below it: a row with the totals of the group is added under the group's last item.

- It totals the columns marked **Add up** in the column settings — the same columns the page subtotal and the carried total use. With no such column the designer shows a warning and the row has nothing to show.
- **Subtotal text** is the text of the row, placed in the columns before the first total, with `{{fields}}` of the group's first item: `Subtotal {{sale_number}}`. Left empty, the row shows the amounts only.
- The row is bold, like the page subtotal and the carried total (`totalStyle` of the table in a layout kept in code changes all three).
- A subtotal is never sent to the next page alone: the item above it moves with it. A group that continues on the next page gets its subtotal where it ends, for the whole group.
- Subtotals are not items: row numbers, page subtotals and carried totals leave them out, so a carried total stays the running total of the items.
- Rounding: amounts are added as numbers and then formatted like the column, so a subtotal of 0.1 + 0.2 prints 0.3.

In a layout kept in code:

```php
'group' => ['field' => 'sale_number', 'content' => 'Delivery {{sale_number}}', 'subtotal' => ['enabled' => true, 'label' => 'Subtotal {{sale_number}}']],
```

**Rows of your own.** A table also draws heading and total rows that your source puts among the rows, when one layout of groups is not enough (a second level of subtotals, a row for a discount). A row with `__row` set to `heading` is drawn across the table with the text of `__label`; a row with `__row` set to `total` shows `__label` and the values it has for the columns, as they are. Their text is printed as it is — `{{fields}}` in it are not read — and they are left out of row numbers and totals.

```php
Field::table('items', [Field::text('name'), Field::text('amount')])->state(fn (Statement $statement) => [
    ['__row' => 'heading', '__label' => 'October'],
    ['name' => 'Delivery 1001', 'amount' => '¥12,000'],
    ['__row' => 'total', '__label' => 'October total', 'amount' => '¥12,000'],
]),
```

## Photos over several pages

A service report, an inspection or a damage claim has as many photos as were taken: two on one visit, fourteen on the next. The **Photos** element lays out the photos of a table's rows in a grid, each with a caption, and the ones that do not fit continue on added pages.

Give the source a table with an image column (and a text column for the caption):

```php
Field::table('photos', [
    Field::image('image')->disk('public'),   // a path on the disk, a data URI or an allowed URL, as for Field::image()
    Field::text('caption'),
])->state(fn (WorkOrder $order) => $order->photos()->orderBy('position')->get()),
```

In the designer, add **Photos** and choose:

| Setting | What it does |
| --- | --- |
| **Photos from** | The table whose rows have the photos (only tables with an image column are offered) |
| **Photo column** | The image column |
| **Caption under each photo** | Text with the row's fields: `{{caption}}`, `{{#}}. {{caption}}` (`#` numbers the photos 1, 2, 3 … across all pages). Text between `{{#caption}}` and `{{/caption}}` is shown only when that column is not empty: `{{#}}{{#caption}}. {{caption}}{{/caption}}` prints `3` for a photo without a caption instead of `3.`. Up to two lines; longer text ends with … |
| **Photos across** | 1 to 6 photos side by side |
| **Photo height**, **Caption height**, **Space between photos** | In mm. Every photo has the same height, so the grid is the same in the preview, the print window and server PDFs. Caption height 0 prints photos only |
| **Fit** | *Show all of it* (default, nothing is cut) or *Fill the box (crop)* |
| **Top / Height on added pages** | Where the grid is on the added pages (as for a [text block](#text-blocks)); empty means from the top margin to the bottom margin |

How the photos are placed:

- The photos fill the element's box from left to right, row by row. The rest go onto **added pages after the item table's pages** (and after a text block's continued pages). With *Show on* → *Last page*, the photos start under the item table where it ends.
- If not even one row fits in the box on that page, the photos start on the first added page.
- Rows without a photo — empty, not an image, or an image that could not be read — are left out and the next photo takes their place. A caption without a photo is not printed.
- With **no photos at all**, the element prints nothing and no page is added. To hide a heading over the photos as well, give the heading **Show only when** → the photo table → **has a value** (a table has a value when it has at least one row; in code: `'visibleIf' => ['field' => 'photos', 'operator' => 'filled']`).
- **Page numbers** count the added pages: `{{page}} / {{pages}}` prints 2 / 2 on the photo page. When several records are printed together, each record gets its own added pages and its own page numbers.
- **Show on** with added photo pages: *Every page* and *Every page but the first* elements (a header, the page number) are on the photo pages too. **The last page is the last photo page**: *Last page* elements (a signature box) move there, and *Every page but the last* elements are on the item table's last page as well. Elements set to **Move up under the table** (the totals) stay under the table on the table's last page.
- Captions are text: values are written as text, never as HTML.

Limits: at most **200 photos** per record and **50 added pages** per photo element. Photos past either limit are not printed (nothing tells you on the page), so keep the photos per report below that or make the photos smaller. One photo element per template continues onto added pages; a second one shows the photos that fit in its box only.

In a layout kept in code (`WarpPrint::syncTemplate()`):

```php
['type' => 'photos', 'x' => 15, 'y' => 150, 'w' => 180, 'h' => 120, 'pages' => 'last',
    'field' => 'photos', 'image' => 'image', 'caption' => '{{#}}. {{caption}}',
    'columns' => 3, 'photoH' => 45, 'captionH' => 9, 'gap' => 4, 'fit' => 'contain', 'nextY' => 20, 'nextH' => 250],
```

A `nextH` too small for one row of photo and caption is dropped when the layout is saved.

## Text, fields and formats

A **text** element is fixed text with fields in double braces:

```text
Invoice {{number}} — {{customer.name}}
Page {{page}} of {{pages}}
Due {{due_on|date:MMM D, YYYY}} · Total {{total|money:EUR}}
```

`{{page}}` and `{{pages}}` count the pages of each record. Choose fields from *Insert a field* instead of typing them.

A **field** element shows one value, with a **Format** chosen in the panel (and text before and after it).

| Format | Written as | Examples |
| --- | --- | --- |
| Number | `number`, `number:2` | 1,234 · 1,234.50 |
| Money | `money`, `money:USD` | ¥1,234 · $1,234.00. *Currency symbol* off: 1,234 |
| Date | `date:pattern` | see below |
| Yes / no | `boolean` | ✓ |
| Capitals | `upper` | ACME |

Date patterns: `YYYY` `YY` `MMMM` `MMM` `MM` `M` `DD` `D` `dddd` `ddd` `HH` `H` `hh` `h` `mm` `ss` `A`, Japanese eras `GGGG` (令和) `G` (R) `E` (8, or 元 for the first year) `EE` (08). Text in `[brackets]` is kept as is. For example `YYYY年M月D日` → 2026年10月22日, `GGGGE年M月D日` → 令和8年10月22日, `GE.M.D` → R8.10.22, `ddd, MMM D` → Thu, Oct 22.

## Calculated fields and conditional styles

### Calculated fields

With nothing selected, **Calculated fields** in the designer's settings lets an admin add a value that is worked out from other fields — a balance, a due date, a line total — without asking a developer. Each has a name, a key, a type (money, number, text, date or yes / no) and a **formula**. The result for the sample record is shown under the formula as you type; a formula that cannot be read says so.

The new value can be placed like any other field (it is listed under *Calculated*) and used in text as `{{calc.key}}`.

```text
total - paid                          the balance
round(subtotal * 0.1)                 10 % tax, rounded half up (also floor(), ceil(), and round(x, 2))
sum(items.amount)  count(items)       a column of the items added up, the number of items (also avg, min, max)
if(total >= 100000, "要承認", "")      a choice (> >= < <= = != and, or, not)
addDays(issued_on, 30)                30 days later (also addMonths, endOfMonth, days(from, to))
concat(customer.name, " 様")           text joined together (also + with text)
```

*For each row of Items* makes a value per item row instead (for example `quantity * unit_price`); it is placed as a column of the item table and can be added up by a document value: `sum(items.calc.gross)`.

Formulas are read by Warp Print's own small parser: they can only use the template's fields, numbers, text in quotes and the functions above. They cannot call code, and a broken formula leaves the value empty instead of stopping the print. Calculated fields are worked out when the page is drawn, so they are not available in [Excel forms](#excel-forms) (use Excel formulas there) or in email subjects.

### Conditional styles

**Style by condition** (in *Behaviour*, for text, fields, boxes and text blocks, and in each column of the item table) changes the colour, background or boldness when a field matches: *is less than 0* → red, *is at least 100000* → bold, *equals Overdue* → red background. Rules are checked in order, and every rule that matches adds its style. In a table column, the condition reads the row's own values.

*Show only when* also has *is greater than*, *is at least*, *is less than*, *is at most* and *contains*. Numbers are compared as numbers and dates (`2026-10-31`) by date.

## Copies and watermarks

**Copies** (with nothing selected in the designer) prints every record more than once, one copy after the other — for example *Original* and *Copy* (原本・控), or three copies for the customer, accounting and the warehouse (up to five). The copy's name is the field `copy.label` and its number `copy.number`: place `{{copy.label}}` in a corner, or show *（控）* only when *Copy name — equals 控*.

A **watermark** is a large, faint text across the middle of every page, on top of everything else. Set its text (fields work: `{{copy.label}}`), size, angle, colour and opacity, and when it shows:

| Show | For example |
| --- | --- |
| Always | *SAMPLE*, *PAID* on a receipt |
| When printed again (reprint) | *COPY* / 再発行 from the second print of the record |
| On copies (from the second copy) | *控* on the copy of a receipt |
| On test prints of the draft | *DRAFT* in the designer's preview, never on real prints |
| When a field matches | *UNPAID* when the status equals *Unpaid* |

## Text blocks

A **text block** holds longer text: terms and conditions, notes, a cover letter. Write it with a few simple marks — HTML is not used, so values are always shown as text:

```text
# Terms                     a heading (## and ### are smaller)
**bold**, *italic*, __underlined__
- a bullet point
1. a numbered point
                            an empty line starts a new paragraph
Payment by {{due_on|date:MMM D, YYYY}}.
```

With **Continue on new pages when it does not fit**, the text that does not fit in the box goes onto **added pages**, split between paragraphs and list items. The added pages also get the elements shown on every page (headers, footers, page numbers — `{{pages}}` counts them too), and the text starts at *Top on added pages* with *Height on added pages* (by default, inside the margins). A text block shown on the last page with *Move up under the table* starts right under the items, so long terms follow the totals. Only one text block per template continues like this.

## Postcards, envelopes and stamps

**Postal code.** The *Postal code* element puts one digit into each of the seven boxes printed on a Japanese postcard (100 × 148 mm). Its defaults fit the postcard: boxes 5.7 × 8 mm, 7 mm apart for the first three digits, 7.6 mm from the third to the fourth and 6.8 mm for the last four. Envelopes differ by maker: print one test page and move the element or change the spacing. *Print the boxes too* draws the boxes for plain paper. Full-width digits and the hyphen in `100-0001` are handled.

**Vertical addresses.** Turn on *Vertical writing* for a text or field. *Kanji numerals* (in the format) turns `1-2-3` into 一ー二ー三, and *Keep letters and digits upright* stands Latin letters and digits upright instead of on their side. The paper sizes include postcards and Naga 3, Naga 4 and Kaku 2 envelopes; the postcard and Naga 3 [starters](#starter-templates) have everything placed.

**Stamps.** The *Stamp* element draws a red seal from a name: a round name seal (認印; a name of up to four kanji or kana is written top to bottom), or a **date stamp** (日付印) with a top line such as a department, the date and the name. When the name is empty nothing is drawn, so a stamp in an approval box appears only once someone has approved — see [Approvals with Warp Flow](#approvals-with-warp-flow). For a company seal (角印), upload its image instead.

## Barcodes

The *Barcode* element draws nine kinds of code. Choose the kind under **Type**; a line below it says what the value must look like. The value is a field of the source or fixed text with `{{fields}}`. Bars are drawn as vectors by the same renderer in the designer, the print window and the server PDF, so they stay sharp at any size.

| Type | Value | What is added for you |
|---|---|---|
| **Code 128** | Any letters, digits and symbols of ASCII | Runs of digits are packed two to a symbol |
| **GS1-128** | Elements with the application identifier in brackets: `(01)04912345678904(10)LOT42` | The FNC1 at the start, and the separator after an element of variable length when another follows |
| **EAN-13 / JAN** | 12 or 13 digits | The check digit, when 12 digits are given |
| **UPC-A** | 11 or 12 digits | The check digit, when 11 digits are given |
| **Code 39** | Capitals, digits, spaces and `- . $ / + %` | Small letters become capitals; the `*` at both ends |
| **ITF / ITF-14** | Digits | 13 digits get the GS1 check digit (ITF-14); another odd number of digits gets a `0` in front |
| **NW-7 (Codabar)** | Digits and `- $ : / . +`, with a start and a stop letter `A`–`D` around them | `A` at both ends when there are none |
| **DataMatrix** (ECC 200) | Text up to 174 codewords: about 170 letters, or 340 digits | The smallest square size from 10 × 10 to 48 × 48 that holds the value |
| **Japan Post customer barcode** | The 7-digit postal code, a space, and the numbers of the address: `1000013 1-3-2-503` | Letters as control codes, padding to 13 characters, the check digit, start and stop bars |

```php
// In a source: the values the barcodes read
Field::text('jan'),                                                              // EAN-13 / JAN
Field::text('carton')->state(fn (Product $product) => '1'.substr($product->jan, 0, 12)),   // ITF-14: 13 digits, the check digit is added
Field::text('gs1')->state(fn (Product $product) => '(01)0'.$product->jan.'(10)'.$product->lot),
```

- **Show the number under the bars** prints what is really in the bars: the check digit that was added, the capitals of Code 39, the start and stop letters of NW-7. (Code 128 and EAN-13 print the value as it was given.)
- When a value does not fit the type — a wrong check digit, a letter in ITF, text too long for DataMatrix — nothing is printed in that place, and the designer says so under the type while the sample value does not fit. A record whose value is empty prints no barcode either.
- **Sizes.** The bars stretch to the box you draw, with a quiet zone of 10 modules left and right inside it. Keep the narrowest bar at 0.25 mm or more for laser and inkjet printers: as a guide, give EAN-13 and UPC-A at least 30 mm, and a Code 128 of 10 characters at least 40 mm. DataMatrix stays square inside its box; leave a margin of one dot around it.
- **Japan Post customer barcode.** Make the box **79.8 × 3.6 mm** (the size for 10-point address text) and keep 2 mm clear around it. The value is the postal code and the *address display number* — the numbers of the address after the town name, joined by hyphens (`1-3-2-503` for 1 丁目 3 番 2 号, room 503). Working that number out from a free-text address is left to your source, as only your app knows how addresses are stored:

  ```php
  Field::text('postal_barcode')->state(function (Customer $customer): string {
      preg_match_all('/\d+/', $customer->address.' '.$customer->address2, $numbers);

      return preg_replace('/\D/', '', $customer->postal_code).' '.implode('-', $numbers[0]);
  }),
  ```

  Japan Post's discount for barcoded mail has its own conditions (number of items, sorting, a test print checked by the post office); print a sample and have it checked before a large mailing.
- **What is not covered.** Code 93, PDF417, GS1 DataBar, rectangular DataMatrix and sizes above 48 × 48, check characters for Code 39 and NW-7 (add them in your `state()` when your reader needs one), and non-ASCII text in DataMatrix read back as text by every scanner (bytes are written as UTF-8 without an ECI mark; stay with ASCII for values a scanner must type).

Every type is tested by drawing the picture and reading it back with a decoder (ZXing), in the unit tests and — on screen and in the server PDF — in the browser check `tests/browser/barcodes.cjs` of the development app. The Japan Post barcode, which that decoder cannot read, is checked bar by bar against the published table and check digit.

## Charts

A monthly report reads faster with a chart next to its table. The *Chart* element draws **bars, a line or a pie** from the rows of a table of the source — the same rows a table element prints, so the chart and the table under it always agree. It is drawn as vector shapes by the same renderer in the designer, the print window and the server PDF: sharp at any size, with text that can be selected and searched in the PDF.

```php
// In a source: a table with a name and one or more numbers for each row
Field::table('monthly', [
    Field::text('month'),
    Field::money('billed', 'JPY'),
    Field::number('invoices'),
])->state(fn (Customer $customer) => $customer->invoices
    ->sortBy('issued_on')
    ->groupBy(fn (Invoice $invoice) => $invoice->issued_on->format('Y-m'))
    ->map(fn ($invoices, string $month) => ['month' => $month, 'billed' => $invoices->sum('total'), 'invoices' => $invoices->count()])
    ->values()),
```

Click **Chart** in the designer: it starts as a bar chart of the first table. On the right:

| Setting | What it does |
|---|---|
| **Bars / Line / Pie** | The kind of chart |
| **Rows from** | The table whose rows are drawn. One bar, point or slice for each row, in the order of the rows |
| **Name of each row** | The column printed under the bars and points, and in the legend of a pie |
| **Values** | The number or money column to draw. *Add another value* puts up to four columns side by side (bars) or as lines, each with its colour; a legend with their names appears from the second one. A pie draws the first value only |
| **Title** | A line above the chart; it may use `{{fields}}` of the record |
| **Show the values** | Writes each value above its bar or point, or the share (%) on the slices of a pie. Values that would not fit are left out |
| **Show the legend** | For a single value in a bar or line chart (with two or more it is always shown) |
| **Name of the slice for the rest** | Pie only: the name of the slice that adds the smaller rows together when there are more than 12 |

- **Numbers.** Values are written like the column they come from: a `Field::money()` column prints `¥430,130` on the axis, above the bars and in the legend. The axis starts at zero and uses round steps; whole-number data (counts) gets whole-number steps. Bars below zero hang down from the zero line.
- **Size and text.** The chart fills the box you draw. The text size is the element's font size (7 pt at first); names on the axis that would overlap are thinned out, and long names are cut with `…`.
- **Limits.** Up to **100 rows** are drawn in a bar or line chart (more are left out, from the end). A pie keeps the 11 largest rows and adds the rest into one slice; rows of zero or less have no slice. A value that is not a number counts as 0.
- **Nothing to draw.** With no rows, no value column, or only zeros in a pie, nothing is printed in the chart's place (the designer shows an empty box). A chart whose table or columns are no longer in the source is reported under it as *Fields that are not in the data*.
- A chart is an ordinary element: it can be limited to the first or last page, follow the end of an item table, or appear only when a field matches.
- Not covered: stacked or horizontal bars, areas, two axes, and charts from separate fields of the record (give the chart a table — a `state()` returning two rows is enough).

In a layout kept in code:

```php
['type' => 'chart', 'x' => 15, 'y' => 42, 'w' => 180, 'h' => 72,
    'kind' => 'bar',                    // bar, line or pie
    'field' => 'monthly',               // the table
    'labelField' => 'month',
    'series' => [['field' => 'billed', 'label' => 'Billed', 'color' => '#2563eb']],
    'title' => 'Billed by month', 'showValues' => true, 'legend' => false,
    'format' => ['kind' => 'number', 'decimals' => 0],   // optional: how values are written, as for a field
    'style' => ['size' => 7.5]],
```

## Labels

![Price tags in US dollars with an EAN-13 barcode, three copies of each](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/labels.png)

A *Labels* template designs **one label**. The sheet is set in the page settings: labels across and down, label size, the left and top margins of the sheet, the gaps between labels, and whether to fill rows or columns first. Presets cover common A4 and Letter sheets.

Print labels with `PrintBulkAction` (or `PrintAction` for one). The print window asks for **Labels already used** — to start on a partly used sheet — and **Copies of each**.

### A label for each item row, as many as the quantity

A delivery note of three item rows — 3 boxes, 1 box, 12 boxes — needs 16 item tags, not one label. Under **How many labels** in the page settings of a label template:

| Setting | Choices | What it does |
|---|---|---|
| **One label for** | *Each record* (default) or *Each row of* one of the source's tables | One label for every row of that table, in the order of the rows. A record without rows prints no label |
| **Labels of each** | *One* (default) or a number field | That many labels of each record or row: `3` gives three, `0` or less gives none, a fraction is rounded up (2.5 → 3), an empty value or text gives one. At most 1,000 from one value |

With *Each row of Items* chosen, the columns of the row appear in the field list as fields of their own (`items.name`, `items.quantity`, …) and can be dragged onto the label, used in text as `{{items.name}}`, or read by a barcode. `{{items.#}}` is the number of the row. The fields of the record itself (`number`, `customer.name`) stay available on every label. Two more fields say which label of a run this is: `{{label.number}} / {{label.count}}` prints `1 / 3`, `2 / 3`, `3 / 3`.

```php
// The source needs nothing new: the table it already has, with the quantity as a number
Field::table('items', [
    Field::text('product.code'),
    Field::text('name'),
    Field::number('quantity'),
]),
```

In a layout kept in code (`WarpPrint::syncTemplate()`), the two settings are `labels.each` and `labels.copiesField`:

```php
'labels' => ['columns' => 3, 'rows' => 7, 'width' => 63.5, 'height' => 38.1, /* … */ 'each' => 'items', 'copiesField' => 'items.quantity'],
```

- **Copies of each** in the print window multiplies the number: quantity 3 and 2 copies give 6 labels, numbered 1 / 6 to 6 / 6.
- The print window shows how many labels will come out next to the page count. One print is limited to **5,000 labels**; when there would be more, only the first 5,000 are drawn and the window says so — select fewer records and print the rest separately.
- *Labels of each* works without rows too: a field of the record (`Field::number('boxes')`) prints that many labels of a record.
- A column of a table can only be placed outside a table in a template that prints one label for each row of that table. Anywhere else the designer reports it under the element as a field that cannot be used, and nothing is printed for it.
- Only the records the user may print are read (`PrintSource::canPrint()` and the tenant scope), as for every other template: the rows come from those records.

## Excel forms

![The fields found in an uploaded Excel invoice, each checked against the data](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/excel.png)

Choose *Excel or Word file* as the kind and upload an `.xlsx` file. In the cells where values go, write fields in double braces, exactly like a text element: `{{customer.name}} 御中`, `No. {{number}}`, `{{issued_on|date:YYYY年M月D日}}`.

- A cell with **only one field** (`{{total}}`) receives the value itself — a number or a date — so the cell's own number format applies (`¥#,##0`, `yyyy/m/d`). A cell with text around the field receives formatted text.
- **Item rows:** write the item fields (`{{items.name}}`, `{{items.quantity}}`) in one row. That row is used for the first item. The next items go into the **empty rows below it that have the same borders** (the ruled rows of a form), and when those run out, rows are added. Formulas in the row (`=E11*G11`) are copied for each item; merged cells are repeated.
- Everything under added rows moves down: cells, merged cells, formulas that point below (`=H25*0.1`), conditional formats, data validation, page breaks, hyperlinks, images and charts' anchors, and the print area. A `SUM(H11:H20)` over the item rows grows to cover the added rows.
- Formulas are recalculated when the file is opened.

The settings page lists every `{{…}}` found in the file and whether the data has that field, and **Download with a sample** fills it with the newest record. Uploading a new file publishes a new version. From the print window, Excel templates download as `.xlsx` (several records: a ZIP). With the Gotenberg driver one record at a time can also be downloaded as PDF.

## Word documents

Contracts, notices and reminders are usually written in Word, by someone who will not learn a designer for them. Choose *Excel or Word file* as the kind and upload the `.docx` file as it is, with fields written in double braces where the values go — the same way as in an Excel form.

```text
Dear {{customer.contact}},

Our records show that invoice {{number}}, issued on {{issued_on|date:MMMM D, YYYY}},
has not been paid yet. The amount due is {{total}}.

| #           | Item           | Qty                | Amount           |
| {{items.#}} | {{items.name}} | {{items.quantity}} | {{items.amount}} |
|             | Total due      |                    | {{total}}        |
```

- **Where fields can be written:** anywhere in the text, in tables, in the header and the footer, in footnotes and in text boxes. Styles, fonts, images, page settings and everything else in the file stay as they are.
- **Formats** follow the field, as in text elements: money as `¥25,630`, dates as `2026/11/12`, and `{{field|format}}` changes it (`{{issued_on|date:MMMM D, YYYY}}`, `{{total|number}}`, `{{customer.name|upper}}`). The document is written in the record's language ([Document language](#document-language)).
- **The value takes the look of the first brace.** Word often saves `{{customer.name}}` in several pieces (after a spelling check, or when part of it was made bold). The pieces are joined when the file is read, and the value gets the font, size and colour of the `{{` it starts with.
- **Item rows:** a table row that contains fields of a table of the source (`{{items.name}}`) is repeated for every item, with the row's settings and formatting. `{{items.#}}` is the number of the row; fields of the record (`{{number}}`) may be used in the row as well. With no items, the row stays once, empty. A table inside a repeated row is filled first.
- **Several lines:** a value with line breaks (a note, an address) becomes lines in the same paragraph.
- The settings page lists every `{{…}}` found, where it is (text, header, footer) and whether the data has that field; **Download with a sample** fills the file with the newest record. Uploading a new file publishes a new version.
- From the print window, Word templates download as `.docx` (**Download Word**; several records: a ZIP with one file each). Emails attach the `.docx`, and background exports make *One Word file per record, in a ZIP*.
- **PDF:** with the Gotenberg driver (LibreOffice) one record at a time can be downloaded, emailed or exported as PDF, as for Excel forms. The Browsershot driver cannot convert Word files.

From code: `WarpPrint::docx('payment-reminder', $invoice)` returns the bytes of the filled `.docx` (several records: a ZIP). `WarpPrint::xlsx()` is for Excel templates and `WarpPrint::docx()` for Word templates; each refuses the other kind.

**What a Word template cannot do** — use a designed layout when you need these:

- Only **table rows** are repeated. A paragraph, a bullet of a list or a whole section is not repeated for each item, and an item field written outside a table row prints nothing.
- **Images** (`Field::image()`: a logo, a seal, a signature) are not inserted; put the fixed images in the file itself. QR codes and barcodes are not drawn.
- There are no conditions: a paragraph cannot be shown or hidden by a field, and there are no totals, page subtotals or carried rows (write the total as a field of the source).
- Fields are read in the document's text. A field inside a Word *field code*, a comment, a tracked deletion, a chart or SmartArt is not replaced; neither is a `{{…}}` that is split across two paragraphs.
- Numbers go in as formatted text: Word does not calculate with them.
- `.doc` (the old format), `.docm` (macros) and `.dotx` files are not accepted; save the file as `.docx`. Files are limited to 10 MB.

## PDF and paper forms

![A scanned credit application filled in on screen: the company, contact, email and address from the customer, a ZIP code one digit per box, a ticked box and a signature](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/forms-fill.png)

Some forms are not yours to design: a government form, a bank's application, a customer's own order sheet, a paper form that has been used in the office for years. Warp Print takes the form **as it is** — a PDF, or photos or scans of the paper — and lets admins put fields on it. Then:

- **Fill it in on screen.** Staff press *Fill in form* on a record. The form opens with the record's values already written in; they type on the page itself, sign with a finger or the mouse, and submit. What they wrote can be saved back to the record.
- **Send it to someone outside.** *Send to fill in* emails a link to a customer, supplier or applicant. They fill in and sign the form in their browser **without an account**. The link works until a date you choose and closes once it is submitted; the filled form waits in *Filled forms* until someone checks it and saves it to the record.
- **Just print it.** Pick the form in the normal *Print* window and the record's values go straight into it — a W-9 for every vendor, a change-of-address form for a customer.

The PDF's own lines and text are never redrawn: values are written on top of the original page, so the result looks exactly like the official form. If the PDF has its own fillable fields (AcroForm), they are found when it is uploaded and filled directly.

### Making a form

1. **Print templates → New → *PDF or paper form*.** Choose the source as for any template.
2. **Choose the form.** One PDF, or PNG / JPEG images (one per page; they are turned into a PDF in your browser). If the PDF has fillable fields, the designer offers to add them all.
3. **Place the fields.** For scans, press **Find boxes on this page**: ruled boxes, check boxes and rows of one-character boxes (postal codes, account numbers) are found and shown as dashed boxes — click one to add it, or *Add all*. Boxes that already contain printed text (the *Company name* label next to the box) are left out. Or add fields from the palette, or drag a field from your source onto the page.
4. **Give each field a label** (shown to the person filling it in) and, if it comes from the record, choose it under *Fill in from*.
5. **Preview** with a sample record, then **Publish**.

![The form designer: the scanned credit application with fields on its boxes, the ZIP code field selected and filled in from the customer's postal code](https://raw.githubusercontent.com/qalainau-labs/filament-warp-print-docs/main/art/forms-designer.png)

Each field has a type:

| Type | Use it for |
|---|---|
| Text, Several lines | Names, addresses, notes. Long text becomes smaller to fit, or wraps on the paper's own ruled lines (*Line spacing*) |
| Number, Amount, Date | Formatted like the rest of Warp Print: `1,234.50`, `$1,234.50`, `2026/11/05`, `令和8年11月5日` |
| One per box | One character per printed box: postal codes, phone and account numbers. Set the number of boxes, the gap and whether to start from the left or the right |
| Check box | A tick, cross, dot or filled box when it is chosen |
| Choice | One of several options with the same name: *circle* "Yes" or "No", tick one of four plans |
| List | Pick one item from a list; the item is written on the form |
| Signature, Photo | Drawn on a pad (finger, pen or mouse) or uploaded, then fitted into the box |
| Seal | A round red seal with a name (印鑑) |
| Calculated | A formula over other fields: `quantity * price`, `sum(items.amount)`, `round(total * 0.1)` |
| Item rows | A table with a fixed number of rows on the form. Rows that do not fit go onto an extra sheet after the form (*Items (continued)*) |
| Fixed text | Text that is the same on every copy, or `{{customer.name}}` from the record |

**One value, several places.** Fields with the same *name* share one value. Add a part with **Add another part** to write a date as `令和 [8] 年 [11] 月 [5] 日` in three separate boxes (formats `GGGG`, `E`, `M`, `D`), or to repeat the applicant's name on page 2.

**Checks.** *Must be filled in*, minimum and maximum length, smallest and largest number, and a format: digits only, email, phone number, Japanese postal code, katakana, hiragana… Full-width digits (`１２３`) are accepted and stored as `123`. The same checks run in the browser while typing and again on the server when the form is saved.

### Filling in from the panel

```php
use Qalainau\FilamentWarpPrint\Actions\FillFormAction;
use Qalainau\FilamentWarpPrint\Actions\SendFormLinkAction;

->recordActions([
    FillFormAction::make()->template('credit-application'),
    SendFormLinkAction::make()->template('credit-application'),
])
```

*Fill in form* opens the form for the record (or continues the draft this user already started). Every change is saved as a draft after a moment. The page can be filled **on the form** or **as a list** — on a phone, the list opens first and shows the part of the paper around each field, so people know where they are writing. *Download PDF* makes the filled PDF in the browser at any time. *Submit* checks the required fields; after that the entry can no longer be changed (*Allow changes again* reopens it).

Filled forms are listed under **Filled forms**, with who filled them in and when. Forms that are not linked to a record (an application form) can be started there with **Fill in a new form**.

### Saving the values back to the record

A field can only change the record if your source allows it:

```php
public function fields(): array
{
    return [
        Field::text('name')->writable(),
        Field::text('phone')->writable(),
        // A relation or a computed value: say how to save it
        Field::text('contact.email')->writable(fn (Customer $record, ?string $value) => $record->contact->update(['email' => $value])),
        Field::date('created_at'), // filled in, never saved back
    ];
}
```

When **staff** submit a form, the writable fields are saved to the record at once (if the record's policy allows `update`; override `canWriteBack()` to decide yourself). Forms sent by **link** are not saved automatically: open the entry, check it, and press **Save to record**. Calculated fields, signatures, photos and item rows are never saved back. A field marked *Cannot be changed* keeps the record's value whatever the browser sends.

### Links for people without an account

`SendFormLinkAction` asks for the email address, a name, how many days the link works (14 by default) and a message. The first address comes from your source:

```php
public function formRecipient(Model $record): array
{
    return ['name' => $record->contact_name, 'email' => $record->email];
}
```

The email contains a button to the form. The page at `/warp-print/forms/{token}` needs no login: the token is long, random and stored only as a hash; the link stops working when it expires or when the form is submitted, and saving is rate limited per link (`forms.link.per_minute`). When they submit, they can leave their name and email, and get the filled PDF to keep. *Send a new link* in *Filled forms* makes a new link and turns the old one off.

Change the email with `->modifyMailUsing(fn (FormLinkMail $mail) => $mail->replyTo('hr@example.com'))`. To react to submissions — create a record from an application form, tell someone — listen for `FormEntrySubmitted`:

```php
use Qalainau\FilamentWarpPrint\Events\FormEntrySubmitted;

Event::listen(function (FormEntrySubmitted $event) {
    if ($event->byLink) {
        Notification::make()->title('A form was submitted')->sendToDatabase($event->entry->template->owner);
    }
});
```

### Filled PDFs on the server

Downloading and printing work in the browser and need nothing on the server. To make filled PDFs **on the server** — to attach them to emails, make many at once in the background, store the submitted PDF, or call `WarpPrint::pdf('w9', $vendor)` from code — Warp Print runs the same JavaScript in **Node.js 18 or later**. Chrome is not needed.

```bash
php artisan warp-print:fonts         # download Noto Sans JP and Noto Serif JP into storage/app/warp-print/fonts
php artisan warp-print:check-forms   # find node, list the fonts and fill a test page
```

If PHP cannot find `node` (PHP-FPM often has a short `PATH`), set the full path: `WARP_PRINT_FORMS_NODE=/usr/bin/node`. Set it to `null` in the config to turn server PDFs off.

**Fonts.** A PDF needs the font inside it. Without an installed font only English letters can be written, and the designer warns about it. `warp-print:fonts` installs Noto Sans JP and Noto Serif JP (SIL Open Font License), which also cover Chinese characters. For Korean, or your own fonts, point `forms.fonts` at TrueType files:

```php
'fonts' => [
    'sans' => ['label' => 'Sans-serif', 'google' => 'Noto Sans KR', 'regular' => storage_path('app/warp-print/fonts/NotoSansKR-Regular.ttf'), 'bold' => storage_path('app/warp-print/fonts/NotoSansKR-Bold.ttf')],
],
```

Only the characters used are embedded, so filled PDFs stay small.

### From code

```php
$forms = WarpPrint::forms();

$entry = $forms->start('credit-application', $customer);          // a draft with the record's values
$errors = $forms->save($entry, ['credit_limit' => '5000'], submit: true, user: auth()->user());
$link = $forms->invite('w9', $vendor, 'ap@vendor.com', 'Ann', days: 7);  // ['entry' => ..., 'url' => ...]
$forms->apply($entry);                                             // save the writable fields to the record
$pdf = $forms->pdf($entry);                                        // needs node

WarpPrint::pdf('change-of-address', $customer);                    // the record's values, no entry
```

An entry keeps the **version** of the form it was started on. If you replace the PDF or move fields later and publish again, entries that were already started are still printed on the old version.

## Printing a list

```php
use Qalainau\FilamentWarpPrint\Actions\PrintTableAction;

->headerActions([
    PrintTableAction::make(),
    // PrintTableAction::make()->title('Open invoices')->landscape()->limit(5000)->exceptColumns(['notes'])
])
```

The list is printed as the table is now: its filters, search and sort, and its visible columns, with the text the table shows (`money()`, `formatStateUsing()`, enum labels, ✓ for icon columns). The heading repeats on every page, with the page number and when it was printed. With more than six columns it turns sideways (`->landscape(false)` to keep it upright). At most 2,000 rows unless `->limit()` says otherwise.

### Totals and groups

A printed list of invoices is usually read for its total, and often for the total of each customer.

```php
PrintTableAction::make()
    ->totals(['total', 'tax'])     // a Total row at the end for these columns
    ->groupBy('customer.name'),    // a heading for each customer, and a Subtotal under each when there are totals

PrintTableAction::make()->totals()->groupBy(),   // follow the table: its Sum summarizers and the group chosen on screen
```

| Method | Argument | What it prints |
|---|---|---|
| `totals(array\|Closure\|bool $columns = true, string\|Closure\|null $label = null)` | Column names; `true` (or nothing) for the columns that have a `Sum` summarizer in the table; `false` for none | A last row with the sums, labelled *Total* (translated) or `$label`. The sum is formatted by the column itself, so a `money('JPY')` column prints `¥16,397,800` |
| `groupBy(string\|Closure\|bool $column = true)` | A column's name, or a field of the record that is not a column (`customer.name`); `true` (or nothing) for the group chosen with Filament's *Group by* (`$table->groups([...])`); `false` for none | A heading row for each group (`Customer: Aoba`), and with totals a *Subtotal* row under each group |

- Neither is on unless you call it: `PrintTableAction::make()` prints the list as before.
- Sums are made from the column's raw value (`getState()`) of the printed rows; values that are not numbers count as 0. A column that is hidden or left out with `exceptColumns()` is not totalled.
- The totals cover **the rows that are printed**. When the list is longer than `->limit()`, the last row says so (*Total of the 2,000 rows printed*) next to the note that the list was cut.
- `groupBy('column')` keeps each group together in the order the groups first appear, with the rows of a group in the table's order. `groupBy()` without a name prints the groups as the screen shows them, and no headings when no group is chosen.
- A heading is never left alone at the bottom of a page, and a subtotal never starts a page without the row above it. Heading texts come from your data and are printed as text.
- The list still contains only what the table's own query returns for this user and tenant: totals and groups are worked out from those rows.

## PDF files on the server

Printing from the print window needs nothing on the server: the browser draws the pages and its print dialog can save a PDF. To make PDF files on the server — the **Download PDF** button, mail attachments, `WarpPrint::pdf()` — choose a driver:

**Browsershot** (Node and Chrome on the server):

```bash
composer require spatie/browsershot
npm install puppeteer
```

```dotenv
WARP_PRINT_PDF_DRIVER=browsershot
# if needed:
WARP_PRINT_NODE_BINARY=/usr/bin/node
WARP_PRINT_CHROME_PATH=/usr/bin/google-chrome
WARP_PRINT_NO_SANDBOX=true   # when Chrome runs as root, e.g. in Docker
```

**Gotenberg** (a Docker container with Chrome and LibreOffice; also turns Excel templates into PDF):

```bash
docker run --rm -p 3000:3000 gotenberg/gotenberg:8
```

```dotenv
WARP_PRINT_PDF_DRIVER=gotenberg
WARP_PRINT_GOTENBERG_URL=http://localhost:3000
```

Both open the same page as the print window and wait until it is drawn. Measured with Browsershot on a laptop (Apple M4 Max): 36 invoices (40 A4 pages) in 1.5 s, 200 price tags in 1.0 s. To use another service, bind your own `Qalainau\FilamentWarpPrint\Pdf\PdfDriver`.

### Checking the setup

After deploying, run the check on the server. It makes one PDF with the configured driver and tells you what happened, without waiting the full two minutes when something is wrong:

```bash
php artisan warp-print:check-pdf
php artisan warp-print:check-pdf --template=invoice --output=storage/app/check.pdf --timeout=60
```

```
 Driver ...................................... browsershot
 Chrome ......... /Applications/Google Chrome.app/Contents/MacOS/Google Chrome
 Page ...................... Built-in test page (210 × 297 mm)
 Time ............................................... 1.03 s
 Pages ................................................... 1
 Fonts .......... HiraginoSansGB-W3, NotoSansJP-Thin, NotoSerifJP-ExtraLight
 Japanese, Chinese, Korean ...... OK (HiraginoSansGB-W3, NotoSansJP-Thin, …)

  INFO  The PDF driver works.
```

- Without `--template` it prints a built-in page with Latin, Japanese, Chinese and Korean text, so it works before any template exists. With `--template=key` it prints that template with sample data (designed templates only, not Excel forms).
- **Fonts** are the fonts embedded in the PDF. When none of them has Japanese, Chinese or Korean characters, it warns that those characters would print as empty boxes, and how to install a font.
- When the PDF cannot be made, it shows the cause (not the whole Chrome command) and a hint: Chrome or Node not found, no sandbox in Docker, Gotenberg not reachable, or the page not finished in time — usually a font stylesheet the server cannot load.
- `--timeout` (seconds, default 30) replaces the driver's timeout for this check only. It exits with 1 when no driver is set or the PDF fails, so it can run in a deploy script.

## Sending by email

```php
use Qalainau\FilamentWarpPrint\Actions\EmailPrintAction;
use Qalainau\FilamentWarpPrint\Actions\EmailPrintBulkAction;

->recordActions([
    PrintAction::make(),
    EmailPrintAction::make(),                    // "Email" on every invoice
])
->toolbarActions([
    EmailPrintBulkAction::make()->template('invoice'),  // each selected invoice to its own customer
])
```

**EmailPrintAction** opens a form with the recipients, Cc, the subject and the message filled in, and sends the record's PDF as an attachment. It is shown when there is something to attach: a [PDF driver](#pdf-files-on-the-server) for documents and labels, or an Excel template (attached as `.xlsx`; with Gotenberg, as PDF or Excel). Addresses are checked before sending.

Say where each record goes in the source:

```php
public function mailRecipients(Model $record): array
{
    return array_filter([$record->customer?->billing_email]);
}
```

The **subject and message** come from the template's settings (*Email*), with fields: `Invoice {{number}} from {{company.name}}`, `The total is {{total}}, due on {{due_on|date:MMM D, YYYY}}.` Without them, the subject is the template name and the record's title. The message is sent as plain text (and as simple HTML with line breaks, never the admin's HTML).

**EmailPrintBulkAction** sends every selected record to **its own** recipients from `mailRecipients()`. The subject and message are written once with fields and filled in for each record. Sending runs [in the background](#large-batches-in-the-background); records without a recipient are skipped, and the notification at the end says how many were sent, skipped and failed.

Every email is logged with its recipients (format *Emailed*), and the PDF is kept as an [issued document](#issued-documents-electronic-bookkeeping) when the template keeps them. To change the mail — a reply-to address, a BCC to accounting — use `->modifyMailUsing(fn (PrintMail $mail, Invoice $record) => $mail->replyTo($record->owner->email)->bcc('accounting@example.com'))`. The mailer is the app's default, or `mail.mailer` in the config. From code: `WarpPrint::send('invoice', $invoice, ['a@example.com'])`.

**After sending.** Every email fires `Qalainau\FilamentWarpPrint\Events\DocumentEmailed` — from `EmailPrintAction`, from `WarpPrint::send()` and once **per email** in a background batch (`EmailPrintBulkAction`, `WarpPrint::queue(..., 'email')`). Use it to note on the record that the document went out:

```php
use Illuminate\Support\Facades\Event;
use Qalainau\FilamentWarpPrint\Events\DocumentEmailed;

// in a service provider's boot()
Event::listen(function (DocumentEmailed $event) {
    if ($event->template->key === 'service-report' && $event->record instanceof WorkOrder) {
        $event->record->forceFill(['report_sent_at' => now()])->saveQuietly();
    }
});
```

| Property | What it holds |
| --- | --- |
| `template` | The `PrintTemplate` that was sent |
| `record` | The record (as read again through the source) |
| `to`, `cc` | The addresses, trimmed (`list<string>`) |
| `user` | Who sent it — in a background batch, the user who asked; `null` when nobody is logged in |
| `attachment` | `'pdf'`, `'xlsx'` or `'docx'` |

The event fires **after the mailer has accepted the email**: Warp Print sends the mail right away rather than putting it on the mail queue, so when the event fires the message has been handed to your mail server (SMTP, SES, …). It does not tell you that it reached the inbox. In a background batch the whole job runs on the queue, so the event fires inside the queue worker, as each email goes out. When sending throws (no recipient, a record the user may not print, the mail server refused it), the event does not fire.

## Large batches in the background

When more records are selected than `batch_limit` (200), the print window does not draw them. It offers to make them **in the background**:

- **One PDF** — very long runs are split into parts of `exports.chunk` records (100) in a ZIP, so Chrome does not run out of memory;
- **One PDF per record, in a ZIP** — named after each record's title;
- **One Excel file per record, in a ZIP** — for Excel templates.

A queued job makes the file as the user who asked — in the same panel, tenant and language, with the same permissions — and logs each record. When it is done, the user gets a **Filament notification** with a **Download** button. The link is signed and works only for that user while logged in; the file is kept for `exports.keep_hours` (72) and then removed by `php artisan model:prune`. If the job fails, the notification says why.

This needs:

1. **A queue worker** (`php artisan queue:work`, or Laravel Horizon). Use `exports.connection` and `exports.queue` to send these jobs to their own queue; a large PDF can take minutes, so give that worker a long `--timeout` (the job allows 3600 seconds).
2. **Database notifications in the panel**: `->databaseNotifications()` on the panel and the `notifications` table (`php artisan make:notifications-table`).
3. **A PDF driver** for PDFs.
4. **The scheduler** running `model:prune` so old files are deleted: `Schedule::command('model:prune', ['--model' => [\Qalainau\FilamentWarpPrint\Models\PrintExport::class]])->daily();`

At most `exports.limit` records (5,000) at a time. From code: `WarpPrint::queue('invoice', $invoices, 'pdf')` (`'pdf_each'`, `'xlsx'` or `'email'`) returns the `PrintExport`.

## Issued documents (electronic bookkeeping)

Japan's electronic bookkeeping law (電子帳簿保存法) asks businesses to keep the invoices they send electronically, unchanged, and to find them by **date, amount and counterparty**. Warp Print can keep every PDF the server makes for chosen templates.

1. Turn it on: `WARP_PRINT_ARCHIVE=true` (config `archive.enabled`). The panel gets an **Issued documents** page, and each template's settings get **Keep the issued PDFs**.
2. Turn that on for the templates that are business documents (invoices, receipts, quotes — not labels).
3. Say what to search by in the source:

```php
public function archiveMeta(Model $record): array
{
    return [
        'number' => $record->number,
        'date' => $record->issued_on,          // the transaction date
        'amount' => $record->total,
        'counterparty' => $record->customer->name,
    ];
}
```

From then on, every PDF the server makes for such a template — **Download PDF**, an email, a background job — is stored on `archive.disk` with its SHA-256 hash, the template and version, the record, the user, the tenant and how it went out (download, email with the recipients, or background). When several records are printed together, each record is kept as its own PDF. Printing the same record again with the same data does not store a second copy; a changed record or a new template version does.

The **Issued documents** page lists them with filters for the date range, the amount range, the counterparty and the template. **Download** gives the stored file; **Check integrity** compares it with its hash. Nothing on the page can change or delete a document, and the model refuses updates and deletes. In the print window of one record, *Download the issued PDF* gives the stored PDF again instead of making a new one — a reprint is then exactly what was sent.

Documents are kept forever unless you set `archive.keep_years` (for example 10) and run `model:prune` for `PrintDocument`. Who may open the page is `WarpPrintPlugin::make()->canViewArchive(fn ($user) => $user->is_accountant)`; `->archive(false)` hides it in a panel. Keeping the files is only part of the law (you also need, for example, rules against changing them, or a timestamp); check what your business needs with your accountant.

## Print log and reprints

Every print is logged in `warp_print_logs`: the template and version, the record, the user, the tenant and whether it was printed (the print dialog was opened), downloaded as a PDF or as Excel. From the second print of the same record with the same template, `print.reprint` is true — show a *Reprint* / *COPY* / 再発行 mark with a condition. Logs are kept for `keep_logs_days` (365); prune older ones from your own scheduler:

```php
Schedule::call(fn () => \Qalainau\FilamentWarpPrint\Models\PrintLog::query()
    ->where('created_at', '<', now()->subDays(config('filament-warp-print.keep_logs_days')))->delete())->daily();
```

## Using it from code

```php
use Qalainau\FilamentWarpPrint\Facades\WarpPrint;

$pdf = WarpPrint::pdf('invoice', $invoice);                 // bytes of a PDF (needs a driver)
$pdf = WarpPrint::pdf('invoice', $invoices);                // many records, one PDF
$xlsx = WarpPrint::xlsx('invoice-excel', $invoice);         // an Excel template, filled (several records: a ZIP)
$docx = WarpPrint::docx('payment-reminder', $invoice);      // a Word template, filled (several records: a ZIP)
$html = WarpPrint::html('invoice', $invoice, ['autoprint' => true]); // a page that draws itself and opens the print dialog

Mail::to($invoice->customer->email)->send(new InvoiceMail($invoice, $pdf));

WarpPrint::send('invoice', $invoice, ['billing@acme.test'], cc: ['boss@acme.test']); // PDF attached, subject from the template
$export = WarpPrint::queue('invoice', $invoices, 'pdf');    // in the background, the user is notified with the download
$document = WarpPrint::issued('invoice', $invoice);         // the PDF last issued for the record (when kept)
['template' => $template, 'missing' => $missing] = WarpPrint::fromStarter('invoice-en', InvoiceSource::class);

// Templates kept in your code: created and published the first time, a new version only when the layout changed
['template' => $template, 'changed' => $changed] = WarpPrint::syncTemplate('invoice', InvoiceSource::class, $layout, name: 'Invoice', note: 'Standard invoice');
```

`syncTemplate()` compares the layout with the published one and ignores the element IDs (which change every time a layout is saved), so it can run in a seeder or on every deploy. It throws when a template with that key exists for another source or type.

**Which tenant.** In a panel with [tenants](#multi-tenancy), `syncTemplate()` keeps the template of the **current tenant**; pass `tenant: $team` for another one. To keep the **shared** template that every tenant uses (no tenant), pass `shared: true` — this works inside a tenant's panel too, for example when a new company signs up:

```php
WarpPrint::syncTemplate('invoice', InvoiceSource::class, $layout, shared: true);   // the template for all tenants
WarpPrint::syncTemplate('invoice', InvoiceSource::class, $layout, tenant: $team);  // this team's own template
```

Passing both `tenant:` and `shared: true` throws an `InvalidArgumentException`. Outside a panel (a seeder, a deploy command) there is no current tenant, so the shared template is kept either way.

The first argument is a template key or a `PrintTemplate`. Records can be models, a collection or keys; they are read again through the source (with its query and `canPrint()`), in the order given. Pass `log: false` to skip the print log and `user:` to print as someone else (for example in a queued job). Options for labels: `['skip' => 2, 'copies' => 3]`.

## Moving templates between environments

Templates live in the database, so the ones an admin designed on staging are not in production yet. Move them as JSON:

- **Export** (in a template's ⋯ menu, or for the selected templates) downloads `warp-print-templates.json`: the published layout (or the draft when never published), the email settings and, for Excel templates, the `.xlsx` file itself.
- **Import templates** (in *More* on the templates page) reads that file. A template whose key already exists gets a new key (`invoice-2`) unless *Replace templates with the same key* is on; it is a draft unless *Publish now* is on (Excel templates are published with their file). The layout is checked like one from the designer, and a template for data that is not registered in the panel is refused.

In a deploy script:

```bash
php artisan warp-print:export invoice delivery-note --output=storage/app/templates.json   # every template when no keys are given
php artisan warp-print:import storage/app/templates.json --overwrite --publish
```

`--tenant-type=team --tenant-id=3` exports or imports the templates of one tenant (without them, the shared templates).

## Approvals with Warp Flow

[Warp Flow](https://warpui.com/flow) adds approval routes to records. Warp Print can print the result — who approved and when — and print only approved records. Nothing extra is installed: these work with any model that uses Warp Flow's `HasApprovals`.

```php
use Qalainau\FilamentWarpPrint\Sources\Concerns\PrintsOnlyApproved;

class PurchaseOrderSource extends PrintSource
{
    use PrintsOnlyApproved; // only approved purchase orders print, mail or export

    public function fields(): array
    {
        return [
            Field::text('number'),
            // ...
            ...Field::approval(),   // approval.status, approval.applicant, approval.approver_1, approval.approved_on_1, ...
        ];
    }
}
```

`Field::approval(path: 'approval', steps: 4)` adds, under *Approval*: the status, whether it is approved and when, the applicant and the submission date, and the approvers of the latest round in the order they approved, each with the date. Put a **date stamp** in each box of an approval column (担当・課長・部長) with *Name* `approval.approver_1` and *Date* `approval.approved_on_1`: each box shows the red stamp once that person has approved, and stays empty before.

`PrintsOnlyApproved` keeps records that are not approved out of every print, PDF, email and background job. To hide the button too, use `PrintAction::make()->visible(fn (PurchaseOrder $record) => $record->approvalStatus() === ApprovalStatus::Approved)`.

Records that need no approval — small orders, or companies without an approval route — can still be printed: override `approvalRequired()`. It returns `true` by default, so every record must be approved; return `false` for the records that print without one. Your policy check (`canPrint()` of `PrintSource`) still applies to them.

```php
public function approvalRequired(Model $record): bool
{
    return $record->approvalStatus() !== null; // never submitted (no route) → prints; submitted → must be approved
    // or: return $record->total >= 100_000;   // only large orders need approval
}
```

## Multi-tenancy

Warp Print works with [Filament's multi-tenancy](https://filamentphp.com/docs/4.x/users/tenancy), so one app can serve many teams or companies (a SaaS). Each tenant gets its own templates, and only its own records can be printed. Panels without tenancy work as before.

### Panel setup

Nothing extra is needed: register the plugin in a panel that has `->tenant()`.

```php
use App\Models\Team;
use Qalainau\FilamentWarpPrint\WarpPrintPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        ->id('app')
        ->path('app')
        ->tenant(Team::class, slugAttribute: 'slug')   // URLs like /app/acme/print-templates
        ->plugin(
            WarpPrintPlugin::make()
                ->sources([InvoiceSource::class, CustomerSource::class])
                // optional: who may create and change templates in this tenant
                ->canManageTemplates(fn (User $user, ?Team $tenant) => $user->isAdminOf($tenant)),
        );
}
```

The models you print need a relationship to the tenant, the same one Filament uses for your resources (by default the tenant model's name in camelCase, here `team()`):

```php
class Invoice extends Model
{
    public function team(): BelongsTo
    {
        return $this->belongsTo(Team::class);
    }
}
```

### What is kept per tenant

| | In a tenant |
| --- | --- |
| **Templates** | A template made in a tenant belongs to that tenant. Other tenants do not see it in the list, in the print dialog or by its URL (404). |
| **Shared templates** | A template without a tenant (for example one you seed) is offered to every tenant. Inside a tenant it cannot be edited or deleted (403), because that would change it for everyone. **Duplicate** makes the tenant's own copy with the same key, which is then used instead of the shared one. |
| **Template keys** | A key must be unique within a tenant. Two tenants may use the same key. `WarpPrint::pdf('invoice', ...)` uses the tenant's `invoice` template if there is one, else the shared one. |
| **Records** | Only the current tenant's records are printed, shown as designer samples or filled into Excel, even if a record key of another tenant is sent from the browser. |
| **Print log** | Each print records the tenant (`tenant_type`, `tenant_id`). |

### Columns and migrations

Warp Print's own tables already have `tenant_type` and `tenant_id` (text, so integer, UUID and ULID keys all work); there is nothing to add. Your printed models need the tenant column of their relationship, for example `$table->foreignId('team_id')->constrained();` on `invoices`.

### Models without a tenant, and other relationship names

A source is scoped to the tenant by default. Change it on the source:

```php
class ProductSource extends PrintSource
{
    protected static string $model = Product::class;

    // products are shared by all tenants: do not scope them
    protected static bool $isScopedToTenant = false;
}

class CustomerSource extends PrintSource
{
    protected static string $model = Customer::class;

    // the relationship to the tenant has another name than the panel's
    protected static ?string $tenantOwnershipRelationshipName = 'teams';
}
```

The relationship may be a `BelongsTo`, a `MorphTo`, or a many-to-many or "through" relationship (for example a customer that belongs to several teams through `customer_team`). For the whole panel, set the name with `->tenant(Team::class, ownershipRelationship: 'company')`. A source whose model *is* the tenant prints only the current tenant.

### Notes

- If a scoped source's model has no relationship to the tenant, printing stops with an error that names the source, instead of printing records of every tenant. Add the relationship, or set `$isScopedToTenant = false`.
- Outside a panel request (a queued job, a command, a mail), there is no current tenant, so only shared templates are found and records are not scoped. Set the tenant first: `Filament::setTenant($team, isQuiet: true);`, then call `WarpPrint::pdf(...)`.
- Your own `query()` and `canPrint()` still apply on top of the tenant scope.
- Uploaded Excel files are stored on the `disk` from the config. A tenant's duplicate of a shared Excel template uses the same file until the tenant uploads its own; Warp Print never deletes the shared file.

## Remote images and SSRF

Images in a print can come from three places: an image uploaded in the designer, a file on your disk (`Field::image()` with a path), or a URL on another website. **URLs are turned off by default.** This section explains why, and how to turn them on safely.

### Why URLs are off by default

To make a PDF on the server, Warp Print opens the page in Chrome on your server (Browsershot or Gotenberg). If that page had an image like `http://169.254.169.254/latest/meta-data/` or `http://localhost:8080/admin`, **your server** would load it, from inside your network. This is called *server-side request forgery* (SSRF). In a SaaS, anyone who can design templates in their tenant, or type an image URL into a record, could use it to read your cloud credentials or internal services.

So Warp Print never gives a URL to the PDF engine. It downloads allowed images itself, checks them, and puts them into the page as data (`data:image/png;base64,...`). The same page is used for the print window, the preview and server PDFs.

### Allowing remote images

Allow every host, a list of hosts, or decide per tenant or user, on the panel:

```php
use Qalainau\FilamentWarpPrint\WarpPrintPlugin;

// every host on the internet (internal addresses are still refused)
WarpPrintPlugin::make()->remoteImages();

// only these hosts; "*." allows the subdomains (img.example.org), not example.org itself
WarpPrintPlugin::make()->remoteImages(['cdn.example.com', '*.example.org']);

// per tenant or user: return false, true or a list of hosts
WarpPrintPlugin::make()->remoteImages(fn (?Team $tenant, ?User $user) => $tenant?->plan === 'business'
    ? ['cdn.example.com', $tenant->asset_host]
    : false);
```

Without a panel setting, the config is used, which also covers queued jobs and code outside a panel (`WarpPrint::pdf()` there uses the default panel's setting, then the config):

```php
// config/filament-warp-print.php
'remote_images' => [
    'allow' => env('WARP_PRINT_REMOTE_IMAGES', false), // false, true, or ['cdn.example.com', '*.example.org']
    'max_bytes' => 2 * 1024 * 1024,
    'timeout' => 5,
    'max_redirects' => 3,
],
```

In `.env`, list hosts separated by commas: `WARP_PRINT_REMOTE_IMAGES=cdn.example.com,*.example.org`, or `true` for every host.

### What is always refused

Even when every host is allowed, an image URL is refused when:

- **The scheme is not `http` or `https`** (`file:`, `gopher:`, `ftp:`, `javascript:` and others), or the URL has a user name or password.
- **The host points to an internal address.** Warp Print looks up the host itself and refuses it if *any* of its addresses is: loopback (`127.0.0.0/8`, `::1`), private (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `fc00::/7`), link-local and cloud metadata (`169.254.0.0/16` such as `169.254.169.254`, `fe80::/10`, `fd00:ec2::254`), carrier-grade NAT (`100.64.0.0/10`), "this network" (`0.0.0.0/8`, `::`), multicast and reserved ranges, documentation ranges, and IPv6 forms that carry an IPv4 address (`::ffff:0:0/96`, NAT64 `64:ff9b::/96`, 6to4 `2002::/16`, Teredo). Writing the address as a number (`http://2130706433/`) or in brackets (`http://[::1]/`) makes no difference.
- **It redirects to such an address.** Redirects are followed by Warp Print, at most `max_redirects` times, and every new URL is checked again from the start.
- **It is too big, too slow or not an image.** At most `max_bytes` (2 MB) and `timeout` seconds (5) for connecting and for the whole download; the download stops as soon as it is over the size. The content must really be PNG, JPEG, GIF or WebP (the first bytes are checked, not the `Content-Type`). SVG files from URLs are refused, because an SVG can point to other URLs.

Warp Print connects to **the address it checked**, not to a new lookup of the name, so a host that changes its DNS answer between the check and the download (*DNS rebinding*) cannot reach inside. It does not use an HTTP proxy for these downloads. The server needs PHP's curl extension for this; without it, remote images are refused.

### What happens when an image is refused

- **In the designer**, the *image URL* box shows why the URL cannot be used (for example *"Images from cdn.other.com are not allowed."* or *"169.254.169.254 points to a private or internal address, which is never allowed."*), and a template that still has a refused image URL is not saved or published: the toolbar says which image to replace. The box is only shown when remote images are allowed for the current tenant and user.
- **When printing**, the image is left empty and everything else prints. A print, a PDF download or a mail with `WarpPrint::pdf()` does not fail because a logo URL is gone or no longer allowed. The reason is written to the log as a warning (`Warp Print left out a remote image.`).

### Uploaded images, data URIs and files on your disk

- **Images uploaded in the designer** are stored in the template as PNG, JPEG, GIF or WebP data, up to 2 MB. They need no network.
- **`data:` URIs** from `Field::image()` are passed on only when they are images (`data:image/png`, `jpeg`, `gif`, `webp` or `svg+xml`). An SVG drawn as an image cannot load other URLs. Other `data:` values (such as `data:text/html`) are dropped.
- **Paths** from `Field::image()` are read from `image_disk` (or `->disk()`) and embedded. Paths with `..`, files outside the disk and files that are not images are refused. If no disk is set, a path is not used.

If an image path comes from what users type or upload (for example a customer's logo), remember that **all tenants share the same disk**: a user could type the path of another tenant's file. Store each tenant's files in its own folder and limit the field to it:

```php
Field::image('customer.logo_path')
    ->disk('public')
    ->directory(fn (Invoice $record) => "teams/{$record->team_id}"), // other paths print no image
```

Also make your upload fields store files in that folder (`FileUpload::make('logo_path')->directory(fn () => 'teams/'.Filament::getTenant()->id)`).

### PDF engines

- **Browsershot:** Warp Print starts Chrome with `--host-resolver-rules`, so it can only look up the hosts of `font_stylesheets` (and `fonts.gstatic.com` for Google Fonts). Add other hosts your page needs in `pdf.browsershot.allow_hosts`. This is a second line of defence: the page already contains no image URLs.
- **Gotenberg:** Gotenberg runs in its own container, so limit what its Chrome may open when you start it, for example `gotenberg --chromium-allow-list='^(file:///tmp/|https://fonts\.(googleapis|gstatic)\.com/)'`, and do not give the container access to internal services it does not need. Excel templates are converted by LibreOffice and contain only the images inside the file.
- **The print window** (printing from the browser) is drawn by the user's own browser, from the same embedded page.

## Fonts

Templates choose between the fonts in `config('filament-warp-print.fonts')`: *Sans-serif*, *Serif* and *Monospace* by default, with Noto Sans JP / Noto Serif JP first so Japanese text looks the same everywhere. `font_stylesheets` loads them from Google Fonts. Add your own:

```php
'fonts' => [
    // ...
    'brand' => ['label' => 'Brand', 'family' => '"Inter", sans-serif'],
],
'font_stylesheets' => [
    'https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=block',
],
```

For server PDFs the server must reach those URLs, or have the fonts installed (on Debian/Ubuntu: `apt install fonts-noto-cjk`).

## Configuration

```bash
php artisan vendor:publish --tag=filament-warp-print-config
```

| Key | Default | |
| --- | --- | --- |
| `disk` | `local` | Where uploaded Excel templates are stored |
| `image_disk` | `public` | Where `Field::image()` paths are read from |
| `remote_images.allow` | `false` | Image URLs: `false`, `true` or a list of hosts. See [Remote images and SSRF](#remote-images-and-ssrf) |
| `remote_images.max_bytes`, `.timeout`, `.max_redirects` | 2 MB, 5 s, 3 | Limits for one remote image |
| `pdf.browsershot.allow_hosts` | `[]` | Hosts the Browsershot Chrome may look up, besides the font stylesheets |
| `currency` | `USD` | Default currency of `Field::money()` |
| `pdf.driver` | `null` | `browsershot`, `gotenberg` or none |
| `fonts`, `font_stylesheets` | Noto Sans / Serif JP | See [Fonts](#fonts) |
| `keep_logs_days` | 365 | How long print logs are meant to be kept |
| `batch_limit` | 200 | Most records drawn in the print window; more are made in the background (`WARP_PRINT_BATCH_LIMIT`) |
| `exports.disk`, `.directory` | `local`, `warp-print/exports` | Where background files are stored |
| `exports.limit` | 5000 | Most records in one background job |
| `exports.chunk` | 100 | Records per PDF part in a background job |
| `exports.keep_hours` | 72 | How long background files can be downloaded |
| `exports.connection`, `.queue` | `null` | The queue for background jobs (`WARP_PRINT_QUEUE_CONNECTION`, `WARP_PRINT_QUEUE`) |
| `mail.mailer` | `null` | The mailer for `EmailPrintAction` (`WARP_PRINT_MAILER`); the app's default when `null` |
| `archive.enabled` | `false` | Keep issued PDFs (`WARP_PRINT_ARCHIVE`). See [Issued documents](#issued-documents-electronic-bookkeeping) |
| `archive.disk`, `.directory` | `local`, `warp-print/documents` | Where issued PDFs are stored |
| `archive.keep_years` | `null` | Years to keep issued PDFs (`null`: forever) |
| `forms.node` | `node` | Node.js used to make filled PDF forms on the server (`WARP_PRINT_FORMS_NODE`); `null` turns it off |
| `forms.fonts` | Noto Sans / Serif JP | TrueType files embedded in filled PDF forms; installed by `warp-print:fonts` |
| `forms.disk`, `.directory` | `local`, `warp-print/entries` | Where the PDF of a submitted form is kept |
| `forms.max_upload` | 20 MB | Largest form PDF that can be uploaded |
| `forms.timeout` | 60 | Seconds to wait for node |
| `forms.link.days`, `.per_minute` | 14, 30 | How long a form link works by default; saves per minute per link and IP |

`WarpPrintPlugin` options:

```php
WarpPrintPlugin::make()
    ->sources([InvoiceSource::class, CustomerSource::class])
    ->canManageTemplates(fn ($user) => $user->is_admin) // who may design templates (default: everyone in the panel)
    ->canUseEntries(fn ($user) => $user->can('fill-forms')) // who sees Filled forms (default: everyone in the panel)
    ->entries(false)                                     // no Filled forms page (no PDF and paper forms in this panel)
    ->templates(false)                                   // hide the templates page in this panel (print buttons still work)
    ->canViewArchive(fn ($user) => $user->is_accountant) // who may open Issued documents (default: everyone in the panel)
    ->archive(false)                                     // hide Issued documents in this panel (documents are still kept)
    ->navigationGroup('Settings')
    ->navigationLabel('Forms')
    ->navigationIcon('heroicon-o-document-text')
    ->navigationSort(90);
```

## Security

- **Templates show only what you list.** The page receives only the fields of the source — never the model's other attributes — and template text is shown as text, never as HTML. Templates cannot run code.
- **Records are read again on the server** through the source's `scopedQuery()` (your `query()` plus the current tenant) and `canPrint()` for every preview, print and download. The print window keeps the source, the records and the offered templates in locked Livewire properties, so they cannot be changed from the browser.
- **Only registered sources** can be chosen for a template, and only classes extending `PrintSource` are ever created from the database.
- **Layouts from the designer are checked** before they are saved: unknown elements and settings are dropped, numbers are kept in range, images must be PNG, JPEG, GIF or WebP data (2 MB at most) or an allowed http(s) URL, and a template has at most 500 elements.
- **No remote images by default.** The server never fetches a URL from a template or a record unless you allow it, and even then never from an internal address. Images are embedded in the page, so the PDF engine opens no image URLs. See [Remote images and SSRF](#remote-images-and-ssrf).
- **Formulas** in calculated fields are read by a small parser that knows only fields, numbers, quoted text, arithmetic, comparisons and a fixed list of functions; nothing is evaluated as code. A formula is at most 500 characters.
- **Background downloads** use a signed, expiring URL and are handed out only to the user who asked, while logged in. **Issued documents** cannot be changed or deleted from the app, and their hash is checked before they are downloaded again.
- **Imported templates** go through the same checks as the designer's; only registered sources are accepted and an Excel file inside must be an `.xlsx` (ZIP) file of at most 10 MB.
- **Excel files** are read without external entities and only the cells' XML is changed; macros (`.xlsm`) are not accepted.
- **Word files** are read the same way: no external entities are loaded, values are written as text nodes (a value cannot add XML to the document), a value is never read again for `{{fields}}`, and macro files (`.docm`) are not accepted.
- **Form links** are 48 random characters, stored only as a SHA-256 hash, expire after the days chosen and close when the form is submitted; the page is rate limited per link and IP, and marked `noindex`. The form PDF behind a link and the panel's form PDFs are served through signed, expiring URLs.
- **Filled-in values are checked on the server** (types, lengths, formats, options, image size) whatever the browser sent. Fields marked *Cannot be changed* keep the record's value. Only fields your source marks `->writable()` are ever saved to a record, and links never save to a record without someone in the panel pressing *Save to record*. PDFs are always made again from the values; a PDF sent from a browser is never accepted.
- **Who designs** is up to `canManageTemplates()`; who prints is up to where you put the buttons and their `->visible()` / `->authorize()`.
- **Your policies apply to the designer and to filling in.** If the template's policy refuses `update`, the designer does not open (403), and saving, publishing or taking a new form PDF answer 403 even when called from the browser; the table hides *Design*. If a filled form's policy refuses `view`, it does not open; if it refuses `update`, the form opens read-only, *Save draft* and *Submit* are not shown, saving answers 403, and *Save to record*, *Reopen* and *Renew link* are hidden. Without policies nothing changes.


### Restricting the designer and filling in per role

```php
use Qalainau\FilamentWarpPrint\Resources\PrintEntries\Pages\FillPrintEntry;

WarpPrintPlugin::make()
    ->canAccessUsing(fn (User $user, string $page): bool => $user->canOpenScreen($page))
    ->canWriteUsing(fn (User $user, string $page): bool => ! $user->isReadOnlyOn($page))
```

`$page` is `DesignPrintTemplate` (the template designer) or `FillPrintEntry` (filling in a form). `canAccessUsing()` returning `false` gives a 403 at the page and refuses its Livewire calls. The designer is only for changing a template, so `canWriteUsing()` returning `false` closes it too; filling in opens read only (no *Save draft* or *Submit*, and *Write back*, *Reopen* and *New link* are hidden). These rules only narrow the plugin's options and your policies.

The lists of templates, entries and issued documents also ask your policy's `viewAny` when you have one for the model, together with `canManageTemplates()`, `canUseEntries()` and `canViewArchive()`. Without a policy nothing changes (also with Filament's `strictAuthorization()`).

## Translations

The panel pages, the designer and the print window are translated into ten languages: English, Japanese, German, French, Spanish, Italian, Brazilian Portuguese (`pt_BR`), Korean, and simplified (`zh_CN`) and traditional (`zh_TW`) Chinese. They follow the app's locale. Publish the translations to change them or add languages:

```bash
php artisan vendor:publish --tag=filament-warp-print-translations
```

The text on the printed pages is whatever your admins write in the templates.

## Limitations

- One table per template flows over pages; other tables show the rows that fit in their box.
- Excel templates: `.xls` (Excel 97–2003) and macros are not supported; images from image fields are not placed in Excel; each item uses one row (a two-row item layout is not repeated as a block); cells that hold a field must contain the whole `{{…}}` in one piece of text (re-type it if Excel split it into differently formatted parts).
- The browser's print dialog may add its own headers and footers (date, URL): turn them off in the dialog. Chrome, Edge and Firefox use the template's paper size; Safari may need the paper chosen in the dialog.
- One photo element per template continues onto added pages, with up to 200 photos and 50 added pages per record.
- One text block per template continues onto added pages; a single paragraph taller than the box is cut at the bottom (split long text into paragraphs).
- Calculated fields are drawn on the page, so they are not in Excel forms or email subjects.
- Watermarks are drawn on documents, not on label sheets.
- Background jobs need a queue worker and database notifications; without notifications the file is made but nobody is told.
- PDF and paper forms: handwriting on a scanned, filled-in form is not read (no OCR). Box finding works on clear scans with ruled lines; check its suggestions. XFA forms (some government PDFs made with Adobe LiveCycle) and password-protected PDFs cannot be used; PDF signature fields and buttons are not read in. Fonts are embedded per family, so a field shows the characters its font has (add a Korean font for Korean).

## Troubleshooting

**The print button opens a window that says there is no template.** Only **published** templates are offered: open the template and press Publish. The template's data must be the source of the record's model.

**The preview stays blank.** Run `php artisan filament:assets` after installing or updating, and check the browser console for errors.

**Japanese text prints in the wrong font, or as boxes, in server PDFs.** Install the fonts on the server (`fonts-noto-cjk`) or make sure it can reach Google Fonts.

**"Warp Print cannot make PDF files on the server".** Set `WARP_PRINT_PDF_DRIVER` (see [PDF files on the server](#pdf-files-on-the-server)). Printing from the print window works without it.

**Filled forms show "?" instead of Japanese, Chinese or Korean characters.** The PDF has no font for them yet: run `php artisan warp-print:fonts` (Korean needs a Korean font in `forms.fonts`).

**"This server cannot make these PDFs itself".** PHP did not find Node.js 18+. Run `php artisan warp-print:check-forms`; if it says node was not found, set `WARP_PRINT_FORMS_NODE` to the full path (`which node`). Downloading and printing filled forms in the browser work without it.

**"Make in the background" never finishes.** Start a queue worker (`php artisan queue:work`) and make sure the panel has `->databaseNotifications()`. A failed job sends a notification with the reason, and `failed_jobs` has the details.

**The Email button is missing.** It is shown when it can attach something: set a PDF driver, or use an Excel template. The template must be published.

**A starter shows "Fields that are not in the data".** Select the marked elements and choose the right fields, or map them once in `starterFields()`.

**An Excel field stays as `{{…}}`.** The cell probably has the field split into parts with different formatting; type it again in one go. The settings page lists every field it found.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## Support

Ask a question or report a bug at https://warpui.com/support?plugin=print. Do not post your license key in your question.

## License

Warp Print is commercial software. See [LICENSE.md](LICENSE.md) for the license terms.
