---
title: API Documentation
---

This document provides structured documentation for the available APIs.\
Each API includes details such as purpose, request structure,
parameters, authentication, and example JSON responses.\
\
Base URL:\
https://localhost/api.php/v1/\
\
Authentication:\
All API endpoints require Bearer Token authentication.\
\
Header Format:\
Authorization: Bearer \[BEARER_TOKEN\]

# chat_summary_fb15

\*\*File:\*\* api_chat_summary_fb15.php

\*\*Method:\*\* POST

\*\*Description:\*\* Generates a summary of past chats for a user using
an external AI service.

## Parameters

  -----------------------------------------------------------------------
  Name              Type              Required          Description
  ----------------- ----------------- ----------------- -----------------
  user_id           int               Yes               User's ID

  number_of_chats   int               No                Number of chat
                                                        summaries to
                                                        retrieve
  -----------------------------------------------------------------------

## Success Response

{\
\"code\": \"1\",\
\"data\": \[\
{\
\"date\": \"21 Dec 2023 10:30 am (A few hours ago)\",\
\"counsellor\": \"Counsellor Name\",\
\"chat_topic\": \"Romantic Relationship Issues\",\
\"feedback_star\": \"4\",\
\"summary\": \"This is a detailed summary of the
conversation.\<br\>\<b\>Close Chat Remarks:\</b\> A remark from the
counsellor.\"\
}\
\]\
}

## Failure Response

{\
\"code\": \"0\",\
\"message\": \"User id does not exist\"\
}

# check_chat_usage

\*\*File:\*\* api_check_chat_usage.php

\*\*Method:\*\* POST

\*\*Description:\*\* Checks a user's usage against their subscribed chat
plan and returns eligibility.

## Parameters

  -------------------------------------------------------------------------
  Name                Type              Required          Description
  ------------------- ----------------- ----------------- -----------------
  user_reference_id   int               Yes               User's ID

  usage_check_type    string            Yes               Chat type (fb15
                                                          or others)

  user_lang           string            No                Language
                                                          preference
  -------------------------------------------------------------------------

## Success Response

{\
\"code\": 1,\
\"minutes_available\": 15,\
\"words_available\": \"\",\
\"show_buttons\": {\
\"count\": 0,\
\"buttons\": \[\]\
}\
}

## Failure Response

{\
\"code\": 0,\
\"text\": \"Your usage for Feel Better in 15 has been exceeded.\",\
\"show_buttons\": {\
\"count\": 3,\
\"buttons\": \[\
{\"type\": \"redirect\", \"text\": \"Go to Therapy over Text\",
\"redirect_code\": \"therapy\"},\
{\"type\": \"redirect\", \"text\": \"View Subscription Plans\",
\"redirect_code\": \"subscription\"},\
{\"type\": \"close\", \"text\": \"Close\", \"redirect_code\":
\"close\"}\
\]\
}\
}
