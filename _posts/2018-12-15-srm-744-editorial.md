---
layout: post
title: "SRM 744 Div II - C++ vs Haskell"
---

SRM 744 was held on December 14th, 2018. The original editorial can be found
[here](https://www.topcoder.com/blog/single-round-match-744-editorials/). I am currently learning
Haskell so I decided to solve all tasks in it as well and then compare to my original C++ solutions.

## [ThreePartSplit - Div 2 Easy](https://community.topcoder.com/stat?c=problem_statement&pm=15235)

**Short problem statement**: We are given a half open interval `[a, d)` which contains `n = d - a`
elements. We need to divide it into three parts such each of them has at least `n div 3` elements.
We need to return the "middle" interval as `[b, c)` (so half open again).

**Solution**: A simple and quick task that can be solved analytically, we just need to be
careful not to mess up +-1 indices. There also multiple possible solutions, but maybe the most
straightforward one (at least for me) was to assign `n div 3` elements to the first two parts and
then whatever is left to the third part.

E.g. if we were given `[0, 8)` as a starting interval, that means we have
numbers `0, 1, 2, 3, 4, 5, 6, 7`. If we divided them as described above, we'd end up with intervals
`[0, 1], [2, 3], [4, 5, 6, 7]` where we "crammed" all the extra stuff into the last interval.

There are also other valid solutions, e.g. we could've put 3 elements in each interval. So if
you are using some editor plugin which compares against the example test cases (like Vimcoder) don't
worry if it reports some of your solutions as wrong.


### C++

{% highlight c++ linenos %}
class ThreePartSplit
{
public:
    vector <int> split(int a, int d) {
        int minIntervalSize = (d - a) / 3;
        return {a + minIntervalSize, a + 2 * minIntervalSize};
    }
};
{% endhighlight %}

It was really cool for me to learn here how we can construct and return vector in a single line,
very clean and readable.

### Haskell

{% highlight haskell linenos %}
threePartSplit :: Int -> Int -> (Int, Int)
threePartSplit a d = (a + minIntervalSize, a + 2 * minIntervalSize)
    where minIntervalSize = (d - a) `div` 3
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 8 | TBD
| Haskell | 3 | TBD

Although Haskell solutions has a bit less lines than C++ one (mostly due to C++ `class` plumbing),
I would say expresiveness-wise solutions are the same. Since it is O(1) analytical solution, we
don't need any looping or mapping or data structures, and with this concise way of 
creating a vector C++ solution is also very nice to read.

## [MagicNumbersAgain - Div 2 Medium](https://community.topcoder.com/stat?c=problem_statement&pm=15235)

**Short problem statement**: Number is _magic_ if it is a perfect square and if its digits alternate
in a fashion of greater, smaller, greater, smaller, ... (e.g. `1 3 2 17 4 2 3`). Given `[A, B]`
interval, find out how many _magic_ numbers it contains.

**Solution**: The naive solution would be to check for each number between `A` and `B` whether it
is a _magic_ number or not. However, due to the constraints (interval can have up to 
10<sup>10</sup> elements), we can see it would not be fast enough in all the possible cases.

But if we could be a bit smarter and somehow first find all the perfect squares in `[A, B]` and
then only examine them instead of every number in the interval, would that be more performant?
Sure it would, and since we will need to examine at most 10<sup>5</sup> squares 
(since 10<sup>5</sup> hits the upper range bound when squared), we can be sure it is going to
be fast enough.

Finding squares is also not a problem, we can just keep iterating over the roots and generate them
until we go over the upper range bound.

### C++

{% highlight c++ linenos %}
class MagicNumbersAgain
{
public:
    bool isMagic(long long num) {
        // Obtain digits of a number.
        vector<int> digits;
        while (num > 0) {
            digits.push_back(num % 10);
            num /= 10;
        }
        reverse(digits.begin(), digits.end());

        // Perform the check on the digits.
        for (int i = 1; i < digits.size(); i++) {
            if (i % 2 == 1) {
                if (digits[i] <= digits[i - 1]) return false;
            } else {
                if (digits[i] >= digits[i - 1]) return false;
            }
        }
        return true;
    }

    int count(long long A, long long B) {
        // Find all squares in [1, B].
        vector<long long> allSquares;
        for (long long root = 1; root * root <= B; root++) {
            allSquares.push_back(root * root);
        }

        // Check for every square if in [A, B] and magic.
        int magicCount = 0;
        for (int i = 0; i < allSquares.size(); i++) {
            long long square = allSquares[i];
            if (square >= A && square <= B && isMagic(square)) magicCount++;
        }

        return magicCount;
    }
};

{% endhighlight %}

A cool trick I learnt here is to generate all the squares from 1 up to B (or even up to the upper
limit) and then later just check if it is in the required range. This frees us of doing the work
of figuring out what is the first square in the range. Theoretically speaking it is not optimal 
since we are doing extra work, but here we can afford it and it keeps the code cleaner and less
error-prone.

Unfortunately I haven't used this trick during the competition and made a mistake in just a
situation described above (finding first square >= `A`) and my solution failed. So it is a good
lesson for the future.

### Haskell

{% highlight haskell linenos %}
threePartSplit :: Int -> Int -> (Int, Int)
threePartSplit a d = (a + minIntervalSize, a + 2 * minIntervalSize)
    where minIntervalSize = (d - a) `div` 3
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 8 | TBD
| Haskell | 3 | TBD

Although Haskell solutions has a bit less lines than C++ one (mostly due to C++ `class` plumbing),
I would say expresiveness-wise solutions are the same. Since it is O(1) analytical solution, we
don't need any looping or mapping or data structures, and with this concise way of 
creating a vector C++ solution is also very nice to read.

