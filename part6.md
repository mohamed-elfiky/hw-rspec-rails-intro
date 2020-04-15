## Part 4: the final controller spec

There's just one example left to write, to check that the TMDb search
results will be made available to the response view.
The RSpec `assigns()` method keeps track of what instance variables
were assigned in the controller method.  Hence `assigns(:movies)`
returns whatever value (if any) was assigned to `@movies` by the
controller action, and our spec just has to verify that the controller
action correctly sets up this variable.  In our case, we've already
arranged to return our doubles as the result of the faked-out method
call, so the correct behavior for `search_tmdb` would be to set
`@movies` to this value.  Doing so and factoring out the common code
between the second and third specs into their own nested `describe`
gives us this final version of the controller spec.  We chose the
description string "after valid search" to name this nested `describe`
block because the examples in this subgroup all assume that a valid
call to `find_in_tmdb` has occurred; that assumption itself is tested
by the first example.  When example groups are nested, any `before`
blocks associated with the outer nesting are executed prior to those
associated with the inner nesting.  So, for example, considering the
test case in lines 18--20 below, the setup code in lines 5--7 is run
first, followed by the setup code in lines 14--17, and finally the
example itself (lines 18--20).


```ruby
  1 require 'rails_helper'
  2 
  3 describe MoviesController do
  4   describe 'searching TMDb' do
  5     before :each do
  6       @fake_results = [double('movie1'), double('movie2')]
  7     end
  8     it 'calls the model method that performs TMDb search' do
  9       expect(Movie).to receive(:find_in_tmdb).with('hardware').
 10         and_return(@fake_results)
 11       post :search_tmdb, {:search_terms => 'hardware'}
 12     end
 13     describe 'after valid search' do
 14       before :each do
 15         allow(Movie).to receive(:find_in_tmdb).and_return(@fake_results)
 16         post :search_tmdb, {:search_terms => 'hardware'}
 17       end
 18       it 'selects the Search Results template for rendering' do
 19         expect(response).to render_template('search_tmdb')
 20       end
 21       it 'makes the TMDb search results available to that template' do
 22         expect(assigns(:movies)).to eq(@fake_results)
 23       end
 24     end
 25   end
 26 end
```

Strictly speaking, for the purposes of this example the stubbed
`find_in_tmdb`
could have returned any value at all, such as the string "I am a
movie", because the _only_ behavior tested by this example
is whether the correct instance variable is being set up and made
available to the view.  In particular, this example doesn't care what
the value of that variable is.  But since we already had doubles set up, it was
easy enough to use them in this example.

**Summary of this part:**

*  An example of the Refactor step of Red--Green--Refactor is to move common setup 
    code into a `before` block, thus DRYing out your specs.
*  Method stubbing with `allow(...).to receive` creates a 
   "test double" method for use in tests, but unlike  `expect(...).to receive`, 
     doesn't require that the
     method actually be called.
* `assigns()`, an RSpec-Rails extension, allows a controller test to inspect
     the values of instance variables set by a controller action.


<details>
  <summary>
  Specify whether each
  of the following RSpec constructs is used to (a) 
  create a seam, (b)  determine the behavior of a seam, (c)
  neither:  (1) <code>assigns()</code> (2) <code>receive</code> (3) <code>expect</code> 
  (4) <code>and_return</code>
  </summary>
  <p><blockquote>
       (1) c, (2) a, (3) b, (4) b
  </blockquote></p>
</details>
<br />


<details>
  <summary>
  Why is it usually preferable to use <code>before(:each)</code> rather than
  <code>before(:all)</code> ?
  </summary>
  <p><blockquote>
    <code>before(:each)</code> is run before each spec in that
    block, setting up identical preconditions for all those specs and
    thereby keeping them Independent.
  </blockquote></p>
</details>
<br />
