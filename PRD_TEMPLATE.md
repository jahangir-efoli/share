# PRD Template for MultiVariants Feature Development

Use this template when requesting new features. Copy this file, rename it `PRD_XXX_FEATURE_NAME.md`, and fill in all sections. The more detail you provide, the faster and more accurate the implementation will be.

---

```markdown
# PRD-XXX: [Feature Name]

## 1. Overview
**Feature Name:** [Name]
**Priority:** [P0 Critical | P1 High | P2 Medium | P3 Low]
**Plan Tier:** [Starter | Standard | Professional | All Plans]
**Estimated Scope:** [Small (1-2 days) | Medium (3-5 days) | Large (1-2 weeks)]

### 1.1 Problem Statement
What problem does this solve for merchants? Why do they need this?

### 1.2 Goal
One sentence describing the desired outcome.

### 1.3 Success Metrics (Optional)
How do we measure if this feature is successful?

---

## 2. User Stories

- As a **merchant**, I want to [action], so that [outcome].
- As a **customer**, I want to [action], so that [outcome].

---

## 3. Functional Requirements

### 3.1 Admin Panel (What the merchant configures)

Describe every UI element, input field, dropdown, toggle, and button the merchant will interact with.

For each element provide:
- **Element type:** text input / number input / dropdown / toggle / checkbox / color picker / file upload / modal / etc.
- **Label:** What the user sees
- **Default value:** What it defaults to
- **Validation:** Min/max, required/optional, allowed formats
- **Behavior:** What happens when the user interacts with it
- **Plan gating:** Which plans can access this (if restricted)

### 3.2 Storefront (What the customer sees)

Describe exactly how this feature appears on the product page:
- Where it renders (position relative to existing elements)
- Visual appearance (layout, sizing, responsive behavior)
- Interaction behavior (click, input, select, upload)
- Validation and error messages
- How it integrates with the existing variant selector and Add to Cart flow

### 3.3 Data Storage

What new data fields need to be stored? Provide:
- Field name
- Data type (string, number, boolean, array, object)
- Where it's stored (ruleset JSON, settings JSON, new file, metafield)
- Default value

### 3.4 API Changes

Any new or modified API endpoints:
- Endpoint path and method
- Request parameters
- Response structure
- Which existing endpoints need modification

---

## 4. UI/UX Specifications

### 4.1 Admin UI

Either provide:
- **Figma designs** (preferred) - link or exported images
- **Written wireframe** - describe layout, positioning, element hierarchy
- **Reference screenshots** - from similar apps showing the desired pattern

### 4.2 Storefront UI

Either provide:
- **Figma designs** (preferred)
- **Written wireframe** with exact positioning
- **Reference screenshots** from competitor apps

### 4.3 Interaction States

Describe all states for each UI element:
- Default / Empty state
- Active / Filled state
- Error state
- Disabled state (plan gating)
- Loading state (if async)
- Mobile responsive behavior

---

## 5. Technical Considerations

### 5.1 Files to Modify
List known files that need changes (the AI will validate and expand this list):
- Admin: `gate/out/sys/adm/index.php`, `gate/out/sys/adm/init.js`
- Backend: new action PHP file, existing endpoints
- Storefront: `mv.js` or new JS files
- Extensions: theme extension Liquid files (if needed)
- Config: `package_items.php` (if plan-gated)

### 5.2 Dependencies
- Does this feature depend on other features?
- Does it affect existing features?
- Are there breaking changes?

### 5.3 Cart Integration
- Does this feature affect the cart (line item properties)?
- Does the Cart Validation Function need updates?
- Does the Cart Transform Function need updates?

### 5.4 Translation Keys
List new translation keys needed for the storefront.

---

## 6. Edge Cases & Constraints

- What happens when [edge case]?
- Maximum limits (max options, max file size, etc.)
- Browser compatibility requirements
- Performance considerations

---

## 7. Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

---

## 8. Attachments

- [ ] Figma designs or screenshots
- [ ] Example from competitor apps (if applicable)
- [ ] Sample data or test cases
```

---

## What Works Best for AI Development

### Figma Designs vs Written Documentation

| Input Type | Effectiveness | Best For |
|------------|---------------|----------|
| **Written PRD (this template)** | Best | Complex business logic, restrictions, data models, API design |
| **Figma Designs / Screenshots** | Great | UI layout, visual positioning, component hierarchy |
| **Both Combined** | Ideal | Full feature implementation |

### Recommendation

For **best results**, provide:
1. **Written PRD** (this template) for all business logic, data storage, validation rules, and API contracts
2. **Screenshots or Figma exports** for UI layout and visual design
3. **Example data** showing expected inputs and outputs

### Why Written PRDs Are Essential (Even With Figma)

Figma designs show WHAT it looks like, but not:
- What data gets stored and where
- Validation rules (min/max, required, format)
- How it interacts with existing features (rulesets, plans, cart)
- Edge cases (what if the merchant has 100 options?)
- Plan gating (which plans get this feature?)
- Translation requirements
- Cart/checkout integration

The AI needs both the visual reference AND the written specification to build the feature correctly.
