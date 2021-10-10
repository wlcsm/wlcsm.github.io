---
weight: 5
title: "Advent of Code 2020 Day 1-2 in Rust"
date: 2021-10-01
author: "Will Cashman"
authorLink: "https://wlcsm.github.io"
description: "Discussion of the solutions to the Day 1 and 2 Advent of Code's solutions in Rust"
resources:

catagories: ["rust"]

lightgallery: true
---

## Using Advent of Code to learn Rust

Rust is a relatively new programming language that boast big benefits compared to its contemporaries. It combines the speed of C, expressiveness of Haskell, all while being memory and thread safe.

I have used Rust a lot this year to implement a polynomial library for my Honours year project, and so I have only really used it for scientific computing. I have barely touched string manipulation, parsing, and so I thought it was about time that I should branch out. The [Advent of Code]() is a set of programming challenges that are rel eased every day of December, leading up to Christmas functioning like an [advent calendar](). The problems are fairly basic and can normally be accomplished in under 50 lines (depending on your chosen programming language). Pr grammars often use this opportunity to learn new languages.

Since I am a math major, I will be taking a very algorithmic approach to the problems and will often discuss their complexity classes. Though I hope this won't obscure the explanation for those who are not interested in it.

## Day 1

The problem can be simplified to this:

	Given a list of numbers, find the two entries that sum to 2020 and multiply them together.

So first lets think of a couple of ways this can be done. The naive way would be to iterate through the list and add it to to every other element to see if it sums to 2020. This is nice and simple, and since we our given list is fairly short (only 200 lines) it will execute fairly quickly.

```rust
fn part1_naive(expense_list: Vec<usize>) -> usize {

    let n = expense_list.len();

    for i in 0..n {
        for j in i..n {
            if expense_list[i] + expense_list[j] == 2020 {
                return expense_list[i] * expense_list[j]
            }
        }
    }
    panic!("No two entries sum to 2020");
}
```

However, this is not the most efficient approach. If we let $n$ denote the number of elements in the list, then this approach uses around $n^2$ operations in the worst case, since the outer loop must execute at most $n$ time, and the inner loop $n-i$ times on the $i^\text{th}$ iteration of the outer loop. Giving a total of approximately
$$ n + (n - 1) + (n - 2) + \cdots + 1 = (n^2 - n)/2 $$
(ignoring constants) operations. More formally, this is $\mathcal O(n^2)$, which is not the most desirable.

Lets look at another technique. Notice that in the outer loop, the thing we really care about is finding a certain element. The problem is that we are performing a linear search in the inner loop to find it which makes it slow. In order to speed up the lookup time, we could use a Hash map to obtain (approximately) constant lookup time.

Therefore the outer loop will iterate $n$ times, and inside the loop will be constant, which means the overall algorithm will be $O(n)$ overall.

