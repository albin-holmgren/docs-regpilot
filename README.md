# RegPilot Documentation

Official documentation for RegPilot - the EU AI Act compliance management platform.

## 📚 About

This repository contains the comprehensive documentation for RegPilot, covering:

- **Getting Started**: Quick guides to get up and running
- **Core Features**: Detailed documentation of all platform features
- **AI Gateway**: API proxy for compliance validation
- **EU AI Act Compliance**: Complete regulatory compliance guides
- **API Reference**: Full API documentation with examples
- **Integration Guides**: Framework-specific integration tutorials

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ installed
- npm or yarn package manager

### Local Development

1. Install the [Mintlify CLI](https://www.npmjs.com/package/mintlify):

```bash
npm i -g mintlify
```

2. Navigate to the docs directory:

```bash
cd docs-regpilot
```

3. Start the development server:

```bash
mintlify dev
```

4. Open [http://localhost:3000](http://localhost:3000) in your browser

The documentation will automatically reload when you make changes to any `.mdx` files.

## 📖 Documentation Structure

```
docs-regpilot/
├── introduction.mdx              # Welcome page
├── quickstart.mdx                # 5-minute quickstart guide
├── installation.mdx              # Installation instructions
├── authentication.mdx            # API authentication guide
├── features/                     # Core features documentation
│   ├── overview-dashboard.mdx
│   ├── compliance-management.mdx
│   ├── violations-tracking.mdx
│   ├── analytics.mdx
│   ├── models-registry.mdx
│   ├── policies.mdx
│   └── alerts.mdx
├── ai-gateway/                   # AI Gateway documentation
│   ├── introduction.mdx
│   ├── api-keys.mdx
│   ├── governor.mdx
│   └── usage-tracking.mdx
├── compliance/                   # EU AI Act compliance guides
│   ├── eu-ai-act-overview.mdx
│   ├── risk-classification.mdx
│   ├── documentation-requirements.mdx
│   ├── prohibited-practices.mdx
│   └── transparency-obligations.mdx
├── api-reference/                # API documentation
│   ├── introduction.mdx
│   ├── authentication.mdx
│   ├── errors.mdx
│   └── ...
└── guides/                       # Integration & best practices
    ├── nextjs-integration.mdx
    ├── react-integration.mdx
    ├── security-best-practices.mdx
    └── ...
```

## ✏️ Contributing

### Adding New Pages

1. Create a new `.mdx` file in the appropriate directory
2. Add frontmatter with title and description:

```mdx
---
title: "Page Title"
description: "Page description for SEO"
---

## Your Content Here
```

3. Add the page to `docs.json` navigation:

```json
{
  "group": "Group Name",
  "pages": [
    "path/to/your/page"
  ]
}
```

### Writing Guidelines

- Use clear, concise language
- Include code examples for technical concepts
- Add visual aids (screenshots, diagrams) where helpful
- Follow the existing page structure and formatting
- Test all code examples before committing

### Mintlify Components

We use Mintlify's built-in components for rich documentation:

```mdx
<Card title="Card Title" icon="icon-name" href="/link">
  Card content
</Card>

<CodeGroup>
```bash
# Code example
```
</CodeGroup>

<Steps>
  <Step title="Step 1">Content</Step>
  <Step title="Step 2">Content</Step>
</Steps>

<AccordionGroup>
  <Accordion title="Question">Answer</Accordion>
</AccordionGroup>

<Warning>Warning message</Warning>
<Note>Note message</Note>
<Tip>Tip message</Tip>
```

## 🚢 Deployment

Documentation is automatically deployed when changes are pushed to the `main` branch via the Mintlify GitHub App.

### Manual Deployment

If needed, you can trigger a manual deployment:

```bash
mintlify deploy
```

## 🔧 Configuration

The `docs.json` file controls:

- **Theme**: Colors and branding
- **Navigation**: Sidebar structure and tabs
- **Anchors**: Global navigation links
- **Logo**: Light and dark mode logos
- **Footer**: Social links and company info

Edit this file to customize the documentation site's appearance and structure.

## 📝 Style Guide

### Headings

- Use `##` for main sections
- Use `###` for subsections
- Use `####` for sub-subsections

### Code Blocks

Always specify the language for syntax highlighting:

````mdx
```typescript
const example = "code";
```
````

### Links

- Use relative links for internal pages: `[Link Text](/path/to/page)`
- Use full URLs for external links: `[Link Text](https://example.com)`

### Images

Store images in the `/images` directory and reference them:

```mdx
<img src="/images/example.png" alt="Description" />
```

## 🐛 Troubleshooting

### Mintlify dev isn't running

Run `mintlify install` to reinstall dependencies:

```bash
mintlify install
```

### Page loads as 404

Ensure you're running the command in the directory containing `docs.json`.

### Changes not appearing

1. Stop the dev server (Ctrl+C)
2. Clear the cache: `mintlify clean`
3. Restart: `mintlify dev`

## 📞 Support

- **Documentation Issues**: [Create an issue](https://github.com/regpilot/docs-regpilot/issues)
- **RegPilot Support**: support@regpilot.com
- **Community**: [GitHub Discussions](https://github.com/regpilot/regpilot/discussions)

## 📄 License

This documentation is licensed under [MIT License](LICENSE).

---

**Built with [Mintlify](https://mintlify.com)** 📖docs.json.

### Resources
- [Mintlify documentation](https://mintlify.com/docs)
