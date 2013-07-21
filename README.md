Practical rank aggregation for timed sports
===========================================

Goals
-----

Strava is a GPS-based mobile app that lets cyclists and runners
compare their times on segments. See
https://strava.zendesk.com/entries/20945952-What-are-Segments- for a
better description of segments.

Each segment comes with a ranking of all athletes that ever completed
that segment. While knowing one's ranking on a particular segment is
fun, it is always ideal:

* The ranking is based on all-time bests, instead of comparing the
  current level of the athletes.
* Top performers are ranked on more segments than average athletes
  because they train more and farther. As a result segment rankings
  overpopulated with top athletes and one's relative rank (expressed
  as a number from 0 to 1) on a segment is usually worse than one's
  global rank among all Strava athletes.
* The difficulty of segments varies with the terrain, and there is no
  indication of how long it takes for the average Strava
  user to complete the segment.

For a given athlete who just completed a new segment or is planning to, we
want to predict their time based on their recent performances on other
segments. This would allow athletes to track their weekly or monthly
progress without ever having to ride or run the same route.


Solution
--------

The general problem of merging partial rankings into a global ranking
that respects partial rankings as much as possible is a classic
optimization problem called "rank aggregation".

Unlike the problems and solutions easily found in the literature [1], we have:
* timed performances, not just ranks
* a very large dataset: millions of athletes and 10-100 times more
  performances

Our proposed solution is an iterative heuristic that consists in
computing the performance of the imaginary globally-median athlete
(GMA) on each segment.

Input data:
* recent performances of each athlete (e.g. union of the last 10
performances and the performances over the last month) on all segments

Output:
* predicted time of the GMA (globally-median athlete) for each segment
* global rank of each athlete

Variables:
* S: list of segments, each containing a list of performances
     per-segment accumulator: time of the GMA
* A: table of athletes, each containing a list of performances
     per-athlete accumulator: weighted average relative performance

Initialization:
  S[i].acc <- median(S[i].perfs)

Loop:
  for all i in athletes:
    # Update athlete scores
    A[i].acc <- weighted_average_relative_perf(A[i].perfs)
  for all j in segments:
    # Adjust performance of GMA whose score is 0 by definition
    S[j].acc <- linreg(S[j].perf.times, S[j].perf.athlete_scores())(0)

Convergence:
When each accumulator changes by less than a certain amount over one
iteration.


References
----------

1. Rank aggregation methods for the Web. Dwork, Kumar, Naor,
   Sivakumar http://www.wisdom.weizmann.ac.il/~naor/PAPERS/rank_www10.html
