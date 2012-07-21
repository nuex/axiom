# Axiom

Axiom is a lightweight web framework, inspired by
[Sinatra](http://sinatrarb.com) and built on top of
[Cowboy](https://github.com/extend/cowboy).

## Getting started

Axiom is built to make creating web applications fast and easy.
A minimal application would look like this:

```erlang
-module(my_app).
-export([start/0, handle/3]).

start() -> axiom:start(?MODULE).

handle('GET', [<<"hi">>], _Request) ->
	<<"Hello world!">>.

```

This handles requests for `GET /hi` and returns "Hello world!".

The third argument, given to the handler contains a proplist of things
we know about the request, such as `params`, `headers` and many more.

For convenience, the return value can be a binary string like in the
example. There are two ways, though,  to be more specific about the
response. The first is to use the `response` record. For that to work
you need to include Axiom's response header file:

```erlang
-include_lib("axiom/include/response.hrl").
```

Then, in your handler specify the body, the status and/or some HTTP
headers:

```erlang
handle('GET', [<<"foo">>], _Request) ->
	Response = #response{},
	Headers = Response#response.headers ++ [{'Set-Cookie', "ping=pong"}],
	#response{headers = Headers, body = <<"Go away!">>}.
```

The `response` record defines sane defaults for all the fields, so you
don't need to specify every one of them:

```erlang
-record(response, {status = 200, headers = [{'Content-Type', "text/html"}], body = <<"">>}).
```

The other way to specify the response is to put the above params into a
proplist:

```erlang
[{status, 403}, {body, <<"Go away!">>}, {headers, []}]
```

Again, you don't need to specify all of them, sane defaults also apply
to the proplist-way.

## Configuration

`axiom:start/1` has a bigger brother called `axiom:start/2`, taking a
proplist as the second argument. Possible properties and their defaults
are as follows:

```erlang
[
	{nb_acceptors: 100},	% acceptor pool size
	{host, '_'},		% host IP
	{port, 7654}		% host port
]
```

## Installation

To use it in your OTP application, add this to your `rebar.config`:

```erlang
{lib_dirs, ["deps"]}.
{deps, [
	{'axiom', "0.0.2", {git, "git://github.com/tsujigiri/axiom.git", {tag, "v0.0.2"}}}
]}.
```

then, as usual:

```
rebar get-deps
rebar compile
```

## Templating

Axiom comes with [Django](https://github.com/django/django) template
support via [erlydtl](https://github.com/evanmiller/erlydtl). To make
use of it in your application, create a directory named `templates` and
in it, create a template, e.g. `my_template.dtl`:

```dtl
<h1>Hello {{who}}</h1>
```

In your handler, specify the template to be rendered:

```erlang
handle('GET', [<<"hello">>], _Request) ->
	axiom:dtl(my_template, [{who, "you"}]).
```

For convenience, the second argument, a proplist of parameters, can have
atoms, lists or binaries as keys. That way request parameters can be put
in there, without you having to convert them first.

The templates are compiled into modules when `rebar compile` is
called.

To see what else erlydtl can do for you, take a look at
[its project page](https://code.google.com/p/erlydtl/).

## License

Please take a look at the `LICENSE` file! (tl;dr: it's the MIT License)
