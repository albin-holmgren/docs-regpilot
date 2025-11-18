# 🎨 Custom Styling Configuration

## ✅ Applied Changes

### Background Colors
Successfully configured custom background colors in `mint.json`:

**Light Mode:**
- Background: `#F7F7F4` ✅

**Dark Mode:**
- Background: `#14120B` ✅

### Layout
- ✅ Set to `sidenav` for full-width sidebar layout

---

## ⚠️ Sidebar Colors - Requires Additional Setup

The sidebar colors you requested:
- Light mode: `#F0EFEA`
- Dark mode: `#1B1913`

**Status:** Created `custom.css` with the styling, but Mintlify's standard configuration doesn't support custom CSS files directly.

### Solutions:

#### Option 1: Contact Mintlify Support (Recommended)
Mintlify Enterprise plans support custom CSS. Contact them to enable:
- Email: support@mintlify.com
- Request: Enable custom CSS for sidebar colors

#### Option 2: Use Mintlify's Built-in Themes
Mintlify has predefined themes that may include sidebar customization:
```json
{
  "theme": "venus" // or "quill", "prism"
}
```

#### Option 3: Wait for Deployment
The `custom.css` file is in your repo. When Mintlify adds support for custom stylesheets, it will automatically apply.

---

## 📄 Current mint.json Configuration

```json
{
  "colors": {
    "primary": "#27272a",
    "light": "#3f3f46",
    "dark": "#18181b",
    "background": {
      "light": "#F7F7F4",  // ✅ Your custom light bg
      "dark": "#14120B"     // ✅ Your custom dark bg
    }
  },
  "layout": "sidenav"  // ✅ Full-width sidebar
}
```

---

## 🎨 custom.css File (Ready for Use)

The file is created and ready. It contains:

```css
/* Light mode sidebar */
[data-theme="light"] aside[class*="sidebar"] {
  background-color: #F0EFEA !important;
}

/* Dark mode sidebar */
[data-theme="dark"] aside[class*="sidebar"] {
  background-color: #1B1913 !important;
}

/* Full width layout */
main {
  max-width: 100% !important;
}

/* Sidebar to corner */
aside[class*="sidebar"] {
  padding-left: 0 !important;
  margin-left: 0 !important;
}
```

---

## 🚀 What's Live Now

After deployment (~2-3 minutes):
- ✅ Background color: `#F7F7F4` (light) / `#14120B` (dark)
- ✅ Full-width sidebar layout
- ⏳ Sidebar colors: Pending Mintlify support

---

## 📞 Next Steps

1. **Test the background colors** at https://docs.regpilot.dev
2. **For sidebar colors**, contact Mintlify support:
   - Mention you have a `custom.css` file ready
   - Request custom CSS enablement
   - Reference your sidebar color requirements

---

## 🔧 Alternative: Browser Extension

As a temporary workaround, you can use a browser extension like "Stylus" to inject the custom CSS locally for preview purposes.

---

**Summary:** Background colors are live! Sidebar colors need Mintlify support enablement.
