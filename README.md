![Splitable](https://www.splitable.com/images/logo.png?1327850834)


## Checkout with Splitable

`checkout with splitable` allows store owners to allow their customers to split the cost of a product or a service. It works pretty similar to `checkout with paypal`.

### How it works

* User clicks on a button or takes an action to indicate the user wants to split the amount.
* Browser sends a request (GET or POST) reques to the store server.
* Store server sends a `POST` request to splitable with valid `api_key`, `total_amount` and other required parameters.
* User sees splitable website with the line items and the total amount that is being split among friends.
* User invites friends to split the total cost.
* Invited users pay their share.
* When everyone pays then the splitable sends a callback to the client site indicating that everyone has paid.
* The callback includes invoice and the api_secret to prevent and forgery.
* In case everyone does not pay and time lapses then callback will indicate that the team was cancelled.

### Getting api_key

In order to checkout with splitable, client needs to do following things:

* Register your company with Splitable.
* Go to company settings page and make a note of `api_key` value.

### Sending the request

When a user clicks on `checkout with splitable` then a `POST` request should be made to `https://splitable.com/api/splits` with following parameters.

<table>
  <tbody>
    <tr>
      <th>Parameters</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
      <th>Default</th>
      <th>Required?</th>
    </tr>
  </tbody>
</table>

    { api_key: '320882ab45',
      invoice: '1234'
      }

* api_key : This is a *required* parameter. This field is used to ensure that it is an authentic request and not a forgery.
* invoice : This is a *required* parameter. Usually it is order id. It is a way for the store to track for which order user wants to split the amount.
* api_notify_url : This is a *required* parameter. This is the url to which callback will be invoked. More information about callback is given below.
* total_amount : This is a *required* parameter. Total amount that needs to be split should be sent *in cents*. Again please note that the value for this field should be in cents.
* api_secret : This is an optional parameter. When splitable sends the callbacks then the callback will contain a parameter called api_secret. Usually the callback will send the api_secret value that is registered at the company level. However if the GET request sends api_secret then that api_secret will be used in the callback.
* expires_in : This is an optional parameter. The value for this value should be an integer. This field indicates how many hours is allowed for a team to close. Typically client should pass 48 or 72. If no value is passed then by default 5 days are allowed for a team to close. If expires_in value is non numeric then no error will be raised. In that case it would be assumed as if no expires_in value is passed. If expires_in value is greater than 120 hours then the value will be ignored and no error will be raised.
* shipping : This is an optional parameter. This field indicates the shipping cost to be shown. Please note that value must be in cents.
* tax : This is an optional parameter. This field indicates the tax to be shown. Please note that value must be in cents.

The response of the request is always a JSON structure.

In case of success the JSON might look like this

    { success: "https://splitable.splitable.com/items/boat-racing/splits/4e284f631c3b1633996fc1f8fb7f8278a80065ec4d53d5b3ed1c/team"}

In case of error the JSON might look like this

    { error: "api_key is missing" }
### Multiple line items

Since splitable supports multiple line items given below is an example of two line items. Splitable supports upto 20 line items for each order. An order with zero line item will be rejected.

* item_name_1: This is a required parameter. This field is used to display the title of the item for which money is being split.
* quantity_1: This is a required parameter. This field indicates the number of the products being purchased.
* amount_1: This is a required parameter. This field indicates the price of the product in cents. Please note that the amount should be in *cents* .
* url_1: This is an optional field. This field indicates the url of the product where user can go to see more details about the product.

Similarly the keys for the second line item will be: item_name_2, quantity_2, amount_2.

Similarly the keys for the third line item will be: item_name_3, quantity_3, amount_3.

### Callback/Webhook

When everyone in team pays or when team is cancelled because everyone did not pay within the allotted time then splitable makes a callback to the client site. The url that is used for callback is the `api_notify_url` that the client provided while signing up for the api services.

* invoice: This is the same value that client passed to splitable when it made `POST` request.
* payment_status: This value will be either `paid` or `cancelled`. `paid` means each team member's credit card has been captured. `cancelled` means the team could not pay within the allocated time frame.
* api_secret: This is to prevent forgery. Usually this value is taken from the company settings. However if the `POST` request contained `api_secret` then that value is used.
* transaction_id: This is so that you have a unique transaction_id in your system regarding this transaction. Also if merchant needs to contact Splitable regarding the payment then this value would be required. This parameter is also required for issuing refund.

Please note that a callback is always a `POST` request.

In order to ensure that client has indeed received the callback Splitable checks the http resonse code for the callback made. If the response code is not `200` then splitable will make another attempt to make the callback in increasing order of time. In total Splitable will make 25 attempts and the distribution of those 25 attempts is given below.

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

### Code in Ruby

`Checkout with Splitable` does not depend on any language. The http `POST` request can be made in any lanauge. We at Splitable.com use `ruby` as programming language. Here are some code snippets which might help you understand the API better.

Below is an example of `POST` request being sent to `https://www.splitable.com/api/splits` and the response is being parsed.

    conn = Faraday.new(:url => 'https://www.splitable.com') do |builder|
      builder.use Faraday::Request::UrlEncoded  # convert request params as "www-form-urlencoded"
      builder.use Faraday::Response::Logger     # log the request to STDOUT
      builder.use Faraday::Adapter::NetHttp     # make http requests with Net::HTTP
    end

    options = base_data(order, request).merge(line_items_data(order, request))
    response = conn.post '/api/splits', options
    data = ActiveSupport::JSON.decode(response.body)

Below is an exampel of how ruby code can respond `200` to the callback made by Splitable.

    render nothing: true, status: 200
