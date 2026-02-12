# .editorconfig

Generate this file at the project root.

## Content

```editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.go]
indent_style = tab
indent_size = 4

[*.{yml,yaml}]
indent_style = space
indent_size = 2

[Taskfile.yml]
indent_style = space
indent_size = 4

[*.md]
indent_style = space
indent_size = 2
trim_trailing_whitespace = false

[*.sql]
indent_style = space
indent_size = 4

[*.{html,templ}]
indent_style = space
indent_size = 2

[*.toml]
indent_style = space
indent_size = 4
```

## Notes

- `root = true` stops editors from searching parent directories for additional `.editorconfig` files.
- Go uses tabs per `gofmt` convention.
- `Taskfile.yml` overrides the default YAML 2-space rule because Taskfile uses 4-space indentation.
- Markdown preserves trailing whitespace (used for line breaks in CommonMark).
