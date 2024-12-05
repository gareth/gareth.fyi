---
layout: post
title:  "Advent of Code: Tracking reports"
date:   2024-12-05
categories: ruby advent-of-code
excerpt_separator: <!--more-->
---

There's not a huge amount to say about Advent of Code day 2. The puzzle is straightforward, but the way I approached the difference between the parts is a little more interesting.

<!--more-->

## Day 2

As ever, I'm not going to repeat the whole problem here, check the [Advent of Code problem page](https://adventofcode.com/2024/day/2) and my [overall solution](https://github.com/gareth/advent-of-code-2024/tree/main/day-02) for full details.

For the first part, I wrapped up the logic to calculate whether a "Report" was "safe" or not by detecting the various ways it could fail and checking if there was no error:

```ruby
class Report
  def initialize(levels)
    @levels = levels
  end

  def error
    # […]
  end

  def safe?
    !error
  end
end

```

The second part of the problem iterates on the first, by changing the definition of "safe" to say that a report is safe if removing any one of its levels turns it into a safe report using the previous definition.

If I was only interested in finding the answers to each problem, I would just edit the original source code. I guarantee the people solving these problems [within 2 minutes](https://adventofcode.com/2024/leaderboard/day/2) of them being released aren't hanging around restructuring their code. But that's not how I like to approach Advent of Code.

1. Both parts of the problem should stay executable - the second part shouldn't break the first
2. The code for the second part should (if possible) highlight the difference between the first part and the second.

This is a useful approach to practice, because in a real-world system you often don't have the luxury of being able to break code - other classes might depend on the existing functionality and shouldn't change.

So I treated day 2 like a simple refactoring: Our new report is `safe?` if it's safe by the previous definition *or* if we can make another report by removing one level.

```ruby
class FixableReport < Report
  def safe?
    super || fixable?
  end

  def fixable?
    @levels.size.times.any? do |i|
      new_levels = @levels.slice(0, i) + @levels.slice((i + 1)..-1)
      Report.new(new_levels).safe?
    end
  end
end
```

This isn't anything particularly complicated, but it demonstrates how putting a minimal structure in place early can make changes easier in the future.
