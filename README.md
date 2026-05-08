**API Documentation:**

**Global Standards Applied**

- **Rate Limiting:** Public routes are throttled at **20 req/min**. Authenticated routes are throttled at **40 req/min** (based on your Laravel api.php middleware).

- **HTTP Status Codes:** Most APIs return 200 OK universally, using an internal "code": 1 (success) or "code": 0 (failure) schema. Hard failures (missing files/auth) return 403 Forbidden, 404 Not Found, or 500 Internal Server Error.

- **Security:** All endpoints check for SECURE_API_ACCESS. Requests hitting the files directly without passing through the Laravel/PHP router will receive a 403 Forbidden.
- **Firebase AppCheck enforcement** applies to mobile application endpoints even when Bearer authentication is present.

### Development Environment Override
In development environments only, Firebase AppCheck validation may be bypassed by sending:
```http
X-Firebase-AppCheck: A1B2C3D4E5F6
```
This override is disabled in staging and production environments.

*Unless otherwise specified, POST endpoints use:*  
**Content-Type: application/x-www-form-urlencoded**

## Versioning

Current API Version: v1
All endpoints are versioned under:
/api.php/v1/

## Authentication
Authenticated endpoints require:

```http
Authorization: Bearer <TOKEN>
```

Mobile endpoints also require:
```http
X-Firebase-AppCheck: <APPCHECK_TOKEN>
```

## Response Standards

The API currently contains legacy and modern response formats.

Possible success indicators include:
- `"code": 1`
- `"response_code": "1"`
- `"status": "success"`
- `"status": true`

Note:
1. Numeric status fields may be returned as either integers or strings depending on legacy endpoint implementation.
2. Boolean values may be returned as native booleans, integers (1/0), or strings depending on legacy endpoint behavior.
3. Validate responses based on endpoint-specific schemas.

## Base URL

```text
https://[BASE_URL]
Development: https://localhost
Staging: https://tamweb.theablemind.com
Production: https://theablemind.com
```

