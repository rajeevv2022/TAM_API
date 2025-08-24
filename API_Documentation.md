TAM API Documentation ![](Aspose.Words.25149484-45c4-49fe-97f2-88d952376241.001.png)

This document provides structured documentation for the available APIs. 

Each API includes details such as purpose, request structure, parameters, authentication, and example JSON responses. 

**Base URL** 

https://localhost/api.php/v1/ 

**Authentication** 

All API endpoints require Bearer Token authentication. Header Format: 

Authorization: Bearer [BEARER\_TOKEN]

**Endpoints Index** 

- chat\_summary\_fb15 
- check\_chat\_usage 
- check\_counsellor\_access 
- check\_maintenance 
- check\_merge\_accounts 
- check\_unique\_user\_name 
- check\_valid\_email 
- confirm\_merge\_accounts 
- create\_audio\_conference 
- create\_user 
- delete\_user\_account 
- get\_current\_plan\_summary 
- get\_current\_share\_permissions 
- get\_random\_quote 
- ignore\_merge\_account 
- initiate\_subscription\_payment 
- send\_registration\_otp 
- show\_plans 
- subscription\_payment\_confirmation 
- tam\_connections\_consent 
- third\_banner 
- update\_user\_access\_code 
- update\_user\_information 
- validate\_referral\_code 
- audio\_transcribe 
- check\_tam\_connections\_consent 
- generate\_chat\_summary 
- get\_complete\_ysam\_post 
- google\_audio\_transcribe 
- list\_tam\_user\_conversations\_dashboard 
- list\_ysam\_posts 
- register\_tam\_connections\_consent 
- tam\_connections\_block\_user 
- tam\_connections\_send\_message 
- translate 
- upload\_audio 
- user\_conversations\_load 
- ysam\_add\_update\_article 
- ysam\_check\_usage\_tam\_connections 
- ysam\_connect\_to\_user 
- ysam\_follow\_user 
- ysam\_get\_all\_categories 
- ysam\_initialize\_form 
- ysam\_report\_user\_post 
- ysam\_review\_message 
- ysam\_translate 
- share\_private\_app\_data 

**chat\_summary\_fb15** 

File: api\_chat\_summary\_fb15.php Method: POST 

Description: Generates a summary of past chats for a user using an external AI service. It fetches chat history, generates a summary if one doesn't exist, and returns the chat history with summaries. The generated summary includes mood analysis and identifies a central topic. 

**Parameters** 

Name 

user\_id number\_of\_chats 

Type  Required int  Yes 

int  No 

Description The user's ID. 

The number of chat summaries to retrieve. Defaults to a minimum message count for summary generation. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"data": [ 

`    `{ 

`      `"date": "21 Dec 2023 10:30 am (A few hours ago)", 

`      `"counsellor": "Counsellor Name", 

`      `"chat\_topic": "Romantic Relationship Issues", 

`      `"feedback\_star": "4", 

`      `"summary": "This is a detailed summary of the conversation.<br><b>Close Chat Remarks:</b> A remark from the counsellor." 

`    `} 

`  `] 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User id does not exist" }

**check\_chat\_usage** 

File: api\_check\_chat\_usage.php Method: POST 

Description: Checks a user's usage against their subscribed chat plan (e.g., Feel Better in 15 or Therapy Over Text) and determines if they can initiate a new chat. 

**Parameters** 

Name user\_reference\_id usage\_check\_type 

user\_lang 

Type  Required  Description 

int  Yes  The user's ID. string  Yes  The type of chat to 

check, either "fb15" or another type. 

string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"minutes\_available": 15,   "words\_available": "",   "show\_buttons": { 

`    `"count": 0, 

`    `"buttons": [] 

`  `} 

}

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"text": "Your usage for Feel Better in 15 has been exceeded. Please consider Therapy Over Text as an alternative.", 

`  `"show\_buttons": { 

`    `"count": 3, 

`    `"buttons": [ 

`      `{ 

`        `"type": "redirect", 

`        `"text": "Go to Therapy over Text", 

`        `"redirect\_code": "therapy" 

`      `}, 

`      `{ 

`        `"type": "redirect", 

`        `"text": "View Subscription Plans",         "redirect\_code": "subscription" 

`      `}, 

`      `{ 

`        `"type": "close", 

`        `"text": "Close", 

`        `"redirect\_code": "close" 

`      `} 

`    `] 

`  `} 

}

**check\_counsellor\_access** 

File: api\_check\_counsellor\_access.php Method: POST 

Description: Authenticates a user with a Google theablemind.com domain and provisions them as a counselor, admin, or ops lead if they do not exist in the system. 

**Parameters** 

Name  Type  Required  Description access\_token  string  Yes  The Google access 

token. 

user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

