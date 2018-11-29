---
layout: post
title: "SRM 742 Div II - C++ vs Haskell"
---

SRM 742 was held on November 28th. Since I was
participating and I am also currently learning Haskell, I decided to solve all the problems in Haskell
as an exercise and then I found it interesting to compare it with C++ solutions.

## [BirthdayCandy - Div 2 Easy](https://community.topcoder.com/stat?c=problem_statement&pm=15213)

**Short problem statement**: Elisa can pick one of the N bags, each of them holding S amount of
candies. Once she picks a bag, she distributes it between her and her K schoolmates in a circular
manner (one to each). Once she cannot make a full circle anymore, she takes the rest for herself.
Pick a bag so Elisa gets maximum amount of candy.

**Solution**: This is a simple task and can be solved mostly analytically,
we just have to check for each bag how many candies Elisa gets and take the optimal one.
To calculate result (how many candies Elisa gets),
for a single bag, we just need `div` and `mod` operations to emulate the described distribution
algorithm.

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

Wow! Haskell solution is really amazing here, and this problem is perfect for it. What is
beautiful in Haskell solution is that we don't have to do any initialization and we can even omit
one argument of the function.

## [SixteenQueens - Div 2 Medium](https://community.topcoder.com/stat?c=problem_statement&pm=15227)

**Short problem statement**: We are given a chessboard 50 x 50 and a K queens are already positioned
on it in such a way that no queen is attacking any other queens. Given number S, return any S
positions on the board where additional S queens can be placed so the mentioned no-attack rule still
holds for all the queens on the board.

**Solution**: What works here is the simplest greedy algorithm - for each queen that is to be 
placed on the board findy any valid position - place the queen there and repeat the algorithm 
until all the queens are placed on the board.

Why does this greedy algorithm work in this case? I am not sure. Chessboard size is 50 x 50 cells
and there can be at most 16 queens on the board, so I guess that makes the problem "sparse" enough
for this to work. But I haven't yet figured out a formal explanation.

### C++

{% highlight c++ linenos %}
class SixteenQueens
{
public:
        // Helper method - check if the given position is "safe" for a queen, taking into
        // account current state on the board.
        bool isFree(int row, int col, vector<int> qRow, vector<int> qCol) {
            // Go through all currently placed queens.
            for (int i = 0; i < qRow.size(); i++) {
                // Is in the same row/col?
                if (qRow[i] == row || qCol[i] == col) {
                    return false;
                }

                // Is on diagonal?
                if (abs(row - qRow[i]) == abs(qCol[i])) {
                    return false;
                }
            }
            // If survived all queens - true.
            return true;
        }

	vector <int> addQueens(vector <int> row, vector <int> col, int add)
	{
            vector<int> sol;

            // Find the best pos for each queen, one by one.
            for (int i = 0; i < add; i++) {

                // All possible valid positions for a given queen.
                vector<int> posRows;
                vector<int> posCols;

                for (int rowIdx = 0; rowIdx < 50; rowIdx++) {
                    for (int colIdx = 0; colIdx < 50; colIdx++) {
                        if (isFree(rowIdx, colIdx, row, col)) {
                            posRows.push_back(rowIdx);
                            posCols.push_back(colIdx);
                        }
                    }
                }
                // Greedy - take the first position we found.
                int bestRow = posRows[0];
                int bestCol = posCols[0];

                // Update state on the board.
                row.push_back(bestRow);
                col.push_back(bestCol);

                // Update the solution.
                sol.push_back(bestRow);
                sol.push_back(bestCol);
            }
            return sol;
	}
};
{% endhighlight %}

### Haskell

{% highlight haskell linenos %}
-- Represents position on the chessboard.
type Pos = (Int, Int)

addQueens :: [Pos] -> Int -> [Pos]
addQueens qPoss add = take add $ foldr addQueen qPoss [1..add]
    where addQueen _ qPoss = (addQueenToBoard qPoss):qPoss

-- Given a board with queens, returns position of the next queen.
addQueenToBoard :: [Pos] -> Pos
addQueenToBoard qPoss = head $ [(row,col) | row <- [0..49], col <- [0..49], isSafe (row,col) qPoss]

-- Given a position and the board with queens, returns whether the position is "safe".
isSafe :: Pos -> [Pos] -> Bool
isSafe (row,col) = and . map isSafeFromQueen
    where 
        isSafeFromQueen (qRow, qCol) = 
            -- Check if in the same row/col.
            qRow /= row && qCol /= col &&
            -- Check if on the same diagonal.
            abs(qRow - row) /= abs(qCol - col)

{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 56 | TBD
| Haskell | 20 | TBD

This problem can also be solved very nicely in Haskell! Of course, here we assumme that there will
always exist a solution, otherwise we would need to add extra logic to handle those invalid cases.

What I like is how easily we can express the problem in Haskell and break it down into the smaller
pieces.
