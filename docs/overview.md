---
type: Reference
title: "Contact Form: Overview"
description: "Fields, layout, styling and known limitation of the single-file HTML/CSS contact form."
tags: [contact-form, html, css, reference]
status: stable
generated: { by: human:Easin, at: 2026-10-07T08:30:08Z }
verified:
  - { by: human:Easin, at: 2026-10-07T08:31:29Z }
---

# Contact Form: Overview

`form.html` is a single-file contact form built with plain HTML and CSS. It has no JavaScript and no dependencies.

## Fields

| Field       | Element      | Type  | Required |
| ----------- | ------------ | ----- | -------- |
| Name        | `<input>`    | text  | yes      |
| Email       | `<input>`    | email | yes      |
| Message     | `<textarea>` | n/a   | yes      |
| LongMessage | `<textarea>` | n/a   | yes      |

The browser validates the required fields and the email format before the form submits.

## Layout and styling

- The form is centered on the page with flexbox.
- It is at most 400px wide and shrinks on small screens.
- Styles are in a `<style>` block in the page head.
- Inputs get a blue border on focus, and the button darkens on hover.

## Opening the form

Double-click `form.html` to open it in any browser. No server is needed.

## Known limitation

The form's `action` is `#`, so submitting it does not send the data anywhere. See [customization.md](customization.md) for how to connect it to a backend.
