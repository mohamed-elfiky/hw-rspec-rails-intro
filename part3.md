## Part 3: Getting the first spec to pass

Whereas a ``bare'' `it` serves as a
placeholder for a yet-to-be-written example, an `it` accompanied
by a `do...end` block is an actual test case.
Modify the first example to look like this:

```ruby
it 'calls the model method that performs TMDb search' do
  post :search_tmdb, {:search_terms => 'hardware'}
end
```

This example shows that RSpec for Rails includes a method `post` that
simulates doing an HTTP POST to your app.  The hash argument
represents the exact contents of `params[]` that your app would see,
so if you are using a form helper (like `form_tag_helper`) to
construct the forms, you need to know what the HTML field names will
be so that you can pass the correct values in this hash.

When you run this spec (or `guard` runs it for you), it will fail
miserably, because there is no route that matches `POST /search_tmdb`
(and even if there was a route, there would be no controller action
defined to receive it).

** Fix these errors by adding the correct route to `config/routes.rb`
to match this POST, and adding a controller action with an empty body
to the Movies controller.  **  
Put another way, that one line of test code drove us to
ensure that our new controller method and the view it will ultimately
render have the correct names and have a matching route.

At this point RSpec reports Green for our first example, but that's not
really accurate because the example itself is incomplete: we
haven't actually checked whether `search_tmdb` in the controller calls a model method to
search TMDb, as the spec requires.  (We did this deliberately in order
to illustrate some of the mechanics necessary to get your first specs
running.  Usually, since each spec tends to be short, you'd complete a
spec before re-running your tests.) 

**This is the most important paragraph in this exercise:**  This test
is _not_ about whether the (not-yet-existing) model method does the
right thing.  Per our initial desiderata, all we are checking here is
that the controller tries to call that method, and passes it the
correct argument(s) (in this case, the value the user typed in the
form field).

So we will set up a double for the model method--the code we wish we had--and check that our
double is called.

The code we wish we had 
will be a class method, since finding movies in TMDb is 
a behavior related to movies
in general and not to a particular instance of a movie.
The code we wish we had would probably accept
a string representing what the user typed, and would probably return a
collection of matching instances as `Movie` objects.
If that method existed, our controller method might therefore
call it like this:

```ruby
@movies = Movie.find_in_tmdb(params[:search_terms])
```

So let's modify our controller test to verify that the controller
action would call such a method:

```ruby
it 'calls the model method that performs TMDb search' do
  fake_results = [double('movie1'), double('movie2')]
  expect(Movie).to receive(:find_in_tmdb).with('hardware').
    and_return(fake_results)
  post :search_tmdb, {:search_terms => 'hardware'}
end
```

Since `find_in_tmdb` is code we don't yet have, the goal here is to
``fake'' the 
behavior it would exhibit if we did have it.  
In particular, 
we use RSpec's `double` method 
to create an array of two ``test double'' `Movie` objects.  
Whereas a real `Movie` object would respond to methods like `title`
and `rating`, the test double would raise an exception if you called
any methods on it.  Given this fact, why would we use  doubles at
all?  The reason is to isolate these specs from the behavior of the
`Movie` class, which might have bugs of its own.  Mock objects, or
simply mocks, are like
puppets whose behavior we completely control, allowing us to
isolate unit tests 
from their collaborator classes and keep tests Independent (the I in
FIRST).
In fact, in RSpec, an alias for `double` is `mock`.  For clarity,
we recommend using `mock` when you're going to ask the
fake object to do things (we'll see this later), and `double` when you just need a
"warm body" that looks like the right kind of object but isn't expected
to exhibit any behaviors of the real object.

What happens when we run this spec?  RSpec will open the `Movie` class
and define a class method called `find_in_tmdb` whose only purpose
is to track whether it gets called, and if so, whether the right
arguments are passed.  Such a "fake" method is a type of double
sometimes called a _method stub_ or a _spy_ (because you can spy on
its behavior).  Critically, _if a method with the same name
  already existed_ in the `Movie` class, it would be 
temporarily "overwritten" or shadowed by the stub.
That's why in our case it doesn't matter that we
haven't written the "real" `find_in_tmdb`: it wouldn't get called anyway!

This use of `expect...to receive` to temporarily replace a ``real'' method
for testing purposes is an example of using a seam.

The next line (really a continuation of the `expect`) specifies that
`find_in_tmdb` should return the collection of doubles we set up in
line 6.  This completes the illusion of "the code we wish we had":
we're calling a method that doesn't yet exist, and supplying the
results we wish it would give if it existed!  If we omit `with`, RSpec
will still check that `find_in_tmdb` gets called, but won't check if
the arguments are what we expected.  If we omit `and_return`, the fake
method call will return `nil` rather than a value chosen by us.
(Technically, in this case it would be OK to omit `and_return`, since
this example isn't checking the return value, but we included it for
illustrative purposes.)

In any case, after each example is run, RSpec performs a _teardown_ step
that restores the classes to their original
condition, so if we wanted to perform these same fake-outs in other examples,
we'd need to specify them in each one (though we'll soon see a
way to DRY out such repetition).  This automatic teardown is another
important part of keeping tests
Independent. 

Now the test fails, but it fails for the right reason: we expected the
controller action `search_tmdb` to call a method named `find_in_tmdb`
in the `Movie` class, but it did not.  _Now_ we can modify the code in
`MoviesController#search_tmdb` to
actually make it pass the test:

```ruby
@movies = Movie.find_in_tmdb(params[:search_terms])
```

If TDD is new to you, this has been a lot to absorb, especially when
testing an app using a powerful  framework such as  Rails.  
Don't worry---now
that you have been exposed to the main concepts, the next round of specs
will go faster.  It takes a bit of faith to jump into this system, but
the reward is well worth it.  
Consider having a sandwich and reviewing the concepts in this part
before moving on.

<details>
  <summary>
  Why must the `receive` expectation  come
  **before** the `post` action in the test?
  </summary>
  <p><blockquote>
  As part of the test setup, the expectation needs to establish a
  method stub for `find_in_tmdb` that can be monitored to make sure it was
  called.  Since the `post` action is eventually going to result in calling
  `find_in_tmdb`, the double must be set up before the `post` 
  occurs.
  </blockquote></p>
</details>
<br />

