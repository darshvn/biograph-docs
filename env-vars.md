# Environment Variables & Configuration

## No `.env` Files

Biograph does not use `.env` files. All configuration is managed through Frappe's built-in systems.

## Frappe Site Config

All site-level configuration lives in `sites/{site}/site_config.json` (managed by Frappe):

| Key | Purpose |
|-----|---------|
| `db_name`, `db_password` | MariaDB credentials |
| `redis_cache`, `redis_queue` | Redis connection URLs |
| `developer_mode` | Enables dev features (used by 1 guest endpoint) |
| `mail_server`, `mail_port` | Email configuration |

## Application Configuration (DocTypes)

### Healthcare Settings (singleton)

Configured via Desk > Healthcare Settings. Key fields:

| Setting | Purpose |
|---------|---------|
| Default appointment type | Used by patient portal for booking |
| Default practitioner charge | Fallback consultation fee |
| Payment gateway | Razorpay/Stripe for portal payments |
| Show diagnostics in portal | Toggle portal diagnostics tab |
| Patient registration web form | Enable/disable public registration |
| SMS settings | Appointment reminder SMS text templates |

### ABDM Settings (singleton, India regional)

| Setting | Purpose |
|---------|---------|
| `client_id` | ABDM API client ID |
| `client_secret` | ABDM API client secret |
| `auth_base_url` | ABDM authentication endpoint |

## Code References

Only one direct reference to Frappe config in the codebase:

```python
# healthcare/www/patient_portal.py
if frappe.conf.developer_mode:
    # allows guest access to patient portal bootstrap in dev mode
```

All other configuration is read from DocTypes via `frappe.get_single("Healthcare Settings")` or `frappe.db.get_single_value(...)`.