[ 

`  `{ 

`    `"code": 1, 

`    `"type": ["A", "C"], 

`    `"c\_id": "123", 

`    `"c\_name": "John Doe", 

`    `"c\_email": "john.doe@theablemind.com", 

`    `"c\_phone": "1234567890", 

`    `"c\_timezone": "Asia/Kolkata", 

`    `"c\_age": "35", 

`    `"c\_gender": "M", 

`    `"c\_picture": "http://example.com/images/john.jpg", 

`    `"c\_lang\_code": "en", 

`    `"c\_country": "IN", 

`    `"c\_video": "http://example.com/videos/john\_intro.mp4",     "message": "" 

`  `} 

]

Failure (code: 0) 

[ 

`  `{ 

`    `"code": 0, 

`    `"type": [], 

`    `"c\_id": "", 

`    `"c\_name": "",     "c\_email": "", 

`    `"c\_phone": "", 

`    `"c\_timezone": "", 

`    `"c\_age": "", 

`    `"c\_gender": "", 

`    `"c\_picture": "", 

`    `"c\_lang\_code": "", 

`    `"c\_country": "", 

`    `"c\_video": "", 

`    `"message": "Access Denied. This feature is only available to employees of The Able Mind." 

`  `} 

]

**check\_maintenance** 

File: api\_check\_maintenance.php Method: POST 

Description: Checks for any ongoing system maintenance and informs the user if specific app features are disabled. 

**Parameters** 

Name  Type  Required  Description user\_lang  string  No  The user's language 

preference. 

timezone  string  No  The user's timezone. **Responses** 

Success (returns a status object) 

{ 

`  `"UPDATION\_INPROGRESS": true, 

`  `"MESSAGE\_TITLE": "Maintenance Alert", 

`  `"MESSAGE": "We are undergoing a scheduled maintenance. The service will be back on Saturday, 24 Aug 2024 [05:30 am IST]", 

`  `"LIVE\_BACK\_TIME": "2024-08-24T00:00:00Z", 

`  `"REGISTRATION": false, 

`  `"FEEL\_BETTER\_IN\_15": true, 

`  `"THERAPY\_OVER\_TEXT": true, 

`  `"NIGHT\_AUXIE": false, 

`  `"TROOPERS\_TOGETHER": true, 

`  `"LIBRARY": false, 

`  `"RESOURCES": false, 

`  `"ASSESSMENTS": false, 

`  `"CONNECTIONS": false, 

`  `"YSAM": false 

}

No Maintenance (returns a default object) 

{ 

`  `"UPDATION\_INPROGRESS": false,   "MESSAGE\_TITLE": "", 

`  `"MESSAGE": "", 

`  `"LIVE\_BACK\_TIME": "", 

`  `"REGISTRATION": false, 

`  `"FEEL\_BETTER\_IN\_15": false,   "THERAPY\_OVER\_TEXT": false,   "NIGHT\_AUXIE": false, 

`  `"TROOPERS\_TOGETHER": false,   "LIBRARY": false, 

`  `"RESOURCES": false, 

`  `"ASSESSMENTS": false, 

`  `"CONNECTIONS": false, 

`  `"YSAM": false 

}

**check\_merge\_accounts** 

File: api\_check\_merge\_accounts.php Method: POST 

Description: Checks if a user has multiple accounts that should be merged, typically based on email or phone number. 

**Parameters** 

Name  Type user\_email  string 

user\_phone  string login\_type  string 

Required  Description 

No  The user's email 

address. 

No  The user's phone 

number. 

No  The login type, 

defaults to 'E' for email. 

**Responses** 

Success (status: success) 

{ 

`  `"status": "success", 

`  `"data": { 

`    `"show\_merge\_screen": true, 

`    `"primary\_account\_name": "User Name", 

`    `"secondary\_account\_name": "Duplicate User", 

`    `"message": "We have found a duplicate account for you. Would you like to merge them?" 

`  `} 

}

Error (status: error) 

{ 

`  `"status": "error", 

`  `"data": "A technical issue has occurred." }

**check\_unique\_user\_name** 

File: api\_check\_unique\_user\_name.php 

Method: POST 

Description: Validates if a user name is unique in the system. 

**Parameters** 

Name  Type  Required user\_name  string  Yes 

login\_key  string  No user\_lang  string  No 

Description 

The user name to validate. 

The user's login key. 

The user's language preference. 

**Responses** 

Success (response\_code: 1) 

{ 

`  `"response\_code": "1",   "message": "success",   "status": "success",   "data": "" 

}

Failure (response\_code: 0) 

{ 

`  `"response\_code": "0", 

`  `"message": "User Name is not unique",   "status": "failure", 

`  `"data": "" 

}

**check\_valid\_email** 

File: api\_check\_valid\_email.php Method: POST 

Description: Validates an email address by checking for a valid format, existing MX records, and whether the domain is a known disposable email provider. 

**Parameters** 

Name  Type  Required  Description user\_email  string  Yes  The email address 

to validate. 

**Responses** 

Success (response\_code: 0) 

{ 

`  `"response\_code": "0", 

`  `"message": "Email id provided is valid.",   "status": "success", 

`  `"data": "" 

}

