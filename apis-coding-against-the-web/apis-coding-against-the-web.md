APIs: Coding Against the Web
================
Steven M. Mortimer
10/19/2018

-   [Background](#background)
    -   [What is an API](#what-is-an-api)
    -   [What makes a SOAP API?](#what-makes-a-soap-api)
    -   [What makes a REST API?](#what-makes-a-rest-api)
    -   [Working with XML and JSON](#working-with-xml-and-json)
-   [Getting started in R](#getting-started-in-r)
    -   [How to authenticate to an API](#how-to-authenticate-to-an-api)
    -   [Keeping your credentials hidden in .Rprofile](#keeping-your-credentials-hidden-in-.rprofile)
    -   [Becoming familiar with HTTP status codes](#becoming-familiar-with-http-status-codes)
    -   [Anatomy of API calls (scheme, hostname, path, query, and body)](#anatomy-of-api-calls-scheme-hostname-path-query-and-body)
    -   [HTTP verbs (GET, POST, PUT, PATCH, DELETE)](#http-verbs-get-post-put-patch-delete)
-   [Using the Square APIs (Connect v1 & v2)](#using-the-square-apis-connect-v1-v2)
    -   [The role of locations in the Square API](#the-role-of-locations-in-the-square-api)
    -   [Pulling transactions](#pulling-transactions)
    -   [Pulling order itemizations](#pulling-order-itemizations)
    -   [Using the squareupr package](#using-the-squareupr-package)
    -   [The performance of the API](#the-performance-of-the-api)
    -   [Next steps](#next-steps)

Background
==========

<center>
<img src="./img/earl-2017-jeremy-morris.png" /> <em>Credit: Jeremy Morris at EARL 2017</em>
</center>
<br>

What is an API
--------------

An <u><b>A</b></u>pplication <u><b>P</b></u>rogramming <u><b>I</b></u>nterface is just an abstraction from a lower level process. Think of this as a black box where you just need to provide some formatted input and you'll get back some formatted output without any need to understand how things are being done.

<center>
<img src="./img/black-box.png" />
</center>
Today, we'll be discussing web-based data APIs. They abstract away the data being stored in a database, so that you can access the data without needing to get access to the database or write ad-hoc queries. Web-based data APIs come in two flavors (SOAP and REST), which we'll discuss next.

<center>
<img src="./img/soap-v-rest.png" />
</center>
<br>

What makes a SOAP API?
----------------------

SOAP stands for <u><b>S</b></u>imple <u><b>O</b></u>bject <u><b>A</b></u>ccess <u><b>P</b></u>rotocol.

It turns out that SOAP APIs can function on multiple computer protocols (HTTP, SMTP, TCP, or JMS). This means that SOAP APIs don't need to function over the web, which is just one easy way to transmit information to people.

SOAP APIs are almost always written in XML (eXtensible Markup Language), and instructions that describe how you can use SOAP APIs are almost always provided in a WSDL (Web Service Description Language). Below is a snapshot of the Salesforce API and how it defines a "PermissionSet" object.

<center>
<img src="./img/salesforce-wsdl.png" />
</center>
As you can see, this code is pretty complicated and hard to read. On the other hand, it is very specific about how to define objects and work with them in the API. For this reason, XML is more secure, ACID-compliant, and all-around more durable despite being harder to work with.

Additional Resources:

-   [The Difference Between SOAP and REST](http://spf13.com/post/soap-vs-rest)
-   [SOAP vs. REST](https://www.upwork.com/hiring/development/soap-vs-rest-comparing-two-apis/)
-   [SOAP vs REST Challenges](http://www.soapui.org/testing-dojo/world-of-api-testing/soap-vs--rest-challenges.html)

What makes a REST API?
----------------------

REST stands for <u><b>RE</b></u>presentational <u><b>S</b></u>tate <u><b>T</b></u>ransfer.

REST uses the Verb-Noun Paradigm where operations map exactly to HTTP protocol actions like `GET`, `POST`, `PUT`, and `DELETE` and the collections of data are nouns. For example, in the Square API you would GET (verb) a customer (noun).

REST APIs are almost always JSON (Javascript Object Notation) where the instructions are provided on a web document (typically Swagger UI style) that you can read and format. The data types are not as strict as a SOAP API. Below is a snapshot of the Square API. You can see that it's much simpler to read than the Salesforce SOAP API shown above.

<center>
<img src="./img/square-api-doc.png" />
</center>
<br>

Working with XML and JSON
-------------------------

It is important to know whether an API is SOAP or REST because, as mentioned, they are linked to processing and returning either XML or JSON. Unfortunately, in R there are a number of competing packages that help handle each format. I recommend trying the **xml2** and **jsonlite** packages first. These are both part of the tidyverse. However, sometimes you may find that the **XML**, **rjson**, or **RJSONIO** packages provide unique functionality that is better for your use case.

Let's consider a tangible example of these two formats, one after the other. For this purpose, imagine that you are working with customer data. Here's what a single customer record might look like in each API format.

**SOAP API (XML)**

    <customer>
      <first_name>Elon</first_name>
      <last_name>Musk</last_name>
      <occupation>Genius</occupation>
    </customer> 

**REST API (JSON)**

    {
      "customer": {
        "first_name": "Elon",
        "last_name": "Musk",
        "occupation": "Genius"
      }
    } 

Getting started in R
====================

Above we mentioned two packages **xml2** and **jsonlite**. These packages will parse API data from XML or JSON to a `list` or `data.frame`. However, we must be able to communicate with the server to even get the data in the first place. This HTTP communication can be done using the **httr** package. Along with the `httr` package it is helpful to load the rest of the **tidyverse**, which contains the **dplyr**, **tidyr**, and **purrr** packages that will aid in further processing the data into a tidy format.

``` r
library(xml2)
library(jsonlite)
library(httr)
library(tidyverse)
```

How to authenticate to an API
-----------------------------

APIs almost always require authenticating, because the authentication process protects an API's data and because creators usually want to track and limit requests accordingly. There are three common types of authentication:

1.  Basic Authentication: You send over your username and password
2.  API Key Authentication: You send over a long key or token in the URL or the Header
3.  OAuth 2.0 Authentication: You authorize a client to interact with your level of permissions

**Basic Authentication**

``` r
response <- GET("http://httpbin.org/basic-auth/user/passwd", 
                authenticate("user", "passwd"))
content(response, as = "parsed")
```

**API Key Authentication**

``` r
# Square API list locations with authentication provided in the header
GET("https://connect.squareup.com/v2/locations", 
    add_headers(Authorization = sprintf("Bearer %s", 
                                        "sq0atp-PQdK9iw65pOwQgTQXtQt6")))

# Open Movie Database movie search by title with the authentication provided in
# the URL
GET(paste0('https://www.omdbapi.com/',
           '?t=The+Godfather&plot=short&apikey=4439909d'))
```

**Note**: Username/Password and Key-based authentication is only considered secure if used together with other security mechanisms, such as HTTPS/SSL. Otherwise, people who snoop and grab your internet traffic could see in plain-text what your credentials are and then turn around to use them themselves.

**OAuth 2.0 Authentication**

This is the most complex and secure method of authentication. It consists of multiple parts, namely:

-   An `Authorization URL`, where you send your request initially
-   A `Token URL`, where you get your token initially
-   A `Refresh URL`, where you'll get a new token after your current token expires
-   `Scopes`, a list of resources that clients can access given the credentials they supply

Fortunately, R takes care of all of this, so long as you create a "client" in R.

``` r
sq_oauth_app <- oauth_app(
  "square",
  key = app_id, 
  secret = app_secret,
  redirect_uri = callback_url
)

sq_oauth_endpoints <- oauth_endpoint(
  request = NULL,
  base_url = sprintf("%s/oauth2",
                     "https://connect.squareup.com"),
  authorize = "authorize", 
  access = "token", 
  revoke = "revoke"
)

sq_token <- oauth2.0_token(
  endpoint = sq_oauth_endpoints,
  app = sq_oauth_app, 
  cache = cache
)

GET("https://connect.squareup.com/v2/locations", 
    config(token = sq_token))
```

Keeping your credentials hidden in .Rprofile
--------------------------------------------

In the API Key Authentication example above, I put my token in the script like this:

``` r
GET("https://connect.squareup.com/v2/locations", 
    add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))
```

This is not really secure because anybody with this script can now pretend to be me. As an alternative, you can have R store your passwords and other sensitive information outside of your script to keep it hidden. Follow the steps below to keep the Square API token hidden on your computer in a way that will allow also you to still be able to reference it in future scripts.

The concept we're going to apply for this purpose involves environment variables in R. The R manual provides some detail about how and where environment variables can be stored:

> R searches for a user profile, a file of R code. The path of this file can be specified by the R\_PROFILE\_USER environment variable (and tilde expansion will be performed). If this is unset, a file called ‘.Rprofile’ is searched for in the current directory or in the user's home directory (in that order). The user profile file is sourced into the workspace.

The first step is figuring out your HOME directory. You can do this by running the following command in R:

``` r
# create the .Rprofile file at this location if it doesn't already exist
Sys.getenv("HOME")
```

    ## [1] "/Users/steven.mortimer"

### Enabling the Viewing of Hidden Files

The .Rprofile file is a "hidden" file because it starts with a period. For this reason you may need to enable "Hidden Files" in your File Explorer (Windows) or Finder (Mac) in order for it to show up in your folders. To do this on Mac you need to open up the terminal and type:

    defaults write com.apple.finder AppleShowAllFiles YES

Finally relaunch Finder by using holding the Option key, then right clicking on the Finder icon in the dock and click Relaunch.

### Adding your credentials to .Rprofile

Open up the .Rprofile file, add the following lines, and save it.

    options(scipen=99, stringsAsFactors=FALSE)
    Sys.setenv(SQUARE_PAT = "sq0atp-PQdK9iw65pOwQgTQXtQt6")

**NOTE**: Ensure that there is a new line at the end of your file so that the second command runs!

### Checking your .Rprofile settings

Now that you've saved your token in you'll need to restart R. This is because R reads the `.Rprofile` everytime you start R, so the restart forces the file to be loaded again. You can restart R by going to `Session -> Restart R` or using the shortcut `CMD+SHIFT+F10` (`CRTL+SHIFT+F10` on Windows). You can check that the token now exists by running:

``` r
Sys.getenv('SQUARE_PAT')
```

    ## [1] "sq0atp-PQdK9iw65pOwQgTQXtQt6"

Becoming familiar with HTTP status codes
----------------------------------------

Before you get started making API calls, it is important to understand various messages that the API will return. HTTP status codes follow a pattern so that, at a glance, understand where the error happened. After that there is usually an "error code" and "error detail" that more fully describes why that error occurred. Here are the ranges of codes and what they mean:

-   2XX: Success!
-   4XX: Your fault!
-   5XX: Their fault!

Here is an example where I wasn't authenticated before making the API call. It's a 401 error, that's my fault.

``` r
response <- GET("https://connect.squareup.com/v2/locations")
response
```

    ## Response [https://connect.squareup.com/v2/locations]
    ##   Date: 2018-10-19 13:55
    ##   Status: 401
    ##   Content-Type: application/json
    ##   Size: 601 B

``` r
content(response, as='parsed')
```

    ## $errors
    ## $errors[[1]]
    ## $errors[[1]]$category
    ## [1] "AUTHENTICATION_ERROR"
    ## 
    ## $errors[[1]]$code
    ## [1] "UNAUTHORIZED"
    ## 
    ## $errors[[1]]$detail
    ## [1] "Your request did not include an `Authorization` http header with an access token. The header value is expected to be of the format \"Bearer TOKEN\" (without quotation marks), where TOKEN is to be replaced with your access token (e.g. \"Bearer ABC123def456GHI789jkl0\"). For more information, see https://docs.connect.squareup.com/api/connect/v2/#requestandresponseheaders. If you are seeing this error message while using one of our officially supported client libraries, please report this to developers@squareup.com. "

Again, here is the right way to provide authentication to the call:

``` r
GET("https://connect.squareup.com/v2/locations",
    add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))
```

Here is an example where I didn't get the URL correct. I forgot to include the "s" to spell "locations" not "location" as the endpoint.

``` r
response <- GET("https://connect.squareup.com/v2/location")
response
```

    ## Response [https://connect.squareup.com/v2/location]
    ##   Date: 2018-10-19 13:55
    ##   Status: 404
    ##   Content-Type: application/json
    ##   Size: 156 B

``` r
content(response, as='parsed')
```

    ## $errors
    ## $errors[[1]]
    ## $errors[[1]]$category
    ## [1] "INVALID_REQUEST_ERROR"
    ## 
    ## $errors[[1]]$code
    ## [1] "NOT_FOUND"
    ## 
    ## $errors[[1]]$detail
    ## [1] "API endpoint for URL path `/v2/location` and HTTP method `GET` is not found."

There are about 40 status codes. Here are tips on how to handle the most common codes you'll see:

-   200: Success: Jump for joy, everything was processed successfully
-   401: Unauthenticated: Make sure you're sending your password or token correctly
-   404: Not Found: Make sure the URL you're sending to is the right one
-   500: Server Error: Double check you're formatting the request correctly

Anatomy of API calls (scheme, hostname, path, query, and body)
--------------------------------------------------------------

Every API call can be broken down into a few different elements. See this example where we can create a new customer record

-   **Scheme**: http or https
-   **Hostname**: connect.squareup.com
-   **Path**: v2/customers
-   **Query**: ?key1=value1&key2=value2
-   **Body**: `{'given_name'='Bob', 'family_name'='Miller', 'note'='API Test'}`

Putting his all together you want to make a request like this:

    POST https://connect.squareup.com/v2/customers
    Authorization: Bearer sq0atp-PQdK9iw65pOwQgTQXtQt6
    {
      "given_name": "Bob",
      "family_name": "Miller",
      "note": "API Test"
    }

Here is how to replicate that from R:

``` r
response <- POST('https://connect.squareup.com/v2/customers', 
                 add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")),
                 body = list(given_name='Bob', 
                             family_name='Miller', 
                             note='API Test'), 
                 encode='json')
status_code(response)
content(response, as='parsed')
```

In the example above we didn't show how the "query" component fits in. A query is information added at the end of the "path" component to control the behavior. It's always represented as a series of key, value pairs after a question mark in the URL. Here is an example where we pull all the transactions for the Corner location between October 5th and 6th and ensure that they are returned in descending order.

``` r
response <- GET(paste0('https://connect.squareup.com/v2/locations/1AWPRVVVFWGQF/transactions',
                       '?begin_time=2018-10-05T00:00:00Z&end_time=2018-10-06T00:00:00Z&sort_order=DESC'),
                add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))
response
```

    ## Response [https://connect.squareup.com/v2/locations/1AWPRVVVFWGQF/transactions?begin_time=2018-10-05T00:00:00Z&end_time=2018-10-06T00:00:00Z&sort_order=DESC]
    ##   Date: 2018-10-19 13:55
    ##   Status: 200
    ##   Content-Type: application/json
    ##   Size: 34.8 kB

As you can see the URL can get very long and complicated. The `httr` package has a function called `build_url()` that makes it easier to construct the right URL from a bunch of component parts. Here is an example where we use the `build_url()` function to create our request URL.

``` r
url <- parse_url("http://connect.squareup.com")
url$scheme <- "https"
url$path <- "v2/locations/1AWPRVVVFWGQF/transactions"
url$query <- list(begin_time = "2018-10-05T00:00:00Z", 
                  end_time = "2018-10-06T00:00:00Z",
                  sort_order = "DESC")
this_url <- build_url(url)
this_url
```

    ## [1] "https://connect.squareup.com/v2/locations/1AWPRVVVFWGQF/transactions?begin_time=2018-10-05T00%3A00%3A00Z&end_time=2018-10-06T00%3A00%3A00Z&sort_order=DESC"

You'll notice here that colon (`:`) character was "escaped" with `%3A`. This is because certain characters are reserved and can be mistaken as part of the URL path, so the `build_url()` function safely handles these by encoding them. This is a handy way to avoid mistakes in forming the URL.

HTTP verbs (GET, POST, PUT, PATCH, DELETE)
------------------------------------------

In the last example you saw that I used two different functions, the `POST()` function and then the `GET()` function. This is because I wanted to do different types of things. In the first example I wanted to "post" the new customer data to the API server. In the second, I wanted to "get" the customer data.

The takeaway is always keep in mind what type of operation you are trying to do. If you are trying to create or update a record, you most likely won't be using the `GET()` function. Using the wrong HTTP verb is an error that can sometimes be hard to debug and is easy to avoid. This table summarizes the basic type of behavior you want to do and the associated verb.

| Verb      | Behavior         | Has Body? |
|-----------|------------------|:---------:|
| GET       | Read Data        |     ✗     |
| POST      | Create/Read Data |     ✓     |
| PUT/PATCH | Update Data      |     ✓     |
| DELETE    | Delete Data      |     ✗     |

Below is a screenshot of the API documentation where they make it very clear what type of HTTP request you should be using if you want to create a customer in contrast to just listing customers.

<img width="400px" src="./img/create-customer.png" /> </br> <img width="400px" src="./img/list-customers.png" />

Using the Square APIs (Connect v1 & v2)
=======================================

Square has developed two different APIs (Connect [v1](https://docs.connect.squareup.com/api/connect/v1) & [v2](https://docs.connect.squareup.com/api/connect/v2)) that are organized to have endpoints that each serve a different type of data. For examples, a URL for customers (`v2/customers`), a URL for payments (`v1/{location_id}/payments`), a URL for the products you sell (`v1/{location_id}/items`), etc. This way you can easily find the type of data you're looking for.

The role of locations in the Square API
---------------------------------------

The Square API stores a lot of information. It is likely that they have a database that is "partitioned" by location so that whenever a new request comes in they can quickly pull the data by navigating directly to that location's partion and then pulling the data based on the request. You'll see that certain endpoints require you to specify the location. Here is an example for the transactions:

`GET /v2/locations/{location_id}/transactions`

So how do you get the location IDs? Use the `ListLocations` endpoint. Here is how to gather the information for every location:

``` r
response <- GET('https://connect.squareup.com/v2/locations',
                add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))
map_df(content(response, as='parsed')$locations, 
       .f=function(x){
         tibble(id = x$id, 
                location_name = x$name)
       })
```

    ## # A tibble: 5 x 2
    ##   id            location_name         
    ##   <chr>         <chr>                 
    ## 1 46FYN9N9RQS54 Crozet Market         
    ## 2 DRDCJ2X8E2PMV Preston Ave. (C'ville)
    ## 3 8T1TYXE840S00 Purvelo (C'ville)     
    ## 4 1AWPRVVVFWGQF The Corner (C'ville)  
    ## 5 50X1GNAWEC8V0 The Yards (DC)

Pulling transactions
--------------------

Now that we know The Corner location has ID `1AWPRVVVFWGQF` we can use it to pull all the recent transactions.

``` r
response <- GET(paste0('https://connect.squareup.com/v2/locations/1AWPRVVVFWGQF/transactions',
                       '?begin_time=2018-10-05T00:00:00Z&end_time=2018-10-06T00:00:00Z&sort_order=DESC'),
                add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))

all_transactions <- content(response, as='parsed')$transactions %>%
  map_df(~as_tibble(modify_if(., ~(length(.x) > 1 | is.list(.x)), list)))

all_transactions
```

    ## # A tibble: 50 x 6
    ##    id          location_id  created_at    tenders product client_id       
    ##    <chr>       <chr>        <chr>         <list>  <chr>   <chr>           
    ##  1 pjGTWWz7Jo… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 2A0DA755-6063-4…
    ##  2 T6Vi0JMrwG… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… E9A6E294-EC46-4…
    ##  3 51JYnnwusU… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 6E9F2BA9-4224-4…
    ##  4 7owdwcBf9E… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… E311E20D-1583-4…
    ##  5 XEhYF31ZeL… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 489B4BBB-3788-4…
    ##  6 TYaBO9W8Oy… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… D7D391F1-1FC4-4…
    ##  7 ZdvZrM5Pv8… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 7DAA3556-30F4-4…
    ##  8 FLdYeuAlVL… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 9FFBC80D-666E-4…
    ##  9 RnQGvM2NO8… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… B67B97C6-8BBA-4…
    ## 10 niU0z6Cl1Y… 1AWPRVVVFWG… 2018-10-05T2… <list … REGIST… 0FC3F2D8-025F-4…
    ## # ... with 40 more rows

The transaction literally only represents a location and timestamp when a transaction occurred. The "tender" where money was exchanged is included inside the "tenders" field:

``` r
all_transactions$tenders[1]
```

    ## [[1]]
    ## [[1]][[1]]
    ## [[1]][[1]]$id
    ## [1] "v6MvMLM8kpC8hXNF5RZj1vMF"
    ## 
    ## [[1]][[1]]$location_id
    ## [1] "1AWPRVVVFWGQF"
    ## 
    ## [[1]][[1]]$transaction_id
    ## [1] "pjGTWWz7Jo79V2BZYzAREaAfV"
    ## 
    ## [[1]][[1]]$created_at
    ## [1] "2018-10-05T21:28:59Z"
    ## 
    ## [[1]][[1]]$amount_money
    ## [[1]][[1]]$amount_money$amount
    ## [1] 1100
    ## 
    ## [[1]][[1]]$amount_money$currency
    ## [1] "USD"
    ## 
    ## 
    ## [[1]][[1]]$processing_fee_money
    ## [[1]][[1]]$processing_fee_money$amount
    ## [1] 30
    ## 
    ## [[1]][[1]]$processing_fee_money$currency
    ## [1] "USD"
    ## 
    ## 
    ## [[1]][[1]]$customer_id
    ## [1] "155YFXMHFX11WSVD4W6YF1SDSC"
    ## 
    ## [[1]][[1]]$type
    ## [1] "CARD"
    ## 
    ## [[1]][[1]]$card_details
    ## [[1]][[1]]$card_details$status
    ## [1] "CAPTURED"
    ## 
    ## [[1]][[1]]$card_details$card
    ## [[1]][[1]]$card_details$card$card_brand
    ## [1] "VISA"
    ## 
    ## [[1]][[1]]$card_details$card$last_4
    ## [1] "2891"
    ## 
    ## [[1]][[1]]$card_details$card$fingerprint
    ## [1] "sq-1-kmjWoTxJbuhTOpy_3jZr9Q40Pa4azzOCa54kl5BmIsf63bG5y0X-ELaPLUDrOr-88A"
    ## 
    ## 
    ## [[1]][[1]]$card_details$entry_method
    ## [1] "EMV"

Pulling order itemizations
--------------------------

In the example above we saw that the transaction data only provides the gross revenue for the transaction and the customer ID. If we want to know more about each customer order, then we need to use the `Payments` endpoint.

``` r
response <- GET(paste0('https://connect.squareup.com/v1/1AWPRVVVFWGQF/payments',
                       '?begin_time=2018-10-05T00:00:00Z&end_time=2018-10-06T00:00:00Z'),
                add_headers(Authorization = sprintf("Bearer %s", "sq0atp-PQdK9iw65pOwQgTQXtQt6")))

all_payments <- content(response, as='parsed') %>%
  map_df(~as_tibble(modify_if(., ~(length(.x) > 1 | is.list(.x)), list)))

head(all_payments$itemizations[1][[1]][[1]], 4)
```

    ## $name
    ## [1] "CACAO MONSTER"
    ## 
    ## $quantity
    ## [1] "1.00000000"
    ## 
    ## $notes
    ## [1] "banana / cacao / dates / cashew butter / house almond milk"
    ## 
    ## $item_variation_name
    ## [1] "Single (16 oz)"

Using the squareupr package
---------------------------

Pulling API data in R is definitely a time consuming process. You have to check the documentation, fiddle with the R code to reproduce the HTTP call, then parse the output into a tidy format. For this reason, R users typically create packages that simplify your workflow and cut out all the processing and guesswork. This is the reason for creating the **squareupr** package. In order to start using the package you must first install it from GitHub like this:

``` r
devtools::install_github('StevenMMortimer/squareupr')
```

And before using the package you need to load it and supply it with your Square API token so that all of the API calls are authenticated properly.

``` r
library(squareupr)
sq_auth(Sys.getenv('SQUARE_PAT'))
```

Now that the pacakge is loaded you can pull transaction data is like this:

``` r
our_locations <- sq_list_locations()
yesterdays_transactions <- sq_list_transactions("The Corner (C'ville)")
yesterdays_transactions
```

    ## # A tibble: 143 x 6
    ##    id          location_id  created_at    tenders product client_id       
    ##    <chr>       <chr>        <chr>         <list>  <chr>   <chr>           
    ##  1 dxsT7eIm8h… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 2F945CEC-D109-4…
    ##  2 RdhHymtIx8… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 934FBFDD-68A5-4…
    ##  3 X2aMXj9pzG… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 5434D236-7DE4-4…
    ##  4 thCALBXhU9… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 580647FC-4B1D-4…
    ##  5 d3zXEM3GvF… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 0E8859D9-0FE0-4…
    ##  6 ZJ7iUrgCYh… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 2890FFC8-AF89-4…
    ##  7 dFAov2DeUe… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… E7B4FD95-9699-4…
    ##  8 NtUupF1Ssu… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 9007A815-4A99-4…
    ##  9 P2gY0V0z31… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… 96076647-C96D-4…
    ## 10 pLSXoxTtyY… 1AWPRVVVFWG… 2018-10-18T2… <list … REGIST… CBC2C31B-502A-4…
    ## # ... with 133 more rows

Or pulling customers from the last 90 days can be done like this

``` r
created_start <- format(Sys.Date()-90, '%Y-%m-%dT00:00:00-00:00')
created_end <- format(Sys.Date(), '%Y-%m-%dT00:00:00-00:00')
our_customers <- sq_search_customers(query = list(filter=
                                                    list(created_at=
                                                           list(start_at=created_start,
                                                                end_at=created_end))))
our_customers$given_name <- "{HIDDEN}"
our_customers$family_name <- "{HIDDEN}"
our_customers %>% select(id, created_at, updated_at, 
                         given_name, family_name, preferences, groups)
```

    ## # A tibble: 7,978 x 7
    ##    id     created_at  updated_at given_name family_name preferences groups
    ##    <chr>  <chr>       <chr>      <chr>      <chr>       <list>      <list>
    ##  1 BCKGB… 2018-08-14… 2018-08-3… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  2 X8DZM… 2018-10-14… 2018-10-1… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  3 KBZVT… 2018-08-31… 2018-08-3… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  4 0M4VW… 2018-07-24… 2018-07-2… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  5 E735V… 2018-09-10… 2018-09-1… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  6 GAG2W… 2018-09-21… 2018-09-2… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  7 VDZZ7… 2018-07-25… 2018-07-2… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  8 RWTS7… 2018-09-03… 2018-09-0… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ##  9 RFJP4… 2018-08-25… 2018-08-2… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ## 10 T1HZ3… 2018-08-29… 2018-08-2… {HIDDEN}   {HIDDEN}    <list [1]>  <list…
    ## # ... with 7,968 more rows

And if you'd like to get the assigned groups there is a function called `sq_extract_cust_groups()` to do that. This is an example of having "helper" functions that continue to process down the API data into a tidy format so that we all can do analyses faster and easier.

``` r
sq_extract_cust_groups(our_customers)
```

    ## # A tibble: 12,110 x 3
    ##    id                      groups.id                  groups.name         
    ##    <chr>                   <chr>                      <chr>               
    ##  1 BCKGBSEV4555AZ0B09VXG7… CQ689YH4KCJMY.LOYALTY_ALL  Loyalty Enrollees   
    ##  2 BCKGBSEV4555AZ0B09VXG7… gv2:Y19WF036Z538AGJVA8Q2E… Instant Profile     
    ##  3 X8DZMWH9N12S6KZED6AZTH… CQ689YH4KCJMY.REACHABLE    Reachable           
    ##  4 X8DZMWH9N12S6KZED6AZTH… gv2:Y19WF036Z538AGJVA8Q2E… Instant Profile     
    ##  5 KBZVT7KPFD1TMN0D1NDAF4… CQ689YH4KCJMY.LOYAL        Regulars            
    ##  6 KBZVT7KPFD1TMN0D1NDAF4… CQ689YH4KCJMY.LOYALTY_ALL  Loyalty Enrollees   
    ##  7 KBZVT7KPFD1TMN0D1NDAF4… gv2:FZT6PS1Y152K3BY4DD1HX… Experiment Group Co…
    ##  8 KBZVT7KPFD1TMN0D1NDAF4… gv2:Y19WF036Z538AGJVA8Q2E… Instant Profile     
    ##  9 0M4VWF9NT9532R8E103Z1F… CQ689YH4KCJMY.REACHABLE    Reachable           
    ## 10 0M4VWF9NT9532R8E103Z1F… gv2:Y19WF036Z538AGJVA8Q2E… Instant Profile     
    ## # ... with 12,100 more rows

The performance of the API
--------------------------

It takes 70 seconds to get a month's worth of transaction data from the Corner location, then it takes another 75 seconds to get the payment data to breakdown each transaction's items. Furthermore, it takes a few minutes to take the JSON and convert it into a tidy format. You can see that the time starts to become a barrier to actually doing data analysis. For this reason we've started pulling down the API data and using that instead of using the API directly. All of that data is being stored here:

<https://github.com/sustainablecommerce/the-juice-laundry/tree/master/data>

Next steps
----------

Our analyses of Square data will require more functions and storing more data. Every step towards making a data flow reproducible and tidy will reduce mistakes and the time to analyze later. So if you come up with a function to do something useful consider adding it to the package or our library to leverage again later.
