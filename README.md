# API Change Log (This Branch)

This README now documents only the implemented changes.

## 1) User Registration API updates

### Endpoint
- `POST /api/auth/register`

### New input fields
- `first_alias` (optional, boolean)
  - If set to `false`, the initial newsletter alias is not created.
- `lifetime` (optional, boolean)
  - If set to `true`, user is created with `lifetime=true`.
- `activated` (optional, boolean)
  - If set to `true`, user is created with `activated=true`.

### Behavior updates
- Added support in `User.create(...)` for `first_alias` (default `True`).
- If `activated=true` is explicitly requested, registration:
  - commits the user,
  - skips activation code creation,
  - skips activation email,
  - returns `200` with `"User account is already activated"`.

## 2) Mailbox API updates

### Create mailbox
- Endpoint: `POST /api/mailboxes`
- New input field: `verified` (optional, boolean)
  - If `true`, mailbox is created as verified.

### Delete mailbox
- Endpoint: `DELETE /api/mailboxes/:mailbox_id`
- API deletion now schedules mailbox deletion with `send_mail=false`.
  - Result: no mailbox-deletion email is sent for API-triggered mailbox deletion.

## 3) Custom Domain manual validation API

### New endpoint
- `PATCH /api/custom_domains/:custom_domain_id/validation`

### Supported fields (all optional, boolean)
- `ownership_verified`
- `verified`
- `spf_verified`
- `dkim_verified`
- `dmarc_verified`

### Validation and permissions
- At least one of the fields above must be provided, otherwise `400`.
- Every provided field must be boolean, otherwise `400`.
- Regular users can update only their own domains.
- Admin users can update any domain.

## 4) Tests added/updated

- `tests/api/test_auth.py`
  - Added coverage for `first_alias`, `lifetime`, `activated`.
  - Verifies no `AccountActivation` is created when `activated=true`.

- `tests/api/test_mailbox.py`
  - Added coverage for `verified=true` on mailbox creation.
  - Verifies API mailbox deletion enqueues job with `send_mail=False`.

- `tests/api/test_custom_domain.py`
  - Added coverage for new `/validation` endpoint:
    - success case,
    - forbidden case (non-owner),
    - admin override case.

## 5) API documentation updates

Updated `docs/api.md` to reflect:
- new `register` parameters and behavior,
- new mailbox `verified` input field,
- API mailbox deletion no-email behavior,
- new custom domain validation endpoint.
