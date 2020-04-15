# Intro to RSpec and TDD

In this assignment you will take a deep dive into both the concepts and mechanics of doing test-driven development (TDD) with the 
RSpec testing framework, using the Guard tool to automatically re-run tests as you change code and using Webmock to provide "canned" responses for tests that exercise code that communicates with an external service.  The assignment is mostly a walkthrough step-by-step tutorial with self-check questions.  It's divided into the following parts:

0. Part 1: basic setup of RSpec and Guard, and how to manually and automatically run specs
0. Part 2: 

TBD

## Part 0: Setup

## The Code We Wish We Had

We will write unit and module-level tests using RSpec.  
(RSpec can also be used for integration tests,
but we prefer Cucumber for that task since it facilitates
dialogue with the customer and automates acceptance as well as 
integration tests.)  RSpec is a domain-specific
language (DSL) for testing Ruby code.  A DSL is a small programming
language designed to ease tackling problems within a single area
(domain) at the expense of generality.  You've already seen examples of
external (standalone) DSLs, such as HTML for describing Web
pages.  RSpec is an _internal_ or _embedded_ DSL: RSpec code
is just Ruby code, but takes advantage of Ruby's features and syntax so
as to make up a "mini-language" focused on the job of testing.
Regular expressions are another example of an internal DSL embedded in
Ruby.

RSpec's facilities help us capture _expectations_ of how our code
should behave.  Such tests are executable specifications or "specs"
written in Ruby, hence the name RSpec.

As the text notes, to capture expectations in tests before there is any code to be
tested, we write a test that exercises
the _code we wish we had_. This step forces us to think not only about
what the code will do, but how it will be used by its callers and
collaborators (other pieces of code that have to work with it).

In the BDD assignment, we augmented RottenPotatoes with a form
allowing the user to search The Open Movie Database (TMDb) for a movie
to add to RottenPotatoes.  Recall that in the MVC architecture, the
controller is the "first point of entry" to the app when the client
makes a request, so we had to pick a name for the controller action
that would eventually handle this form submission; we called the
controller action `search_tmdb`.
Of course, since no  such action exists in the `MoviesController`,
the Cucumber step
fails (showing red) when you tried to actually run the scenario. 
In the rest of this exercise we will use TDD to develop the controller method.

## Part 1: TDD drives creating route, view, and controller method


In the MVC architecture, the controller's job is to respond to a user
interaction, call the appropriate model method(s) to retrieve or
manipulate any necessary data, and generate an appropriate view.  We
might therefore describe the _desired_ behavior of our
as-yet-nonexistent controller method as follows:

1. We expect that our controller method would call a model method to
perform the TMDb search, passing  it the search terms typed by the user.

2. Then, we expect the coontroller method to select the correct view template
(Show Search Results) for rendering.

3. Finally, we expect it to make the actual  search results available to that template.

Note that none of the methods or templates in this list of
desiderata actually exists yet!  That is the essence of TDD: write a
concrete and concise list of the desired behaviors (the spec), and use
it to drive the creation of the methods and templates.

In the root directory of a working solution to the [BDD
assignment](https://github.com/saasbook/hw-bdd-cucumber), 
edit the `Gemfile` to include the following gems  in the `test` section:

```ruby
gem 'themoviedb-api'  # in main group
group :test do
  gem 'byebug'
  gem 'rspec-rails', '>= 3.5.0'
  gem 'guard-rspec'
  gem 'rails-controller-testing'
end
```

Rerun `bundle install` to get the gems.  Then run `rails generate
rspec:install` to ensure the files needed by RSpec are in place.

Edit the file `spec/rails_helper.rb` to include `require 'byebug'` at
the top, so we can drop into the debugger as needed to get tests working.

Finally, create the file `spec/controllers/movies_controller_spec.rb`
containing the following, which corresponds to the three expectations
articulated above.  (By convention over configuration, 
RSpec expects the specs for `app/controllers/movies_controller.rb` to
live in `spec/controllers/movies_controller_spec.rb`; similarly for models.)


```ruby
require 'rails_helper'

describe MoviesController do
  describe 'searching TMDb' do
    it 'calls the model method that performs TMDb search'
    it 'selects the Search Results template for rendering'
    it 'makes the TMDb search results available to that template'
  end
end

```
    
Line 3 says that the following specs **describe** the behavior of the
`MoviesController` class.  Because this class has several
methods, line 4 says that this first set of specs describes the behavior
of the method that searches TMDb.  As you can see, 
`describe` can be followed by
either a class name or a descriptive documentation string, and
`describe` blocks can be nested.  As we'll see later, nesting allows
grouping sets of related examples that share some of the same setup or
teardown code.

The next three lines are placeholders for what RSpec calls **examples**, 
a short piece of code
that tests _one_ specific behavior.  We
haven't written any test code yet, but  we can
execute these test skeletons.

There are three ways to do so (try them now):

1. Running `bundle exec rspec `_filename_ runs the tests in a single file, such as
`movies_controller_spec.rb` above.

2. Running `bundle exec rspec` with no arguments runs all the tests.

3. Running `bundle exec guard init rspec` once will setup the files
necessary for the Guard tool to re-run tests automatically as files
change.  Do that now, and add the `Guardfile` to your repo.  Now you
can start `bundle exec guard`.
For the rest of the
exercise, we'll assume that `guard` is running and that whenever you
add tests or application code you will get immediate feedback from RSpec.

In any case, you can see that examples (`it` clauses) containing no code are
  displayed in yellow as "pending".

into the \C{search\_terms} field and submits the form.  As we know,
in a Rails app
the \C{params} hash is automatically populated with the data submitted
\index{params[]!Red-Green-Refactor TDD}
in a form so that the controller method can examine it.
Happily, RSpec provides a \C{post}
method that simulates posting a form to a controller action: the first
argument is the action name (controller method) that will receive the
post, and the second argument is a hash that will become the \C{params}
seen by the controller action.  We can now write the first line of our
first spec, as Figure~\ref{fig:list2} shows.  As the next screencast
shows, though, we must overcome a couple of hurdles just to get to the
Red phase of Red--Green--Refactor.
