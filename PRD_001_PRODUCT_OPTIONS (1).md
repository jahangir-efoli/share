# PRD-001: Product Options (Custom Options Beyond Shopify's 3-Option Limit)

**Version:** 2.0 (Updated from FRS document)
**Source:** MultiVariants FRS (Product Option).pdf
**Branch:** claude/multivariants-shopify-app-IDStM

---

## 1. Overview

**Feature Name:** Product Options
**Priority:** P1 High
**Plan Tier:** Standard + Professional (hidden/locked for Starter)
**Estimated Scope:** Large (2-3 weeks)

### 1.1 Problem Statement

Shopify limits products to 3 variant options. Merchants need unlimited custom fields on the product page — engraving text, file uploads (logos, artwork), date pickers, color/image selections — that Shopify's native system cannot support. Currently they need a separate app adding cost and complexity.

### 1.2 Goal

Allow merchants to add unlimited custom product options within a MultiVariants ruleset. Options are rendered on the storefront, validated before add-to-cart, passed as Shopify cart line item properties, and visible in the order.

### 1.3 Success Metrics

- 30%+ of Standard/Professional merchants use at least 1 custom option within 60 days
- Reduction in merchant support requests for "custom fields"
- Increase in Professional plan upgrades

---

## 2. User Stories

- As a **merchant**, I want to add text inputs so customers can enter personalization (engraving, name, gift message).
- As a **merchant**, I want to add dropdown/radio/checkbox/swatch selectors beyond Shopify's 3-option limit.
- As a **merchant**, I want customers to upload files (logo, artwork) with their order.
- As a **merchant**, I want to add extra charges for specific option choices.
- As a **merchant**, I want to control whether the MultiVariants table is shown alongside my custom options.
- As a **merchant**, I want to reorder options via drag-and-drop so they display in the right sequence.
- As a **customer**, I want clearly labeled options on the product page that I fill in before adding to cart.
- As a **customer**, I want to see a price adjustment displayed when I select a premium option.

---

## 3. Functional Requirements

### 3.1 Admin Panel — Product Options Tab

#### 3.1.1 Tab Location & Layout

- **Location:** On the Create/Edit Ruleset page, new tab alongside existing "Advanced restrictions" tab
- **Tab label:** "Product Options"
- **Header:** "Product Options" (bold/600 weight)
- **Subtitle:** "Add unlimited custom options beyond Shopify's 3-option limit, giving your customers more choices and flexibility on the product page." (gray, smaller font)

**Empty State:**
- Dashed-border container, centered
- Button: `+ Add option`
- Text below button: "Choose the product option you want to set for your products."

**Populated State:**
- Vertical list of configured option cards (each collapsible)
- Drag handle on the left of each card for reordering
- At the bottom: `+ Add more options` button

#### 3.1.2 "Add Product Option" Modal

**Trigger:** Click `+ Add option` or `+ Add more options`

**Modal Header:** "Add Product Option" with close (X) icon top-right

**Selection List:** Vertical list of 10 option types. Each row has:
- Radio button (left) — **single selection only** (selecting new auto-deselects previous)
- Option type name + short description
- Eye icon (right) — preview

**Footer:**
- `Cancel` — closes modal, no changes
- `Add Now` (primary) — **disabled until a radio button is selected**; on click, transitions to the configuration panel for that option type

**Available option types:**

| # | Type | Customer Input |
|---|------|---------------|
| 1 | Number | Numeric input |
| 2 | Text | Short or long text input |
| 3 | Dropdown | Select from list |
| 4 | Radio | Single choice radio buttons |
| 5 | Checkbox | Multiple choice checkboxes |
| 6 | Button Swatch | Clickable text buttons |
| 7 | Color Swatch | Clickable color circles |
| 8 | Image Swatch | Clickable image thumbnails |
| 9 | File Attachment | File upload (drag-drop or click) |
| 10 | Date & Time | Date picker + time picker |

#### 3.1.3 Common Option Card Structure

Each configured option appears as a collapsible card with:

**Card header bar:**
- Drag handle icon (left) — for reordering
- Option type name + description
- Gear icon (⚙) — opens "Option Customization" styling popup (see §3.1.14)
- Delete icon (trash) — removes option (with confirmation)
- Collapse/Expand chevron (right)

**Common fields on ALL option types:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Option Title | Text input | Yes | "" | Max 100 chars. Displays on storefront. May show price: "Title (+$X.XX)" |
| Short Description | Text/HTML input | No | "" | Shown below title on storefront. Supports basic HTML |
| Help Text | Text input | No | "" | Shown below input field on storefront |
| Required | Dropdown (Yes/No) | Yes | No | If Yes, Add to Cart blocked until filled |

---

#### 3.1.4 Number Option — Configuration Fields

Accessed when merchant selects "Number" from modal.

| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Option Title | Text | Yes | "" | Max 100 chars |
| Short Description | Text/HTML | No | "" | - |
| Placeholder | Text | No | "Write your number" | Max 100 chars |
| Help Text | Text | No | "" | - |
| Minimum Value | Number | No | null | Integer or decimal |
| Maximum Value | Number | No | null | Must be >= min if both set |
| Price | Number | No | 0 | Applied when field is filled. Displayed as "(+$X.XX)" next to title |
| Required | Dropdown (Yes/No) | Yes | No | - |

**Storefront display example:**
```
Number Editions (+$8.00)
[This classic piece is a sustainable staple for any wardrobe.]
[Write your number          ]  ← input field
[Help text here]
```

**Gear icon → Option Customization popup fields:**
- Input Field Background Color
- Input Field Border Color
- Placeholder Typography Color
- *(Live preview panel on right side)*

---

#### 3.1.5 Text Option — Configuration Fields

Accessed when merchant selects "Text" from modal.

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Input Type | Radio (Short text / Long text) | Yes | Short text | Short = single-line `<input>`. Long = multi-line `<textarea>` |
| Option Title | Text | Yes | "" | Max 100 chars |
| Short Description | Text/HTML | No | "" | - |
| Placeholder | Text | No | "Write your text" | Max 100 chars |
| Help Text | Text | No | "" | - |
| Maximum Character | Number | No | null | Input blocked at this limit; validation error if exceeded |
| Price | — | — | — | See pricing type below |
| Required | Dropdown (Yes/No) | Yes | No | - |

**Text Pricing (3 types — merchant selects one):**

| Pricing Type | Behavior | Example |
|-------------|----------|---------|
| **Single word** | Price × number of words entered | "Hello World" = 2 words × $2 = $4 |
| **Each letter** | Price × number of characters entered | "abc" = 3 chars × $2 = $6 |
| **Overall text** | Fixed price regardless of length | Any text = $15 flat |

- Price value field: decimal number (e.g., 2.00)
- Displayed as "(+$X.XX)" next to option title on storefront
- Price updates dynamically as customer types

