---
layout: post
title: "SRM 739 Div II - C++ vs Haskell"
---

SRM 739 was held on October 10th. The original editorial can be found 
[here](https://www.topcoder.com/blog/single-round-match-739-editorials/). Since I was
participating and I am also currently learning Haskell, I decided to solve all the problems in Haskell
as an exercise and then I found it interesting to compare it with C++ solutions.

## [HungryCowsEasy - Div 2 Easy](https://community.topcoder.com/stat?c=problem_statement&pm=15099&rd=17298)

**Short problem statement**: On a straight line cows and barns are located, and each of them is
assigned a position on that line (x coordinate). For each cow determine the barn that is closest
to it. Among two equally close barns choose the one with the lower position.

Due to modest input size limits, here it is more than enough to employ a straightforward 
brute-force solution. For each cow we can
examine all the barns and see which one is the closest. If two barns are equally close,
one with the lower position is preferred.

### C++

{% highlight c++ linenos %}
class HungryCowsEasy
{
public:
	vector <int> findFood(vector <int> cowPositions, vector <int> barnPositions)
	{
            vector<int> cowChoice;

            // Process each cow separately.
            for (int cowIdx = 0; cowIdx < cowPositions.size(); cowIdx++) {
                int cowPosition = cowPositions[cowIdx];

                // Find the nearest barn.
                int nearestBarnIdx = 0;
                for (int barnIdx = 0; barnIdx < barnPositions.size(); barnIdx++) {
                    int currBestDist = abs(cowPosition - barnPositions[nearestBarnIdx]);
                    int newDist = abs(cowPosition - barnPositions[barnIdx]);

                    if (newDist < currBestDist) {
                        nearestBarnIdx = barnIdx;
                    } else if (newDist == currBestDist) {
                        if (barnPositions[barnIdx] < barnPositions[nearestBarnIdx]) {
                            nearestBarnIdx = barnIdx;
                        }
                    }
                }
                cowChoice.push_back(nearestBarnIdx);
            }
	    return cowChoice;
	}
};
{% endhighlight %}

### Haskell

{% highlight haskell linenos %}
findFood :: [Int] -> [Int] -> [Int]
findFood cowPoss barnPoss = map findBarnForCow cowPoss
    where
        findBarnForCow cowPos = fst $ foldr1 (determineBetterBarn cowPos) $ zip [0..] barnPoss

        determineBetterBarn cowPos barn1 barn2
            | dist1 < dist2 = barn1 
            | dist2 < dist1 = barn2
            | otherwise     = if snd barn1 < snd barn2 then barn1 else barn2
            where 
                dist1 = abs(cowPos - snd barn1)
                dist2 = abs(cowPos - snd barn2)
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 30 | TBD
| Haskell | 12 | TBD

Both solutions are short, but I really like how Haskell solution is more readable while being 
shorter when compared to C++ solution. In Haskell we could very easily create pairs `(idx, value)`
and did not have to explicitly handle them as we did in C++.

## [ForumPostMedium - Div 2 Medium](https://community.topcoder.com/stat?c=problem_statement&pm=15095&rd=17298)

**Short problem statement**: We are given two timestamps in `hh:mm:ss` format, time when the forum
post was published and current time. We have to determine how long ago was a post published and
express it through the appropriate string (e.g. "a few seconds ago", "X minutes ago", ...).

This problem can actually be solved almost entirely analytically - we just transform both
timestamps to seconds, get the difference and then transform it from seconds back to the 
required format (hour,min and sec diff). There is on case to be aware of - if current time
is before the publishing time, it means the post was published the day before.

### C++
{% highlight c++ linenos %}
class ForumPostMedium
{
public:
        int getTimeInSeconds(string time) {

            string delimiter = ":";
            string token;
            int pos = 0;
            vector<int> tokens;
            while ((pos = time.find(delimiter)) != string::npos) {
                token = time.substr(0, pos);
                tokens.push_back(stoi(token));

                time.erase(0, pos + delimiter.length());
            }
            tokens.push_back(stoi(time));

            return tokens[0] * 3600 + tokens[1] * 60 + tokens[2];
        }

	string getShownPostTime(string currentTime, string exactPostTime)
	{
            int currentTimeInSec = getTimeInSeconds(currentTime);
            int postTimeInSec = getTimeInSeconds(exactPostTime);

            int totalSecDiff = currentTimeInSec - postTimeInSec;
            if (postTimeInSec > currentTimeInSec) {
                totalSecDiff = 24 * 3600 - (postTimeInSec - currentTimeInSec);
            }

            // Get hour, min, and sec diff from sec diff.
            int hourDiff = totalSecDiff / 3600;
            int minDiff = (totalSecDiff % 3600) / 60;

            if (hourDiff == 0 && minDiff == 0) {
                return "few seconds ago";
            }
            if (hourDiff == 0) {
                return to_string(minDiff) + " minutes ago";
            }
            return to_string(hourDiff) + " hours ago";
	}
};
{% endhighlight %}

### Haskell
{% highlight haskell linenos %}
getShownPostTime :: String -> String -> String
getShownPostTime currentTime exactPostTime
    | hourDiff == 0 && minDiff == 0 = "few seconds ago"
    | hourDiff == 0                 = show minDiff ++ " minutes ago"
    | otherwise                     = show hourDiff ++ " hours ago"
    where
        -- Convert string timestamp notations to seconds.
        currentTimeInSec = hmsToSec currentTime
        exactPostTimeInSec = hmsToSec exactPostTime

        -- Calculate the difference in seconds between the currentTime and
        -- postTime.
        totalSecDiff
            -- This is the case where post has been published a day before (E.g.
            -- if currentTime is "17:23:34" and postTime is "19:02:19"
            | exactPostTimeInSec > currentTimeInSec = 24 * 3600 - absDiff 
            | otherwise                             = absDiff
            where absDiff = abs(currentTimeInSec - exactPostTimeInSec)

        -- Translate difference in seconds to hour diff and min diff.
        hourDiff = totalSecDiff `div` 3600
        minDiff = (totalSecDiff `mod` 3600) `div` 60

hmsToSec :: String -> Int
hmsToSec hmsStr = h * 3600 + m * 60 + s
    where (h:m:s:_) = map read $ splitByColon hmsStr

splitByColon :: String -> [String]
splitByColon [] = []
splitByColon (':':str) = splitByColon str
splitByColon str = firstToken : (splitByColon rest)
    where (firstToken, rest) = break (==':') str
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 43 | TBD
| Haskell | 32 | TBD

Not such a huge difference here! What is interesting is that both solutions are actually very
similar, since this problem can be solved in a completely analytical way, we don't need even one
for loop in the C++ solution (except in the helper method to get time in seconds from the original
format).  
It was actually quite straightforward to write a Haskell solution from the C++ solution.

## [CheckPolygon - Div 2 Hard](https://community.topcoder.com/stat?c=problem_statement&pm=15098&rd=17298)

**Short problem statement**: We are given points respectively describing polygon in a 2D space.
The task is to determine whether a polygon is a *regular* polygon, which means it satisfies certain
conditions, such as that two adjacent segments are not collinear and none two segments are
intersecting.  
If the polygon is *regular* we should also calculate and return its area.

Input size limits are also not a problem here, so we just need to check if polygon satisfies
all the requirements. This task was mostly about geometry - determining whether two segments
interesect with each other, checking if point lines on a segment, calculating area of the polygon
etc. It was also important to keep all the calculations in integers so we don't get into precision
problems.

Here are some links I found useful in understanding the formulas employed in the solution:

* Orientation of 3 ordered points - [link](https://www.geeksforgeeks.org/orientation-3-ordered-points/)
* Area of a polygon (shoelace formula) - [link](https://www.geeksforgeeks.org/area-of-a-polygon-with-given-n-ordered-vertices/)
* Check if two segments intersect - [link](https://www.geeksforgeeks.org/check-if-two-given-line-segments-intersect/)
* Check if three points are collinear - [link](https://www.geeksforgeeks.org/program-check-three-points-collinear/)

### C++
{% highlight c++ linenos %}
typedef pair<int, int> Point;
#define x first
#define y second

typedef pair<Point, Point> Segment;

class CheckPolygon
{
public:
        int orientation(Point a, Point b, Point c) {
            long long o = (long long)(b.y - a.y) * (c.x - b.x) - (long long)(c.y - b.y) * (b.x - a.x);

            if (o > 0) return 1;
            if (o < 0) return 2;
            return 0;
        }

        bool areCollinear(Point a, Point b, Point c) {
            // If area of a triangle is 0 -> points are collinear
            return orientation(a, b, c) == 0;
        }

        bool isPointOnSegment(Point point, Segment segment) {
            Point sA = segment.first;
            Point sB = segment.second;

            // Must be collinear and within the bounding rectangle of
            // the segment.
            return orientation(point, sA, sB) == 0
                && (point.x <= max(sA.x, sB.x) && point.x >= min(sA.x, sB.x))
                && (point.y <= max(sA.y, sB.y) && point.y >= min(sA.y, sB.y));
        }

        bool doSegmentsIntersect(Segment a, Segment b) {
            // NOTE(matija): what if segments are colinear?

            return orientation(a.first, a.second, b.first) != orientation(a.first, a.second, b.second) &&
                orientation(b.first, b.second, a.first) != orientation(b.first, b.second, a.second);
        }

        long long getPolygonArea(vector<Segment> segments) {
            long long area = 0;
            for (int i = 0; i < segments.size(); i++) {
                Point sA = segments[i].first;
                Point sB = segments[i].second;

                area += (long long)sA.x * sB.y - (long long)sB.x * sA.y;
            }
            return area;
        }

	string check(vector <int> X, vector <int> Y)
	{
            // Create segments of a polygon.
            vector<pair<Point, Point> > segments;
            for (int pointIdx = 0; pointIdx < X.size(); pointIdx++) {
                Point a = {X[pointIdx], Y[pointIdx]};
                Point b = {X[(pointIdx + 1) % X.size()], Y[(pointIdx + 1) % X.size()]};
                segments.push_back({a, b});
            }

            // Check every segment.
            for (int segmentIdx = 0; segmentIdx < segments.size(); segmentIdx++) {
                // Has positive length?
                // Is non-colinear with the next segment?
                // Does not intersect with other non-neighbouring segments?
                
                Segment s = segments[segmentIdx];
                int nextSegmentIdx = (segmentIdx + 1) % segments.size();
                int prevSegmentIdx = (segmentIdx - 1) < 0 ? segments.size() - 1 : segmentIdx - 1; 

                if (areCollinear(s.first, s.second, segments[nextSegmentIdx].second)) {
                    return "Not simple";
                }

                // Check with all non-neighbour segments.
                for (int i = 0; i < segments.size(); i++) {
                    if (i == segmentIdx || i == nextSegmentIdx || i == prevSegmentIdx) 
                        continue;

                    if (doSegmentsIntersect(s, segments[i])) {
                        return "Not simple";
                    }
                }
            }

            long long area = abs(getPolygonArea(segments));
            stringstream s;
            s << area/2 << '.' << "05"[area % 2];

            return s.str();
	}
};
{% endhighlight %}

### Haskell
{% highlight haskell linenos %}
type Point = (Int, Int)
type Segment = (Point, Point)

orientation :: Point -> Point -> Point -> Int
orientation (ax, ay) (bx, by) (cx, cy)
    | o > 0     = 1
    | o < 0     = 2
    | otherwise = 0
    where o = (by - ay) * (cx - bx) - (cy - by) * (bx - ax)

areCollinear :: Point -> Point -> Point -> Bool
areCollinear a b c = (==0) $ orientation a b c

isPointOnSegment :: Point -> Segment -> Bool
isPointOnSegment point@(px, py) (sPoint1, sPoint2) = 
    -- Point must be collinear with the segment.
    orientation point sPoint1 sPoint2 == 0 &&
    -- Point must be within the bounding box of the segment.
    px <= max sax sbx && px >= min sax sbx &&
    py <= max say sby && py >= min say sby
    where
        (sax, say) = sPoint1
        (sbx, sby) = sPoint2

doSegmentsIntersect :: Segment -> Segment -> Bool
doSegmentsIntersect (a1, a2) (b1, b2) = 
    orientation a1 a2 b1 /= orientation a1 a2 b2 &&
    orientation b1 b2 a1 /= orientation b1 b2 a2

getPolygonArea :: [Segment] -> Int
getPolygonArea = foldr addSegmentToArea 0
    where addSegmentToArea ((p1x, p1y), (p2x, p2y)) area = area + p1x * p2y - p2x * p1y

checkPolygon :: [Int] -> [Int] -> String
checkPolygon xs ys = 
    if all isGood segmentsWithNextAndNonAdj then areaStr else "Not simple"
    where
        isGood (s, next, nnss) = not (areCollinear sPoint1 sPoint2 nextPoint2) 
                                 && all (not . doSegmentsIntersect s) nnss
            where
                (sPoint1, sPoint2) = s
                (_, nextPoint2) = next

        segmentsWithNextAndNonAdj = getNextAndNonAdjacent segments

        segments = getSegmentsFromPoints $ zip xs ys

        areaStr = (show $ areaNum `div` 2) ++ decimalPart
            where 
                areaNum = abs $ getPolygonArea segments
                decimalPart = if areaNum `mod` 2 == 0 then ".0" else ".5"

getSegmentsFromPoints :: [Point] -> [Segment]
getSegmentsFromPoints ps = zipWith (\p1 p2 -> (p1, p2)) ps psCircL1
    where psCircL1 = take (length ps) $ drop 1 $ cycle ps 
        
getNextAndNonAdjacent :: [Segment] -> [(Segment, Segment, [Segment])]
getNextAndNonAdjacent xs = zipWith getNextAndNonAdjacentForOne xsIndexed (repeat xsIndexed)
    where
        xsIndexed = zip [1..] xs

        getNextAndNonAdjacentForOne (idx, x) xsIdx = (x, next, nonAdjElems) 
            where 
                next = snd $ head $ drop idx $ concat $ repeat xsIdx
                nonAdjElems = map snd $ filter (not . areAdjacent (length xsIdx) idx . fst) xsIdx

        areAdjacent listLength idx1 idx2 = idx1 == idx2 || diff == 1 || diff == listLength - 1
            where diff = abs (idx1 - idx2)
{% endhighlight %}

### Comparison

|---
| Language | Lines | 100 runs
|-
| C++ | 93 | TBD
| Haskell | 68 | TBD

Solutions are here again pretty similar! Problem can be nicely decomposed in a functional
way, we need a function for checking each condition, and then we just need to perform checks
for every segment of the polygon.
