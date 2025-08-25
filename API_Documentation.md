This document provides structured documentation for the available
APIs.  
Each API includes details such as purpose, request structure,
parameters, authentication, and example JSON responses.

---

### `Base` URL

https://localhost/api.php/v1/

---

### `Authentication`

All API endpoints require Bearer Token authentication.

Header Format:

Authorization: Bearer [BEARER_TOKEN]

---

### `Endpoints` Index

- [chat_summary_fb15](#chat_summary_fb15)

- [check_chat_usage](#check_chat_usage)

- [check_counsellor_access](#check_counsellor_access)

- [check_maintenance](#check_maintenance)

- [check_merge_accounts](#check_merge_accounts)

- [check_unique_user_name](#check_unique_user_name)

- [check_valid_email](#check_unique_user_name)

- [confirm_merge_accounts](#confirm_merge_accounts)

- [create_audio_conference](#create_audio_conference)

- [create_user](#create_user)

- [delete_user_account](#delete_user_account)

- [get_current_plan_summary](#get_current_plan_summary)

- [get_current_share_permissions](#get_current_share_permissions)

- [get_random_quote](#get_random_quote)

- [ignore_merge_account](#ignore_merge_account)

- [initiate_subscription_payment](#initiate_subscription_payment)

- [send_registration_otp](#send_registration_otp)

- [show_plans](#show_plans)

- [subscription_payment_confirmation](#subscription_payment_confirmation)

- [tam_connections_consent](#tam_connections_consent)

- [third_banner](#third_banner)

- [update_user_access_code](#update_user_access_code)

- [update_user_information](#update_user_information)

- [validate_referral_code](#validate_referral_code)

- [audio_transcribe](#audio_transcribe)

- [check_tam_connections_consent](#check_tam_connections_consent)

- [generate_chat_summary](#generate_chat_summary)

- [get_complete_ysam_post](#get_complete_ysam_post)

- [google_audio_transcribe](#google_audio_transcribe)

- [list_tam_user_conversations_dashboard](#list_tam_user_conversations_dashboard)

- [list_ysam_posts](#list_ysam_posts)

- [register_tam_connections_consent](#register_tam_connections_consent)

- [tam_connections_block_user](#tam_connections_block_user)

- [tam_connections_send_message](#tam_connections_send_message)

- [translate](#translate)

- [upload_audio](#upload_audio)

- [user_conversations_load](#user_conversations_load)

- [ysam_add_update_article](#ysam_add_update_article)

- [ysam_check_usage_tam_connections](#ysam_check_usage_tam_connections)

- [ysam_connect_to_user](#ysam_connect_to_user)

- [ysam_follow_user](#ysam_follow_user)

- [ysam_get_all_categories](#ysam_get_all_categories)

- [ysam_initialize_form](#ysam_initialize_form)

- [ysam_report_user_post](#ysam_report_user_post)

- [ysam_review_message](#ysam_review_message)

- [ysam_translate](#ysam_translate)

- [share_private_app_data](#share_private_app_data)

---

### `chat_summary_fb15`

**File:** `api_chat_summary_fb15.php`

**Method:** `POST`

**Description:** Generates a summary of past chats for a user using an
external AI service. It fetches chat history, generates a summary if one
doesn't exist, and returns the chat history with summaries. The
generated summary includes mood analysis and identifies a central topic.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_id | int | Yes | The user's ID. |
| number_of_chats | int | No | The number of chat summaries to retrieve. Defaults to a minimum message count for summary generation. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "data": [
    {
      "date": "21 Dec 2023 10:30 am (A few hours ago)",
      "counsellor": "Counsellor Name",
      "chat_topic": "Romantic Relationship Issues",
      "feedback_star": "4",
      "summary": "This is a detailed summary of the\nconversation.\n**Close Chat Remarks:** A remark from the\ncounsellor."
    }
  ]
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User id does not exist"
}
```

---

### `check_chat_usage`

**File:** `api_check_chat_usage.php`

**Method:** `POST`

**Description:** Checks a user's usage against their subscribed chat plan
(e.g., Feel Better in 15 or Therapy Over Text) and determines if they
can initiate a new chat.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The user's ID. |
| usage_check_type | string | Yes | The type of chat to check, either "fb15" or another type. |
| user_lang | string | No | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "minutes_available": 15,
  "words_available": "",
  "show_buttons": {
    "count": 0,
    "buttons": []
  }
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "text": "Your usage for Feel Better in 15 has been exceeded. Please\nconsider Therapy Over Text as an alternative.",
  "show_buttons": {
    "count": 3,
    "buttons": [
      {
        "type": "redirect",
        "text": "Go to Therapy over Text",
        "redirect_code": "therapy"
      },
      {
        "type": "redirect",
        "text": "View Subscription Plans",
        "redirect_code": "subscription"
      },
      {
        "type": "close",
        "text": "Close",
        "redirect_code": "close"
      }
    ]
  }
}
```

---

### `check_counsellor_access`

**File:** `api_check_counsellor_access.php`

**Method:** `POST`

**Description:** Authenticates a user with a Google theablemind.com domain
and provisions them as a counselor, admin, or ops lead if they do not
exist in the system.

## Parameters

| Name         | Type   | Required | Description                     |
|--------------|--------|----------|---------------------------------|
| access_token | string | Yes      | The Google access token.        |
| user_lang    | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
[
  {
    "code": 1,
    "type": [
      "A",
      "C"
    ],
    "c_id": "123",
    "c_name": "John Doe",
    "c_email": "john.doe@theablemind.com",
    "c_phone": "1234567890",
    "c_timezone": "Asia/Kolkata",
    "c_age": "35",
    "c_gender": "M",
    "c_picture": "http://example.com/images/john.jpg",
    "c_lang_code": "en",
    "c_country": "IN",
    "c_video": "http://example.com/videos/john_intro.mp4",
    "message": ""
  }
]
```

Failure (code: 0)

```json
[
  {
    "code": 0,
    "type": [],
    "c_id": "",
    "c_name": "",
    "c_email": "",
    "c_phone": "",
    "c_timezone": "",
    "c_age": "",
    "c_gender": "",
    "c_picture": "",
    "c_lang_code": "",
    "c_country": "",
    "c_video": "",
    "message": "Access Denied. This feature is only available to employees\nof The Able Mind."
  }
]
```

---

### `check_maintenance`

**File:** `api_check_maintenance.php`

**Method:** `POST`

**Description:** Checks for any ongoing system maintenance and informs the
user if specific app features are disabled.

## Parameters

| Name      | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| user_lang | string | No       | The user's language preference. |
| timezone  | string | No       | The user's timezone.            |

## Responses

Success (returns a status object)

```json
{
  "UPDATION_INPROGRESS": true,
  "MESSAGE_TITLE": "Maintenance Alert",
  "MESSAGE": "We are undergoing a scheduled maintenance. The service will\nbe back on Saturday, 24 Aug 2024 [05:30 am IST]",
  "LIVE_BACK_TIME": "2024-08-24T00:00:00Z",
  "REGISTRATION": false,
  "FEEL_BETTER_IN_15": true,
  "THERAPY_OVER_TEXT": true,
  "NIGHT_AUXIE": false,
  "TROOPERS_TOGETHER": true,
  "LIBRARY": false,
  "RESOURCES": false,
  "ASSESSMENTS": false,
  "CONNECTIONS": false,
  "YSAM": false
}
```

No Maintenance (returns a default object)

```json
{
  "UPDATION_INPROGRESS": false,
  "MESSAGE_TITLE": "",
  "MESSAGE": "",
  "LIVE_BACK_TIME": "",
  "REGISTRATION": false,
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

---

### `check_merge_accounts`

**File:** `api_check_merge_accounts.php`

**Method:** `POST`

**Description:** Checks if a user has multiple accounts that should be
merged, typically based on email or phone number.

## Parameters

| Name       | Type   | Required | Description                                |
|------------|--------|----------|--------------------------------------------|
| user_email | string | No       | The user's email address.                  |
| user_phone | string | No       | The user's phone number.                   |
| login_type | string | No       | The login type, defaults to 'E' for email. |

## Responses

Success (status: success)

```json
{
  "status": "success",
  "data": {
    "show_merge_screen": true,
    "primary_account_name": "User Name",
    "secondary_account_name": "Duplicate User",
    "message": "We have found a duplicate account for you. Would you like to\nmerge them?"
  }
}
```

Error (status: error)

```json
{
  "status": "error",
  "data": "A technical issue has occurred."
}
```

---

### `check_unique_user_name`

**File:** `api_check_unique_user_name.php`

**Method:** `POST`

**Description:** Validates if a user name is unique in the system.

## Parameters

| Name      | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| user_name | string | Yes      | The user name to validate.      |
| login_key | string | No       | The user's login key.           |
| user_lang | string | No       | The user's language preference. |

## Responses

Success (response_code: 1)

```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": ""
}
```

Failure (response_code: 0)

```json
{
  "response_code": "0",
  "message": "User Name is not unique",
  "status": "failure",
  "data": ""
}
```

---

### `check_valid_email`

**File:** `api_check_valid_email.php`

**Method:** `POST`

**Description:** Validates an email address by checking for a valid format,
existing MX records, and whether the domain is a known disposable email
provider.

## Parameters

| Name       | Type   | Required | Description                    |
|------------|--------|----------|--------------------------------|
| user_email | string | Yes      | The email address to validate. |

## Responses

Success (response_code: 0)

```json
{
  "response_code": "0",
  "message": "Email id provided is valid.",
  "status": "success",
  "data": ""
}
```

Failure (response_code: -1)

```json
{
  "response_code": "-1",
  "message": "Email domain (mailinator.com) is invalid.",
  "status": "failure",
  "data": ""
}
```

---

### `confirm_merge_accounts`

**File:** `api_confirm_merge_accounts.php`

**Method:** `POST`

**Description:** Confirms and completes the merging of two user accounts by
linking two different user profiles into a single account.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_id | int | Yes | The ID of the user initiating the merge. |
| duplicate_id | int | Yes | The ID of the account to merge into the primary account. |
| login_type | string | Yes | The primary login type for the merged account ('E' for email or 'P' for phone). |

## Responses

Success (status: success)

```json
{
  "status": "success",
  "data": "Accounts successfully merged"
}
```

Error (status: error)

```json
{
  "status": "error",
  "data": "Error merging accounts"
}
```

---

### `create_audio_conference`

**File:** `api_create_audio_conference.php`

**Method:** `POST`

**Description:** Creates or updates an audio conference.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| conference_id | string | Yes | Unique ID for the conference. If it exists, all data will be updated; otherwise, it will be inserted. |
| conference_duration | int | No | Duration of the session in minutes. Defaults to 60. |
| conference_start_time | string | Yes | Conference start date and time in "Y-m-d H:i:s" format. |
| conference_counsellor_keys | string | Yes | Comma-separated list of counsellor keys. |
| conference_title | string | Yes | Title of the conference. |
| max_simultaneous_connections | int | No | Maximum number of simultaneous connections. Defaults to 20. |

## Responses

Success (response_code: 1)

```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": ""
}
```

Failure (response_code: 0)

```json
{
  "response_code": "0",
  "message": "Error Creating / Updating Conference",
  "status": "failure",
  "data": ""
}
```

---

### `create_user`

**File:** `api_create_user.php`

**Method:** `POST`

**Description:** Registers a new user account with various details such as
name, email, phone, and demographic information.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| login_type | string | Yes | 'E' for email or 'P' for phone. |
| user_name | string | Yes | User's name. |
| user_email | string | Conditional | Required if login_type is 'E'. |
| user_phone_number | string | Conditional | Required if login_type is 'P'. |
| user_password | string | No | The user's password. |
| user_age | string | No | User's age. |
| user_sex | string | No | User's gender. |
| user_lang | string | No | The user's language preference. |
| external_login | string | No | External login identifier. |
| user_guardian_name | string | No | Guardian name. |
| user_guardian_phone | string | No | Guardian phone. |
| user_guardian_relationship | string | No | Guardian relationship. |
| client_ip_address | string | No | Client IP address. |
| device_id | string | No | Device ID. |
| access_code | string | No | Access code. |

## Responses

Success (response_code: 1)

```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": "",
  "show_user_usage_policy": ""
}
```

Failure (response_code: 0)

```json
{
  "response_code": "0",
  "message": "User Registration failed as Login Type field missing",
  "status": "failure",
  "data": "",
  "show_user_usage_policy": ""
}
```

---

### `delete_user_account`

**File:** `api_delete_user_account.php`

**Method:** `POST`

**Description:** Deletes a user's account and removes their session from the
database. It requires a delete_account flag and user/session details for
validation.

## Parameters

| Name           | Type   | Required | Description                              |
|----------------|--------|----------|------------------------------------------|
| delete_account | bool   | Yes      | A flag to confirm account deletion.      |
| user_id        | int    | Yes      | The ID of the user to be deleted.        |
| session_token  | string | Yes      | The user's session token for validation. |
| user_lang      | string | No       | The user's language preference.          |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "message": "delete_success"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "message": "generic_error"
}
```

---

### `get_current_plan_summary`

**File:** `api_get_current_plan_summary.php`

**Method:** `POST`

**Description:** Retrieves a summary of a user's current subscription plan,
including details like plan type and end date.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | No       | The user's ID.                  |
| user_email        | string | No       | The user's email.               |
| user_key          | string | No       | A user key.                     |
| user_lang         | string | No       | The user's language preference. |
| client_ip_address | string | No       | The user's IP address.          |

## Responses

Success (response_code: 1)

```json
{
  "response_code": 1,
  "message": "success",
  "status": "success",
  "data": {
    "end_date": "2024-12-31 23:59:59",
    "plan_type": "Monthly Subscription",
    "price_paid": "499",
    "status": "Active"
  }
}
```

Failure (response_code: 0)

```json
{
  "response_code": 0,
  "message": "No active plan found",
  "status": "failure",
  "data": {}
}
```

---

### `get_current_share_permissions`

**File:** `api_get_current_share.php`

**Method:** `POST`

**Description:** Fetches the user's current sharing permissions for various
app features, such as a journal and mood tracker.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| user_lang         | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "message": "settings_loaded",
  "data": {
    "journal_shared": true,
    "mood_tracker_shared": false,
    "habit_tracker_shared": true,
    "anxiety_tracker_shared": false,
    "assessments_shared": true
  }
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "message": "user_missing"
}
```

---

### `get_random_quote`

**File:** `api_get_random_quote.php`

**Method:** `POST`

**Description:** Retrieves a random quote from the database.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_lang | string | No | The preferred language for the quote. |
| user_reference_id | string | No | The user's ID to fetch personalized quotes. |

## Responses

Success

```json
{
  "quote": "The only way to do great work is to love what you do.",
  "author": "Steve Jobs"
}
```

Failure

```json
{
  "quote": "",
  "author": ""
}
```

---

### `ignore_merge_account`

**File:** `api_ignore_merge_account.php`

**Method:** `POST`

**Description:** Ignores the flag to merge a user's accounts.

## Parameters

| Name    | Type   | Required | Description    |
|---------|--------|----------|----------------|
| user_id | string | Yes      | The user's ID. |

## Responses

Success (status: success)

```json
{
  "status": "success",
  "data": "Merge flag ignored for user"
}
```

Error (status: error)

```json
{
  "status": "error",
  "data": "Technical issue"
}
```

---

### `initiate_subscription_payment`

**File:** `api_initiate_subscription_payment.php`

**Method:** `POST`

**Description:** Initiates a subscription payment process, validates the
user and plan, creates a payment entry in the database, and returns the
necessary data for a Razorpay payment gateway.

## Parameters

| Name                  | Type   | Required | Description                           |
|-----------------------|--------|----------|---------------------------------------|
| user_reference_id     | int    | Yes      | The user's ID.                        |
| plan_type             | string | Yes      | The plan's category code.             |
| subscription_price    | string | No       | The subscription price.               |
| subscription_discount | int    | No       | The subscription discount percentage. |
| referral_code         | string | No       | A referral code to apply a discount.  |

## Responses

Success (code: 1)

```json
{  
"code": "1",  
"razorpay_json":
"{"key":"...","amount":1000,"currency":"INR","name":"The Able
Mind","description":"Monthly Plan","image":"...","prefill":{"name":"John
Doe","email":"john.doe@example.com","contact":"9876543210"},"notes":{"merchant_order_id":"...","category":"MONTHLY","payment_gst_fees":10,"payment_price_wo_fees":100,"price_id":123},"order_id":"...","theme":{"color":"#5F9EA0"},"send_sms_hash":true,"method":{"netbanking":true,"card":true,"wallet":true,"upi":true,"emi":false,"pay
later":false}}"  
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "Error initiating subscription."
}
```

---

### `send_registration_otp`

**File:** `api_send_registration_otp.php`

**Method:** `POST`

**Description:** Generates a one-time password (OTP) and sends it to a
user's email for registration purposes.

## Parameters

| Name       | Type   | Required | Description                     |
|------------|--------|----------|---------------------------------|
| user_email | string | Yes      | The user's email address.       |
| user_lang  | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "Error sending OTP."
}
```

---

### `show_plans`

**File:** `api_show_plans.php`

**Method:** `POST`

**Description:** Retrieves a list of available subscription plans for a
user, taking into account their location and any special offers.

## Parameters

| Name      | Type    | Required | Description                          |
|-----------|---------|----------|--------------------------------------|
| user_key  | string  | No       | The user's key.                      |
| user_lang | string  | No       | The user's language preference.      |
| new       | boolean | No       | A flag to request a new JSON format. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success",
  "data": [
    {
      "plan_id": 1,
      "plan_name": "Monthly Plan",
      "price": "499",
      "currency": "INR",
      "features": [
        "Feature 1",
        "Feature 2"
      ]
    }
  ]
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "error_retrieving_plans"
}
```

---

### `subscription_payment_confirmation`

**File:** `api_subscription_payment_confirmation.php`

**Method:** `POST`

**Description:** Confirms a payment made through the Razorpay gateway. It
verifies the payment signature, updates the database records for the
subscription and payment, and sends a confirmation email/notification to
the user.

## Parameters

| Name                | Type   | Required | Description                          |
|---------------------|--------|----------|--------------------------------------|
| razorpay_order_id   | string | Yes      | The order ID from Razorpay.          |
| razorpay_payment_id | string | Yes      | The payment ID from Razorpay.        |
| razorpay_signature  | string | Yes      | The payment signature from Razorpay. |
| user_lang           | string | No       | The user's language preference.      |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "plan_confirmation"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "payment_failed"
}
```

---

### `tam_connections_consent`

**File:** `api_tam_connections_consent_status.php`

**Method:** `POST`

**Description:** This API checks if a user has consented to the "TAM
Connections" feature. It returns the consent message if consent is
required.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| user_lang         | string | No       | The user's language preference. |

## Responses

Consent Required (code: 0)

```json
{
  "code": "0",
  "text": {
    "title": "Consent Title",
    "introduction_1": "Introduction 1",
    "introduction_2": "Introduction 2",
    "introduction_guidelines": "Guidelines",
    "consent_line_1": "Consent Line 1",
    "consent_line_2": "Consent Line 2",
    "consent_line_3": "Consent Line 3 with {XXXX} replacement",
    "consent_line_4": "Consent Line 4",
    "consent_line_5": "Consent Line 5",
    "consent_line_6": "Consent Line 6",
    "consent_line_7": "Consent Line 7",
    "consent_line_8": "",
    "consent_line_9": "",
    "consent_final": "Final consent text",
    "consent_text": "Accept",
    "cancel_text": "Decline"
  }
}
```

Consent Granted (code: 1)

```json
{
  "code": "1"
}
```

---

### `third_banner`

**File:** `api_third_banner.php`

**Method:** `POST`

**Description:** Retrieves information for a third banner, including details
on next appointments, Therapy Over Text (TOT) usage, and available
counselor slots.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The user's ID. |
| slots | int | No | The number of counselor slots to retrieve. Defaults to 2. |
| user_lang | string | No | The user's language preference. |

## Responses

Success (code: 1)

```json
{  
"code": "1",  
"next_appointment": {  
"date_full_display": "Monday, 25 Aug 2024",  
"date_display": "25 Aug 24",  
"time_start_display": "10:00 am",  
"time_end_display": "11:00 am",  
"counsellor": "Jane Doe"  
},  
"tot": {  
"daily_limit_message": "Words used: 150/500 today"  
},  
"next_slot": {  
"counsellor": "John Smith",  
"next_counsellor_slot": [  
{  
"slot_id": 1,  
"date": "2024-08-26",  
"time_start": "09:00",  
"time_end": "10:00"  
}
]
```
},  
"tam_library_url": "http://example.com/library",  
"tam_resource_center_url": "http://example.com/resources"  
}

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User id cannot be empty"
}
```

---

### `update_user_access_code`

**File:** `api_update_user_access_code.php`

**Method:** `POST`

**Description:** Updates a user's access code and plan information.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| access_code       | string | Yes      | The new access code.            |
| user_lang         | string | No       | The user's language preference. |

## Responses

Success (response_code: 1)

```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": {
    "code": "1",
    "message": "Access code updated successfully"
  }
}
```

Failure (response_code: 0)

```json
{
  "response_code": "0",
  "message": "Error",
  "status": "failure",
  "data": {
    "error": "Access code is invalid."
  }
}
```

---

### `update_user_information`

**File:** `api_update_user_information.php`

**Method:** `POST`

**Description:** Updates a user's profile information, including name,
email, phone, and guardian details.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| primary_login_key | string | Yes | The user's primary login key (email or phone). |
| user_reference_id | int | No | The user's ID. |
| user_name | string | No | User's name. |
| user_age | string | No | User's age. |
| user_sex | string | No | User's gender. |
| user_country | string | No | User country. |
| user_email | string | No | User email. |
| user_phone | string | No | User phone. |
| user_guardian_name | string | No | Guardian name. |
| user_guardian_email | string | No | Guardian email. |
| user_guardian_phone | string | No | Guardian phone. |
| user_guardian_relationship | string | No | Guardian relationship. |
| access_code | string | No | Access code. |
| employee_id | string | No | Employee ID. |
| user_lang | string | No | Language preference. |

## Responses

Success (status: 1)

```json
{
  "status": "1",
  "message": "success",
  "data": {
    "id": "123",
    "name": "Updated Name"
  }
}
```

Failure (status: -1)

```json
{
  "status": "-1",
  "message": "Error updating user information"
}
```

---

### `validate_referral_code`

**File:** `api_validate_referral_code.php`

**Method:** `POST`

**Description:** Validates a referral code against a user's subscription and
applies the corresponding discount if the code is valid.

## Parameters

| Name                       | Type   | Required | Description                     |
|----------------------------|--------|----------|---------------------------------|
| user_reference_id          | int    | Yes      | The user's ID.                  |
| referral_code              | string | Yes      | The referral code to validate.  |
| subscription_category_code | string | Yes      | The plan's category code.       |
| user_lang                  | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "discount": 10,
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "discount": "",
  "message": "error_code_inactive"
}
```

---

### `audio_transcribe`

**File:** `audio_transcribe.php`

**Method:** `POST`

**Description:** Transcribes an audio file into text using a third-party
service (Whisper API). It can also generate an audio response from the
transcribed text.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| file_url | string | Yes | The URL of the audio file. |
| generate_audio | string | Yes | "Y" to generate audio, "N" otherwise. |
| api_key | string | Yes | The API key for the transcription service. |
| user_lang_code | string | No | The user's language code. |
| user_gender | string | No | The user's gender ('M', 'F', or 'O') for voice generation. |
| return_type | string | No | "1" to return audio only, otherwise returns text and audio. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "text": "This is the transcribed text.",
  "language": "en",
  "language_full": "English",
  "audio": "http://example.com/audio/response.mp3"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "text": "voice_note_missing",
  "language": "not_detected",
  "language_full": "not_detected"
}
```

---

### `check_tam_connections_consent`

**File:** `check_tam_connections_consent.php`

**Method:** `POST`

**Description:** This API checks if a user has consented to the "TAM
Connections" feature. It returns the consent message if consent is
required.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| user_lang         | string | No       | The user's language preference. |

## Responses

Consent Required (code: 0)

```json
{
  "code": "0",
  "message": "Consent is required",
  "data": {
    "title": "Consent Title",
    "introduction_1": "Introduction 1",
    "introduction_2": "Introduction 2",
    "introduction_guidelines": "Guidelines",
    "consent_line_1": "Consent Line 1",
    "consent_line_2": "Consent Line 2",
    "consent_line_3": "Consent Line 3 with {XXXX} replacement",
    "consent_line_4": "Consent Line 4",
    "consent_line_5": "Consent Line 5",
    "consent_line_6": "Consent Line 6",
    "consent_line_7": "Consent Line 7",
    "consent_line_8": "",
    "consent_line_9": "",
    "consent_final": "Final consent text",
    "consent_text": "Accept",
    "cancel_text": "Decline"
  }
}
```

Consent Granted (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

---

### `generate_chat_summary`

**File:** `generate_chat_summary.php`

**Method:** `POST`

**Description:** Generates a summary for a specific chat session or for all
sessions requiring a summary. This is intended to be used as a cron job
or a background process.

## Parameters

| Name       | Type | Required | Description                           |
|------------|------|----------|---------------------------------------|
| session_id | int  | No       | The specific session ID to summarize. |

## Responses

Success

```json
{
  "close_summary": "The summary has been generated successfully."
}
```

Error

```json
{
  "close_summary": "Error while generating the summary"
}
```

---

### `get_complete_ysam_post`

**File:** `get_complete_ysam_post.php`

**Method:** `POST`

**Description:** Retrieves the full content of a single YSAM post, including
details about its author and any related information.

## Parameters

| Name              | Type   | Required | Description                             |
|-------------------|--------|----------|-----------------------------------------|
| user_reference_id | int    | Yes      | The ID of the user requesting the post. |
| ysam_id           | int    | Yes      | The ID of the YSAM post to retrieve.    |
| user_lang         | string | No       | The user's language preference.         |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "data": {
    "ysam_id": "123",
    "ysam_title": "My Story",
    "ysam_body": "The full content of the post...",
    "author_name": "Jane Doe",
    "category": "Anxiety"
  },
  "message": ""
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### `google_audio_transcribe`

**File:** `google_audio_transcribe.php`

**Method:** `POST`

**Description:** Transcribes an audio file into text using Google Cloud
Speech-to-Text.

## Parameters

| Name            | Type   | Required | Description                       |
|-----------------|--------|----------|-----------------------------------|
| audio_file_path | string | Yes      | The local path to the audio file. |
| user_lang       | string | No       | The user's language preference.   |
| api_key         | string | Yes      | The Google Cloud API key.         |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "transcription": "This is the transcribed text."
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "error": "Error processing audio file."
}
```

---

### `list_tam_user_conversations_dashboard`

**File:** `list_tam_user_conversations_dashboard.php`

**Method:** `POST`

**Description:** Retrieves a list of user conversations to display on a
dashboard. It can be filtered by a search query.

## Parameters

| Name              | Type   | Required | Description                             |
|-------------------|--------|----------|-----------------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                          |
| search            | string | No       | A search query to filter conversations. |
| user_lang         | string | No       | The user's language preference.         |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "",
  "data": [
    {
      "conversation_id": "1",
      "user_name": "User 1",
      "last_message": "Hello, how are you?",
      "timestamp": "2024-08-24 15:30:00"
    }
  ]
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### `list_ysam_posts`

**File:** `list_ysam_posts.php`

**Method:** `POST`

**Description:** Fetches a list of "Your Story and Mine" (YSAM) posts based
on various filters.

## Parameters

| Name            | Type    | Required | Description                             |
|-----------------|---------|----------|-----------------------------------------|
| page_number     | int     | No       | The page number for pagination.         |
| posts_per_page  | int     | No       | The number of posts to return per page. |
| ysam_user       | int     | No       | The user ID to filter posts by.         |
| hashtag         | string  | No       | A hashtag to filter posts by.           |
| category        | string  | No       | The category code to filter posts by.   |
| search_criteria | string  | No       | A search term.                          |
| published       | boolean | No       | Flag to filter for published posts.     |
| only_count      | boolean | No       | Flag to only return the count of posts. |
| user_id         | int     | No       | The current user's ID.                  |
| user_lang       | string  | No       | The user's language preference.         |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "",
  "data": [
    {
      "ysam_id": "1",
      "ysam_title": "First Post",
      "ysam_body_snippet": "This is a snippet of the first post...",
      "author_name": "Author 1",
      "created_at": "2024-08-24 15:00:00"
    }
  ]
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "count": 0,
  "message": "processing_error"
}
```

---

### `register_tam_connections_consent`

**File:** `register_tam_connections_consent.php`

**Method:** `POST`

**Description:** Records a user's consent to participate in the "TAM
Connections" feature, saving the consent message and other details to
the database.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| consent_string    | string | Yes      | The consent message string.     |
| user_lang         | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "Error registering consent"
}
```

---

### `tam_connections_block_user`

**File:** `tam_connections_block_user.php`

**Method:** `POST`

**Description:** Blocks another user from connecting or communicating with
the current user.

## Parameters

| Name                    | Type   | Required | Description                       |
|-------------------------|--------|----------|-----------------------------------|
| user_reference_id       | int    | Yes      | The current user's ID.            |
| user_reference_block_id | int    | Yes      | The ID of the user to be blocked. |
| user_lang               | string | No       | The user's language preference.   |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id is not specified"
}
```

---

### `tam_connections_send_message`

**File:** `tam_connections_send_message.php`

**Method:** `POST`

**Description:** Sends a message from one user to another within the "TAM
Connections" feature. It handles message validation, language detection,
and translation.

## Parameters

| Name                   | Type   | Required | Description                       |
|------------------------|--------|----------|-----------------------------------|
| user_reference_id      | int    | Yes      | The sender's user ID.             |
| recipient_reference_id | int    | Yes      | The recipient's user ID.          |
| message                | string | Yes      | The message content.              |
| type                   | string | Yes      | Message type, e.g., 'T' for text. |
| user_lang              | string | No       | The user's language preference.   |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "text": "Hello, how are you?",
  "detected_language": "en",
  "special_message": "",
  "sender_date_display": "24 Aug 2024 03:30 pm",
  "recipient_text": "Hallo, wie geht es Ihnen?",
  "receipient_date_display": "24 Aug 2024 03:30 pm"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "text": "User not authorized"
}
```

---

### `translate`

**File:** `translate.php`

**Method:** `POST`

**Description:** Translates text from one language to another using the
Google Translate API. It can also save the translated message to the
database.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| text_to_translate | string | Yes | The text to be translated. |
| target_language | string | Yes | The two-letter code for the target language. |
| message_id | int | No | The ID of the message to update in the database. |
| from_counsellor | bool | No | A flag indicating if the message is from a counselor. |
| testing | bool | No | A flag for testing purposes. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "text": "Translated text here.",
  "detected_language": "en",
  "detected_language_full": "English"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "text": "Translation Error : The text to translate cannot be empty."
}
```

---

### `upload_audio`

**File:** `upload_audio.php`

**Method:** `POST`

**Description:** Handles the upload of an audio file, either from a file
input or from a Base64-encoded string.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| type | string | Yes | The upload type, "R" for recorded audio (Base64), or a different value for a file upload. |
| recordedAudio | string | Conditional | Required if type is 'R', the Base64-encoded audio data. |
| audioFile | file | Conditional | Required if type is not 'R', the audio file to upload. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "message": "File is successfully uploaded.",
  "file": "/path/to/uploaded_file.wav"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "message": "No audio file to upload.",
  "file": ""
}
```

---

### `user_conversations_load`

**File:** `user_conversations_load.php`

**Method:** `POST`

**Description:** Loads a user's conversation history based on their
connection type and room name.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The user's ID. |
| connection_type | string | Yes | The type of conversation, 'S' for a single user or 'G' for a group. |
| room_name | string | Yes | The name of the conversation room. |
| user_lang | string | No | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "",
  "data": {
    "room_name": "ysam_conv_123_456",
    "chat_history": [
      {
        "id": "1",
        "sender_id": "123",
        "message": "Hello!",
        "timestamp": "2024-08-24 15:00:00"
      }
    ]
  }
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id or Connection type is not specified"
}
```

---

### `ysam_add_update_article`

**File:** `ysam_add_update_article.php`

**Method:** `POST`

**Description:** Creates a new YSAM post or updates an existing one with a
title, body, hashtags, and category.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| ysam_update | string | Yes | 'Y' to update an existing article, or 'N' to add a new one. |
| user_reference_id | int | Yes | The user's ID. |
| ysam_id | int | Conditional | Required if updating, the ID of the post to update. |
| ysam_body | string | Yes | The main content of the post. |
| ysam_hashtags | string | No | A comma-separated list of hashtags. |
| ysam_title | string | Yes | The title of the post. |
| ysam_user_name | string | Yes | The name of the user posting the article. |
| ysam_category_id | int | Yes | The category ID of the article. |
| ysam_narration | string | No | A narration for the article. |
| user_lang | string | No | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "text": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "text": "User Id not specified."
}
```

---

### `ysam_check_usage_tam_connections`

**File:** `ysam_check_usage_tam_connections.php`

**Method:** `POST`

**Description:** Checks if a user has exceeded their daily message limit for
"TAM Connections".

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| user_lang         | string | No       | The user's language preference. |

## Responses

Within Limit (code: 1)

```json
{
  "code": "1",
  "message": "You have 5 messages remaining for today."
}
```

Limit Exceeded (code: 0)

```json
{
  "code": "0",
  "message": "Message limit exceeded. Please try again tomorrow."
}
```

---

### `ysam_connect_to_user`

**File:** `ysam_connect_to_user.php`

**Method:** `POST`

**Description:** Updates the connection status between two users within the
YSAM feature.

## Parameters

| Name                | Type   | Required | Description                         |
|---------------------|--------|----------|-------------------------------------|
| user_reference_id   | int    | Yes      | The current user's ID.              |
| author_reference_id | int    | Yes      | The ID of the user to connect with. |
| user_lang           | string | No       | The user's language preference.     |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id or Author Id not specified"
}
```

---

### `ysam_follow_user`

**File:** `ysam_follow_user.php`

**Method:** `POST`

**Description:** Allows a user to follow or unfollow another user in the
YSAM section.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The current user's ID. |
| user_to_follow | int | Yes | The ID of the user to follow/unfollow. |
| follow | string | No | 'Y' to follow, 'N' to unfollow. Defaults to 'Y'. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### `ysam_get_all_categories`

**File:** `ysam_get_all_categories.php`

**Method:** `POST`

**Description:** Retrieves a list of all available YSAM post categories.

## Parameters

| Name      | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| user_lang | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "",
  "data": [
    {
      "category_id": "1",
      "category_name": "Anxiety",
      "category_code": "anx"
    }
  ]
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### `ysam_initialize_form`

**File:** `ysam_initialize_form.php`

**Method:** `POST`

**Description:** Retrieves data required to initialize the YSAM post
creation form.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| user_reference_id | int    | Yes      | The user's ID.                  |
| user_lang         | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "",
  "data": {
    "user_name": "Jane Doe",
    "user_picture": "http://example.com/profile.jpg",
    "categories": [
      {
        "category_id": "1",
        "category_name": "Anxiety"
      }
    ]
  }
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### `ysam_report_user_post`

**File:** `ysam_report_user_post.php`

**Method:** `POST`

**Description:** Allows a user to report a YSAM post for inappropriate
content or other reasons.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The user's ID. |
| ysam_id | int | Yes | The ID of the post to report. |
| previous_reported | string | No | 'Y' if previously reported, 'N' otherwise. Defaults to 'N'. |
| reason | string | Yes | The reason for reporting the post. |
| user_lang | string | No | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": "1",
  "message": "success"
}
```

Failure (code: 0)

```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### `ysam_review_message`

**File:** `ysam_review_message.php`

**Method:** `POST`

**Description:** Reviews the content of a message for inappropriate or
harmful content using an external AI service before it is sent.

## Parameters

| Name              | Type   | Required | Description                     |
|-------------------|--------|----------|---------------------------------|
| text_to_translate | string | Yes      | The message to be reviewed.     |
| user_lang         | string | No       | The user's language preference. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "detected_language": "en",
  "link": "",
  "timestamp": "2024-08-24 15:00:00"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "text": "The message has been flagged as inappropriate.",
  "timestamp": "Sat, 24 Aug 2024 (03:00 pm)"
}
```

---

### `ysam_translate`

**File:** `ysam_translate.php`

**Method:** `POST`

**Description:** Translates a message from a user in the YSAM feature,
handling language detection and providing a translated message for the
recipient.

## Parameters

| Name          | Type   | Required | Description                     |
|---------------|--------|----------|---------------------------------|
| message       | string | Yes      | The message to translate.       |
| sender        | int    | Yes      | The sender's ID.                |
| recipient     | int    | Yes      | The recipient's ID.             |
| user_lang     | string | No       | The user's language preference. |
| user_timezone | string | No       | The user's timezone.            |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "text": "Translated message text",
  "detected_language": "en",
  "detected_language_full": "English",
  "timestamp_display": "Sat, 24 Aug 2024 (03:00 pm)"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "text": "Error : Could not detect language",
  "timestamp_display": "Sat, 24 Aug 2024 (03:00 pm)"
}
```

---

### `share_private_app_data`

**File:** `api_share_with_counsellor.php`

**Method:** `POST`

**Description:** Updates the user's sharing permissions for private app
features. The endpoint securely updates the database with the user's
preferences.

## Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| user_reference_id | int | Yes | The user's ID. |
| user_lang | string | No | The user's language preference. |
| journal_shared | int | No | 1 to share the journal, 0 otherwise. |
| mood_tracker_shared | int | No | 1 to share the mood tracker, 0 otherwise. |
| habit_tracker_shared | int | No | 1 to share the habit tracker, 0 otherwise. |
| anxiety_tracker_shared | int | No | 1 to share the anxiety tracker, 0 otherwise. |
| assessments_shared | int | No | 1 to share the assessments, 0 otherwise. |

## Responses

Success (code: 1)

```json
{
  "code": 1,
  "message": "settings_updated"
}
```

Failure (code: 0)

```json
{
  "code": 0,
  "message": "user_missing"
}
```