Failure (response\_code: -1) 

{ 

`  `"response\_code": "-1", 

`  `"message": "Email domain (mailinator.com) is invalid.",   "status": "failure", 

`  `"data": "" 

}

**confirm\_merge\_accounts** 

File: api\_confirm\_merge\_accounts.php Method: POST 

Description: Confirms and completes the merging of two user accounts by linking two different user profiles into a single account. 

**Parameters** 

}

Name  Type user\_id  int 

duplicate\_id  int login\_type  string 

Required  Description 

Yes  The ID of the user 

initiating the merge. 

Yes  The ID of the 

account to merge into the primary account. 

Yes  The primary login 

type for the merged account ('E' for email or 'P' for phone). 

}

**Responses** 

Success (status: success) 

{ 

`  `"status": "success", 

`  `"data": "Accounts successfully merged" }

Error (status: error) 

{ 

`  `"status": "error", 

`  `"data": "Error merging accounts" 

}

**create\_audio\_conference** 

File: api\_create\_audio\_conference.php 

Method: POST 

Description: Creates or updates an audio conference. 

**Parameters** 


Name conference\_id 

conference\_duration conference\_start\_time conference\_counsellor\_keys 

Type  Required  Description string  Yes  Unique ID for the 

conference. If it exists, all data will be updated; otherwise, it will be inserted. 

int  No  Duration of the 

session in minutes. Defaults to 60. 

string  Yes  Conference start 

date and time in "Y-m-d H:i:s" format. 

string  Yes  Comma-separated 

list of counsellor keys. 


conference\_title  string max\_simultaneous\_connections  int 

Yes  Title of the 

conference. 

No  Maximum number 

of simultaneous connections. Defaults to 20. 


**Responses** 

Success (response\_code: 1) 

{ 

`  `"response\_code": "1",   "message": "success",   "status": "success", 


`  `"data": "" }

Failure (response\_code: 0) 

{ 

`  `"response\_code": "0", 

`  `"message": "Error Creating / Updating Conference",   "status": "failure", 

`  `"data": "" 

}

**create\_user** 

File: api\_create\_user.php Method: POST 

Description: Registers a new user account with various details such as name, email, phone, and demographic information. 

**Parameters** 

Name login\_type 

user\_name user\_email 

user\_phone\_number user\_password 

user\_age user\_sex user\_lang 

external\_login 

Type  Required  Description string  Yes  'E' for email or 'P' 

for phone. 

string  Yes  User's name. string  Conditional  Required if 

login\_type is 'E'. 

string  Conditional  Required if 

login\_type is 'P'. string  No  The user's 

password. 

string  No  User's age. string  No  User's gender. string  No  The user's 

language preference. 

string  No  External login 

identifier. 

user\_guardian\_name  string user\_guardian\_phone  string user\_guardian\_relationship  string 

client\_ip\_address  string device\_id  string access\_code  string 

No  Guardian name. No  Guardian phone. No  Guardian 

relationship. 

No  Client IP address. No  Device ID. 

No  Access code. 

**Responses** 

Success (response\_code: 1) 

{ 

`  `"response\_code": "1", 

`  `"message": "success", 

`  `"status": "success", 

`  `"data": "", 

`  `"show\_user\_usage\_policy": "" }

Failure (response\_code: 0) 

{ 

`  `"response\_code": "0", 

`  `"message": "User Registration failed as Login Type field missing",   "status": "failure", 

`  `"data": "", 

`  `"show\_user\_usage\_policy": "" 

}

**delete\_user\_account** 

File: api\_delete\_user\_account.php Method: POST 

Description: Deletes a user's account and removes their session from the database. It requires a delete\_account flag and user/session details for validation. 

**Parameters** 

Name  Type delete\_account  bool 

user\_id  int session\_token  string user\_lang  string 

Required  Description 

Yes  A flag to confirm 

account deletion. 

Yes  The ID of the user to 

be deleted. 

Yes  The user's session 

token for validation. No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"message": "delete\_success" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"message": "generic\_error" }

**get\_current\_plan\_summary** 

File: api\_get\_current\_plan\_summary.php Method: POST 

Description: Retrieves a summary of a user's current subscription plan, including details like plan type and end date. 

**Parameters** 

Name user\_reference\_id user\_email user\_key user\_lang 

Type  Required int  No string  No string  No string  No 

Description 

The user's ID. The user's email. A user key. 

The user's language preference. 

client\_ip\_address  string  No **Responses** 

Success (response\_code: 1) 

{ 

`  `"response\_code": 1, 

`  `"message": "success", 

`  `"status": "success", 

`  `"data": { 

`    `"end\_date": "2024-12-31 23:59:59",     "plan\_type": "Monthly Subscription",     "price\_paid": "499", 

`    `"status": "Active" 

`  `} 

}

Failure (response\_code: 0) 

{ 

`  `"response\_code": 0, 

`  `"message": "No active plan found",   "status": "failure", 

`  `"data": {} 

}

