GC Web API Standards
======================

Working requirements for Government of Canada APIs based on the [White House Web API Standards](https://github.com/WhiteHouse/api-standards)

Presently a Draft from the TBS Web Interoperability Working Group without a set deliverable date.  RFC to [WET - GC Web API Standards](https://github.com/wet-boew/wet-boew-api-standards).

* [Style guide](#style-guide)
* [Standards](#standards)
* [Registration](#registration)
* [Hypertext constraint] (#hypertext-constraint)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error handling](#error-handling)
* [Official Languages](#official-languages)
* [Versions](#versions)
* [Record limits and offsets](#record-limits)
* [Request & Response Examples](#request-response-examples)

## Style guide

For the remainder of this document code, arguments and other undefiend technical statements will be `code fenced` as to be easily distinguishable from standard text.

Arguments text will be used as follows:

* Arguments stated in the URL will be followed by an equals sign '=' as `argument=`
* Arguments stated as a header will be followed by a colon ':' as `argument:`

## Standards

This document provides a standard along with examples for Government of Canada Web APIs, encouraging consistency, maintainability, and best practices across applications. Government of Canada APIs aim to balance a truly RESTful API interface with a positive developer experience (DX).

This document borrows heavily from:

* [The White House Web API Standards](https://github.com/WhiteHouse/api-standards)
* [Fieldings Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [Designing HTTP Interfaces and RESTful Web Services](http://munich2012.drupal.org/program/sessions/designing-http-interfaces-and-restful-web-services)
* API Facade Pattern, by Brian Mulloy, Apigee
* Web API Design, by Brian Mulloy, Apigee
* Google Maps multiple language support

## Registration

( must be described. )

## Hypertext constraint

* The HTTP Header `Accept:` is to be used to negotiate the API version and format:
   * JSON: `Accept: vnd.company.myapp.customer-v3+xml` or `Accept: application/json`
   * XML: `Accept: vnd.company.myapp.customer-v3+json` or `Accept: application/xml`

## RESTful URLs

### Standards for RESTful URLs

* A URL identifies a resource.
* URLs should include nouns, not verbs.  Nouns have properties, verbs do not.
* Filters are simple parameters
* Nouns should be plural as the resource to multiple instances of that noun.
* You shouldn’t need to go deeper than resource/identifier/resource.
* As a whole APIs should define complexity with multiple noun end points.
* URL v. header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * If it doesn’t change the logic for each response, like OAuth info, put it in the header.
* Parameters that are not explicitly filters should be prefixed by a `$` (dollar) sign:
    * Specify optional fields in a comma separated list under the `$fields` parameter.
    * Paging via `$limit`, `$offset`, and `$page`
* Access to individual records should be in the form of `/api/{resource}/{id}`

### Good URL examples
* List of earthquakes:
    * <http://example.gc.ca/api/earthquakes/>
* Filtering is a query:
    * <http://example.gc.ca/api/earthquakes?source=usgs>
* Specify optional fields in a comma separated list:
    * <http://example.gc.ca/api/earthquakes/1234?$fields=datetime,magnitude,source>

### Bad URL examples
* Non-plural noun:
    * <http://example.gc.ca/earthquake>
    * <http://example.gc.ca/earthquake/1234>
* Verb in URL:
    * <http://example.gc.ca/earthquake/1234/create>
* Filter outside of query string
    * <http://example.gc.ca/earthquakes/2011/desc>

## HTTP Verbs

Because this specification deals only in retrieving public records, only the HTTP `GET` method need by supported by compliant APIs.

## Responses
* No values in keys
    * Good: `{ "name" : "Bogart", "breed": "Bulldog" }`
    * Bad: `{ "Bogart": "bulldog" }`
* No internal-specific names (e.g. "node" and "taxonomy term")
    * Good: `{ "dog_id": 12345 }`
    * Bad: `{ "did": 12345 }`
* APIs should be self-consistent about their use of `camelCase` or `snake_case`:
    * Good: `{ "first_name" : "Stephen", "last_name" : "Harper" }` 
    * Bad: `{ "first_name" : "Stephen", "lastName" : "Harper" }` 

## Error handling

Use the following standard HTTP status codes in the header for your responses. Other codes may also be returned, as long as they're 

* `200 OK`: The request was successful
* `400 - Bad Request`: The request that the client made was invalid for some reason: 
* `404 Not Found`: Resource not found, either where an API doesn't exist, or where an individual record cannot be found 
* `429 Too Many Requests`: When a client has exceeded its throttling limit 
* `500 Internal Server Error`: There was an internal error or exception that prevented the request from being served 

The payload of error responses should include the same HTTP status code as the header, a message for the developer, message for the end-user (when appropriate), an internal error code (corresponding to some specific internally determined error number), and links where developers can find more info.

    {
        "status" : "400",
        "developerMessage" : "Verbose, plain language description of the problem. Provide developers
        suggestions about how to solve their problems here",
        "userMessage" : "This is a message that can be passed along to end-users, if needed.",
        "errorCode" : "444444",
        "more info" : "http://example.gc.ca/developer/path/to/help/for/444444,
        http://drupal.org/node/444444",
    }

General response codes are preferable where a clear code is not available.  General codes should serve for automated recognition of general state and proper error handling offer a path to resolution.

## Localization 

### Resources

- [How to localize your API | API UX](http://apiux.com/2013/04/25/how-to-localize-your-api/)

### Localized output formats and response headers

Localized output is denoted by a colon (`:`) in the field name, followed by the language code from [BCP-47](http://tools.ietf.org/html/bcp47) . Fields that do not need to be localized need not include a language code. For example:

    {
        "title:en" : "Biosphere Reserve LiDAR Survey",
        "title:fr" : "Levé LiDAR aux environs du Réserve de biosphere",
        "quantity" : 42
    }

By default, all language options are returned for localized APIs.

Along with the API's response, it should include the `Content-Language` header, as defined in [RFC3282](http://www.ietf.org/rfc/rfc3282.txt). Example, for two languages returned:

    Content-Language: en, fr

### Caller-specified language

User selected language is preferred as it allows for a lighter footprint and a more common language independent framework.

This is best executed through a `$language` query parameter. Language components are changed in place with the selected language code. Non-localized fields are always included.

    GET http://example.gc.ca/api/article/123-456/?$language=en>
    ...
    ---
    HTTP/1.1 200 OK
    ...
    Content-Language: en
    {
        "title:en" : "Biosphere Reserve LiDAR Survey",
        "quantity" : 42
    }

To request multiple languages, the `$language` parameter may be repeated, or a comma-separated list of language codes may be specified.

### Language selection via HTTP headers

The requested language can also be negotiated via the `Accept-Langauge` header, as defined in [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). Example, for a preference for French over English:

    GET /movie/gone_with_the_wind HTTP/1.1
    Accept-Language: fr,en;q=0.4
    ...
    ---
    HTTP/1.1 200 OK
    Content-Language: fr 
    ...
    {
        "title:fr" : "Levé LiDAR aux environs du Réserve de biosphere",
        "quantity" : 42
    }

## Versions

* APIs should always have a version number to maintain relation 
* Versions should be integers, not decimal numbers, prefixed with ‘v’
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* Major version numbers are required if a change can produce changes in logic
* APIs should be maintained at least one version back, and for at least 6 months.
* The HTTP Header "Accept" must be defined and accepted to negotiate the API version 
    * "Accept: application/vnd.company.myapp.customer-v3+xml"
* Version numbers should be returned via an `X-Version` header as well:
    * `X-Version: v1`
* Unless overridden via a header or parameter, APIs should default to their latest version

## Record limits, offsets, indexes and metadata

These elements are to be added where possible and relevant.  Consideration for the client creates requirements such as moblie device limitations and dataset size.

### Limits

* If no limit is specified, return results with a default limit.
    * Sanity check to a reasonable return size
    * Sanity check to a reasonable execution time
    * For small datasets limits may exceed row count

* Limits are row / object limits
    * The organization decides what is to be in a row or object
    * Limits apply to the topmost logical row or object

Limits are to be defined as the singular `$limit=` followed by an integer.

Example use:

* <http://example.gc.ca/api/dataset?$limit=10>

### Offsets

Offsets apply to the structured data returned from the API distinct from internal indexing in the data.  For the purpose of explanation we'll assume rows/objects return are numbered 1, 2, 3, 4, 5 to the limit.

The general logic is to shift to what would be the subsequent entry by the offset amount.  For a query that returns 1,2,3 an offset of 1 should return 2,3,4.

Offsets are to be defined as the singular `$offset=` followed by an integer.

Example use:

* <http://example.gc.ca/api/dataset?$limit=25&$offset=75>
    * For row is base 1 rows 76 through 100 should be returned

### Pages

Much like offsets defined above `$page=` is an offset incremented by the `$limit=` argument.  If `$limit=` is set to 25 and `$page=` is set to 2 the total offset is 50, if the `$page=` is set to 3 the total offset is 150.

Example use:

* <http://example.gc.ca/api/dataset?$limit=25&$page=3>
    * For row is base 1 rows 76 through 100 should be returned

### Continue from

`$continueFrom=` is an offset with a value based on the sort order of results returned as an index.

Use `$continueFrom=` to reliably request the following page of results without risk of skipping or receiving duplicate rows/objects due to insertions/deletions happening at the same time.

The value to pass to `continueFrom=` is returned in the metadata of each response, when any rows/objects are returned.
Typically it is a single value copied from the last row/object, and could be a date, name, internal id or any other sortable type.

Example use:
* <http://example.gc.ca/api/dataset?$limit=25&$continueFrom=76>
    * For all row bases rows 76 through 100 should be returned
* <http://example.gc.ca/api/dataset?limit=25&continueFrom=20130101.010101>
    * For all row bases rows starting from key 20130101.010101 through the next 25 rows from that index

### Metadata

Information relevant to record limits, offsets and indexes should also be included as described in the example resonse as a nested element.  Only relevant elements ( "count", "limit", "offset", "page" and "continueFrom" ) are required.

    {
        "metadata": {
            "resultset": {
                "count": 25,
                "limit": 25,
                "offset": 75,
            }
        },
        "results": [...]
    }

    {
        "metadata": {
            "resultset": {
                "count": 25,
                "limit": 25,
                "page": 3,
            }
        },
        "results": [...]
    }

    {
        "metadata": {
            "resultset": {
                "count": 25,
                "limit": 25,
                "continueFrom": "20130101.010101",
            }
        },
        "results": [...]
    }

## Documentation minimums

( must be described. )

## Callbacks

( must be described. )

## Request & Response Examples

### API Resources

  - [GET /earthquakes](#get-earthquakes)
  - [GET /earthquakes/[id]](#get-earthquakesid)
  - [POST /earthquakes/[id]/articles](#post-earthquakesidarticles)

### GET /earthquakes

Example: http://example.gc.ca/api/v1/earthquakes

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /earthquakes/[id]

Example: http://example.gc.ca/api/v1/earthquakes/[id]

    {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }

### POST /earthquakes/[id]/articles

Example: Create – POST  http://example.gc.ca/api/v1/earthquakes/[id]/articles

    {
        "title": "Raising Revenue",
        "author_first_name": "Jane",
        "author_last_name": "Smith",
        "author_email": "jane.smith@example.gc.ca",
        "year": "2012"
        "month": "August"
        "day": "18"
        "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "

    }