All endpoints below are relative to the base URL.

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/check_counsellor_access \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "access_token=ya29.a0AfB_byC..." \
-d "user_lang=en"
```

**Success Schema (200 OK):**

```json
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
```

**Error Schema (200 OK - Soft Fail):**

```json
[
{
"code": 0,
"type": [],
"message": "Access Denied. This feature is only available to employees of The Able Mind."
}
]
```

## 2. Check Maintenance

Checks if the application is currently under maintenance and returns the availability status of specific modules.

- **Endpoint:** /api.php/v1/check_maintenance

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 20 requests / minute (Public Route)

- **Security Notes:** Unauthenticated. Relies on Firebase AppCheck to prevent abuse.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_lang | String | No | Language code to localize the maintenance message (default: en). |
| timezone | String | No | User's timezone to calculate the LIVE_BACK_TIME display (default: Asia/Kolkata). |

**cURL Example:**

```bash
curl -X POST https://[BASE_URL]/api.php/v1/check_maintenance \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_lang=en" \
-d "timezone=America/New_York"
```

**Success Schema (200 OK):**

```json
{
"UPDATION_INPROGRESS": true,
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
"YSAM": false
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/check_merge_accounts \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_email=test@example.com" \
-d "login_type=E"
```

**Success Schema (200 OK):** *(Note: Internal wrapper returns data dynamically based on the underlying check_merge_accounts helper function).*

```json
{
"status": "success",
"data": {
"merge_required": true,
"primary_user_id": 123,
"accounts_to_merge": [124, 125]
}
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/check_unique_user_name \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_name=johndoe99"
```

**Success Schema (200 OK - Name Available):**

```json
{
"response_code": "1",
"message": "success",
"status": "success",
"data": ""
}
```
**Error Schema (200 OK - Name Taken):**

```json
{
"response_code": "0",
"message": "Provided username is already taken. Please try another one.",
"status": "failure",
"data": ""
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/check_valid_email \
-H "Authorization: Bearer YOUR_LARAVEL_TOKEN" \
-H "X-Firebase-AppCheck: YOUR_APP_CHECK_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "user_email=legit.user@gmail.com"
```

**Success Schema (200 OK - Valid):**

```json
{
"response_code": "1",
"message": "Email id provided is valid.",
"status": "success",
"data": ""
}
```

**Error Schema (200 OK - Disposable/Invalid):**

```json
{
"response_code": "-1",
"message": "Email id provided is a disposable id. [BASE_URL] (tempmail.com) is invalid.",
"status": "failure",
"data": ""
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/confirm_merge_accounts \
-H "Authorization: Bearer <TOKEN>" \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "user_id=123" \
-d "account_list=124,125"
```

**Success Schema (200 OK):**

```json
{
"status": "success",
"data": "Accounts merged successfully"
}
```

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

```json
{
"response_code": "1",
"message": "success",
"status": "success",
"data": ""
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/create_user \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "login_type=E" \
-d "user_email=newuser@example.com" \
-d "user_name=Jane Doe" \
-d "user_password=SecurePass123"
```

**Success Schema (200 OK):**

```json
{
"response_code": "1",
"status": "success",
"message": "Registration successful",
"data": { "user_id": 123 },
"show_user_usage_policy": ""
}
```

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

```json
{
"code": 1,
"message": "Account successfully deleted"
}
```

**Error Schema (200 OK - Active Subscription):**

```json
{
"code": 0,
"message": "Cannot delete account with an active subscription."
}
```

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

```json
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
```

## 11. Get Current Plan Summary

Returns a comprehensive summary of the user's active subscription plan, limitations, and expiry dates.

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

```json
{
"response_code": 1,
"status": "success",
"message": "success",
"data": {
"plan_name": "Premium",
"end_date": "2026-12-31",
"fb15_minutes_per_day": 60,
"thot_tracking_unit": "w",
"trial_category": "N"
}
}
```

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

```json
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
```

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

```json
{
"quote": "The best way out is always through.",
"author": "Robert Frost"
}
```

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

```json
{
"status": "success",
"data": "Merge ignored successfully"
}
```

## 15. Initiate Subscription Payment

Creates a Razorpay Order ID for purchasing a subscription, factoring in GST, applicable referral discounts, and corporate pricing.

- **Endpoint:** /api.php/v1/initiate_subscription_payment

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Generates a secure txnid and stores preliminary INITIATED status in global_mysql_payment_table.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**             |
|-------------------|----------|--------------|-----------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users).                    |
| plan_type         | String   | Yes          | Subscription Category Code. |
| referral_code     | String   | No           | Ambassador referral code.   |
| user_lang         | String   | No           | Language code.              |

**Success Schema (200 OK):**

```json
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
```

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

```json
{
"code": "1",
"message": "OTP sent successfully"
}
```

## 17. Share App Data via Email

Compiles a comprehensive HTML report of the user's active trackers (Journal, Mood, Anxiety, Habits, Assessments) and emails it securely to a designated counsellor.

- **Endpoint:** /api.php/v1/share_app_data_email

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Includes Strict regex filtering (`/[<>{}\\|\\\\]/`) on counsellor_message to prevent XSS injection. Validates email domain. Limited to 1 share per day per user.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| user_reference_id | Integer | Yes | User ID (reservation users). |
| counsellor_email | String | Yes | Valid email address of the receiving counsellor. |
| counsellor_message | String | No | Custom message to inject into the HTML email. |

**Success Schema (200 OK):**

```json
{
"code": 1,
"message": "Progress report successfully shared with your counsellor."
}
```

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

```json
{
"code": 1,
"message": "Settings updated successfully"
}
```

## 19. Show Plans

Fetches all available subscription plans relevant to the user, masking irrelevant plans if the user is bound to a corporate domain.

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

```json
{
"status": "success",
"data": [
{
"plan_code": "PREM1M",
"price": 999,
"duration": "1 month",
"description": "Unlimited text therapy and library access."
}
]
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/subscription_payment_confirmation \
-H "Authorization: Bearer <TOKEN>" \
-d "razorpay_order_id=order_123abc" \
-d "razorpay_payment_id=pay_123abc" \
-d "razorpay_signature=a1b2c3d4e5f6g7h8..."
```

**Success Schema (200 OK):**

```json
{
"code": "1",
"message": "Your Premium plan is now active."
}
```

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

```json
{
"code": "1",
"next_appointment": { "date": "2026-06-01", "time": "10:00 AM" },
"tot": { "status": "active" },
"next_slot": { "counsellor": "Jane Doe", "next_counsellor_slot": ["10:00 AM", "11:00 AM"] },
"tam_library_url": "https://[BASE_URL]/tam-app-library.php?token=...",
"tam_resource_center_url": "https://[BASE_URL]/tam-app-resource-center.php?token=..."
}
```

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

```json
{
"response_code": 1,
"message": "Access code updated successfully",
"status": "success",
"data": { }
}
```

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

```json
{
"status": "1",
"message": "Information updated successfully"
}
```

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

```json
{
"code": "1",
"discount": "10.00",
"message": "You've received a 10% discount through John's referral."
}
```

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

```json
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
```

## 26. Check Chat Usage (FB15 / TOT)

Monitors and enforces limits on "Feel Better in 15" (minutes) or "Therapy Over Text" (messages/words) based on the user's active plan.

- **Endpoint:** /api.php/v1/check_chat_usage

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Checks the user's global plan tier. Enforces a hardcoded cooling period for FB15.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**                    |
|-------------------|----------|--------------|------------------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users).                           |
| usage_check_type  | String   | No           | "fb15" or "tot" (default: "fb15"). |
| user_lang         | String   | No           | Language code.                     |

**Success Schema (200 OK - Usage Allowed):**

```json
{
"code": 1,
"minutes_available": 45,
"words_available": "",
"show_buttons": { "count": 0, "buttons": [] }
}
```

**Error Schema (200 OK - Usage Exceeded):**

```json
{
"code": 0,
"text": "You have exhausted your daily chat limit.",
"show_buttons": {
"count": 2,
"buttons": [
{ "type": "redirect", "text": "Upgrade Plan", "redirect_code": "subscription" },
{ "type": "close", "text": "Close", "redirect_code": "close" }
]
}
}
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
|----|----|----|----|
| audio_message | String | Yes\* | Server-generated temporary upload path returned by /upload_audio. (\*Not required if check_lang_support is passed). |
| check_lang_support | Flag | No | If sent, endpoint only checks if the language is supported (bypasses transcription). |
| return_type | Integer | No | 0: Text, 1: Audio (TTS), 2: Text+Audio. |
| mode | String | No | TTS mode ("save" returns a file URL, "stream" returns raw audio bytes). |
| user_gender | String | No | "M", "F", or "O" (used to map TTS voice). |

**Success Schema (200 OK):**

```json
{
"code": 1,
"text": "Hello, I need some help.",
"language": "en",
"language_full": "English",
"audio": "https://[BASE_URL]/translate_api/voice_notes/uid.mp3"
}
```

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

```json
{
"close_summary": "The user expressed feelings of isolation. Coping mechanisms were discussed."
}
```

## 29. Generate WebSocket Token

Dispenses a short-lived JWT token securely signed by the backend, allowing the frontend Dart client to connect to the WebSocket server.

- **Endpoint:** /api.php/v1/generate_ws_token

- **Method:** POST

- **Content-Type:** application/x-www-form-urlencoded

- **Rate Limit:** 40 requests / minute

- **WebSocket Lifecycle:** The client uses the returned JWT to connect to wss://socket.theablemind.com:3000. Token expires in 1 hour.

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**               |
|-------------------|----------|--------------|-------------------------------|
| user_reference_id | Integer  | Yes          | User ID (reservation users) requesting the token. |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "JWT successful",
"data": {
"jwt": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
}
```

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

```json
{
"code": "1",
"data": {
"ysam_id": "987",
"title": "My Journey",
"content": "Full article content..."
}
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/google_audio_transcribe \
-H "Authorization: Bearer <TOKEN>" \
-H "Content-Type: application/json" \
-d '{"audio_message": "/server/path/audio.mp3", "user_lang": "en"}'
```

**Success Schema (200 OK):**

```json
{
"code": "1",
"language": "en-US",
"transcript": "I am feeling much better today."
}
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

```json
{
"code": "1",
"message": "",
"data": {
"posts": [
{ "ysam_id": "101", "title": "Coping with Stress" }
],
"total_pages": 5
}
}
```

## 33. Connections Block User

Blocks a user within the TAM Connections module, preventing further messaging between the two parties.

- **Endpoint:** /api.php/v1/connections_block_user

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| blocking_user_id | String | Yes | The ID of the user performing the block. |
| blocked_user_id | String | Yes | The ID of the user being blocked. |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "User blocked successfully",
"data": []
}
```

## 34. Connections Check Consent

Verifies if a user has signed the active community guidelines/consent for Connections.

- **Endpoint:** /api.php/v1/check_connections_consent

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----------|----------|--------------|-----------------|
| user_id  | Integer  | Yes          | User ID (reservation users).        |

**Success Schema (200 OK - Consent Required):**

```json
{
"status": true,
"message": "consent_required",
"data": {
"required": true,
"guidelines": {
"title": "Community Guidelines",
"consent_text": "I Agree"
}
}
}
```

## 35. Connections Check Usage

Ensures the user has not exceeded their daily allowed connection interactions based on their plan limits.

- **Endpoint:** /api.php/v1/connections_check_usage

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description** |
|-------------------|----------|--------------|-----------------|
| user_reference_id | String   | Yes          | User ID (reservation users).        |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "",
"data": { "allowed": true }
}
```

## 36. Connections Dashboard

Loads the primary inbox view for TAM Connections, including active 1-1 chats, joined groups, unread counts, and groups available to join.

- **Endpoint:** /api.php/v1/user_conversations_dashboard

- **Method:** POST

- **Rate Limit:** 40 requests / minute

**Parameters:**

| **Name**          | **Type** | **Required** | **Description** |
|-------------------|----------|--------------|-----------------|
| user_reference_id | String   | Yes          | User ID (reservation users).        |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Success",
"data": {
"conversations": [],
"total_unread": 3,
"discover_groups": []
}
}
```

## 37. Connections Get Messages

Fetches conversation history in batches, utilizing sequence numbers for ordering, and generates secure URLs for attachments.

- **Endpoint:** /api.php/v1/get_connection_messages

- **Method:** POST

- **Pagination Rules:** Cursor-based pagination using lastSeq. Pass the highest message_sequence from the previous response to get the next batch. Returns up to 50 messages per call.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| auth_user_id | Integer | Yes | User ID (reservation users) (fallback to user_id). |
| room_name | String | Yes | The unique identifier for the chat room. |
| lastSeq | Integer | No | The sequence ID of the last fetched message (default 0). |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "success",
"data": {
"messages": [
{
"message_sequence": 1,
"server_seq": 1,
"client_message_id": "uuid-123",
"sender_id": 123,
"message_text": "Hello!",
"message_type": "text",
"attachment_url": null,
"created_at": "2026-05-08 10:00:00",
"is_mine": true
}
]
}
}
```

## 38. Connections Group Subscribe

Safely subscribes a user to a group if they meet the Open, Corporate, or Subscription-based access rules.

- **Endpoint:** /api.php/v1/connections_group_subscribe

- **Method:** POST

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Uses FOR UPDATE row-locking to ensure atomic member count increments.

**Parameters:**

| **Name** | **Type** | **Required** | **Description**            |
|----------|----------|--------------|----------------------------|
| user_id  | Integer  | Yes          | User ID (reservation users) joining the group. |
| group_id | Integer  | Yes          | Group ID.                  |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Successfully subscribed to the group",
"data": []
}
```

## 39. Connections Group Unsubscribe

Safely removes a user from a group, decrementing the active member count and clearing their unread state history.

- **Endpoint:** /api.php/v1/connections_group_unsubscribe

- **Method:** POST

**Parameters:**

| **Name** | **Type** | **Required** | **Description**            |
|----------|----------|--------------|----------------------------|
| user_id  | Integer  | Yes          | User ID (reservation users) leaving the group. |
| group_id | Integer  | Yes          | Group ID.                  |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Successfully unsubscribed",
"data": []
}
```

## 40. Connections Register Consent

Records a user's formal acceptance of the connection guidelines and calculates their expiry period (validity days).

- **Endpoint:** /api.php/v1/register_connections_consent

- **Method:** POST

**Parameters:**

| **Name** | **Type** | **Required** | **Description**            |
|----------|----------|--------------|----------------------------|
| user_id  | Integer  | Yes          | User ID (reservation users) providing consent. |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Consent registered successfully",
"data": []
}
```

## 41. Connections Send Message

A robust controller for sending messages in 1-on-1 or Group chats. Performs safety classifications via LLM (checking for self-harm, external links) and automatically translates cross-language 1-on-1 chats.

- **Endpoint:** /api.php/v1/connections_send_message

- **Method:** POST

- **Content-Type:** multipart/form-data

- **Rate Limit:** 40 requests / minute

- **Security Notes:** Implements strict Mime-Type checking via finfo_file for file uploads. Max size is regulated by CONNECTIONS_MAX_ATTACHMENT_SIZE. Generates image thumbnails automatically.

**Parameters:**

| **Name** | **Type** | **Required** | **Description** |
|----|----|----|----|
| sender_reference_id | String | Yes | ID of the sender. |
| recipient_reference_id | Integer | Cond. | ID of the recipient (required if room_type is 'S'). |
| room_name | String | Yes | The unique identifier for the chat room. |
| room_type | String | Yes | 'S' (Single/1-1) or 'G' (Group). |
| message | String | Cond. | The text message content (required if no attachment). |
| attachment | File | Cond. | Binary file attachment (required if no message). |
| client_message_id | String | No | Local ID to prevent duplicate insertions. |
| reply_to_message_id | Integer | No | ID of the message being replied to. |

**cURL Example (Multipart Upload):**

```bash
curl -X POST https://[BASE_URL]/api.php/v1/connections_send_message \
-H "Authorization: Bearer <TOKEN>" \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-F "sender_reference_id=123" \
-F "room_name=room-uuid" \
-F "room_type=G" \
-F "message=Check out this file!" \
-F "attachment=@/path/to/local/image.jpg"
```

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "success",
"data": {
"text": "Check out this file!",
"message_sequence": 42,
"client_message_id": "local-uuid-123",
"attachment_url": "https://[BASE_URL]/api.php?get_attachment=1&...",
"detected_language": "en"
}
}
```

## 42. User Load Conversations

Initializes and loads the metadata (participants, room validity) required to open a specific chat room.

- **Endpoint:** /api.php/v1/user_conversations_load

- **Method:** POST

**Parameters:**

| **Name**          | **Type** | **Required** | **Description**              |
|-------------------|----------|--------------|------------------------------|
| user_reference_id | String   | Yes          | User ID (reservation users).                     |
| connection_type   | String   | Yes          | 'S' (Single) or 'G' (Group). |
| room_name         | String   | Yes          | The unique room identifier.  |

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Success",
"data": {
"room_name": "room-uuid",
"participants": []
}
}
```

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

```json
{
"code": 1,
"text": "Translated content here",
"detected_language": "hi",
"detected_language_full": "Hindi"
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/upload_audio \
-H "Authorization: Bearer <TOKEN>" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "type=R" \
-d "recordedAudio=UklGRiQAAABXQVZFZm10IBAAAAABAAEAQB8AA..."
```

**Success Schema (200 OK):**

```json
{
"code": 1,
"message": "file uploaded successfully. Please enter your authorization code and continue.",
"file": "/server/path/audio_uuid.wav"
}
```

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

```json
{
"status": true,
"message": "Article successfully submitted",
"data": { "ysam_id": 102 }
}
```

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

```bash
curl -X POST https://[BASE_URL]/api.php/v1/ysam_connect_to_user \
-H "Authorization: Bearer <TOKEN>" \
-H "X-Firebase-AppCheck: <APPCHECK_TOKEN>" \
-d "user_reference_id=123" \
-d "author_reference_id=456"
```

**Success Schema (200 OK):**

```json
{
"status": true,
"message": "Connection request sent successfully",
"data": []
}
```

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

```json
{
"status": true,
"message": "You are now following this user",
"data": []
}
```

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

```json
{
"status": true,
"message": "",
"data": [
{ "category_id": 1, "category_name": "Mental Health" },
{ "category_id": 2, "category_name": "Workplace Stress" }
]
}
```

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

```json
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
```

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

```json
{
"status": true,
"message": "Post has been successfully reported.",
"data": []
}
```

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

```bash
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
```

**Success Schema (200 OK):**

```json
{
"code": 1,
"text": "I feel much better today, thank you.",
"detected_language": "es",
"detected_language_full": "Spanish",
"timestamp_display": "Fri, 08 May 2026 (11:06 pm)"
}
```