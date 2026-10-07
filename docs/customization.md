---
type: Guide
title: "Contact Form: Customization Guide"
description: "How to change the contact form's colors and width, and suggested next steps."
tags: [contact-form, html, css, customization]
status: draft
generated: { by: human:Easin, at: 2026-10-07T07:02:33Z }
---

# Contact Form: Customization Guide

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
