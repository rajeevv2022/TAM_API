**API Documentation:**

**Global Standards Applied**

- **Rate Limiting:** Public routes are throttled at **20 req/min**. Authenticated routes are throttled at **40 req/min** (based on your Laravel api.php middleware).

- **HTTP Status Codes:** Most APIs return 200 OK universally, using an internal "code": 1 (success) or "code": 0 (failure) schema. Hard failures (missing files/auth) return 403 Forbidden, 404 Not Found, or 500 Internal Server Error.

- **Security:** `html/api/v1/*.php` files exit with plain `Forbidden` unless `SECURE_API_ACCESS` is defined. `html/api.php` defines that constant before including the mapped script.

- **Firebase App Check:** Not read or enforced by `html/api.php`. When the mobile app calls Laravel first, App Check applies at that layer per Laravel configuration.

### Mobile app path (Laravel proxy)

The Flutter app calls **Laravel** routes (for example POST {APP_BASE}/user_conversations_dashboard) with **JSON** bodies (Content-Type: application/json) and standard mobile headers. Laravel controllers forward to the legacy **/api.php/v1/{apiName}** endpoints using server-side bearer + form-style POST fields.

**Runtime rules (implemented):**

- **Connections dashboard (inbox):** Laravel returns { "status", "message", "data": { "conversations", "total_unread", "discover_groups", "dashboard_message", … } }. Clients also accept a **legacy-flat** shape where conversations and dashboard_message appear at the **root** (backwards compatibility).
- **Connections get messages (`get_connection_messages`):** Canonical upstream fields read in `html/api/v1/tam_connections_get_messages.php` are `room_name`, `lastSeq`, `beforeSeq`, `user_id` (or JWT-populated `auth_user_id` via `$_REQUEST`), and `user_lang`. Laravel may accept additional aliases; they are not read by this legacy file.
- **Connections send message:** Upstream persists **reply_to_message_id** only. The Laravel proxy accepts **reply_to_id** as an alias when reply_to_message_id is omitted (logged in app.debug only). Fields such as reply_to_content / reply_to_sender / timestamp_browser are **not** read by the legacy send handler and are not forwarded as distinct upstream fields.
- **Connections group subscribe:** Upstream requires **user_id** (reservation user id) and **group_id**.

### Contract governance (Phase 2)

- **Canonical vs compatibility:** Each Connections subsection below documents **canonical** legacy field names. **Compatibility aliases** accepted only at the Laravel proxy are listed under *Mobile app path (Laravel proxy)* above—not as legacy /api.php requirements.
- **Artifacts (Flutter repo):** TAM_FLUTTER/docs/governance/README.md, docs/governance/snapshots/critical_api_envelopes.json, and generated docs/governance/generated/CONTRACT_INVENTORY.* from the PHP builder.
- **Curated crosswalk (Dart, tool-only):** TAM_FLUTTER/tool/governance/contract_governance_crosswalk.dart — critical Flutter AppConstants ↔ Laravel route segment ↔ api.json key.
- **Local validation:** From TAM_FLUTTER/, run dart run tool/contract_governance.dart (fails if a curated route or api.json key is missing).
- **Broad inventory:** From tam-admin-application/tools/, run php governance_build_inventory.php (regenerates route/env/api.json lists).
- **Drift telemetry (mobile):** Repeated invalid **WebSocket token** JSON envelopes (HTTP 200 but missing data.jwt) emit a bounded Crashlytics breadcrumb (see ContractDriftWatch). Repeated **malformed WebSocket wire frames** (JSON decode failure, non-object root, or non-string type) emit a bounded Crashlytics event (ContractDriftWatch.noteMalformedWsWire). Maintenance unknown-key checks remain **debug-only** (debugAssertMaintenanceResponseContract).

### Operational governance (Phase 3 — release & deployment safety)

