# Part 2: Using the TMDb API


Like many SaaS applications, TMDb has a RESTful API (application
programming interface) that supports actions such as  ``search for movies matching
a keyword'' or ``retrieve detailed information about a specific
movie''.
To prevent abuse and track each user of the API separately, each developer
wishing to make calls to the API must first obtain their own _API key_
by requesting one via the TMDb 
website.  API requests that do not include a valid API key
are not honored, returning an error instead.
For request URI's containing a valid API key,
TMDb returns a JSON object as the result of the request encoded by the URI.  This
flow---construct a RESTful URI that may include an API key, receive a
JSON response---is a common pattern for interacting with
external services.

One way to call such an API  from Ruby would be to
use the `URI` class in the Ruby standard library to construct the
request URI, use the  `Net::HTTP` class
to issue the request to TMDb,
and parse the resulting JSON object.
But sometimes we can be more productive by standing on the shoulders of
others, as we can in this case.
The gem `themoviedb-api`
is a user-contributed Ruby "wrapper" around TMDb's RESTful
API, mentioned on the TMDb API documentation pages.  
Not every RESTful
API has a corresponding Ruby library, but for those that do, such as
TMDb, the library can hide the API details behind a few simple Ruby
methods.
In this case, the gem conveniently constructs the correct
RESTful URIs, performs the remote service calls, and parses the JSON
results into Ruby objects representing movies, playlists, and so on.

** TBD: Exercise:  get an API key and manually try out some requests
from the `irb` command line.**

<details>
  <summary>
    True or false: in order to use
  the TMDb API from another language such as Java, we would need a Java
  library equivalent to `themoviedb-api` gem.
  </summary>
  <p><blockquote>
   False:  the API consists of a set of HTTP requests and JSON responses,
   so as long as we can transmit and receive bytes over TCP/IP and have
   the ability to parse strings (the JSON responses), we can use the APIs
   without a special library.
  </blockquote></p>
</details>
<br />

<details>
  <summary>
    true or false: in order to use
  the tmdb api from another language such as java, we would need a java
  library equivalent to `themoviedb-api` gem.
  </summary>
  <p><blockquote>
   false:  the api consists of a set of http requests and json responses,
   so as long as we can transmit and receive bytes over tcp/ip and have
   the ability to parse strings (the json responses), we can use the apis
   without a special library.
  </blockquote></p>
</details>
<br />

