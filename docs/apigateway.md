# Exposing your Actions via API Gateway (Experimental)

In OpenWhisk you can invoke your actions using the [REST API](./reference.md#rest-api) using an HTTP Request only via a __POST__ method.
This requires the HTTP client to make the request using the OpenWhisk authorization API key which is a master
key that in addition of allowing to invoke an action, also allows for deleting and creating more actions.

This experimental feature will allow you to invoke an action with HTTP methods other than POST and without the action's authorization API key.

Use the CLI to expose your OpenWhisk actions through the OpenWhisk API Gateway. 

## OpenWhisk CLI configuration
This experimental feature only works with the new OpenWhisk authentication model in which each namespace now has a unique authentication key associated with it.
Follow instructions in [Configure CLI](../../docs/README.md#setting-up-the-openwhisk-cli) on how to set the authentication key for your specific namespace.


## Expose an OpenWhisk action

Let's expose a simple action that is already pre-installed with OpenWhisk

```
$ wsk api-experimental create /hello /echo get /whisk.system/utils/echo
```
```
ok: created api /echo GET for action /whisk.system/utils/echo
https://21ef035.api-gw.mybluemix.net/hello/echo
```
A new URL is generated exposing the `echo` action via a __GET__ HTTP method.

Let's give it a try by sending a HTTP request to the URL.
```
$ curl https://21ef035.api-gw.mybluemix.net/hello/echo?marco=polo
```
This will invoke the `echo` action, returning back a JSON string with the parameters sent.
```
{
  "marco":"polo"
}
```

You can pass parameters to the action via simple query parameters, or via request body.

### Exposing multiple actions

Let's say you want to expose a set of actions for a book club for your friends.
You have a series of actions to implement your backend for the book club:

| action | http method | description |
| ----------- | ----------- | ------------ |
| getBooks    | GET | get book details  |
| postBooks   | POST | adds a book |
| putBooks    | PUT | updates book details |
| deleteBooks | DELETE | deletes a book |

Let's create an API for the book club, named `Book Club`, with `/club` as its HTTP URL base path and `books` as its resource.
```
$ wsk api-experimental create -n "Book Club" /club /books get getBooks
$ wsk api-experimental create /club /books post postBooks
$ wsk api-experimental create /club /books put putBooks
$ wsk api-experimental create /club /books delete deleteBooks
```

Notice that the first action exposed with base path `/club` gets the API label with name `Book Club` any other actions exposed under `/club` will be associated with `Book Club`

Lets list all the actions that we just exposed.

```
$ wsk api-experimental list
ok: apis
Action                   Verb          API Name        URL
getBooks                   get         Book Club       https://2ef15285-gws.api-gw.mybluemix.net/club/books
postBooks                 post         Book Club       https://2ef15285-gws.api-gw.mybluemix.net/club/books
putBooks                   put         Book Club       https://2ef15285-gws.api-gw.mybluemix.net/club/books
deleteBooks             delete         Book Club       https://2ef15285-gws.api-gw.mybluemix.net/club/books
```

Now just for fun lets add a new book `JavaScript: The Good Parts` with a HTTP __POST__
```
$  curl -X POST -d '{"name":"JavaScript: The Good Parts", "isbn":"978-0596517748"}' https://2ef15285-gws.api-gw.mybluemix.net/club/books
```
```
{
  "result": "success"
}
```

Lets get a list of books using our action `getBooks` via HTTP __GET__
```
$  curl -X GET https://2ef15285-gws.api-gw.mybluemix.net/club/books
```
```
{
  "result": [{"name":"JavaScript: The Good Parts", "isbn":"978-0596517748"}]
}
```

You can delete all of the exposed URLs using either the base path `/club` or API name label `"Book Club"`:
```
$ wsk api-experimental delete /club
```
```
ok: deleted API /club
```

- **Note**: This feature is currently an experimental offering that enables users an early opportunity to try it out and provide feedback. The following feedback has already been received:
  - No ability to control HTTP access control for  Cross-Origin Resource Sharing (CORS).
  - Only Content Type `application/json` is supported for request and response.
  - No programatic way to control the response from the OpenWhisk action.
  - All OpenWhisk actions are expose via public access, no ability to configure a custom API key.
  - Path parameters are not supported, only query parameter and request body.
  - If the API is created without an API name, the name will be the base path and this cannot be changed
  - When re-creating APIs via input file, the API needs to be deleted first.
  - When exporting APIs this contain your OpenWhisk API key, this information is sensitive, no templating available.
