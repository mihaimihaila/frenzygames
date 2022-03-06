# Mahjong Zen BugFix March 5th Investigation

Author: *Mihai*, last modified: _06/03/2022_

---

On March 5h, 2022 I finally fixed a bug in Mahjong Zen that was causing the game to crash after watching an ad.
This is the retrospective of how the bug was introduced, investigated and fixed after affecting the game for more than 3 months.

## How it all started

Mahjong Zen contains interstitial video ads from Vungle. The ads provide revenue to suport free play. Over the past few months I heard several complains from users describing ads that "won't load" or "loading and freezing the game". The cause of those reports is unclear to me but I suspect it has to do with the browser version the users have installed locally, the status of Windows Updates or their internet connectivity. These reports were sporadic, but enough to make me look for a solution that would attempt to fix the problem.

## The incorrect solution

In order to deal with the ads that won't load problem, I decided to use a timer that would attempt to display the ad, wait 60 seconds, then start a game of mahjong.
I put together a simple implementation of a 60 seconds timer.
The code looks simple enough and I decided to start testing it manually to put it to the test.

### Testing: Previously working behavior

When the ad loads and completes playing the ad can be closed and the game begins. This existing behavior was still working correctly after the change.

### Testing: The timeout fix

To test the newly implemented fix I had to replicate the environment first described by the users. I prevent the local ads from being displayed and sure enough after 60 seconds the timeout mechanism would kick in and start the game.

## The mistake

At this point I considered the fix good and the game ready for launch. The new version was soon uploaded and in the hands of customers.
The problem is that I haven't explored all possible scenarios related to ad watching and tested creatively the implications of the timeout.
It took over 3 months to finally piece together another scenario that would cause the game to crash after watching an ad.

## Not everyone watches the ads

It turns out that some players don't just stare at the screen while an unskippable ad is playing. They sometimes do something else while the ad is playing like check their phone or refill their coffee. This scenario is the one I haven't tested and that causes the crash.

1. The user loads a video ad. The ad loads succesfully and starts playing.
1. The ad finished playind and can be dismissed, but it is not dismissed (for whatever reason)
1. The timeout expires and the game starts "underneath" the already finished video ad
1. The players returns to the game and notices the finished video ad and closes it
1. At this point the program attemps to start a game as a reward for watching the ad. As the game is already started, the game crashes.

## The solution

The easiest way to fix this issue is to remove the timeout "fix" that was applied in the first place. It was a hasty fix that was hiding a problem rather than fixing it and it triggered a new, more serious bug.

## Lessons learned

* After implementing a fix, attempt to break the game by exploring possible circumstances that relates to the fix.
* The bug was discovered by closely analyzing code changes that were done around the time the issue was first reported by users. I could have discovered the issue earlier if I would have analyzed the changes earlier.
* As this change affected a significant function of the game, a feature flag turning the "fix" on and off would have allowed to control and deploy a fix as soon as it was discovered instead of having to rollout a fix.

## Timeline

1st of December, 2021, the bug is release in 0.0.80.0 version of Mahjong Zen
Several attempts are done in the following released to understand the problem by adding logs and more analytics events.
5th of March, 2022 the issue is identified
5th of March, 2022 the issue is fixed
5th of March, 2022 an update containing the fixed is relased in the Microsoft Store