The user's IP address. 

**get\_current\_share\_permissions** 

File: api\_get\_current\_share.php Method: POST 

Description: Fetches the user's current sharing permissions for various app features, such as a journal and mood tracker. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"message": "settings\_loaded", 

`  `"data": { 

`    `"journal\_shared": true, 

`    `"mood\_tracker\_shared": false, 

`    `"habit\_tracker\_shared": true, 

`    `"anxiety\_tracker\_shared": false,     "assessments\_shared": true 

`  `} 

}

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"message": "user\_missing" }

**get\_random\_quote** 

File: api\_get\_random\_quote.php 

Method: POST 

Description: Retrieves a random quote from the database. 

**Parameters** 

Name user\_lang 

user\_reference\_id 

Type  Required  Description string  No  The preferred 

language for the quote. 

string  No  The user's ID to 

fetch personalized quotes. 

**Responses** 

Success 

{ 

`  `"quote": "The only way to do great work is to love what you do.",   "author": "Steve Jobs" 

}

Failure 

{ 

`  `"quote": "",   "author": "" }

**ignore\_merge\_account** 

File: api\_ignore\_merge\_account.php 

Method: POST 

Description: Ignores the flag to merge a user's accounts. 

**Parameters** 

Name  Type  Required  Description user\_id  string  Yes  The user's ID. 

**Responses** 

Success (status: success) 

{ 

`  `"status": "success", 

`  `"data": "Merge flag ignored for user" }

Error (status: error) 

{ 

`  `"status": "error", 

`  `"data": "Technical issue" }

**initiate\_subscription\_payment** 

File: api\_initiate\_subscription\_payment.php Method: POST 

Description: Initiates a subscription payment process, validates the user and plan, creates a payment entry in the database, and returns the necessary data for a Razorpay payment gateway. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. plan\_type  string  Yes  The plan's category 

code. 

subscription\_price  string  No  The subscription 

price. subscription\_discount  int  No  The subscription 

discount 

percentage. 

referral\_code  string  No  A referral code to 

apply a discount. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"razorpay\_json": "{"key":"...","amount":1000,"currency":"INR","name":"The Able Mind","description":"Monthly Plan","image":"...","prefill":{"name":"John Doe","email":"john.doe@example.com","contact":"9876543210"},"notes":{"m erchant\_order\_id":"...","category":"MONTHLY","payment\_gst\_fees":10,"pay ment\_price\_wo\_fees":100,"price\_id":123},"order\_id":"...","theme":{"colo r":"#5F9EA0"},"send\_sms\_hash":true,"method":{"netbanking":true,"card":t rue,"wallet":true,"upi":true,"emi":false,"pay later":false}}" 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "Error initiating subscription." }

**send\_registration\_otp** 

File: api\_send\_registration\_otp.php Method: POST 

Description: Generates a one-time password (OTP) and sends it to a user's email for registration purposes. 

**Parameters** 

Name  Type  Required  Description user\_email  string  Yes  The user's email 

address. 

user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "Error sending OTP." }

**show\_plans** 

File: api\_show\_plans.php Method: POST 

Description: Retrieves a list of available subscription plans for a user, taking into account their location and any special offers. 

**Parameters** 

Name  Type  Required user\_key  string  No user\_lang  string  No 

new  boolean  No 

Description The user's key. 

The user's language preference. 

A flag to request a new JSON format. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success", 

`  `"data": [ 

`    `{ 

`      `"plan\_id": 1, 

`      `"plan\_name": "Monthly Plan", 

`      `"price": "499", 

`      `"currency": "INR", 

`      `"features": ["Feature 1", "Feature 2"]     } 

`  `] 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "error\_retrieving\_plans" }

**subscription\_payment\_confirmation** 

File: api\_subscription\_payment\_confirmation.php Method: POST 

Description: Confirms a payment made through the Razorpay gateway. It verifies the payment signature, updates the database records for the subscription and payment, and sends a confirmation email/notification to the user. 

**Parameters** 

Name  Type  Required  Description razorpay\_order\_id  string  Yes  The order ID from 

Razorpay. 

razorpay\_payment\_id  string  Yes  The payment ID 

from Razorpay. razorpay\_signature  string  Yes  The payment 

signature from 

Razorpay. 

user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "plan\_confirmation" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "payment\_failed" }

**tam\_connections\_consent** 

File: api\_tam\_connections\_consent\_status.php Method: POST 

Description: This API checks if a user has consented to the "TAM Connections" feature. It returns the consent message if consent is required. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Consent Required (code: 0) 

