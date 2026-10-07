---
type: Guide
title: "Contact Form: Customization Guide"
description: "How to customize the contact form: colors, width and next steps."
owner: human:EasinTanvir
status: stable
generated: { by: "human:EasinTanvir", at: "2026-10-07" }
verified:
  - { by: "human:EasinTanvir", at: "2026-10-07" }
tags: [contact-form, html, css, customization, how-to]
---

The `for` attribute on the label must match the input's `id`.

## Change the colors

All colors are in the `<style>` block of `form.html`:

| Color     | Used for                      |
| --------- | ----------------------------- |
| `#4a7bd0` | Button and input focus border |
| `#3a64ad` | Button hover                  |
| `#f2f4f8` | Page background               |

Replace these values to match your brand.

## Change the width

Edit `max-width: 400px;` in the `form` rule.

## Next steps

- Add JavaScript to show a success message after submitting.
- Add server-side validation. Browser validation can be bypassed.
- Add spam protection, such as a honeypot field or a CAPTCHA.
