# IDE Integration Guide for Contentstack Live Preview Documentation

This guide explains how developers can integrate these Contentstack Live Preview documentation files into their preferred IDEs and development environments for easy access during implementation.

## Overview

These documentation files are designed to be accessed directly within your development environment, allowing AI coding assistants and developers to reference implementation patterns without leaving their IDE.

## Integration Methods

### 1. VS Code Integration

#### Method A: Workspace Integration

1. **Copy documentation folder** to your project root:

   ```bash
   cp -r /path/to/contentstack-vibe-docs/.contentstack-vibe-docs ./docs/contentstack-live-preview
   ```

2. **Add to workspace settings** (`.vscode/settings.json`):

   ```json
   {
     "files.associations": {
       "*.md": "markdown"
     },
     "markdown.preview.linkify": true,
     "markdown.preview.typographer": true
   }
   ```

3. **Create quick access tasks** (`.vscode/tasks.json`):
   ```json
   {
     "version": "2.0.0",
     "tasks": [
       {
         "label": "Open Live Preview Docs",
         "type": "shell",
         "command": "code",
         "args": ["./docs/contentstack-live-preview/"],
         "group": "build",
         "presentation": {
           "echo": true,
           "reveal": "silent",
           "focus": false,
           "panel": "shared"
         }
       }
     ]
   }
   ```

#### Method B: Extension-Based Access

1. **Install Markdown extensions**:

   - `yzhang.markdown-all-in-one`
   - `shd101wyy.markdown-preview-enhanced`

2. **Create bookmark file** (`bookmarks.md`) in your project:

   ```markdown
   # Contentstack Live Preview Quick Links

   ## Core Documentation

   - [Base Concepts](./docs/contentstack-live-preview/01-live-preview-base-concepts.md)
   - [Implementation Guide](./docs/contentstack-live-preview/02-live-preview-implementation.md)

   ## Framework Guides

   - [Next.js CSR](./docs/contentstack-live-preview/03-live-preview-next-csr.md)
   - [Next.js SSR](./docs/contentstack-live-preview/04-live-preview-next-ssr.md)
   - [Nuxt CSR](./docs/contentstack-live-preview/05-live-preview-nuxt-csr.md)
   - [Nuxt SSR](./docs/contentstack-live-preview/06-live-preview-nuxt-ssr.md)
   ```

### 2. JetBrains IDEs (WebStorm, IntelliJ, etc.)

#### Setup Documentation Access

1. **Add documentation as module**:

   - Right-click project root → New → Directory
   - Name: `docs/contentstack-live-preview`
   - Copy documentation files into this directory

2. **Configure file watchers** for Markdown:

   - Go to Settings → Tools → File Watchers
   - Add Markdown watcher for live preview

3. **Create live templates** for common patterns:
   ```xml
   <!-- File: .idea/templates/Contentstack_Live_Preview.xml -->
   <templateSet group="Contentstack Live Preview">
     <template name="cslp" value="// Contentstack Live Preview Setup&#10;// See: ./docs/contentstack-live-preview/$FRAMEWORK$-$MODE$.md&#10;" description="Quick reference to Live Preview docs" toReformat="false" toShortenFQNames="true">
       <variable name="FRAMEWORK" expression="" defaultValue="next" alwaysStopAt="true" />
       <variable name="MODE" expression="" defaultValue="csr" alwaysStopAt="true" />
     </template>
   </templateSet>
   ```

### 3. Vim/Neovim Integration

#### Using with fzf and telescope

1. **Add to your vimrc/init.lua**:

   ```lua
   -- Contentstack docs quick access
   vim.api.nvim_create_user_command('ContentstackDocs', function()
     require('telescope.builtin').find_files({
       prompt_title = "Contentstack Live Preview Docs",
       cwd = vim.fn.getcwd() .. "/docs/contentstack-live-preview",
       find_command = {'find', '.', '-name', '*.md', '-type', 'f'}
     })
   end, {})

   -- Keybinding
   vim.keymap.set('n', '<leader>csd', ':ContentstackDocs<CR>', { desc = 'Open Contentstack docs' })
   ```

2. **Create documentation shortcuts**:
   ```vim
   " Quick access commands
   command! CSLPBase :e ./docs/contentstack-live-preview/01-live-preview-base-concepts.md
   command! CSLPNext :e ./docs/contentstack-live-preview/03-live-preview-next-csr.md
   command! CSLPNuxt :e ./docs/contentstack-live-preview/05-live-preview-nuxt-csr.md
   ```

### 4. Sublime Text Integration

#### Package and Project Setup

1. **Create project file** (`.sublime-project`):

   ```json
   {
     "folders": [
       {
         "path": "."
       },
       {
         "path": "./docs/contentstack-live-preview",
         "name": "📚 Live Preview Docs"
       }
     ],
     "settings": {
       "word_wrap": true,
       "rulers": [80, 120],
       "markdown_extension": "md"
     }
   }
   ```