{ 

`  `"code": "0", 

`  `"text": { 

`    `"title": "Consent Title", 

`    `"introduction\_1": "Introduction 1", 

`    `"introduction\_2": "Introduction 2", 

`    `"introduction\_guidelines": "Guidelines", 

`    `"consent\_line\_1": "Consent Line 1", 

`    `"consent\_line\_2": "Consent Line 2", 

`    `"consent\_line\_3": "Consent Line 3 with {XXXX} replacement",     "consent\_line\_4": "Consent Line 4", 

`    `"consent\_line\_5": "Consent Line 5", 

`    `"consent\_line\_6": "Consent Line 6", 

`    `"consent\_line\_7": "Consent Line 7", 

`    `"consent\_line\_8": "", 

`    `"consent\_line\_9": "", 

`    `"consent\_final": "Final consent text", 

`    `"consent\_text": "Accept", 

`    `"cancel\_text": "Decline" 

`  `} 

}

Consent Granted (code: 1) 

{ 

`  `"code": "1" }

**third\_banner** 

File: api\_third\_banner.php Method: POST 

Description: Retrieves information for a third banner, including details on next appointments, Therapy Over Text (TOT) usage, and available counselor slots. 

**Parameters** 

Name user\_reference\_id slots 

user\_lang 

Type  Required  Description 

int  Yes  The user's ID. 

int  No  The number of 

counselor slots to retrieve. Defaults to 2. 

string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"next\_appointment": { 

`    `"date\_full\_display": "Monday, 25 Aug 2024", 

`    `"date\_display": "25 Aug 24", 

`    `"time\_start\_display": "10:00 am", 

`    `"time\_end\_display": "11:00 am", 

`    `"counsellor": "Jane Doe" 

`  `}, 

`  `"tot": { 

`    `"daily\_limit\_message": "Words used: 150/500 today"   }, 

`  `"next\_slot": { 

`    `"counsellor": "John Smith", 

`    `"next\_counsellor\_slot": [ 

`      `{ 

`        `"slot\_id": 1, 

`        `"date": "2024-08-26", 

`        `"time\_start": "09:00", 

`        `"time\_end": "10:00" 

`      `} 

`    `] 

`  `}, 

`  `"tam\_library\_url": "http://example.com/library", 

`  `"tam\_resource\_center\_url": "http://example.com/resources" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User id cannot be empty" }

**update\_user\_access\_code** 

File: api\_update\_user\_access\_code.php 

Method: POST 

Description: Updates a user's access code and plan information. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. access\_code  string  Yes  The new access 

code. 

user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (response\_code: 1) 

{ 

`  `"response\_code": "1", 

`  `"message": "success", 

`  `"status": "success", 

`  `"data": { 

`    `"code": "1", 

`    `"message": "Access code updated successfully"   } 

}

Failure (response\_code: 0) 

{ 

`  `"response\_code": "0", 

`  `"message": "Error", 

`  `"status": "failure", 

`  `"data": { 

`    `"error": "Access code is invalid."   } 

}

**update\_user\_information** 

File: api\_update\_user\_information.php Method: POST 

Description: Updates a user's profile information, including name, email, phone, and guardian details. 

**Parameters** 

Name  Type  Required  Description primary\_login\_key  string  Yes  The user's primary 

login key (email or phone). 

user\_reference\_id  int user\_name  string user\_age  string user\_sex  string user\_country  string user\_email  string user\_phone  string user\_guardian\_name  string user\_guardian\_email  string user\_guardian\_phone  string user\_guardian\_relationship  string 

access\_code  string employee\_id  string user\_lang  string 

No  The user's ID. No  User's name. 

No  User's age. 

No  User's gender. No  User country. No  User email. 

No  User phone. 

No  Guardian name. No  Guardian email. No  Guardian phone. No  Guardian 

relationship. 

No  Access code. No  Employee ID. No  Language 

preference. 

**Responses** 

Success (status: 1) 

{ 

`  `"status": "1", 

`  `"message": "success", 

`  `"data": { 

`    `"id": "123", 

`    `"name": "Updated Name"   } 

}

Failure (status: -1) 

{ 

`  `"status": "-1", 

`  `"message": "Error updating user information" }

**validate\_referral\_code** 

File: api\_validate\_referral\_code.php Method: POST 

Description: Validates a referral code against a user's subscription and applies the corresponding discount if the code is valid. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. referral\_code  string  Yes  The referral code 

to validate. 

subscription\_category\_code  string  Yes  The plan's category 

code. 

user\_lang  string  No  The user's 

language preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"discount": 10, 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"discount": "", 

`  `"message": "error\_code\_inactive" }

**audio\_transcribe** 

File: audio\_transcribe.php Method: POST 

Description: Transcribes an audio file into text using a third-party service (Whisper API). It can also generate an audio response from the transcribed text. 

**Parameters** 

Name  Type file\_url  string 

generate\_audio  string api\_key  string 

user\_lang\_code  string user\_gender  string 

return\_type  string 

Required  Description Yes  The URL of the 

audio file. 

Yes  "Y" to generate 

audio, "N" otherwise. 

Yes  The API key for the 

transcription service. 

No  The user's language 

code. 

No  The user's gender 

('M', 'F', or 'O') for voice generation. 

No  "1" to return audio 

only, otherwise returns text and audio. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"text": "This is the transcribed text.", 

