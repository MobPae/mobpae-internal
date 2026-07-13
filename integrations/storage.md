# Integration: Cloudflare R2 Storage

Used for all private file uploads: KYC documents, selfies, and profile photos.

---

## What R2 Is

Cloudflare R2 is an S3-compatible object storage service. It is used exactly like AWS S3 with the AWS SDK, but pointed at a Cloudflare endpoint. There is no egress fee, which makes it cheaper for frequent signed URL generation.

---

## Backend Module

**Module:** `src/storage/`  
**Service:** `src/storage/storage.service.ts`  
**SDK:** `@aws-sdk/client-s3` + `@aws-sdk/s3-request-presigner`

```typescript
const s3Client = new S3Client({
  region: 'auto',
  endpoint: `https://${CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: R2_ACCESS_KEY_ID,
    secretAccessKey: R2_SECRET_ACCESS_KEY,
  },
});
```

---

## StorageService Methods

| Method | Description |
|---|---|
| `upload(key, buffer, mimetype)` | Upload a file to R2 |
| `getSignedUrl(key, expiresInSeconds)` | Generate a pre-signed GET URL |
| `delete(key)` | Delete a file |
| `exists(key)` | Check if a key exists |

---

## File Key Convention

All files are stored with a structured key that encodes the file type and owner:

```
{folder}/{employeeId}/{documentType}/{filename}

Examples:
employees/kyc/{employeeId}/AADHAR/document.jpg
employees/kyc/{employeeId}/PAN/document.pdf
employees/kyc/{employeeId}/SALARY_SLIP/document.jpg
employees/selfie/{employeeId}/selfie.jpg
employees/profile/{employeeId}/photo.jpg
```

The key is what gets stored in the database (e.g., `KycDocument.filePath`, `Employee.selfieUrl`).

---

## Signed URL Flow

Files are private (no public access). To display a file in the frontend, the backend generates a signed URL valid for 15 minutes.

```
1. Frontend calls: GET /files/signed-url?key=employees/kyc/{id}/AADHAR/file.jpg
2. Backend: checks that requesting user has permission to view this file
3. Backend: calls s3Client.getSignedUrl(key, { expiresIn: 900 })
4. Returns: { url: "https://..." }
5. Frontend displays: <img src={url} />
```

The signed URL expires in 15 minutes. If a drawer is left open and the URL expires, refreshing the page or re-opening the drawer fetches a fresh URL.

---

## Permission Checks

The `GET /files/signed-url` endpoint enforces ownership:
- **EMPLOYEE:** can only fetch their own files
- **EMPLOYER:** can fetch files for their own employees
- **ADMIN:** can fetch any file

---

## Environment Variables

```env
CLOUDFLARE_ACCOUNT_ID=xxxx
R2_BUCKET_NAME=mobpae-uploads
R2_ACCESS_KEY_ID=xxxx
R2_SECRET_ACCESS_KEY=xxxx
```

---

## Upload Size Limits

| File Type | Max Size |
|---|---|
| KYC Document | 10 MB |
| Selfie | 5 MB |
| Profile Photo | 5 MB |

Size limits enforced by NestJS `FileInterceptor` in the controller.

---

## File Types Accepted

| Category | Accepted MIME Types |
|---|---|
| KYC Documents | `image/jpeg`, `image/png`, `application/pdf` |
| Selfie | `image/jpeg`, `image/png` |
| Profile Photo | `image/jpeg`, `image/png` |
