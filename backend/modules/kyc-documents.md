# Module: kyc-documents

Handles KYC document uploads and admin verification.

**Controller:** `src/kyc-documents/kyc-documents.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/kyc-documents` | EMPLOYEE | Upload a KYC document |
| GET | `/kyc-documents/my` | EMPLOYEE | Get own KYC documents |
| GET | `/kyc-documents/employee/:id` | ADMIN | Get one employee's documents |
| GET | `/kyc-documents/pending` | ADMIN | All pending documents |
| GET | `/kyc-documents/pending-by-employer/:id` | ADMIN | Pending docs for one employer |
| GET | `/kyc-documents/grouped-by-employee` | ADMIN | Documents grouped by employee |
| GET | `/kyc-documents` | ADMIN | All documents (paginated) |
| POST | `/kyc-documents/:id/verify` | ADMIN | Approve a document |
| POST | `/kyc-documents/:id/reject` | ADMIN | Reject with note |

---

## Document Types

| Type | Description |
|---|---|
| `AADHAR` | Aadhaar card (front + back) |
| `PAN` | PAN card |
| `SALARY_SLIP` | Recent salary slip (last 3 months) |
| `OTHER` | Any other document |

Each employee can have one document per type. Uploading a new document replaces the previous one (for resubmission after rejection).

---

## Status Flow

```
PENDING → VERIFIED ✅
        → REJECTED ❌ (with rejection note)
```

All three documents must be VERIFIED for the employee to be eligible for an advance.

---

## File Storage

Files are uploaded to Cloudflare R2 with the key pattern:
```
kyc/{employeeId}/{documentType}/{uuid}.{ext}
```

The DB stores only the file key (not the URL). The frontend fetches a signed URL from `GET /files/signed-url?key={key}` which is valid for 15 minutes.