**Gear icon → Option Customization popup fields:**
- Input Field Background Color
- Input Field Border Color
- Placeholder Typography Color
- *(Live preview: shows label + styled input)*

---

#### 3.1.6 Dropdown Option — Configuration Fields

Accessed when merchant selects "Dropdown" from modal.

**Header fields:**

| Field | Type | Required | Default |
|-------|------|----------|---------|
| Option Title | Text | Yes | "" |
| Short Description | Text/HTML | No | "" |
| Help Text | Text | No | "" |
| Required | Dropdown (Yes/No) | Yes | No |

**Attributes table** (each row = one dropdown item):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder rows |
| Dropdown Title | Text input | Label of the item (e.g., "Small", "Medium") |
| Image | File upload | Optional image per item. Uploaded to Shopify Files API → CDN URL stored |
| Price | Number | Additional price when this item is selected |
| Default | Checkbox | Pre-selects this item. **Only one row can be default** |
| Delete | Trash icon | Removes this row |

- `+ Add Attribute` button adds new row
- Minimum 1 row required to save
- Rows can be dragged to reorder (storefront reflects same order)

**Storefront display:**
```
Dropdown (+$5.00)
[Description text]
[Select an option ▾]   ← dropdown selector
[Help text]
```

**Gear icon → Option Customization popup fields:**
- Input Field Background Color
- Input Field Border Color
- Placeholder Typography Color
- *(Live preview: styled dropdown)*

---

#### 3.1.7 Radio Option — Configuration Fields

Accessed when merchant selects "Radio" from modal.

**Header fields:** Same as Dropdown (Title, Short Description, Help Text, Required)

**Attributes table** (each row = one radio choice):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder rows |
| Radio Title | Text input | Label (e.g., "Choice 01") |
| Image | File upload | Optional image per choice. CDN URL stored via Shopify Files API |
| Price | Number | Additional price when selected |
| Default | Checkbox | Pre-selects this choice. **Only one allowed** |
| Delete | Trash icon | Removes row |

- `+ Add Attribute` button
- Minimum 1 row required

**Gear icon → Option Customization popup fields (8 fields):**
- Default radio background color
- Default radio border color
- Active radio background color
- Active radio border color
- Image box size
- Image box border radius
- Image box border color
- Typography color
- *(Live preview: radio buttons with selected state + optional images)*

---

#### 3.1.8 Checkbox Option — Configuration Fields

Accessed when merchant selects "Checkbox" from modal.

**Header fields:** Same as Dropdown (Title, Short Description, Help Text, Required)

**Required behavior for Checkbox:** If Yes → customer must select **at least one** checkbox.

**Attributes table** (each row = one checkbox item):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder rows |
| Checkbox Title | Text input | Label (e.g., "Choice 01") |
| Image | File upload | Optional image per item. CDN URL via Shopify Files API |
| Price | Number | Price added when this item is checked |
| Default | Checkbox | Pre-checks this item. **Multiple defaults allowed** |
| Delete | Trash icon | Removes row |

- `+ Add Attribute` button
- Minimum 1 row required
- Total price = sum of all checked items' prices

**Gear icon → Option Customization popup fields (8 fields):**
- Default checkbox background color
- Default checkbox border color
- Active checkbox background color
- Active checkbox border color
- Image box size
- Image box border radius
- Image box border color
- Typography color

---

#### 3.1.9 Button Swatch Option — Configuration Fields

Accessed when merchant selects "Button Swatch" from modal. Renders as clickable text buttons.

**Header fields:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Swatch Style | Radio (Single choice / Multiple choice) | Yes | Single choice | Single = one active at a time. Multiple = toggle on/off |
| Option Title | Text | Yes | "" | - |
| Short Description | Text/HTML | No | "" | - |
| Help Text | Text | No | "" | - |
| Required | Dropdown (Yes/No) | Yes | No | If multiple: at least one required |

**Attributes table** (each row = one swatch button):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder |
| Swatch Title | Text input | Text shown inside button (e.g., "S", "M", "L") |
| Price | Number | Price when this swatch is selected |
| Default | Checkbox | Single mode: only one. Multiple mode: many allowed |
| Delete | Trash icon | Removes row |

**Note:** Button Swatch has NO image field per attribute (text-only buttons).

**Gear icon → Option Customization popup fields (10 fields):**
- Default swatch background color
- Default swatch border color
- Active swatch background color
- Active swatch border color
- Swatch border radius
- Swatch top & bottom padding
- Swatch right & left padding
- Default swatch title text color
- Active swatch title text color

---

#### 3.1.10 Color Swatch Option — Configuration Fields

Accessed when merchant selects "Color Swatch" from modal. Renders as clickable color circles/boxes.

**Header fields:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Swatch Style | Radio (Single choice / Multiple choice) | Yes | Single choice | - |
| Option Title | Text | Yes | "" | - |
| Short Description | Text/HTML | No | "" | - |
| Help Text | Text | No | "" | - |
| Required | Dropdown (Yes/No) | Yes | No | - |

