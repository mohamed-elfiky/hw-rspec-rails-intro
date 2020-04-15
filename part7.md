## Part 7: TDD for the Model


Our next task will be to use TDD to create the model method
`find_in_tmdb` that we've so far been stubbing out.  Since this method is
supposed to call the actual TMDb service, we will again need to use
stubbing, this time to avoid having our examples depend on the behavior of
a remote Internet service.

Like many SaaS applications, TMDb has a RESTful API that supports
actions such as  "search for movies matching 
a keyword" or "retrieve detailed information about a specific
movie".
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




By convention over configuration, specs for the `Movie`
model go in `spec/models/movie_spec.rb`.
Here is the happy path for calling `find_in_tmdb` in the form of a
model spec, and the one line of code necessary in the `Movie` class to
make it pass.

```ruby
require 'rails_helper'

describe Movie do
  describe 'searching Tmdb by keyword' do
    it 'calls Tmdb gem with title keywords' do
      expect(Tmdb::Search).to receive(:movie).with('Inception')
      Movie.find_in_tmdb('Inception')
    end
  end
end
```

```ruby
class Movie < ActiveRecord::Base

  def self.find_in_tmdb(string)
    Tmdb::Search.movie(string)
  end

  # rest of file elided for brevity
end
```

You might wonder why the controller doesn't just call the
`Tmdb::Search.movie` method itself, rather than having that method
"wrapped" in a model method `Movie.find_in_tmdb`.  
There are two reasons.  First, if the TMDb gem's API changes, perhaps
to accommodate a change to the TMDb service API itself, we can
insulate the controller from those changes because all the knowledge
of how to use the gem to communicate with the service is encapsulated
inside the `Movie` class.
This indirection is an example of
separating things that change from those that stay the same.
The second and more important reason is
that this spec is subtly incomplete: `find_in_tmdb` has additional jobs to
do.  Our test cases have been based on the _explicit requirement_
that the user-provided name of a movie should be used to query TMDb.
But in fact, querying TMDb requires a valid API key.


* Todo: what if key invalid or not given?

The revised spec in Figure~\ref{fig:api_key_exception} expresses this
implicit requirement as a new spec.
Note that we have created `context` blocks to separate specs related to
the happy path and specs related to the sad path.  `context` is just
an alias for `describe` to make specs more readable, just as
`Given/When/Then` are all aliases to the same method in Cucumber.
Future happy-path specs can go inside the first group and future
sad-path specs can go inside the second group.


This leads to a new \x{implicit requirement} that we discovered while
experimenting with the gem: "It should raise an "invalid key"
exception if an invalid key is provided." 


Unfortunately, this spec will actually call the real TMDb
service every time it is executed, making the spec neither **F**ast
(each call takes a few seconds to complete) nor **R**epeatable (the test
will behave differently if TMDb is down or your computer is not
connected to the Internet).
Even if you only ran tests while connected to the Internet, it is very
bad etiquette to have your tests constantly contacting a production
service.

We can fix this by introducing a seam that isolates the caller from the
callee.
We know from Screencast~\ref{tmdb_gem_v4} that when an invalid key is
used, `Tmdb::Movie.find` raises `Tmdb::InvalidApiKeyError`,
  
so we mimic that behavior with a stub,
as line~10 of Figure~\ref{fig:movie_spec_3} shows.
Observe that the argument of `expect` in line 11---the call to
\modmeth---is in curly braces, the shorthand for a `do\ldots{`end}
block.  This is because
we _expect_ the call to raise an exception, but if a spec
actually raises an exception, it stops the testing run!
So in order to make the spec **S**elf-checking, we "encapsulate"
the code that will raise in exception in a `do\ldots{`end}---that is,
in a block---that RSpec can execute in a controlled environment in
which it can catch any 
exceptions and match them to our expectation.
  

This spec fails for the right reason, that is, because we
haven't added code to \modmeth{} to check for an exception in the gem.
Figure~\ref{fig:movie_3} shows the new code added to \modmeth{} to
make the spec pass.



```ruby
require 'rails_helper'

describe Movie do
  describe 'searching Tmdb by keyword' do
    context 'with valid API key' do
      # elided for brevity
    end
    context 'with invalid API key' do
      before(:each) do
        allow(Tmdb::Movie).to receive(:find).and_raise(Tmdb::InvalidApiKeyError)
      end
      # all specs in this block can take advantage of line 10's stub
      it 'raises an InvalidKeyError' do
        expect { Movie.find_in_tmdb('Inception') }.
          to raise_error(Movie::InvalidKeyError)
      end
      # other sad-path specs here...
    end
  end
end
```


\begin{checkyourself}
  Given that failing to initialize a valid API key causes \T{themoviedb}
  gem to raise an exception, why
  doesn't line~8 of Figure~\ref{fig:api_key_exception} raise an
  exception? 
  \begin{answer}
     Line~7 replaces the \C{Tmdb::Movie.find} call with a stub,
     preventing the "real" method from executing and raising an exception.
  \end{answer}  
\end{checkyourself}


\begin{checkyourself}
  Considering line 13 of Figure~\ref{fig:api_key_exception}, suppose
  we didn't wrap the call to \modmeth\ in a lambda-expression using
  curly braces.
  What would happen and why?
  \begin{answer}
    If \modmeth{} behaves correctly and raises the exception, the spec will
    still fail because the exception will stop the run.  If \modmeth{}
    behaves incorrectly and fails to raise an exception, the spec will fail because
    the assertion matcher \C{raise\_error} expects one.  Therefore the
    test would always fail whether \modmeth{} was behaving correctly or not.
  \end{answer}
\end{checkyourself}

* TOdo: happy path: what does Tmdb gem return?  How to convert into
list of matches?

* Todo: what if no results match?




[The Movie DB API wrapper
gem](https://github.com/18Months/themoviedb-api)

