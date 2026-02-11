# .gitignore

Generate this file at the project root.

## Content

```gitignore
# Binaries
*.exe
*.exe~
*.dll
*.so
*.dylib
/tmp/

# Test
*.test
*.out
coverage.html

# Go workspace
go.work
go.work.sum

# Environment
.env

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Air
tmp/

# Debug
__debug_bin*
```

## Notes

- `.env` is gitignored (contains local secrets); `.env.example` is NOT gitignored.
- The `tmp/` directory is used by Air for hot-reload builds.
- Vendored frontend assets in `assets/static/` are committed to the repo because `//go:embed static` (in `assets/embed.go`) requires them at build time. A fresh clone must compile without extra steps.