**Source of truth:** Runtime code (Laravel proxies, CurlPhp.php, legacy html/api/v1/*.php, Flutter parsers) wins over this document. Update APIs.md and TAM_FLUTTER/docs/governance/snapshots/critical_api_envelopes.json when those layers change.

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

## Base URL

text
https://[BASE_URL]
Development: https://localhost
Staging: https://tamweb.theablemind.com
Production: https://theablemind.com


All endpoints below are relative to the base URL.

### Documentation depth by section

Field-level tables for **nested** response objects (arrays of maps, `data.*` shapes) are present for: **§21 Third Banner**, **§30 Get Complete YSAM Post**, **§32 List YSAM Posts**, **§34–§42** (Connections family + user load), **§52 Get Default Countries**. Remaining numbered sections still use shorter parameter/example blocks; extend them using the same table style when auditing each `html/api/v1/*.php` file.

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

text
access_token=ya29.a0AfB_byC...&user_lang=en


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


## 2. Check Maintenance

Checks if the application is currently under maintenance and returns the availability status of specific modules.

- **Legacy endpoint:** /api.php/v1/check_maintenance
- **Laravel route (mobile):** POST {APP_BASE}/check_maintenance with **JSON** body (same field names: user_lang, timezone).

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded (legacy direct). **Mobile:** application/json to Laravel.

- **Rate Limit:** 20 requests / minute (Public Route)

- **Security Notes:** Unauthenticated. Relies on Firebase AppCheck to prevent abuse.

**Canonical response keys (mobile MaintenanceConfig):**

- **UPDATION_INPROGRESS** (bool): global maintenance lock when true.
- **Feature keys** (bool): REGISTRATION, FEEL_BETTER_IN_15, THERAPY_OVER_TEXT, NIGHT_AUXIE, TROOPERS_TOGETHER, LIBRARY, RESOURCES, ASSESSMENTS, CONNECTIONS, YSAM — true means feature **enabled**.
- **Meta:** MESSAGE_TITLE, MESSAGE, LIVE_BACK_TIME, SHORT_MESSAGE (optional tile hint).

Older examples may show MAINTENANCE_ACTIVE; the **runtime** mobile client uses **UPDATION_INPROGRESS** (see Flutter MaintenanceConfig).

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


**Success Schema (200 OK):**

json
{
"UPDATION_INPROGRESS": false,
"MESSAGE_TITLE": "Scheduled Maintenance",
"MESSAGE": "We will be back by Monday, 01 Jan 2026 [10:00 am EST]",
"LIVE_BACK_TIME": "2026-01-01T15:00:00Z",
"REGISTRATION": true,
"FEEL_BETTER_IN_15": false,
"THERAPY_OVER_TEXT": false,
"NIGHT_AUXIE": false,
"TROOPERS_TOGETHER": false,
"LIBRARY": false,
"RESOURCES": false,
"ASSESSMENTS": false,
"CONNECTIONS": false,
"YSAM": false,
"SHORT_MESSAGE": "Offline"
}

** Note: **
true = enabled
false means:
	- disabled
	- unavailable
	- hidden
	- under maintenance
depending on client behavior

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
| user_email | String | Cond. | User's email address (required if checking by email). |
| user_phone | String | Cond. | User's phone number (required if checking by phone). |
| login_type | String | No | "E" for Email, "P" for Phone (default: "E"). |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/check_merge_accounts \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_email=test@example.com" \
-d "login_type=E"


**Success Schema (200 OK):** *(Note: Internal wrapper returns data dynamically based on the underlying check_merge_accounts helper function).*

json
{
"status": "success",
"data": {
"merge_required": true,
"primary_user_id": 123,
"accounts_to_merge": [124, 125]
}
}


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
"message": "Email id provided is a disposable id. [BASE_URL] (tempmail.com) is invalid.",
"status": "failure",
"data": ""
}


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


**Success Schema (200 OK):**

json
{
"status": "success",
"data": "Accounts merged successfully"
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

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/create_user \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "login_type=E" \
-d "user_email=newuser@example.com" \
-d "user_name=Jane Doe" \
-d "user_password=SecurePass123"


**Success Schema (200 OK):**

json
{
"response_code": "1",
"status": "success",
"message": "Registration successful",
"data": { "user_id": 123 },
"show_user_usage_policy": ""
}

** Note: **
- The optional show_user_usage_policy field is dynamically generated using the user's assigned subscription category and current monthly feature restrictions.

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


## 11. Get Current Plan Summary

Returns a comprehensive summary of the user's active subscription plan, feature access, monthly usage limits, and expiry details.

- **Endpoint:** /api.php/v1/get_current_plan_summary

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                |
|-------------------|----------|--------------|--------------------------------|
| user_reference_id | Integer  | Yes          | User ID (reservation users).                       |
| user_email        | String   | No           | Primary user email.            |
| user_key          | String   | No           | Fallback for user email/phone. |
| user_lang         | String   | No           | Language code.                 |

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

## 12. Get Current Share Permissions

Fetches the user's current database toggles for sharing data (Journal, Mood, Habits) with counsellors.

- **Endpoint:** /api.php/v1/get_current_share_permissions

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

**Success Schema (200 OK):**

json
{
"status": "success",
"data": "Merge ignored successfully"
}


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
| referral_code     | String   | No           | Ambassador referral code.   |
| user_lang         | String   | No           | Language code.              |

**Success Schema (200 OK):**

json
{
  "code": "1",
  "razorpay_json": {
    "key": "rzp_live_...",
    "amount": 150000,
    "currency": "INR",
    "order_id": "order_...",
    "prefill": {
      "name": "John Doe",
      "email": "j@example.com"
    }
  }
}


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

**Success Schema (200 OK):**

json
{
"code": "1",
"message": "OTP sent successfully"
}


## 17. Share App Data via Email

Compiles a comprehensive HTML report of the user's active trackers (Journal, Mood, Anxiety, Habits, Assessments) and emails it securely to a designated counsellor.

- **Endpoint:** /api.php/v1/share_app_data_email

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Includes Strict regex filtering (/[<>{}\\|\\\\]/) on counsellor_message to prevent XSS injection. Validates email domain. Limited to 1 share per day per user.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | Integer | Yes | User ID (reservation users). |
| counsellor_email | String | Yes | Valid email address of the receiving counsellor. |
| counsellor_message | String | No | Custom message to inject into the HTML email. |

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Progress report successfully shared with your counsellor."
}


## 18. Share Private App Data (Permissions Toggle)

Updates the binary/encrypted database toggles dictating which trackers the user consents to share with their internal counsellor.

- **Endpoint:** /api.php/v1/share_private_app_data

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

**Success Schema (200 OK):**

json
{
"code": 1,
"message": "Settings updated successfully"
}


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
"message": "Your Premium plan is now active."
}


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
| user_id     | Integer  | Yes          | User ID (reservation users).                          |
| user_email  | String   | No           | User's email address.             |
| access_code | String   | No           | The corporate access code.        |
| employee_id | String   | No           | The user's corporate employee ID. |

**Success Schema (200 OK):**

json
{
"response_code": 1,
"message": "Access code updated successfully",
"status": "success",
"data": { }
}


## 23. Update User Information

Modifies core profile details, including contact numbers, guardian information (if the user is/was a minor), and access codes.

- **Endpoint:** /api.php/v1/update_user_information

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| primary_login_key | String | Yes | Email or Phone used to identify the user natively. |
| user_email | String | No | Updated email. |
| user_phone | String | No | Updated phone number. |
| user_guardian_name | String | Cond. | Required if minor. |
| user_guardian_phone | String | Cond. | Required if minor. |
| user_guardian_relationship | String | Cond. | Required if minor. |
| access_code | String | No | Corporate access code. |
| employee_id | String | No | Corporate employee ID. |

**Success Schema (200 OK):**

json
{
"status": "1",
"message": "Information updated successfully"
}


## 24. Validate Referral Code

Validates an ambassador referral code to dynamically calculate and apply discounts (routing logic based on domestic vs international pricing).

- **Endpoint:** /api.php/v1/validate_referral_code

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                 |
|-------------------|----------|--------------|---------------------------------|
| user_reference_id | String   | No           | User ID (reservation users).                        |
| referral_code     | String   | Yes          | The referral code to check.     |
| plan_type         | String   | Yes          | The subscription category code. |

**Success Schema (200 OK):**

json
{
"code": "1",
"discount": "10.00",
"message": "You've received a 10% discount through John's referral."
}


## 25. Chat Summary (Feel Better in 15)

Fetches LLM-powered summaries for a user's past "Feel Better in 15" chat sessions to give them a quick clinical or mood overview of past interactions.

- **Endpoint:** /api.php/v1/chat_summary_fb15

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_id | String | Yes | User ID (reservation users). |
| number_of_chats | Integer | No | Max historical chats to fetch and summarize (default:4). |

**Success Schema (200 OK):**

json
{
"code": "1",
"data": [
{
"date": "10 May 2026 10:00 am (2 days ago)",
"counsellor": "Jane Doe",
"chat_topic": "Anxiety Management",
"feedback_star": "5",
"summary": "User discussed work stress and coping strategies.<br><b>Close Chat Remarks:</b> Client reported feeling calmer."
}
]
}


## 26. Check Chat Usage (FB15 / TOT)

Monitors and enforces usage limits for "Feel Better in 15" (FB15) and "Therapy Over Text" (THoT) based on the user's active subscription plan.

- **Endpoint:** /api.php/v1/check_chat_usage

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Checks the user's active subscription category, global usage rules, and remaining monthly usage allowances before permitting access.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                    |
|-------------------|----------|--------------|------------------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users).                           |
| usage_check_type  | String   | No           | "fb15" or "tot" (default: "fb15"). |
| user_lang         | String   | No           | Language code.                     |

**Success Schema (200 OK - Usage Allowed):**

json
{
  "code": 1,
  "minutes_available": 45,
  "monthly_words_available": 85000,
  "show_buttons": {
    "count": 0,
    "buttons": []
  }
}

**Usage Enforcement Rules**
- FB15 usage may be controlled using:
    fb15_minutes_per_day
    fb15_minutes_per_month
- THoT usage is enforced using:
    thot_words_per_month
- Legacy message-count and tracking-unit based THoT enforcement has been deprecated.

**Error Schema (200 OK - Usage Exceeded):**

json
{
"code": 0,
"text": "You have exhausted your available usage limit for this plan.",
"show_buttons": {
"count": 2,
"buttons": [
{ "type": "redirect", "text": "Upgrade Plan", "redirect_code": "subscription" },
{ "type": "close", "text": "Close", "redirect_code": "close" }
]
}
}


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
|----|----|----|----|
| audio_message | String | Yes\* | Server-generated temporary upload path returned by /upload_audio. (\*Not required if check_lang_support is passed). |
| check_lang_support | Flag | No | If sent, endpoint only checks if the language is supported (bypasses transcription). |
| return_type | Integer | No | 0: Text, 1: Audio (TTS), 2: Text+Audio. |
| mode | String | No | TTS mode ("save" returns a file URL, "stream" returns raw audio bytes). |
| user_gender | String | No | "M", "F", or "O" (used to map TTS voice). |

**Success Schema (200 OK):**

json
{
"code": 1,
"text": "Hello, I need some help.",
"language": "en",
"language_full": "English",
"audio": "https://[BASE_URL]/translate_api/voice_notes/uid.mp3"
}


## 28. Generate Chat Summary (Cron / Manual)

Generates a structured clinical summary for a specific chat session using an LLM.

- **Endpoint:** /api.php/v1/generate_chat_summary

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| session_id | Integer | Cond. | Session ID to summarize. |
| update_all | String | Cond. | Pass "update_all" to process the entire backlog (requires elevated privileges/cron). |

**Success Schema (200 OK):**

json
{
"close_summary": "The user expressed feelings of isolation. Coping mechanisms were discussed."
}


## 29. Generate WebSocket Token

## Endpoint

```text
POST /api.php/v1/generate_ws_token
```

## Purpose

Issues an HS256 JWT (`tam_connections_jwt_secret`, payload `iss`=`theablemind_web`, `aud`=`tam_api`, `id`=`user_reference_id`, `name` decrypted display name, `iat`/`exp` with **3600s** lifetime) for WebSocket authentication.

## Authentication

Bearer required per `html/api.php` (JWT or static `BEARER_TOKEN` from `html/api/api.json`). This script does **not** require the bearer to match the `user_reference_id`; callers must align them at the application layer.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| user_reference_id | integer | Yes | — | Cast with `(int)`; values `<= 0` yield failure. |
| user_lang | string | No | en | Loaded for `session_error` includes on failure paths only. |

## Success Response

```json
{
  "status": true,
  "message": "JWT successful",
  "data": {
    "jwt": "<string>"
  }
}
```

## Failure Response

Typical missing user:

```json
{
  "status": false,
  "message": "User not provided",
  "data": []
}
```

Other failures use the same envelope; `message` is the exception message text.

## Notes

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

Converts incoming server-side audio files to FLAC format and transcribes them directly via Google Speech-to-Text API.

- **Endpoint:** /api.php/v1/google_audio_transcribe

- **Method:** POST

- **Content-Type:** application/json

- **Rate Limit:** 40 requests / minute

- **Security Notes:** The provided file path must reference a server-generated temporary upload and cannot reference arbitrary filesystem locations.

**Parameters (JSON Body):**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| audio_message | String | Yes | Server-generated temporary upload path returned by /upload_audio. |
| user_lang | String | No | BCP-47 Language code. |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/google_audio_transcribe \
-H "Authorization: Bearer <TOKEN>" \
-H "Content-Type: application/json" \
-d '{"audio_message": "/server/path/audio.mp3", "user_lang": "en"}'


**Success Schema (200 OK):**

json
{
"code": "1",
"language": "en-US",
"transcript": "I am feeling much better today."
}


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
| No conversation row for derived `tam_room_{min}_{max}` room | `error_blocking_user_text` |
| Plan summary error | `user_plan_insufficient_for_block` |
| Short / trial plan (`plan_duration` contains `day`) | `user_plan_insufficient_for_block` |
| Update affects 0 rows | `error_blocking_user_text` |
| Throwable in helper | `processing_error` |

Wrapper catch (e.g. missing user id before helper): `message` is exception string such as `User Id is not specified`. Invalid JSON from helper yields `message` **`invalid_request`**.

## Notes

- Room name inside helper: `tam_room_{min(blocking,blocked)}_{max(blocking,blocked)}`.


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

## Purpose

Returns up to **50** single-chat rows for one room (content field **not** decrypted in this endpoint), either newer than `lastSeq` or strictly older than `beforeSeq`.

## Authentication

Bearer required. User id resolution: `$_REQUEST['auth_user_id']` if set (JWT path in `html/api.php`), else `$_POST['user_id']`.

## Request Parameters

| Parameter | Type | Required | Default | Notes |
|---|---|---|---|---|
| room_name | string | Yes | — | Empty with empty user → `missing_fields`. |
| user_id | string/int | Conditional | — | Required when JWT does not populate `auth_user_id`. |
| lastSeq | mixed | No | 0 | Used only when `beforeSeq` is **0**; bound as integer for SQL `message_sequence > ?`. |
| beforeSeq | integer | No | 0 | When `> 0`, selects rows with `message_sequence < ?` ordered DESC then reversed; fetches `pageLimit+1` to set `has_more_older`. |
| user_lang | string | No | en | Loads `language_config/ysam_conversation/config_ysam_{user_lang}.php` only (for `missing_fields` / `conversation_not_found` / `technical_issue` constants). |

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
        "client_message_id": null,
        "sender_id": 1,
        "message_text": "<ciphertext from DB column tam_conversations_message_original; not decrypted in this endpoint>",
        "message_type": "text",
        "attachment_url": null,
        "created_at": "YYYY-MM-DD HH:MM:SS",
        "is_mine": true
      }
    ],
    "has_more_older": false
  }
}
```

### `data.messages[]` row (`tam_connections_get_messages.php`)

| Field | Type | Notes |
|---|---|---|
| message_sequence | integer | DB `message_sequence`. |
| server_seq | integer | Same as `message_sequence`. |
| client_message_id | string \| null | From DB. |
| sender_id | integer | `tam_conversations_user_id`. |
| message_text | string | **Ciphertext** from `tam_conversations_message_original` (not decrypted here). |
| message_type | string | From DB; defaults to `text` if null. |
| attachment_url | string \| null | Signed URL when attachment filename column set. |
| created_at | string | `tam_conversations_message_date`. |
| is_mine | boolean | `sender_id` equals requesting user. |

`attachment_url` is built by the local `generateAttachmentUrl` in `tam_connections_get_messages.php`: HMAC-SHA256 over `"{filename}|{exp}"` with `TAM_ATTACHMENT_SECRET`, URL prefix `TAM_BASE_URL + "/api.php?get_attachment=1&file=…&exp=…&sig=…"`. This differs from `functions_ysam.php::generateAttachmentUrl` (uses `hash('sha256', filename)` in the signed payload and `/api.php/v1/get_attachment?`). `html/api.php` attachment handling expects the **hashed-filename** variant and a request path containing `api.php/v1/`; clients must treat URL compatibility as deployment-specific.

## Failure Response

| Case | Body |
|---|---|
| Missing `room_name` or user id | `{"status":false,"message":"<missing_fields>","data":[]}` |
| Unknown room | `{"status":false,"message":"<conversation_not_found>","data":[]}` |
| DB Throwable | `{"status":false,"message":"<technical_issue>","data":[]}` |

## Notes

- `message_text` in this endpoint is **still encrypted at rest** in DB; PHP selects column `tam_conversations_message_original` into the array key `message_text` without decrypting (runtime returns ciphertext strings for content).


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
| reply_to | object \| null | See below. |

**Group (`connection_type` = `G`):** same keys except **`client_message_id`** is always **null** in PHP; **`translated`**, **`original_message`** are **omitted** (single-only return shape); `message_text` is decrypted group message body.

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

## 43. Core Translate

Translates text automatically using an LLM. Detects the source language and updates asynchronous therapy chat logs.

- **Endpoint:** /api.php/v1/translate

- **Method:** POST

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| text | String | Yes | Text to translate. |
| target | String | No | Target language code (default "en"). |
| type | String | No | "live" or "thot" (Therapy Over Text). |
| message_id | String | No | DB message ID to update the translated string into. |
| from_counsellor | String | No | "1" if sent by a counsellor, "0" if user. |
| user_gender | String | No | "M" or "F". |

**Success Schema (200 OK):**

json
{
"code": 1,
"text": "Translated content here",
"detected_language": "hi",
"detected_language_full": "Hindi"
}


## 44. Upload Audio

Uploads audio files to the server. Supports standard multipart file uploads or Base64 encoded recorded strings.

- **Endpoint:** /api.php/v1/upload_audio

- **Method:** POST

- **Content-Type:** multipart/form-data

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| type | String | Yes | 'R' for base64 recorded audio, or 'F' for file. |
| recordedAudio | String | Cond. | Base64 encoded string (if type is 'R'). |
| audioFile | File | Cond. | Binary file upload (if type is 'F'). |

**cURL Example (Base64 Mode):**

bash
curl -X POST https://[BASE_URL]/api.php/v1/upload_audio \
-H "Authorization: Bearer <TOKEN>" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "type=R" \
-d "recordedAudio=UklGRiQAAABXQVZFZm10IBAAAAABAAEAQB8AA..."


**Success Schema (200 OK):**

json
{
"code": 1,
"message": "file uploaded successfully. Please enter your authorization code and continue.",
"file": "/server/path/audio_uuid.wav"
}


## 45. YSAM Add/Update Article

Creates a new post or edits an existing one in the "Your Story and Mine" (YSAM) community feed.

- **Endpoint:** /api.php/v1/ysam_add_update_article

- **Method:** POST

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | Author's User ID (reservation users). |
| ysam_update | String | No | "Y" to update an existing article, "N" for new. |
| ysam_id | String | Cond. | Post ID (required if ysam_update is "Y"). |
| ysam_title | String | Yes | Title of the post. |
| article_body | String | Yes | Content of the post. |
| ysam_category_id | String | Yes | The ID of the category it belongs to. |
| ysam_hashtags | String | No | Comma-separated hashtags. |
| ysam_user_name | String | Yes | Author's display name or pseudonym. |

**Success Schema (200 OK):**

json
{
"status": true,
"message": "Article successfully submitted",
"data": { "ysam_id": 102 }
}


## 46. YSAM Connect to User

Sends a direct connection request (to initiate a 1-on-1 chat) to the author of a "Your Story and Mine" (YSAM) post.

- **Endpoint:** /api.php/v1/ysam_connect_to_user

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | The ID of the user requesting the connection. |
| author_reference_id | String | Yes | The ID of the YSAM author they want to connect with. |
| user_lang | String | No | Language code for localization. |

**cURL Example:**

bash
curl -X POST https://[BASE_URL]/api.php/v1/ysam_connect_to_user \
-H "Authorization: Bearer <TOKEN>" \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "user_reference_id=123" \
-d "author_reference_id=456"


**Success Schema (200 OK):**

json
{
"status": true,
"message": "Connection request sent successfully",
"data": []
}


## 47. YSAM Follow User

Toggles the "follow" status, allowing a user to subscribe to future posts from a specific YSAM author in their feed.

- **Endpoint:** /api.php/v1/ysam_follow_user

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | The ID of the user initiating the follow. |
| user_to_follow | String | Yes | The ID of the author being followed. |

**Success Schema (200 OK):**

json
{
"status": true,
"message": "You are now following this user",
"data": []
}


## 48. YSAM Get All Categories

Retrieves a complete list of categories available for YSAM posts, localized into the requested language. Used for filtering feeds or populating dropdowns.

- **Endpoint:** /api.php/v1/ysam_get_all_categories

- **Method:** GET / POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**  | **Type** | **Required** | **Description**                     |
|-----------|----------|--------------|-------------------------------------|
| user_lang | String   | No           | BCP-47 Language code (default: en). |

**Success Schema (200 OK):**

json
{
"status": true,
"message": "",
"data": [
{ "category_id": 1, "category_name": "Mental Health" },
{ "category_id": 2, "category_name": "Workplace Stress" }
]
}


## 49. YSAM Initialize Form

Aggregates metadata required to render the YSAM post creation form on the frontend, including the user's current posting eligibility and available categories.

- **Endpoint:** /api.php/v1/ysam_initialize_form

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description** |
|-------------------|----------|--------------|-----------------|
| user_reference_id | String   | Yes          | User ID (reservation users).        |
| user_lang         | String   | No           | Language code.  |

**Success Schema (200 OK):**

json
{
"status": true,
"message": "success",
"data": {
"categories": [
{ "category_id": 1, "category_name": "Mental Health" }
],
"user_can_post": true
}
}


## 50. YSAM Report User Post

Flags a YSAM post for review by the moderation team. Records the user submitting the report and their given reason.

- **Endpoint:** /api.php/v1/ysam_report_user_post

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | String | Yes | ID of the user reporting the post. |
| ysam_id | String | Yes | ID of the post being reported. |
| reason | String | Yes | The reason for the report (e.g., "Inappropriate content", "Spam"). |
| user_lang | String | No | Language code. |

**Success Schema (200 OK):**

json
{
"status": true,
"message": "Post has been successfully reported.",
"data": []
}


## 50a. YSAM Review Message

LLM-based **pre-send safety classification** for plain text (used by TAM Connections before connections_send_message, and available for other YSAM flows). Classifies user text and returns allow/block metadata.

- **Legacy endpoint:** /api.php/v1/ysam_review_message
- **Laravel route (mobile):** POST {APP_BASE}/ysam_review_message with **JSON** body (text, user_lang). Laravel forwards only those fields upstream.

- **Method:** POST

- **Rate Limit:** 40 requests / minute (authenticated + App Check)

**Parameters (canonical):**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| text | String | Yes | Message body to classify (trimmed server-side). |
| user_lang | String | No | BCP-47 language code for localized classifier messages (default en). |

**Success schema (200 OK, typical allow):**

json
{
  "status": true,
  "code": "1",
  "text": "user message echo",
  "category_check": {}
}


**Blocked / error (200 OK, typical):**

json
{
  "status": false,
  "code": "0",
  "message": "blocked_or_error_key",
  "category_check": {}
}


**Governance anchor:** heading ## 50a. YSAM Review Message is checked by dart run tool/contract_governance.dart against AppConstants.YSAM_REVIEW_MESSAGE ↔ Laravel ysam_review_message ↔ api.json key ysam_review_message.

## 51. YSAM Translate

Dynamically translates messages specifically within the YSAM chat/comments ecosystem. Contains heavy optimizations to skip LLM calls for emojis, numbers, or ultra-short strings to save compute costs.

- **Endpoint:** /api.php/v1/ysam_translate

- **Method:** POST

- **Content-Type:** application/json OR application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Uses a database transaction to safely update the tam_connections_conversation_messages table and update the last_login timestamps of the participants securely.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| text | String | Yes | The text to translate. |
| target | String | No | Target language code (default: en). |
| user_lang | String | No | Fallback source language code (default: en). |
| room | String | Yes | The chat room identifier (ysam_conversations_room_name). |
| sender | Integer | Yes | The ID of the user sending the message. |
| timestamp | String | No | Message timestamp. Defaults to current server time. |
| user_gender | String | No | "M" or "F" to provide contextual hints to the LLM. |
| user_timezone | String | No | User's timezone (default: global app timezone). |

**cURL Example (JSON Payload):**

bash
curl -X POST https://[BASE_URL]/api.php/v1/ysam_translate \
-H "Authorization: Bearer <TOKEN>" \
-H "Content-Type: application/json" \
-d '{
"text": "Me siento mucho mejor hoy, gracias.",
"target": "en",
"room": "ysam-room-999",
"sender": 123,
"user_timezone": "America/New_York"
}'


**Success Schema (200 OK):**

json
{
"code": 1,
"text": "I feel much better today, thank you.",
"detected_language": "es",
"detected_language_full": "Spanish",
"timestamp_display": "Fri, 08 May 2026 (11:06 pm)"
}


## 52. Get Default Countries

## Endpoint

```text
POST /api.php/v1/get_default_countries
```

## Purpose

Returns active countries, all currencies, and hard-coded default country/currency codes.

## Authentication

Bearer required. No POST fields are read by `html/api/v1/api_get_default_countries.php`.

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

### `countries[]` row

| Field | Type | Notes |
|---|---|---|
| name | string | `countries.country_name` where `active = 1`. |
| country_code | string | `countries.country_code`. |
| code | string | Duplicate of `country_code` (same column). |

### `currencies[]` row

| Field | Type | Notes |
|---|---|---|
| name | string | `currencies.currency_name`. |
| currency_code | string | `currencies.currency_code`. |
| symbol | string | `currencies.currency_symbol`, or **""** if null. |

## Failure Response

Same top-level keys; `countries` and `currencies` are empty arrays; defaults remain `IN` / `INR`.