`  `"language": "en", 

`  `"language\_full": "English", 

`  `"audio": "http://example.com/audio/response.mp3" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"text": "voice\_note\_missing",   "language": "not\_detected", 

`  `"language\_full": "not\_detected" }


**check\_tam\_connections\_consent** 

File: check\_tam\_connections\_consent.php Method: POST 

Description: This API checks if a user has consented to the "TAM Connections" feature. It returns the consent message if consent is required. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Consent Required (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "Consent is required", 

`  `"data": { 

`    `"title": "Consent Title", 

`    `"introduction\_1": "Introduction 1", 

`    `"introduction\_2": "Introduction 2", 

`    `"introduction\_guidelines": "Guidelines", 

`    `"consent\_line\_1": "Consent Line 1", 

`    `"consent\_line\_2": "Consent Line 2", 

`    `"consent\_line\_3": "Consent Line 3 with {XXXX} replacement",     "consent\_line\_4": "Consent Line 4", 

`    `"consent\_line\_5": "Consent Line 5", 

`    `"consent\_line\_6": "Consent Line 6", 

`    `"consent\_line\_7": "Consent Line 7", 

`    `"consent\_line\_8": "", 

`    `"consent\_line\_9": "", 

`    `"consent\_final": "Final consent text", 

`    `"consent\_text": "Accept", 

`    `"cancel\_text": "Decline" 

`  `} 

}

Consent Granted (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

**generate\_chat\_summary** 

File: generate\_chat\_summary.php Method: POST 

Description: Generates a summary for a specific chat session or for all sessions requiring a summary. This is intended to be used as a cron job or a background process. 

**Parameters** 

Name  Type  Required  Description session\_id  int  No  The specific session 

ID to summarize. 

**Responses** 

Success 

{ 

`  `"close\_summary": "The summary has been generated successfully." }

Error 

{ 

`  `"close\_summary": "Error while generating the summary" }

**get\_complete\_ysam\_post** 

File: get\_complete\_ysam\_post.php Method: POST 

Description: Retrieves the full content of a single YSAM post, including details about its author and any related information. 

**Parameters** 

Name user\_reference\_id 

ysam\_id user\_lang 

Type  Required  Description 

int  Yes  The ID of the user 

requesting the post. 

int  Yes  The ID of the YSAM 

post to retrieve. string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"data": { 

`    `"ysam\_id": "123", 

`    `"ysam\_title": "My Story", 

`    `"ysam\_body": "The full content of the post...",     "author\_name": "Jane Doe", 

`    `"category": "Anxiety" 

`  `}, 

`  `"message": "" 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "technical\_issue" }

**google\_audio\_transcribe** 

File: google\_audio\_transcribe.php 

Method: POST 

Description: Transcribes an audio file into text using Google Cloud Speech-to-Text. 

**Parameters** 

Name audio\_file\_path 

user\_lang api\_key 

Type  Required string  Yes 

string  No string  Yes 

Description 

The local path to the audio file. 

The user's language preference. 

The Google Cloud API key. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"transcription": "This is the transcribed text." }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"error": "Error processing audio file." }

**list\_tam\_user\_conversations\_dashboard** 

File: list\_tam\_user\_conversations\_dashboard.php Method: POST 

Description: Retrieves a list of user conversations to display on a dashboard. It can be filtered by a search query. 

**Parameters** 

Name user\_reference\_id search 

user\_lang 

Type  Required int  Yes string  No 

string  No 

Description The user's ID. 

A search query to filter conversations. 

The user's language preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "", 

`  `"data": [ 

`    `{ 

`      `"conversation\_id": "1", 

`      `"user\_name": "User 1", 

`      `"last\_message": "Hello, how are you?",       "timestamp": "2024-08-24 15:30:00" 

`    `} 

`  `] 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "technical\_issue" }

**list\_ysam\_posts** 

File: list\_ysam\_posts.php 

Method: POST 

Description: Fetches a list of "Your Story and Mine" (YSAM) posts based on various filters. 

**Parameters** 

Name page\_number 

posts\_per\_page ysam\_user hashtag category 

search\_criteria published 

only\_count user\_id user\_lang 

Type  Required  Description 

int  No  The page number 

for pagination. 

int  No  The number of posts 

to return per page. int  No  The user ID to filter 

posts by. 

string  No  A hashtag to filter 

posts by. 

string  No  The category code 

to filter posts by. 

string  No  A search term. boolean  No  Flag to filter for 

published posts. 

boolean  No  Flag to only return 

the count of posts. int  No  The current user's 

ID. 

string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1",   "message": "",   "data": [ 

`    `{ 

`      `"ysam\_id": "1", 

`      `"ysam\_title": "First Post", 

`      `"ysam\_body\_snippet": "This is a snippet of the first post...",       "author\_name": "Author 1", 

`      `"created\_at": "2024-08-24 15:00:00" 

`    `} 

`  `] 

}

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"count": 0, 

