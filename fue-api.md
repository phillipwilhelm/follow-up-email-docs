# Follow-Up Emails Rest API

## Requirements

You must be using Follow-Up Emails 4.1 and the REST API must be enabled under Follow-Up Emails > Settings > System Performance.

## EndPoint

The REST API is accessible via:

    http://siteurl.com/fue-api/v1

## Authentication

### API Keys

The API Keys need to be generated from the WordPress Profile page by an administrator. If WooCommerce (2.2+)
is already installed, the API Keys generated by WooCommerce can also be used with the API instead of using separate keys
for Follow-Up Emails and WooCommerce.

### Over HTTPS

You may use HTTP Basic Auth by providing the API Consumer Key as the username and the API Consumer Secret as the password.

### Over HTTP

The API uses a ["one-legged" oAuth implementation](http://tools.ietf.org/html/rfc5849). See this [PHP Client](https://gist.github.com/75nineteen/db277427bf16880c7d17) for an example implementation.

## Pagination

Requests that return multiple items will be paginated to 20 items by default. 
You can specify further pages with the `?page` parameter:

`GET /emails?page=2`

Page number is 1-based and ommiting the `?page` parameter will return the first page.

The total number of resources and pages are always included in the `X-FUE-Total` and `X-FUE-TotalPages` HTTP headers.

## Errors

When an error occurs, an `errors` object is returned containing an array of error codes and messages.

* errors
    * errors[n].code
    * errors[n].message

# Endpoints

## Emails

### Create a New Follow-Up Email

    POST /emails

__Basic Parameters__

| Parameter         | Email Type | Description 
|-------------------|:----------:|-------------
| type              | All        | The email's type. See [Email Types](#email-types)
| template          | All        | The email template to use. Defaults to `WooCommerce`.
| name              | All        | The name used to identify the email 
| subject           | All        | The email's subject line
| message           | All        | Content body of the email. HTML or plain-text.
| status            | All        | `active`, `inactive`, or `archived`
| trigger           | All        | Refer to the triggers under the available [email types](#email-types)
| interval          | All        | Decimal that is used together with the `duration` to define the sending schedule
| duration          | All        | Possible values are `minutes`, `hours`, `days`, `weeks`, `months`, `years`, and `date`
| always\_send      | storewide  | Boolean. Passing true will disregard the `priority` and send all emails that match the event's parameters
| tracking\_code    | All        | URL tracking code. See this [tool to get the URL for Google Analytics](https://support.google.com/analytics/answer/1033867?hl=en)
| send\_date        | All        | Used when the `interval` is 'date', in [ISO 8601 format](http://www.w3.org/QA/Tips/iso-date)
| product\_id       | storewide, reminder, subscription, wootickets | The email will only be triggered if the purchased product matches this setting
| category\_id      | storewide  | Similar in nature to `product_id` but only available to `storewide` emails
| campaign          | All        | The campaign(s) this email belongs to. Campaign will be created if it doesn't exist. Separate with comma if passing multiple campaigns. See [Campaigns](#campaigns)
| requirements      | storewide, sensei  | Set additional requirements that allows for a more controlled email sending. See [Requirements](#requirements-1)

__@todo Document custom and advanced parameters__

Returns an `email` object on success

    {
        email: {
            id: "101",
            template: "WooCommerce",
            name: "After Sales Email"
            subject: "Thank You for Your Purchase!",
            message: "...",
            trigger: "processing",
            interval: 10,
            duration: "minutes",
            campaigns: [
                "sales"
            ],
            ...
        }
    }

### Update Email

    POST /emails/<id>

Updates an existing email using the same parameters for creating new emails.

### List Emails

    GET /emails
   
#### Filters

| Filter    | Description
|-----------|--------------
| type      | The email type (e.g. `storewide`)
| status    | Email status. `active`, `inactive` or `archived`
| campaign  | Only show emails under the given campaign
| limit     | The number of results to return per page

    GET /emails?filter[type]=signup&filter[status]=active&filter[limit]=10
    
Returns a list of `email`s

    [
        {
            email: {
                id: "101",
                template: "WooCommerce",
                name: "After Sales Email"
                subject: "Thank You for Your Purchase!",
                message: "...",
                trigger: "processing",
                interval: 10,
                duration: "minutes",
                ...
            }
        },
        {
            email: {
                id: "109",
                template: "Sensei",
                name: "Course Sign Up",
                subject: "Welcome to {course_name}!",
                message: "...",
                trigger: "course_signup",
                interval: 1,
                duration: "minutes",
                ...
            }
        }
    ]

### Retrieve an Email

    GET /emails/<id>

Returns an `email` object.

    {
        email: {
            id: "101",
            template: "WooCommerce",
            name: "After Sales Email"
            subject: "Thank You for Your Purchase!",
            message: "...",
            trigger: "processing",
            interval: 10,
            duration: "minutes",
            ...
        }
    }
    
### Send a Manual Email

    POST /emails/send/<id>
    
Sends a `manual` email.

__Parameters__

| Param                 | Descriptions
|-----------------------|----------------
| send_type             | `email` or `subscribers` are supported by default.
| recipient_email       | Email address of the recipient if `send_type` is `email`
| email_list            | The `list` to send to the email to if `send_type` is `subscribers`. Passing an empty value will send to all subscribers.
| schedule_email        | `1` to enable scheduling. Defaults to `0`
| sending_schedule_date | Required if `schedule_email` is enabled. RFC3339 date/time format
| send_again            | `1` to send the same email again after a certain period of time. Defaults to `0`
| interval              | Integer used in scheduling the second email
| interval_duration     | Valid values are `minutes`, `hours`, `days`, `weeks`, `months` or `years`
| tracking              | Google Analytics tracking code that will be appended to URLs in the email
    
## Campaigns
    
### List Campaigns

    GET /campaigns
    
Returns a list of `campaign`s

    [
        {
        "campaign": 
        {
            "id": "api",
            "name": "API"
        }
    },
    {
        "campaign": 
            {
                "id": "intro",
                "name": "Intro"
            }
        }
    ]
    
### List All Emails in a Campaign

    GET /campaigns/<id>
    
Returns a list of `email`s

    [
       {
           "email":
           {
               "id": 3455,
               "created_at": "2015-01-15 07:13:04",
               "type": "storewide",
               "template": "WooCommerce",
               "name": "Your Order - {store_name}",
               "subject": "Order Complete",
               "message": "{section:header}Hello {customer_name}!{/section}",
               "status": "archived",
               "trigger": "completed",
               "trigger_string": "10 minutes after Order Status: completed",
               "interval": "10",
               "duration": "minutes",
               "always_send": "0",
               "product_id": "0",
               "category_id": "0",
               "campaigns":
               [
                   "order-flow"
               ]
           }
       },
       ...
    ]
    
## Newsletters

### List Newsletter Lists

    GET /newsletter/lists
    
Returns all available newsletter lists

    {
        "lists":
        [
            "campaign2015",
            "customers"
        ]
    }

### List Subscribers

    GET /newsletter/subscribers?filter[list]=campaign2015
    
Returns a list of `subscriber`s. A filter is available to only return subscribers belonging to a particular `list`.

    [
        {
            "subscriber":
            {
                "id": "11",
                "email": "jdoe@example.com",
                "date_added": "2015-04-09 09:38:57",
                "email_list": "campaign2015"
            }
        },
        ...
    ]
    
### Add New Subscribers

    POST /newsletter/subscribers
    
| Key       | Description
------------|----------------
| email     | The email address to add. Separate emails using a comma to add multiple emails at once
| list      | The list to add the emails to

Returns a list of the created `subscribers`

    {
        "subscribers":
        [
            {
                "id": "25",
                "email": "demo1@example.com",
                "date_added": "2015-04-28 11:31:32",
                "email_list": ""
            },
            {
                "id": "26",
                "email": "demo2@example.com",
                "date_added": "2015-04-28 11:31:32",
                "email_list": ""
            },
            {
                "id": "27",
                "email": "demo3@example.com",
                "date_added": "2015-04-28 11:31:32",
                "email_list": ""
            }
        ]
    }

### Edit a Subscriber

    POST /newsletter/subscribers/<id>
    
| Key       | Description
------------|--------------
| email     | Optional. Change the email address of the subscriber.
| lists     | Optional. Assign subscriber to the given lists. Separate with comma if using multiple lists. Pass an empty value to remove from all lists.

### Delete Subscribers

    DELETE /newsletter/subscribers/<id>
    
## Requirements

### Requirement Types

| Identifier                | Email Type | Description
----------------------------|------------|---------------------------------
| bought_products           | storewide  | Restrict email to customers who have bought any of the specified product IDs
| bought_categories         | storewide  | Similar to `bought_products` except that the check is made against the category IDs a customer had previously bought products from
| first_purchase            | storewide  | Email will match customers who are purchasing for the first time
| order_total_below         | storewide  | Match orders where the order total **is below** the specified value
| order_total_above         | storewide  | Match orders where the order total **exceeds** the specified value
| total_orders_below        | storewide  | Match customers whose number of orders **is below** the specified value
| total_orders_above        | storewide  | Match customers whose number of orders **exceeds** the specified value
| total_purchases_below     | storewide  | Match customers whose accumulated order total **is below** the specified value
| total_purchases_above     | storewide  | Match customers whose accumulated order total **exceeds** the specified value
| have_not_started_first_lesson | sensei | Match learners who are signed up to a course but have not started the first lesson. Specify a course by passing a course ID as the value.  
| have_not_completed_a_lesson   | sensei | Match learners who are signed up to a course but have not completed any lessons. Specify a course by passing a course ID as the value.
| have_not_completed_a_course   | sensei | Match learners who are signup up but have not yet completed a course. Specify a course by passing a course ID as the value.
| have_not_taken_quiz           | sensei | Match learners who have not yet taken a quiz. Specify a lesson by passing a lesson ID as the value.
| have_failed_quiz              | sensei | Match learners who have failed a lesson quiz. Specify a lesson by passing a lesson ID as the value.
| have_passed_quiz              | sensei | Match learners who have passed a lesson quiz. Specify a lesson by passing a lesson ID as the value.

## Email Templates
    
### List Templates

    GET /emails/templates
    
Returns a list of `template`s

    [
       {
           "template":
           {
               "id": "fue-responsive-one-column.html",
               "name": "Responsive 1 Column",
               "sections":
               [
                   "teaser",
                   "image_600px-wide",
                   "title",
                   "subtitle",
                   "body_one",
                   "title_two",
                   "subtitle_two",
                   "body_two",
                   "address"
               ]
           }
       },
       {
           "template":
           {
               "id": "WooCommerce",
               "name": null,
               "sections":
               [
               ]
           }
       }
    ]


## Queue

### List Queue

    GET /queue
    
#### Filters

| Filter                | Description
|-----------------------|-------------
| user_id               | Return items matching a user's ID
| user_email            | Match the recipient's email address
| order_id              | WooCommerce Order ID that triggered the email
| product_id            | WooCommerce Product ID that triggered the email
| email                 | Return item's linked `email` object
| cart                  | Boolean. Pass `true` to retrieve cart emails
| sent                  | Boolean. Pass `true` to retrieve sent items
| status                | `0`: Suspended; `1`: Active
| date_sent_from        | [RFC3339 date/time format](https://www.ietf.org/rfc/rfc3339.txt) in UTC timezone
| date_sent_to          | [RFC3339 date/time format](https://www.ietf.org/rfc/rfc3339.txt) in UTC timezone
| date_scheduled_from   | [RFC3339 date/time format](https://www.ietf.org/rfc/rfc3339.txt) in UTC timezone
| date_scheduled_to     | [RFC3339 date/time format](https://www.ietf.org/rfc/rfc3339.txt) in UTC timezone
| limit                 | Limit the number of results per page

    GET /queue?filter[sent]=false&filter[status]=1&filter[date_scheduled_from]=2014-10-30T00:00:00Z&filter[date_scheduled_to]=2014-12-31T23:59:59Z
    
Returns a list of `item`s

    [
      {
        "item": {
          "id": "251",
          "user_id": "1",
          "user_email": "john@test.com",
          "order_id": "0",
          "product_id": "0",
          "email": {...},
          "date_scheduled": "2014-12-17T08:06:50Z",
          "is_cart": "0",
          "is_sent": "0",
          "date_sent": "2014-12-17T00:06:53Z",
          "email_trigger": "API Edited - Manual Email",
          "meta": {
            "recipient": [
              1,
              "jane@test.com",
              "janedoe"
            ],
            "user_id": 1,
            "email_address": "jane@test.com",
            "user_name": "janedoe",
            "subject": "Test",
            "message": "Hello Jane!"
          }
        }
      },
      {
        "item": {
          "id": "8",
          "user_id": "0",
          "user_email": "",
          "order_id": "1150",
          "product_id": "0",
          "email": {...},
          "date_scheduled": "2014-12-04T07:59:25Z",
          "is_cart": "0",
          "is_sent": "1",
          "date_sent": "2014-12-03T23:59:40Z",
          "email_trigger": "1 minute after Order Status: processing",
          "meta": ""
        }
      }
      ...
    ]
      
### Get a Specific Queue Item

    GET /queue/<item_id>
    
Returns an `item` object

    {
      "item": {
        "id": "8",
        "user_id": "0",
        "user_email": "",
        "order_id": "1150",
        "product_id": "0",
        "email": {...},
        "date_scheduled": "2014-12-04T07:59:25Z",
        "is_cart": "0",
        "is_sent": "1",
        "date_sent": "2014-12-03T23:59:40Z",
        "email_trigger": "1 minute after Order Status: processing",
        "meta": ""
      }
    }
    
### Create a Queue Item
    
    POST /queue
    
__Parameters__

| Parameter         | Description
|-------------------|------------
| email_id          | Email ID linked to the queue item
| user_id           | User ID of the customer. 0 for guests
| user_email        | Email address of the recipient
| order_id          | WooCommerce Order ID
| product_id        | WooCommerce Product ID
| send_date         | Scheduled send date in UTC 
| is_cart           | Flag for add_to_cart emails. 1 for cart emails, 0 otherwise.
| is_sent           | Flag for sent emails. 1 if email has been sent.
| meta              | An array of metadata. Values depend on the email type and trigger
| status            | 0 for suspended and 1 for active

Returns the new `item` object

### Edit a Queue Item

    POST /queue/<item_id>
    
__Parameters__

| Parameter         | Description
|-------------------|------------
| email_id          | Email ID linked to the queue item
| user_id           | User ID of the customer. 0 for guests
| user_email        | Email address of the recipient
| order_id          | WooCommerce Order ID
| product_id        | WooCommerce Product ID
| send_date         | Scheduled send date in UTC 
| is_cart           | Flag for add_to_cart emails. 1 for cart emails, 0 otherwise.
| is_sent           | Flag for sent emails. 1 if email has been sent.
| meta              | An array of metadata. Values depend on the email type and trigger
| status            | 0 for suspended and 1 for active

Returns the updated `item` object

### Delete a Queue Item

    DELETE /queue/<item_id>
    
Returns a `message` object and a `202` HTTP status on success

## Reports

### List Available Reports

    GET /reports
    
Returns a `reports` object

    {
       "reports":
       [
           "emails",
           "emails/clicks",
           "emails/opens",
           "emails/ctor",
           "users",
           "excludes"
       ]
    }

### Basic Email Reports

    GET /reports/emails
    
#### Filters

| Filter    | Description 
|-----------|-------------
| limit     | Limit the number of results per page

    GET /reports/emails?filter[limit]=25
    
Returns an `emails` object

    {
       "emails":
       [
           {
               "email": "Processing",
               "email_trigger": "1 minute after Order Status: processing",
               "num_sent": "6",
               "num_opens": "0",
               "num_clicks": "0"
           },
           {
               "email": "Marketing Upsell",
               "email_trigger": "1 minute after Order Status: processing",
               "num_sent": "3",
               "num_opens": "0",
               "num_clicks": "0"
           },
           {
               "email": "Sports Only",
               "email_trigger": "1 minute After Booking Status: confirmed",
               "num_sent": "4",
               "num_opens": "0",
               "num_clicks": "0"
           }
           ...
       ]
    }

### Get Link Click Details

    GET /reports/emails/clicks
    
#### Filters

| Filter    | Description 
|-----------|-------------
| limit     | Limit the number of results per page

    GET /reports/emails/clicks?filter[limit]=25
    
Returns a `clicks` object

    {
       "clicks":
       [
           {
               "queue_id": "485",
               "email_id": "1377",
               "user_id": "0",
               "email": "john@test.com",
               "target_url": "https://shop.homeurl.com",
               "date_added": "2015-01-21 14:30:41"
           },
           {
               "queue_id": "470",
               "email_id": "697",
               "user_id": "27",
               "email": "test-wp-user@email.com",
               "target_url": "https://shop.homeurl.com/my-account",
               "date_added": "2015-01-20 11:44:54"
           }
           ...
       ]
    }
    
### Get Email Open Details

    GET /reports/emails/opens
    
#### Filters

| Filter    | Description 
|-----------|-------------
| limit     | Limit the number of results per page

    GET /reports/emails/opens?filter[limit]=25
    
Returns an `opens` object

    {
       "opens":
       [
           {
               "queue_id": "485",
               "email_id": "1377",
               "user_id": "0",
               "email": "john@test.com",
               "date_added": "2015-01-21 14:30:41"
           },
           {
               "queue_id": "470",
               "email_id": "697",
               "user_id": "27",
               "email": "test-wp-user@email.com",
               "date_added": "2015-01-20 11:44:54"
           }
           ...
       ]
    }
    
### Get Top Click to Open Rate (CTOR) Emails

    GET /reports/emails/ctor
    
Returns a `ctor` object

    {
        "ctor":
        [
            {
                "email_id": "311",
                "clicks":   "50",
                "opens":    "211",
                "ctor":     "23.7"
            },
            {
                "email_id": "330",
                "clicks": "27",
                "opens": "121",
                "ctor": "22.3"
            }
            ...
        ]
    }
    
### User Reports

    GET /reports/users
    
### Filters
    
| Filter    | Description
|-----------|---------------
| limit     | Limit the results returned per page

    GET /reports/users?filter[limit]=15
    
Returns a `users` object.

    {
       "users":
       [
           {
               "id": 1,
               "name": "John Test",
               "sent": 63,
               "opens": 0,
               "clicks": 0
           },
           {
               "id": 27,
               "name": "test-wp-user",
               "sent": 1,
               "opens": 1,
               "clicks": 0
           },
           {
               "id": 0,
               "name": "Miley West",
               "sent": 4,
               "opens": 2,
               "clicks": 1
           }
           ...
       ]
    }
    
### List Opt-Outs

    GET /reports/excludes
    
### Filters
    
| Filter    | Description
|-----------|---------------
| limit     | Limit the results returned per page

    GET /reports/excludes?filter[limit]=15
    
Returns an `optouts` object

    {
       "optouts":
       [
           {
               "email_name": "Marketing Campaign A",
               "email_address": "jd@mail.com",
               "date": "2015-01-02T03:21:46Z"
           }
       ]
    }



# Email Types

    GET /

## Follow-Up Emails

### Sign Up Emails - `signup`

Sign up emails will send to a new user in your store

__Triggers__

| Trigger| Description
|--------|---------------------|
| signup | after user signs up |

### Manual Emails - `manual`

Manual emails allow you to create email templates for you and your team to utilize when you need to send emails immediately to customers or prospective customers.

__Triggers__

none

## WooCommerce

### Storewide Emails - `storewide`

Storewide emails will send to a buyer of any product within your store based upon the criteria you define when creating your emails.

__Triggers__

| Trigger                       | Description
|-------------------------------|------------------------
| pending                       | Pending order status
| processing                    | Processing order status
| on-hold                       | On-hold order status
| completed                     | Completed order status
| cancelled                     | Cancelled order status
| refunded                      | Refunded order status
| failed                        | Failed order status
| \<custom status\>             | Any other custom order status registered with WooCommerce
| first\_purchase               | Triggers after a customer purchases from the store for the first time.
| cart                          | Triggers right after a customer add a product to his cart
| product\_purchase\_above\_one | Triggered when a customer purchases a product more than once
| refund\_manual                | Triggers after a manual refund had been processed. WooCommerce 2.2+ only.
| refund_successful             | Triggers after a refund has been processed by a payment gateway. WooCommerce 2.2+ only.
| refund\_failed                | Triggers after a refund request had failed. WooCommerce 2.2+ only.

### Customer Emails - `customer`

Customer specific emails will re-engage your customers in the future by following up with emails specifically related to total purchases, dollar amounts, and other customer lifetime value metrics.

__Triggers__

| Trigger               | Description
|-----------------------|------------------------
| after\_last\_purchase | Used for scheduling emails to send after a set amount of time from the time of their last purchase
| order\_total\_above   | Triggered on orders where the order total is greater than the value set in the email
| order\_total\_below   | Triggered on orders where the order total is lower than the value set in the email
| purchase\_above\_one  | Triggers on every purchase after the first one
| total\_orders         | Triggered after a customer has reached the number of orders set in the email
| total\_purchases      | Triggered after a customer has surpassed the total purchased amount set in the email

### Reminder Emails - `reminder`

Reminder emails will send to a user based upon the quantity of products purchased. They will automatically trigger at a set interval for a duration equal to the quantity purchased.

__Triggers__

| Trigger       | Description
|---------------|------------------------
| processing    | Triggered after an order reaches the processing status
| completed     | Triggered after an order reaches the completed status