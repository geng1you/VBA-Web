VBA-Web
=======

VBA-Web makes working with complex webservices and APIs easy with VBA on Windows and Mac. It includes support for authentication, automatically converting and parsing JSON, working with cookies and headers, and much more.

Getting started
---------------

- Download the [latest release (v4.0.0-rc.2)](https://github.com/VBA-tools/VBA-Web/releases)
- To install/upgrade in an existing file, use `VBA-Web - Installer.xlsm`
- To start from scratch on Excel, `VBA-Web - Blank.xlsm` has everything setup and ready to go

For more details see the [Wiki](https://github.com/VBA-tools/VBA-Web/wiki)

Examples
-------

The following examples demonstrate using the Google Maps API to get directions between two locations.

### GetJSON Example
```VB.net
Function GetDirections(Origin As String, Destination As String) As String
    ' Create a RestClient for executing requests
    ' and set a base url that all requests will be appended to
    Dim MapsClient As New RestClient
    MapsClient.BaseUrl = "https://maps.googleapis.com/maps/api/"
    
    ' Use GetJSON helper to execute simple request and work with response
    Dim Resource As String
    Dim Response As RestResponse
    
    Resource = "directions/json?origin=" & Origin & "&destination=" & Destination & "&sensor=false"
    Set Response = MapsClient.GetJSON(Resource)
    
    ' => GET https://maps.../api/directions/json?origin=...&destination=...&sensor=false
    
    ProcessDirections Response
End Function

Public Sub ProcessDirections(Response As RestResponse)
    If Response.StatusCode = Ok Then
        Dim Route As Dictionary
        Set Route = Response.Data("routes")(1)("legs")(1)

        Debug.Print "It will take " & Route("duration")("text") & _
            " to travel " & Route("distance")("text") & _
            " from " & Route("start_address") & _
            " to " & Route("end_address")
    Else
        Debug.Print "Error: " & Response.Content
    End If
End Sub
```

There are 3 primary components in Excel-REST: 

1. `RestRequest` for defining complex requests
2. `RestClient` for executing requests
3. `RestResponse` for dealing with responses. 
 
In the above example, the request is fairly simple, so we can skip creating a `RestRequest` and instead use the `Client.GetJSON` helper to GET json from a specific url. In processing the response, we can look at the `StatusCode` to make sure the request succeeded and then use the parsed json in the `Data` parameter to extract complex information from the response. 

### RestRequest Example

If we wish to have more control over the request, the following example uses `RestRequest` to define a complex request.

```VB.net
Function GetDirections(Origin As String, Destination As String) As String
    Dim MapsClient As New RestClient
    ' ... Setup client using GetJSON Example
    
    ' Create a RestRequest for getting directions
    Dim DirectionsRequest As New RestRequest
    DirectionsRequest.Resource = "directions/{format}"
    DirectionsRequest.Method = httpGET
    
    ' Set the request format -> Sets {format} segment, content-types, and parses the response
    DirectionsRequest.Format = json
    
    ' (Alternatively, replace {format} segment directly)
    DirectionsRequest.AddUrlSegment "format", "json"
    
    ' Add parameters to the request (as querystring for GET calls and body otherwise)
    DirectionsRequest.AddParameter "origin", Origin
    DirectionsRequest.AddParameter "destination", Destination
    
    ' Force parameter as querystring for all requests
    DirectionsRequest.AddQuerystringParam "sensor", "false"
    
    ' => GET https://maps.../api/directions/json?origin=...&destination=...&sensor=false
    
    ' Execute the request and work with the response
    Dim Response As RestResponse
    Set Response = MapsClient.Execute(DirectionsRequest)
    
    ProcessDirections Response
End Function

Public Sub ProcessDirections(Response As RestResponse)
    ' ... Same as previous examples
End Sub
```

The above example demonstrates some of the powerful feature available with `RestRequest`. Some of the features include:

- Url segments (Replace {segment} in resource with value)
- Method (GET, POST, PUT, PATCH, DELETE)
- Format (json and url-encoded) for content-type and converting/parsing request and response
- Parameters and QuerystringParams
- Body
- Cookies
- Headers

For more details, see the `RestRequest` page in with [Wiki](https://github.com/VBA-tools/VBA-Web/wiki/RestRequest)

### Authentication Example

The following example demonstrates using an authenticator with Excel-REST to query Twitter. The `TwitterAuthenticator` (found in the `authenticators/` [folder](https://github.com/VBA-tools/VBA-Web/tree/master/authenticators)) uses Twitter's OAuth 1.0a authentication and details of how it was created can be found in the [Wiki](https://github.com/VBA-tools/VBA-Web/wiki/Implementing-your-own-IAuthenticator).

```VB.net
Function QueryTwitter(query As String) As RestResponse
    Dim TwitterClient As New RestClient
    TwitterClient.BaseUrl = "https://api.twitter.com/1.1/"
    
    ' Setup authenticator
    Dim TwitterAuth As New TwitterAuthenticator
    TwitterAuth.Setup _
        ConsumerKey:="Your consumer key", _
        ConsumerSecret:="Your consumer secret"
    Set TwitterClient.Authenticator = TwitterAUth
    
    ' Setup query request
    Dim Request As New RestRequest
    Request.Resource = "search/tweets.{format}"
    Request.Format = json
    Request.Method = httpGET
    Request.AddParameter "q", query
    Request.AddParameter "lang", "en"
    Request.AddParameter "count", 20
    
    ' => GET https://api.twitter.com/1.1/search/tweets.json?q=...&lang=en&count=20
    '    Authorization Bearer Token... (received and added automatically via TwitterAuthenticator)
    
    Set QueryTwitter = TwitterClient.Execute(Request)
End Function
```

For more details, check out the [Wiki](https://github.com/VBA-tools/VBA-Web/wiki) and [Examples](https://github.com/VBA-tools/VBA-Web/tree/master/examples)

### Release Notes

#### 4.0.0

- Mac support!
- Custom converters
- Switch to `WinHttpRequest`
- Switch to [VBA-tools/VBA-JSON](https://github.com/VBA-tools/VBA-JSON)

#### 3.1.0

- Add `Request.RequestFormat`, `Request.ResponseFormat`, and `Request.Accept` for setting separate request and response formats (e.g. form-urlencoded request with json response)
- Add `LogRequest` and `LogResponse` for better logging detail (enable with `RestHelpers.EnableLogging = True`)
- Allow headers and content-type to be set in authenticator `BeforeExecute`
- __3.1.1__ Fix importing class incorrectly as module bug
- __3.1.2__ Add XML and plain text formats
- __3.1.3__ Fix hard dependency for XML
- __3.1.4__ Fix logging in `PrepareProxyForHttpRequest`

#### 3.0.0

- Add `Client.GetJSON` and `Client.PostJSON` helpers to GET and POST JSON without setting up request
- Add `AfterExecute` to `IAuthenticator` (Breaking change, all IAuthenticators must implement this new method)
- __3.0.1__ Add `DigestAuthenticator`, new helpers, and cleanup
- __3.0.2__ Switch timeout to `Long` and remove `RestClientBase` (out of sync with v3)
- __3.0.3__ Update OAuth1, deprecate `IncludeCacheBreaker`, update True/False formatting to lowercase, add LinkedIn example
- __3.0.4__ Fix formatting of parameters with spaces for OAuth1 and add logging
- __3.0.5__ Allow Array and Collection for Body in `Request.AddBody` and `Client.PostJSON`
- __3.0.6__ Convert Empty to `null` for json
- __3.0.7__ Add `install.bat` script for easy installation and upgrade

#### 2.3.0

- Add `form-urlencoded` format and helpers
- Combine Body + Parameters and Querystring + Parameters with priority given to Body or Querystring, respectively

#### 2.2.0

- Add cookies support with `Request.AddCookie(key, value)` and `Response.Cookies`
- __2.2.1__ Add `Response.Headers` collection of response headers

#### 2.1.0

- Add Microsoft Scripting Runtime dependency (for Dictionary support)
- Add `RestClient.SetProxy` for use in proxy environments
- __2.1.1__ Use `Val` for number parsing in locale-dependent settings
- __2.1.2__ Add raw binary `Body` to `RestResponse` for handling files (thanks [@berkus](https://github.com/berkus))
- __2.1.3__ Bugfixes and refactor

#### 2.0.0

- Remove JSONLib dependency (merged with RestHelpers)
- Add RestClientBase for future use with extension for single-client applications
- Add build scripts for import/export
- New specs and bugfixes
- __2.0.1__ Handle duplicate keys when parsing json
- __2.0.2__ Add Content-Length header and 408 status code for timeout

#### 1.1.0

Major Changes:

- Integrate Excel-TDD to fully test Excel-REST library
- Handle timeouts for sync and async requests
- Remove reference dependencies and use CreateObject instead

Bugfixes:

- Add cachebreaker as querystring param only
- Add Join helpers to resolve double-slash issue between base and resource url
- Only add "?" for querystring if querystring will be created and "?" isn't present
- Only put parameters in body if there are parameters

#### 0.2

- Add async support

### About

- Design based heavily on the awesome [RestSharp](http://restsharp.org/)
- Author: Tim Hall
- License: MIT