`  `"message": "processing\_error" }

**register\_tam\_connections\_consent** 

File: register\_tam\_connections\_consent.php Method: POST 

Description: Records a user's consent to participate in the "TAM Connections" feature, saving the consent message and other details to the database. 

**Parameters** 

Name user\_reference\_id consent\_string 

user\_lang 

Type  Required int  Yes string  Yes 

string  No 

Description The user's ID. 

The consent message string. 

The user's language preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "Error registering consent" }

**tam\_connections\_block\_user** 

File: tam\_connections\_block\_user.php 

Method: POST 

Description: Blocks another user from connecting or communicating with the current user. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The current user's 

ID. 

user\_reference\_block\_id  int  Yes  The ID of the user to 

be blocked. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id is not specified" }

**tam\_connections\_send\_message** 

File: tam\_connections\_send\_message.php Method: POST 

Description: Sends a message from one user to another within the "TAM Connections" feature. It handles message validation, language detection, and translation. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The sender's user 

ID. 

recipient\_reference\_id  int message  string type  string user\_lang  string 

Yes  The recipient's user 

ID. 

Yes  The message 

content. 

Yes  Message type, e.g., 

'T' for text. 

No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"text": "Hello, how are you?", 

`  `"detected\_language": "en", 

`  `"special\_message": "", 

`  `"sender\_date\_display": "24 Aug 2024 03:30 pm", 

`  `"recipient\_text": "Hallo, wie geht es Ihnen?", 

`  `"receipient\_date\_display": "24 Aug 2024 03:30 pm" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"text": "User not authorized" }

**translate** 

File: translate.php Method: POST 

Description: Translates text from one language to another using the Google Translate API. It can also save the translated message to the database. 

**Parameters** 

}

Name text\_to\_translate 

target\_language message\_id from\_counsellor testing 

Type  Required string  Yes 

string  Yes int  No bool  No bool  No 

Description 

The text to be translated. 

The two-letter code for the target language. 

The ID of the message to update in the database. 

A flag indicating if the message is from a counselor. 

A flag for testing purposes. 

}

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"text": "Translated text here.", 

`  `"detected\_language": "en", 

`  `"detected\_language\_full": "English" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"text": "Translation Error : The text to translate cannot be empty." 

}

**upload\_audio** 

File: upload\_audio.php Method: POST 

Description: Handles the upload of an audio file, either from a file input or from a Base64- encoded string. 

**Parameters** 


Name  Type type  string 

recordedAudio  string audioFile  file 

Required  Description 

Yes  The upload type, "R" 

for recorded audio (Base64), or a different value for a file upload. 

Conditional  Required if type is 

'R', the Base64-

encoded audio data. Conditional  Required if type is 

not 'R', the audio file 

to upload. 


**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"message": "File is successfully uploaded.",   "file": "/path/to/uploaded\_file.wav" 

}

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"message": "No audio file to upload.",   "file": "" 

}


**user\_conversations\_load** 

File: user\_conversations\_load.php Method: POST 

Description: Loads a user's conversation history based on their connection type and room name. 

**Parameters** 

Name user\_reference\_id connection\_type 

room\_name user\_lang 

Type  Required  Description 

int  Yes  The user's ID. string  Yes  The type of 

conversation, 'S' for a single user or 'G' for a group. 

string  Yes  The name of the 

conversation room. string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "", 

`  `"data": { 

`    `"room\_name": "ysam\_conv\_123\_456", 

`    `"chat\_history": [ 

`      `{ 

`        `"id": "1", 

`        `"sender\_id": "123", 

`        `"message": "Hello!", 

`        `"timestamp": "2024-08-24 15:00:00"       } 

`    `] 

`  `} 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id or Connection type is not specified" }

**ysam\_add\_update\_article** 

File: ysam\_add\_update\_article.php Method: POST 

Description: Creates a new YSAM post or updates an existing one with a title, body, hashtags, and category. 

**Parameters** 

Name ysam\_update 

user\_reference\_id ysam\_id 

ysam\_body ysam\_hashtags 

ysam\_title ysam\_user\_name 

ysam\_category\_id ysam\_narration user\_lang 

Type  Required  Description 

string  Yes  'Y' to update an 

existing article, or 'N' to add a new one. 

int  Yes  The user's ID. 

int  Conditional  Required if 

updating, the ID of the post to update. 

string  Yes  The main content of 

the post. 

string  No  A comma-separated 

list of hashtags. 

string  Yes  The title of the post. string  Yes  The name of the 

user posting the article. 

int  Yes  The category ID of 

the article. 

string  No  A narration for the 

article. 

string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"text": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"text": "User Id not specified." }

**ysam\_check\_usage\_tam\_connections** 

File: ysam\_check\_usage\_tam\_connections.php 

Method: POST 

Description: Checks if a user has exceeded their daily message limit for "TAM Connections". 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Within Limit (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "You have 5 messages remaining for today." }

Limit Exceeded (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "Message limit exceeded. Please try again tomorrow." }

