# resources-api-v1

[![CCCC draft](https://img.shields.io/badge/CCCC-draft-yellow.svg)][cccc]
[![Build Status](https://travis-ci.org/schul-cloud/resources-api-v1.svg?branch=master)][travis]
[![PyPI](https://img.shields.io/pypi/v/schul-cloud-resources-api-v1.svg)][pypi]
[![npm (scoped)](https://img.shields.io/npm/v/@schul-cloud/schul-cloud-resources-api-v1.svg)][npm]

The API specification to add content to the Schul-Cloud.
This is the [Resources API from the architecture][arch].
It should be the minimal example of what works for all content providers.

## Communication

- If you like to work on this on bring in new ideas, you can open [an issue][new-issue] and discuss with us.
- You can also join [the mailinglist][mailinglist].
  The purpose of the mailinglist is to have another form of communication available which is more accepted than Github.

The design process follows the [Collective Code Construction Contract][cccc].

## Repository Structure
[repository-structure]: #repository-structure

- [api-definition][api-definition]  
  Here, you can find the swagger API definition.
  You can try it out [on swaggerhub][swag1].
  The api definition is incomplete and uses the [resource json schema][resource-schema].
- [schemas][schemas]  
  The [resource schema][resource-schema] is defined there.
  If you want to see what a resource looks like, you can find examples there.
  The schema can be used to verify objects if they can be used as a resource.
  If you write your own [crawler] (LINK: TODO), this may come in handy.
  The examples are tested and allow test-driven development of the schema.
  If you have additional ideas about what a resource is, you can submit
  it to there.
- [generators][generators]  
  These scripts use the [swagger api definition][api-definition] to generate
  client and server libraries.
  Whenever the api changes, a [python library][python-library] is created and pushed to [PyPI][pypi].
  You can use this library to access and test servers and resources.
- [scripts][scripts] and [.travis.yml](.travis.yml)  
  These scripts are used to run the continuous integration tests of the api to ensure
  it does not contain some obvious mistakes.

## API
[api]: #api

These are the API endpoints defined in the [documentation][arch].

## Resources API
[resources]: #resources-api

The resources API is specified in the [api-definition][api-definition].
You can view it on [swaggerhub][swag1].
This API is tested and implemented by the [schul_cloud_resources_server_tests][rstest].
If you want to implement the API, please refer to the tests.

Errors follow the [error schema][error-schema].

### Authorization
The API only specifies how to authenticate.
Depending on the implementation, it differs where you get the authentication from.

A recommendation is that if you could not authenticate,
the server shows a page telling you where to get accepted credentials.

The api specifies authentication via api key and [basic authentication][basic-auth].
If you want to add another authentication mechanism, please [open an issue][new-issue].

It is clearly defined how to do [basic access authentication][basic-auth].
Instead of no authentication and basic authentication, 
the `Authorization` header can be set to support api key authentication.

Example:

    Authorization: api-key key=base64encodedkey

Because the ``Authorization`` header is used, one cannot authenticate with both,
an api key and basic authentication.

Further Reading:

- http://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-auth-using-authorization-header.html
- http://www.ietf.org/rfc/rfc2617.txt via http://stackoverflow.com/a/11420667/1320237
- https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html
- https://tools.ietf.org/html/rfc7235#section-4.2

### Search API
[search]: #search

This API is tested and implemented by the [schul_cloud_search_tests][stest].
If you want to implement the API, please refer to the tests.

- `GET /v1/search?Q=WORDS?filter[ATTRIBUTE]=...&filter[ATTRIBIUTE2]=...&page[offset]=SKIP?page[limit]=LIMIT`  
  Search for some words and filter the result.
  - **Q** (required)  
    The query string `WORDS`. They should search at least these object attributes:
    - title
    - content
  - **ATTRIBUTE** (optional)  
    To filter attributes of the objects, see [the jsonapi definitions][filter].
    Example: `filter[title]=my%20title`  
    All objects returned must have the filter applied.
    Filtering must work with strings. It can work with other types.
    If an attribute is filtered which is an array, the value must be inside the array.
    If an attribute is filtered but not present, the object is not chosen.
    Example:
    ```
    objects: [ {"license": ["MIT", "GPL"]}, {"license": ["MIT"]} ]
    query: `?filter[license]=GPL`
    result:  [ {"license": ["MIT", "GPL"]} ]
    ```
  - pagination according to [feathers](http://jsonapi.org/format/#fetching-pagination).
    - `page[offset]` (optional)   
      a positive integer  
      how many object shall be skipped.
      If more objects are skipped than are there, data is empty.
    - `page[limit]` (optional)
      how many object shall be returned.  
      This is the maximum number of objects that shall be returned.
      Less objects can be returned.

  Result:
  - See the [examples](schemas/search-response/examples/valid)
  - The [schema](schemas/search-response) for the body
  - Headers:
    - `Content-Type`: `application/vnd.api+json`
  - Errors, see the [error][error-schema] schema:
    - 400  
      also see [error object](http://jsonapi.org/format/#error-objects) on how to create informative errors about query parameters
      - [unprocessable query parameters](http://jsonapi.org/format/#query-parameters)
      - [sorting](http://jsonapi.org/format/#fetching-sorting)
        
        > If the server does not support sorting as specified in the query parameter sort, it MUST return 400 Bad Request.

Related Work:
- Inspiration: [feathers](https://docs.feathersjs.com/api/databases/querying.html)
- pagination inspired by [feathers](https://docs.feathersjs.com/api/databases/common.html#pagination).


## Research

DONE

- **LOM**  
  LOM shall be an inspiration. [Wikipedia](https://en.wikipedia.org/wiki/Learning_object_metadata)
- **Schema.org**  
  http://schema.org/docs/gs.html  
  These schemata are used on web-sites to mark parts the of HTML site as specific content.
  This is useful if the search engine outputs HTML.
  E.g. an author can be shown in the [Person](http://0.3-2e.schemaorgae.appspot.com/Person) schema.
  - schema.org: http://0.3-2e.schemaorgae.appspot.com/CreativeWork
    -> provider
- **Learning Object**  
  https://en.wikipedia.org/wiki/Learning_object
  - [IEEE LOM][ieee-lom]
    for 
    - Learning object interactivity
    - 7. LangString
      - `DateTime`
      - `Language` 
    - 5.2 `Learning Resource Type`
    - 5.1 `Interactivity Type` active, expositive, mixed
    - 5.9 Typical learning time
    - 5.8 difficulty
    - 5.7 typical age range

TODO

- bildungsserver, apache lucene, elixier - statt elastisearch
- Example: http://dsb-client.readthedocs.io/en/latest/descriptions.html

## Automated Build

The automated build is done by [Travis-CI][travis].
It does the following:

- Test the [schemas][schemas] for validity, usind the tests.
- Test the [api-definition][api-definition] for validity.
- Generate a [Python client][pypi]. The client library is used by the [server tests][rstest].

## Further Reading
- [README Driven Development][rdd]
- [HTTP statuses](https://httpstatuses.com/)
- [License Choice](http://hintjens.com/blog:41)


[rdd]: http://tom.preston-werner.com/2010/08/23/readme-driven-development.html
[arch]: https://schul-cloud.github.io/blog/2017-04-24/extensible-content-delivery#architecture
[content-crawl-api]: https://github.com/schul-cloud/schulcloud-content-crawler#attributes
[rfc2046]: https://tools.ietf.org/html/rfc2046
[ieee-lom]: http://129.115.100.158/txlor/docs/IEEE_LOM_1484_12_1_v1_Final_Draft.pdf
[swag1]: https://app.swaggerhub.com/apis/niccokunzmann/schul-cloud-content-api/1.0.0
[schemas]: ./schemas
[api-definition]: ./api-definition/
[pypi]: https://pypi.python.org/pypi/schul-cloud-resources-api-v1
[travis]: https://travis-ci.org/schul-cloud/resources-api-v1
[api-definition]: api-definition
[resource-schema]: schemas/resource
[generators]: generators
[scripts]: scripts
[python-library]: generators/python_client/
[rstest]: https://github.com/schul-cloud/schul_cloud_resources_server_tests
[new-issue]: https://github.com/schul-cloud/resources-api-v1/issues/new
[basic-auth]: https://en.wikipedia.org/wiki/Basic_access_authentication
[cccc]: https://rfc.zeromq.org/spec:42/C4/
[filter]: http://jsonapi.org/format/#fetching-filtering
[error-schema]: schemas/error
[mailinglist]: https://lists.hpi.uni-potsdam.de/listinfo/schul-cloud-content-dienste-offen
[stest]: https://github.com/schul-cloud/schul_cloud_search_tests
[npm]: https://www.npmjs.com/package/@schul-cloud/schul-cloud-resources-api-v1
