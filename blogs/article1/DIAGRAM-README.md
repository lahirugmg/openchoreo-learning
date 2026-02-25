# OpenChoreo Architecture Diagrams

This directory contains the architecture diagrams for the OpenChoreo local setup.

## Files

### Diagram Source Files

1. **architecture-diagram.mmd**
   - Basic Mermaid diagram without styling
   - Clean version for documentation
   - ~60 lines

2. **architecture-diagram-enhanced.mmd**
   - Enhanced version with:
     - Color-coded planes (Control, Data, Build, Observability)
     - Emojis for visual clarity
     - Numbered flow arrows (① → ⑪)
     - Custom theme with better colors
   - Recommended for presentations
   - ~100 lines

### Generated PNG Files

1. **architecture-diagram.png**
   - Generated from `architecture-diagram.mmd`
   - Size: 175KB (1251 x 1441 pixels)
   - Transparent background
   - Basic styling

2. **architecture-diagram-enhanced.png**
   - Generated from `architecture-diagram-enhanced.mmd`
   - Size: 184KB (2800 x 2000 pixels)
   - White background
   - Full colors and emojis
   - **Recommended for Medium article**

## How to Regenerate

### Prerequisites

Install Mermaid CLI:
```bash
npm install -g @mermaid-js/mermaid-cli
```

### Generate Basic Diagram

```bash
mmdc -i architecture-diagram.mmd \
     -o architecture-diagram.png \
     -b transparent \
     -w 2400 \
     -H 1800
```

### Generate Enhanced Diagram

```bash
mmdc -i architecture-diagram-enhanced.mmd \
     -o architecture-diagram-enhanced.png \
     -b white \
     -w 2800 \
     -H 2000
```

### Parameters Explained

- `-i` : Input Mermaid file
- `-o` : Output PNG file
- `-b` : Background color (transparent/white/color)
- `-w` : Width in pixels
- `-H` : Height in pixels

### Custom Sizes

For social media:
```bash
# Twitter/X card (1200x675)
mmdc -i architecture-diagram-enhanced.mmd -o twitter-card.png -b white -w 1200 -H 675

# LinkedIn post (1200x627)
mmdc -i architecture-diagram-enhanced.mmd -o linkedin-post.png -b white -w 1200 -H 627

# Instagram post (1080x1080)
mmdc -i architecture-diagram-enhanced.mmd -o instagram-post.png -b white -w 1080 -H 1080

# GitHub README (800x600)
mmdc -i architecture-diagram-enhanced.mmd -o github-readme.png -b transparent -w 800 -H 600
```

## Online Editing

You can also edit and preview these diagrams online:

1. **Mermaid Live Editor**: https://mermaid.live/
   - Copy the content from `.mmd` files
   - Edit and preview in real-time
   - Download as PNG/SVG

2. **VS Code Extension**:
   ```bash
   code --install-extension bierner.markdown-mermaid
   ```
   - Preview Mermaid diagrams in VS Code
   - Ctrl+Shift+V to preview

## Diagram Components

### Control Plane (Blue)
- **Port**: 8080/8443
- **Components**:
  - 🎭 Backstage Console
  - 🔐 Thunder (Identity)
  - 🔌 OpenChoreo API
  - ⚙️ Controller Manager
  - 🌉 Cluster Gateway

### Data Plane (Purple)
- **Port**: 19080/19443
- **Components**:
  - 🤖 Cluster Agent
  - 🚀 Your Applications
  - 🔀 Gateway

### Build Plane (Orange) - Optional
- **Port**: 10081/10082
- **Components**:
  - 🔄 Argo Workflows
  - 📦 Container Registry

### Observability Plane (Teal) - Optional
- **Port**: 11080-11085
- **Components**:
  - 👁️ Observer API
  - 🔍 OpenSearch
  - 📈 Prometheus

## Flow Explanation

The numbered arrows show the typical flow:

1. **① Developer Access**: User accesses console via browser
2. **② Resource Management**: API/Controller manages K8s resources
3. **③ Secure Connection**: Data Plane agent connects via WebSocket
4. **④ Workload Creation**: Agent creates application workloads
5. **⑤ End User Access**: Users access deployed applications
6. **⑥-⑦ Traffic Routing**: Gateway routes to applications
7. **⑧-⑩ Build Pipeline**: Optional CI/CD workflow
8. **⑪ Observability**: Apps send metrics/logs

## Customization Tips

### Change Colors

In the enhanced diagram, modify the `%%{init: ...}%%` section:

```javascript
'primaryColor':'#1976d2',        // Control plane color
'secondaryColor':'#f3e5f5',      // Data plane color
'tertiaryColor':'#e8f5e9',       // Apps color
```

### Add More Components

```mermaid
ComponentName["🔧 Display Name<br/><small>Description</small>"]
```

### Change Layout

- `graph TB` - Top to Bottom (current)
- `graph LR` - Left to Right
- `graph BT` - Bottom to Top
- `graph RL` - Right to Left

Example:
```mermaid
graph LR  # Change to left-to-right layout
    ...
```

## Usage in Medium Article

The enhanced diagram is already referenced in `medium-article-local-setup.md`:

```markdown
![OpenChoreo Multi-Plane Architecture](architecture-diagram-enhanced.png)
```

When publishing to Medium:
1. Upload `architecture-diagram-enhanced.png` to Medium's image uploader
2. Update the image URL in the article
3. Keep the Mermaid code in the collapsible section for readers who want to customize

## License

These diagrams are part of the OpenChoreo learning materials and follow the same license as the main OpenChoreo project.

## Contributing

To improve these diagrams:
1. Edit the `.mmd` files
2. Regenerate PNGs
3. Test in the Medium article
4. Submit a PR with both source and generated files