**Attributes table** (each row = one color):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder |
| Swatch Title | Text input | Label (e.g., "Red", "Green") |
| Color Picker | HEX color input | Color displayed as visual swatch (e.g., #F24F4F) |
| Price | Number | Price when this color is selected |
| Default | Checkbox | Single: one only. Multiple: many allowed |
| Delete | Trash icon | Removes row |

**Gear icon → Option Customization popup fields (12 fields):**
- Default swatch background color
- Default swatch border color
- Active swatch background color
- Active swatch border color
- Swatch color box radius
- Swatch border radius
- Swatch top & bottom padding
- Swatch right & left padding
- Show swatch label (toggle: on/off)
- Label position (e.g., Right)
- Default swatch title text color
- Active swatch title text color

---

#### 3.1.11 Image Swatch Option — Configuration Fields

Accessed when merchant selects "Image Swatch" from modal. Renders as clickable image thumbnails.

**Header fields:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Swatch Style | Radio (Single choice / Multiple choice) | Yes | Single choice | - |
| Option Title | Text | Yes | "" | - |
| Short Description | Text/HTML | No | "" | - |
| Help Text | Text | No | "" | - |
| Required | Dropdown (Yes/No) | Yes | No | - |

**Attributes table** (each row = one image option):

| Column | Type | Notes |
|--------|------|-------|
| Drag handle | Icon | Reorder |
| Swatch Title | Text input | Label (e.g., "Trending", "Professional") |
| Upload Image | File upload (required) | Image displayed as clickable thumbnail. Uploaded via Shopify Files API → CDN URL stored in JSON |
| Price | Number | Price when this option is selected |
| Default | Checkbox | Single: one only. Multiple: many allowed |
| Delete | Trash icon | Removes row |

**Admin upload flow for swatch images:** Merchant selects file → JS POSTs to `/gate/out/adm/action/upload_option_image.php` → PHP calls `stagedUploadsCreate` → uploads to GCS → calls `fileCreate` → returns CDN URL → stored in JSON attribute `imageUrl`.

**Gear icon → Option Customization popup fields (12 fields):**
- Default swatch background color
- Default swatch border color
- Active swatch background color
- Active swatch border color
- Swatch image box radius
- Swatch border radius
- Swatch top & bottom padding
- Swatch right & left padding
- Show swatch label (toggle: on/off)
- Label position (e.g., Right)
- Default swatch title text color
- Active swatch title text color

---

#### 3.1.12 File Attachment Option — Configuration Fields

Accessed when merchant selects "File Attachment" from modal. Customers upload files on the product page.

**Configuration fields:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Option Title | Text | Yes | "" | E.g., "Upload Your Logo" |
| Short Description | Text/HTML | No | "" | E.g., "Upload your logo for printing" |
| Help Text | Text | No | "" | E.g., "Accepted: JPEG, PNG. Max 50MB." |
| Price | Number | No | 0 | Applied when customer uploads a file |
| Max Upload Filesize | Number (MB) | No | 10 | Range: 1–50 MB. File exceeding limit is rejected |
| Required | Dropdown (Yes/No) | Yes | No | If Yes, Add to Cart blocked until file uploaded |
| Show for Each Variant | Checkbox | No | unchecked | If checked: upload field appears inside each variant row in the MultiVariants table (per-variant upload). If unchecked: single upload at product level |

**File Types (multi-select checkboxes):**

Merchant selects which file formats are allowed. Options:

| Type | MIME Type | Shopify Resource | Max Size |
|------|-----------|-----------------|---------|
| JPEG | `image/jpeg` | `IMAGE` | 20 MB |
| PNG | `image/png` | `IMAGE` | 20 MB |
| WEBP | `image/webp` | `IMAGE` | 20 MB |
| SVG | `image/svg+xml` | `IMAGE` | 20 MB |
| HEIC | `image/heic` | `IMAGE` | 20 MB |
| GIF | `image/gif` | `IMAGE` | 20 MB |
| TIFF | `image/tiff` | `IMAGE` | 20 MB |
| PSD | `application/octet-stream` | `FILE` ⚠️ | ~2 MB (bug) |
| GLB | `model/gltf-binary` | `MODEL_3D` | 500 MB |
| USDZ | `model/vnd.usdz+zip` | `MODEL_3D` | 500 MB |
| MP4 | `video/mp4` | `VIDEO` | 1 GB |
| MOV | `video/quicktime` | `VIDEO` | 1 GB |
| WEBM | `video/webm` | `VIDEO` | 1 GB |

> ⚠️ **PSD limitation:** Shopify's `FILE` resource has a known platform bug where files >~2MB fail at the Google Cloud Storage upload step (HTTP 400 "Metadata part is too large"). PSD files over 2MB should display a user-friendly error: *"PSD files must be under 2MB. Please save as PNG or JPEG instead."*

> **File Storage Implementation Note:** Customer uploads are proxied through the app server (PHP) which uses the shop's admin token to upload to Shopify Files API. See §3.4 for the upload endpoint and §5.3 for the full technical implementation.

**Gear icon → Option Customization popup fields (7 fields):**
- File attachment box background color
- File attachment box border color
- Upload file button background color
- Upload file button typography color
- File attachment box border radius
- File attachment box top & bottom padding
- File attachment box right & left padding

---

#### 3.1.13 Date & Time Option — Configuration Fields

Accessed when merchant selects "Date & Time" from modal.

**Configuration fields:**

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| Option Title | Text | Yes | "" | E.g., "Delivery Date" |
| Short Description | Text/HTML | No | "" | - |
| Help Text | Text | No | "" | - |
| Required | Dropdown (Yes/No) | Yes | No | If Yes, enabled fields must be filled |
| Show Date | Checkbox | No | checked | If enabled, date picker shown on storefront |
| Show Time | Checkbox | No | unchecked | If enabled, time picker shown on storefront |
| Price | Number | No | 0 | Fixed price applied when date/time is selected |

**Availability (day selector):**

Merchant selects which days are available. 7 checkboxes:

| Day | Default |
|-----|---------|
| Saturday | unchecked |
| Sunday | unchecked |
| Monday | checked |
| Tuesday | checked |
| Wednesday | checked |
| Thursday | checked |
| Friday | checked |

- Only selected days are selectable in the date picker on storefront
- Unselected days are visually disabled/grayed out

**No gear icon for Date & Time** (no visual customization in FRS).

---

#### 3.1.14 Option Customization Popup (Gear Icon)

Each option type (except Date & Time) has a gear icon that opens a modal with styling controls.

**Popup structure:**
- Title: "Option Customization"
- Left panel: styling fields (color pickers, number inputs, toggles) — may be scrollable
- Right panel: live preview reflecting all changes in real-time
- Footer: `Save` (applies styles to that option instance) | `Cancel` (closes without saving)

Styles are stored per option instance in the `config.customization` object in the ruleset JSON.

**Default behavior:** If no customization configured, app applies default theme styles.

---

#### 3.1.15 Advanced Options (Collapsible Section)

Below the Product Options list on the tab, a collapsible "Advanced Options" drawer:

**Toggle:** Clickable header "Advanced Options" with caret icon (expand/collapse)

**Setting 1: Display MultiVariants Listing**
- Type: Radio buttons
- Options: `Yes` | `No`
- Default: `Yes`
- Function: Controls whether the MultiVariants table/grid/swatch is visible on the product page alongside custom options. If `No`, only custom options are shown (no MV table).

**Setting 2: Product Option Position**
- Type: Radio buttons
- Options: `Above MultiVariants listing` | `Below MultiVariants listing`
- Default: `Below MultiVariants listing`
- **Conditional visibility:** Only shown/active when "Display MultiVariants Listing" = `Yes`
- Function: Controls stacking order of custom options relative to MV table

**Storage:** Both settings stored in the ruleset JSON under `productOptionsConfig`.

---

#### 3.1.16 Option Ordering (Drag & Drop)

- Each option card has a drag handle icon on the left
- Merchant clicks, holds, drags to new position, releases
- New order is reflected immediately in the admin UI
- Order is persisted when merchant clicks "Save Changes"
- `sortOrder` field (0-based integer) is stored per option in the JSON array
- Storefront renders options in same sequence as admin

---

### 3.2 Storefront (Customer-Facing)

#### 3.2.1 Positioning

Controlled by the "Advanced Options" settings (§3.1.15):

**When "Display MultiVariants Listing" = Yes + Position = Below (default):**
```
┌─────────────────────────────────┐
│  [MultiVariants Title]           │
│  [Variant Table/Grid/Swatch]     │
│  [Variant rows with qty/price]   │
│                                  │
│  ── Product Options ──           │  ← Custom options here
│  [Option 1]                      │
│  [Option 2]                      │
│  ...                             │
│                                  │
│  [Add to Cart] [Checkout]        │
└─────────────────────────────────┘
```

**When "Display MultiVariants Listing" = Yes + Position = Above:**
```
┌─────────────────────────────────┐
│  ── Product Options ──           │  ← Custom options above
│  [Option 1]                      │
│  ...                             │
│                                  │
│  [MultiVariants Table]           │
│  [Add to Cart] [Checkout]        │
└─────────────────────────────────┘
```

**When "Display MultiVariants Listing" = No:**
```
┌─────────────────────────────────┐
│  ── Product Options ──           │  ← Only custom options shown
│  [Option 1]                      │
│  [Option 2]                      │
│  ...                             │
│  [Add to Cart] [Checkout]        │
└─────────────────────────────────┘
```

#### 3.2.2 Common Rendering Rules (All Option Types)

- Each option shows: **title** (with price if set: "Title (+$X.XX)"), short description (HTML rendered), input element, help text
- Required options: red asterisk (*) after title
- Validation errors shown inline below each field in red text
- Options render in the same order as configured in admin
- Options must be touch-friendly and full-width on mobile

#### 3.2.3 Text Price Calculation (Storefront JS)

```javascript
// Per word pricing
if (pricingType === 'per_word') {
  const words = input.trim().split(/\s+/).filter(w => w.length > 0);
  totalOptionPrice = words.length * priceValue;
}
// Per letter pricing
if (pricingType === 'per_letter') {
  totalOptionPrice = input.length * priceValue;
}
// Overall text (fixed price if any text entered)
if (pricingType === 'overall') {
  totalOptionPrice = input.length > 0 ? priceValue : 0;
}
```
Price display updates dynamically on every keypress.

#### 3.2.4 File Attachment Storefront Behavior

**Standard mode (Show for Each Variant = off):**
- Upload area (dashed border box) with upload button
- Supports click-to-browse + drag-and-drop
- Shows accepted file formats below area
- After upload: filename + file size + remove (X) button
- Upload progress indicator while uploading
- File POSTed to App Proxy → PHP → Shopify Files API → CDN URL returned
- CDN URL held in JS until add-to-cart

**Variant-level mode (Show for Each Variant = on):**
- Upload field appears inside EACH variant row in the MV table
- Each variant has its own independent upload
- Cart property key includes variant title: `_mv_file_{VariantTitle}`

#### 3.2.5 Cart Integration — Line Item Properties

All option values added as Shopify cart line item properties when customer clicks "Add to Cart".

**Property naming convention:** `_mv_option_{Option Title}` (prefix `_` hides from customer view but shows in Shopify Admin order detail)

```javascript
properties: {
  "_mv_option_Engraving Text":    "Happy Birthday",
  "_mv_option_Number Editions":   "42",
  "_mv_option_Gift Wrap":         "Premium (+$5.00)",
  "_mv_option_Add-ons":           "Cheese (+$1.50), Peppers",
  "_mv_option_Color":             "Red",
  "_mv_option_Style":             "Trending, Typography",
  "_mv_option_Delivery Date":     "2026-04-15",
  "_mv_option_Delivery Time":     "11:00 AM",
  "_mv_option_Upload Logo":       "https://cdn.shopify.com/s/files/1/.../logo.jpg",
  "_mv_file_Blue/L":              "https://cdn.shopify.com/s/files/1/.../variant_file.jpg"
}
```

#### 3.2.6 Price Adjustment Behavior

- All option price adjustments added to variant's total price display in real-time
- Formula: `Total = (Variant Price × Qty) + Σ(Option Prices × Qty)`
- Checkbox / multi-select swatches: sum of all selected items
- Text pricing: recalculated on each keystroke
- **Phase 1:** Display only. Actual checkout price enforcement via Cart Transform Function is Phase 2.

#### 3.2.7 Add-to-Cart Validation

Before proceeding, validate:
1. Required text/number fields are not empty
2. Text fields do not exceed max characters
3. Number fields are within min/max range
4. Required dropdown/radio has a selection
5. Required checkbox/swatch has at least one selection
6. Required file attachment has a completed upload
7. Required date is selected and falls on an available day
8. Required time is selected (if Show Time enabled)

On failure: block add-to-cart, show inline red error per field, scroll to first error.

---

### 3.3 Data Storage

All product options stored in the ruleset JSON at `/shop/{alias}/usr/set/{id}.json`.

#### New top-level fields in Ruleset JSON:

```json
{
  "...existing fields...",
  "productOptionsConfig": {
    "displayMVListing": true,
    "optionPosition": "below"
  },
  "productOptions": [
    {
      "id": "opt_abc123",
      "type": "text",
      "sortOrder": 0,
      "title": "Engraving Text",
      "shortDescription": "Personalize your item",
      "helpText": "Max 30 characters",
      "required": true,
      "config": {
        "inputType": "short",
        "placeholder": "Write your text",
        "maxCharacters": 30,
        "pricingType": "per_letter",
        "priceValue": 2.00,
        "customization": {
          "inputBgColor": "#ffffff",
          "inputBorderColor": "#cccccc",
          "placeholderColor": "#999999"
        }
      }
    },
    {
      "id": "opt_def456",
      "type": "number",
      "sortOrder": 1,
      "title": "Number Editions",
      "shortDescription": "",
      "helpText": "Enter between 1 and 300",
      "required": false,
      "config": {
        "placeholder": "Write your number",
        "minValue": 1,
        "maxValue": 300,
        "price": 8.00,
        "customization": {
          "inputBgColor": "#ffffff",
          "inputBorderColor": "#cccccc",
          "placeholderColor": "#999999"
        }
      }
    },
    {
      "id": "opt_ghi789",
      "type": "dropdown",
      "sortOrder": 2,
      "title": "Dropdown",
      "shortDescription": "",
      "helpText": "",
      "required": true,
      "config": {
        "choices": [
          { "id": "ch_001", "title": "Small", "imageUrl": "", "price": 0, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Medium", "imageUrl": "", "price": 2.00, "default": true, "sortOrder": 1 },
          { "id": "ch_003", "title": "Large", "imageUrl": "https://cdn.shopify.com/s/files/1/.../large.jpg", "price": 5.00, "default": false, "sortOrder": 2 }
        ],
        "customization": {
          "inputBgColor": "#ffffff",
          "inputBorderColor": "#cccccc",
          "placeholderColor": "#999999"
        }
      }
    },
    {
      "id": "opt_jkl012",
      "type": "radio",
      "sortOrder": 3,
      "title": "Radio Editions",
      "shortDescription": "",
      "helpText": "",
      "required": true,
      "config": {
        "choices": [
          { "id": "ch_001", "title": "Choice 01", "imageUrl": "", "price": 0, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Choice 02", "imageUrl": "", "price": 5.00, "default": true, "sortOrder": 1 }
        ],
        "customization": {
          "defaultBgColor": "#ffffff",
          "defaultBorderColor": "#cccccc",
          "activeBgColor": "#0070f3",
          "activeBorderColor": "#0070f3",
          "imageBoxSize": "40",
          "imageBoxBorderRadius": "4",
          "imageBoxBorderColor": "#cccccc",
          "typographyColor": "#333333"
        }
      }
    },
    {
      "id": "opt_mno345",
      "type": "checkbox",
      "sortOrder": 4,
      "title": "Add-ons",
      "shortDescription": "",
      "helpText": "",
      "required": false,
      "config": {
        "choices": [
          { "id": "ch_001", "title": "Cheese", "imageUrl": "", "price": 1.50, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Mushroom", "imageUrl": "", "price": 2.00, "default": true, "sortOrder": 1 }
        ],
        "customization": {
          "defaultBgColor": "#ffffff",
          "defaultBorderColor": "#cccccc",
          "activeBgColor": "#0070f3",
          "activeBorderColor": "#0070f3",
          "imageBoxSize": "40",
          "imageBoxBorderRadius": "4",
          "imageBoxBorderColor": "#cccccc",
          "typographyColor": "#333333"
        }
      }
    },
    {
      "id": "opt_pqr678",
      "type": "button_swatch",
      "sortOrder": 5,
      "title": "Button Swatch Editions",
      "shortDescription": "",
      "helpText": "",
      "required": false,
      "config": {
        "swatchStyle": "single",
        "choices": [
          { "id": "ch_001", "title": "Swatch 01", "price": 0, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Swatch 02", "price": 10.00, "default": true, "sortOrder": 1 }
        ],
        "customization": {
          "defaultBgColor": "#ffffff",
          "defaultBorderColor": "#cccccc",
          "activeBgColor": "#0070f3",
          "activeBorderColor": "#0070f3",
          "borderRadius": "4",
          "paddingTopBottom": "8",
          "paddingLeftRight": "12",
          "defaultTextColor": "#333333",
          "activeTextColor": "#ffffff"
        }
      }
    },
    {
      "id": "opt_stu901",
      "type": "color_swatch",
      "sortOrder": 6,
      "title": "Color Swatch Editions",
      "shortDescription": "",
      "helpText": "",
      "required": false,
      "config": {
        "swatchStyle": "single",
        "choices": [
          { "id": "ch_001", "title": "Red", "colorHex": "#F24F4F", "price": 0, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Blue", "colorHex": "#4F7CF2", "price": 5.00, "default": true, "sortOrder": 1 }
        ],
        "customization": {
          "defaultBgColor": "#f0f0f0",
          "defaultBorderColor": "#cccccc",
          "activeBgColor": "#ffffff",
          "activeBorderColor": "#0070f3",
          "colorBoxRadius": "50%",
          "borderRadius": "4",
          "paddingTopBottom": "4",
          "paddingLeftRight": "4",
          "showLabel": true,
          "labelPosition": "right",
          "defaultTextColor": "#333333",
          "activeTextColor": "#0070f3"
        }
      }
    },
    {
      "id": "opt_vwx234",
      "type": "image_swatch",
      "sortOrder": 7,
      "title": "Image Swatch Editions",
      "shortDescription": "",
      "helpText": "",
      "required": false,
      "config": {
        "swatchStyle": "multiple",
        "choices": [
          { "id": "ch_001", "title": "Trending", "imageUrl": "https://cdn.shopify.com/s/files/1/.../trending.jpg", "price": 0, "default": false, "sortOrder": 0 },
          { "id": "ch_002", "title": "Professional", "imageUrl": "https://cdn.shopify.com/s/files/1/.../professional.jpg", "price": 10.00, "default": true, "sortOrder": 1 }
        ],
        "customization": {
          "defaultBgColor": "#f0f0f0",
          "defaultBorderColor": "#cccccc",
          "activeBgColor": "#ffffff",
          "activeBorderColor": "#0070f3",
          "imageBoxRadius": "4",
          "borderRadius": "4",
          "paddingTopBottom": "4",
          "paddingLeftRight": "4",
          "showLabel": true,
          "labelPosition": "right",
          "defaultTextColor": "#333333",
          "activeTextColor": "#0070f3"
        }
      }
    },
    {
      "id": "opt_yz0567",
      "type": "file_attachment",
      "sortOrder": 8,
      "title": "Upload Logo",
      "shortDescription": "Upload your logo for printing",
      "helpText": "Max 50MB. JPEG, PNG, WEBP accepted.",
      "required": true,
      "config": {
        "price": 15.00,
        "maxFileSizeMB": 50,
        "showForEachVariant": false,
        "allowedTypes": ["JPEG", "PNG", "WEBP"],
        "customization": {
          "boxBgColor": "#f9f9f9",
          "boxBorderColor": "#cccccc",
          "buttonBgColor": "#0070f3",
          "buttonTextColor": "#ffffff",
          "boxBorderRadius": "8",
          "paddingTopBottom": "20",
          "paddingLeftRight": "20"
        }
      }
    },
    {
      "id": "opt_ab1234",
      "type": "date_time",
      "sortOrder": 9,
      "title": "Delivery Date",
      "shortDescription": "",
      "helpText": "Select your preferred delivery date",
      "required": true,
      "config": {
        "price": 0,
        "showDate": true,
        "showTime": false,
        "availableDays": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
      }
    }
  ]
}
```

#### New Feature Flag in `/pate/config/package_items.php`:

```php
"product_options" => ["standard_monthly", "standard_yearly", "professional_monthly", "professional_yearly"]
```

---

### 3.4 API Changes

#### 3.4.1 Modified: Product Data Endpoint (`/gate/out/usr/data/product/index.php`)

Add `productOptions` and `productOptionsConfig` to the response JSON:

```php
// After existing $response is built:
$response['productOptionsConfig'] = $set['productOptionsConfig'] ?? [
    'displayMVListing' => true,
    'optionPosition' => 'below'
];
$response['productOptions'] = $set['productOptions'] ?? [];
```

#### 3.4.2 Modified: Update Ruleset Endpoint (`/gate/out/adm/action/update_rule_set.php`)

Accept and save `productOptions` and `productOptionsConfig` from POST body:

```php
if (isset($_POST['productOptions'])) {
    $set['productOptions'] = json_decode($_POST['productOptions'], true) ?? [];
}
if (isset($_POST['productOptionsConfig'])) {
    $set['productOptionsConfig'] = json_decode($_POST['productOptionsConfig'], true) ?? [];
}
```

#### 3.4.3 New Admin Action: Image Upload for Option Attributes

**File:** `gate/out/adm/action/upload_option_image.php`

Used by merchant when uploading images for Image Swatch attributes or choice images in Dropdown/Radio/Checkbox.

```
POST /gate/out/adm/action/upload_option_image.php

Auth: Admin session (existing session check)
Request: multipart/form-data { file, shop }

Response success: { "status": "success", "url": "https://cdn.shopify.com/..." }
Response error:   { "status": "error", "message": "..." }
```

**Implementation:** Validate image (type/size ≤ 20MB) → `stagedUploadsCreate` with `resource: IMAGE` → upload to GCS staged URL → `fileCreate` mutation → poll until `fileStatus == READY` → return CDN URL.

#### 3.4.4 New App Proxy Endpoint: Customer File Upload

**File:** `gate/out/usr/upload/index.php`

**Add to `shopify.app.toml`:**
```toml
[app_proxy]
url = "https://sapp.multivariants.com/gate/out/usr/upload/"
subpath = "multivariants-upload"
prefix = "apps"
```

Storefront URL: `https://{shop}.myshopify.com/apps/multivariants-upload`

```
POST https://{shop}.myshopify.com/apps/multivariants-upload

Request: multipart/form-data { file, option_id }
Shopify adds query params: shop, timestamp, signature, path_prefix

Response success: { "status": "success", "url": "https://cdn.shopify.com/..." }
Response error:   { "status": "error", "message": "..." }
```

**PHP flow:**
1. Verify App Proxy HMAC signature (query string `signature` parameter, using HMAC-SHA256 with API secret)
2. Load shop access token from `/shop/{alias}/sys/data/shop.json`
3. Validate file type against option's `allowedTypes` config
4. Validate file size against option's `maxFileSizeMB` config
5. Map MIME type to Shopify resource: IMAGE (jpeg/png/webp/svg/heic/gif/tiff → 20MB), VIDEO (mp4/mov/webm → 1GB), MODEL_3D (glb/usdz → 500MB), FILE (psd → ⚠️ 2MB practical limit)
6. Call `stagedUploadsCreate` → upload to GCS → `fileCreate` → poll until READY → return CDN URL
7. **PSD/USDZ limitation:** If FILE resource and size > 2MB, return error: "This file type must be under 2MB due to a platform limitation."

**Required scopes** (add to `$_API_SCOPES` in `pate/config/api.php`): `read_files,write_files`

**Shopify Files API mutation reference:**
```graphql
# Step 1
mutation stagedUploadsCreate($input: [StagedUploadInput!]!) {
  stagedUploadsCreate(input: $input) {
    stagedTargets {
      url          # POST multipart to this URL (Google Cloud Storage)
      resourceUrl  # Pass to fileCreate as originalSource
      parameters { name value }
    }
    userErrors { field message }
  }
}

# Step 3
mutation fileCreate($files: [FileCreateInput!]!) {
  fileCreate(files: $files) {
    files {
      id
      fileStatus
      ... on MediaImage { image { url } }
      ... on GenericFile { url }
      ... on Video { sources { url } }
    }
    userErrors { field message }
  }
}

# Step 4: Poll until READY
query getFileStatus($id: ID!) {
  node(id: $id) {
    ... on MediaImage { fileStatus image { url } }
    ... on GenericFile { fileStatus url }
    ... on Video { fileStatus sources { url } }
  }
}
```

---

## 4. UI/UX Specifications

### 4.1 Admin UI Design Patterns

- **Tab navigation:** Polaris `Tabs` component. "Product Options" tab alongside "Advanced restrictions"
- **Empty state:** Polaris `EmptyState` component with dashed border. Centered `+ Add option` button and instruction text
- **Option cards:** Polaris `Card` with collapsible body. Header row: drag handle + type label + gear + trash + chevron
- **Modal:** Polaris `Modal` component. Radio list of 10 option types. Footer: Cancel + Add Now (disabled until selection)
- **Attribute tables:** Custom repeatable rows inside card. Columns: drag handle, fields, delete icon. `+ Add Attribute` button below table
- **Color picker:** HTML `<input type="color">` or Polaris-compatible color picker
- **File upload (admin):** Standard `<input type="file">` button. Show thumbnail preview after upload. Show spinner during upload
- **Gear popup:** Polaris `Modal` with two-panel layout: left (style fields) + right (live preview iframe or CSS-in-JS preview)
- **Drag and drop:** Use HTML5 Drag and Drop API or SortableJS for both option ordering and attribute row ordering

### 4.2 Storefront UI Design Patterns

- **Container:** Rendered below (or above) the MV app block based on `optionPosition` setting
- **Option wrapper:** Each option in a `<div class="mv-option-{type}">` wrapper
- **Labels:** `<label>` element with bold styling. Required asterisk in red: `<span class="mv-required">*</span>`
- **Short description:** `<div class="mv-option-description">` renders HTML
- **Help text:** `<div class="mv-option-help">` below input, gray smaller text
- **Error messages:** `<div class="mv-option-error">` red text, shown inline below field
- **Swatch layouts:** Flexbox wrap for button, color, image swatches
- **Mobile:** 100% width, min touch target 44×44px for swatches and buttons
- **File upload zone:** Dashed border box, centered upload icon + button text, drag-over highlight state

### 4.3 Interaction States

| Element | Default | Active/Filled | Error | Disabled |
|---------|---------|--------------|-------|---------|
| Text/Number input | Empty + placeholder | Blue border | Red border + error msg below | Gray, no interaction |
| Dropdown | "Select an option" | Expanded list | Red border | Gray |
| Radio button | Unchecked circle | Filled circle (blue) | Red border on group | Gray |
| Checkbox | Unchecked box | Checkmark (blue) | Red border on group | Gray |
| Button swatch | Normal (default style) | Active (style from customization) | Red border | Gray |
| Color swatch | Circle, default border | Active border + indicator | Red border on group | Gray overlay |
| Image swatch | Thumbnail, default border | Active border/overlay | Red border on group | Gray overlay |
| File upload area | Dashed border | Drag-over: blue border highlight | Red border + error | Gray, no click |
| Date picker | Empty input | Calendar open, days highlighted | Red border | Unavailable days grayed |
| Time picker | Empty | Selected value shown | Red border | Grayed |

---

## 5. Technical Considerations

### 5.1 Files to Modify

**Admin (PHP/HTML rendering):**
- `gate/out/sys/adm/index.php` — Add "Product Options" tab content, option card HTML templates, "Add option" modal HTML, gear popup HTML
- `gate/out/sys/adm/init.js` — Add JS for: modal open/close, radio selection, option CRUD, attribute table add/remove/reorder, gear popup open/save, drag-and-drop order, admin image upload AJAX

**Backend (PHP endpoints):**
- `gate/out/adm/action/update_rule_set.php` — Accept and save `productOptions` + `productOptionsConfig`
- `gate/out/usr/data/product/index.php` — Include `productOptions` + `productOptionsConfig` in response
- NEW: `gate/out/adm/action/upload_option_image.php` — Admin image upload to Shopify Files API
- NEW: `gate/out/usr/upload/index.php` — Customer file upload App Proxy endpoint

**Storefront:**
- `extensions/theme-app-extension/assets/mv.js` — Render product options, handle validation, include values in `/cart/add.js` call, handle file upload AJAX to App Proxy

**Config:**
- `pate/config/package_items.php` — Add `product_options` feature flag
- `pate/config/api.php` — Add `read_files,write_files` to `$_API_SCOPES`
- `shopify.app.toml` — Add `[app_proxy]` configuration block

### 5.2 Admin JS Architecture (init.js additions)

```javascript
// Option ID generation
function generateOptionId() {
  return 'opt_' + Math.random().toString(36).substr(2, 9);
}

// In-memory state (JS object)
let productOptions = [];       // Array of option objects
let productOptionsConfig = {}; // displayMVListing, optionPosition

// Add option from modal
function addOption(type) {
  const opt = {
    id: generateOptionId(),
    type: type,
    sortOrder: productOptions.length,
    title: '',
    shortDescription: '',
    helpText: '',
    required: false,
    config: getDefaultConfig(type)
  };
  productOptions.push(opt);
  renderOptionCard(opt);
}

// Serialize to form fields on Save
function serializeOptions() {
  document.getElementById('productOptionsJson').value = JSON.stringify(productOptions);
  document.getElementById('productOptionsConfigJson').value = JSON.stringify(productOptionsConfig);
}
```

Add hidden form fields to the ruleset save form:
```html
<input type="hidden" id="productOptionsJson" name="productOptions">
<input type="hidden" id="productOptionsConfigJson" name="productOptionsConfig">
```

### 5.3 Shopify Files API — Complete PHP Implementation

**`upload_option_image.php` and `usr/upload/index.php` shared helper (add to `pate/lib/ShopifyFiles.php`):**

```php
class ShopifyFiles {
    public static function upload(string $filePath, string $filename, string $mimeType, string $resource, $shopApi): string {
        // Step 1: stagedUploadsCreate
        $mutation = '
            mutation stagedUploadsCreate($input: [StagedUploadInput!]!) {
              stagedUploadsCreate(input: $input) {
                stagedTargets { url resourceUrl parameters { name value } }
                userErrors { field message }
              }
            }
        ';
        $variables = ['input' => [[
            'filename' => $filename,
            'mimeType' => $mimeType,
            'resource' => $resource,
            'fileSize' => (string)filesize($filePath),
            'httpMethod' => 'POST',
        ]]];
        $result = $shopApi->graphql($mutation, $variables);
        $target = $result['data']['stagedUploadsCreate']['stagedTargets'][0];

        // Step 2: Upload to GCS staged URL
        $postFields = [];
        foreach ($target['parameters'] as $p) {
            $postFields[$p['name']] = $p['value'];
        }
        $postFields['file'] = new CURLFile($filePath, $mimeType, $filename);
        $ch = curl_init($target['url']);
        curl_setopt_array($ch, [
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => $postFields,
            CURLOPT_RETURNTRANSFER => true,
        ]);
        $uploadResponse = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);
        if ($httpCode !== 201 && $httpCode !== 204) {
            throw new Exception("Staged upload failed: HTTP $httpCode");
        }

        // Step 3: fileCreate
        $createMutation = '
            mutation fileCreate($files: [FileCreateInput!]!) {
              fileCreate(files: $files) {
                files { id fileStatus ... on MediaImage { image { url } } ... on GenericFile { url } }
                userErrors { field message }
              }
            }
        ';
        $createResult = $shopApi->graphql($createMutation, [
            'files' => [['originalSource' => $target['resourceUrl'], 'contentType' => $resource]]
        ]);
        $file = $createResult['data']['fileCreate']['files'][0];
        $fileId = $file['id'];

        // Step 4: Poll until READY (max 10 attempts, 1s apart)
        $pollQuery = '
            query getFile($id: ID!) {
              node(id: $id) {
                ... on MediaImage { fileStatus image { url } }
                ... on GenericFile { fileStatus url }
                ... on Video { fileStatus sources { url mimeType } }
              }
            }
        ';
        for ($i = 0; $i < 10; $i++) {
            sleep(1);
            $pollResult = $shopApi->graphql($pollQuery, ['id' => $fileId]);
            $node = $pollResult['data']['node'];
            if ($node['fileStatus'] === 'READY') {
                return $node['image']['url'] ?? $node['url'] ?? $node['sources'][0]['url'];
            }
            if ($node['fileStatus'] === 'FAILED') {
                throw new Exception("File processing failed on Shopify");
            }
        }
        throw new Exception("File processing timed out");
    }
}
```

### 5.4 Dependencies

- Does NOT affect the existing restriction system (min/max qty, bundles, etc.)
- Compatible with ALL display layouts: list, dropdown, swatch, grid/matrix
- The `productOptions` data is a separate array — no changes to existing ruleset fields
- The `displayMVListing = No` setting hides the MV table entirely — `mv.js` must check this flag before rendering the variant table
- `read_files` + `write_files` scopes must be added to OAuth scopes — **this will trigger a re-authorization prompt for existing merchants**

### 5.5 Cart Integration

- Options passed as line item properties (prefix `_mv_option_`) — no Cart Validation Function changes needed for Phase 1
- Price adjustments are display-only in Phase 1 — Cart Transform Function changes are Phase 2
- No changes needed to the Cart Data endpoint (`/gate/out/usr/data/cart/`)

### 5.6 Translation Keys (New)

```
PO_SectionTitle         = "Product Options"
PO_Required             = "Required"
PO_Optional             = "Optional"
PO_SelectOption         = "Select an option"
PO_FileUploadBtn        = "Upload File"
PO_FileUploading        = "Uploading..."
PO_FileUploaded         = "File uploaded: {filename}"
PO_FileRemove           = "Remove"
PO_FileError            = "Upload failed. Please try again."
PO_FileTypeError        = "File type not allowed. Accepted: {types}"
PO_FileSizeError        = "File exceeds {size}MB limit."
PO_DateSelect           = "Select date"
PO_TimeSelect           = "Select time"
PO_PriceAddon           = "(+{currency}{price})"
PO_ErrorRequired        = "{field} is required."
PO_ErrorMaxChars        = "{field} must be {max} characters or less."
PO_ErrorMinValue        = "{field} must be at least {min}."
PO_ErrorMaxValue        = "{field} must be {max} or less."
PO_ErrorDateUnavailable = "Selected date is not available. Please choose another."
PO_AdvancedOptions      = "Advanced Options"
PO_DisplayMVListing     = "Display MultiVariants Listing"
PO_OptionPosition       = "Product Option Position"
PO_PositionAbove        = "Above MultiVariants listing"
PO_PositionBelow        = "Below MultiVariants listing"
```

---

## 6. Edge Cases & Constraints

| Scenario | Expected Behavior |
|----------|------------------|
| Max options per ruleset | 20 options maximum. Enforce in admin JS and PHP save. Show error if exceeded. |
| Max attribute rows per option | 50 rows per dropdown/radio/checkbox/swatch. Enforce in JS. |
| Max file size | 50MB configurable by merchant. Hard limit enforced server-side. |
| PSD/USDZ > 2MB via Shopify Files | Return error: "PSD/USDZ files must be under 2MB due to a platform limitation. Use PNG/JPEG instead." |
| Two options with identical title | Append " (2)", " (3)" etc. to cart property key to avoid collision: `_mv_option_Title (2)` |
| Products with no variants | Options work normally. `showForEachVariant` on File Attachment renders one upload (since there's one default variant) |
| MV Display = No with variant restrictions | MV restriction enforcement still runs via Cart Validation Function. Only the UI table is hidden |
| Required option + no choices added | Admin must show validation: "Add at least one choice before saving" |
| File upload fails mid-process | Clear the upload state, show error message, allow retry. Block add-to-cart if required |
| Date availability = no days checked | All days selectable (treat as "no restriction"). Admin warning: "No available days will allow all days" |
| Customer uploads on mobile | File picker opens native mobile file browser. No drag-and-drop on mobile |
| Network failure during upload | Show retry button. Keep file selected. Don't clear on failure |
| Concurrent add-to-cart clicks | Debounce add-to-cart. File must be fully uploaded before cart AJAX fires |
| Ruleset with no productOptions | `productOptions = []` — don't render any options section on storefront |
| Drag reorder with unsaved changes | Changes held in JS memory. Order only persisted on explicit Save |

---

## 7. Acceptance Criteria

### Admin

- [ ] "Product Options" tab visible on Create/Edit Ruleset page (Standard + Professional plans only)
- [ ] Starter plan: tab hidden or shown with plan upgrade prompt
- [ ] "Add option" modal opens when clicking `+ Add option` / `+ Add more options`
- [ ] Modal uses radio buttons — only one type selectable at a time
- [ ] "Add Now" button disabled until selection made
- [ ] All 10 option types can be added and configured
- [ ] Number option: title, placeholder, min, max, price, required fields save correctly
- [ ] Text option: input type (short/long), pricing type (per word/letter/overall), max chars save correctly
- [ ] Dropdown/Radio/Checkbox: attribute rows with title, image, price, default, drag, delete function correctly
- [ ] Button Swatch: single/multiple choice style setting works. No image per attribute
- [ ] Color Swatch: color picker works, single/multiple style works
- [ ] Image Swatch: image upload works, CDN URL stored in JSON
- [ ] File Attachment: allowed types checkboxes, max size, "Show for Each Variant" toggle all save correctly
- [ ] Date & Time: Show Date/Show Time checkboxes, availability days multi-select saves correctly
- [ ] Gear icon opens customization popup with correct fields per option type
- [ ] Customization popup live preview updates in real-time
- [ ] Options can be drag-reordered. Order persists after save and page reload
- [ ] Options can be deleted with trash icon
- [ ] Advanced Options section: Display MV Listing and Position settings save correctly
- [ ] Position setting hidden when Display MV Listing = No
- [ ] Saving the ruleset persists all `productOptions` and `productOptionsConfig` in JSON

### Storefront

- [ ] Custom options render in correct position (above/below MV table per config)
- [ ] MV table hidden when `displayMVListing = false`
- [ ] Options render in same order as configured in admin
- [ ] Required options show asterisk (*)
- [ ] Short description HTML renders correctly
- [ ] Text option: Short text = single-line input. Long text = textarea
- [ ] Text option: Price updates dynamically as customer types (per word/letter/overall)
- [ ] Number option: Min/max range enforced with error message
- [ ] Dropdown: Pre-selected default works. One selection only
- [ ] Radio: Pre-selected default works. One selection only. Images shown if configured
- [ ] Checkbox: Multiple selections. Sum of prices shown. Images shown if configured
- [ ] Button Swatch: Single/multiple mode works. Price updates on selection
- [ ] Color Swatch: Color circles shown. Single/multiple mode. Label shown/hidden per config
- [ ] Image Swatch: Thumbnails shown. Single/multiple mode. Label shown/hidden per config
- [ ] File Attachment (standard): Upload via click or drag-drop. File validates type/size. CDN URL received
- [ ] File Attachment (variant-level): Upload field in each variant row. Per-variant URLs in cart
- [ ] Date picker: Only available days are selectable. Others are grayed
- [ ] Time picker: Shows if "Show Time" enabled
- [ ] Price adjustments shown as "(+$X.XX)" and update total in real-time
- [ ] Required validation blocks add-to-cart. Error messages shown inline
- [ ] All option values appear as `_mv_option_*` line item properties in the cart
- [ ] Line item properties visible in Shopify Admin order detail view
- [ ] All option types are touch-friendly and work on mobile

### Data

- [ ] `productOptions` array stored in ruleset JSON under correct structure
- [ ] `productOptionsConfig` object stored in ruleset JSON
- [ ] Feature flag `product_options` in `package_items.php` gates Standard + Professional only
- [ ] Product data API (`/gate/out/usr/data/product/`) includes `productOptions` in response
- [ ] Admin image upload stores Shopify CDN URL in JSON (not local path)
- [ ] Customer uploaded files stored in Shopify Files (CDN URLs in cart properties)
- [ ] `read_files` + `write_files` scopes added to API config

---

## 8. Phased Rollout

### Phase 1 (This PRD)
- All 10 option types — admin configuration and storefront rendering
- Cart line item properties
- File upload via App Proxy + Shopify Files API
- Price display (dynamic, real-time)
- Drag-and-drop ordering
- Option Customization (gear icon styling)
- Advanced Options (display MV listing, position)

### Phase 2 (Future)
- Price enforcement via Cart Transform Function (actual checkout price update)
- Conditional option logic (show Option B only if Option A = "Yes")

### Phase 3 (Future)
- Option templates (save/reuse option sets across rulesets)
- Analytics on option usage per order
- Bulk import/export of option configurations

---

## 9. File Upload: Shopify Files API vs Alternatives

| Approach | Admin Token Exposed? | Third-Party | Max Size | Files in Shopify Admin? | Backend Required? |
|----------|---------------------|------------|---------|------------------------|------------------|
| **App Proxy + Shopify Files API** (recommended) | No | None | Images 20MB, Video 1GB, GLB/USDZ 500MB, PSD ~2MB (bug) | Yes, directly | Yes (PHP) |
| Cloudinary (unsigned preset) | No (public key only) | Cloudinary | 10MB free | No (URL only) | Optional |
| Uploadcare (public key) | No (public key only) | Uploadcare | 100MB free | No (URL only) | No |
| AWS S3 presigned URL | No | AWS | 5TB | No (URL only) | Yes (presign) |

**Recommendation:** Use App Proxy + Shopify Files API as primary approach. For PSD/USDZ files > 2MB, show a user-friendly error until Shopify resolves the known `FILE` resource bug. Revisit Cloudinary as fallback for non-image types if the Shopify bug persists past the initial release.

---

*PRD Version 2.0 — Updated from FRS document (MultiVariants FRS Product Option.pdf)*
*Backup of v1: `docs/prd/PRD_001_PRODUCT_OPTIONS_v1_backup.md`*
