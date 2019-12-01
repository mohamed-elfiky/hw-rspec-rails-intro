## Part 5: Refactor

Before we write another example, we consider the Refactor step of
Red--Green--Refactor.  
Modify your controller spec file to look like this:

```ruby
require 'rails_helper'

describe MoviesController do
  describe 'searching TMDb' do
    before :each do
      @fake_results = [double('movie1'), double('movie2')]
    end
    it 'calls the model method that performs TMDb search' do
      expect(Movie).to receive(:find_in_tmdb).with('hardware').
        and_return(@fake_results)
      post :search_tmdb, {:search_terms => 'hardware'}
    end
    it 'selects the Search Results template for rendering' do
      allow(Movie).to receive(:find_in_tmdb).and_return(fake_results)
      post :search_tmdb, {:search_terms => 'hardware'}
      expect(response).to render_template('search_tmdb')
    end
    it 'makes the TMDb search results available to that template'
  end
end
```

As the new  code above implies, there is now a setup block that is
executed before **each** of the examples within the `describe` group,
similar to the `Background` section of a Cucumber feature,
whose steps are performed before each scenario.  
(There is also `before(:all)`, which runs setup code just once for
a whole group of tests; but you risk making your tests dependent on each
other by using it, since it's easy for hard-to-debug dependencies to
creep in that are 
only exposed when tests are run in a different order or when only a
subset of tests are run.)

While the concept of factoring out common setup into a `before` block
is straightforward, we had to make one syntactic change to make it
work, because of the way RSpec is implemented.
Specifically, we had to change `fake_results` into an instance
variable `@fake_results`,
because local variables occurring inside each example aren't visible
outside that example.
In contrast, instance variables of an example group are visible
to all examples in that group.
Since we are setting the value in the `before :each`
block, every test case will see the same initial value of
`@fake_results`.

<details>
  <summary>
  What kind of object do you think `@fake_results` is an instance
  variable of?
  </summary>
  <p><blockquote>
  It's an instance variable not of the class under test
  (`MoviesController`), but
  of the `Test::Spec::ExampleGroup` object that
  represents a group of test cases. 
  </blockquote></p>
</details>
<br />
