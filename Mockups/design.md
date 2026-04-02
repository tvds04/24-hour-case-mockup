# Design System Document: Industrial Editorial

## 1. Overview & Creative North Star

### The Creative North Star: "Precision Brutalism"
This design system moves beyond the standard "construction company" template. We are shifting from a generic industrial look to a high-end editorial experience that honors the raw, structural integrity of concrete while utilizing sophisticated digital-first techniques.

The system is defined by **Precision Brutalism**: an aesthetic that combines the heavy, sharp-edged nature of the industrial sector with the refined clarity of modern editorial design. We reject "softness" (rounded corners) in favor of aggressive geometric precision. We reject "clutter" (divider lines) in favor of tonal layering and monumental typography. By leveraging intentional asymmetry—specifically through the use of high-contrast diagonal sectioning—we create a sense of forward momentum and structural strength.

---

## 2. Colors

The palette is anchored by the structural depth of `secondary` (#5F5E5E) and `on_surface` (#1A1C1C), punctuated by the high-visibility `primary` (#006780) and `primary_container` (#00AFD7).

### The "No-Line" Rule
**Explicit Instruction:** Do not use 1px solid borders to define sections or containers.
Traditional lines create visual noise that detracts from the "monolithic" feel of the brand. Boundaries must be defined through:
- **Background Shifts:** Use `surface_container_low` sections transitioning into `surface` sections.
- **Tonal Contrast:** Place a `surface_container_highest` element directly against a `primary` background.

### Surface Hierarchy & Nesting
Treat the UI as a series of stacked, physical materials.
- **Base Level:** `background` (#F9F9F9) or `surface`.
- **In-Page Sections:** Use `surface_container_low` for content blocks that need subtle distinction.
- **Elevated Interactive Elements:** Use `surface_container_lowest` (pure white) to create a "lifted" effect for cards sitting on a grey foundation.

### The "Glass & Gradient" Rule
To add visual "soul," primary action areas and Hero sections should utilize the **Industrial Gradient**: a subtle shift from `primary` (#006780) to `primary_container` (#00AFD7) at a 135-degree angle. For floating overlays or navigation menus, apply **Glassmorphism**: use `surface` with 80% opacity and a `20px` backdrop-blur to maintain structural depth while allowing the rich imagery of heavy machinery and texture to bleed through.

---

## 3. Typography

The typography strategy relies on the contrast between the authoritative, geometric **Inter** and the utilitarian, approachable **Public Sans**.

* **Display & Headlines (Inter):** These are our "Structural Beams." Use `display-lg` (3.5rem) with tight letter-spacing (-0.02em) to create impact. These should always be set in Bold or Black weights to mimic the weight of the brand's logo.
* **Titles (Public Sans):** These act as the organizational labels for our technical data. They provide a high-readability bridge between the "monumental" headlines and the "functional" body text.
* **Body & Labels (Public Sans):** Executed in `body-md` (0.875rem), the body text provides the "Specifications." It is clean, professional, and stays out of the way of the structural hierarchy.

---

## 4. Elevation & Depth

In a system defined by sharp edges and 0px radii, elevation must be handled with extreme delicacy to avoid looking "dated."

* **The Layering Principle:** Depth is achieved by stacking `surface-container` tiers.
* *Example:* A data table (`surface_container_highest`) sitting inside a workspace (`surface_container_low`).
* **Ambient Shadows:** For floating elements like modals or tooltips, use "Atmospheric Shadows."
* **Shadow Value:** `0px 20px 40px rgba(26, 28, 28, 0.06)`.
* The shadow is wide, soft, and uses the `on_surface` tint to ensure it feels like a natural part of the environment, not a digital effect.
* **The "Ghost Border" Fallback:** If a container requires definition against an identical background, use a **Ghost Border**: `outline_variant` (#BCC8CE) at 15% opacity.
* **Asymmetric Slants:** To mirror the "Slanted" brand identity, use diagonal CSS clips on section backgrounds. A 3-5 degree variance creates energy and breaks the "boxed" layout common in junior designs.

---

## 5. Components

### Buttons (The "I-Beam" Style)
* **Radius:** Always `0px`.
* **Primary:** Background: `primary` (#006780); Text: `on_primary` (#FFFFFF). Use a `4px` left-accent border of `primary_container` on hover.
* **Secondary:** Background: `transparent`; Border: `Ghost Border` (2px).
* **Tertiary:** Text-only with a sharp arrow icon (→).

### Input Fields
* **Style:** Underline-only or solid-fill using `surface_container_high`.
* **Focus:** Transition the background to `surface_container_highest` and change the bottom 2px border to `primary`. No rounded corners.

### Cards & Lists
* **Prohibition:** Never use divider lines between list items.
* **Separation:** Use `spacing-6` (1.5rem) of vertical white space or alternating backgrounds between `surface` and `surface_container_low`.
* **Diagonal Accents:** Use the slanted divider pattern (from the Hero reference) as a small decorative "tag" in the top-right corner of cards to indicate status or category.

### Industrial Chips
* **Design:** Rectangular (`0px` radius).
* **Color:** `secondary_container` with `on_secondary_container` text. For "Action" states, use `primary_fixed`.

---

## 6. Do’s and Don’ts

### Do:
* **DO** use extreme vertical spacing (`spacing-20` and `spacing-24`) between major sections to let the "monolithic" typography breathe.
* **DO** use high-quality, high-contrast photography of concrete textures and construction sites as background layers behind "Glass" containers.
* **DO** align all text to a rigid grid. Asymmetry should come from background "slants," not from chaotic text placement.

### Don't:
* **DON'T** use a single pixel of border-radius. Every component must be sharp.
* **DON'T** use generic "Grey" shadows. Always tint your shadows with the `on_surface` color to maintain tonal depth.
* **DON'T** use center-alignment for long-form content. Stick to strong left-aligned "Editorial" layouts to maintain a professional, technical feel.
* **DON'T** allow "float" without purpose. Every elevated element must be a deliberate layer in the surface hierarchy.