# Module: files & storage

Handles file uploads (KYC, profile photo) and secure file access.

**Controller:** `src/files/files.controller.ts`  
**Service:** `src/files/files.service.ts`  
**Storage:** `src/storage/storage.service.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/files/upload` | ANY | Upload a file (multipart) |
| GET | `/files/signed-url` | ANY (authenticated) | Generate a 15-minute signed URL |

---

## Storage: Cloudflare R2

All files are stored in a **private** Cloudflare R2 bucket. Files are never publicly accessible. Every file access goes through the backend signed URL endpoint.

R2 is S3-compatible, so the backend uses `@aws-sdk/client-s3` and `@aws-sdk/s3-request-presigner`.

---

## File Key Structure

Files are stored with organized key paths:

| File Type | Key Pattern |
|---|---|
| KYC documents | `kyc/{employeeId}/{documentType}/{uuid}.{ext}` |
| Profile photo | `profile/{employeeId}/{uuid}.{ext}` |

The DB stores only the **file key** (e.g. `kyc/abc123/AADHAR/def456.jpg`), never the full URL.

---

## Signed URL Flow

```
Frontend needs to display a file:
  GET /files/signed-url?key=kyc/abc123/AADHAR/def456.jpg
  ↓
Backend generates S3 presigned URL (15-minute expiry)
  ↓
Frontend renders file from signed URL
```

---

## StorageService Methods

```typescript
upload(key: string, file: Buffer, contentType: string): Promise<void>
getSignedUrl(key: string, expiresIn?: number): Promise<string>
delete(key: string): Promise<void>
exists(key: string): Promise<boolean>
```
