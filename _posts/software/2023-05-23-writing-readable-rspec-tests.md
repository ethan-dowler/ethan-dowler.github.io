---
layout: post
title: 'Software: Writing Readable RSpec Tests'
date: 2023-05-23 08:00 +0000
permalink: /software/writing-readable-rspec-tests
---

## Prerequisite

This article assumes you know the basics of the [**Ruby**](https://www.ruby-lang.org/) language and are familiar with [**RSpec**](https://rspec.info/). I will be covering my "framework" for writing readable tests that are easy to extend and re-use. All code examples use Ruby 3.2 syntax.

## Tests as Documentation

Using the `--format documentation` and `--color` option, we can see our test cases as full sentences. Adding the `--color` option (or `--colour` if that's your thing) makes the output really pop!

```bash
bundle exec rspec --format documentation --color
```

Or better yet, create a `.rspec` file in your repo that makes these options the default.

```bash
# .rspec
--format documentation
--color
```

Now, when you run `bundle exec rspec`, you'll get the documenation format and color automatically!

## Hey, That's Out of Context!

Everyone loves a good soundbite. They can be funny, informative, or rage-inducing. But you know what I hate about soundbites? They are taken out of context! When we hear a 10-15 second audio clip or video, we have no idea _why_ the person in the clip is saying what they're saying. The same goes for our tests.

We can't just test _what_ our code does, we need to know _when_ it does it. Said another way, we need to know **in what circumstance does this code behave this way**.

You may have seen tests like this:

**Figure A**

```ruby
describe Sun do
  describe "#visible?" do
    it "returns true" do
      # test code
    end

    it "returns false" do
      # test code
    end
  end
end
```

What?! The sun is both visible and _not_ visible? This makes no sense! Let's take a closer look:

**Figure B**

```ruby
describe Sun do
  describe "#visible?" do
    it "returns true" do
      sun = Sun.new(time_of_day: "12pm")

      expect(sun.visible?).to be(true)
    end

    it "returns false" do
      sun = Sun.new(time_of_day: "12am")

      expect(sun.visible?).to be(false)
    end
  end
end
```

Ok, _now_ it's clear that the key difference here is _when_ the sun is visible. We didn't know that until we looked at the test code. We don't want to have to look through the inner workings of our tests to know what we're testing.

## Enter Context

This helpful RSpec method allows us to give the reader of the code a detailed explanation of the context surrounding the test.

Let's write the same tests using `context` to define the circumstances of the test:

**Figure C**

```ruby
describe Sun do
  describe "#visible?" do
    context "when the time of day is 12pm" do
      it "returns true" do
        # test code
      end
    end

    context "when the time of day is 12am" do
      it "returns false" do
        # test code
      end
    end
  end
end
```

That's better, but we can still make some improvements. Just looking at the names of the contexts, we don't know how much is different between the two tests. What makes the time 12pm? Is that the only thing that's different?

Take this quote from a **Sciencing.com** article titled [_Why Should You Only Test for One Variable at a Time in an Experiment?_](https://sciencing.com/should-only-test-one-variable-time-experiment-11414533.html):

> Testing only one variable at a time lets you analyze the results of your experiment to see how much a single change affected the result. If you're testing two variables at a time, you won't be able to tell which variable was responsible for the result.

## Enter Let

We can define any change-able variables we need using `let`. If a nested context redefines a `let`, then RSpec is smart enough to use the nested `let`. In this way, we can redifine our variables, so we know exactly what is changing between contexts:

**Figure D**

```ruby
describe Sun do
  let(:sun) { Sun.new(time_of_day:) }

  describe "#visible?" do
    context "when the time of day is 12pm" do
      let(:time_of_day) { "12pm" }

      it "returns true" do
        expect(sun.visible?).to be(true)
      end
    end

    context "when the time of day is 12am" do
      let(:time_of_day) { "12am" }

      it "returns false" do
        expect(sun.visible?).to be(false)
      end
    end
  end
end
```

Notice anything about the `it` blocks? We are calling the _same method_ on the _same object_. The only difference is what we're expecting. _And_ we have very clearly defined the **circumstance in which the code should behave this way**.

As a bonus, when we run the test using `rspec --format documentation`, we get a clean output of how (and when!) our code should behave.

```
Sun
  #visible?
    when the time of day is 12pm
      returns true
    when the time of day is 12am
      returns false
```

Add the `--color` option for more pizazz.

## Semantic RSpec

Although the tests are now clear, readable, and serve as documentation, we still have a few minor improvements to make.

RSpec gives us the methods `subject` and `described_class` that we can use to make our intentions clearer. I believe they also help us stick to testing best practices by reminding us the role that each variable is playing.

Use `subject` define the object under test. In our example, this is the `sun` variable. We can update the `let(:sun)` line like this:

```ruby
subject(:sun) { Sun.new(time_of_day:) }
```

Similarly, `described_class` is automatically assumed to be the class given in the top-most `#describe` block. It ensures we're always referencing the class under test. We can refine our new `subject` line further like this:

```ruby
subject(:sun) { described_class.new(time_of_day:) }
```

The full test file:

**Figure E**

```ruby
describe Sun do
  subject(:sun) { described_class.new(time_of_day:) }

  describe "#visible?" do
    context "when the time of day is 12pm" do
      let(:time_of_day) { "12pm" }

      it "returns true" do
        expect(sun.visible?).to be(true)
      end
    end

    context "when the time of day is 12am" do
      let(:time_of_day) { "12am" }

      it "returns false" do
        expect(sun.visible?).to be(false)
      end
    end
  end
end
```

## Taking It Further

### One Caveat

It's important to note that this `Sun` class is only concerned with the visbility of the sun to a viewer _on Earth_. If we need to know the visibility of the sun on Mars, we may want to make separate `EarthSun` and `MarsSun` classes to encapsulate the logic for each planet.

### A New Requirement

But even our Earth-centric logic might expand. What if we need to distinguish between different parts of Earth? In Alaska, depending on the season, the sun might be visible all day or not visible at all during the day.

We can add new contexts for Alaska in summer and winter. This greatly increases the number of tests in our test suite, but notice how each context only changes one variable, and the individual tests all look the same. The only thing that's changing are the inputs. 

**Figure F**

```ruby
describe Sun do
  subject(:sun) { described_class.new(time_of_day:, position:, season:) }

  describe "#visible?" do
    context "when in Kansas" do
      let(:position) { Position.new(lat: 39.626945, long: -97.644008) }

      context "when the season is summer" do
        let(:season) { :summer }

        # PREVIOUS TEST CODE
      end

      context "when the season is winter" do
        let(:season) { :winter }

        # NOTE: the tests for winter can be the same as the tests for
        # summer since the logic for seasons is the same in the mainland
      end
    end

    context "when in Alaska" do
      let(:position) { Position.new(lat: 66.160507, long: -153.369141) }

      context "when the season is summer" do
        let(:season) { :summer }

        context "when the time of day is 12pm" do
          let(:time_of_day) { "12pm" }

          it "returns true" do
            expect(sun.visible?).to be(true)
          end
        end

        context "when the time of day is 12am" do
          let(:time_of_day) { "12am" }

          it "returns true" do
            expect(sun.visible?).to be(true)
          end
        end
      end

      context "when the season is winter" do
        let(:season) { :winter }

        context "when the time of day is 12pm" do
          let(:time_of_day) { "12pm" }

          it "returns false" do
            expect(sun.visible?).to be(false)
          end
        end

        context "when the time of day is 12am" do
          let(:time_of_day) { "12am" }

          it "returns false" do
            expect(sun.visible?).to be(false)
          end
        end
      end
    end
  end
end
```



## Practice on Your Own

Want to practice writing semantic, readable, self-documenting tests? Try wrting a test suite for the [Gilded Rose Kata](https://github.com/emilybache/GildedRose-Refactoring-Kata). This classic kata is a great place to practice writing tests because there's no way you'll be able to understand the code at first glance. Remember to use `context` and `let` to isolate and define exactly _when_ the code should behave the way it behaves. Don't peek at the source code. Write tests based soley on the [requirements](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/main/GildedRoseRequirements.txt).
