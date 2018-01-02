# Swashbuckle.Examples
A simple library which adds the `[SwaggerRequestExample]`, `[SwaggerResponseExample]` attributes to [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle). Can also add read the `[Description]` attributes off your Response objects, and can also add an input box for entering an Authorization header.

Blog articles: https://mattfrear.com/2016/01/25/generating-swagger-example-requests-with-swashbuckle/
and: https://mattfrear.com/2015/04/21/generating-swagger-example-responses-with-swashbuckle/

## Request example

Populate swagger's `definitions.YourObject.example` with whatever object you like.

This is great for manually testing and demoing your API as it will prepopulate the request with some useful data, so that 
when you click the example request in order to populate the form, instead of getting an autogenerated request like this:

![autogenerated request with crappy data](https://mattfrear.files.wordpress.com/2016/01/untitled.png?w=700)

You’ll get your desired example, with useful valid data, like this:

![custom request with awesome data](https://mattfrear.files.wordpress.com/2016/01/capture2.jpg?w=700)

You can see the example output in the underlying swagger.json file, which you can get to by starting your solution and 
navigating to /swagger/docs/v1

![swagger.json](https://mattfrear.files.wordpress.com/2016/01/capture.jpg)

## Response example

Allows you to add custom data to the example response shown in Swagger. So instead of seeing the default boring data like so:

![response with crappy data](https://mattfrear.files.wordpress.com/2015/04/response-old.png)

You'll see some more realistic data (or whatever you want):

![response with awesome data](https://mattfrear.files.wordpress.com/2015/04/response-new.png?w=700&h=358)

## Documenting Response properties (new!)

Lets you add a comment-like description to properties on your response, e.g.
![descriptions](https://mattfrear.files.wordpress.com/2017/09/descriptions.jpg)

## Authorization header input box (new!)

Adds an input so that you can send an Authorization header to your API. Useful for API endpoints that have JWT token
authentication. e.g.

![authorization](https://mattfrear.files.wordpress.com/2017/09/authorization.jpg)

## Installation
Install the [NuGet package](https://www.nuget.org/packages/Swashbuckle.Examples/), then enable whichever filters 
you need when you enable Swagger:

```
GlobalConfiguration.Configuration 
    .EnableSwagger(c =>
        {
            c.SingleApiVersion("v1", "WebApi");

            // Enable Swagger examples
            c.OperationFilter<ExamplesOperationFilter>();

            // Enable swagger response descriptions
            c.OperationFilter<DescriptionOperationFilter>();
        })

```

## How to use - Request examples

Decorate your controller methods with the included SwaggerRequestExample attribute:

```
[SwaggerRequestExample(typeof(DeliveryOptionsSearchModel), typeof(DeliveryOptionsSearchModelExample))]
public async Task<IHttpActionResult> DeliveryOptionsForAddress(DeliveryOptionsSearchModel search)
```

Now implement it, in this case via a DeliveryOptionsSearchModelExample (which should implement IExamplesProvider), 
which will generate the example data. It should return the type you specified when you specified the `[SwaggerRequestExample]`.
	
```
public class DeliveryOptionsSearchModelExample : IExamplesProvider
{
    public object GetExamples()
    {
        return new DeliveryOptionsSearchModel
        {
            Lang = "en-GB",
            Currency = "GBP",
            Address = new AddressModel
            {
                Address1 = "1 Gwalior Road",
                Locality = "London",
                Country = "GB",
                PostalCode = "SW15 1NP"
            },
            Items = new[]
            {
                new ItemModel
                {
                    ItemId = "ABCD",
                    ItemType = ItemType.Product,
                    Price = 20,
                    Quantity = 1,
                    RestrictedCountries = new[] { "US" }
                }
            }
        };
    }
```

In the Swagger document, this will populate the request's schema object [example property](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#schemaExample).
The spec for this says:

Field Name | Type | Description
---|:---:|---
example | Any | A free-form property to include an example of an instance for this schema.

### List Request examples
As of version 3.5, `List<T>` request examples are supported. For any `List<T>` in the request, you may define a SwaggerRequestExample for `T`. 
Your IExamplesProvider should only return a single `T` and not a `List<T>`.
Working example:

```
[SwaggerRequestExample(typeof(PeopleRequest), typeof(ListPeopleRequestExample), jsonConverter: typeof(StringEnumConverter))]
public IHttpActionResult GetPersonList(List<PeopleRequest> peopleRequest)
{

// and then:

public class ListPeopleRequestExample : IExamplesProvider
{
    public object GetExamples()
    {
        return new PeopleRequest { Title = Title.Mr, Age = 24, FirstName = "Dave in a list", Income = null };
    }
}

```

## How to use - Response examples

Decorate your methods with the new SwaggerResponseExample attribute:
```
[SwaggerResponse(HttpStatusCode.OK, Type=typeof(IEnumerable<Country>))]
[SwaggerResponseExample(HttpStatusCode.OK, typeof(CountryExamples))]
[SwaggerResponse(HttpStatusCode.BadRequest, Type = typeof(IEnumerable<ErrorResource>))]
public async Task<HttpResponseMessage> Get(string lang)
```

Now you’ll need to add an Examples class, which will implement IExamplesProvider to generate the example data

```	
public class CountryExamples : IExamplesProvider
{
    public object GetExamples()
    {
        return new List<Country>
        {
            new Country { Code = "AA", Name = "Test Country" },
            new Country { Code = "BB", Name = "And another" }
        };
    }
}
```

In the Swagger document, this will populate the response's [example object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#exampleObject).
The spec for this says:

Field Pattern | Type | Description
---|:---:|---
{mime type} | Any | The name of the property MUST be one of the Operation produces values (either implicit or inherited). The value SHOULD be an example of what such a response would look like.

Example response for application/json mimetype of a Pet data type:

```js
{
  "application/json": {
    "name": "Puma",
    "type": "Dog",
    "color": "Black",
    "gender": "Female",
    "breed": "Mixed"
  }
}
```

Note that this differs from the Request example in that the mime type is a required property on the response example but not so on the request example.

### Known issues
- Although you can add a response examples for each HTTP status code (200, 201, 400, 404 etc), and they will all appear in the
swagger.json, **only one example for responses will display on the swagger-ui page**. This is due to a bug in swagger-ui. [Issue 9](https://github.com/mattfrear/Swashbuckle.AspNetCore.Examples/issues/9) You may want to use Swashbuckle's  `[SwaggerResponseRemoveDefaults]` attribute to remove 200 from your list of response codes, if for example your API returns 201s and not 200s [Issue 26](https://github.com/mattfrear/Swashbuckle.AspNetCore.Examples/issues/26)

- The response example is displayed wrapped in a JSON object which has the media type, i.e. "application/json" as the key, and the example as the value. 
[Issue 8](https://github.com/mattfrear/Swashbuckle.Examples/issues/8) Our response example is correct as per the Swagger spec, so I'm not sure 
why it is being displayed incorrectly - I suspect it's a bug in swagger-ui, as this didn't happen with older versions of Swashbuckle.

- Response examples which are only a simple type, i.e. string, int, etc and not an object may cause the swagger-ui page to stop rendering correctly. See [Issue 25](https://github.com/mattfrear/Swashbuckle.Examples/issues/25) for a possible workaround.

- Request examples are only supported when the request parameter is in the body of the request, and not on the querystring. This is a limitation of the Swagger 2.0 spec.

- For requests, in the Swagger 2.0 spec there is only one schema for each request object defined across all the API endpoints. So if you are using the same request object in multiple API endpoints,
i.e. on multiple controller actions like this:

```
DeliveryOptions.cs
public async Task<IHttpActionResult> DeliveryOptionsForAddress(DeliveryOptionsSearchModel search)
...

// maybe in some other controller, e.g. Search.cs
public async Task<IHttpActionResult> Search(DeliveryOptionsSearchModel search)
```

That DeliveryOptionsSearchModel object is only defined once in the entire Swagger document and it can only have one **request** example defined.

## How to use - Document response properties
Define the SwaggerResponse, as usual:
```
[HttpPost]
[Route("api/values/person")]
[SwaggerResponse(200, typeof(PersonResponse), "Successfully found the person")]
public PersonResponse GetPerson([FromBody]PersonRequest personRequest)
{
```
Now add `System.ComponentModel.Description` attributes to your Response object:
```
public class PersonResponse
{
	public int Id { get; set; }

	[Description("The first name of the person")]
	public string FirstName { get; set; }

	public string LastName { get; set; }

	[Description("His age, in years")]
	public int Age { get; set; }
```

## How to use - Authorization input

Just enable the `AuthorizationInputOperationFilter` as described in the Installation section above. Note this this will add an
Authorization input to EVERY endpoint, regardless of if the endpoint is actually secured.

## Pascal case or Camel case?
The default is camelCase. If you want PascalCase you can pass in a `DefaultContractResolver` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), typeof(DefaultContractResolver))]`

## Render Enums as strings
By default `enum`s will output their integer values. If you want to output strings you can pass in a `StringEnumConverter` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), jsonConverter: typeof(StringEnumConverter))]`
