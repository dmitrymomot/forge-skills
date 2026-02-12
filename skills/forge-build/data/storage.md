# File Storage

Loaded when the feature involves file uploads, downloads, or S3 storage.

---

## Context Methods

| Method | Description |
|---|---|
| `c.Upload(field, opts...)` | Upload file from form field → `*storage.FileInfo` |
| `c.UploadFromURL(url, opts...)` | Upload from remote URL → `*storage.FileInfo` |
| `c.Download(key)` | Download file → `io.ReadCloser` |
| `c.DeleteFile(key)` | Delete file from storage |
| `c.FileURL(key, opts...)` | Generate URL for file |
| `c.Storage()` | Direct storage client access |

---

## Upload Options

```go
c.Upload("avatar",
    storage.WithPrefix("avatars"),               // key prefix (folder)
    storage.WithTenant(c.UserID()),              // multi-tenant prefix
    storage.WithACL(storage.ACLPublicRead),      // public access
    storage.WithValidation(
        storage.MaxSize(5 * 1024 * 1024),        // 5MB max
        storage.MinSize(1024),                    // 1KB min
        storage.ImageOnly(),                      // images only
        storage.AllowedTypes("image/*"),           // MIME filter
        storage.NotEmpty(),                         // no empty files
    ),
)
```

---

## URL Options

```go
// Public URL (permanent)
url, err := c.FileURL(fileKey, storage.WithPublic())

// Signed URL (temporary, private files)
url, err := c.FileURL(fileKey, storage.WithSigned(15 * time.Minute))

// Download URL (forces browser download)
url, err := c.FileURL(fileKey, storage.WithDownload("report.pdf"))
```

---

## FileInfo Struct

Returned by `Upload()` and `UploadFromURL()`:

```go
type FileInfo struct {
    Key         string // Storage key (use this to reference the file)
    Size        int64  // File size in bytes
    ContentType string // MIME content type
    ACL         ACL    // Access control
}
```

Store `FileInfo.Key` in your database to reference files later.

---

## Upload Handler Pattern

```go
func (h *ProfileHandler) uploadAvatar(c forge.Context) error {
    info, err := c.Upload("avatar",
        storage.WithPrefix("avatars"),
        storage.WithTenant(c.UserID()),
        storage.WithACL(storage.ACLPublicRead),
        storage.WithValidation(
            storage.MaxSize(2 * 1024 * 1024),
            storage.ImageOnly(),
        ),
    )
    if err != nil {
        return forge.ErrBadRequest("invalid file: " + err.Error())
    }

    // Store file key in database
    err = h.repo.UpdateUserAvatar(c, repository.UpdateUserAvatarParams{
        ID:        c.UserID(),
        AvatarKey: info.Key,
    })
    if err != nil {
        c.LogError("failed to update avatar", "error", err)
        return forge.ErrInternal("something went wrong")
    }

    // Return public URL
    url, err := c.FileURL(info.Key, storage.WithPublic())
    if err != nil {
        c.LogError("failed to generate file URL", "error", err)
        return forge.ErrInternal("something went wrong")
    }
    return c.JSON(http.StatusOK, map[string]string{"avatar_url": url})
}
```

---

## Download Handler Pattern

```go
func (h *FileHandler) download(c forge.Context) error {
    fileID := forge.Param[string](c, "id")

    file, err := h.repo.GetFileByID(c, fileID)
    if err != nil {
        return forge.ErrNotFound("file not found")
    }

    // Generate signed download URL
    url, err := c.FileURL(file.StorageKey, storage.WithSigned(5*time.Minute), storage.WithDownload(file.Name))
    if err != nil {
        c.LogError("failed to generate download URL", "error", err)
        return forge.ErrInternal("something went wrong")
    }
    return c.Redirect(http.StatusTemporaryRedirect, url)
}
```

---

## Multi-Tenant Storage

Use `WithTenant()` to isolate files per tenant:

```go
// Upload — prefixes key with tenant ID
info, _ := c.Upload("file", storage.WithTenant(tenantID))
// Key becomes: "tenantID/original-key"

// URL — same key works
url, _ := c.FileURL(info.Key, storage.WithPublic())
```

---

## DB Columns for File References

When a table stores file references, add these columns:

```sql
avatar_key VARCHAR(255) NOT NULL DEFAULT '',   -- storage key from FileInfo.Key
-- or for optional files:
document_key VARCHAR(255) NULL,
```

Query to update:
```sql
-- name: UpdateUserAvatar :exec
UPDATE users SET avatar_key = $2, updated_at = now() WHERE id = $1;
```

---

## Storage Config

Required in `cmd/config.go`:

```go
type Config struct {
    Storage storage.Config `envPrefix:"STORAGE_"`
}
```

Env vars: `STORAGE_BUCKET`, `STORAGE_ACCESS_KEY`, `STORAGE_SECRET_KEY`, `STORAGE_ENDPOINT`, `STORAGE_REGION`, `STORAGE_PUBLIC_URL`, `STORAGE_PATH_STYLE`
