# PRD-001: Product Options (Custom Options Beyond Shopify's 3-Option Limit)

## 1. Overview

**Feature Name:** Product Options
**Priority:** P1 High
**Plan Tier:** Standard + Professional (locked/hidden for Starter)
**Estimated Scope:** Large (1-2 weeks)

### 1.1 Problem Statement

Shopify limits products to a maximum of 3 variant options (e.g., Color, Size, Material). Many merchants need additional custom fields on the product page - such as engraving text, custom file uploads (logos, artwork), date pickers, or extra selection options - that Shopify's native variant system cannot support. Currently, merchants must install a separate app for this, adding cost and complexity.

### 1.2 Goal

Allow merchants to add unlimited custom product options (beyond Shopify's 3-option limit) directly within a MultiVariants ruleset, giving customers more choices and flexibility on the product page. These options are passed as cart line item properties and visible in the order.

### 1.3 Success Metrics

- Increase in Professional plan upgrades (feature is a differentiator)
- Reduction in merchant requests for "custom fields" support
- Merchant adoption rate: 30%+ of Professional merchants use at least 1 custom option within 60 days

---

## 2. User Stories

- As a **merchant**, I want to add custom text input fields to my product page, so that customers can enter personalization details (e.g., engraving, monogram).
- As a **merchant**, I want to add a file upload option, so that customers can upload artwork, logos, or reference images with their order.
- As a **merchant**, I want to add dropdown/radio/checkbox selectors beyond the 3-option limit, so that customers can choose from additional product attributes.
- As a **merchant**, I want to set some options as required and others as optional, so that I can control the ordering flow.
- As a **merchant**, I want to add extra charges for certain option choices, so that premium customizations are priced appropriately.
- As a **customer**, I want to see clearly labeled custom options on the product page and fill them in before adding to cart.
- As a **customer**, I want to upload a file and see confirmation that it was accepted.

---

## 3. Functional Requirements

### 3.1 Admin Panel (Merchant Configuration)

#### 3.1.1 New Tab: "Product Options" (on Create/Edit Ruleset Page)

**Location:** On the create/edit ruleset page, add a new tab alongside "Advanced restrictions" tab.

**Tab layout:**
- **Header:** "Product options"
- **Subtext:** "Add unlimited custom options beyond Shopify's 3-option limit, giving your customers more choices and flexibility on the product page."
- **Empty state:** Dashed border area with "+ Add option" button and text "Choose option types you want to set for your products."
- **Populated state:** Vertical list of configured options, each collapsible, with drag-to-reorder support. At the bottom: "+ Add more options" button.

#### 3.1.2 "Add Option" Modal

**Trigger:** Click "+ Add option" button
**Type:** Modal/popup overlay
**Content:** List of option types with checkbox + eye icon (preview):

| Option Type | Description | Input for Customer |
|-------------|-------------|--------------------|
| **Text** | Single-line text input | Free text (e.g., "Enter name for engraving") |
| **Number** | Numeric input | Number value (e.g., "Enter quantity") |
| **Dropdown** | Select from predefined choices | Dropdown selector |
| **Radio** | Single choice from options | Radio button group |
| **Checkbox** | Multiple choices from options | Checkbox group |
| **Button Swatch** | Text-based swatch buttons | Clickable text buttons |
| **Color Swatch** | Color picker / color buttons | Clickable color circles |
| **Image Swatch** | Image-based swatch buttons | Clickable image thumbnails |
| **File Attachment** | File upload | File picker + drag-and-drop |
| **Date and Time** | Date/time picker | Calendar + time selector |

**Behavior:**
- Merchant can select multiple option types at once
- Click "Add option" button to add all selected types
- Click "Cancel" to close modal without adding
- Each option type can be added multiple times (e.g., two text fields)

#### 3.1.3 Option Configuration (Per Option, Collapsible Card)

Each added option appears as a collapsible card with:

**Header bar:**
- Option type name + description (e.g., "Checkbox - Allow merchants to offer customizable choices for a single product")
- Delete button (trash icon)
- Settings button (gear icon) - opens advanced settings
- Collapse/expand toggle (chevron)

**Common fields (all option types):**

| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Option title | Text input | Yes | "" | Max 100 chars |
| Short description | Text input | No | "" | Max 255 chars, supports HTML |
| Help text | Text input | No | "" | Max 255 chars |
| Required | Dropdown (Yes/No) | Yes | "No" | - |

**Type-specific fields:**

**Text:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Placeholder text | Text input | No | "" | Max 100 chars |
| Max characters | Number input | No | 500 | Min: 1, Max: 5000 |
| Price adjustment | Number input | No | 0 | Min: 0, decimal allowed |

**Number:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Placeholder text | Text input | No | "" | Max 100 chars |
| Min value | Number input | No | 0 | - |
| Max value | Number input | No | 99999 | Must be >= min |
| Step | Number input | No | 1 | Min: 0.01 |
| Price adjustment | Number input | No | 0 | Min: 0 |

**Dropdown / Radio / Checkbox / Button Swatch:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Choices table | Repeatable rows | Yes (min 1) | 1 empty row | - |
| - Choice title | Text input | Yes | "" | Max 100 chars |
| - Price | Number input | No | 0 | Min: 0, decimal allowed |
| - Default | Checkbox | No | unchecked | Only 1 default for radio/dropdown |
| + Add attribute | Button | - | - | Adds new row |
| - Delete row | Button (trash) | - | - | Min 1 row must remain |

**Color Swatch:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Choices table | Repeatable rows | Yes (min 1) | 1 empty row | - |
| - Swatch label | Text input | Yes | "" | Max 50 chars |
| - Color value | Color picker | Yes | #000000 | Valid hex color |
| - Price | Number input | No | 0 | Min: 0 |
| - Default | Checkbox | No | unchecked | - |

**Image Swatch:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Choices table | Repeatable rows | Yes (min 1) | 1 empty row | - |
| - Swatch label | Text input | Yes | "" | Max 50 chars |
| - Image | File upload | Yes | - | Max 2MB, jpg/png/webp |
| - Price | Number input | No | 0 | Min: 0 |
| - Default | Checkbox | No | unchecked | - |

**File Attachment:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Allowed file types | Multi-select | No | All types | jpg, png, gif, webp, pdf, svg, ai, eps, psd, zip |
| Max file size (MB) | Number input | No | 10 | Min: 1, Max: 50 |
| Max files | Number input | No | 1 | Min: 1, Max: 10 |
| Upload instructions | Text input | No | "Drag and drop or click to upload" | Max 255 chars |
| Price adjustment | Number input | No | 0 | Min: 0 |

**Date and Time:**
| Field | Type | Required | Default | Validation |
|-------|------|----------|---------|------------|
| Date format | Dropdown | No | "MM/DD/YYYY" | MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD |
| Include time | Toggle (Yes/No) | No | "No" | - |
| Min date | Date picker | No | null | - |
| Max date | Date picker | No | null | Must be >= min date |
| Disable specific days | Multi-select | No | none | Sun, Mon, Tue, Wed, Thu, Fri, Sat |
| Price adjustment | Number input | No | 0 | Min: 0 |

#### 3.1.4 Option Ordering

- Drag-and-drop to reorder options in the list
- Order determines display order on storefront
- Stored as array position in JSON

### 3.2 Storefront (Customer-Facing)

#### 3.2.1 Positioning

Custom options render **below** the MultiVariants variant listing and **above** the Add to Cart button.

```
┌─────────────────────────────────┐
│  [MultiVariants Title]           │
│  [Variant Table/Grid/Swatch]     │
│  [Variant rows with qty/price]   │
│                                  │
│  ── Custom Options Section ──    │  ← NEW
│  [Option 1: Text input]         │
│  [Option 2: Dropdown]           │
│  [Option 3: File upload]        │
│  [Option 4: Checkbox group]     │
│                                  │
│  [Total Price: $XX.XX]           │
│  [Add to Cart] [Checkout]        │
└─────────────────────────────────┘
```

#### 3.2.2 Rendering Rules

- Each option shows: **title** (bold), help text (below, smaller), input element
- Required options show red asterisk (*) after title
- Short description renders as HTML below the title (if set)
- Options with price adjustments show "+$X.XX" next to the choice label
- Total price updates dynamically when options with price adjustments are selected
- Validation errors show inline below each field in red text

#### 3.2.3 File Upload Behavior

- Drag-and-drop zone with dashed border
- Click to open file picker
- Show upload progress bar during upload
- Show file name and size after upload, with remove (X) button
- Files are uploaded to the server and stored temporarily
- File URL is passed as a cart line item property
- Server endpoint: `POST /gate/out/usr/data/upload/` (new endpoint)
- Files stored in: `/shop/{alias}/usr/uploads/{timestamp}_{filename}`
- Auto-cleanup: uploaded files older than 7 days are purged

#### 3.2.4 Cart Integration

All custom option values are added as **Shopify cart line item properties**:

```javascript
// When adding to cart via /cart/add.js:
{
  id: variantId,
  quantity: qty,
  properties: {
    "_mv_option_Engraving Text": "Happy Birthday John",
    "_mv_option_Gift Wrap": "Yes (+$3.00)",
    "_mv_option_Delivery Date": "2026-04-15",
    "_mv_option_Logo Upload": "https://sapp.multivariants.com/shop/example/usr/uploads/1711371000_logo.png",
    "_mv_option_Extra Topping": "Cheese (+$1.50), Mushroom (+$2.00)"
  }
}
```

**Property naming convention:** `_mv_option_{Option Title}`
- Prefix `_mv_option_` prevents collision with other properties
- Visible in Shopify order details, cart page, and checkout

#### 3.2.5 Price Adjustment Behavior

- Price adjustments are **added to the variant price** in the total calculation
- Total = (Variant Price * Quantity) + Sum(Option Price Adjustments * Quantity)
- The total price display updates in real-time as options are selected
- Price adjustments are per-item (multiplied by quantity)
- For checkbox options, multiple selections sum their adjustments
- **Note:** Price adjustments are display-only on the storefront. Actual price changes must be enforced via Cart Transform Function or draft orders (Phase 2).

### 3.3 Data Storage

#### New Fields in Ruleset JSON (`/shop/{alias}/usr/set/{id}.json`)

```json
{
  "...existing fields...",

  "productOptions": [
    {
      "id": "opt_1001",
      "type": "text",
      "title": "Engraving Text",
      "shortDescription": "",
      "helpText": "Enter up to 20 characters",
      "required": true,
      "sortOrder": 0,
      "config": {
        "placeholder": "Enter text here",
        "maxCharacters": 20,
        "priceAdjustment": 5.00
      }
    },
    {
      "id": "opt_1002",
      "type": "checkbox",
      "title": "Extra Toppings",
      "shortDescription": "",
      "helpText": "",
      "required": false,
      "sortOrder": 1,
      "config": {
        "choices": [
          { "id": "ch_1", "title": "Cheese", "price": 1.50, "default": false },
          { "id": "ch_2", "title": "Mushroom", "price": 2.00, "default": false },
          { "id": "ch_3", "title": "Peppers", "price": 1.00, "default": true }
        ]
      }
    },
    {
      "id": "opt_1003",
      "type": "file_attachment",
      "title": "Upload Logo",
      "shortDescription": "Upload your logo for printing",
      "helpText": "Accepted: JPG, PNG, PDF. Max 10MB.",
      "required": true,
      "sortOrder": 2,
      "config": {
        "allowedTypes": ["jpg", "png", "pdf"],
        "maxFileSizeMB": 10,
        "maxFiles": 1,
        "uploadInstructions": "Drag and drop or click to upload",
        "priceAdjustment": 0
      }
    },
    {
      "id": "opt_1004",
      "type": "date_time",
      "title": "Delivery Date",
      "shortDescription": "",
      "helpText": "Select your preferred delivery date",
      "required": true,
      "sortOrder": 3,
      "config": {
        "dateFormat": "MM/DD/YYYY",
        "includeTime": false,
        "minDate": null,
        "maxDate": null,
        "disabledDays": ["Sun"],
        "priceAdjustment": 0
      }
    }
  ]
}
```

#### New Feature Flag

In `/pate/config/package_items.php`:
```php
"product_options" => ["standard_monthly", "standard_yearly", "professional_monthly", "professional_yearly"]
```

### 3.4 API Changes

#### Modified Endpoint: Product Data (`/gate/out/usr/data/product/`)

Add `productOptions` to the response:

```json
{
  "multivariants": {
    "set": { "...existing..." },
    "product": { "...existing..." },
    "notice": { "...existing..." },
    "productOptions": [
      {
        "id": "opt_1001",
        "type": "text",
        "title": "Engraving Text",
        "helpText": "Enter up to 20 characters",
        "required": true,
        "config": { "placeholder": "Enter text here", "maxCharacters": 20, "priceAdjustment": 5.00 }
      }
    ]
  }
}
```

#### New Endpoint: File Upload

```
POST /gate/out/usr/data/upload/index.php

Request: multipart/form-data
  - file: uploaded file
  - shop: shop URL
  - option_id: option ID (for validation)

Response (success):
{
  "status": "success",
  "url": "https://sapp.multivariants.com/shop/example/usr/uploads/1711371000_logo.png",
  "filename": "logo.png",
  "size": 245760
}

Response (error):
{
  "status": "error",
  "message": "File size exceeds 10MB limit"
}
```

---

## 4. UI/UX Specifications

### 4.1 Admin UI

**Reference:** See attached screenshots showing:
1. Product Options tab on the create/edit ruleset page (tab alongside Advanced restrictions)
2. "Add option" modal with option type list
3. Expanded checkbox option configuration with choices table

**Key Design Patterns (from screenshots):**
- Tab-based navigation: "Advanced restrictions" | "Product options"
- Collapsible cards per option (collapsed: type name + description + action icons; expanded: all fields)
- Action icons per option card: Delete (trash), Settings (gear), Collapse/Expand (chevron)
- "Add attribute" button inside choice tables
- "+ Add more options" button at the bottom of the options list
- Empty state: dashed border box with "+ Add option" centered

### 4.2 Storefront UI

- Options render as a vertical stack below the variant table
- Each option is a labeled form field
- Match the store's theme typography and spacing
- File upload: dashed border drop zone with icon
- Date picker: native browser date input or custom calendar widget
- Swatches: inline flex layout matching MultiVariants' existing swatch style

### 4.3 Interaction States

| Element | Default | Active | Error | Disabled |
|---------|---------|--------|-------|----------|
| Text input | Empty with placeholder | Typing, blue border | Red border + error msg | Grayed out |
| Dropdown | "Select..." placeholder | Open, highlighted option | Red border + error msg | Grayed out |
| Checkbox | Unchecked | Checked (blue) | Red border on group | Grayed out |
| File upload | Dashed border + icon | Dragging over (blue border) | Red border + error msg | Grayed out |
| Date picker | Empty | Calendar open | Red border + error msg | Grayed out |

---

## 5. Technical Considerations

### 5.1 Files to Modify

**Admin:**
- `gate/out/sys/adm/index.php` - Add "Product options" tab UI, option type modal, option config forms
- `gate/out/sys/adm/init.js` - Add JS for option CRUD, modal, drag-reorder, choice table management

**Backend:**
- `gate/out/adm/action/update_rule_set.php` - Handle `productOptions` array in ruleset save
- `gate/out/usr/data/product/index.php` - Include `productOptions` in response
- NEW: `gate/out/usr/data/upload/index.php` - File upload endpoint

**Storefront:**
- `extensions/theme-app-extension/assets/mv.js` - Render options, handle validation, include in cart add
- `extensions/theme-app-extension/blocks/app-embed.liquid` - May need updates for file upload

**Config:**
- `pate/config/package_items.php` - Add `product_options` feature flag

### 5.2 Dependencies

- No dependencies on other unreleased features
- Does NOT affect existing restriction system
- Compatible with all display layouts (list, dropdown, swatch, grid)
- Works alongside existing custom comment field (but supersedes it for Professional plan)

### 5.3 Cart Integration

- Options are added as line item properties (no Cart Validation Function changes needed for Phase 1)
- Price adjustments are display-only in Phase 1 (actual price enforcement via Cart Transform in Phase 2)
- File upload URLs are included as line item properties

### 5.4 Translation Keys

```
ProductOptionsTitle         = "Product Options"
ProductOptionsRequired      = "Required"
ProductOptionsOptional      = "Optional"
ProductOptionsFileUpload    = "Drag and drop or click to upload"
ProductOptionsFileRemove    = "Remove"
ProductOptionsFileUploading = "Uploading..."
ProductOptionsFileError     = "Upload failed. Please try again."
ProductOptionsDateSelect    = "Select date"
ProductOptionsChoose        = "Choose an option"
ProductOptionsPriceAdd      = "+{currency}{price}"
ProductOptionsValidationReq = "{option_name} is required"
ProductOptionsValidationMax = "{option_name} must be {max} characters or less"
ProductOptionsValidationFile= "File must be {types} and under {size}MB"
```

---

## 6. Edge Cases & Constraints

- **Max options per ruleset:** 20 (prevent performance issues)
- **Max choices per dropdown/radio/checkbox:** 50
- **Max file size:** 50MB hard limit (configurable by merchant up to this limit)
- **File storage cleanup:** Cron job or webhook to delete uploads older than 7 days
- **Concurrent uploads:** Allow 1 upload at a time per option field
- **Option title collision:** If two options have the same title, append " (2)" to the cart property name
- **Products without variants:** Options still work (they're per-ruleset, not per-variant)
- **Mobile responsiveness:** All option inputs must be touch-friendly and full-width on mobile
- **Price adjustments with bundles:** Price adjustments apply per-unit within the bundle quantity
- **Draft orders / Shopify POS:** Line item properties are visible but price adjustments may not apply

---

## 7. Acceptance Criteria

### Admin
- [ ] "Product options" tab visible on create/edit ruleset page (Standard + Professional plans)
- [ ] "Add option" modal shows all 10 option types
- [ ] Can add, configure, reorder, and delete options
- [ ] Each option type has its specific configuration fields
- [ ] Choice-based options (dropdown, radio, checkbox, swatches) have repeatable attribute rows
- [ ] File upload option supports allowed types and max size configuration
- [ ] Saving the ruleset persists all option data in the ruleset JSON
- [ ] Options tab is locked/hidden for Starter plan

### Storefront
- [ ] Custom options render below variant table, above Add to Cart
- [ ] Required options prevent Add to Cart if not filled
- [ ] Validation messages show inline for each field
- [ ] File upload works (drag-drop and click-to-browse)
- [ ] Date picker works with configured format and constraints
- [ ] Price adjustments update the total price display in real-time
- [ ] All option values are added as cart line item properties
- [ ] Options are visible on the cart page and in order details

### Data
- [ ] Options are stored in ruleset JSON under `productOptions` array
- [ ] Product data API includes options in response
- [ ] Uploaded files are stored and accessible via URL
- [ ] File cleanup removes uploads older than 7 days

---

## 8. Attachments

- [Screenshot 1] Admin: Product options tab with empty state and "Add option" button
- [Screenshot 2] Admin: "Add option" modal showing all 10 option types
- [Screenshot 3] Admin: Expanded checkbox option showing configuration fields and choice table
- [Feature Brief PDF] Pages covering existing custom comment field (for reference)

---

## 9. Phased Rollout (Suggested)

### Phase 1 (This PRD)
- All 10 option types in admin
- Storefront rendering and validation
- Cart line item properties
- File upload (basic)
- Price adjustments (display only)

### Phase 2 (Future)
- Price adjustments enforced via Cart Transform Function
- Conditional options (show option B only if option A = "Yes")
- Option templates (save/reuse option sets across rulesets)
- Analytics on option usage

### Phase 3 (Future)
- Option-based inventory tracking
- Option-based variant creation (dynamic Shopify variants)
- Bulk import/export of option configurations