The last option seems to have worse complexity, however it actually performs the best for reasonable inputs since although the hash map has constant lookup time, that constant may be very large. To test, I made two tests, one using the default hash function in the standard library, and another using [rustc\_hash](https://crates.io/crates/rustc-hash); a fast hash function that is highly suitable for integer keys.

Here we sort the list and initialise two variable `low` and `high` to be either ends of the array. If the sum of the elements at `low` and `high` is less than 2020, then we increase `low` (since the list is sorted, this will increase our sum) if it is less, then we decrease high.

```rust
use std::cmp::Ordering;

fn part1_fast(mut expense_list: Vec<usize>) -> usize {

    expense_list.sort();

    let mut low = 0;
    let mut high = expense_list.len() - 1;

    while low <= high {
        match (expense_list[low] + expense_list[high]).cmp(&2020) {
            Ordering::Equal   => return expense_list[low] * expense_list[high],
            Ordering::Less    => low += 1,
            Ordering::Greater => high -= 1,
        }
    }
    panic!("No two entries sum to 2020");
}
```

Here I have used the `Ordering` construct in the standard library to simplify the thing. Otherwise I could have written the (not as nice)
```rust
if expense_list[low] + expense_list[high] == 2020 {
    return expense_list[low] * expense_list[high],
} else if expense_list[low] + expense_list[high] < 2020 {
    low += 1
} else {
    h -= 1
}
```

Now lets look at the benchmarks to see how they perform on the puzzle input (200 lines)

```
test test::bench_fast       ... bench:       3,275 ns/iter (+/- 74)
test test::bench_hashmap    ... bench:      10,530 ns/iter (+/- 183)
test test::bench_rustc_hash ... bench:       2,875 ns/iter (+/- 156)
test test::bench_naive      ... bench:       5,760 ns/iter (+/- 587)
```

It seems that choosing the right hash function can make a great impact on the performance. Though the naive algorithm didn't perform too poorly for this test, since it is $\mathcal O(n^2)$ we would expect it to perform much poorer on large test cases.


## Part 2

Here in part 2, we iterate through the list and then call our Part 1 solution on the remainder. If we use the sorting algorithm from Part 1 we don't need to sort the list every time, so this would naturally be the best choice of the three. 

# Day 2

Day 2's problem is somewhat simpler algorithmically. Here we are given a list of passwords as well as a small schema which specifies a rule the password must satisfy in order for it to be valid. So our problem is to iterate through the lines of the line and count the number of valid passwords.

Now this is the kind of work I am unfamiliar with. Though I did know enough to know that *regular expressions* would be very useful for extracting the data. 

An example of a line in the input is

```
15-16 p: ppppppppppplppppp
```

Which says that there needs to be 15 to 16 of the characters 'p' in the password `ppppppppppplppppp` in order for it to be valid. So we much extract `15`, `16`, `p`, `ppppppppppplppppp` from the text.

After fiddling around a bit on [Online Regex](https://regex101.com/) I arrived at the regular expression

```
(\d+)-(\d+) (\w): (\w+)
```

Let break this down a bit: `\d` matches a single digit and `\w` matches a non-digit character. Appending `+` to the end of a pattern indicates that one of more of the pattern must be present, `\d+` matches one or more digits e.g. $4$, $16$, and `\w+` matches one or more word characters e.g. `foo`, `bar`.  Parentheses `( )`specify a *capture group* which allows us to refer to each of the matches in the capture group so that we may extract the data out of the pattern after it has been matched

Lets see how to do this in Rust:

Add the `regex` crate to write by adding the line 

```toml
[dependencies]
regex = "1.0"
```

To your project's `Cargo.toml` file.

To create a regex we write

```rust
let pass_regex = regex::Regex::new(r"(\d+)-(\d+) (\w): (\w+)").unwrap();
```

Then if we can iterate over the capture groups using 

```rust
let cap = pass_regex.captures_iter(&line).next().unwrap();
let pass = PassData {
    begin: cap[1].parse::<usize>().unwrap(),
    end:   cap[2].parse::<usize>().unwrap(),
    chr:   cap[3].parse::<char>().unwrap(),
    pass:  &cap[4]
};
```

Lets break this down:

* `pass_regex.captures_iter(&line)` applies our regex to `line` and return an iterator over all matches (not *captures*!)
* `.next().unwrap()` Gets the first match of our regex. In our case we know that there is a guaranteed match and that there is only doing to be one.
* `cap` is an array which contains the captures inside the match as well as the entire match itself as the first element. For instance, if we had applied the regex to `15-16 p: ppppppppppplppppp` then `cap` would be
```
cap = ["15-16 p: ppppppppppplppppp:, "15", "16", "p", "ppppppppppplppppp"]
```

Thus all we need to do is parse our data and store it in the `PassData` struct.

Overall this gives

```rust
struct PassData<'a> {
    begin: usize,
    end:   usize,
    chr:   char,
    pass:  &'a str,
}

use std::fs::File;
use std::io::{BufRead, BufReader};

// Counts the number of passwords in "filepath" that are validated by the "policy" function
fn count_valid_pass(filepath: &str, policy: fn(PassData) -> bool) -> usize {

    let pass_regex = regex::Regex::new(r"(\d+)-(\d+) (\w): (\w+)").unwrap();
    let file = File::open(filepath).unwrap();

    BufReader::new(file)
        .lines()
        .filter(|line| {
            let line = line.as_ref().unwrap();
            let cap = pass_regex.captures_iter(&line).next().unwrap();
            let pass = PassData {
                begin: cap[1].parse::<usize>().unwrap(),
                end:   cap[2].parse::<usize>().unwrap(),
                chr:   cap[3].parse::<char>().unwrap(),
                pass:  &cap[4]
            };
            policy(pass)
        })
        .count()
}

fn valid_password_p1(data: PassData) -> bool {
    let occurences = data.pass.chars().filter(|c| *c == data.chr).count();
    data.begin <= occurences && occurences <= data.end
}
```
Note here that we do not need to load the whole file into memory, but rather we can process it one a time which we do here.

This isn't the best solution however as what we care about is actually *parsing* the data rather than *validating* it, which is what regular expressions are more suited towards. For this we can actually use a parser library achieve this. Chris Biscardi has a [great article](https://www.christopherbiscardi.com/advent-of-code-2020-in-rust-day-2-regex-vs-parser-combinators-with-nom) of how one can use the `nom` crate to create a *parser* rather than a regular expression for better performance for this exact problem, and [Log Rocket](https://blog.logrocket.com/parsing-in-rust-with-nom/) gives a great introduction to using the `nom` parser library

I haven't encountered parsers yet and so I am thinking of learning about it and writing my experience as it looks like a much more elegant way to solve these kinds of problems.

## Part 2

Part 2 asks us to change the validation function. Fortunately, since we wrote our code in a very modular way, this is very simple fix to make. In this one, passwords are valid if either $15$ (exclusively) or $16$ is the character 'p'. This gives us the fairly simple code below, which we may pass into the `count_valid_pass` function as before

```rust
fn valid_password_p2(data: PassData) -> bool {
    let mut chars = data.pass.chars();
    let first = chars.nth(data.begin - 1).unwrap();
    let second = chars.nth(data.end - data.begin - 1).unwrap();

    let xor = |a: bool, b: bool| (a && !b) || (!a && b);

    xor(first == data.chr, second == data.chr)
}
```
