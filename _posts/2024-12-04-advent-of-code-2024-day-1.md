---
layout: post
title:  "Advent of Code: Getting started"
date:   2024-12-04
categories: ruby advent-of-code
excerpt_separator: <!--more-->
---

It's time for [Advent of Code](https://adventofcode.com/2024) again! I'm starting a couple of days late because my laptop got stolen, but the first few days should be easy enough to catch up on.

My first solves are going to be in Ruby, but I hope I'll have time to come back and try again in some other languages too afterwards.

I'll upload my progress to [my AoC Github repository](https://github.com/gareth/advent-of-code-2024), but let's talk about any interesting highlights!

<!--more-->

## Solving an Advent of Code problem

Every day in December Advent of Code publishes a new puzzle. Each puzzle comes in two parts: the first part introduces a concept with a little story and a problem to solve, and then the second part adds some kind of complication that requires the problem to be solved a different way - with some alterate or additional processing.

That's interesting, because the puzzles feel like they're deliberately designed to say to you "You thought you understand the requirements? We'll now we're changing them!" - the kind of situation that comes up *all the time* in software engineering.

I want to keep my solutions for the first and second parts of each puzzle separate, so although I'm going to copy my "part 1" code into a different "part 2" file as a starting point, I'm sure it will be useful to extract code and share it between those two files at some point.

## Day 1

(I'm not repeating the entire puzzle here, [catch up for yourself](https://adventofcode.com/2024/day/1) on the AoC site)

So it looks like the 2024 event is following the same formula as normal:

We're given the first part of the problem and some "sample" input, and we're told the correct solution for the sample input to verify that we understand what's going on. Then we're given a much larger input and told to give the corresponding correct answer. Simple!

### Structuring the repository

I'm setting up some basic structure to make this easy to run. Every puzzle is generally self-contained, so it gets its own directory, and then each part of the puzzle gets its own runnable Ruby file

```
.
└── day-01
    ├── a.rb
    ├── b.rb
    ├── input
    └── input.sample
```

With that in place, I can run either part of the puzzle with either form of input - that lets me check the sample solution with the same code I'm calculating the final solution:

```
$ ruby day-01/a.rb day-01/input
[…]
```

### Handling file input

I used to think it was complicated to handle input files passed to a Ruby script like this. I always got tied up in manipulating the [`ARGV` array](https://ruby-doc.org/3.3.6/Object.html#ARGV) using an option parser library like Ruby's standard [OptionParser library](https://ruby-doc.org/3.3.6/optparse/tutorial_rdoc.html), but actually it's much simpler to handle than that.

The Ruby [ARGF class](https://ruby-doc.org/3.3.6/ARGF.html) is designed exactly for this purpose - it behaves like an input stream containing the contents of every file passed to the script. Instead of manually calling something like `File.open(argument).read` for every file you're passed, a simple `ARGF.read`

Similarly, the top-level `Kernel#gets` method will automatically return lines from all of the passed files in order.

And as a bonus, both of these methods automatically fall back to reading `STDIN` if a) there are no files passed to the script, or b) the filename `-` is passed. These are basic CLI conventions and it's apparently really easy to make a ruby script conform to them!

If your program needs to support flags and arguments *as well as* input files then you'll need to be a little more clever about it, but for AoC our handling is trivial.

### (Finally) solving the puzzle

#### Part 1

Let's see how that file handling works by processing the puzzle's input, which is two lists presented side by side:

```
3   4
4   3
2   5
1   3
3   9
3   3
```

As mentioned, we can read each input line whether that's from a file or `STDIN` by repeatedly calling `gets`:

```ruby
left = []
right = []

while (line = gets)
  values = line.split(/\s+/)
  left << values[0].to_i
  right << values[1].to_i
end
```

Since each Advent of Code puzzle uses fixed input, we can skip some of the validation that we'd want to put in for anything handling dynamic user input - we know every line will have two integer values. Maybe later puzzles will make that trickier?

Once we have the lists extracted, we can sort each of them and compare the differences

`Array#zip` is the perfect way to pair up corresponding values from two arrays, and the block form of `Array#sum` makes it easy to add up the corresponding differences:

```ruby
puts left.sort.zip(right.sort).sum { |l, r| (l - r).abs }
```

My original solution used the much less clear `Array#inject` method to build up the total using an aggregator/memo field, but I think everyone will agree that the above solution is so much clearer.

```ruby
puts left.sort.zip(right.sort).inject(0) { |memo, (l, r)| memo + (l - r).abs } # 🤮
```

#### Part 2

For part 2, we're not calculating the difference of each pair of numbers any more, we have to multiply each entry in the left hand list by the number of times it appears on the right hand side.

We can count the number of times each number appears in the right hand list by taking advantage of `Object#itself` which just returns the object you call it on. That might sound pretty useless, but when you use it in conjunction with `Enumerable#group_by` then you end up with the object itself as the **key**, and an array of all of the matches as the **values**:

```ruby
> 1.itself
=> 1

> [1, 2, 3, 4, 1, 3, 1].group_by(&:itself)
=> { 1 => [1, 1, 1],
     2 => [2],
     3 => [3, 3],
     4 => [4] }
```

The number we actually need to multiply is the *lengths* of these arrays, so we can chain on a `Hash#transform_values` call to replace each array with its size:

```ruby
multipliers = right.group_by(&:itself).transform_values(&:size)

puts(left.sum { |l| l * multipliers.fetch(l, 0) })
```

And that's it! Our answers match and we win our first two stars for the year!

