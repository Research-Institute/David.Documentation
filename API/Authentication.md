# Authentication

Authentication is done using the [OpenId Connect](http://openid.net/connect/) spec (an extension to [OAuth2](https://tools.ietf.org/html/rfc6749)) using the [openiddict-core](https://github.com/openiddict/openiddict-core) package.

We currently support Client Credentials and Password Flow.

# Authorization

Resource authorization is implemented at the service layer using [JsonApiDotnetCore](https://github.com/Research-Institute/json-api-dotnet-core).
