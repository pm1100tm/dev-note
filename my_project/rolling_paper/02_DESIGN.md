---
name: Pastel Memories
colors:
  surface: '#fdf9f3'
  surface-dim: '#dddad4'
  surface-bright: '#fdf9f3'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f7f3ed'
  surface-container: '#f1ede7'
  surface-container-high: '#ebe8e2'
  surface-container-highest: '#e6e2dc'
  on-surface: '#1c1c18'
  on-surface-variant: '#504441'
  inverse-surface: '#31302d'
  inverse-on-surface: '#f4f0ea'
  outline: '#827470'
  outline-variant: '#d4c3be'
  surface-tint: '#76574e'
  primary: '#76574e'
  on-primary: '#ffffff'
  primary-container: '#fad0c4'
  on-primary-container: '#77574e'
  inverse-primary: '#e6beb2'
  secondary: '#3a675c'
  on-secondary: '#ffffff'
  secondary-container: '#bcedde'
  on-secondary-container: '#406d61'
  tertiary: '#3c627c'
  on-tertiary: '#ffffff'
  tertiary-container: '#b7defc'
  on-tertiary-container: '#3c637c'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#ffdbd0'
  primary-fixed-dim: '#e6beb2'
  on-primary-fixed: '#2c160f'
  on-primary-fixed-variant: '#5d4037'
  secondary-fixed: '#bcedde'
  secondary-fixed-dim: '#a1d0c3'
  on-secondary-fixed: '#00201a'
  on-secondary-fixed-variant: '#204e44'
  tertiary-fixed: '#c8e6ff'
  tertiary-fixed-dim: '#a5cbe9'
  on-tertiary-fixed: '#001e2e'
  on-tertiary-fixed-variant: '#234b63'
  background: '#fdf9f3'
  on-background: '#1c1c18'
  surface-variant: '#e6e2dc'
typography:
  display-lg:
    fontFamily: Quicksand
    fontSize: 48px
    fontWeight: '700'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Quicksand
    fontSize: 32px
    fontWeight: '600'
    lineHeight: 40px
  headline-lg-mobile:
    fontFamily: Quicksand
    fontSize: 28px
    fontWeight: '600'
    lineHeight: 36px
  headline-md:
    fontFamily: Quicksand
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-lg:
    fontFamily: Hanken Grotesk
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Hanken Grotesk
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-md:
    fontFamily: Hanken Grotesk
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
    letterSpacing: 0.01em
  label-sm:
    fontFamily: Hanken Grotesk
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  xs: 4px
  sm: 12px
  md: 24px
  lg: 48px
  xl: 80px
  container-max: 1200px
  gutter: 24px
---

## Brand & Style

The brand personality is rooted in warmth, nostalgia, and the gentle preservation of digital sentiment. It aims to evoke the emotional response of receiving a physical handwritten note—intimate, soft, and unhurried.

The design style is a hybrid of **Minimalism** and **Tactile Softness**. It prioritizes heavy whitespace and a restricted pastel palette to create a "breathable" interface that doesn't overwhelm the user. To achieve the requested nostalgia, the system utilizes soft-depth shadows and high-quality typography that feels human and approachable rather than corporate or rigid. The interface should feel like a clean, organized scrap-book of memories.

## Colors

This design system utilizes a palette of low-saturation, high-vibrancy pastels to maintain a cheerful yet calm atmosphere.

- **Primary (Pale Peach):** Used for main actions, active states, and brand highlights. It provides a warm, sun-kissed feel.
- **Secondary (Mint):** Used for success states, secondary accents, and "growth" related elements.
- **Tertiary (Sky Blue):** Used for informational accents and interactive variations.
- **Surface (Warm Cream):** The base background of the application. It is softer on the eyes than pure white, enhancing the nostalgic, paper-like feel.
- **Typography (Deep Soft Gray):** A high-contrast but "matte" gray used to ensure readability while avoiding the clinical sharpness of pure black.

## Typography

The typography strategy pairs the rounded, friendly geometry of **Quicksand** for headings with the clean, legible, and modern proportions of **Hanken Grotesk** for functional text.

- **Headlines:** Use Quicksand to reinforce the brand's soft and welcoming nature. The letter spacing on larger displays is slightly tightened to maintain visual cohesion.
- **Body & UI Elements:** Hanken Grotesk provides a sophisticated neutral balance, ensuring that even in data-dense areas, the interface remains clear and professional.
- **Hierarchy:** Use font weight rather than size alone to distinguish between primary information and secondary metadata.

## Layout & Spacing

This design system uses a **fluid-to-fixed grid** model. On desktop, content is contained within a 1200px max-width container with a 12-column grid. On mobile, the layout shifts to a single column with generous 24px side margins to emphasize the content's importance.

Spacing follows an 8px rhythmic scale. Use `lg` and `xl` spacing for vertical section separation to preserve the "Minimalist" breathing room. Gutters are kept at a consistent 24px to ensure elements have sufficient negative space to feel light and airy.

## Elevation & Depth

To match the "Pastel" aesthetic, depth is achieved through **Ambient Shadows** and **Tonal Layering** rather than harsh borders.

1.  **Low Elevation (Surface-Level):** Elements like input fields and inactive cards use a very thin 1px border in a slightly darker cream tone or a soft mint.
2.  **Medium Elevation (Interactive):** Hover states and active cards utilize a soft, multi-layered shadow with a large blur radius (12px to 20px) and low opacity (5-8%). The shadow color is slightly tinted with the Primary color (#FAD0C4) rather than pure black to keep the look warm.
3.  **High Elevation (Overlays):** Modals and dropdowns use a deeper shadow and a subtle backdrop blur (4px) to draw focus without completely disconnecting from the warm background.

## Shapes

The shape language is defined by **Soft Roundedness**. All standard components (Buttons, Inputs, Cards) use a default radius of 0.5rem (8px).

Larger containers like cards or image wrappers should utilize the `rounded-xl` (24px) setting to create a friendly, "pillow-like" appearance. This high radius is critical to the brand's approachable and non-threatening identity. Avoid sharp corners in all instances, including icons.

## Components

- **Buttons:** Primary buttons use a solid Peach fill with white text. Secondary buttons use a Mint ghost style (border only). All buttons should have a subtle 2px vertical offset shadow to feel "pressable."
- **Cards:** Use a high-radius corner (24px) and a white background against the cream page surface. The shadow should be very soft and diffused. Cards are the primary container for "memories."
- **Lateral Navigation Bar (LNB):** A slim, vertical sidebar with high-transparency backgrounds. Active states are indicated by a soft Sky Blue pill-shaped highlight behind the icon/label.
- **Input Fields:** These should have a subtle cream-to-white gradient fill. On focus, the border transitions to a soft Peach with a 4px outer glow of the same color.
- **Chips:** Used for tags or categories, these should be pill-shaped with light pastel backgrounds and slightly darker text of the same hue (e.g., Mint background with dark teal text).
- **Interactive States:** All transitions should use a slow, "ease-in-out" curve (approx 300ms) to maintain the gentle brand feeling.
