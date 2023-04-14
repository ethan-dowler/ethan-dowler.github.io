---
layout: post
title: 'Writing Readable RSpec Tests'
date: 2023-05-01 08:00 +0000
permalink: /software/writing-readable-rspec-tests
---

INTRO

This article uses the Gilded Rose Kata in Ruby. [Source Code](https://github.com/ethan-dowler/gilded-rose-kata)

## Tests as Documentation

Using the `--format documentation` and `--color` option, we can see our test cases as full sentences. Adding the `--color` option (or `--colour` if that's your thing) makes the output really pop!

```bash
bundle exec rspec --format documentation --color
```

[INSERT PICTURE OF RSPEC DOCUMENTATION OUTPUT]

Or better yet, create a `.rspec` file in your repo that makes these options the default.

```bash
# .rspec
--format documentation
--color
```

Now, when you run `bundle exec rspec`, you'll get the documenation format and color automatically!

## Hey, That's Out of Context!

Everyone loves a good soundbite. They can be funny, informative, or rage-inducing. But you know what I hate about soundbites? They are taken out of context! When we hear a 10-15 second audio clip or video, we have no idea _why_ the person in the clip is saying/doing what they're doing. The same goes for our tests.

We can't just test _what_ our code does, we need to know _when_ it does it. Said another way, we need to know **under what circumstances does this code behave this way**.

Enter `context`. This helpful RSpec method allows us to give the reader of the code a detailed explanation of the context surrounding this test. Look at the `it` blocks in Figure A

**Figure A**

```ruby
describe Sun do
  describe "#visible?" do
    it "is visible" do
      # test code
    end

    it "is not visible" do
      # test code
    end
  end
end
```

What?! A sun is both visible and _not_ visible? This makes no sense! See the full test in Figure B.

Figure B
```ruby
describe Sun do
  describe "#visible?" do
    it "is visible" do
      sun = Sun.new(time_of_day: "12pm")

      expect(sun.bright?).to eq(true)
    end

    it "is not visible" do
      sun = Sun.new(time_of_day: "12am")

      expect(subject.bright?).to eq(false)
    end
  end
end
```

Ok, _now_ it's clear that the key difference here is _when_ the sun is visible. We didn't know that until we looked at the test code. Do you want to have to look through the inner workings of your test code

