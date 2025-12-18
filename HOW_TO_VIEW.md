# How to View Architecture Diagrams

## Quick Solution (Recommended)

**Simply open `ARCHITECTURE_VIEWER.html` in any web browser!**

1. Navigate to the project folder
2. Double-click on `ARCHITECTURE_VIEWER.html`
3. All diagrams will render automatically in your browser

This is the easiest way to view all the diagrams properly rendered.

---

## Alternative Methods

### Method 1: VS Code with Extension

1. Install the **"Markdown Preview Mermaid Support"** extension in VS Code
2. Open `ARCHITECTURE.md`
3. Press `Cmd+Shift+V` (Mac) or `Ctrl+Shift+V` (Windows) to open preview
4. Diagrams will render in the preview pane

**Extension to install:**
- Search for: `bierner.markdown-mermaid` or `bierner.markdown-preview-mermaid-support`

### Method 2: Online Mermaid Editor

1. Go to [https://mermaid.live](https://mermaid.live)
2. Copy any diagram code from `ARCHITECTURE.md` (the code between ` ```mermaid` and ` ``` `)
3. Paste it into the editor
4. View the rendered diagram

### Method 3: GitHub

1. Push the `ARCHITECTURE.md` file to a GitHub repository
2. GitHub natively supports Mermaid diagrams
3. View the file on GitHub to see all diagrams rendered

### Method 4: Other Markdown Viewers

- **Obsidian**: Supports Mermaid natively
- **Typora**: Supports Mermaid diagrams
- **Mark Text**: Supports Mermaid diagrams
- **GitLab**: Supports Mermaid in markdown files

---

## File Structure

```
ShaadiApp/
├── ARCHITECTURE.md          # Markdown file with Mermaid diagrams
├── ARCHITECTURE_VIEWER.html  # HTML file - Open in browser (EASIEST!)
└── HOW_TO_VIEW.md           # This file
```

---

## Troubleshooting

**If diagrams don't show in HTML file:**
- Make sure you have internet connection (diagrams load from CDN)
- Try a different browser (Chrome, Firefox, Safari, Edge)
- Check browser console for errors (F12)

**If diagrams don't show in VS Code:**
- Make sure the extension is installed and enabled
- Try reloading VS Code window (`Cmd+R` or `Ctrl+R`)
- Check if the extension is compatible with your VS Code version

---

## Recommended Approach

**Use `ARCHITECTURE_VIEWER.html`** - It's the simplest and most reliable way to view all diagrams without any setup!

