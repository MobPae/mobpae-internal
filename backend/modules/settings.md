# Module: settings

Key-value store for platform-wide configuration and CMS content.

---

## Settings Table

Generic key-value store: `Setting { key: string, value: string }`.

| Key | Value Format | Description |
|---|---|---|
| `PLATFORM_FEE_CONFIG` | `{"amount": 175, "currency": "INR"}` | Platform fee per advance |

---

## App Information Module

The `app-information` module manages CMS content displayed in the employee app.

**Types:**

| Type | Used For |
|---|---|
| `ABOUT` | About MobPae page |
| `PRIVACY_POLICY` | Privacy Policy text |
| `TERMS_CONDITIONS` | Terms & Conditions text |
| `HOW_IT_WORKS` | Explainer content |
| `FAQ` | Frequently asked questions |
| `CONTACT` | Contact information |
| `WHATS_NEW` | Release notes / feature highlights |

Content is managed from the admin panel and fetched by the employee app.

---

## Endpoints

### Settings
| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/settings` | ADMIN | Get all settings |
| PUT | `/settings/:key` | ADMIN | Update a setting value |

### App Information
| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/app-information` | ANY | Get all active app information items |
| GET | `/app-information/:type` | ANY | Get one item by type |
| PUT | `/app-information/:type` | ADMIN | Update content |