2. **Add build system** for quick access:
   ```json
   {
     "shell_cmd": "open '$file_path'",
     "file_regex": "^(.+):([0-9]+):([0-9]+): (.+)$",
     "selector": "text.html.markdown",
     "name": "Open Markdown File"
   }
   ```

## Git Integration Strategies

### Option 1: Git Submodule (Recommended)

```bash
# Add as submodule
git submodule add https://github.com/your-repo/contentstack-vibe-docs.git docs/contentstack-live-preview

# Update submodule
git submodule update --remote docs/contentstack-live-preview
```

### Option 2: Git Subtree

```bash
# Add as subtree
git subtree add --prefix=docs/contentstack-live-preview https://github.com/your-repo/contentstack-vibe-docs.git main --squash

# Update subtree
git subtree pull --prefix=docs/contentstack-live-preview https://github.com/your-repo/contentstack-vibe-docs.git main --squash
```

### Option 3: Copy and Version Control

```bash
# Simple copy approach
mkdir -p docs/contentstack-live-preview
cp -r /path/to/contentstack-vibe-docs/.contentstack-vibe-docs/* docs/contentstack-live-preview/

# Add to .gitignore if you want to exclude
echo "docs/contentstack-live-preview/" >> .gitignore
```

## AI Assistant Integration

### GitHub Copilot

These docs are optimized for GitHub Copilot. When the files are in your workspace:

1. **Reference in comments**:

   ```javascript
   // Implementing Contentstack Live Preview for Next.js CSR
   // Reference: ./docs/contentstack-live-preview/03-live-preview-next-csr.md
   ```

2. **Use specific patterns** from the docs in your code comments to trigger relevant suggestions.

### Cursor AI

1. **Add to context** by including doc files in your project
2. **Reference in chat**: "@docs What's the setup for Nuxt SSR Live Preview?"

### Other AI Assistants

- Ensure documentation files are in the workspace
- Reference specific implementation patterns in comments
- Use the troubleshooting sections for debugging assistance

## Project Structure Recommendations

### Recommended Folder Structure

```
your-project/
├── src/
├── docs/
│   └── contentstack-live-preview/
│       ├── 01-live-preview-base-concepts.md
│       ├── 02-live-preview-implementation.md
│       ├── 03-live-preview-next-csr.md
│       ├── 04-live-preview-next-ssr.md
│       ├── 05-live-preview-nuxt-csr.md
│       ├── 06-live-preview-nuxt-ssr.md
│       └── 07-ide-integration.md
├── .contentstack-live-preview-config.json
├── package.json
└── README.md
```

### Configuration File Example

Create `.contentstack-live-preview-config.json`:

```json
{
  "framework": "next",
  "mode": "csr",
  "documentation": "./docs/contentstack-live-preview/",
  "implementation_checklist": [
    "SDK installed",
    "Environment variables set",
    "Live Preview component created",
    "Route handlers configured"
  ]
}
```

## Quick Reference Commands

### Terminal Shortcuts

Add these aliases to your shell config (`.zshrc`, `.bashrc`):

```bash
# Contentstack Live Preview shortcuts
alias cslp-docs="cd docs/contentstack-live-preview && ls -la"
alias cslp-next="code docs/contentstack-live-preview/03-live-preview-next-csr.md"
alias cslp-nuxt="code docs/contentstack-live-preview/05-live-preview-nuxt-csr.md"
alias cslp-base="code docs/contentstack-live-preview/01-live-preview-base-concepts.md"
```

## Troubleshooting IDE Integration

### Common Issues

1. **Documentation not found**:

   - Verify file paths match your project structure
   - Check if files were copied correctly
   - Ensure IDE is indexing the documentation folder

2. **Markdown not rendering**:

   - Install appropriate Markdown extensions
   - Check file associations in IDE settings
   - Verify Markdown preview is enabled

3. **AI assistant not finding docs**:
   - Ensure files are in the workspace root or subdirectory
   - Add documentation path to AI assistant context
   - Reference specific file paths in comments

### Performance Considerations

- Documentation files are lightweight (< 50KB each)
- No impact on build processes
- Can be excluded from production builds via `.gitignore` or build tools

## Best Practices

1. **Keep docs updated**: Regularly sync with the latest version
2. **Use consistent paths**: Maintain the same documentation folder structure across projects
3. **Reference in code**: Add comments linking to relevant documentation sections
4. **Team alignment**: Ensure all team members have the same setup
5. **Version control**: Decide on submodule vs. copy strategy based on team needs

This integration approach ensures that Contentstack Live Preview implementation guidance is always available within your development environment, making it easier for both developers and AI assistants to implement features correctly.
