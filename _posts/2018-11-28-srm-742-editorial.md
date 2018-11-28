---
layout: post
title: "SRM 742 Div II - C++ vs Haskell"
---

SRM 739 was held on November 28th. The original editorial can be found 
[here](https://www.topcoder.com/blog/single-round-match-739-editorials/). Since I was
participating and I am also currently learning Haskell, I decided to solve all the problems in Haskell
as an exercise and then I found it interesting to compare it with C++ solutions.

## [BirthdayCandy - Div 2 Easy](https://community.topcoder.com/stat?c=problem_statement&pm=15213)

**Short problem statement**: Elisa can pick one of the N bags, each of them holding S amount of
candies. Once she picks a bag, she distributes it between her and her K schoolmates in a circular
manner (one to each). Once she cannot make a full circle anymore, she takes the rest for herself.
Pick a bag so Elisa gets maximum amount of candy.

This is a simple task and can be solved mostly analytically, we just have to check for each bag how
many candies Elisa gets and take the optimal one. To calculate result (how many candies Elisa gets),
for a single bag, we just need `div` and `mod` operations.

### C++

{% highlight c++ linenos %}
class BirthdayCandy
{
public:
	int mostCandy(int K, vector <int> candy)
	{
            int maxCandy = 0;
            for (int i = 0; i < candy.size(); i++) {
                maxCandy = max(maxCandy, candy[i] / (K + 1) + candy[i] % (K + 1));
            }
            return maxCandy;
	}
};

{% endhighlight %}

### Haskell

{% highlight haskell linenos %}
mostCandy :: Int -> [Int] -> Int
mostCandy k = maximum . map (\c -> c `div` (k+1) + c `mod` (k+1))
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 8 | TBD
| Haskell | 2 | TBD

Wow! Haskell solution is really amazing here, and this problem is perfect for it. 
