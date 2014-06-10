# HTTP API 设计指南
> 翻译自 `HTTP API Design Guide` [https://github.com/interagent/http-api-design](https://github.com/interagent/http-api-design)

## 前言

这篇指南介绍了大量的 http + json 的api设计风格，最初摘录自heroku平台的api设计指引 [Heroku 平台 API 指引](https://devcenter.heroku.com/articles/platform-api-reference).

这篇指南除了当前那些API，同时也适用于heroku平台新集成的api,我们希望那些在heroku之外的api设计者也感兴趣！

我们的目标是一致性，专注业务逻辑同时避免设计上的空想，我们一直在寻找一种良好的，统一的，显而易见的API设计方式，未必只有一种方式。

我们假设你熟悉基本的 http+json api 设计方式，但是，不一定包含在此篇指南里面的所有内容。

## 内容

*  [返回合适的状态码](#返回合适的状态码)
*  [提供全部可用的资源](#provide-full-resources-where-available)
*  [在请求的body体使用JSON数据](#accept-serialized-json-in-request-bodies)
*  [提供资源的唯一标识](#provide-resource-uuids)
*  [提供标准的时间戳](#provide-standard-timestamps)
*  [使用UTC时间ISO8601格式](#use-utc-times-formatted-in-iso8601)
*  [使用统一的路径格式](#use-consistent-path-formats)
*  [使用小写字母的路径和属性](#downcase-paths-and-attributes)
*  [Nest foreign key relations](#nest-foreign-key-relations)
*  [支持方便的无id间接引用](#support-non-id-dereferencing-for-convenience)
*  [构建错误信息](#generate-structured-errors)
*  [Support caching with Etags](#support-caching-with-etags)
*  [Trace requests with Request-Ids](#trace-requests-with-request-ids)
*  [Paginate with ranges](#paginate-with-ranges)
*  [Show rate limit status](#show-rate-limit-status)
*  [Version with Accepts header](#version-with-accepts-header)
*  [Provide machine-readable JSON schema](#provide-machine-readable-json-schema)
*  [Provide human-readable docs](#provide-human-readable-docs)
*  [Provide executable examples](#provide-executable-examples)
*  [Describe stability](#describe-stability)
*  [Require SSL](#require-ssl)
*  [Pretty-print JSON by default](#pretty-print-json-by-default)

### 返回合适的状态码

为每一次的响应返回合适的HTTP状态码. 成功的HTTP响应应该使用如下的状态码:

* `200`: `GET`请求成功, 以及`DELETE`或
  `PATCH` 同步请求完成
* `201`: `POST` 同步请求完成
* `202`: `POST`, `DELETE`, 或 `PATCH` 异步请求将要完成
* `206`: `GET` 请求成功, 这里只是一部分状态码: 更多 

对于用户请求的错误情况，及服务器的异常错误情况，请查阅完整的HTTP状态码[HTTP response code spec](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

### 提供全部可用的资源

提供全部可显现的资源 (例如. 这个对象的所有属性) 不用考虑全部可能响应的状态码，总是在响应码为200或是201时返回所有可用资源, 包含 `PUT`/`PATCH` 和 `DELETE`
请求, 例如:

```
$ curl -X DELETE \  
  https://service.com/apps/1f9b/domains/0fd4

HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
...
{
  "created_at": "2012-01-01T12:00:00Z",
  "hostname": "subdomain.example.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

当请求状态码为202时，不返回所有可用资源,
e.g.:

```
$ curl -X DELETE \  
  https://service.com/apps/1f9b/dynos/05bd

HTTP/1.1 202 Accepted
Content-Type: application/json;charset=utf-8
...
{}
```

### 在请求的body体使用JSON数据

在 `PUT`/`PATCH`/`POST` 请求的body体使用JSON格式数据, 而不是使用 form 表单形式的数据. 这里我们使用JSON格式的body请求创建对称的格式数据, 例如.:

```
$ curl -X POST https://service.com/apps \
    -H "Content-Type: application/json" \
    -d '{"name": "demoapp"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "demoapp",
  "owner": {
    "email": "username@example.com",
    "id": "01234567-89ab-cdef-0123-456789abcdef"
  },
  ...
}
```

### 提供资源的唯一标识

在默认情况给每一个资源一个`id`属性. 用此作为唯一标识除非你有更好的理由不用.不要使用那种在服务器上或是资源中不是全局唯一的标识，比如自动增长的id标识。

返回的唯一标识要用小写字母并加个分割线格式 `8-4-4-4-12`, 例如.:

```
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

### 提供标准的时间戳

提供默认的资源创建时间，更新时间 `created_at` and `updated_at` ,
e例如:

```json
{
  ...
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T13:00:00Z",
  ...
}
```

这些时间戳可能不适用于某些资源，这种情况下可以忽略省去。

### 使用ISO8601的国际化时间格式 

在接收的返回时间数据时只使用UTC格式. 查阅ISO8601时间格式,
例如:

```
"finished_at": "2012-01-01T12:00:00Z"
```

### 使用统一的资源路径

#### 资源命名

使用复制形式为资源命名，而不是在系统中有争议的一个单个资源 (例如，在大多数系统中，给定的用户帐户只有一个). 这种方式保持了特定资源的统一性.

#### 形为

好的末尾展现形式不许要指定特殊的资源形为，在某些情况下，指定特殊的资源的形为是必须的,用一个标准的`actions`前缀去替代他, 清楚的描述他:

```
/resources/:resource/actions/:action
```

例如.

```
/runs/{run_id}/actions/stop
```

### 路径和属性要用小写字母

使用小写字母并用`-`短线分割路径名字,并且紧跟着主机域名 e.g:

```
service-api.com/users
service-api.com/app-setups
```

同样属性也要用小写字母, 但是属性名字要用下划线`_`分割，以至于这样在javaScript语言中不用输入引号。 例如.:

```
service_class: "first"
```

### Nest foreign key relations

Serialize foreign key references with a nested object, e.g.:

```json
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0..."
  },
  ...
}
```
  
Instead of e.g:

```json
{
  "name": "service-production",
  "owner_id": "5d8201b0...",
  ...
}
```

This approach makes it possible to inline more information about the
related resource without having to change the structure of the response
or introduce more top-level response fields, e.g.:

```json
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0...",
    "name": "Alice",
    "email": "alice@heroku.com"
  },
  ...
}
```

### Support non-id dereferencing for convenience

In some cases it may be inconvenient for end-users to provide IDs to
identify a resource. For example, a user may think in terms of a Heroku
app name, but that app may be identified by a UUID. In these cases you
may want to accept both an id or name, e.g.:

```
$ curl https://service.com/apps/{app_id_or_name}
$ curl https://service.com/apps/97addcf0-c182
$ curl https://service.com/apps/www-prod
```

Do not accept only names to the exclusion of IDs.

### Generate structured errors

Generate consistent, structured response bodies on errors. Include a
machine-readable error `id`, a human-readable error `message`, and
optionally a `url` pointing the client to further information about the
error and how to resolve it, e.g.:

```
HTTP/1.1 429 Too Many Requests
```

```json
{
  "id":      "rate_limit",
  "message": "Account reached its API rate limit.",
  "url":     "https://docs.service.com/rate-limits"
}
```

Document your error format and the possible error `id`s that clients may
encounter.

### Support caching with Etags

Include an `ETag` header in all responses, identifying the specific
version of the returned resource. The user should be able to check for
staleness in their subsequent requests by supplying the value in the
`If-None-Match` header.

### Trace requests with Request-Ids

Include a `Request-Id` header in each API response, populated with a
UUID value. If both the server and client log these values, it will be
helpful for tracing and debugging requests.

### Paginate with Ranges

Paginate any responses that are liable to produce large amounts of data.
Use `Content-Range` headers to convey pagination requests. Follow the
example of the [Heroku Platform API on Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges)
for the details of request and response headers, status codes, limits,
ordering, and page-walking.

### Show rate limit status

Rate limit requests from clients to protect the health of the service
and maintain high service quality for other clients. You can use a
[token bucket algorithm](http://en.wikipedia.org/wiki/Token_bucket) to
quantify request limits.

Return the remaining number of request tokens with each request in the
`RateLimit-Remaining` response header.

### Version with Accepts header

Version the API from the start. Use the `Accepts` header to communicate
the version, along with a custom content type, e.g.:

```
Accept: application/vnd.heroku+json; version=3
```

Prefer not to have a default version, instead requiring clients to
explicitly peg their usage to a specific version.

### Minimize path nesting

In data models with nested parent/child resource relationships, paths
may become deeply nested, e.g.:

```
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}
```

Limit nesting depth by preferring to locate resources at the root
path. Use nesting to indicate scoped collections. For example, for the
case above where a dyno belongs to an app belongs to an org:

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

### Provide machine-readable JSON schema

Provide a machine-readable schema to exactly specify your API. Use
[prmd](https://github.com/interagent/prmd) to manage your schema, and ensure
it validates with `prmd verify`.

### Provide human-readable docs

Provide human-readable documentation that client developers can use to
understand your API.

If you create a schema with prmd as described above, you can easily
generate Markdown docs for all endpoints with with `prmd doc`.

In addition to endpoint details, provide an API overview with
information about:

* Authentication, including acquiring and using authentication tokens.
* API stability and versioning, including how to select the desired API
  version.
* Common request and response headers.
* Error serialization format.
* Examples of using the API with clients in different languages.

### Provide executable examples

Provide executable examples that users can type directly into their
terminals to see working API calls. To the greatest extent possible,
these examples should be usable verbatim, to minimize the amount of
work a user needs to do to try the API, e.g.:

```
$ export TOKEN=... # acquire from dashboard
$ curl -is https://$TOKEN@service.com/users
```

If you use [prmd](https://github.com/interagent/prmd) to generate Markdown
docs, you will get examples for each endpoint for free.

### Describe stability

Describe the stability of your API or its various endpoints according to
its maturity and stability, e.g. with prototype/development/production
flags.

See the [Heroku API compatibility policy](https://devcenter.heroku.com/articles/api-compatibility-policy)
for a possible stability and change management approach.

Once your API is declared production-ready and stable, do not make
backwards incompatible changes within that API version. If you need to
make backwards-incompatible changes, create a new API with an
incremented version number.

### Require TLS

Require TLS to access the API, without exception. It’s not worth trying
to figure out or explain when it is OK to use TLS and when it’s not.
Just require TLS for everything.

### Pretty-print JSON by default

The first time a user sees your API is likely to be at the command line,
using curl. It’s much easier to understand API responses at the
command-line if they are pretty-printed. For the convenience of these
developers, pretty-print JSON responses, e.g.:

```json
{
  "beta": false,
  "email": "alice@heroku.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "last_login": "2012-01-01T12:00:00Z",
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

Instead of e.g.:

```json
{"beta":false,"email":"alice@heroku.com","id":"01234567-89ab-cdef-0123-456789abcdef","last_login":"2012-01-01T12:00:00Z", "created_at":"2012-01-01T12:00:00Z","updated_at":"2012-01-01T12:00:00Z"}
```

Be sure to include a trailing newline so that the user’s terminal prompt
isn’t obstructed.

For most APIs it will be fine performance-wise to pretty-print responses
all the time. You may consider for performance-sensitive APIs not
pretty-printing certain endpoints (e.g. very high traffic ones) or not
doing it for certain clients (e.g. ones known to be used by headless
programs).