**ysam\_connect\_to\_user** 

File: ysam\_connect\_to\_user.php 

Method: POST 

Description: Updates the connection status between two users within the YSAM feature. 

**Parameters** 

Name user\_reference\_id 

author\_reference\_id user\_lang 

Type  Required  Description 

int  Yes  The current user's 

ID. 

int  Yes  The ID of the user to 

connect with. 

string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id or Author Id not specified" }

**ysam\_follow\_user** 

File: ysam\_follow\_user.php 

Method: POST 

Description: Allows a user to follow or unfollow another user in the YSAM section. 

**Parameters** 

Name user\_reference\_id 

user\_to\_follow follow 

Type  Required  Description 

int  Yes  The current user's 

ID. 

int  Yes  The ID of the user to 

follow/unfollow. string  No  'Y' to follow, 'N' to 

unfollow. Defaults 

to 'Y'. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id not specified" }

**ysam\_get\_all\_categories** 

File: ysam\_get\_all\_categories.php 

Method: POST 

Description: Retrieves a list of all available YSAM post categories. 

**Parameters** 

Name  Type  Required  Description user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "", 

`  `"data": [ 

`    `{ 

`      `"category\_id": "1", 

`      `"category\_name": "Anxiety",       "category\_code": "anx" 

`    `} 

`  `] 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "technical\_issue" }

**ysam\_initialize\_form** 

File: ysam\_initialize\_form.php 

Method: POST 

Description: Retrieves data required to initialize the YSAM post creation form. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "", 

`  `"data": { 

`    `"user\_name": "Jane Doe", 

`    `"user\_picture": "http://example.com/profile.jpg",     "categories": [ 

`      `{ 

`        `"category\_id": "1", 

`        `"category\_name": "Anxiety" 

`      `} 

`    `] 

`  `} 

}

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id not specified" }

**ysam\_report\_user\_post** 

File: ysam\_report\_user\_post.php Method: POST 

Description: Allows a user to report a YSAM post for inappropriate content or other reasons. 

**Parameters** 

Name user\_reference\_id ysam\_id 

previous\_reported 

reason user\_lang 

Type  Required  Description 

int  Yes  The user's ID. 

int  Yes  The ID of the post to 

report. 

string  No  'Y' if previously 

reported, 'N' otherwise. Defaults to 'N'. 

string  Yes  The reason for 

reporting the post. string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": "1", 

`  `"message": "success" }

Failure (code: 0) 

{ 

`  `"code": "0", 

`  `"message": "User Id not specified" }

**ysam\_review\_message** 

File: ysam\_review\_message.php Method: POST 

Description: Reviews the content of a message for inappropriate or harmful content using an external AI service before it is sent. 

**Parameters** 

Name  Type  Required  Description text\_to\_translate  string  Yes  The message to be 

reviewed. 

user\_lang  string  No  The user's language 

preference. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"detected\_language": "en", 

`  `"link": "", 

`  `"timestamp": "2024-08-24 15:00:00" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"text": "The message has been flagged as inappropriate.",   "timestamp": "Sat, 24 Aug 2024 (03:00 pm)" 

}

**ysam\_translate** 

File: ysam\_translate.php Method: POST 

Description: Translates a message from a user in the YSAM feature, handling language detection and providing a translated message for the recipient. 

**Parameters** 

Name  Type message  string 

sender  int recipient  int user\_lang  string 

user\_timezone  string 

Required  Description Yes  The message to 

translate. 

Yes  The sender's ID. Yes  The recipient's ID. No  The user's language 

preference. 

No  The user's timezone. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"text": "Translated message text", 

`  `"detected\_language": "en", 

`  `"detected\_language\_full": "English", 

`  `"timestamp\_display": "Sat, 24 Aug 2024 (03:00 pm)" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"text": "Error : Could not detect language", 

`  `"timestamp\_display": "Sat, 24 Aug 2024 (03:00 pm)" }

**share\_private\_app\_data** 

File: api\_share\_with\_counsellor.php Method: POST 

Description: Updates the user's sharing permissions for private app features. The endpoint securely updates the database with the user's preferences. 

**Parameters** 

Name  Type  Required  Description user\_reference\_id  int  Yes  The user's ID. user\_lang  string  No  The user's language 

preference. 

journal\_shared  int mood\_tracker\_shared  int habit\_tracker\_shared  int anxiety\_tracker\_shared  int 

assessments\_shared  int 

No  1 to share the 

journal, 0 otherwise. No  1 to share the mood 

tracker, 0 otherwise. No  1 to share the habit 

tracker, 0 otherwise. 

No  1 to share the 

anxiety tracker, 0 otherwise. 

No  1 to share the 

assessments, 0 otherwise. 

**Responses** 

Success (code: 1) 

{ 

`  `"code": 1, 

`  `"message": "settings\_updated" }

Failure (code: 0) 

{ 

`  `"code": 0, 

`  `"message": "user\_missing" }
