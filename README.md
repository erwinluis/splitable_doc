![Splitable](https://www.splitable.com/assets/logo.png?1327850834)


## Splitable API

With Splitable API, your customers can split the cost of a total amount with their friends. It integrates directly with your checkout page.

Watch a video of how split(able) works with a shopping cart: http://vimeo.com/37293414

### How it works

* Your customers selects split(able) as their payment option.
* Browser sends a `GET or POST` request to your server.
* Your server sends a `POST` request to split(able) with valid `api_key`, `total_amount` and other required parameters.
* User is redirected to split(able)'s payment hub which displays the line items and total amount to be split.
* User invites friends and pays an amount toward the total cost.
* Each invitee receives an invitation with a unique link to the payment hub.
* From the invitation link, invitees are automatically signed in and prompted to pay toward the total amount.
* When the total amount is paid, within the given time frame, split(able) sends a callback to your server indicating that the split was successful.
* If the total amount is not reached and the given time frame has elapsed, the callback will indicate that the team was cancelled.

### How to get your api_key

In order to use Splitable API, you need to:

* [Register](https://www.splitable.com/sign-up) your company with split(able).
* Go to company settings page and make a note of `api_key` value.

## Creating a Split

Using this api you can create a split.

### Parameters

In order to create a split send `POST` request to `https://yourcompany.splitable.com/api/splits` with following parameters:

`api_key`: This parameter is used to ensure that it is an authentic request. This is a *required* parameter.

`invoice`: An identifier from your site to keep track of what will be split, often represented as an order id. This is a *required* parameter.

`api_notify_url`: This is the callback url which split(able) will use to notify your site if a split is successful or not. More information about the callback is given below. This is a *required* parameter.

`total_amount`: This is the total amount to be split. The value for this parameter must be *in cents*. Again, please note that this value must be *in cents*. This is a *required* parameter.

`api_secret`: When split(able) sends the callback, it will contain this parameter. By default, the callback will send the `api_secret` value located in your company settings page. However if the `POST` request contains an `api_secret` then that `api_secret` will be used in the callback. This is an optional parameter.

`expires_in`: This parameter indicates how many hours a split will remain open, i.e. 24, 48, or 72. If no value is passed, the default will be 120 hours (5 days). If expires_in value is non-numeric, or greater than 120 hours, then the value passed will be ignored, no error will be raised, and the value will be set to default. The value of this parameter must be an integer. This is an optional parameter.

`shipping`: This parameter indicates the total shipping cost to be displayed. Please note that value must be *in cents*. This is an optional parameter. 

`tax`: This parameter indicates the total tax amount to be displayed. Please note that value must be *in cents*. This is an optional parameter.

`description`: This parameter is used to describe the split. The description maps to a text area, which is displayed on the split page. This is an optional parameter.


### Multiple line items

Split(able) supports multiple line items. Below are the parameters for passing line items. An order with zero line items will be rejected.

`item_name_1`: This is the title of the item. This is a required parameter.

`quantity_1`: This is the quantity of the item. This is a required parameter.

`amount_1`: This is the unit price of the item, *in cents*. Again, please note that the amount should be in *cents*. This is a required parameter.

`url_1`: This is the url of the item, where a user can view more details about the item. This is an optional field.

Similarly, the keys for the second line item will be: `item_name_2`, `quantity_2`, `amount_2`, `url_2`.

The keys for the third line item will be: `item_name_3`, `quantity_3`, `amount_3`, `url_3`.

### Response from creating a split

The response is a JSON object with two keys: split_url and split_id.

The split_url is the url to which you should redirect your user - it is the payment hub for the split.

The response of the request is always a JSON structure.

Sample success response:

    { split_url: "https://yourcompanyname.splitable.com/splits/4e284f631c3b1633996fc1f8fb7f8278a80065ec4d53d5b3ed1c/team",
      split_id:  "4e284f631c3b1633996fc1f8fb7f8278a80065ec4d53d5b3ed1c"  }

Sample error response:

    { error: "api_key is missing" }


### An example using curl

```
 curl https://nimbleshop.splitable.com/api/splits \
       -d api_key=92746e4d66cb8993 \
       -d invoice=3789273937239273 \
       -d "api_notify_url=http://www.acme.com/instant_payment_notification/splitable" \
       -d total_amount=35000 \
       -d exires_in=48 \
       -d shipping=1000 \
       -d tax=4000 \
       -d "description=Campus books has the best selection of educational books in the market" \
       -d "item_name_1=Essential Calculus Book" \
       -d quantity_1=1 \
       -d amount_1=20000 \
       -d "url_1=http://www.yourcompany.com/products/essential-calculus-book" \
       -d "item_name_2=Marine Biology Book" \
       -d amount_2=10000 \
       -d quantity_2=1 \
       -d "url_2=http://www.yourcompany.com/products/marine-biology-book"
```

Please note that in the above example `nimbleshop` is the company
subdomain. You should use your compan's subdomain instead of
'nimbleshop' when you are using the API.

### An example using Ruby

Spltiable API can be used in any language. The http `POST` request can be made in any lanauge. We at split(able) use `ruby` as the programming language. Here are some code snippets which might help you understand the API better.

Below is an example of `POST` request being sent to `https://www.splitable.com/api/splits` and the response is being parsed.

    conn = Faraday.new(:url => 'https://www.splitable.com') do |builder|
      builder.use Faraday::Request::UrlEncoded  # convert request params as "www-form-urlencoded"
      builder.use Faraday::Response::Logger     # log the request to STDOUT
      builder.use Faraday::Adapter::NetHttp     # make http requests with Net::HTTP
    end

    options = base_data(order, request).merge(line_items_data(order, request))
    response = conn.post '/api/splits', options
    data = ActiveSupport::JSON.decode(response.body)

Below is an example of how ruby code can respond `200` to the callback made by split(able).

    render nothing: true, status: 200

## Request to get the email address and names of all paid users

This api allows you to get the list of name and email of the people who
have paid towards the split.

### Parameters

Make a `GET` request with following parameters:

`api_key`: This parameter is used to ensure that it is an authentic request. This is a *required* parameter.

`split_id`: split_id is needed to uniquely identify the split.

### An example using curl

```
curl https://nimbleshop.splitable-draft.com/api/splits/d8c1f41e9be8758dfa7099fc58e4798d7a3e7a123e8d6f116519/paid_members?api_key=92746e4d66cb8993
```

Please note that in the above example `nimbleshop` is the company
subdomain. You should use your compan's subdomain instead of
'nimbleshop' when you are using the API.

## Callback/Webhook from Splitable

When the split has been successfully paid, or the split is cancelled, split(able) makes a callback to your site. The URL for that callback is the `api_notify_url` that you provided (above).

### Parameters

Splitable will make a `POST` request to the callback url with following
parameters.

`invoice`: This is the same value passed to split(able) in the `POST` request (above).

`payment_status`: This value will be either `paid` or `cancelled`. `paid` means each team member's credit card has been captured. `cancelled` means the team didn't reach the total amount within given time frame.

`api_secret`: The default callback `api_secret` is located in your company settings page. However, if the `POST` request (above) contains a different `api_secret`, then that `api_secret` will be used in the callback. This is to prevent forgery. This is an optional parameter.

`transaction_id`: This is so you may have a unique `transaction_id` in your system regarding this transaction. Split(able) will ask for this `transaction_id` if you need to contact us. This parameter is also required for issuing a refund.

### An example in curl

```
curl http://www.nimbleshop.net/payment_notifications/splitable \
       -d invoice=3789273937239273 \
       -d payment_status=paid \
       -d api_secret=82746e2d66cb8993 \
       -d transaction_id=852973493383974
```

### Client acknowledgement

To ensure that you've received the callback, split(able) checks the http response code for the callback made. If the response code is not `200` then split(able) will make another attempt to make the callback in increasing order of time. In total, split(able) will make 25 attempts, and the distribution of those 25 attempts is given below.

    1st attempt : less than a minute
    2nd attempt : less than a minute
    3rd attempt : 1 minute
    4th attempt : 4 minutes
    5th attempt : 11 minutes
    6th attempt : 22 minutes
    7th attempt : 40 minutes
    8th attempt : about 1 hour
    9th attempt : about 2 hours
    10th attempt : about 3 hours
    11th attempt : about 4 hours
    12th attempt : about 6 hours
    13th attempt : about 8 hours
    14th attempt : about 11 hours
    15th attempt : about 14 hours
    16th attempt : about 18 hours
    17th attempt : about 23 hours
    18th attempt : 1 day
    19th attempt : 1 day
    20th attempt : 2 days
    21st attempt : 2 days
    22nd attempt : 3 days
    23rd attempt : 3 days
    24th attempt : 4 days
    25th attempt : 5 days

## Contact Us

We are constantly working on improvements and releasing new features. Please reach out to us with any questions at hello@splitable.com.

