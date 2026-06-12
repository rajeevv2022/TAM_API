**API Documentation:**

**Global Standards Applied**

- **Rate Limiting:** Public routes are throttled at **20 req/min**. Authenticated routes are throttled at **40 req/min** (based on your Laravel api.php middleware).

- **HTTP Status Codes:** Most apis return 200 OK universally, using an internal "code": 1 (success) or "code": 0 (failure) schema. Hard failures (missing files/auth) return 403 Forbidden, 404 Not Found, or 500 Internal Server Error.

- **Security:** `html/api/v1/*.php` files exit with plain `Forbidden` unless `SECURE_API_ACCESS` is defined. `html/api.php` defines that constant before including the mapped script.

- **Firebase App Check:** Not read or enforced by `html/api.php`. When the mobile app calls Laravel first, App Check applies at that layer per Laravel configuration.

- [Versioning](#versioning)
- [Authentication](#authentication)
- [Response Standards](#response-standards)
  - [Nested objects and arrays](#nested-objects-and-arrays)
  - [Razorpay payment amounts](#razorpay-payment-amounts)
- [Base URL](#base-url)
  - [Documentation depth by section](#documentation-depth-by-section)
- [1. Check Counsellor Access](#1-check-counsellor-access)
  - [Runtime (`html/api/v1/api_check_counsellor_access.php`)](#runtime-htmlapiv1api_check_counsellor_accessphp)
- [2. Check Maintenance](#2-check-maintenance)
- [3. Check Merge Accounts](#3-check-merge-accounts)
- [4. Check Unique User Name](#4-check-unique-user-name)
  - [Runtime (`html/api/v1/api_check_unique_user_name.php`)](#runtime-htmlapiv1api_check_unique_user_namephp)
- [5. Check Valid Email](#5-check-valid-email)
  - [Runtime (`html/api/v1/api_check_valid_email.php`)](#runtime-htmlapiv1api_check_valid_emailphp)
- [6. Confirm Merge Accounts](#6-confirm-merge-accounts)
- [7. Create Audio Conference](#7-create-audio-conference)
  - [Runtime (`html/api/v1/api_create_audio_conference.php`)](#runtime-htmlapiv1api_create_audio_conferencephp)
- [8. Create User](#8-create-user)
  - [Runtime (`html/api/v1/api_create_user.php`)](#runtime-htmlapiv1api_create_userphp)
- [9. Delete User Account](#9-delete-user-account)
  - [Runtime (`html/api/v1/api_delete_user_account.php`)](#runtime-htmlapiv1api_delete_user_accountphp)
- [10. Generate FAQ](#10-generate-faq)
  - [Runtime (`html/api/v1/api_generate_faq.php`)](#runtime-htmlapiv1api_generate_faqphp)
- [11. Get Current Plan Summary](#11-get-current-plan-summary)
  - [Runtime (`html/api/v1/api_get_current_plan_summary.php`)](#runtime-htmlapiv1api_get_current_plan_summaryphp)
- [12. Get Current Share Permissions](#12-get-current-share-permissions)
  - [Runtime (`html/api/v1/api_get_current_share.php`)](#runtime-htmlapiv1api_get_current_sharephp)
- [13. Get Random Quote](#13-get-random-quote)
  - [Runtime (`html/api/v1/api_get_random_quote.php`)](#runtime-htmlapiv1api_get_random_quotephp)
- [14. Ignore Merge Account](#14-ignore-merge-account)
  - [Runtime (`html/api/v1/api_ignore_merge_account.php`)](#runtime-htmlapiv1api_ignore_merge_accountphp)
- [15. Initiate Subscription Payment](#15-initiate-subscription-payment)
  - [Runtime (`html/api/v1/api_initiate_subscription_payment.php`)](#runtime-htmlapiv1api_initiate_subscription_paymentphp)
- [16. Send Registration OTP](#16-send-registration-otp)
  - [Runtime (`html/api/v1/api_send_registration_otp.php`)](#runtime-htmlapiv1api_send_registration_otpphp)
- [17. Share App Data via Email](#17-share-app-data-via-email)
  - [Runtime (`html/api/v1/api_share_email_counsellor.php`)](#runtime-htmlapiv1api_share_email_counsellorphp)
- [18. Share Private App Data (Permissions Toggle)](#18-share-private-app-data-permissions-toggle)
  - [Runtime (`html/api/v1/api_share_with_counsellor.php`)](#runtime-htmlapiv1api_share_with_counsellorphp)
- [19. Show Plans](#19-show-plans)
  - [Runtime (`html/api/v1/api_show_plans.php`)](#runtime-htmlapiv1api_show_plansphp)
- [20. Subscription Payment Confirmation](#20-subscription-payment-confirmation)
  - [Runtime (`html/api/v1/api_subscription_payment_confirmation.php`)](#runtime-htmlapiv1api_subscription_payment_confirmationphp)
- [21. Third Banner (Dashboard Data)](#21-third-banner-dashboard-data)
- [22. Update User Access Code](#22-update-user-access-code)
- [23. Update User Information](#23-update-user-information)
- [23.1 Ambassador Program Signup (Legacy Web)](#231-ambassador-program-signup-legacy-web)
- [24. Validate Referral Code](#24-validate-referral-code)
- [25. Chat Summary (Feel Better in 15)](#25-chat-summary-feel-better-in-15)
- [26. Check Chat Usage (FB15 / TOT)](#26-check-chat-usage-fb15--tot)
- [27. Audio Transcribe (Main)](#27-audio-transcribe-main)
- [28. Generate Chat Summary (Cron / Manual)](#28-generate-chat-summary-cron--manual)
- [29. Generate WebSocket Token](#29-generate-websocket-token)
- [30. Get Complete YSAM Post](#30-get-complete-ysam-post)
- [31. Google Audio Transcribe (Fallback)](#31-google-audio-transcribe-fallback)
- [32. List YSAM Posts](#32-list-ysam-posts)
  - [`data.data[]` post object (`get_all_ysam_posts`, app branch)](#datadata-post-object-get_all_ysam_posts-app-branch)
- [33. Connections Block User](#33-connections-block-user)
- [Endpoint](#endpoint)
- [Purpose](#purpose)
- [Authentication](#authentication-1)
- [Request Parameters](#request-parameters)
- [Success Response](#success-response)
  - [`data` object (success)](#data-object-success)
- [Failure Response](#failure-response)
- [Notes](#notes)
- [34. Connections Check Consent](#34-connections-check-consent)
- [Endpoint](#endpoint-1)
- [Purpose](#purpose-1)
- [Authentication](#authentication-2)
- [Request Parameters](#request-parameters-1)
- [Success Response](#success-response-1)
- [Failure Response](#failure-response-1)
- [35. Connections Check Usage](#35-connections-check-usage)
- [Endpoint](#endpoint-2)
- [Purpose](#purpose-2)
- [Authentication](#authentication-3)
- [Request Parameters](#request-parameters-2)
- [Success Response](#success-response-2)
- [Failure Response](#failure-response-2)
- [Notes](#notes-1)
- [36. Connections Dashboard](#36-connections-dashboard)
- [Endpoint](#endpoint-3)
- [Purpose](#purpose-3)
- [Authentication](#authentication-4)
- [Request Parameters](#request-parameters-3)
- [Success Response](#success-response-3)
  - [Raw JSON from `list_ysam_user_conversations` (before `tam_connections_dashboard.php`)](#raw-json-from-list_ysam_user_conversations-before-tam_connections_dashboardphp)
  - [Wrapped JSON from `tam_connections_dashboard.php`](#wrapped-json-from-tam_connections_dashboardphp)
  - [`data.conversations[]` row (`list_ysam_user_conversations`)](#dataconversations-row-list_ysam_user_conversations)
  - [`data.discover_groups[]` row](#datadiscover_groups-row)
- [Failure Response](#failure-response-3)
- [Notes](#notes-2)
- [37. Connections Get Messages](#37-connections-get-messages)
- [37a. Connections Mark Messages Seen](#37a-connections-mark-messages-seen)
- [Endpoint](#endpoint-4)
- [Purpose](#purpose-4)
- [Authentication](#authentication-5)
- [Request Parameters](#request-parameters-4)
- [Success Response](#success-response-4)
  - [`data.messages[]` row (`tam_connections_get_messages.php`)](#datamessages-row-tam_connections_get_messagesphp)
- [Failure Response](#failure-response-4)
- [Notes](#notes-3)
- [38. Connections Group Subscribe](#38-connections-group-subscribe)
- [Endpoint](#endpoint-5)
- [Purpose](#purpose-5)
- [Authentication](#authentication-6)
- [Request Parameters](#request-parameters-5)
- [Success Response](#success-response-5)
- [Failure Response](#failure-response-5)
- [Notes](#notes-4)
- [39. Connections Group Unsubscribe](#39-connections-group-unsubscribe)
- [Endpoint](#endpoint-6)
- [Purpose](#purpose-6)
- [Authentication](#authentication-7)
- [Request Parameters](#request-parameters-6)
- [Success Response](#success-response-6)
- [Failure Response](#failure-response-6)
- [Notes](#notes-5)
- [40. Connections Register Consent](#40-connections-register-consent)
- [Endpoint](#endpoint-7)
- [Purpose](#purpose-7)
- [Authentication](#authentication-8)
- [Request Parameters](#request-parameters-7)
- [Success Response](#success-response-7)
- [Failure Response](#failure-response-7)
- [Notes](#notes-6)
- [41. Connections Send Message](#41-connections-send-message)
- [Endpoint](#endpoint-8)
- [Purpose](#purpose-8)
- [Authentication](#authentication-9)
- [Request Parameters](#request-parameters-8)
- [Supported Attachment MIME Types](#supported-attachment-mime-types)
- [Success Response](#success-response-8)
  - [`data` fields — success (`message` **success**, single or group)](#data-fields--success-message-success-single-or-group)
  - [`data` fields — idempotent duplicate (`message` **duplicate**)](#data-fields--idempotent-duplicate-message-duplicate)
- [Failure Response](#failure-response-8)
- [Notes](#notes-7)
- [42. User Load Conversations](#42-user-load-conversations)
- [Endpoint](#endpoint-9)
- [Purpose](#purpose-9)
- [Authentication](#authentication-10)
- [Request Parameters](#request-parameters-9)
- [Success Response](#success-response-9)
  - [`data` room fields (`app_tam_connections_messages`)](#data-room-fields-app_tam_connections_messages)
  - [`data.messages[]` row (`format_message_payload`)](#datamessages-row-format_message_payload)
  - [`reply_to` object (`build_reply_block`, when non-null)](#reply_to-object-build_reply_block-when-non-null)
- [Failure Response](#failure-response-9)
- [Notes](#notes-8)
- [43. Core Translate](#43-core-translate)
- [44. Upload Audio](#44-upload-audio)
- [45. YSAM Add/Update Article](#45-ysam-addupdate-article)
- [46. YSAM Connect to User](#46-ysam-connect-to-user)
- [47. YSAM Follow User](#47-ysam-follow-user)
- [48. YSAM Get All Categories](#48-ysam-get-all-categories)
- [49. YSAM Initialize Form](#49-ysam-initialize-form)
- [50. YSAM Report User Post](#50-ysam-report-user-post)
- [50a. YSAM Review Message](#50a-ysam-review-message)
- [51. YSAM Translate](#51-ysam-translate)
- [52. Get Default Countries](#52-get-default-countries)
- [53. API Health (probe)](#53-api-health-probe)
- [Endpoint](#endpoint-10)
- [Purpose](#purpose-10)
- [Authentication](#authentication-11)
- [Request Parameters](#request-parameters-10)
- [Success Response](#success-response-10)
  - [`countries[]` row](#countries-row)
  - [`currencies[]` row](#currencies-row)
- [Failure Response](#failure-response-10)


### Mobile app path (Laravel proxy)

The Flutter app calls **Laravel** routes (for example POST {APP_BASE}/user_conversations_dashboard) with **JSON** bodies (Content-Type: application/json) and standard mobile headers. Laravel controllers forward to the legacy **/api.php/v1/{apiName}** endpoints using server-side bearer + form-style POST fields.

**Runtime rules (implemented):**

- **Connections dashboard (inbox):** Laravel returns { "status", "message", "data": { "conversations", "total_unread", "discover_groups", "dashboard_message", … } }. Clients also accept a **legacy-flat** shape where conversations and dashboard_message appear at the **root** (backwards compatibility).
- **Connections get messages (`get_connection_messages`):** Canonical upstream fields read in `html/api/v1/tam_connections_get_messages.php` are `room_name`, `lastSeq`, `beforeSeq`, `user_id` (or JWT-populated `auth_user_id` via `$_REQUEST`), and `user_lang`. Laravel proxy: `POST /api/get_connection_messages` (Passport). Legacy decrypts `message_text` in `tam_connections_replay_message_row()`; replay rows for the sender include optional `delivery_status` (`sent` \| `delivered` \| `seen`). Laravel may accept additional aliases; they are not read by this legacy file.
- **Connections mark messages seen (`connections_mark_messages_seen`):** Canonical upstream fields: `room_name`, `room_type` (`S` \| `G`, optional — inferred when omitted), `message_ids` (JSON array or JSON string of `client_message_id` values), `user_id` / `auth_user_id`, optional `user_lang`. Laravel proxy: `POST /api/connections_mark_messages_seen` → legacy → `ConnectionsMessageSeenService` publishes advisory `message_status` (`seen`) on Redis `chat_messages`.
- **Connections send message:** Upstream persists **reply_to_message_id** only. The Laravel proxy accepts **reply_to_id** as an alias when reply_to_message_id is omitted (logged in app.debug only). Fields such as reply_to_content / reply_to_sender / timestamp_browser are **not** read by the legacy send handler and are not forwarded as distinct upstream fields.
- **Connections group subscribe:** Upstream requires **user_id** (reservation user id) and **group_id**.

### Contract governance (Phase 2)

- **Canonical vs compatibility:** Each Connections subsection below documents **canonical** legacy field names. **Compatibility aliases** accepted only at the Laravel proxy are listed under *Mobile app path (Laravel proxy)* above—not as legacy /api.php requirements.
- **Artifacts (Flutter repo):** TAM_FLUTTER/docs/governance/README.md, docs/governance/snapshots/critical_api_envelopes.json, and generated docs/governance/generated/CONTRACT_INVENTORY.* from the PHP builder.
- **Curated crosswalk (Dart, tool-only):** TAM_FLUTTER/tool/governance/contract_governance_crosswalk.dart — critical Flutter AppConstants ↔ Laravel route segment ↔ api.json key.
- **Local validation:** From TAM_FLUTTER/, run dart run tool/contract_governance.dart (fails if a curated route or api.json key is missing).
- **Broad inventory:** From tam-admin-application/tools/, run php governance_build_inventory.php (regenerates route/env/api.json lists).
- **Drift telemetry (mobile):** Repeated invalid **WebSocket token** JSON envelopes (HTTP 200 but missing data.jwt) emit a bounded Crashlytics breadcrumb (see ContractDriftWatch). Repeated **malformed WebSocket wire frames** (JSON decode failure, non-object root, or non-string type) emit a bounded Crashlytics event (ContractDriftWatch.noteMalformedWsWire). Maintenance unknown-key checks remain **debug-only** (debugAssertMaintenanceResponseContract).

### Documentation maintenance (`tools/audit_apis_md.php`)

**Last full audit:** 2026-06-06 — **55 / 55** `api.json` keys clean (`tools/audit_apis_md_report.json`).

| Check | What it does |
|-------|----------------|
| `api.json` coverage | Every `allowedApis` key must appear in `apis.md` and have a matching `html/api/v1/{v1}.php` file. |
| POST parameter drift | Flags `$_POST` / `$_REQUEST` keys not mentioned near the API section in `apis.md`. |
| Envelope hint | Classifies each handler as `status_message`, `code_text`, or `mixed` for reviewer focus. |
| `auth_user_id` | Flags handlers that read JWT-populated `auth_user_id` but do not document it nearby. |

Run before merging API changes: `D:\php-8.3\php.exe tools/audit_apis_md.php` (human-readable), `--json` for CI, or `--out=tools/audit_apis_md_report.json` (default report path). Exit code `0` when all 55 `api.json` keys have a numbered section with endpoint path and documented POST keys. **Runtime behavior always wins** over this document when they disagree — update `apis.md` after fixing or accepting the code path.

### Operational governance (Phase 3 — release & deployment safety)

**Source of truth:** Runtime code (Laravel proxies, CurlPhp.php, legacy html/api/v1/*.php, Flutter parsers) wins over this document. Update apis.md and TAM_FLUTTER/docs/governance/snapshots/critical_api_envelopes.json when those layers change.

| Layer | Role |
|-------|------|
| **Flutter** | Calls POST {BASE_URL}/{AppConstants.*_URI} with bearer + AppCheck; parses JSON envelopes documented per feature. |
| **Laravel routes/api.php** | Authenticated mobile routes; maps to App\Http\Controllers\Api\* methods. |
| **CurlPhp.php** | Resolves env('API_*') to upstream legacy URLs. |
| **html/api/api.json** | allowedApis keys → v1 PHP script basenames executed by the router. |
| **html/api/v1/*.php** | Legacy implementation and canonical field names for upstream. |
| **WebSocket** | JWT from generate_ws_token → RemoteConfigService.tamWebSocketUrl → /socket/?token=; inbound frames are JSON objects with string type (see table below). |

**Pre-release commands (no external services):** from TAM_FLUTTER/, run dart run tool/release_integrity.dart (Phase 3A+3C: Flutter ↔ Laravel route ↔ controller PHP ↔ CurlPhp env usage ↔ api.json v1 script on disk ↔ optional --laravel-env= URL checks), dart run tool/contract_regression.dart (Phase 3B envelope snapshot warnings), and dart run tool/contract_governance.dart (Phase 2 crosswalk).

**WebSocket inbound contracts (Flutter TamWebSocketService):** All frames are JSON with string type. Room-scoped delivery uses room when present; otherwise the client falls back to the sole subscribed room or the active chat room for typing/status.

| type | Required / commonly used fields | Notes |
|--------|-----------------------------------|--------|
| message | server_seq, messageId, from, content; optional translated_text, timestamp / timestamp_browser, nested reply_to (messageId, content, sender) | Bumps dashboard inbound events; dedupes by messageId / seq. |
| recovery | Same as message | Merged into history **without** dashboard unread/toast side-effects. |
| message_status | messageId, status | Merges delivery state with monotonic rank (sent → delivered → seen). |
| delivered | messageId | Promotes local row to delivered. |
| status | roomUsers (list of maps with userId or string ids) **or** userId + online | Presence for active recipient. |
| typing | from, typing (bool) | Ignores self-typing; 3s debounce timer. |
| Client → server join / leave | room, from | Ref-counted per feature owners. |

**Mark seen (HTTP, authoritative):** See [§37a Connections Mark Messages Seen](#37a-connections-mark-messages-seen). Laravel route `POST /api/connections_mark_messages_seen`; legacy `POST /api.php/v1/connections_mark_messages_seen`.

**Contract ownership:** Product-critical HTTP paths are curated in TAM_FLUTTER/tool/governance/contract_governance_crosswalk.dart. Broad route/env lists are **generated** (governance_build_inventory.php) — treat as inventory, not behavioral specs.

### Development Environment Override
In development environments only, Firebase AppCheck validation may be bypassed by sending:
http
X-Firebase-AppCheck: A1B2C3D4E5F6

This override is disabled in staging and production environments.

*Unless otherwise specified, POST endpoints use:*  
**Content-Type: application/x-www-form-urlencoded**

## Versioning

Current API Version: `v1`. Legacy scripts live under `html/api/v1/`. The HTTP router is `html/api.php` (not `html/api/api.php`).

## Authentication

**Enforced by `html/api.php` before any `html/api/v1/*.php` executes:**

| Requirement | Runtime behavior |
|---|---|
| HTTPS | Non-HTTPS requests receive JSON `{"error":"HTTPS required"}` with HTTP 403. |
| `Authorization` header | Required. Must be `Bearer <token>`. Missing/invalid format → JSON `{"error":"Missing or Invalid Authorization"}` with HTTP 401. |
| Bearer validation | If `<token>` contains `.`, it is decoded as HS256 JWT using `tam_connections_jwt_secret` (same secret as WebSocket JWT issuance). On success, `$_REQUEST['auth_user_id']` is set to payload `id`. If JWT decode fails, `<token>` is compared (timing-safe) to `BEARER_TOKEN` from `html/api/api.json`. If neither succeeds → JSON `{"error":"Invalid Authorization"}` with HTTP 403. |
| Firebase App Check | Not enforced in `html/api.php`. |

**Request body:** Raw JSON bodies are merged into `$_POST` by `html/api.php` before the v1 script runs (same keys as JSON root).

## Response Standards

There is **no single global envelope**. Each v1 script echoes its own JSON (or plain `Forbidden` when `SECURE_API_ACCESS` is absent).

Across endpoints you may see, non-exhaustively:

- Boolean `status` (`true` / `false`) with `message` (string) and `data` (array or object).
- String `"code"` such as `"1"` / `"0"` (legacy list payloads).
- HTTP 500 from the router with `{"error":"Internal server error"}` on fatal errors in the router wrapper.

Numeric and boolean-like fields may appear as strings or native JSON types depending on the script. Always branch on the **specific** endpoint implementation.

### Nested objects and arrays

Where responses include arrays of objects (for example `conversations[]`, `messages[]`, `data.posts[]`), this document lists **each object field** in a table: type, presence (`S`ingle / `G`roup / both), and runtime notes. Endpoint sections that still use only a minimal JSON example are marked as **summary**; prefer the field tables when present.

### Razorpay payment amounts

Razorpay uses **two different unit systems** on the same order/payment. Clients must not assume all numeric fields are paise/cents.

| Field / location | Unit | Type | Notes |
|------------------|------|------|-------|
| `razorpay_json.amount` (§15 initiate) | **Minor** | integer | Value sent to Razorpay Checkout / Flutter SDK. Scale = `10^currency_minor_unit`. |
| Razorpay `order.amount`, `payment.amount` | **Minor** | integer | Same scale as above. |
| Razorpay `payment.fee`, `payment.tax`, `payment.base_amount` | **Minor (INR paise)** | integer | Always divided by 100 server-side for INR ledger math, even on FX payments. |
| Order/payment `notes.payment_gst_fees`, `notes.payment_price_wo_fees`, `notes.convenience_charge` | **Major** | string/number | Human-readable payment-currency amounts. **Never** divide by 100 when reading notes. |
| `notes.note_amount_unit` | — | string | Always `"major"` on newly created orders. |
| `notes.note_amount_currency` | — | string | ISO code for monetary note fields (often user/display currency on subscription, payment currency on counselling checkout). |
| `notes.allocation[].payment_due` | **Major** | number | Payment-currency major units per outstanding item (web counselling checkout). |

**Exponent source:** `currencies.currency_minor_unit` (0, 2, or 3) via `get_currency_details()` → `payment_decimal_places`. Matches [Razorpay supported currencies](https://razorpay.com/docs/payments/payments/international-payments/#supported-currencies).

**Conversion helpers (legacy PHP):** `html/includes/tam_payment_amount_helpers.php` — `tam_to_minor_units()`, `tam_from_minor_units()`, `tam_build_razorpay_order_notes()`, `tam_assert_razorpay_payment_amounts()`.

**Amount examples by currency** (same major payable, different `amount` integer):

| Currency | `currency_minor_unit` | Major payable | `amount` (minor) sent to Razorpay |
|----------|----------------------|---------------|-----------------------------------|
| INR | 2 | ₹1500.00 | `150000` |
| USD | 2 | $19.99 | `1999` |
| JPY | 0 | ¥1500 | `1500` |
| KWD | 3 | 1.234 KWD | `1230` (last digit zero per Razorpay 3-decimal rule) |

**§15 `razorpay_json` parsed object — key amount fields:**

| Field | Type | Unit | Example (INR) | Example (USD) | Example (JPY) |
|-------|------|------|---------------|---------------|---------------|
| `amount` | integer | minor | `150000` | `1999` | `1500` |
| `currency` | string | — | `"INR"` | `"USD"` | `"JPY"` |
| `notes.payment_price_wo_fees` | string | major | `"1271.19"` | `"16.94"` | `"1271"` |
| `notes.payment_gst_fees` | string | major | `"228.81"` | `"3.05"` | `"0"` |
| `notes.note_amount_unit` | string | — | `"major"` | `"major"` | `"major"` |

**Confirmation (§20):** Before posting, the server calls `tam_assert_razorpay_payment_amounts()` — order and payment minor amounts must match, and (when an `INITIATED` row exists) must match `reservation_payments.payment_price` in payment currency. Mismatch returns localized `error_payment_amount_mismatch` (HTTP 200 with `"code":"0"` or exception path depending on handler).

**Automated tests:** `tam-admin-application/tests/unit/Payments/RazorpayAmountHelpersTest.php` (`php artisan test --filter=RazorpayAmount`).

## Base URL

text
https://[BASE_URL]
Development: https://localhost
Staging: https://tamweb.theablemind.com
Production: https://theablemind.com


All endpoints below are relative to the base URL.

### Documentation depth by section

Field-level **Runtime (`html/api/v1/...`)** subsections document request/response shapes per handler. Re-validate after code changes with:

```bash
D:\php-8.3\php.exe tools/audit_apis_md.php
```

The script cross-checks every `api.json` `allowedApis` key against `html/api/v1/{script}.php` (POST parameters, envelope style) and `docs/apis.md` coverage. Exit code `0` = no structural gaps; non-zero lists APIs needing doc updates.

## 1. Check Counsellor Access

Validates a Google OAuth2 access token for an employee, provisions or updates their access rights in the system, and returns their profile details and roles.

- **Endpoint:** /api.php/v1/check_counsellor_access

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Enforces email domain matching (@theablemind.com). Validates Google tokens directly against the Google Client API.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| access_token | String | Yes | Google OAuth2 access token provided by the frontend Google Login SDK. |
| user_lang | String | No | BCP-47 language code (default: en). |

**Example Payload (Form Data):**

```text
access_token=ya29.a0AfB_byC...&user_lang=en
```

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_counsellor_access \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "access_token=ya29.a0AfB_byC..." \
-d "user_lang=en"


**Success Schema (200 OK):**

json
[
{
"code": 1,
"type": ["A", "C", "M", "O"],
"c_id": 123,
"c_name": "John Doe",
"c_email": "john@theablemind.com",
"c_phone": "9876543210",
"c_timezone": "Asia/Kolkata",
"c_age": 35,
"c_gender": "Male",
"c_picture": "https://.../profile.jpg",
"c_lang_code": "en",
"c_country": "IN",
"c_video": "https://.../intro.mp4",
"message": ""
}
]


**Error Schema (200 OK - Soft Fail):**

json
[
{
"code": 0,
"type": [],
"message": "Access Denied. This feature is only available to employees of The Able Mind."
}
]

### Runtime (`html/api/v1/api_check_counsellor_access.php`)

| Item | Detail |
|------|--------|
| **HTTP body** | Single JSON **array** with **exactly one** object (`echo json_encode($output)`; `$output` is a list with one `push`). |
| **HEAD** | No body; connection closed. |
| **Success object fields** | `code` (int `1` on success), `type` (string array: `"A"` admin, `"O"` ops lead, `"M"` manager, `"C"` app counsellor), `c_id` (Laravel `users.id` or new insert id), `c_name`, `c_email`, `c_phone`, `c_timezone`, `c_age` (computed), `c_gender`, `c_picture` / `c_video` (URL or `""`), `c_lang_code`, `c_country`, `message` (`""` on success). |
| **Failure object fields** | Same keys; `code` is int `0`; `type` is `[]`; counsellor fields empty strings; `message` is translated error text. |
| **Google verify branch** | `authenticate_token` returns `code` as int `1`/`0` (PHP compares to `"0"` in one branch — treat `code` as numeric in clients). |

## 2. Check Maintenance

Checks if the application is currently under maintenance and returns the availability status of specific modules.

- **Legacy endpoint:** /api.php/v1/check_maintenance
- **Laravel route (mobile):** POST {APP_BASE}/check_maintenance with **JSON** body (same field names: user_lang, timezone).

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded (legacy direct). **Mobile:** application/json to Laravel.

- **Rate Limit:** 20 requests / minute (Public Route)

- **Security Notes:** Laravel mobile route `POST /check_maintenance` is public (throttle only). **Direct** `html/api.php` v1 calls still require `Authorization: Bearer` per global §Authentication. Firebase App Check applies at the Laravel layer when configured.

**Runtime (`html/api/v1/api_check_maintenance.php`):** response is a single JSON **object** (no wrapper). Emitted keys (after `finally` normalization):

| Key | When present | Type / meaning |
|-----|----------------|----------------|
| `UPDATION_INPROGRESS` | Always at emit | bool. `true` when an active maintenance row exists and `app_update` drives global maintenance UI; idle default `false`. (`MAINTENANCE_ACTIVE` is stripped before echo.) |
| `APP_UPDATE_VERSION` | Always | string. Target app version from SQL when `app_update == 1`; else `""`. |
| `MESSAGE_TITLE`, `MESSAGE`, `LIVE_BACK_TIME` | Always | strings. `LIVE_BACK_TIME` is ISO-8601 UTC (`Y-m-d\TH:i:s\Z`) when a row exists; else `""`. |
| `SHORT_MESSAGE` | Always at emit | string. From `maintenance_short_text` constant when active; idle `""`. (`short_message` lowercase key is renamed/unset in `finally`.) |
| `REGISTRATION`, `FEEL_BETTER_IN_15`, `THERAPY_OVER_TEXT`, `NIGHT_AUXIE`, `TROOPERS_TOGETHER`, `LIBRARY`, `RESOURCES`, `ASSESSMENTS`, `CONNECTIONS`, `YSAM` | Always | bool. **`true` = module enabled** (`!(bool)$*_disabled` from SQL). Idle template defaults all to `true`. |

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_lang | String | No | Language code to localize the maintenance message (default: en). |
| timezone | String | No | User's timezone to calculate the LIVE_BACK_TIME display (default: Asia/Kolkata). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_maintenance \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_lang=en" \
-d "timezone=America/New_York"


**Success Schema (200 OK):** (idle example — feature flags `true` = enabled)

json
{
"UPDATION_INPROGRESS": false,
"APP_UPDATE_VERSION": "",
"MESSAGE_TITLE": "",
"MESSAGE": "",
"LIVE_BACK_TIME": "",
"REGISTRATION": true,
"FEEL_BETTER_IN_15": true,
"THERAPY_OVER_TEXT": true,
"NIGHT_AUXIE": true,
"TROOPERS_TOGETHER": true,
"LIBRARY": true,
"RESOURCES": true,
"ASSESSMENTS": true,
"CONNECTIONS": true,
"YSAM": true,
"SHORT_MESSAGE": ""
}

## 3. Check Merge Accounts

Checks if there are multiple accounts associated with the provided email or phone number that need to be merged to prevent data fragmentation.

- **Endpoint:** /api.php/v1/check_merge_accounts

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Handled within the auth:api middleware group.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_email | String | Cond. | Trimmed; used as **primary login** when `login_type` is **E** (default). |
| user_phone | String | Cond. | Trimmed; used as **primary login** when `login_type` is **P**. |
| login_type | String | No | `E` = email path, `P` = phone path (case-insensitive in `api_check_merge_accounts.php`). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_merge_accounts \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_email=test@example.com" \
-d "login_type=E"


**Success / body (200 OK):** `api_check_merge_accounts.php` assigns `$output = json_decode(check_merge_accounts(...), true)` and echoes `json_encode($output)` with **no** extra wrapper.

Top-level shape from `check_merge_accounts()` in `html/functions.php`:

| Field | Type | Notes |
|-------|------|--------|
| `status` | string | `"success"` for normal completion (including empty input / no duplicates). **`"error"`** only from the script `catch` block (not from the helper). |
| `merge_count` | int | `count(data)` when `data` is an array of merge candidates. |
| `current_user_id` | int or null | Primary reservation user id when the login resolves; else `null`. |
| `data` | array or string | **Array** of candidate objects (see table below), or a **string** error/human message (`"No primary login key provided"`, `"No user found for provided email"`, or opt-out early exit with empty array — still `status: success`). |

**`data[]` object fields** (each merge candidate):

| Field | Type |
|-------|------|
| `old_user_id` | int |
| `user_email` | string (partially masked or `"-"`) |
| `phone_number` | string (partially masked or `"-"`) |
| `user_name` | string (masked) |
| `last_login` | string (formatted `l, d M Y` or `""`) |
| `user_gender` | string (`Male` / `Female` / `Others`) |
| `valid_email` | `"Y"` or `"N"` |
| `login_type` | string (DB `external_login`) |

## 4. Check Unique User Name

Validates whether a requested username is available or already taken by another user.

- **Endpoint:** /api.php/v1/check_unique_user_name

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Prevents username enumeration by rate-limiting.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_name | String | Yes | The requested username. |
| login_key | String | No | The user's primary login key (email/phone). Strips out + characters natively. |
| user_lang | String | No | Language code for error localization (default: en). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_unique_user_name \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_name=johndoe99"


**Success Schema (200 OK - Name Available):**

json
{
"response_code": "1",
"message": "success",
"status": "success",
"data": ""
}

**Error Schema (200 OK - Name Taken):**

json
{
"response_code": "0",
"message": "Provided username is already taken. Please try another one.",
"status": "failure",
"data": ""
}


### Runtime (`html/api/v1/api_check_unique_user_name.php`)

| Item | Detail |
|------|--------|
| **Envelope** | Single JSON object: `response_code` (string `"1"` / `"0"`), `message`, `status`, `data` (always `""` string in normal paths). |
| **Blank name** | `response_code` `"0"`, `message` `"User Name cannot be blank"`, `status` `"failure"`. |
| **Taken name** | Loads `language_config/create_user/create_user_config_{user_lang}.php` and sets `message` from `username_already_used` constant when defined. |
| **Catch** | Same envelope with localized `technical_issue` / `high_traffic` text. |

## 5. Check Valid Email

Performs deep validation on an email address, checking MX DNS records and ensuring it is not a temporary/disposable email domain.

- **Endpoint:** /api.php/v1/check_valid_email

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Connects to an external API (api.mailcheck.ai) to verify disposable status. Hard-timeouts after 10 seconds to prevent application hanging.

**Parameters:**

| **Name**   | **Type** | **Required** | **Description**            |
|------------|----------|--------------|----------------------------|
| user_email | String   | Yes          | Email address to validate. |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_valid_email \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_email=legit.user@gmail.com"


**Success Schema (200 OK - Valid):**

json
{
"response_code": "1",
"message": "Email id provided is valid.",
"status": "success",
"data": ""
}


**Error Schema (200 OK - Disposable/Invalid):**

json
{
"response_code": "-1",
"message": "Email id provided is a disposable id. Domain (tempmail.com) is invalid.",
"status": "failure",
"data": ""
}


### Runtime (`html/api/v1/api_check_valid_email.php`)

| `response_code` | `status` | When |
|-----------------|----------|------|
| `"1"` | success | Disposable API unreachable (`curl` failure) — proceed-with-caution path; or disposable check returned non-disposable. |
| `"0"` | success | Domain is in allowlist `valid_email_domains` (`make_output("0", …)` — message *"Email id provided is valid."*). |
| `"-1"` | failure | Invalid format, no MX, domain in `invalid_email_domains`, or disposable API reports `disposable: true`. |
| `"0"` | failure | Uncaught `Throwable` in `catch` (`make_output("0", $e->getMessage(), "failure")`). |

All codes are **strings**. `data` is always present on the envelope (string, often `""`).

## 6. Confirm Merge Accounts

Executes the account merging process, consolidating the selected duplicate accounts into the primary User ID (reservation users).

- **Endpoint:** /api.php/v1/confirm_merge_accounts

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Requires active Bearer token. Logs the action against the user's audit trail.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_id | Integer | Yes | The primary User ID (reservation users) to merge into. |
| account_list | String | Yes | Comma-separated list of old User ID (reservation users)s to merge (e.g., 124,125). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/confirm_merge_accounts \
-H "Authorization: Bearer <TOKEN>" \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "user_id=123" \
-d "account_list=124,125"


**Runtime (`html/api/v1/api_confirm_merge_accounts.php`):**

| Path | Body |
|------|------|
| **Happy path** | **Raw echo** of the **integer** return value from `merge_user_accounts_confirm($connection, $user_id, $account_list)` — **not** JSON. Typical success value is `0` (see `merge_user_accounts` in `html/functions.php`). |
| **Throwable** | JSON object: `{"status":"error","data":"<localized or technical string>"}`. |

`merge_user_accounts_confirm` passes the third argument straight into `merge_user_accounts` as the **old** user id. If `account_list` contains commas, PHP coerces the string to an integer (**first numeric segment only**); clients that need multiple merges should call once per `old_user_id` or extend the script to loop.

**Success Schema (200 OK):** *(legacy echoes a bare integer — example below is illustrative; clients must tolerate non-JSON on success.)*

text
0


**Error Schema (200 OK):**

json
{
"status": "error",
"data": "Technical issue message"
}


## 7. Create Audio Conference

Creates or updates a scheduled audio conference room and assigns moderators/counsellors.

- **Endpoint:** /api.php/v1/create_audio_conference

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| conference_id | String | Yes | Unique ID for the conference room. |
| conference_duration | Integer | No | Duration in minutes (default: 60). |
| conference_start_time | String | Yes | Start time in Y-m-d H:i:s format. |
| conference_counsellor_keys | String | Yes | Comma-separated primary keys of assigned counsellors. |
| conference_title | String | Yes | Title of the conference. |
| max_simultaneous_connections | Integer | No | Max participants allowed (default: 20). |

**Success Schema (200 OK):**

json
{
"response_code": "1",
"message": "success",
"status": "success",
"data": ""
}


### Runtime (`html/api/v1/api_create_audio_conference.php`)

| `response_code` | `status` | When |
|-----------------|----------|------|
| `"1"` | success | `create_update_audio_conference` returned `'1'`. |
| `"0"` | failure | Function returned `'-1'` (DB error) **or** validation branch (`$data == '3'`) for missing/invalid mandatory fields **or** `strtotime` produced sentinel used as error (`$date == 3` guard). |
| `"0"` | failure | `catch`: localized DB messages or generic *"Error Creating / Updating Conference"*. |

HEAD requests: no JSON body is written (`echo` skipped when method is `HEAD`); `$output` stays the initialized empty array for those calls.

## 8. Create User

Registers a new user account, handling profile details, guardian information (for minors), and corporate access codes.

- **Endpoint:** /api.php/v1/create_user

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 20 requests / minute (Public Registration)

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| login_type | String | Yes | "E" (Email) or "P" (Phone). |
| user_email | String | Cond. | Email (required if phone is blank). |
| user_phone_number | String | Cond. | Phone (required if email is blank). |
| user_name | String | No | Full name of the user. |
| user_password | String | No | Plain text password (hashed server-side). |
| user_age | String | No | Age of the user. |
| user_sex | String | No | Gender. |
| external_login | String | No | "Y" or "N" for SSO. |
| user_guardian_name | String | Cond. | Required if user is a minor (<18). |
| user_guardian_phone | String | Cond. | Required if user is a minor. |
| user_guardian_relationship | String | Cond. | Required if user is a minor. |
| access_code | String | No | Corporate / Plan access code. |
| device_id | String | No | Client device identifier. |
| user_lang | String | No | Language code (default `en`); must be in the script’s allow-list or forced to `en`. |
| client_ip_address | String | No | Passed through to `create_user(...)` as `$user_client_ip_address` (geo / access-code validation). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/create_user \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "login_type=E" \
-d "user_email=newuser@example.com" \
-d "user_name=Jane Doe" \
-d "user_password=SecurePass123"


**Success Schema (200 OK):** *(shape from `create_user(..., $call_from_app = true)` in `html/functions.php` — user id is **`reservation_user_id`**, not nested under `data`.)*

json
{
"response_code": "1",
"status": "success",
"message": "Localized registration / trial summary string",
"invalid_code": "N",
"reservation_user_id": 123,
"invalid_email": "N",
"show_user_usage_policy": ""
}


** Note: **
- **`show_user_usage_policy`**: long-form usage / plan-restriction text for the UI (may be empty).
- **`invalid_code`** / **`invalid_email`**: string flags returned on several success paths (`"Y"` / `"N"` style).
- Failures use the same top-level keys with `response_code` **`"0"`**, `status` **`failure`**, and **`show_user_usage_policy`** commonly `""`.

### Runtime (`html/api/v1/api_create_user.php`)

| Item | Detail |
|------|--------|
| **Envelope** | Single JSON object from `echo json_encode($output)` after `create_user` returns JSON decoded into `$output`, or from early validation / `catch`. |
| **Admin session** | For each non-HEAD request the script sets `$_SESSION['user_is_admin']`, `$_SESSION['user_id']`, `$_SESSION['user_name']` to fixed admin values so `create_user` internal permission checks succeed — scoped to this request only until `session_destroy()` in `finally`. |
| **`response_code`** | String **`"1"`** / **`"0"`** on normal paths. |
| **Success extras** | `invalid_code`, `invalid_email`, `reservation_user_id` (int), `show_user_usage_policy` (string, may be multi-line). |
| **HEAD** | No body (`echo` skipped). |

## 9. Delete User Account

Deletes a user account securely. Fails if the user has an active subscription or upcoming reservations.

- **Endpoint:** /api.php/v1/delete_user_account

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**  | **Type** | **Required** | **Description**              |
|-----------|----------|--------------|------------------------------|
| user_id   | String   | Yes          | User ID (reservation users) to delete.           |
| user_lang | String   | No           | Language code (default: en). |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Account successfully deleted"
}


**Error Schema (200 OK - Active Subscription):**

json
{
"code": 0,
"message": "Cannot delete account with an active subscription."
}


### Runtime (`html/api/v1/api_delete_user_account.php`)

| Field | Type | Notes |
|-------|------|--------|
| `code` | int | `1` success; `0` all failure / empty-input / exception paths. |
| `message` | string | From `language_config/delete_account/config_delete_user_{user_lang}.php` constants (`delete_success`, `valid_subscription_exists`, `elevated_access`, `not_authorized`, etc.) or `session_error` / `technical_issue` / `high_traffic` on `catch`. |

**`delete_user_cp(..., true)` return → `code` mapping**

| Return | `code` | Typical constant key |
|--------|--------|----------------------|
| `"1"` | 1 | `delete_success` |
| `"3"` | 0 | `not_authorized` |
| `"2"` | 0 | `elevated_access` |
| `"4"` | 0 | `valid_subscription_exists` |
| `"5"` | 0 | `delete_confirmation_message_active_reservation` |
| `"6"` | 0 | `generic_error` |
| `"7"` | 0 | `user_does_not_exist` |
| (empty `user_id`) | 0 | `generic_error` |

## 10. Generate FAQ

Retrieves localized FAQs and dynamic subscription plan descriptions with variables (pricing, duration) replaced in real-time.

- **Endpoint:** /api.php/v1/generate_faq

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                     |
|-------------------|----------|--------------|-------------------------------------|
| user_reference_id | Integer  | Yes          | User ID (reservation users) tracking.|
| user_lang         | String   | No           | BCP-47 Language code (default: en). |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "",
"data": [
{
"heading": "General Queries",
"items": [
{
"question_id": "1",
"question": "What is The Able Mind?",
"response": "The Able Mind is an app..."
}
]
}
]
}


### Runtime (`html/api/v1/api_generate_faq.php`)

| HTTP | When |
|------|------|
| 200 | Valid user, FAQs built; body `{"code":1,"message":"","data":[...]}`. |
| 400 | `user_reference_id` missing or `<= 0`. |
| 404 | User not found or blocked (`user_blocked <> 0` excluded by query). |
| 405 | Method not `POST` (checked after language file load). |
| 500 | Missing `language_config/journal/journal_config_{lang}.php`, DB/query failure, or uncaught `Throwable` in outer `catch` (`code` 0, `message` from `retrieve_processing_error`, `data` `[]`). |

**`data[]` section object**

| Field | Type |
|-------|------|
| `heading` | string |
| `items` | array of `{ "question_id", "question", "response" }` — `response` may include HTML snippets and substituted `(AAAA)` trial duration / `(XXXX)` plan list. |

## 11. Get Current Plan Summary

Returns a comprehensive summary of the user's active subscription plan, feature access, monthly usage limits, and expiry details.

- **Endpoint:** /api.php/v1/get_current_plan_summary

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                |
|-------------------|----------|--------------|--------------------------------|
| user_reference_id | Integer  | Conditional  | User ID (reservation users). Required unless `user_email` or `user_key` resolves the user via `get_user_id_by_primary_login`. |
| user_email        | String   | No           | Primary user email.            |
| user_key          | String   | No           | Fallback for user email/phone. |
| user_lang         | String   | No           | Language code.                 |
| client_ip_address | String   | No           | Forwarded to `get_current_plan_summary(...)` as last argument. |

**Success Schema (200 OK):**

json
{
"response_code": 1,
"status": "success",
"message": "success",
"data": {
    "status": "1",
    "regime": "new",
    "plan_show_app": "Premium active till 31 Dec 2026",
    "message": "Your Premium subscription is active till 31 Dec 2026",
    "plan_type": "PRM",
    "plan_source": "PURCHASE",
    "subscription_category_code": "PRM",
    "subscription_category_name": "Premium",
    "category_id": "3",
    "trial_category": "N",
    "end_date": "2026-12-31 23:59:59",
"fb15_minutes_per_day": 60,
    "fb15_minutes_per_month": 1800,
    "thot_words_per_month": 100000,
    "thot_response_time": 24,
    "tam_connections_enabled": "Y",
    "tide_over_together_enabled": "Y",
    "voice_notes": "Y",
    "personal_consultation_enabled": "Y",
    "subscription_category_global_usage": "N",
    "subscription_category_global_minutes": 0,
    "enable_thot_attachment": "Y",
    "org_id": "",
    "org_name": "",
    "delete_confirmation": {
      "type": "Q",
      "message": "Are you sure you want to delete your account?"
    }
  }
}

**Notes:**
- THoT limits are now enforced using monthly word limits.
- FB15 limits may include both daily and monthly caps depending on the subscription configuration.
- Message-count based THoT limits and tracking-unit fields are deprecated and no longer returned.

### Runtime (`html/api/v1/api_get_current_plan_summary.php`)

| Field | Type | Notes |
|-------|------|--------|
| `response_code` | int **or** string | Success path sets **integer** `1`. Initial template and `catch` use **string** `"0"` — clients should treat truthily, not by strict type. |
| `status` | string | `"success"` / `"failure"`. |
| `message` | string | `"success"` on OK; on empty decode **`no_plan`** constant; on error localized DB / **`no_plan`**. |
| `data` | object or array | On success, **exact object** returned by `get_current_plan_summary` (JSON-decoded). On failure **`[]`**. |

**POST inputs:** `user_reference_id`, optional `user_email` (also strips `+` for `user_key` resolution), optional `user_key` if email empty, `user_lang`, optional `client_ip_address`.

## 12. Get Current Share Permissions

Fetches the user's current database toggles for sharing data (Journal, Mood, Habits) with counsellors.

- **Endpoint:** /api.php/v1/get_current_share_permissions

- **Legacy handler file:** `html/api/v1/api_get_current_share.php` (see `api.json` mapping).

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Automatically decrypts binary/encrypted permission flags from the reservation_users table.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description** |
|-------------------|----------|--------------|-----------------|
| user_reference_id | Integer  | Yes          | User ID (reservation users).        |
| user_lang         | String   | No           | Language code.  |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Settings loaded successfully",
"data": {
"journal_shared": true,
"mood_tracker_shared": false,
"habit_tracker_shared": true,
"anxiety_tracker_shared": false,
"assessments_shared": true
}
}


### Runtime (`html/api/v1/api_get_current_share.php`)

| HTTP | When |
|------|------|
| 200 | User found; body `{"code":1,"message":"<settings_loaded constant>","data":{...}}`. |
| 400 | `user_reference_id` missing or invalid. |
| 404 | No row for user or `user_blocked <> 0`. |
| 405 | Not `POST`. |
| 500 | Missing journal language file, or `catch` with `retrieve_processing_error`. |

**`data` object (all booleans after decrypt)**

| Key | Meaning |
|-----|---------|
| `journal_shared` | |
| `mood_tracker_shared` | |
| `habit_tracker_shared` | |
| `anxiety_tracker_shared` | |
| `assessments_shared` | |

## 13. Get Random Quote

Retrieves a random motivational quote localized to the user's language.

- **Endpoint:** /api.php/v1/get_random_quote

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | No | User ID (reservation users) (for personalized tracking to prevent repeats). |
| user_lang | String | No | Language code (default: en). |

**Success Schema (200 OK):**

json
{
"quote": "The best way out is always through.",
"author": "Robert Frost"
}


### Runtime (`html/api/v1/api_get_random_quote.php`)

| Item | Detail |
|------|--------|
| **Envelope** | JSON object with exactly **`quote`** and **`author`** strings. |
| **Empty `user_reference_id`** | Forces `user_lang` to **`en`**, calls `get_random_quote_old` (legacy `tam_inspirational_quotes` table). |
| **With `user_reference_id`** | Calls `get_random_quote` (personalized `tam_inspirational_quotes_new` + history). |
| **Return handling** | Helpers return a **PHP array**; the API normalizes array vs JSON string before echoing. |
| **Errors** | `catch` → `{"quote":"","author":""}`. |
| **HEAD** | No JSON body (`echo` skipped when method is `HEAD`). |

## 14. Ignore Merge Account

Sets a flag to permanently dismiss prompts asking the user to merge duplicate accounts.

- **Endpoint:** /api.php/v1/ignore_merge_account

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**  | **Type** | **Required** | **Description** |
|-----------|----------|--------------|-----------------|
| user_id   | String   | Yes          | User ID (reservation users).        |
| user_lang | String   | No           | Language code.  |

**Success Schema (200 OK):** (`ignore_merge_flag_for_user` returns only `status`; there is no `data` string in the helper.)

json
{
"status": "success"
}


### Runtime (`html/api/v1/api_ignore_merge_account.php`)

| Item | Detail |
|------|--------|
| **Happy path** | `ignore_merge_flag_for_user($connection, $user_id)` returns JSON `{"status":"success"}`; script `json_decode`s then echoes. |
| **Catch** | `{"status":"error","data":"<localized or raw message>"}` — `data` holds `high_traffic`, `technical_issue`, or `$e->getMessage()` in the default branch. |
| **`$output` init** | Pre-initialized as `[]` so `HEAD` / edge paths do not reference an undefined variable. |

## 15. Initiate Subscription Payment

Creates a Razorpay Order ID for purchasing a subscription, factoring in GST, applicable referral discounts, and corporate pricing.

- **Endpoint:** /api.php/v1/initiate_subscription_payment

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Generates a secure txnid and stores preliminary INITIATED status in global_mysql_payment_table.

** Note: **
- Users with an active lite-duration plan may be prevented from purchasing overlapping lite-duration plans.
- Certain plan combinations may be automatically hidden or disabled based on active subscription status.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**             |
|-------------------|----------|--------------|-----------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users).                    |
| plan_type         | String   | Yes          | Subscription Category Code. |
| promo_attestation | String   | No           | Deep-link promo attestation token; consumed via `tam_consume_subscription_promo_attestation()` when present. |
| referral_code     | String   | No           | Ambassador referral code.   |
| user_lang         | String   | No           | Language code.              |

**Not read by legacy handler:** `subscription_price`, `subscription_discount` (pricing comes from `calculate_subscription_pricing()` + DB category/corporate/referral logic only).

**Success Schema (200 OK):** **`razorpay_json` is a JSON-encoded string** (the PHP script sets `"razorpay_json" => json_encode($json_data)`), not a nested object. Clients must `JSON.parse` that string if they need an object.

The top-level `amount` inside the parsed `razorpay_json` object is Razorpay **minor units** (integer), **not** rupees/dollars. See [Razorpay payment amounts](#razorpay-payment-amounts) for INR / USD / JPY / KWD examples. Monetary fields inside `notes` are **major units** and must not be divided by 100.

json
{
  "code": "1",
  "razorpay_json": "{\"key\":\"...\",\"amount\":150000,\"currency\":\"INR\",\"user_currency\":\"INR\",\"payment_currency\":\"INR\",\"order_id\":\"order_...\",\"prefill\":{...},\"notes\":{\"merchant_order_id\":\"...\",\"category\":\"PREMIUM\",\"payment_price_wo_fees\":\"1271.19\",\"payment_gst_fees\":\"228.81\",\"price_id\":\"12345\",\"note_amount_unit\":\"major\",\"note_amount_currency\":\"INR\"},\"theme\":{...},\"send_sms_hash\":true,\"method\":{...}}"
}


**Multi-currency examples (parsed `razorpay_json` excerpts):**

| Scenario | `amount` | `currency` | `notes.note_amount_currency` |
|----------|----------|------------|------------------------------|
| Domestic INR subscription | `150000` | `INR` | `INR` (major ₹1500.00) |
| USD payment (2 dp) | `1999` | `USD` | user/display currency code |
| JPY payment (0 dp) | `1500` | `JPY` | user/display currency code |
| USD fallback when display ≠ payment | `1999` | `USD` | user currency; `payment_currency` also `USD` |

**Additional runtime fields inside `razorpay_json`:** `user_currency`, `payment_currency`, `conversion_rate_to_user_currency`, `conversion_rate_to_payment_currency` (FX metadata for the Flutter checkout UI).


### Runtime (`html/api/v1/api_initiate_subscription_payment.php`)

| `code` | Body |
|--------|------|
| `"1"` | `razorpay_json` string as above (Razorpay checkout options + `order_id`). |
| `"0"` | `message` only — repurchase blocked, lite cap, missing user, gateway/DB `catch`, or invalid params (`error_payment_generic` / localized equivalents). |

**Additional notes:** `plan_functions.php` + `config_plans_text_{user_lang}.php`; transaction name `api_initiate_subscription_payment`. Repurchase window constant `UPGRADE_ONLY_WINDOW` = **5 days** (`5 * 24 * 3600` seconds). Lite **1 day** overlap rules still apply per script.

## 16. Send Registration OTP

Generates and sends a One-Time Password to the user's email address for pre-registration verification.

- **Endpoint:** /api.php/v1/send_registration_otp

- **Method:** POST

- **Rate Limit:** 20 requests / minute (Public Route)

**Parameters:**

| **Name**   | **Type** | **Required** | **Description**                         |
|------------|----------|--------------|-----------------------------------------|
| user_email | String   | Yes          | Email address to receive the OTP.       |
| user_lang  | String   | No           | Language code for localized email body. |

**Success Schema (200 OK):** *(on successful send, `code` is numeric `1` in `generate_otp_for_api`; failures may use `0`.)*

json
{
"code": 1,
"message": "Localized verification message (email masked in copy)",
"token": "md5_hex_of_otp",
"otp_error_message": "Localized generic OTP error hint"
}


### Runtime (`html/api/v1/api_send_registration_otp.php`)

| Item | Detail |
|------|--------|
| **Helper** | `generate_otp_for_api($connection, $user_email, $user_lang)` in `html/functions_otp.php` returns JSON **string**; script decodes to array. |
| **`code` typing** | Success / rate paths use **integer** `0` or `1`; `catch` uses **string** `"0"` — treat loosely. |
| **APCu** | OTP throttling and locks use APCu keys `login_token_OTP_*` / `otp_lock_*`. |
| **Failures** | Duplicate merged account, blocked user, invalid email, mail send failure, lock timeout, etc. — see helper branches. |

## 17. Share App Data via Email

Compiles a comprehensive HTML report of the user's active trackers (Journal, Mood, Anxiety, Habits, Assessments) and emails it securely to a designated counsellor.

- **Endpoint:** /api.php/v1/share_app_data_email

- **Legacy handler file:** `html/api/v1/api_share_email_counsellor.php` (`api.json` → `share_app_data_email`).

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Includes Strict regex filtering (/[<>{}\\|\\\\]/) on counsellor_message to prevent XSS injection. Validates email domain. Limited to 1 share per day per user.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | Integer | Yes | User ID (reservation users). |
| counsellor_email | String | Yes | Valid email address of the receiving counsellor. |
| counsellor_message | String | No | Custom message to inject into the HTML email. |
| user_lang | String | No | Lowercased from POST; drives `journal_config_{lang}.php` (default `en`). |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Progress report successfully shared with your counsellor."
}


### Runtime (`html/api/v1/api_share_email_counsellor.php`)

| HTTP | When |
|------|------|
| 200 | Success path after PHPMailer `send()`; body `{"code":1,"message":"..."}` with localized confirmation (`share_email_confirmation` + feature list). |
| 400 | Invalid or missing `user_reference_id`. |
| 404 | User missing / blocked. |
| 405 | Not `POST`. |
| 500 | Missing journal language file early exit **or** uncaught `Throwable` (`processing_error`). |

**Validation:** `counsellor_message` rejected if it matches `/[<>{}\[\]|\\\\]/`. At least one share flag must be decrypted `1` on the user; **one share per calendar day** per `counsellor_journal_shares`. Engagement must not be all empty (`share_email_no_data`). `code` is **integer** `0`/`1` in responses.

## 18. Share Private App Data (Permissions Toggle)

Updates the binary/encrypted database toggles dictating which trackers the user consents to share with their internal counsellor.

- **Endpoint:** /api.php/v1/share_private_app_data

- **Legacy handler file:** `html/api/v1/api_share_with_counsellor.php` (`api.json` mapping).

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**               | **Type** | **Required** | **Description**        |
|------------------------|----------|--------------|------------------------|
| user_reference_id      | Integer  | Yes          | User ID (reservation users).               |
| journal_shared         | Integer  | Yes          | 1 (true) or 0 (false). |
| mood_tracker_shared    | Integer  | Yes          | 1 (true) or 0 (false). |
| habit_tracker_shared   | Integer  | Yes          | 1 (true) or 0 (false). |
| anxiety_tracker_shared | Integer  | Yes          | 1 (true) or 0 (false). |
| assessments_shared     | Integer  | Yes          | 1 (true) or 0 (false). |
| user_lang | String | No | Drives `journal_config_{lang}.php` (default `en`). |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Settings updated successfully"
}


### Runtime (`html/api/v1/api_share_with_counsellor.php`)

| HTTP | When |
|------|------|
| 200 | Updates applied (or no-op when decrypted values already match); `{"code":1,"message":"<settings_updated constant>"}`. |
| 400 | Missing `user_reference_id`, missing any of the five `*_shared` POST keys, non-digit value, or value not in `{0,1}`. |
| 404 | User not found / blocked. |
| 405 | Not `POST`. |
| 500 | Missing journal language file at bootstrap **or** `processing_error` on `catch`. |

**POST:** all five keys are **required** (`isset`); values must be digit strings `0` or `1`. Updates only columns whose decrypted value differs; values stored encrypted via `$ncrypt->encrypt`.

## 19. Show Plans

Fetches all publicly available subscription plans relevant to the user.  
Plans may be filtered or hidden based on corporate restrictions, existing active plans, lite-plan eligibility rules, regional pricing, or account status.

- **Endpoint:** /api.php/v1/show_plans

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_key | String | No | Primary login key/email to determine corporate status. |
| new | String | No | Set to "1" to receive the updated structured JSON format. |
| user_lang | String | No | Language code. |

**Success Schema (200 OK):**

json
[
  {
    "code": "1",
    "message": "",
    "default_card_to_show": "card_PREM1M",
    "cards": [
      {
        "id": "plans_heading",
        "type": "L",
        "text": "Choose a plan"
      },
      {
        "id": "gift_card",
        "type": "G",
        "plan_name": "Gift a Subscription",
        "url": "https://example.com/gift-a-subscription.php?key=XXXX",
        "text": [
          "Gift premium emotional wellness support",
          "Flexible plan options available",
          "Instant delivery supported"
        ],
        "show_purchase_button": "Y",
        "show_purchase_text": "Gift Now"
      },
      {
        "id": "card_PREM1M",
"plan_code": "PREM1M",
        "type": "C",
        "plan_name": "Premium Monthly",
        "recommended_flag": "Y",
        "active": "N",
        "active_plan_message": "",
        "show_price": "Y",
        "subscription_price": "999",
        "gst_rate": "18",
        "transaction_fees": "2",
        "international_flag": "N",
        "user_currency": "₹",
        "conversion_rate_user": "1",
        "price_to_show": "₹999 valid for 1 month",
        "show_purchase_button": "Y",
        "show_purchase_text": "Purchase",
        "url": "https://example.com/confirm-payment.php?...",
        "payment_description": "Subscription payment for Premium Monthly",
        "text": [
          "Daily emotional wellness support",
          "Priority response time",
          "Voice notes enabled",
          "TAM Connections enabled"
        ]
      },
      {
        "id": "history_card",
        "type": "H",
        "text": "Purchase History",
        "plan_purchase_history": [
          {
            "package_type": "Premium Monthly",
            "start_date": "01 Jan 2026",
            "end_date": "31 Jan 2026",
            "package_price": "₹999",
            "payment_status": "Paid",
            "plan_booked_date": "Monday, 01 Jan 2026",
            "package_status": "Active",
            "active_status": "Y",
            "receipt_button_show": "Y",
            "receipt_button_label": "Download Receipt",
            "receipt_download_url": "https://example.com/subscription-pricing.php?download_app_subscription_receipt=XXXX"
          }
        ]
      }
    ]
  }
]

Card Types
| **Type** | **Description** |
|---|-----------------|
| L |Informational label/message card
| G |Gift subscription card
| C |Subscription plan card
| H |Purchase history card

** Notes: **
- Pricing returned is already localized based on the user currency and country.
- Purchase buttons may be disabled depending on:
	existing active plans,
	corporate restrictions,
	lite-plan purchase restrictions,
	maintenance windows.
- Text inside a plan card contains human-readable feature lines intended for direct UI rendering.
- Plan restrictions are now based on:
	subscription_category_fb15_month
	subscription_category_thot_words_month

### Runtime (`html/api/v1/api_show_plans.php`)

| Item | Detail |
|------|--------|
| **Body** | `show_plans_for_user($connection, $user_key, $user_lang, $new_json)` return value, `json_decode`d when string. **`new=1`** selects the newer structured payload builder inside `plan_functions.php`. |
| **Success** | Typically a **JSON array** whose first element is an object with `code`, `message`, `default_card_to_show`, `cards` (see illustrative §19 example). Exact card schemas depend on `plan_functions.php`. |
| **Catch** | Object shape `{"code":"0","message":"<localized error_retrieving_plans / DB / high_traffic / technical_issue>"}`. |

## 20. Subscription Payment Confirmation

Verifies the Razorpay payment signature, marks the order as paid in the database, activates the package, and dispatches multi-channel activation notifications.

- **Endpoint:** /api.php/v1/subscription_payment_confirmation

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Uses Razorpay PHP SDK for verifyPaymentSignature. Wrapped in a strict MySQL transaction to prevent partial package activation.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| razorpay_order_id | String | Yes | Order ID returned by Razorpay. |
| razorpay_payment_id | String | Yes | Payment ID returned by Razorpay. |
| razorpay_signature | String | Yes | Hash signature for backend verification. |
| user_lang | String | No | Language code for notification bodies. |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/subscription_payment_confirmation \
-H "Authorization: Bearer <TOKEN>" \
-d "razorpay_order_id=order_123abc" \
-d "razorpay_payment_id=pay_123abc" \
-d "razorpay_signature=a1b2c3d4e5f6g7h8..."


**Success Schema (200 OK):**

json
{
"code": "1",
"message": "Your Premium plan is now active.",
"payment_status": "CAPTURED"
}


### Runtime (`html/api/v1/api_subscription_payment_confirmation.php`)

| `code` | When |
|--------|------|
| `"-1"` | Missing `razorpay_order_id`, `razorpay_payment_id`, or `razorpay_signature`. |
| `"0"` | Signature verification failed (`error_transaction_invalid`), Razorpay order not `PAID` / payment_failed branches, DB/network `catch`, or other failures (`code` always string in these paths). |
| `"1"` | Paid order processed; `message` from `plan_confirmation` constant with `XXXX` replaced by category name; **`payment_status`** echoes Razorpay payment status (e.g. `"CAPTURED"`). |

**Amount verification:** After Razorpay signature verification, the handler normalizes order/payment entities and runs `tam_assert_razorpay_payment_amounts()` — `order.amount`, `payment.amount`, and (when present) the initiated `reservation_payments.payment_price` must agree in **minor units** for the payment currency. Failure surfaces as `"code":"0"` with localized `error_payment_amount_mismatch`. Ledger posting uses `process_razorpay_payment_logic()` (minor units on `payment.amount`; major units in `notes` only for audit metadata).

**Transport:** `respondAndExit` echoes the raw JSON string stored in `$output_data`. **`HEAD`:** the main POST block is skipped; the connection is closed without a response body.

## 21. Third Banner (Dashboard Data)

Aggregates key data for the mobile UI dashboard, including upcoming appointments, next available slots, and deep links.

- **Endpoint:** /api.php/v1/third_banner

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | User ID (reservation users). |
| slots | Integer | No | Number of upcoming slots to fetch (default: 2). |
| user_lang | String | No | Language code. |

**Success Schema (200 OK):**

Root keys from `html/api/v1/api_third_banner.php`:

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` success, `"0"` when `user_reference_id` empty or error. |
| next_appointment | mixed | Return value of `get_upcoming_reservation` (structure defined in `html/banner_functions.php` / helpers). |
| tot | mixed | `get_upcoming_tot_session($connection)`. |
| next_slot | object | Keys **`counsellor`** (string name or **""**) and **`next_counsellor_slot`** (array of slot strings, possibly empty). |
| tam_library_url | string | `global_url + "tam-app-library.php?token=" + encrypted user_reference_id`. |
| tam_resource_center_url | string | Same pattern for `tam-app-resource-center.php`. |

Example (illustrative shapes only):

json
{
"code": "1",
"next_appointment": {},
"tot": {},
"next_slot": { "counsellor": "", "next_counsellor_slot": [] },
"tam_library_url": "https://[BASE_URL]/tam-app-library.php?token=...",
"tam_resource_center_url": "https://[BASE_URL]/tam-app-resource-center.php?token=..."
}


## 22. Update User Access Code

Updates a user's corporate access code and/or employee ID to instantly grant them eligibility for specific B2B sponsored plans.

- **Endpoint:** /api.php/v1/update_user_access_code

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**    | **Type** | **Required** | **Description**                   |
|-------------|----------|--------------|-----------------------------------|
| user_id     | Integer  | Yes          | User ID (reservation users). Cast to `(int)` in handler; fallback lookup via `user_email` when zero. |
| user_email  | String   | No           | User's primary login key (leading `+` stripped). Used to resolve `user_id` when it is `0`. |
| access_code | String   | No           | Corporate or coupon access code. When **empty**, the current plan is removed. |
| employee_id | String   | No           | Corporate employee ID. Required when the target corporate plan has `authorized_only = 'Y'`. |
| user_lang   | String   | No           | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| response_code | mixed | Mirrors `code` from `update_user_access_code()`. Integer **`1`** on success, string **`"0"`** on failure. Special values: `"NOUSER"`, `3` (employee ID required). |
| message | string | Localized result message. |
| status | string | `"success"` or `"failure"`. |
| data | object | Full inner JSON returned by the helper (includes `code`, `message`, and optionally `end_date`, `source`). On error contains the error message or localized constant. |

**Runtime (`html/api/v1/api_update_user_access_code.php`)**

1. Delegates to `update_user_access_code($connection, $user_id, $access_code, $user_key, $employee_id, true, $user_lang)` in `functions.php`.
2. The helper returns a **JSON string**; the handler `json_decode`s it.
3. If the decoded JSON contains an `error` key, the response is wrapped with `status:"failure"` and `response_code:"0"`.
4. On success, `response_code` inherits the helper's `code` — which can be integer `1` (plan applied / removed / no change) or string `"NOUSER"` or integer `0` / `3`.
5. `HEAD` requests skip all processing; the `finally` block still echoes `$output` — but `$output` is **uninitialised** for `HEAD`, so clients must not parse the body.

Helper `code` values from `update_user_access_code()`:

| `code` | Meaning |
|---|---|
| `1` (int) | Access code applied, removed, or no-change. |
| `0` (int) | Invalid code, capacity exhausted, region mismatch, corporate ID not approved, or generic system error. |
| `"NOUSER"` (string) | User not found by ID or primary login. |
| `3` (int) | Employee ID required for authorized-only corporate codes. |

**Catch envelope (Throwable):**

```json
{ "response_code": "0", "message": "Error", "status": "failure", "data": "<localized constant>" }
```

## 23. Update User Information

Modifies core profile details, including contact numbers, guardian information (if the user is/was a minor), and access codes.

- **Endpoint:** /api.php/v1/update_user_information

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| primary_login_key | String | Yes | Email or Phone used to identify the user. Leading `+` stripped. Hashed via HMAC-SHA256 for DB lookup. |
| user_email | String | No | Updated email. Validated via `validate_api_user_email`. |
| user_phone | String | No | Updated phone number (`+`, `.`, space stripped). Validated via `validate_user_phone`. |
| user_guardian_name | String | Cond. | Required if minor; triggers guardian update. |
| user_guardian_phone | String | Cond. | Required if minor. Must differ from `user_phone`. |
| user_guardian_relationship | String | Cond. | Required if minor. |
| access_code | String | No | Corporate / coupon access code. Delegated to `update_user_access_code` at the end of the transaction. |
| employee_id | String | No | Corporate employee ID. Encrypted before storage. |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` success, `"0"` failure. |
| message | string | Localized result message. |

`code` is always a **string** — the helper `update_user_information_app` returns `json_encode(array("code" => "1", ...))`.

**Runtime (`html/api/v1/api_update_user_information.php`)**

1. Delegates to `update_user_information_app(...)` in `functions.php`, which returns a JSON string (or array in edge cases). The handler normalises with `is_string($data) ? json_decode($data, true) : $data`.
2. The helper runs inside a **transaction** (`begin_transaction` / `commit_transaction` / `rollback_transaction`).
3. Processing order: validate email → validate phone → validate guardian → update guardian → update PII (`reservation_users_p_details` + `reservation_users`) → log history → apply access code via `update_user_access_code`.
4. If the access code application fails and `access_code` was non-empty, the entire transaction is **rolled back**.
5. If `access_code` is empty and PII rows were updated, `code:"1"` with `profile_updated` constant is returned.

**Failure variants:**

| `code` | Scenario |
|---|---|
| `"0"` | Email invalid, phone invalid, guardian email/phone same as user, guardian update failed, access code invalid, user not found by `primary_login_key`. |
| `"-1"` | Exception caught in script-level `catch` block (high traffic / technical issue). |

**Catch envelope (Throwable — script level):**

```json
{ "status": "-1", "message": "<localized error constant>" }
```

Note: the script-level error uses `status` (not `code`) with value `"-1"`, while the helper's own errors use `code:"0"`.

## 23.1 Ambassador Program Signup (Legacy Web)

Self-service ambassador application flow served by the legacy PHP page `html/individual-ambassador-program.php`. This is **not** registered in `api.json` / Laravel `api.php`; it uses same-origin POST handlers on the page URL.

**Eligibility:** Applicants must be **18 years or older** (validated client-side and in `create_ambassador_signup`). India-based individuals only (enforced operationally at admin approval).

**Post-submit behaviour:** The applicant receives an **online confirmation only** (on-screen message + applicant email). Registration does **not** complete until an administrator approves the request in `cp_admin.php` and `create_new_ambassador_record` provisions `subscription_referral_master`.

**Implementation:** `html/functions_ambassador.php` (`generate_validation_token`, `create_ambassador_signup`, `ambassador_email_already_registered`).

### 23.1.1 Validate Email (send token)

- **Endpoint:** `individual-ambassador-program.php?validate_email`
- **Method:** POST (session cookie required)

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| ambassador_email | String | Yes | Applicant email. |
| ambassador_name | String | No | Used in token email salutation. |

**Response (`code` string):**

| `code` | Meaning |
|---|---|
| `"1"` | Token emailed; session `ambassador_email_verified_status` set to `"N"`. |
| `"0"` | Validation failure (invalid/disposable email, duplicate pending signup, or already an active ambassador). |

Duplicate detection checks `tam_ambassador_signup`, `subscription_referral_master`, and linked `reservation_users` hashes via `ambassador_email_already_registered`.

### 23.1.2 Confirm Email Token

- **Endpoint:** `individual-ambassador-program.php?confirm_email_token`
- **Method:** POST

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| token | String | Yes | 6-digit token from email; must match session `ambassador_email_token`. |
| ambassador_email | String | Yes | Email being verified. |

| `code` | Meaning |
|---|---|
| `"1"` | Token valid; returns HTML fragment for age/phone/gender/PAN fields. Session `ambassador_email_verified_status` → `"Y"`. |
| `"0"` | Invalid token. |

### 23.1.3 Complete Signup

- **Endpoint:** `individual-ambassador-program.php?signup`
- **Method:** POST

Requires session `ambassador_email_verified_status == "Y"` and `ambassador_email_verified_value` matching posted `ambassador_email`.

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| ambassador_name | String | Yes | Full name. |
| ambassador_email | String | Yes | Verified email. |
| ambassador_age | Number | Yes | Completed years; **18–80**. |
| ambassador_phone | String | Yes | Country code + national number (e.g. `91` + 10 digits). |
| ambassador_gender | String | Yes | `M`, `F`, or `O`. |
| ambassador_pan | String | Yes | PAN or `NA` / `N/A`. |

| `code` | Meaning |
|---|---|
| `"1"` | Signup row inserted (`tam_ambassador_signup_approved = 'N'`). Applicant + admin emails sent (`ambassador_registration`). Online confirmation message returned. |
| `"0"` | Email not authenticated, session mismatch, or already an active ambassador. |
| `"3"` | Duplicate mobile/email signup or invalid phone. |
| `"4"` | Age below 18 or above 80. |
| `"5"` | Invalid gender. |
| `"6"` | Invalid PAN. |

**Notifications on successful signup:**

| Recipient | `notification_type` | Producer |
|---|---|---|
| Applicant | `ambassador_registration` | `create_ambassador_signup` (applicant confirmation email) |
| Admin approvers | `ambassador_registration` | `create_ambassador_signup` (admin alert) |

**Admin approval (`cp_admin.php`):**

1. Pending sign-ups load via `?update_sign_up_ambassador_div` → `getsignupambassadordetails()` (`tam_ambassador_signup_approved = 'N'`).
2. **Approve** (India domicile): `.approve_ambassador` pre-fills the ambassador form; admin sets discounts/payouts and submits `?create_new_ambassador_record`.
3. **Reject**: dedicated **Reject** button on each pending row (India and non-India). Non-India **Approve** also offers a reject shortcut. Calls `?reject_ambassador` with `rejection_reason` → `reject_ambassador_request()`.
   - `non_india_domicile` — applicant outside India
   - `ineligible` — does not meet program criteria (default)
   - `duplicate` — duplicate application
   - `incomplete` — missing or unverifiable information
4. On successful approval, `create_new_ambassador_record`:
   - Inserts `subscription_referral_master` + `subscription_referral_financials_master`
   - Sets `tam_ambassador_signup_approved = 'Y'` for the linked `ambassador_signup_id`
   - Sends welcome email + WhatsApp (`ambassador_registration`, ~3714)
5. Rejection sets `tam_ambassador_signup_rejected = 'Y'` and emails the applicant using `build_ambassador_rejection_email_body()` for the selected `rejection_reason`.

**Access:** `user_is_admin`, `user_is_manager`, `user_is_receptionist`, or `user_is_ambassador_admin`.

**Registration status UI (`has_user_registered_previously`):**

| Status | Meaning |
|---|---|
| `R` | Active ambassador (`subscription_referral_master`) |
| `Y` | Signup row approved |
| `N` | Signup pending admin approval |
| `NR` | Not registered |

## 24. Validate Referral Code

Validates an ambassador referral code to dynamically calculate and apply discounts (routing logic based on domestic vs international pricing).

- **Endpoint:** /api.php/v1/validate_referral_code

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | No | User ID (reservation users). Used to determine country code for domestic / international pricing. When empty, defaults to Indian pricing. |
| referral_code | String | Yes | The referral code to validate. |
| plan_type | String | Yes | Subscription category code. Resolved to a `category_id` via `get_category_id_from_code`. |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` valid, `"0"` invalid / error. |
| discount | string | Percentage discount (e.g. `"10.00"`). Present only when `code:"1"` or `code:"0"` with an inactive/expired match. Empty string `""` when code invalid. Absent when `plan_type` or `referral_code` is empty. |
| message | string | Localized message. On success, uses `referral_code_message` constant with `{AAAA}` → discount % and `{BBBB}` → referrer first name. |

**Runtime (`html/api/v1/api_validate_referral_code.php`)**

1. Requires `functions_ambassador.php` (not `functions.php`).
2. Output variable is `$data` (not `$output`), echoed via `json_encode($data)` in `finally`.
3. Validation order: `referral_code` empty → `plan_type` empty → resolve `category_id` → SQL lookup in `subscription_referral_master` + `subscription_referral_financials_master`.
4. Pricing column selected dynamically: `ambassador_referral_domestic_discount` when `$india_pricing_flag` is `true`, else `ambassador_referral_international_discount`.
5. The query checks `subscription_referral_code_valid_till >= CURDATE()` and joins to `reservation_users` for referrer name and block/guardian status.
6. If `user_blocked != 0` or `guardian_approved != 'Y'`, returns `code:"0"` with `error_code_inactive` constant.

**Failure variants:**

| Condition | `code` | `message` |
|---|---|---|
| `referral_code` empty | `"0"` | `error_empty_code` constant |
| `plan_type` empty | `"0"` | `error_plan_missing` constant |
| `category_id` not found | `"0"` | `error_plan_missing` constant |
| No matching row in DB (expired / wrong plan) | `"0"` | `error_code_invalid` constant |
| Referrer blocked or guardian not approved | `"0"` | `error_code_inactive` constant |

**Catch envelope (Throwable):**

```json
{ "code": "0", "message": "<localized error constant>" }
```

## 25. Chat Summary (Feel Better in 15)

Fetches LLM-powered summaries for a user's past "Feel Better in 15" chat sessions to give them a quick clinical or mood overview of past interactions.

- **Endpoint:** /api.php/v1/chat_summary_fb15

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_id | String | Yes | The `users.id` (FB15 chat-system user ID), **not** `reservation_users.user_id`. Validated via `SELECT * FROM users WHERE id = $user_id`. |
| number_of_chats | Integer | No | Max *successful* summaries to collect. Default: `MIN_PREVIOUS_CHATS` constant. Sessions with ≤5 messages or failed LLM calls don't count toward this limit but still appear in the output. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` success, `"0"` failure. Always a **string**. |
| data | array | Array of summary objects (see below). Present only on `code:"1"`. Ordered **chronologically** (`array_unshift` builds reverse; result is oldest → newest). |
| message | string | Present only on `code:"0"`. |

**`data[]` object:**

| Field | Type | Notes |
|---|---|---|
| date | string | `"dd MMM YYYY hh:mm am/pm (timeAgo)"` e.g. `"10 May 2026 10:00 am (2 days ago)"`. |
| counsellor | string | `users.name` of the counsellor for that session. |
| chat_topic | string | LLM-extracted topic or `"No specific topic identified"`. HTML entity quotes replaced. |
| feedback_star | string | Star rating or `"Not Provided"` when empty. |
| summary | string | LLM-generated summary + `"<br><b>Close Chat Remarks:</b> "` + sentence-cased `close_remark`. May be `"Insufficient information..."` or `"Error while generating summary..."` for edge cases. |

**Runtime (`html/api/v1/api_chat_summary_fb15.php`)**

1. Looks up the user in `users` table (not `reservation_users`). If not found → `code:"0"`, `message:"User id is not valid"`.
2. If `user_id` is empty → `code:"0"`, `message:"User id does not exist"`.
3. Iterates closed sessions (`close_reason IS NOT NULL AND <> ''`), ordered DESC by `created_at`.
4. For each session: if `total_messages > 5` and no cached `chat_summary`, calls `llm_execute('generate_chat_summary', ...)` to generate summary and topic. Result is persisted to `tam_chat_sessions.chat_summary` / `chat_categorization`.
5. Sessions where `chat_summary` equals `"Insufficient information..."` or `"Error Generating Summary"` don't count toward the `number_of_chats` limit.
6. The local `toSentenceCase()` and `timeAgo()` helper functions are defined inline in the script (not in `functions.php`).

**Catch envelope (Throwable):**

```json
{ "code": "0", "message": "<localized error constant>" }
```

## 26. Check Chat Usage (FB15 / TOT)

Monitors and enforces usage limits for "Feel Better in 15" (FB15) and "Therapy Over Text" (THoT) based on the user's active subscription plan.

**Laravel mobile route:** There is **no** direct `POST /api/check_chat_usage` route (removed from `routes/api.php`). FB15 assignment and other Laravel flows call `EntitlementService::checkChatUsage()` → `CurlPhp::api_check_chat_usage()` server-side. The legacy script below remains the **authoritative** entitlement implementation.

- **Endpoint:** /api.php/v1/check_chat_usage

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Checks the user's active subscription category, global usage rules, and remaining monthly usage allowances before permitting access.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | User ID (`reservation_users.user_id`). Throws exception (code 400) when empty. |
| usage_check_type | String | No | `"fb15"` or `"tot"` (default: `"fb15"`). |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK — Usage Allowed):**

| Field | Type | Notes |
|---|---|---|
| code | integer | **`1`** = allowed. Always an **int** (not string). |
| minutes_available | integer \| string | Remaining FB15 minutes in the current 30-day cycle. Empty string `""` when check type is `tot` or plan is global. |
| words_available | integer \| string | Remaining TOT words in the current 30-day cycle. Empty string `""` when check type is `fb15` or plan is global. |
| text | string | Only present on global-plan override. Empty string for normal paths. |
| show_buttons | object | `{ "count": 0, "buttons": [] }` on success. |

**`show_buttons.buttons[]` object (when present):**

| Field | Type | Notes |
|---|---|---|
| type | string | `"redirect"` or `"close"`. |
| text | string | Button label (localized constant). |
| redirect_code | string | Target: `"subscription"`, `"therapy"`, `"fb15"`, `"profile"`, `"ticket"`, or `"close"`. |

**Runtime (`html/api/v1/api_check_chat_usage.php`)**

1. Calls `get_current_plan_summary(...)` from `plan_functions.php` to get the user's active plan. If no plan → exception (code 404).
2. **FB15 cooling period:** If `usage_check_type` is `"fb15"`, calls `check_fb15_session_cooling_period(...)` from `plan_functions.php`. If elapsed seconds < `global_fb15_cooling_period * 60`, returns `code:0` with the cooling-period message and support buttons.
3. **Global plan check:** Calls `user_global_plan_details(...)`. If the plan is global and enabled (`code:1`), returns `code:1` with unlimited access. If global but disabled (`code:0`), returns `code:0` with the global plan text and extra "access_code" redirect button.
4. **30-day cycle window:** `get_subscription_cycle_window(start_date)` computes the current billing cycle as 30-day slices from subscription start. Used for both FB15 and TOT usage queries.
5. **FB15 path:** `get_fb15_minutes_used(...)` sums session durations (first user message → last non-system message) for the cycle. If `minutes_left_cycle ≤ 10`, returns exceeded with THoT/plans/close buttons.
6. **TOT path:** `get_tot_usage(...)` sums `tam_async_chat_messages.word_count` for the cycle. If `words_left ≤ 0` and not a trial plan, returns exceeded with FB15/close buttons.
7. Output is encoded with `JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR`.

**Decision tree summary:**

| Condition | `code` | Key fields |
|---|---|---|
| User empty | 0 | `technical_issue` (via `catch`) |
| No active plan | 0 | `technical_issue` (via `catch`) |
| Cooling period active (FB15 only) | 0 | `text` with timer message, support buttons |
| Global plan enabled | 1 | Unlimited; empty minutes/words |
| Global plan disabled | 0 | `text` from global plan, access_code + support buttons |
| FB15 minutes ≤ 10 remaining | 0 | `text` from `exceeded_fb15_usage` constant, THoT + plans + close buttons |
| TOT words ≤ 0 remaining | 0 | `text` from `exceeded_tot_usage` constant, FB15 + close buttons |
| No subscription category code | 0 | `text` from `invalid_subscription_code` constant |
| Within limits | 1 | `minutes_available` or `words_available` populated |

**Catch envelope (Throwable):**

All `catch` paths set `text` to the localized **`technical_issue`** constant (MySQL errors and generic exceptions alike — exception messages are **not** echoed to clients).

```json
{ "code": 0, "text": "<technical_issue constant>", "show_buttons": { "count": 2, "buttons": [/* support + close */] } }
```

## 27. Audio Transcribe (Main)

Transcribes audio using an LLM abstraction layer. It supports Text-to-Speech (TTS) generation based on user gender and language.

- **Endpoint:** /api.php/v1/audio_transcribe

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes (1):** Verifies valid audio format via FFmpeg before delegating to the LLM to prevent injection attacks.
- **Security Notes (2):** The provided file path must reference a server-generated temporary upload and cannot reference arbitrary filesystem locations.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| audio_message | String | Yes* | Server path returned by /upload_audio. Sanitised via `htmlspecialchars`. *Not required if `check_lang_support` is sent. |
| check_lang_support | Flag | No | If present (any value), endpoint only checks language support — bypasses transcription entirely. |
| return_type | String | No | `"0"` = text only (default), `"1"` = audio only (TTS), `"2"` = text + audio. Compared with `==` (loose). |
| mode | String | No | `"save"` (default) returns a file URL, `"stream"` returns raw audio bytes (sets `Content-Type: audio/mpeg` and exits). |
| user_gender | String | No | `"M"`, `"F"`, or `"O"` (default). Mapped to Google TTS gender: `MALE` / `FEMALE` / `NEUTRAL`. |
| user_lang | String | No | Language code (default `"en"`). Used for Whisper language hint and TTS locale mapping. |

**Success Schema (200 OK):**

The response shape depends on `check_lang_support` vs transcription mode:

**Mode: `check_lang_support`**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1` = supported, `0` = unsupported, `-1` = no language code provided. |
| text | string | `"supported"`, localized `translation_improvement` message, or `"no language code"`. |

**Mode: Transcription (`return_type` = 0)**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1` success, `0` failure. |
| text | string | Transcribed text. On failure: localized `transcription_warning` or `voice_note_missing`. |
| language | string | Detected language BCP-47 code (e.g. `"en"`). `not_detected` constant on failure. |
| language_full | string | Full language name from `$app_languages_name` map. |

**Mode: Transcription (`return_type` = 1 — audio only)**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1`. |
| language | string | Detected language code. |
| language_full | string | Full language name. |
| audio | string | File URL path (when `mode:"save"`) or raw bytes stream (when `mode:"stream"` — response exits with `Content-Type: audio/mpeg`). |

**Mode: Transcription (`return_type` = 2 — text + audio)**

All fields from `return_type:0` plus `audio` field.

**Runtime (`html/api/v1/audio_transcribe.php`)**

1. Does **not** require `main.php` or `functions.php`. Directly loads `config.php` and FFmpeg autoloader.
2. Defines helper functions inline: `fetchGoogleVoices`, `selectBestVoice`, `buildTTSRequestPayload`, `streamGoogleTTS`, `isValidAudioFile`, `transcribeAudioWithAbstraction`, `ensureDirExists`.
3. Transcription: `transcribeAudioWithAbstraction` validates audio via FFmpeg, copies/converts to WAV if needed, then delegates to `llm_execute('audio_story_analysis', ...)`.
4. TTS voice selection: `selectBestVoice` tries Neural2 → Wavenet → Standard tiers, matching language + gender. Google voices list is cached in APCu for 24h.
5. `streamGoogleTTS` calls Google Cloud TTS API. In `"save"` mode, writes MP3 to `translate_api/voice_notes/` and returns the relative URL. In `"stream"` mode, echoes raw bytes with audio headers and **exits** (no JSON wrapper).
6. Temporary files are cleaned up after transcription (`unlink`).
7. `HEAD` requests skip all processing; the `finally` block echoes `$output` only for non-HEAD.
8. The `catch` block catches `Exception` (not `Throwable`), setting `code:0` with the exception message.

## 28. Generate Chat Summary (Cron / Manual)

Generates a structured clinical summary for a specific chat session using an LLM. Operates in two modes: single-session (API call) and batch backlog (CLI cron).

- **Endpoint:** /api.php/v1/generate_chat_summary

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| session_id | Integer | Cond. | `tam_chat_sessions.session_id` to summarize. Must be numeric. Required when not running in batch mode. |
| update_all | String | Cond. | CLI-only: pass `"update_all"` as `$argv[1]` to process the entire backlog of sessions without summaries (created after 2024-01-01). |

At least one of `session_id` or `update_all` must be provided; otherwise the script echoes `{"error":"NO PARAMETER PROVIDED"}` and exits.

**Success Schema (200 OK — single session):**

| Field | Type | Notes |
|---|---|---|
| close_summary | string | LLM-generated summary text. On failure: `"Error while generating the summary"` or `"Insufficient information in chat to generate a summary"`. |

**Runtime (`html/api/v1/generate_chat_summary.php`)**

1. Two modes determined at startup: if `session_id` is present and numeric → `$close_chat_summary = true` (single session). If `$argv[1] == 'update_all'` → batch cron mode.
2. **Single-session mode:**
   - Queries `tam_chat_messages` count for the session. If `total_messages ≤ 5`, sets `close_summary` to "Insufficient information..." and persists that to DB.
   - Otherwise, concatenates all messages (with role labels) via `GROUP_CONCAT`, sanitises special characters, decrypts user gender, then calls `llm_execute('generate_chat_summary', ...)`.
   - On LLM success: persists `chat_summary` + `chat_categorization` and returns `{"close_summary": "<summary>"}`.
   - On LLM failure: logs to `cron_logs/Close_Chat_Summary`, persists nothing, returns `{"close_summary": "Error while generating the summary"}`.
3. **Batch cron mode:**
   - Sets `max_execution_time` to `0`.
   - Selects all sessions where `chat_summary` is NULL or empty, `close_reason` is set, and `created_at > '2024-01-01'`.
   - Iterates and generates summaries via the same LLM call. On failure, sets `chat_summary = null` (allows retry on next run).
   - Logs progress and count to `cron_logs/Chat_Summary`.
   - Does **not** return JSON to stdout in batch mode (output is log-only); the `finally` block still calls `echo json_encode($output)` but `$output` may be empty.
4. Message sanitisation replaces `"`, `'`, `` ` ``, `{`, `}`, `[`, `]` with typographic equivalents and converts `<br>` to newlines before sending to LLM.
5. The `ncrypt->decrypt` call on `users.gender` determines the pronoun context passed to the LLM.

**Catch envelope (Throwable):**

Single-session: `{"close_summary": "Error while generating the summary"}`.
Batch: logs error and stack trace; no JSON output.

## 29. Generate WebSocket Token

Issues an HS256 JWT for WebSocket authentication (TAM Connections real-time messaging).

- **Endpoint:** /api.php/v1/generate_ws_token

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Authentication:** Bearer required per `html/api.php`. This script does **not** require the bearer to match the `user_reference_id`; callers must align them at the application layer.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | Integer | Yes | Reservation user ID. Cast to `(int)`; values `<= 0` throw exception (code 400). |
| user_lang | String | No | Language code (default `"en"`). Only used on failure paths for localized error messages. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success. |
| message | string | `"JWT successful"`. |
| data | object | `{ "jwt": "<HS256 token string>" }`. |

**JWT payload claims:**

| Claim | Value |
|---|---|
| `iss` | `"theablemind_web"` |
| `aud` | `"tam_api"` |
| `id` | `user_reference_id` (integer) |
| `name` | Decrypted `reservation_users_p_details.user_name` (empty string if user not found or blocked). |
| `iat` | `time()` at generation. |
| `exp` | `time() + 3600` (1-hour lifetime). |

**Runtime (`html/api/v1/generate_websocket_token.php`)**

1. The `api.json` key is `generate_ws_token` → maps to script `generate_websocket_token.php`.
2. Defines an inline `get_username($conn, $user_id)` function that queries `reservation_users_p_details` joined with `reservation_users` (filters `user_blocked = 0`). Returns decrypted `user_name` or empty string.
3. If `user_id <= 0`, throws `Exception("User not provided", 400)`.
4. JWT is generated via `JWT::encode($payload, tam_connections_jwt_secret, 'HS256')` — the `JWT` class is loaded via `main.php` autoloading.
5. Unlike most handlers, the `$output` variable is set inside the `try` block **after** JWT generation, so on `HEAD` requests (which skip nothing — this handler has no HEAD guard on the main logic), the JWT is still generated but the response body is suppressed in `finally`.

**Failure Schema:**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `false`. |
| message | string | Exception message text (e.g. `"User not provided"`, `"Database prepare statement failed: ..."`, or localized constant). |
| data | array | Empty array `[]` (not object). |

**Notes:**

- WebSocket URL construction is client-side; token is passed as `socket/?token=<jwt>` per mobile client.
- JWT `id` claim is the numeric reservation `user_id` passed as `user_reference_id`.

## 30. Get Complete YSAM Post

Retrieves the full, un-truncated content of a specific "Your Story and Mine" (YSAM) post.

- **Endpoint:** /api.php/v1/get_complete_ysam_post

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**              |
|-------------------|----------|--------------|------------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users) requesting the post. |
| ysam_id           | String   | Yes          | The ID of the YSAM post.     |

**Success Schema (200 OK):**

`ysam_show_modal_story` returns a **JSON string**. `get_complete_ysam_post.php` sets:

```json
{
"code": "1",
  "data": "<string: inner JSON from ysam_show_modal_story — clients must json-decode this string to obtain the object below>"
}
```

After decoding **`data`**, success object fields (`functions.php` `ysam_show_modal_story`, app branch) are:

| Field | Type | Notes |
|---|---|---|
| code | string | Inner **`"1"`** (duplicates outer meaning). |
| ysam_id | integer | |
| ysam_user_story_title | string | HTML-escaped, newlines → `<br>`. |
| ysam_user_name | string | HTML-escaped decrypted name. |
| ysam_user_story_submitted_date | string | HTML-escaped elapsed string. |
| story | string | HTML-escaped body with `<br>` for newlines. |
| ysam_user_id | integer | Author id. |
| hashtags | array | `{ id: int, value: string }` entries, values HTML-escaped. |
| ysam_collection_type_name | string | |
| following | string | `reservation_user_ysam_following_active` or **`"N"`**. |
| connected_to_user | string | Connect approved **`"Y"`** / **`"N"`**. |
| estimated_read_time | string | |
| request_connect_to_user_message | string | Localized template with author name. |
| connection_confirm_button | string | Constant. |
| connection_cancel_button | string | Constant. |
| tam_connections_conversation_active | string | **`"Y"`** / **`"N"`** from `check_existing_ysam_conversation`. |
| story_language | string | Detected post language. |
| show_translate_option | string | **`"Y"`** or **`"N"`**. |
| story_translated_title | string | Empty when same language as UI. |
| story_translated_body | string | Empty when same language. |
| translate_label_text | string | |
| original_label_text | string | |

Failure inner JSON: `{"code":"0","message":"..."}`.


## 31. Google Audio Transcribe (Fallback)

Converts incoming server-side audio files to FLAC format and transcribes them directly via Google Speech-to-Text API. This is a **fallback** endpoint used when the primary LLM-based transcription (§27) is unavailable.

- **Endpoint:** /api.php/v1/google_audio_transcribe

- **Method:** POST

- **Content-Type:** application/json

- **Rate Limit:** 40 requests / minute

- **Security Notes:** The provided file path must reference a server-generated temporary upload and cannot reference arbitrary filesystem locations.

**Parameters (JSON Body):**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| audio_message | String | Yes | Server path returned by /upload_audio. Read from raw `php://input` JSON body. |
| user_lang | String | No | Language code for localized error messages (default `"en"`). **Not** the audio language hint — audio language is auto-detected via OpenAI Whisper. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` success, `"0"` failure. Always a **string**. |
| language | string | BCP-47 language code returned by Google Speech-to-Text (e.g. `"en-US"`, `"hi-IN"`). |
| transcript | string | The transcribed text. |

**Runtime (`html/api/v1/google_audio_transcribe.php`)**

1. Reads JSON body via `file_get_contents('php://input')`, not `$_POST`.
2. Processing pipeline: FFmpeg converts audio → FLAC → OpenAI Whisper detects language → maps full language name to BCP-47 via `bcp_47_codes_supported` constant → Google Speech-to-Text transcribes with that locale.
3. Inline helpers: `copyFile`, `detect_language` (calls OpenAI Whisper `verbose_json`), `transcribeAudioWithGoogle` (Google Cloud Speech V1 via service account).
4. The `detect_language` function calls `api.openai.com/v1/audio/transcriptions` with `whisper-1` model to get `response_format: verbose_json`. Only the `language` field (full English name) is used.
5. The detected language name (e.g. `"hindi"`) is looked up in `bcp_47_codes_supported` to get the Google locale code (e.g. `"hi-IN"`). If no mapping exists → `code:"0"` with `error_understanding_audio`.
6. Both the FLAC temp file and original audio file are deleted after processing (`unlink`).
7. **Important:** Some responses are echoed directly inside the `try` block (not via `$output`), and the `finally` block also echoes `json_encode($output)`. This means for the success path, the JSON from `transcribeAudioWithGoogle` is echoed directly, and then an **additional** `json_encode($output)` may be echoed (where `$output` is undefined). Clients should parse only the first JSON object.

**Failure variants:**

| Condition | Response |
|---|---|
| Empty body or missing `audio_message` | `{"code":"0","error":"<issue_processing_voice_note>"}` |
| FFmpeg conversion fails | Exception with `conversion_failed` constant |
| Whisper returns no language | `{"code":"0","error":"<trouble_processing_audio>"}` |
| Language not in BCP-47 map | `{"code":"0","error":"<error_understanding_audio>"}` |
| Google Speech returns no alternatives | `{"code":"0","error":"<trouble_processing_audio>"}` |
| Google Speech exception | `{"code":"0","message":"<issue_processing_voice_note>"}` |

**Catch envelope (Throwable):**

```json
{ "code": "0", "message": "<issue_processing_voice_note constant>" }
```

## 32. List YSAM Posts

Fetches a paginated feed of YSAM posts with optional filters for hashtags, categories, or specific authors.

- **Endpoint:** /api.php/v1/list_ysam_posts

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Pagination Rules:** Uses page_number (offset) and posts_per_page (limit).

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | User ID (reservation users). |
| page_number | Integer | No | Page number (default 1). |
| posts_per_page | Integer | No | Limit per page (default 10). |
| post_reference_user_id | String | No | Filter by specific author ID. |
| hashtag_id | String | No | Filter by hashtag string. |
| category | String | No | Filter by category ID. |
| search_criteria | String | No | Search keyword. |
| only_count | String | No | "Y" to return only total counts instead of post data. |

**Success Schema (200 OK):**

`get_all_ysam_posts` with `$redirect_from_app === true` returns an array `["code" => 1, "count" => <int>, "data" => <array of post objects>]`. `list_ysam_posts.php` wraps again:

```json
{
"code": "1",
"message": "",
"data": {
    "code": 1,
    "count": 0,
    "data": []
}
}
```

So **`data.data`** is the post array; **`data.count`** is total rows for the query (before pagination). When there are no rows, inner return is `{"code":1,"count":0,"message":"<localized string>}` (still `code` integer **1** in PHP array for that branch). The outer wrapper still uses `"code":"1"` string and puts the whole inner array under `data`.

### `data.data[]` post object (`get_all_ysam_posts`, app branch)

When `user_reference_id` is non-empty (typical app call), each element includes at least:

| Field | Type | Notes |
|---|---|---|
| ysam_id | mixed | Post id. |
| ysam_user_story_title | string | Truncated title (≤40 chars + `...`). |
| author_user_reference_id | int | Same as `ysam_user_id` in PHP. |
| ysam_user_name | string | Decrypted display name. |
| ysam_user_story_submitted_date | string | Humanized elapsed time (`time_elapsed_string` with lang). |
| story | string | Stripped/truncated body HTML for list. |
| read_more | string | **`"Y"`** or **`"N"`**. |
| narration | string | Absolute or relative URL to narration audio, or **""**. |
| hashtags | array | List of `{ "id", "value" }` when `redirect_from_app`; else raw string from SQL. |
| ysam_user_id | int | Author reservation id. |
| following | string | Follow flag / id from SQL, or **""** if no viewer id. |
| connected | string | Connect approved flag, or **""**. |
| ysam_published_status | mixed | |
| ysam_collection_type_name | string | |
| ysam_collection_type_id | mixed | Special handling when `1006` for read-time / body rules. |
| estimated_read_time | string | Localized minute string when `redirect_from_app`. |
| ysam_conversation_active | string | From `check_existing_ysam_conversation` or **""** / **"N"`**. |
| post_language | string | `ysam_detected_language`. |
| podcast_duration | string | Formatted duration or **""**. |
| language | string | User/content language code. |
| podcast_language | string | |
| podcast_language_text | string | May be unset if no podcast language. |
| translate_label_text | string | Empty when same language. |
| original_label_text | string | Empty when same language. |
| original_story | string | Empty when same language. |
| original_title | string | Empty when same language. |

When `user_reference_id` is empty, `following` / `connected` / `ysam_conversation_active` differ (see `html/functions.php` around the two `ysam_titles[]` branches).

**Catch envelope (Throwable):** flat top-level object — **not** wrapped in `status` / `data`:

```json
{ "code": 0, "count": 0, "message": "<localized high_traffic | technical_issue | processing_error>" }
```

## 33. Connections Block User

## Endpoint

```text
POST /api.php/v1/connections_block_user
```

## Purpose

Deactivates the 1:1 TAM Connections conversation between two reservation users when plan rules allow blocking.

## Authentication

Bearer required (`html/api.php`). `auth_user_id` is not read by this script.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| blocked_user_id | string | Yes | — | Escaped with `mysqli_real_escape_string`; cast to `(int)` inside `tam_connections_block_user`. |
| blocking_user_id | string | Yes | — | Same as above. |
| user_lang | string | No | en | Selects `language_config/ysam_conversation/common/config_ysam_{user_lang}.php` inside helper. |

## Success Response

`tam_connections_block_user` returns `status` **true**, `message` from localized constant `user_has_been_blocked_text`, and `data` containing at least:

```json
{
"status": true,
  "message": "<localized string>",
  "data": {
    "conversation_id": 0,
    "blocked_user_id": 0
  }
}
```

### `data` object (success)

| Field | Type | Notes |
|---|---|---|
| conversation_id | integer | `tam_conversations_master_id` updated row. |
| blocked_user_id | integer | The blocked party’s user id. |

## Failure Response

Representative shapes (all use `status` **false**, `data` **[]**, `message` localized constant or `processing_error`):

| Condition | `message` source (constant name) |
|---|---|
| Invalid user ids | `processing_error` |
| No conversation row for derived `room_{min}_{max}` room | `error_blocking_user_text` |
| Plan summary error | `user_plan_insufficient_for_block` |
| Short / trial plan (`plan_duration` contains `day`) | `user_plan_insufficient_for_block` |
| Update affects 0 rows | `error_blocking_user_text` |
| Throwable in helper | `processing_error` |

Wrapper catch (e.g. missing user id before helper): `message` is exception string such as `User Id is not specified`. Invalid JSON from helper yields `message` **`invalid_request`**.

## Notes

- Room name inside helper: `room_{min(blocking,blocked)}_{max(blocking,blocked)}` (stored in `tam_connections_conversation_master.tam_conversations_room_name`).

## 34. Connections Check Consent

## Endpoint

```text
POST /api.php/v1/check_connections_consent
```

## Purpose

Determines whether the user must accept the current Connections guideline text for their language.

## Authentication

Bearer required. Script does not read `auth_user_id`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_id | integer | Conditional | 0 | If empty and `user_reference_id` is set, `user_id` is copied from `user_reference_id` before cast. |
| user_reference_id | integer/string | Conditional | — | Alias copied into `user_id` when `user_id` empty. |
| user_lang | string | No | en | Must exist as `language_config/ysam_conversation/common/config_ysam_{user_lang}.php` and matching `config_ysam_{user_lang}.php` (both required). |

## Success Response

**Consent not required (row exists and `tam_connections_consent_expiry_date` ≥ server date `Y-m-d`):**

```json
{
  "status": true,
  "message": "success",
  "code": "1",
  "data": {
    "required": false
  }
}
```

**Consent required (no row, expired, or first-time):**

```json
{
"status": true,
"message": "consent_required",
  "code": "2",
  "text": { },
  "consent_string": "<imploded constant text>",
"data": {
"required": true,
    "guidelines": { }
  }
}
```

`text` and `data.guidelines` are the same object. Object `guidelines` (and root `text` when present) contains exactly these keys from `generate_connection_guidelines` in `tam_connections_check_consent.php`:

| Field | Type | Notes |
|---|---|---|
| title | string | Localized constant `title`. |
| introduction_1 | string | Constant `introduction_1`. |
| introduction_2 | string | Constant `introduction_2`. |
| introduction_guidelines | string | Constant `introduction_guidelines`. |
| consent_line_1 | string | Constant `consent_line_1`. |
| consent_line_2 | string | Constant `consent_line_2`. |
| consent_line_3 | string | `{XXXX}` replaced by `ysam_messages_limit_per_day`. |
| consent_line_4 | string | Constant `consent_line_4`. |
| consent_line_5 | string | Constant `consent_line_5`. |
| consent_line_6 | string | Constant `consent_line_6`. |
| consent_line_7 | string | Constant `consent_line_7`. |
| consent_line_8 | string | Always **""**. |
| consent_line_9 | string | Always **""**. |
| consent_final | string | Constant `consent_final`. |
| consent_text | string | Constant `consent_button`. |
| cancel_text | string | Constant `reject_button`. |

`consent_string` is the server-side canonical text: `implode('~~', [title, introduction_1, …, consent_final])` (excludes `consent_text` / `cancel_text`).

## Failure Response

**Invalid user id (`user_id` ≤ 0 after resolution):**

```json
{
  "status": false,
  "message": "<generic_error constant>",
  "code": "3",
  "data": []
}
```

**Throwable in script:** `status` **false**, `message` from `high_traffic` / `technical_issue` / `generic_error` constants depending on mysqli error code; `data` **[]** (no `code` field on this path in PHP).


## 35. Connections Check Usage

## Endpoint

```text
POST /api.php/v1/connections_check_usage
```

## Purpose

Returns whether the reservation user may send more TAM Connections messages today (plus staff bypass).

## Authentication

Bearer required. Script does not read `auth_user_id`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_reference_id | string | Yes | — | Passed through `mysqli_real_escape_string` then cast `(int)` in `check_usage_tam_connections`. |
| user_lang | string | No | en | Restricted to the same allow-list array as other Connections v1 scripts; invalid values become `en`. |

## Success Response

```json
{
"status": true,
  "message": "success",
  "data": {
    "allowed": true
  }
}
```

## Failure Response

```json
{
  "status": false,
  "message": "<localized string>",
  "data": {
    "allowed": false
  }
}
```

`message` values from `check_usage_tam_connections` include localized `processing_error`, `not_subscribed`, `exceeded_usage` (with `{XXXX}` replaced by `ysam_messages_limit_per_day`), or `processing_error` on exception.

## Notes

- If PHP session flags `user_is_admin`, `user_is_manager`, `user_is_counsellor`, `user_is_ops_lead`, or `user_is_receptionist` are set, the helper returns `allowed` **true** without quota checks.


## 36. Connections Dashboard

## Endpoint

```text
POST /api.php/v1/user_conversations_dashboard
```

## Purpose

Returns merged 1:1 and group inbox rows, unread total, discoverable groups, and localized tab labels.

## Authentication

Bearer required. Script does not read `auth_user_id`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_reference_id | string | Yes | — | Escaped then passed to `list_ysam_user_conversations`; empty yields `generic_error` envelope. |
| user_lang | string | No | en | Restricted to mobile allow-list; invalid → `en`. |

## Success Response

### Raw JSON from `list_ysam_user_conversations` (before `tam_connections_dashboard.php`)

Emitted as a **JSON string** then `json_decode`d. On success, root keys are:

| Field | Type | Notes |
|---|---|---|
| code | string | `"1"` success, `"0"` failure. |
| conversations | array | Merged 1:1 + group rows (max 100). See table below. |
| total_unread | integer | Sum of `unread_messages` for **active** conversations, de-duplicated by `connection_type` + `id`. |
| discover_groups | array | Up to 100 groups user may join. See table below. |
| tabs | object | Keys `conversations`, `discover` — localized UI tab titles from constants. |
| show_discover_tab | boolean | `true` when `discover_groups` is non-empty. |
| dashboard_message | string | Localized heading from `connections_dashboard_heading` or `no_connections_dashboard_heading`. |

### Wrapped JSON from `tam_connections_dashboard.php`

When `code` is present, the script **replaces** the whole output with:

| Field | Type | Notes |
|---|---|---|
| status | boolean | `code == "1"`. |
| message | string | `$output["text"] ?? ""` — usually **empty** on success (inner JSON has no `text`). |
| data | object | **Only** three keys are forwarded: `conversations`, `total_unread`, `discover_groups`. |

**Dropped by the wrapper (present only in raw inner JSON):** `tabs`, `show_discover_tab`, `dashboard_message`, inner `code`. Clients that need those fields must read the inner payload before wrapping or extend `tam_connections_dashboard.php`.

```json
{
"status": true,
  "message": "",
"data": {
"conversations": [],
    "total_unread": 0,
"discover_groups": []
}
}
```

### `data.conversations[]` row (`list_ysam_user_conversations`)

Built in `html/functions_ysam.php`. `connection_type` **`S`** = single chat, **`G`** = group.

| Field | Type | S | G | Notes |
|---|---|:---:|:---:|---|
| connection_type | string | ✓ | ✓ | `"S"` or `"G"`. |
| conversation_master_id | integer | ✓ | ✓ | For **S**: `tam_conversation_summary.conversation_id`. For **G**: `tam_connections_groups.tam_connections_group_id`. |
| conversation_active | boolean | ✓ | ✓ | From summary / group `active` flag (`"Y"` → true). |
| blocked_by_current_user | boolean | ✓ | ✓ | Single: inactive conversation and `inactive_by` equals current user. Group: always false in loop. |
| room | string | ✓ | ✓ | Room name (`tam_conversation_summary.room_name` or group room name). |
| following_user_id | integer \| null | ✓ | — | Other user id in 1:1; **null** for groups. |
| connection_group_name | string \| null | — | ✓ | Group display name; **null** for single. |
| connection_description | string \| null | — | ✓ | Group description; **null** for single. |
| subscribed_to_group | boolean | ✓ | ✓ | Always **true** when `connection_type === "G"`; always **false** when `"S"` (literal in PHP). |
| user_name | string | ✓ | ✓ | Single: decrypted peer `user_ysam_name` / `user_name` after batch map. Group: remains **""**. |
| last_message | string | ✓ | ✓ | Prefix **`You: `** when last message user is current user. Body from `buildMessagePreview`; empty text may become localized `connections_system_start_conversation`. |
| display_date | string | ✓ | ✓ | After hydration: `formatDisplayDateWithTZ` using **viewing** user’s timezone (not raw DB string). |
| unread_messages | integer | ✓ | ✓ | From summary / `tam_group_unread`. |
| target_language | string | ✓ | ✓ | Currently hard-coded **`"en"`** in PHP. |
| last_message_type | string | ✓ | ✓ | e.g. `text`, `image`, from summary. |

### `data.discover_groups[]` row

| Field | Type | Notes |
|---|---|---|
| group_id | integer | `tam_connections_group_id`. |
| room | string | `tam_connections_group_room_name`. |
| connection_group_name | string | Group name. |
| connection_description | string | Group description. |
| member_count | integer | Current member count. |
| group_subscribed | string | Literal **`"N"`** (user is not in this list if already subscribed). |

## Failure Response

Inner JSON uses `"code": "0"` with root key **`error`** (no `text`). The dashboard wrapper still maps `status` to `(code == "1")` and `message` to `$output["text"] ?? ""`, so this failure returns **`message` as an empty string** with `data` fields defaulting to empty arrays / zero.

## Notes

- Unread is incremented only for rows where `conversation_active` is true and each `connection_type_id` is seen once per response build.
- After building the list, PHP runs `UPDATE tam_connections_groups_subscriptions SET tam_connections_groups_user_last_connected = NOW()` for the current user (non-blocking side effect).


## 37. Connections Get Messages

## Endpoint

```text
POST /api.php/v1/get_connection_messages
```

**Laravel proxy (mobile):** `POST /api/get_connection_messages` — Passport `auth:api`; forwards `user_id` from token `reference_user_id` (never trusts a mismatched client `user_id`). Pre-checks room access via `Crit002eRoomAccessService` (deny envelope below). Returns legacy JSON decoded as-is (HTTP 200).

## Purpose

Returns up to **50** message rows for one **single-chat** room (`tam_connections_conversation_master`), either newer than `lastSeq` or strictly older than `beforeSeq`. Bodies are **decrypted** in `tam_connections_replay_message_row()` before echo.

## Authentication

Bearer required. User id resolution: `$_REQUEST['auth_user_id']` if set (JWT path in `html/api.php`), else `$_POST['user_id']`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| room_name | string | Yes | — | Empty with empty user → `missing_fields`. |
| user_id | string/int | Conditional | — | Required when JWT does not populate `auth_user_id`. |
| lastSeq | mixed | No | 0 | Used only when `beforeSeq` is **0**; bound as integer for SQL `message_sequence > ?`. |
| beforeSeq | integer | No | 0 | When `> 0`, selects rows with `message_sequence < ?` ordered DESC then reversed; fetches `pageLimit+1` to set `has_more_older`. |
| user_lang | string | No | en | Loads `language_config/ysam_conversation/config_ysam_{user_lang}.php` (for `missing_fields` / `technical_issue` constants). |

`room_type` is **not** read by this script; room access is inferred via `tam_connections_assert_can_read_room(..., null)`.

## Success Response

```json
{
  "status": true,
  "message": "success",
  "data": {
    "messages": [
      {
        "message_sequence": 1,
        "server_seq": 1,
        "client_message_id": "uuid-or-null",
        "sender_id": 1,
        "message_text": "decrypted plain text",
        "message_type": "text",
        "attachment_url": null,
        "created_at": "YYYY-MM-DD HH:MM:SS",
        "is_mine": true,
        "original_message": null,
        "delivery_status": "sent"
      }
    ],
    "has_more_older": false
  }
}
```

### `data.messages[]` row (`tam_connections_replay_message_row`)

| Field | Type | Notes |
|---|---|---|
| message_sequence | integer | DB `message_sequence`. |
| server_seq | integer | Same as `message_sequence`. |
| client_message_id | string \| null | From DB. |
| sender_id | integer | `tam_conversations_user_id`. |
| message_text | string | **Decrypted** plain text. Sender (`is_mine`) sees original; recipient sees remote translation when present, else original. |
| message_type | string | From DB; defaults to `text` if null. |
| attachment_url | string \| null | Signed URL only when `tam_conversations_attachment_url` is populated. **Most attachment messages** store files in `tam_connections_attachments` (joined in `user_conversations_load` §42, not here) — expect **`null`** on replay for typical attachment sends. |
| created_at | string | `tam_conversations_message_date`. |
| is_mine | boolean | `sender_id` equals requesting user. |
| original_message | string \| null | Present when remote decrypted text differs from original (translation path); else **null**. |
| delivery_status | string | **Only when `is_mine` is true** and column `tam_conversations_message_status` exists. Wire values: `sent`, `delivered`, `seen` (SQL `read` maps to `seen`). |

`attachment_url` is built by the local `generateAttachmentUrl` in `tam_connections_get_messages.php`: HMAC-SHA256 over `"{filename}|{exp}"` with `TAM_ATTACHMENT_SECRET`, URL prefix `TAM_BASE_URL + "/api.php?get_attachment=1&file=…&exp=…&sig=…"`. This differs from `functions_ysam.php::generateAttachmentUrl` (uses `hash('sha256', filename)` in the signed payload and `/api.php/v1/get_attachment?`). `html/api.php` attachment handling expects the **hashed-filename** variant and a request path containing `api.php/v1/`; clients must treat URL compatibility as deployment-specific.

## Failure Response

| Case | HTTP | Body |
|---|---|---|
| Missing `room_name` or user id | 200 | `{"status":false,"message":"<missing_fields>","data":[]}` |
| Room not found or read access denied | 200 | `{"status":false,"message":"<missing_fields>","data":[]}` (`tam_connections_assert_can_read_room` failure uses the same constant) |
| DB Throwable | 200 | `{"status":false,"message":"<technical_issue>","data":[]}` |
| Laravel: missing `reference_user_id` | 422 | `{"status":false,"message":"web_user_id_required"}` |
| Laravel: room access denied | 200 | `{"status":false,"message":"missing_fields","data":[]}` |

## Notes

- Message bodies are encrypted at rest; this endpoint decrypts via `safeDecrypt()` before returning `message_text`.
- For initial room open with block UI metadata, use `user_conversations_load` (§42). This endpoint is for gap recovery (`lastSeq`) and older history (`beforeSeq`).

## 37a. Connections Mark Messages Seen

## Endpoint

```text
POST /api.php/v1/connections_mark_messages_seen
```

**Laravel proxy (mobile):** `POST /api/connections_mark_messages_seen` — Passport `auth:api`; actor is token `reference_user_id`. On success returns `{"status":true,"message":"success","data":{"updated":[...]}}` (HTTP 200). After legacy persistence, Laravel publishes advisory WebSocket `message_status` with `status: "seen"` on Redis channel `chat_messages` (not authoritative; SQL is source of truth).

## Purpose

Persists seen state for TAM Connections:

- **Single chat (`room_type` = `S`):** Sets `tam_conversations_message_status = 'read'` for peer messages (not sender's own) matching `client_message_id`; resets `tam_conversation_summary` unread counters and updates `last_seen` for the viewer.
- **Group chat (`room_type` = `G`):** Clears `tam_group_unread.unread_count` for the viewer; per-message SQL status is not updated (WS advisory only for listed `message_ids`).

## Authentication

Bearer required. User id: `$_REQUEST['auth_user_id']` if set, else `$_POST['user_id']`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| room_name | string | Yes | — | Trimmed; empty → failure. |
| message_ids | array \| JSON string | Yes | — | Non-empty list of `client_message_id` values. Legacy accepts JSON string in POST (decoded in PHP). |
| room_type | string | No | inferred | `S` or `G` (case-insensitive). When omitted, inferred from `tam_connections_conversation_room_exists` / `tam_connections_group_room_exists`. |
| user_id | string/int | Conditional | — | Required when JWT does not populate `auth_user_id`. |
| user_lang | string | No | en | Loads YSAM conversation language constants. |

## Success Response

```json
{
  "status": true,
  "message": "success",
  "data": {
    "updated": [
      {
        "client_message_id": "abc-123",
        "sender_id": 42
      }
    ]
  }
}
```

### `data.updated[]` row (`tam_connections_mark_messages_seen`)

| Field | Type | Notes |
|---|---|---|
| client_message_id | string | Message id that was processed. |
| sender_id | integer \| null | Peer sender user id for single-chat SQL updates; **null** for group rooms (advisory list only). |

Empty `updated` when ids were already read, not found, or viewer lacks room access (legacy still returns `status: true` only when handler entered the success branch with valid fields and `tam_connections_assert_can_read_room` passed).

## Failure Response

| Case | HTTP | Body |
|---|---|---|
| Missing user, `room_name`, or `message_ids` | 200 | `{"status":false,"message":"<missing_fields>","data":{"updated":[]}}` |
| Room read access denied | 200 | `{"status":false,"message":"<missing_fields>","data":{"updated":[]}}` |
| Uncaught Throwable | 200 | `{"status":false,"message":"error","data":{"updated":[]}}` |
| Laravel: missing `reference_user_id` | 422 | `{"status":false,"message":"web_user_id_required"}` |
| Laravel: missing `room_name` or `message_ids` | 422 | `{"status":false,"message":"missing_fields"}` |
| Laravel: room access denied | 200 | `{"status":false,"message":"missing_fields","data":[]}` |

## Notes

- Authoritative seen state is SQL (`tam_conversations_message_status` / group unread tables). WebSocket `message_status` / `seen` frames are advisory for sender UI refresh.
- Replay after mark-seen: `get_connection_messages` returns `delivery_status: "seen"` on the sender's own rows when SQL status is `read`.


## 38. Connections Group Subscribe

## Endpoint

```text
POST /api.php/v1/connections_group_subscribe
```

## Purpose

Subscribes a user to a group when access rules pass; may increment `member_count` once per net-new subscription.

## Authentication

Bearer required. Script does not read `auth_user_id`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_id | integer | Yes | — | Cast `(int)`; falsy with `group_id` yields `error_subscribing` constant message. |
| group_id | integer | Yes | — | Cast `(int)`. |
| user_lang | string | No | en | Loads `html/language_config/ysam_conversation/common/config_ysam_{user_lang}.php`. |

## Success Response

```json
{
"status": true,
  "message": "<success_subscribing constant>",
"data": []
}
```

## Failure Response

| Case | Body |
|---|---|
| Access SQL returns 0 rows | `{"status":false,"message":"<not_allowed_to_join_group>","data":[]}` |
| Throwable | `{"status":false,"message":"<high_traffic|technical_issue|exception text>","data":[]}` |

## Notes

- Uses `begin_transaction` / `commit_transaction` / `rollback_transaction` with `FOR UPDATE` on the group row.


## 39. Connections Group Unsubscribe

## Endpoint

```text
POST /api.php/v1/connections_group_unsubscribe
```

## Purpose

Marks subscription unsubscribed, decrements member count when still subscribed, deletes `tam_group_unread` row.

## Authentication

Bearer required.

## Request Parameters

Same as subscribe: `user_id`, `group_id`, optional `user_lang` for error constants.

## Success Response

```json
{
"status": true,
  "message": "<success_unsubscribing constant>",
"data": []
}
```

## Failure Response

Non-POST, missing ids, missing group row, or exception paths return `status` **false** with localized `invalid_request`, `error_unsubscribing`, or `technical_issue` / exception message.

## Notes

- Successful JSON is emitted **once** from the script’s `finally` block (`$output` array).


## 40. Connections Register Consent

## Endpoint

```text
POST /api.php/v1/register_connections_consent
```

## Purpose

Inserts (or no-ops on duplicate) a `tam_connections_consent` row with the canonical consent string for the user’s language.

## Authentication

Bearer required.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_id | integer | Conditional | 0 | Copied from `user_reference_id` when `user_id` empty. |
| user_reference_id | mixed | Conditional | — | Alias for `user_id`. |
| user_lang | string | No | en | Loads `common/config_ysam_{user_lang}.php` only. |

## Success Response

```json
{
"status": true,
  "code": "1",
  "text": "<consent_registered constant>",
  "message": "<consent_registered constant>",
"data": []
}
```

## Failure Response

Invalid user:

```json
{
  "status": false,
  "code": "0",
  "text": "<generic_error constant>",
  "message": "<generic_error constant>",
  "data": []
}
```

Throwable wrapper: `status` **false**, `message` localized, `data` **[]** (no `code` / `text` on that path).

## Notes

- Consent string is `implode('~~', …)` of the localized title, introductions, lines 1–7, and `consent_final` (see script).


## 41. Connections Send Message

## Endpoint

```text
POST /api.php/v1/connections_send_message
```

## Purpose

Classifies text via `_shared_tam_classify.php` / `classifyUserText`, enforces plan and daily send limits, optionally handles multipart attachment storage, encrypts message bodies with global `$ncrypt`, updates summary tables, and returns send metadata.

## Authentication

Bearer required. Sender id: `$_REQUEST['auth_user_id']` if set, else `$_POST['sender_reference_id']` (empty string allowed and fails later validation).

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_lang | string | No | en | First gated by `app_supported_languages`; later re-read escaped from POST. |
| client_message_id | string | null | null | Escaped; stored on insert; duplicate DB error **1062** triggers idempotent success branch when the duplicate is on this id. |
| sender_reference_id | string/int | Conditional | "" | Used only when `auth_user_id` absent. |
| recipient_reference_id | integer | Conditional | 0 | Required when `room_type` is **S** (after trim/empty checks). |
| message | string | Conditional | "" | Trimmed; may be empty only when `attachment` file upload present. |
| room_name | string | Yes | — | Escaped. |
| room_type | string | No | S | Uppercased to `S` or `G`. |
| reply_to_message_id | integer | null | null | `0` normalized to `null`. |
| attachment | file | Conditional | — | `$_FILES['attachment']`; requires `UPLOAD_ERR_OK`. |

Parameters **not** read by this PHP file include `timestamp_browser`, `reply_to_content`, `reply_to_sender` (any Laravel-only aliases are not upstream fields here).

## Supported Attachment MIME Types

Validated with `finfo_file` MIME (not extension). Max size `CONNECTIONS_MAX_ATTACHMENT_SIZE` bytes; oversize uses localized `invalid_attachment` with `(XXXX)` replaced by `CONNECTIONS_MAX_ATTACHMENT_SIZE_MB`. Image types are validated with `getimagesize`. Stored under `CONNECTIONS_ATTACHMENT_FOLDER` with UUID filename; JPEG/PNG/WebP receive JPEG thumbnail in `CONNECTIONS_ATTACHMENT_FOLDER_THUMB` as `thumb_{uuid}.jpg`.

| MIME | Stored `message_type` |
|---|---|
| application/pdf | file |
| image/jpeg, image/png, image/webp | image |
| audio/mpeg, audio/wav, audio/ogg, audio/webm | audio |

## Success Response

**Single chat (`room_type` **S**), normal insert:**

```json
{
"status": true,
"message": "success",
"data": {
    "text": "<plaintext sender message>",
    "message_sequence": 1,
    "client_message_id": null,
    "attachment_url": null,
    "attachment_thumb_url": null,
    "detected_language": "en",
    "special_message": "",
    "sender_date_display": "Mon, 01 Jan 2026 (03:45 pm)",
    "recipient_text": "<translated or original>",
    "recipient_date_display": "Mon, 01 Jan 2026 (03:45 pm)"
}
}
```

`special_message` is non-empty when classifier returns `link` = **Y** (localized `link-provided` stripped of brackets).

**Group chat (`room_type` **G**), success:** same envelope except `recipient_text` duplicates plaintext `message` and both date displays use sender timezone formatting path in PHP.

**Idempotent duplicate (`client_message_id` unique violation):**

```json
{
  "status": true,
  "message": "duplicate",
  "data": {
    "message_sequence": 0
  }
}
```

(`message_sequence` read from existing row, may be null if lookup fails.)

### `data` fields — success (`message` **success**, single or group)

| Field | Type | S | G | Notes |
|---|---|:---:|:---:|---|
| text | string | ✓ | ✓ | Plain sender message text (not DB ciphertext in response). |
| message_sequence | integer | ✓ | ✓ | Allocated sequence. |
| client_message_id | string \| null | ✓ | ✓ | Echo of request id. |
| attachment_url | string \| null | ✓ | ✓ | Signed URL from `generateAttachmentUrl` in send script, or null. |
| attachment_thumb_url | string \| null | ✓ | ✓ | Thumbnail URL for images, else null. |
| detected_language | string | ✓ | ✓ | Classifier language code. |
| special_message | string | ✓ | ✓ | Non-empty when classifier link flag **Y**. |
| sender_date_display | string | ✓ | ✓ | Formatted in sender’s timezone. |
| recipient_text | string | ✓ | ✓ | Single: translated text for recipient; group: same as `text`. |
| recipient_date_display | string | ✓ | ✓ | Single: recipient-local formatted string; group: same as sender side display. |

### `data` fields — idempotent duplicate (`message` **duplicate**)

| Field | Type | Notes |
|---|---|---|
| message_sequence | integer \| null | Loaded from existing row by `client_message_id`. |

## Failure Response

Envelope is always `{"status":false,"message":"<string>","data":[]}` except Throwable catch which may use localized high-traffic / technical strings from `session_error` includes.

Non-exhaustive `message` values / sources:

| Message | When |
|---|---|
| `processing_error` constant | Missing sender/recipient/room/room_type; missing user rows; classifier category code `"0"`; DB errors in non-duplicate paths. |
| `not_subscribed` | Plan summary error. |
| `tam_connections_not_available` / `tam_connections_trial_message` | Plan flag `tam_connections_enabled` != `Y`. |
| `exceeded_usage` | Daily count > `ysam_messages_limit_per_day`. |
| `classification_failed` | `classifyUserText` === false. |
| Category error string | From classifier payload when code `"0"`. |
| `not_subscribed_to_group` | Group sender not subscribed. |
| `not_allowed_to_send_message` | Inactive group or access SQL returns 0 rows. |
| `Empty message` | Exception before constants (empty text and no attachment). |
| `Invalid_upload` / localized attachment errors | Upload validation failures. |
| `File upload failed` | `move_uploaded_file` failure. |
| Localized `invalid_image` | Thumbnail source decode failure. |

## Notes

- Single-chat inserts use `UPDATE tam_connections_conversation_master SET last_sequence = LAST_INSERT_ID(last_sequence+1)` to allocate `message_sequence`.
- Message bodies stored encrypted (`tam_conversations_message_original`, `tam_conversations_message_remote`).

## 42. User Load Conversations

## Endpoint

```text
POST /api.php/v1/user_conversations_load
```

## Purpose

Loads up to **50** messages for a room via `app_tam_connections_messages` with `mode` **`initial`** and cursor **0**, plus conversation state and (for single chat) block UI strings.

## Authentication

Bearer required. Script does not read `auth_user_id`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_reference_id | string | Yes | — | Escaped; empty throws exception `"User Id or Connection type is not specified"`. |
| connection_type | string | Yes | — | Must be **`S`** or **`G`** (other values throw same exception as empty user). |
| room_name | string | Yes | — | Escaped; empty throws `"Room is empty"`. |
| user_lang | string | No | en | Mobile allow-list normalization applies. |

## Success Response

On success `app_tam_connections_messages` returns a PHP array echoed as JSON:

```json
{
"status": true,
  "message": "success",
"data": {
    "room_name": "",
    "conversation_active": true,
    "blocked_by_current_user": false,
    "blocked_by_other_user": false,
    "messages": [],
    "current_user_name": "",
    "user_timezone": "",
    "user_gender": "",
    "show_block_user_flag": "Y",
    "block_user_button_text": "",
    "block_user_confirmation_text": ""
}
}
```

### `data` room fields (`app_tam_connections_messages`)

| Field | Type | S | G | Notes |
|---|---|:---:|:---:|---|
| room_name | string | ✓ | ✓ | Requested room name. |
| conversation_active | boolean | ✓ | ✓ | `tam_conversations_active === 'Y'` for single; groups always **true** in code path. |
| blocked_by_current_user | boolean | ✓ | ✓ | See dashboard logic for single; groups false. |
| blocked_by_other_user | boolean | ✓ | ✓ | Inactive conversation and inactive user ≠ current user (single); groups false. |
| messages | array | ✓ | ✓ | Up to 50 rows, ascending order after query (reversed from DESC). |
| current_user_name | string | ✓ | ✓ | Decrypted YSAM display name. |
| user_timezone | string | ✓ | ✓ | From `reservation_users.user_timezone` (decrypted). |
| user_gender | string | ✓ | ✓ | Decrypted `sex` field. |
| show_block_user_flag | string | ✓ | — | **`"Y"`** or **`"N"`** only when `connection_type === 'S'` and conversation active; always **`"N"`** for **G** in PHP return. |
| block_user_button_text | string | ✓ | — | Localized constant when block UI shown; else **""**. |
| block_user_confirmation_text | string | ✓ | — | Localized constant when block UI shown; else **""**. |

### `data.messages[]` row (`format_message_payload`)

**Single (`connection_type` = `S`):**

| Field | Type | Notes |
|---|---|---|
| message_sequence | integer | |
| server_seq | integer | Same as `message_sequence`. |
| client_message_id | string \| null | From DB. |
| sender_id | integer | |
| message_text | string | Decrypted: sender sees original, recipient sees remote translated text. |
| message_type | string | Default `text`. |
| attachment_url | string \| null | From `functions_ysam::generateAttachmentUrl` after decrypting stored filename. |
| attachment_thumb_url | string \| null | Thumbnail URL or null. |
| created_at | string | Message timestamp. |
| is_mine | boolean | |
| translated | boolean | True when remote ciphertext differs from original ciphertext. |
| original_message | string \| null | Plain original when `translated` is true; else **null**. |
| delivery_status | string | **Only when `is_mine` is true** and `tam_conversations_message_status` column exists. Values: `sent`, `delivered`, `seen`. |
| reply_to | object \| null | See below. |

**Group (`connection_type` = `G`):** same keys except **`client_message_id`** is always **null** in PHP; **`translated`**, **`original_message`**, **`delivery_status`** are **omitted** (single-only return shape); `message_text` is decrypted group message body.

### `reply_to` object (`build_reply_block`, when non-null)

| Field | Type | Notes |
|---|---|---|
| messageId | integer | Parent’s `reply_sequence` (message sequence of quoted message). |
| content | string | HTML-escaped; placeholder HTML if missing. |
| sender | integer | `reply_sender` user id. |
| sender_name | string | HTML-escaped decrypted name or **`User`**. |
| is_available | boolean | True when `reply_sequence` non-empty. |

## Failure Response

Exception paths from the wrapper:

```json
{
  "status": false,
  "message": "<exception text or technical_issue constant>",
  "data": []
}
```

Helper failure (`status` false from `app_tam_connections_messages`):

```json
{
  "status": false,
  "message": "<error_starting_conversation or other helper message>",
  "data": []
}
```

## Notes

- Pagination `mode` / `cursor` are **not** exposed as POST inputs in `tam_user_load_conversations.php`; only the `initial` / `0` path is used. Older history uses `get_connection_messages` with `beforeSeq` / `lastSeq`.

## 42a. ThOT Async — Replay Messages (Stage 2)

Authoritative SQL history for Therapy Over Text (ThOT/TOT). Does **not** replace `POST async-chat` send; use replay for reads after send, FCM, or reconnect.

- **Endpoint:** `GET /api/thot/conversations/{sessionId}/messages` (alias: `GET /api/async/replay/{sessionId}/messages`)
- **Method:** GET
- **Auth:** Laravel Passport (`Authorization: Bearer`)
- **Feature flag:** `THOT_REPLAY_API_ENABLED` (server `config/async.php`; mobile runtime-config)

**Query parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| after_sequence | integer | No | Keyset cursor — return rows with `message_sequence` greater than this value. |
| after_uuid | uuid | No | Tie-breaker when `after_sequence` is equal. |
| limit | integer | No | Page size (default 50, max 100). |
| order | string | No | `asc` (default) or `desc`. |

**Visibility rules:**

| Viewer | Counsellor messages (`status=2`) |
|---|---|
| User (session owner) | Only when `released_to_user=true`, `delivery_state=released`, and `available_at <= now()`. |
| Counsellor (assigned) | All messages in session. |

**Success (200):**

```json
{
  "status": true,
  "data": {
    "session_id": 12345,
    "viewer_role": "user",
    "messages": [
      {
        "product": "thot_async",
        "message_uuid": "…",
        "message_sequence": 1,
        "sender_role": "user",
        "display_body": "…",
        "translation_status": "completed",
        "delivery_state": "released",
        "available_at": "2026-05-23T12:00:00+00:00",
        "legacy": { "status": "1", "msgType": "0" }
      }
    ],
    "cursor": {
      "oldest_sequence": 1,
      "newest_sequence": 10,
      "has_more": false,
      "next_after_sequence": null
    }
  }
}
```

**Related write endpoints (unchanged):**

| Endpoint | Purpose |
|---|---|
| `POST /api/counselor-async-user` | Open/resume ThOT session |
| `POST /api/async-chat` | User send (response still returns legacy list when replay disabled on client) |
| `POST /api/tot-user-audio-to-text` | Voice note |

**Counsellor web (session auth):** `GET /tot-chat/replay/{session_id}` — same JSON shape; gated by `THOT_COUNSELLOR_REPLAY_JSON_ENABLED`.

## 42b. ThOT Async — WebSocket notification nudges (Stage 3)

Notification-only private channels (Soketi/Pusher). **No message bodies on the wire** — clients debounce and call replay (§42a).

**Channels:** `private-async.session.{sessionId}`, `private-async.counsellor.{userId}`, `private-async.user.{userId}`

**Events (broadcast as):**

| Event | When | Client action |
|---|---|---|
| `AsyncMessageAvailable` | User message translated; counsellor message released | `GET` replay |
| `AsyncConversationUpdated` | Session status / assignment change | Refresh inbox / replay |
| `AsyncUnreadCountChanged` | Unread counters shift | Badge update |

**Envelope:**

```json
{
  "contract_version": "1.0",
  "product": "thot_async",
  "event": "AsyncMessageAvailable",
  "event_id": "uuid",
  "emitted_at": "ISO8601",
  "session_id": 12345,
  "payload": {
    "message_uuid": "…",
    "sender_role": "user",
    "released_at": null
  }
}
```

**Flags:** `THOT_WS_ENABLED`, `THOT_WS_NUDGES_ENABLED` (server + mobile runtime-config). ThOT RTDB nudges removed in Stage 8 — WS only.

## 42c. ThOT Async — Delayed delivery (Stage 4)

Counsellor messages are held until `available_at` (`THOT_RELEASE_DELAY_SECONDS`, default **900**). User FCM and WS nudges fire on **release**, not on counsellor send.

**Release:** `php artisan thot:release-pending-messages` (scheduled every minute when `THOT_RELEASE_SCHEDULER_ENABLED=true`).

**POST `async-chat` (user send)** when `THOT_ASYNC_CHAT_MINIMAL_RESPONSE=true` (defaults with `THOT_REPLAY_API_ENABLED`):

```json
{
  "status": true,
  "use_replay": true,
  "response": {
    "mode": "ack",
    "sent": {
      "message_uuid": "…",
      "message_sequence": 12,
      "status": "1",
      "created": true
    },
    "messages": []
  }
}
```

Clients must load history via §42a replay. Held counsellor rows are excluded from user replay and legacy list filters (`released_to_user`, `delivery_state=released`, `available_at <= now()`).

**Rollback:** `THOT_RELEASE_DELAY_SECONDS=0` (instant release after translate) or `THOT_ASYNC_CHAT_MINIMAL_RESPONSE=false` to restore full `response` array.

## 42d. ThOT Async — Mobile FCM + replay + WS (Stage 5)

Flutter loads authoritative history from §42a on session start, send ack, FCM (`async_counsellor_message`), and WS nudges (`AsyncMessageAvailable`, `AsyncConversationUpdated`).

**Runtime-config flags:** `THOT_REPLAY_API_ENABLED`, `THOT_WS_ENABLED`, `THOT_WS_NUDGES_ENABLED`, `THOT_ASYNC_CHAT_MINIMAL_RESPONSE`, `THOT_PUSHER_APP_KEY`, `THOT_WS_HOST`, `THOT_WS_PORT`, `THOT_WS_SCHEME` (falls back to `CHAT_FB15_*` when ThOT keys empty).

**Broadcast auth:** `POST /broadcasting/auth` with Passport bearer — channel `private-async.session.{sessionId}`.

**Counsellor web:** WS nudge triggers immediate JSON replay (no RTDB listener, no `counsellorMsgReceivedTime` delay).

## 42e. ThOT Async — Counsellor inbox / dashboards (Stage 6–8)

When WS flags are on:

- **No** RTDB paths (Stage 8 removed all `tam_chat_messages/async/*` code).
- Inbox/list surfaces refresh via `private-async.counsellor.{counsellorId}` or `private-async.ops`.
- **async-chats-list** reloads via `GET /tot-chats-list` on WS nudge.
- Dashboard badges use existing REST endpoints on nudge.

**Rollback:** Restore pre–Stage 8 blade/controller RTDB from git tag if WS alone is insufficient.

## 42f. ThOT Async — Mobile replay-only (Stage 7)

When `THOT_MOBILE_REPLAY_ONLY_ENABLED=true` (requires `ASYNC_PIPELINE_ENABLED` + `THOT_REPLAY_API_ENABLED`):

- **`POST async-chat`** returns ack only (`use_replay: true`, `response.mode=ack`, `response.sent.message_uuid`) — **no message array**.
- Flutter **never** assigns `response` list to UI; history from §42a replay only.
- Voice-note send path still uses `POST async-chat` after upload; UI refreshes via replay.

**Rollback:** `THOT_MOBILE_REPLAY_ONLY_ENABLED=false` — client may use legacy full `response` array when `THOT_ASYNC_CHAT_MINIMAL_RESPONSE=false`.

## 42g. ThOT Async — Zero RTDB (Stage 8)

ThOT messaging no longer reads or writes Firebase RTDB.

| Surface | Authority |
|---------|-----------|
| Message bodies | SQL (`tam_async_chat_messages`) + §42a replay |
| Realtime nudges | WS (`AsyncMessageAvailable`, `AsyncConversationUpdated`, `AsyncUnreadCountChanged`) |
| Counsellor drafts | Browser `localStorage` via `thot-counsellor-draft.js` |
| Mobile | Replay-only (§42f); no RTDB imports |

**Removed:** `ThotRtdbNudgeService`, `THOT_RTDB_NUDGES_ENABLED`, all `tam_chat_messages/async/*` and `tam_thot_draft_messages` paths in ThOT controllers and blades.

**Rollback:** Restore Stage 8 git tag; re-enable RTDB service and blade Firebase modules (WS + replay continue to work independently).

## 42h. ThOT Async — Production hardening

**Polling removed:** Counsellor/ops chat-boat use JSON replay bootstrap + WS nudge only. `user-msg-get` HTML poll retired.

**Release guards:** Atomic SQL claim + `fcm_dispatched_at` / `ws_release_nudge_dispatched_at` prevent duplicate FCM/WS under concurrent workers.

**Translation chaos (non-prod):** `THOT_CHAOS_TRANSLATION_TIMEOUT`, `THOT_CHAOS_TRANSLATION_MALFORMED`, `THOT_CHAOS_TRANSLATION_FAILURE_RATE` — no-op when `APP_ENV=production`.

**Health:** `GET /admin/thot/health` — held/overdue release counts and translation failure backlog.

## 43. Core Translate

Translates text automatically using an LLM. Detects the source language and optionally updates asynchronous therapy chat (ThOT) message rows.

- **Endpoint:** /api.php/v1/translate

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| text | String | Yes | Text to translate. URL-decoded and consecutive dots collapsed. |
| target | String | No | Target language code (default `"en"`). |
| type | String | No | `"live"` (default) or `"tot"` (Therapy Over Text). Controls whether `tam_async_chat_messages` is updated. |
| message_id | String | No | `tam_async_chat_messages.id`. When provided, the translated text is persisted back to DB. |
| session_id | String | No | ThOT session ID. Auto-resolved from `message_id` when `type != "live"`. |
| from_counsellor | String | No | `"1"` if counsellor message (updates `translate_message` column), `"0"` if user (updates `message` column). |
| user_lang | String | No | Source language hint for detection (default `"en"`). |
| user_gender | String | No | `"M"` or `"F"` — mapped to `"male"` / `"female"` for LLM context. |
| testing | String | No | `"1"` includes extra `db_message` field in response. |
| verbosity | String | No | LLM verbosity level (default `"low"`). |
| reasoning | String | No | LLM reasoning level (default `"minimal"`). |
| thot_attachment_only | String | No | `"1"` to refresh word_count only for ThOT attachment rows (no translation). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1` success, `0` failure. Always an **int**. |
| text | string | Translated text (or original if source == target). |
| detected_language | string | Detected source language code. |
| detected_language_full | string | Full language name from `SUPPORTED_LANGUAGES`. |
| db_message | string | Only present when `testing:"1"`. Describes DB update status. |

**Runtime (`html/api/v1/translate.php`)**

1. Loads `llm_helper.php`, `llm_prompts.php`, `llm_provider.php`, `llm_executor.php` for the LLM abstraction layer.
2. **ThOT attachment fast path:** When `thot_attachment_only:"1"` and `type:"tot"` and `from_counsellor:"0"`, only calls `async_chat_refresh_user_word_count` (no LLM call). Returns `code:1` with empty text.
3. **ThOT empty text fast path:** When `type:"tot"` and text is empty with a `message_id`, refreshes word_count only.
4. **Language detection:** `identifyLanguage()` (from `llm_helper.php`). Urdu detected as Hindi when `user_lang` is `"hi"`.
5. **Translation logging:** Every request is logged to `app_translation_log` table before translation.
6. **Same-language path:** When `sourceLanguage === targetLanguage`, no LLM call. For ThOT messages, the DB row is updated with detected language metadata. Attachment-only rows (`msgType=1` or `message='attachment'`) are preserved — only `detected_lang` / `detected_lang_name` are updated.
7. **Different-language path:** Calls `llm_execute('translate', ...)` via `handleTranslation()`. For ThOT, updates `message` (user) or `translate_message` (counsellor) column. For `type:"live"`, no DB write to `tam_async_chat_messages`.
8. **Word count refresh:** After any ThOT user message DB update, `async_chat_refresh_user_word_count` recalculates `word_count` using Unicode-aware tokenization (CJK characters counted as ¼ word each, ceil'd).
9. Error is logged to DB via `logtoDB`.

**ThOT DB update paths:** When `type=tot` and `message_id` is set, successful DB updates always return `{code:1,text,detected_language,detected_language_full}`; `db_message` is added only when `testing:"1"`.

**Catch envelope (Throwable):**

```json
{ "code": 0, "text": "Translation Error : <exception message>" }
```

## 44. Upload Audio

Uploads audio files to the server. Supports standard multipart file uploads or Base64 encoded recorded strings. Returns the server-side path for use with §27/§31 transcription endpoints.

- **Endpoint:** /api.php/v1/upload_audio

- **Method:** POST

- **Content-Type:** multipart/form-data (file mode) or application/x-www-form-urlencoded (base64 mode)

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| type | String | Yes | `"R"` for base64 recorded audio, `"F"` for file upload. |
| recordedAudio | String | Cond. | Base64-encoded audio data (required when `type:"R"`). Decoded and saved as `.wav`. |
| audioFile | File | Cond. | Binary file upload (required when `type:"F"`). Saved with original extension, prefixed with `uniqid()`. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1` success, `0` failure. Always an **int**. |
| message | string | Human-readable status message. |
| file | string | Absolute server path to the uploaded file. Empty string `""` on failure. |

**Laravel proxy (mobile):** `POST /api/upload_audio` → `YsamApiController::ysamUserAudioUpload` — multipart `file` field; success `{ "status": true, "url": "http://…", "file_path": "<server path>" }`; failure `{ "status": false, "url": "" }` (HTTP 400 on exception). **Different** from legacy `{code,message,file}` below.

**Runtime (`html/api/v1/upload_audio.php`)**

1. Loads `private/config.php` via `require_once(__DIR__ . '/../../../private/config.php')` (no `main.php` or `functions.php`). Upload directory is `global_audio_voice_note_dir` constant.
2. Creates the upload directory recursively if it doesn't exist (`mkdir(..., 0777, true)`).
3. **Base64 mode (`type:"R"`):** `base64_decode` → `file_put_contents` with filename `audio_<uniqid>.wav`.
4. **File mode (`type:"F"`):** `move_uploaded_file` with filename `<uniqid>_<original_name>`.
5. No database connection or session management (lightweight upload-only handler).
6. Does **not** validate audio format — format validation happens at the transcription endpoints (§27/§31).

**Failure variants:**

| Condition | `code` | `message` |
|---|---|---|
| `type:"R"` but `recordedAudio` missing | `0` | `"Invalid request method."` |
| `type:"F"` but `audioFile` missing | `0` | `"No audio file to upload."` |
| `file_put_contents` / `move_uploaded_file` fails | `0` | `"There was some error uploading the file."` / `"File upload failed."` |
| Exception | `0` | `"There was some error uploading the file."` |

## 45. YSAM Add/Update Article

Creates a new post or edits an existing one in the "Your Story and Mine" (YSAM) community feed.

- **Endpoint:** /api.php/v1/ysam_add_update_article

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | Author's User ID (reservation users). Escaped via `mysqli_real_escape_string`. |
| ysam_update | String | No | `"Y"` to update an existing article. Any other value or empty → new article (`"add"`). |
| ysam_id | String | Cond. | Post ID (required if updating). |
| ysam_title | String | Yes | Title of the post. Validated non-empty. |
| article_body | String | Yes | Content body. Required for non-podcast categories (`ysam_category_id != 1006`). |
| ysam_category_id | String | Yes | Category / story type ID. `1006` is the podcast category (requires `ysam_narration`). |
| ysam_hashtags | String | No | Comma-separated hashtags. Cleaned, deduped, title-cased, prefixed with `#`. |
| ysam_user_name | String | Yes | Author's display name / pseudonym. Validated for uniqueness and appropriateness. |
| ysam_narration | String | Cond. | Audio narration file path (required when `ysam_category_id` is `1006`). |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success, `false` on failure. |
| message | string | Localized result message. |
| data | object/array | Contains `ysam_id` on success. Empty array `[]` on failure. |

**Runtime (`html/api/v1/ysam_add_update_article.php`)**

1. Delegates to `submit_ysam(...)` in `functions.php`. The helper returns a **JSON string**; the handler `json_decode`s it.
2. All POST parameters are escaped with `mysqli_real_escape_string` before passing to the helper.
3. The `ysam_update` parameter is mapped: `"Y"` → `"update"`, anything else → `"add"`.
4. The helper validates: title non-empty, body non-empty (unless podcast), username unique (`validate_unique_user_name_ysam`), username appropriate (`appropriate_user_name`).
5. Content is checked via `check_ysam_content()` (LLM-based classification) before submission.
6. Includes `functions_ysam.php` for YSAM-specific helpers and `ffmpeg` autoloader for narration processing.
7. **Catch output format inconsistency:** The `catch` block sets `$output` to `json_encode(...)` (a JSON **string**), while the `finally` echoes `is_array($output) ? json_encode($output) : $output`. So on error, the string is echoed directly.

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "<error message>", "data": [] }
```

## 46. YSAM Connect to User

Sends a direct connection request (to initiate a 1-on-1 chat) to the author of a "Your Story and Mine" (YSAM) post.

- **Endpoint:** /api.php/v1/ysam_connect_to_user

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | The ID of the user requesting the connection. Escaped via `mysqli_real_escape_string`. |
| author_reference_id | String | Yes | The ID of the YSAM author to connect with. |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success, `false` on failure. |
| message | string | Localized message. On success: `connection_approved` constant with author's display name substituted. |
| data | array | Empty array `[]`. |

**Runtime (`html/api/v1/ysam_connect_to_user.php`)**

1. Delegates to `update_connect_to_user($connection, $user_id, $author_id, $user_lang, true)` in `functions.php`.
2. The helper returns a JSON string; the handler `json_decode`s it.
3. Self-connection (`user_id === author_id`) returns `code:"0"` with `connection_self_error`.
4. Checks user's current plan for `tam_connections_enabled == "Y"` via `get_current_plan_summary`. If not enabled → `status:false` with `upgrade_subscription`.
5. On success: creates/updates a `tam_connections_conversation_master` room (`room_<min>_<max>`) via `INSERT ... ON DUPLICATE KEY UPDATE`, then calls `update_user_connection` within a transaction.
6. The `$output` echoing uses `is_array($output) ? json_encode($output) : $output` pattern.

**Failure variants:**

| Condition | `status` | `message` |
|---|---|---|
| Self-connection | N/A (inner `code:"0"`) | `connection_self_error` constant |
| Plan doesn't allow connections | `false` | `upgrade_subscription` constant |
| Exception | `false` | Localized error constant or exception message |

## 47. YSAM Follow User

Toggles the "follow" status, allowing a user to subscribe to or unsubscribe from future posts by a specific YSAM author.

- **Endpoint:** /api.php/v1/ysam_follow_user

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | The ID of the user initiating the follow/unfollow. Escaped via `mysqli_real_escape_string`. |
| user_to_follow | String | Yes | The ID of the author being followed/unfollowed. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true`. |
| message | string | `follow_user_success_label` or `unfollow_user_success_label` constant depending on the new toggle state. |
| data | array | Empty array `[]`. |

**Runtime (`html/api/v1/ysam_follow_user.php`)**

1. Delegates to `update_user_following($connection, $user_id, $user_to_follow, true)` in `functions_ysam.php`.
2. Validates both users exist in `reservation_users` + `reservation_users_p_details` (expects exactly 2 rows). Throws `"Invalid User"` otherwise.
3. **Toggle logic:** Queries `reservation_user_ysam_follow` for existing row. If found, flips `reservation_user_ysam_following_active` between `"Y"` and `"N"`. If not found, inserts new row with `"Y"`.
4. Includes `PHPMailer` autoload (no email send on the toggle path in current code).
5. Success `message` uses `$new_follow` in `update_user_following()` — `follow_user_success_label` vs `unfollow_user_success_label`.
6. The handler `catch` block sets `$output` to a `json_encode`'d string (helper returns JSON string on success).

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "<error message>", "data": [] }
```

## 48. YSAM Get All Categories

Retrieves a complete list of categories available for YSAM posts, localized into the requested language. Used for filtering feeds or populating dropdowns.

- **Endpoint:** /api.php/v1/ysam_get_all_categories

- **Method:** GET / POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_lang | String | No | Language code (default `"en"`). Determines which `ysam_collection_type` rows are returned (`language_code` column). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success, `false` if no categories found. |
| message | string | `"success"` or `processing_error` constant. |
| data | array | Array of category objects. Empty array `[]` on failure. |

**`data[]` object:**

| Field | Type | Notes |
|---|---|---|
| category_id | mixed | `ysam_collection_type_id` from DB. |
| category_name | string | `ysam_collection_type_name` from DB, localized per `user_lang`. |
| supports_audio | boolean | `true` when `category_id == 1006` (podcast category). |
| content_mode | string | `"audio"` for category `1006`, else `"text"`. |

**Runtime (`html/api/v1/ysam_get_all_categories.php`)**

1. Delegates to `get_all_categories($connection, $user_lang, true)` in `functions.php`.
2. The helper queries `ysam_collection_type WHERE language_code = $user_lang`.
3. When `redirected_from_app` is `true` (always for API calls), wraps the result in `{ "status", "message", "data" }`. Without this flag, returns a bare JSON array.
4. The helper returns a **JSON string**; the handler echoes it via the `is_array($output) ? json_encode($output) : $output` pattern (it will be a string).

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "<error message>", "data": [] }
```

## 49. YSAM Initialize Form

Aggregates metadata required to render the YSAM post creation form on the frontend, including the user's current pseudonym, localized labels, available categories, and podcast recording constraints.

- **Endpoint:** /api.php/v1/ysam_initialize_form

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | User ID (reservation users). Escaped via `mysqli_real_escape_string`. |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success, `false` if user not found. |
| message | string | `"success"` or `error_do_not_proceed` constant. |
| data | object | Form metadata (see below). Empty array `[]` on failure. |

**`data` object fields (when `call_from_app = true`):**

| Field | Type | Notes |
|---|---|---|
| ysam_form_heading | string | Localized form heading constant. |
| ask_for_ysam_user_name | string | `"Y"` if user has no YSAM pseudonym yet, `"N"` if they do. |
| current_user_ysam_name | string | Decrypted current YSAM pseudonym (empty if none). |
| ysam_user_name_message_label | string | Localized prompt: either "name exists" or "name required" variant. |
| story_title_label | string | Localized label. |
| story_hashtag_label | string | Localized label. |
| story_category_label | string | Localized label. |
| categories | array | Result of `get_all_types()` — `[{ "category_id", "category_name", "supports_audio", "content_mode" }]`. |
| story_textarea_placeholder | string | Localized placeholder text. |
| podcast_title | string | Localized podcast section heading. |
| podcast_label | string | Localized recording instructions (with duration limits substituted). |
| submission_confirm_button | string | Localized button text. |
| submission_cancel_button | string | Localized button text. |
| minimum_recording_length | mixed | `ysam_narration_min_duration` constant (seconds). |
| short_recording_message | string | Localized message for recordings shorter than minimum. |
| maximum_recording_length | mixed | `ysam_narration_duration` constant (seconds). |
| max_recording_exceeded_message | string | Localized message for recordings longer than maximum. |

**Runtime (`html/api/v1/ysam_initialize_form.php`)**

1. Delegates to `initialize_ysam_form($connection, $user_id, $user_lang, true)` in `functions_ysam.php`.
2. Queries `reservation_users` + `reservation_users_p_details` for the user's `user_ysam_name`. If user not found → `status:false`.
3. Uses `get_all_types()` (not `get_all_categories()`) to fetch categories. These are from the `ysam_collection_type` table filtered by lang.
4. Numbers in recording duration labels are localized to the user's script via `replace_numbers_with_local()`.
5. The helper returns a JSON string; handler echoes via the `is_array ? json_encode : echo` pattern.

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "<error message>", "data": [] }
```

## 50. YSAM Report User Post

Flags a YSAM post for review by the moderation team or **withdraws** an existing report (toggle behaviour).

- **Endpoint:** /api.php/v1/ysam_report_user_post

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| user_reference_id | String | Yes | ID of the user reporting the post. Escaped via `mysqli_real_escape_string`. |
| ysam_id | String | Yes | ID of the post being reported. |
| reason | String | Yes | Report reason text. Max 150 chars. Validated for HTML and SQL injection patterns. |
| user_lang | String | No | Language code (default `"en"`). |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true` on success (report recorded or withdrawn). |
| message | string | Localized message: `post_objection_recorded` or `post_objection_withdrawn`. |
| data | object | `{ "post_reported_by_user": "Y" }` (reported) or `{ "post_reported_by_user": "N" }` (withdrawn). |

**Runtime (`html/api/v1/ysam_report_user_post.php`)**

1. Delegates to `report_post($connection, $ysam_id, $previous_reported, $reason, $user_id, $user_lang, true)` in `functions_ysam.php`.
2. **Bug note:** The handler passes `$previous_reported` which is **undefined** in the script scope. The helper uses `$prev_reported` parameter but the actual toggle logic is determined internally by querying `ysam_reported_posts` — so the undefined variable has no functional impact.
3. **Validation:** Rejects HTML characters (`containsHtmlChars`), SQL injection patterns (`containsSqlInjection`), and reasons > 150 characters. Validation failures return `code:"2"` in web mode, `status:false` in app mode.
4. **Toggle logic (transactional):**
   - If user already reported this post → **withdraws** the report (DELETE + decrement `report_count` with `GREATEST(..., 0)`).
   - If not previously reported → **inserts** new report (INSERT + increment `report_count`).
5. **Auto-moderation:** After inserting a report, checks if `report_count >= MAX_REPORTED_FOR_SOFT_DELETE`. If so, sets `ysam_published_status = 0` (soft-deletes the post).
6. If `user_id` is empty (no session or POST value), returns `status:false` with `processing_error`.

**Failure variants:**

| Condition | `status` | `message` |
|---|---|---|
| HTML chars in reason | `false` | `invalid_input_special_characters` |
| SQL injection in reason | `false` | `invalid_input_patterns` |
| Reason > 150 chars | `false` | `invalid_input_length` |
| User empty | `false` | `processing_error` |
| Withdraw failed (0 rows) | `false` | `post_objection_invalid` |

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "<error message>", "data": [] }
```

## 50a. YSAM Review Message

LLM-based **pre-send safety classification** for plain text (used by TAM Connections before `connections_send_message`, and available for other YSAM flows). Classifies user text and returns allow/block metadata.

- **Legacy endpoint:** /api.php/v1/ysam_review_message
- **Laravel route (mobile):** POST {APP_BASE}/ysam_review_message with **JSON** body (text, user_lang). Laravel forwards only those fields upstream.

- **Method:** POST

- **Rate Limit:** 40 requests / minute (authenticated + App Check)

**Parameters (canonical):**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| text | String | Yes | Message body to classify. Trimmed server-side; empty string → immediate `code:"0"`. |
| user_lang | String | No | Language code for localized classifier messages (default `"en"`). Validated against `app_supported_languages`. |

**Success Schema (200 OK — allowed):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `true`. |
| code | string | `"1"`. |
| text | string | Echo of the original user text. |
| category_check | object | Inner result from `classifyUserText()` — structure depends on LLM response. Contains at least `code` key. |

**Blocked Schema (200 OK — blocked):**

| Field | Type | Notes |
|---|---|---|
| status | boolean | `false`. |
| code | string | `"0"`. |
| message | string | `category_check.error` value or `"blocked"` fallback. |
| category_check | object | Inner classification result. |

**Runtime (`html/api/v1/ysam_review_message.php`)**

1. Loads `helpers/llm_executor.php` and `_shared_tam_classify.php` for the `classifyUserText()` function.
2. `HEAD` requests exit immediately (no processing).
3. If `text` is empty → `{ "status": false, "message": "empty", "code": "0" }`.
4. Calls `classifyUserText($text, $user_lang)`. If the function returns `false` → classification failed response with `classification_failed` constant.
5. Inspects `category_check.code`: if `"1"` → allowed (echoes original text). If `"0"` → blocked (returns `category_check.error` as `message`).
6. Logs errors to DB via `logtoDB` on exception.
7. Sets timezone to `Asia/Kolkata` explicitly.

**Catch envelope (Throwable):**

```json
{ "status": false, "message": "error", "code": "0" }
```

**Governance anchor:** heading ## 50a. YSAM Review Message is checked by dart run tool/contract_governance.dart against AppConstants.YSAM_REVIEW_MESSAGE ↔ Laravel ysam_review_message ↔ api.json key ysam_review_message.

## 51. YSAM Translate

Dynamically translates messages within the YSAM/Connections chat ecosystem. Contains heavy optimizations to skip LLM calls for emojis, numbers, or ultra-short strings. Persists translated messages and updates participant timestamps.

- **Endpoint:** /api.php/v1/ysam_translate

- **Method:** POST

- **Content-Type:** application/json OR application/x-www-form-urlencoded (auto-detected: if `$_POST` is empty, reads `php://input` as JSON)

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Uses a database transaction to safely insert into `tam_connections_conversation_messages` and update last_login timestamps.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|---|---|---|---|
| text | String | Yes | The text to translate. Consecutive ellipses collapsed (`...+` → `... `). |
| target | String | No | Target language code (default `"en"`). |
| user_lang | String | No | Source language hint / fallback (default `"en"`). |
| room | String | Yes | Chat room identifier — canonical format `room_{minUserId}_{maxUserId}` stored in `tam_connections_conversation_master.tam_conversations_room_name`. |
| sender | Integer | Yes | Reservation user ID of the message sender. Cast to `(int)`. |
| timestamp | String | No | Message timestamp (default: current server datetime `Y-m-d H:i:s`). |
| user_gender | String | No | `"M"` or `"F"` → mapped to `"male"` / `"female"` for LLM context. |
| user_timezone | String | No | IANA timezone (default: `global_application_timezone`). Used for display formatting. |

**Success Schema (200 OK):**

| Field | Type | Notes |
|---|---|---|
| code | integer | `1` success, `0` failure. Always an **int**. |
| text | string | Translated text (or original if same language / skip conditions). Empty string if input was empty. |
| detected_language | string | Detected source language code (e.g. `"es"`). Defaults to `"en"` if text was empty. |
| detected_language_full | string | Full language name (e.g. `"Spanish"`). `"Unknown"` if not in `SUPPORTED_LANGUAGES`. `"English"` if text was empty. |
| timestamp_display | string | Formatted date in user's timezone: `"Fri, 08 May 2026 (11:06 pm)"`. |

**Laravel proxy (mobile):** `POST /api/ysam_translate` → `YsamApiController::translateArticle` — Passport `auth:api`; `sender` is always token `reference_user_id` (client `sender` mismatch logged only). Pre-checks room via `Crit002eRoomAccessService`. Returns legacy `{code,text,detected_language,detected_language_full,timestamp_display}` decoded as-is (HTTP 200).

**Runtime (`html/api/v1/ysam_translate.php`)**

1. Loads `main.php` and `html/helpers/llm_*.php` (same LLM stack as `translate.php`).
2. **Input auto-detection:** If `$_POST` is empty, parses `php://input` as JSON and assigns to `$_POST`.
3. **Fast skip conditions** (no LLM call):
   - Text ≤ 1 character (`mb_strlen`).
   - Numbers/symbols only (`/^[\p{N}\p{S}\s]+$/u`).
   - Emoji only (`isEmojiOnly` strips all `\p{Emoji}\p{Z}\p{C}` and checks if nothing remains).
4. **Language detection:**
   - ASCII fast-path: if text is all ASCII and `user_lang == "en"`, source is `"en"` (no LLM call).
   - Otherwise: `identifyLanguage()` from `llm_helper.php`.
   - Urdu normalised to Hindi when `user_lang` is `"hi"`.
5. **Translation:** `handleTranslation()` calls `llm_execute('translate', ...)`. Only invoked when `!skipTranslation && sourceLanguage !== targetLanguage`.
6. **DB connection:** Opens its **own** `mysqli_connect` (not from `main.php`), sets timezone and charset.
7. **Message persistence (transactional):**
   - Looks up `tam_conversations_master_id` from `tam_connections_conversation_master` by `tam_conversations_room_name = room`. Invalid/missing room → exception.
   - Inserts into `tam_connections_conversation_messages` with original + translated text, detected language, timezone, and timestamp.
   - Updates `tam_conversations_user_id_1_last_login` or `tam_conversations_user_id_2_last_login` based on sender match.
   - **Weak translation guard:** `is_weak_translation()` — if the translated text is too similar to the original (and languages differ), the insert is **skipped**.
8. **Empty text:** Returns `code:1` with empty text, `"en"` / `"English"`, and current timestamp display (no DB write).

**Catch envelope (Throwable):**

```json
{ "code": 0, "text": "Error : <exception message>", "timestamp_display": "<formatted date>" }
```

## 52. Get Default Countries

## Endpoint

```text
POST /api.php/v1/get_default_countries
```

**Laravel proxy (mobile):** public `GET /api/get_default_countries` → same payload shape (no bearer on Laravel route).

## Purpose

Returns active countries, all currencies, and hard-coded default country/currency codes.

## Authentication

Bearer required for direct `html/api.php` v1. No POST fields are read by `html/api/v1/api_get_default_countries.php`.

## Request Parameters

None.

## Success Response

Top-level JSON only (no `status` / `message` envelope):

```json
{
  "countries": [],
  "currencies": [],
  "default_country": "IN",
  "default_currency": "INR"
}
```

`countries[]`

| Field | Type | Notes |
|---|---|---|
| name | string | `countries.country_name` where `active = 1`. |
| country_code | string | `countries.country_code`. |
| code | string | Duplicate of `country_code` (same column). |

`currencies[]`

| Field | Type | Notes |
|---|---|---|
| name | string | `currencies.currency_name`. |
| currency_code | string | `currencies.currency_code`. |
| symbol | string | `currencies.currency_symbol`, or **""** if null. |

## Failure Response

Same top-level keys; `countries` and `currencies` are empty arrays; defaults remain `IN` / `INR`.

## 53. API Health (probe)

Lightweight liveness probe for legacy router monitoring. Listed in `html/api/api.json` as `api_health` → `api_health.php`.

## Endpoint

```text
POST /api.php/v1/api_health
```

## Authentication

Bearer required (same `html/api.php` gate as other v1 scripts).

## Request Parameters

None.

## Success Response

```json
{
  "status": "ok",
  "timestamp": "YYYY-MM-DDTHH:MM:SSZ"
}
```

`timestamp` is UTC ISO-8601 from `gmdate()`.

## Failure Response

| Case | Body |
|---|---|
| Direct script access without router | Plain text `Forbidden` (HTTP 403) |
| Missing/invalid bearer | Router JSON error (HTTP 401/403) |

---

## Deep Link Trust apis (Laravel)

**Base path:** `/api/deep-links`  
**Protocol version:** `1.0.0`  
**Documentation:** `tam-admin-application/docs/features/deep-link/` (see also `TAM_FLUTTER/lib/services/deep_link/deep_link_protocol_spec.md`)

### GET `/api/deep-links/protocol`

Returns protocol version metadata (no auth).

### GET `/api/deep-links/keys`

Returns trust key metadata only (`kid`, `issuer`, `sigv`, `active_from`, `expires_at`). **No secrets.**

### GET `/api/deep-links/rollout-config`

Returns push deep-link rollout governance (no auth). Cached up to 300s. Metadata only — no secrets.

**Response:**

| Field | Type | Notes |
|---|---|---|
| useDeepLinkPushRouting | bool | When true, Flutter routes FCM/external opens through DeepLinkService |
| enableLegacyPushFallback | bool | When true, legacy `key` handlers may run if deep link fails |
| enforceSignedPushRoutes | bool | When true, unsigned sensitive push routes are rejected |
| rolloutStage | string | `observe_only` \| `dual_emit` \| `soft_enforce` \| `hard_enforce` |
| protocolVersion | string | e.g. `1.0.0` |
| updatedAt | string | ISO-8601 |
| routePolicies | object | Per-route `{ fallbackAllowed, signedRequired }` metadata |
| fallbackRetirementReadinessScore | int | 0–100 replay-route fallback retirement progress |
| staleClientRiskScore | int | 0–100 stale-client legacy routing risk estimate |
| staleClientMode | string | `observe_only` \| `warn_only` \| `restrict_high_trust` |

**Laravel env:** see `docs/features/deep-link/environment-configuration.md` and `.env.deep-link.example`.

### Artisan governance audits

| Command | Purpose |
|---|---|
| `php artisan deep-links:audit-legacy-navigation` | Scan Flutter/Laravel for notification `pushNamed` bypasses |
| `php artisan deep-links:audit-email-templates` | Scan email/HTML for `tam://`, handcrafted sensitive URLs |

Options: `--json`, `--ci` (fail fast on high severity).

### POST `/api/deep-links/sign`

**Auth:** `auth:api`  
**Rate limit:** 30/min per user  

**Body:**

| Field | Type | Required |
|---|---|---|
| base_url | string | yes |
| path_segments | string[] | no |
| query_params | object | no |
| ttl_seconds | int | no (60–604800) |
| replay_protected | bool | no |
| campaign | string | no |

**Response:** `{ status, url, protocol_version }`

### POST `/api/deep-links/revoke`

**Auth:** `auth:api` + observability role  
**Body:** `{ type: signature\|campaign\|route\|issuer\|nonce, key, reason? }`

### POST `/api/deep-links/ops-snapshot`

**Auth:** `auth:api`  
**Body:** `{ counters: { "<category>": <int> }, protocol_version? }` — metadata only, max 32 keys.

### POST `/api/deep-links/validate-fixtures`

Staging/local only. Runs backend/Flutter parity validation.

