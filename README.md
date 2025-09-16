# The Able Mind API Documentation
This document provides structured documentation for the available APIs.
Each API includes details such as purpose, request structure, parameters, authentication, and example JSON responses.

## Base URL
<https://(domain)/api.php/v1/>

## Authentication
All API endpoints require Bearer Token authentication.
Header Format:
Authorization: Bearer [BEARER_TOKEN]

## Endpoints Index
- [chat_summary_fb15](#chat_summary_fb15)
- [check_chat_usage](#check_chat_usage)
- [check_counsellor_access](#check_counsellor_access)
- [check_maintenance](#check_maintenance)
- [check_merge_accounts](#check_merge_accounts)
- [check_unique_user_name](#check_unique_user_name)
- [check_valid_email](#check_valid_email)
- [confirm_merge_accounts](#confirm_merge_accounts)
- [create_audio_conference](#create_audio_conference)
- [create_user](#create_user)
- [delete_user_account](#delete_user_account)
- [get_current_plan_summary](#get_current_plan_summary)
- [get_current_share_permissions](#get_current_share_permissions)
- [get_random_quote](#get_random_quote)
- [generate_faq](#generate_faq)
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
 

### API Endpoints

### chat_summary_fb15
- **File**: api_chat_summary_fb15.php.
- **Description**: Generates a summary of past chats for a user using an external AI service. It fetches chat history, generates a summary if one doesn't exist, and returns the chat history with summaries. The generated summary includes mood analysis and identifies a central topic.
- **Method**: POST
- **Parameters**: 
  - user_id (required, int): The user's ID.
  - number_of_chats (optional, int): The number of chat summaries to retrieve. Defaults to a minimum message count for summary generation.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "data": [
    {
      "date": "21 Dec 2023 10:30 am (A few hours ago)",
      "counsellor": "Counsellor Name",
      "chat_topic": "Romantic Relationship Issues",
      "feedback_star": "4",
      "summary": "This is a detailed summary of the conversation.<br><b>Close Chat Remarks:</b> A remark from the counsellor."
    }
```
  ]

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User id does not exist"
}
```

---

### check_chat_usage
- **File**: api_check_chat_usage.php.
- **Description**: Checks a user's usage against their subscribed chat plan (e.g., Feel Better in 15 or Therapy Over Text) and determines if they can initiate a new chat.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - usage_check_type (required, string): The type of chat to check, either "fb15" or another type.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "minutes_available": 15,
  "words_available": "",
  "show_buttons": {
    "count": 0,
    "buttons": []
  }
```

### }
  - Failure (code: 0):
```json
{
  "code": 0,
  "text": "Your usage for Feel Better in 15 has been exceeded. Please consider Therapy Over Text as an alternative.",
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
```
    ]
  }

### }

---

### check_counsellor_access
- **File**: api_check_counsellor_access.php.
- **Description**: Authenticates a user with a Google theablemind.com domain and provisions them as a counselor, admin, or ops lead if they do not exist in the system.
- **Method**: POST
- **Parameters**: 
  - access_token (required, string): The Google access token.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
[
  {
    "code": 1,
    "type": ["A", "C"],
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
```

### ]
  - Failure (code: 0):
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
    "message": "Access Denied. This feature is only available to employees of The Able Mind."
  }
```

### ]

---

### check_maintenance
- **File**: api_check_maintenance.php.
- **Description**: Checks for any ongoing system maintenance and informs the user if specific app features are disabled.
- **Method**: POST
- **Parameters**: 
  - user_lang (optional, string): The user's language preference.
  - timezone (optional, string): The user's timezone.
- **JSON Output**: 
  - Success (returns a status object):
```json
{
  "UPDATION_INPROGRESS": true,
  "MESSAGE_TITLE": "Maintenance Alert",
  "MESSAGE": "We are undergoing a scheduled maintenance. The service will be back on Saturday, 24 Aug 2024 [05:30 am IST]",
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
  - No Maintenance (returns a default object):
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

### check_merge_accounts
- **File**: api_check_merge_accounts.php.
- **Description**: Checks if a user has multiple accounts that should be merged, typically based on email or phone number.
- **Method**: POST
- **Parameters**: 
  - user_email (optional, string): The user's email address.
  - user_phone (optional, string): The user's phone number.
  - login_type (optional, string): The login type, defaults to 'E' for email.
- **JSON Output**: 
  - Success (status: success):
```json
{
  "status": "success",
  "data": {
    "show_merge_screen": true,
    "primary_account_name": "User Name",
    "secondary_account_name": "Duplicate User",
    "message": "We have found a duplicate account for you. Would you like to merge them?"
  }
```

### }
  - Error (status: error):
```json
{
  "status": "error",
  "data": "A technical issue has occurred."
}
```

---

### check_unique_user_name
- **File**: api_check_unique_user_name.php.
- **Description**: Validates if a user name is unique in the system.
- **Method**: POST
- **Parameters**: 
  - user_name (required, string): The user name to validate.
  - login_key (optional, string): The user's login key.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (response_code: 1):
```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": ""
}
```
  - Failure (response_code: 0):
```json
{
  "response_code": "0",
  "message": "User Name is not unique",
  "status": "failure",
  "data": ""
}
```

---

### check_valid_email
- **File**: api_check_valid_email.php.
- **Description**: Validates an email address by checking for a valid format, existing MX records, and whether the domain is a known disposable email provider.
- **Method**: POST
- **Parameters**: 
  - user_email (required, string): The email address to validate.
- **JSON Output**: 
  - Success (response_code: 0):
```json
{
  "response_code": "0",
  "message": "Email id provided is valid.",
  "status": "success",
  "data": ""
}
```
  - Failure (response_code: -1):
```json
{
  "response_code": "-1",
  "message": "Email domain (mailinator.com) is invalid.",
  "status": "failure",
  "data": ""
}
```

---

### confirm_merge_accounts
- **File**: api_confirm_merge_accounts.php.
- **Description**: Confirms and completes the merging of two user accounts by linking two different user profiles into a single account.
- **Note**: The file api_confirm_merge_accounts.php was not provided, so the following documentation is based on common API patterns for this type of function.
- **Method**: POST
- **Parameters**: 
  - user_id (required, int): The ID of the user initiating the merge.
  - duplicate_id (required, int): The ID of the account to merge into the primary account.
  - login_type (required, string): The primary login type for the merged account ('E' for email or 'P' for phone).
- **JSON Output**: 
  - Success (status: success):
```json
{
  "status": "success",
  "data": "Accounts successfully merged"
}
```
  - Error (status: error):
```json
{
  "status": "error",
  "data": "Error merging accounts"
}
```

---

### create_audio_conference
- **File**: api_create_audio_conference.php.
- **Description**: Creates or updates an audio conference.
- **Method**: POST
- **Parameters**: 
  - conference_id (required, string): Unique ID for the conference. If it exists, all data will be updated; otherwise, it will be inserted.
  - conference_duration (optional, int): Duration of the session in minutes. Defaults to 60.
  - conference_start_time (required, string): Conference start date and time in "Y-m-d H:i:s" format.
  - conference_counsellor_keys (required, string): Comma-separated list of counsellor keys.
  - conference_title (required, string): Title of the conference.
  - max_simultaneous_connections (optional, int): Maximum number of simultaneous connections. Defaults to 20.
- **JSON Output**: 
  - Success (response_code: 1):
```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": ""
}
```
  - Failure (response_code: 0):
```json
{
  "response_code": "0",
  "message": "Error Creating / Updating Conference",
  "status": "failure",
  "data": ""
}
```

---

### create_user
- **File**: api_create_user.php.
- **Description**: Registers a new user account with various details such as name, email, phone, and demographic information.
- **Method**: POST
- **Parameters**: 
  - login_type (required, string): 'E' for email or 'P' for phone.
  - user_name (required, string): User's name.
  - user_email (required if login_type is 'E', string): User's email.
  - user_phone_number (required if login_type is 'P', string): User's phone number.
  - user_password (optional, string): The user's password.
  - user_age (optional, string): User's age.
  - user_sex (optional, string): User's gender.
  - user_lang (optional, string): The user's language preference.
  - external_login, user_guardian_name, user_guardian_phone, user_guardian_relationship, client_ip_address, device_id, access_code: All are optional strings.
- **JSON Output**: 
  - Success (response_code: 1):
```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": "",
  "show_user_usage_policy": ""
}
```
  - Failure (response_code: 0):
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

### delete_user_account
- **File**: api_delete_user_account.php.
- **Description**: Deletes a user's account and removes their session from the database. It requires a delete_account flag and user/session details for validation.
- **Method**: POST
- **Parameters**: 
  - delete_account (required, bool): A flag to confirm account deletion.
  - user_id (required, int): The ID of the user to be deleted.
  - session_token (required, string): The user's session token for validation.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "message": "delete_success"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "message": "generic_error"
}
```

---

### get_current_plan_summary
- **File**: api_get_current_plan_summary.php.
- **Description**: Retrieves a summary of a user's current subscription plan, including details like plan type and end date.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (optional, int): The user's ID.
  - user_email (optional, string): The user's email.
  - user_key (optional, string): A user key.
  - user_lang (optional, string): The user's language preference.
  - client_ip_address (optional, string): The user's IP address.
- **JSON Output**: 
  - Success (response_code: 1):
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
```

### }
  - Failure (response_code: 0):
```json
{
  "response_code": 0,
  "message": "No active plan found",
  "status": "failure",
  "data": {}
}
```

---

### get_current_share_permissions
- **File**: api_get_current_share.php.
- **Description**: Fetches the user's current sharing permissions for various app features, such as a journal and mood tracker.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```

### }
  - Failure (code: 0):
```json
{
  "code": 0,
  "message": "user_missing"
}
```

---

### get_random_quote
- **File**: api_get_random_quote.php.
- **Description**: Retrieves a random quote from the database.
- **Method**: POST
- **Parameters**: 
  - user_lang (optional, string): The preferred language for the quote.
  - user_reference_id (optional, string): The user's ID to fetch personalized quotes.
- **JSON Output**: 
  - Success:
```json
{
  "quote": "The only way to do great work is to love what you do.",
  "author": "Steve Jobs"
}
```
  - Failure:
```json
{
  "quote": "",
  "author": ""
}
```

---

### **generate_faq**

-   **File**: `api_generate_faq.php`.
-   **Description**: Retrieves a list of Frequently Asked Questions (FAQs) localized in the user's preferred language. The API dynamically inserts user-relevant information, such as the default trial period and a list of available subscription plans, directly into the FAQ responses.
-   **Method**: `POST`
-   **Parameters**:
    -   `user_reference_id` (required, int): The user's ID.
    -   `user_lang` (optional, string): The two-letter language code for localization. Defaults to 'en'.
-   **JSON Output**:
    -   Success (code: 1):

        ```json
        {
          "code": 1,
          "message": "",
          "data": [
            {
              "heading": "General Questions",
              "items": [
                {
                  "question_id": "1",
                  "question": "Is there a free trial?",
                  "response": "Yes, we offer a free trial of 3 days for new users to explore our services."
                },
                {
                  "question_id": "2",
                  "question": "What happens after my trial ends?",
                  "response": "After the trial, you can choose from one of our subscription plans to continue."
                }
              ]
            },
            {
              "heading": "Subscription",
              "items": [
                {
                  "question_id": "3",
                  "question": "What plans do you offer?",
                  "response": "We offer the following plans: <ul><li><b>Monthly Plan</b> - Billed every month.</li><li><b>Quarterly Plan</b> - Billed every 3 months.</li></ul>"
                }
              ]
            }
          ]
        }
        ```

    -   Failure (code: 0):

        ```json
        {
          "code": 0,
          "message": "User id cannot be empty",
          "data": []
        }
        ```


---

### ignore_merge_account
- **File**: api_ignore_merge_account.php.
- **Description**: Ignores the flag to merge a user's accounts.
- **Method**: POST
- **Parameters**: 
  - user_id (required, string): The user's ID.
- **JSON Output**: 
  - Success (status: success):
```json
{
  "status": "success",
  "data": "Merge flag ignored for user"
}
```
  - Error (status: error):
```json
{
  "status": "error",
  "data": "Technical issue"
}
```

---

### initiate_subscription_payment
- **File**: api_initiate_subscription_payment.php.
- **Description**: Initiates a subscription payment process, validates the user and plan, creates a payment entry in the database, and returns the necessary data for a Razorpay payment gateway.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - plan_type (required, string): The plan's category code.
  - subscription_price (optional, string): The subscription price.
  - subscription_discount (optional, int): The subscription discount percentage.
  - referral_code (optional, string): A referral code to apply a discount.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "razorpay_json": "{\"key\":\"...\",\"amount\":1000,\"currency\":\"INR\",\"name\":\"The Able Mind\",\"description\":\"Monthly Plan\",\"image\":\"...\",\"prefill\":{\"name\":\"John Doe\",\"email\":\"john.doe@example.com\",\"contact\":\"9876543210\"},\"notes\":{\"merchant_order_id\":\"...\",\"category\":\"MONTHLY\",\"payment_gst_fees\":10,\"payment_price_wo_fees\":100,\"price_id\":123},\"order_id\":\"...\",\"theme\":{\"color\":\"#5F9EA0\"},\"send_sms_hash\":true,\"method\":{\"netbanking\":true,\"card\":true,\"wallet\":true,\"upi\":true,\"emi\":false,\"pay later\":false}}"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "Error initiating subscription."
}
```

---

### send_registration_otp
- **File**: api_send_registration_otp.php.
- **Description**: Generates a one-time password (OTP) and sends it to a user's email for registration purposes.
- **Method**: POST
- **Parameters**: 
  - user_email (required, string): The user's email address.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "Error sending OTP."
}
```

---

### show_plans
- **File**: api_show_plans.php.
- **Description**: Retrieves a list of available subscription plans for a user, taking into account their location and any special offers.
- **Method**: POST
- **Parameters**: 
  - user_key (optional, string): The user's key.
  - user_lang (optional, string): The user's language preference.
  - new (optional, boolean): A flag to request a new JSON format.
- **JSON Output**: 
  - Success (code: 1):
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
      "features": ["Feature 1", "Feature 2"]
    }
```
  ]

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "error_retrieving_plans"
}
```

---

### subscription_payment_confirmation
- **File**: api_subscription_payment_confirmation.php.
- **Description**: Confirms a payment made through the Razorpay gateway. It verifies the payment signature, updates the database records for the subscription and payment, and sends a confirmation email/notification to the user.
- **Method**: POST
- **Parameters**: 
  - razorpay_order_id (required, string): The order ID from Razorpay.
  - razorpay_payment_id (required, string): The payment ID from Razorpay.
  - razorpay_signature (required, string): The payment signature from Razorpay.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "plan_confirmation"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "payment_failed"
}
```

---

### tam_connections_consent
- **File**: api_tam_connections_consent_status.php.
- **Description**: This API checks if a user has consented to the "TAM Connections" feature. It returns the consent message if consent is required.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Consent Required (code: 0):
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
```

### }
  - Consent Granted (code: 1):
```json
{
  "code": "1"
}
```

---

### third_banner
- **File**: api_third_banner.php.
- **Description**: Retrieves information for a third banner, including details on next appointments, Therapy Over Text (TOT) usage, and available counselor slots.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - slots (optional, int): The number of counselor slots to retrieve. Defaults to 2.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
    ]
  },
  "tam_library_url": "<http://example.com/library",>
  "tam_resource_center_url": "<http://example.com/resources">

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User id cannot be empty"
}
```

---

### update_user_access_code
- **File**: api_update_user_access_code.php.
- **Description**: Updates a user's access code and plan information.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - access_code (required, string): The new access code.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (response_code: 1):
```json
{
  "response_code": "1",
  "message": "success",
  "status": "success",
  "data": {
    "code": "1",
    "message": "Access code updated successfully"
  }
```

### }
  - Failure (response_code: 0):
```json
{
  "response_code": "0",
  "message": "Error",
  "status": "failure",
  "data": {
    "error": "Access code is invalid."
  }
```

### }

---

### update_user_information
- **File**: api_update_user_information.php.
- **Description**: Updates a user's profile information, including name, email, phone, and guardian details.
- **Method**: POST
- **Parameters**: 
  - primary_login_key (required, string): The user's primary login key (email or phone).
  - user_reference_id (optional, int): The user's ID.
  - user_name, user_age, user_sex, user_country, user_email, user_phone, user_guardian_name, user_guardian_email, user_guardian_phone, user_guardian_relationship, access_code, employee_id, user_lang: All are optional strings.
- **JSON Output**: 
  - Success (status: 1):
```json
{
  "status": "1",
  "message": "success",
  "data": {
    "id": "123",
    "name": "Updated Name"
  }
```

### }
  - Failure (status: -1):
```json
{
  "status": "-1",
  "message": "Error updating user information"
}
```

---

### validate_referral_code
- **File**: api_validate_referral_code.php.
- **Description**: Validates a referral code against a user's subscription and applies the corresponding discount if the code is valid.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - referral_code (required, string): The referral code to validate.
  - subscription_category_code (required, string): The plan's category code.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "discount": 10,
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "discount": "",
  "message": "error_code_inactive"
}
```

---

### audio_transcribe
- **File**: audio_transcribe.php.
- **Description**: Transcribes an audio file into text using a third-party service (Whisper API). It can also generate an audio response from the transcribed text.
- **Method**: POST
- **Parameters**: 
  - file_url (required, string): The URL of the audio file.
  - generate_audio (required, string): "Y" to generate audio, "N" otherwise.
  - api_key (required, string): The API key for the transcription service.
  - user_lang_code (optional, string): The user's language code.
  - user_gender (optional, string): The user's gender ('M', 'F', or 'O') for voice generation.
  - return_type (optional, string): "1" to return audio only, otherwise returns text and audio.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "text": "This is the transcribed text.",
  "language": "en",
  "language_full": "English",
  "audio": "http://example.com/audio/response.mp3"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "text": "voice_note_missing",
  "language": "not_detected",
  "language_full": "not_detected"
}
```

---

### check_tam_connections_consent
- **File**: api_tam_connections_consent_status.php.
- **Description**: This API checks if a user has consented to the "TAM Connections" feature. It returns the consent message if consent is required.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Consent Required (code: 0):
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
```

### }
  - Consent Granted (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```

---

### generate_chat_summary
- **File**: generate_chat_summary.php.
- **Description**: Generates a summary for a specific chat session or for all sessions requiring a summary. This is intended to be used as a cron job or a background process.
- **Method**: POST
- **Parameters**: 
  - session_id (optional, int): The specific session ID to summarize.
- **JSON Output**: 
  - Success:
```json
{
  "close_summary": "The summary has been generated successfully."
}
```
  - Error:
```json
{
  "close_summary": "Error while generating the summary"
}
```

---

### get_complete_ysam_post
- **File**: get_complete_ysam_post.php.
- **Description**: Retrieves the full content of a single YSAM post, including details about its author and any related information.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The ID of the user requesting the post.
  - ysam_id (required, int): The ID of the YSAM post to retrieve.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### google_audio_transcribe
- **File**: google_audio_transcribe.php.
- **Description**: Transcribes an audio file into text using Google Cloud Speech-to-Text.
- **Method**: POST
- **Parameters**: 
  - audio_file_path (required, string): The local path to the audio file.
  - user_lang (optional, string): The user's language preference.
  - api_key (required, string): The Google Cloud API key.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "transcription": "This is the transcribed text."
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "error": "Error processing audio file."
}
```

---

### list_tam_user_conversations_dashboard
- **File**: list_tam_user_conversations_dashboard.php.
- **Description**: Retrieves a list of user conversations to display on a dashboard. It can be filtered by a search query.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - search (optional, string): A search query to filter conversations.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
  ]

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### list_ysam_posts
- **File**: list_ysam_posts.php.
- **Description**: Fetches a list of "Your Story and Mine" (YSAM) posts based on various filters.
- **Method**: POST
- **Parameters**: 
  - page_number (optional, int): The page number for pagination.
  - posts_per_page (optional, int): The number of posts to return per page.
  - ysam_user (optional, int): The user ID to filter posts by.
  - hashtag (optional, string): A hashtag to filter posts by.
  - category (optional, string): The category code to filter posts by.
  - search_criteria (optional, string): A search term.
  - published (optional, boolean): Flag to filter for published posts.
  - only_count (optional, boolean): Flag to only return the count of posts.
  - user_id (optional, int): The current user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
  ]

### }
  - Failure (code: 0):
```json
{
  "code": 0,
  "count": 0,
  "message": "processing_error"
}
```

---

### register_tam_connections_consent
- **File**: api_tam_connections_consent_register.php.
- **Description**: Records a user's consent to participate in the "TAM Connections" feature, saving the consent message and other details to the database.
- **Method**: POST
- **Parameters**: 
  - user_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "text": "consent_registered"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "text": "generic_error"
}
```

---

### tam_connections_block_user
- **File**: tam_connections_block_user.php.
- **Description**: Blocks another user from connecting or communicating with the current user.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The current user's ID.
  - user_reference_block_id (required, int): The ID of the user to be blocked.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id is not specified"
}
```

---

### tam_connections_send_message
- **File**: tam_connections_send_message.php.
- **Description**: Sends a message from one user to another within the "TAM Connections" feature. It handles message validation, language detection, and translation.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The sender's user ID.
  - recipient_reference_id (required, int): The recipient's user ID.
  - message (required, string): The message content.
  - type (required, string): Message type, e.g., 'T' for text.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
  - Failure (code: 0):
```json
{
  "code": "0",
  "text": "User not authorized"
}
```

---

### translate
- **File**: translate.php.
- **Description**: Translates text from one language to another using the Google Translate API. It can also save the translated message to the database.
- **Method**: POST
- **Parameters**: 
  - text_to_translate (required, string): The text to be translated.
  - target_language (required, string): The two-letter code for the target language.
  - message_id (optional, int): The ID of the message to update in the database.
  - from_counsellor (optional, bool): A flag indicating if the message is from a counselor.
  - testing (optional, bool): A flag for testing purposes.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "text": "Translated text here.",
  "detected_language": "en",
  "detected_language_full": "English"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "text": "Translation Error : The text to translate cannot be empty."
}
```

---

### upload_audio
- **File**: upload_audio.php.
- **Description**: Handles the upload of an audio file, either from a file input or from a Base64-encoded string.
- **Method**: POST
- **Parameters**: 
  - type (required, string): The upload type, "R" for recorded audio (Base64), or a different value for a file upload.
  - recordedAudio (required if type is 'R', string): The Base64-encoded audio data.
  - audioFile (required if type is not 'R', file): The audio file to upload.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "message": "File is successfully uploaded.",
  "file": "/path/to/uploaded_file.wav"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "message": "No audio file to upload.",
  "file": ""
}
```

---

### user_conversations_load
- **File**: user_conversations_load.php.
- **Description**: Loads a user's conversation history based on their connection type and room name.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - connection_type (required, string): The type of conversation, 'S' for a single user or 'G' for a group.
  - room_name (required, string): The name of the conversation room.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
    ]
  }

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id or Connection type is not specified"
}
```

---

### ysam_add_update_article
- **File**: ysam_add_update_article.php.
- **Description**: Creates a new YSAM post or updates an existing one with a title, body, hashtags, and category.
- **Method**: POST
- **Parameters**: 
  - ysam_update (required, string): 'Y' to update an existing article, or 'N' to add a new one.
  - user_reference_id (required, int): The user's ID.
  - ysam_id (required if updating, int): The ID of the post to update.
  - ysam_body (required, string): The main content of the post.
  - ysam_hashtags (optional, string): A comma-separated list of hashtags.
  - ysam_title (required, string): The title of the post.
  - ysam_user_name (required, string): The name of the user posting the article.
  - ysam_category_id (required, int): The category ID of the article.
  - ysam_narration (optional, string): A narration for the article.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "text": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "text": "User Id not specified."
}
```

---

### ysam_check_usage_tam_connections
- **File**: ysam_check_usage_tam_connections.php.
- **Description**: Checks if a user has exceeded their daily message limit for "TAM Connections".
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Within Limit (code: 1):
```json
{
  "code": "1",
  "message": "You have 5 messages remaining for today."
}
```
  - Limit Exceeded (code: 0):
```json
{
  "code": "0",
  "message": "Message limit exceeded. Please try again tomorrow."
}
```

---

### ysam_connect_to_user
- **File**: ysam_connect_to_user.php.
- **Description**: Updates the connection status between two users within the YSAM feature.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The current user's ID.
  - author_reference_id (required, int): The ID of the user to connect with.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id or Author Id not specified"
}
```

---

### ysam_follow_user
- **File**: ysam_follow_user.php.
- **Description**: Allows a user to follow or unfollow another user in the YSAM section.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The current user's ID.
  - user_to_follow (required, int): The ID of the user to follow/unfollow.
  - follow (optional, string): 'Y' to follow, 'N' to unfollow. Defaults to 'Y'.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### ysam_get_all_categories
- **File**: ysam_get_all_categories.php.
- **Description**: Retrieves a list of all available YSAM post categories.
- **Method**: POST
- **Parameters**: 
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
  ]

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "technical_issue"
}
```

---

### ysam_initialize_form
- **File**: ysam_initialize_form.php.
- **Description**: Retrieves data required to initialize the YSAM post creation form.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
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
```
    ]
  }

### }
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### ysam_report_user_post
- **File**: ysam_report_user_post.php.
- **Description**: Allows a user to report a YSAM post for inappropriate content or other reasons.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - ysam_id (required, int): The ID of the post to report.
  - previous_reported (optional, string): 'Y' if previously reported, 'N' otherwise. Defaults to 'N'.
  - reason (required, string): The reason for reporting the post.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": "1",
  "message": "success"
}
```
  - Failure (code: 0):
```json
{
  "code": "0",
  "message": "User Id not specified"
}
```

---

### ysam_review_message
- **File**: ysam_review_message.php.
- **Description**: Reviews the content of a message for inappropriate or harmful content using an external AI service before it is sent.
- **Method**: POST
- **Parameters**: 
  - text_to_translate (required, string): The message to be reviewed.
  - user_lang (optional, string): The user's language preference.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "detected_language": "en",
  "link": "",
  "timestamp": "2024-08-24 15:00:00"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "text": "The message has been flagged as inappropriate.",
  "timestamp": "Sat, 24 Aug 2024 (03:00 pm)"
}
```

---

### ysam_translate
- **File**: ysam_translate.php.
- **Description**: Translates a message from a user in the YSAM feature, handling language detection and providing a translated message for the recipient.
- **Method**: POST
- **Parameters**: 
  - message (required, string): The message to translate.
  - sender (required, int): The sender's ID.
  - recipient (required, int): The recipient's ID.
  - user_lang (optional, string): The user's language preference.
  - user_timezone (optional, string): The user's timezone.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "text": "Translated message text",
  "detected_language": "en",
  "detected_language_full": "English",
  "timestamp_display": "Sat, 24 Aug 2024 (03:00 pm)"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "text": "Error : Could not detect language",
  "timestamp_display": "Sat, 24 Aug 2024 (03:00 pm)"
}
```

---

### share_private_app_data
- **File**: api_share_with_counsellor.php.
- **Description**: Updates the user's sharing permissions for private app features. The endpoint securely updates the database with the user's preferences.
- **Method**: POST
- **Parameters**: 
  - user_reference_id (required, int): The user's ID.
  - user_lang (optional, string): The user's language preference.
  - journal_shared (required, int): 1 to share the journal, 0 otherwise.
  - mood_tracker_shared (required, int): 1 to share the mood tracker, 0 otherwise.
  - habit_tracker_shared (required, int): 1 to share the habit tracker, 0 otherwise.
  - anxiety_tracker_shared (required, int): 1 to share the anxiety tracker, 0 otherwise.
  - assessments_shared (required, int): 1 to share the assessments, 0 otherwise.
- **JSON Output**: 
  - Success (code: 1):
```json
{
  "code": 1,
  "message": "settings_updated"
}
```
  - Failure (code: 0):
```json
{
  "code": 0,
  "message": "user_missing"
}
```
