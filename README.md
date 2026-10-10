# Permutations LeetCode: The Backtracking Template That Turns LeetCode 46 Into a Five-Minute Pattern Problem

Type "permutations leetcode" into a search bar and there's usually one of two things going on. Either you've hit LeetCode 46, written a nested-loop mess, and watched it fall apart on the third element — or you've solved it once, forgotten the template, and need the shape back before an interview tomorrow.

Both problems have the same root cause. Permutations isn't a problem you memorise. It's a template you learn once and then recognise for the rest of your career.

## What LeetCode 46 actually asks

The statement is short: given an array `nums` of distinct integers, return all possible permutations, in any order.

Constraints are the tell here:

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- all integers are unique

Six elements means 6! = 720 permutations. That's the real signal. When the input cap is that small and the output is "every arrangement," the interviewer isn't testing your optimisation instincts. They're testing whether you can build a decision tree cleanly.

The problem's acceptance rate sits around 82% — one of the higher numbers for a backtracking Medium — which tells you most people who attempt it eventually get the recursion right. The ones who don't usually get stuck on the same two things: forgetting to copy the path before appending, and not knowing when to pick an element versus when to advance an index.

## Why this is a backtracking problem, not a nested-loop problem

Take `nums = [1, 2, 3]`. You need `[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]`.

The mental model that makes this click: you're filling positions one at a time. For position 0, you can put any of the three numbers. For position 1, you can put any number you haven't used. For position 2, you only have one left.

That's a tree. Each level is a position, each branch is a choice, and every leaf is a complete permutation. Since permutations are the leaves, you have n! leaves, and each leaf sits n levels deep — which is where the O(n · n!) complexity comes from.

The generic backtracking template looks like this:

python
def dfs(start_index, path):
    if is_leaf(start_index):
        report(path)
        return
    for edge in get_edges(start_index):
        path.add(edge)
        dfs(start_index + 1, path)
        path.pop()


Three lines matter and they're always the same three: **choose, recurse, unchoose**. Everything else in permutation problems is a matter of what you carry in `get_edges` and what counts as a leaf.

This is the frame where algo.monster earns its keep if you want a second pass at it — their whole curriculum is organised around explicit named templates like this one rather than problem-by-problem answers, and the backtracking module walks the "choose / recurse / unchoose" shape across permutations, subsets, and combination-sum variants back to back. 👉 [See how AlgoMonster teaches the backtracking template](https://algo.monster?fpr=li69)

## Solution 1: the used-array approach

This is the version most people write first, and the one interviewers can read fastest.

python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        used = [False] * len(nums)

        def backtrack(path):
            if len(path) == len(nums):
                res.append(path[:])       # copy, not reference
                return
            for i in range(len(nums)):
                if used[i]:
                    continue
                used[i] = True
                path.append(nums[i])
                backtrack(path)
                path.pop()                # undo
                used[i] = False

        backtrack([])
        return res


What's happening in plain terms: at every recursive call you scan the whole array, skip anything already taken, then take one element and recurse. The `used` array is the pruning step — it's what stops `[1,1,3]` from ever being built.

Two details are load-bearing:

1. `path[:]` at the leaf. Append `path` directly and every entry in your result becomes an alias to the same list, which mutates back to `[]` as the recursion unwinds. This is the single most common bug in the problem, and it fails in a way that looks like a logic error rather than a reference error.
2. `used[i] = False` after the recursive call. Undo both mutations, not just one. The `path.pop()` without the `used` reset gives you fewer permutations than expected, not more.

## Solution 2: in-place swapping

The swap version skips the `used` array entirely and does its choosing inside the input array.

python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        n = len(nums)

        def backtrack(start):
            if start == n:
                res.append(nums[:])
                return
            for i in range(start, n):
                nums[start], nums[i] = nums[i], nums[start]
                backtrack(start + 1)
                nums[start], nums[i] = nums[i], nums[start]   # swap back

        backtrack(0)
        return res


At index `start`, each remaining candidate is swapped into that slot, the recursion handles position `start + 1`, and then the swap is undone so the next candidate gets a fair turn.

Which one should you write in an interview? Swap if you're comfortable with mutation and want to save the `used` array. Used-array if you want code that reads like the pattern and is trivially extendable to `Permutations II`, where duplicates make the pruning logic the entire point of the problem. Most people find the used-array version easier to modify under pressure, and that's the one that shows up most often in editorial solutions.

## Complexity, and why it's not O(n!)

Both approaches run in **O(n · n!)** time. There are n! permutations and each one costs O(n) to build or copy into the result list. Auxiliary space is **O(n)** — the recursion stack plus either the `used` array or a few swap temporaries — not counting the output itself, since the result list is what it is.

Some write-ups quote O(n · n!) for space because they're counting the output. That's technically conservative and awkward to defend out loud. The cleaner statement is O(n) auxiliary, O(n · n!) total including the returned list.

The practical takeaway: nothing you do makes this asymptotically better. There's no clever trick, because you're required to output all n! permutations. If an interviewer pushes on optimisation, the honest answer is that output size is the lower bound.

## Where people actually get stuck

Four failure modes cover nearly every wrong submission:

- **Aliasing the path.** Covered above. Symptom: result is a list of empty lists.
- **Forgetting the unchoose step.** Pops the path but leaves the flag set (or vice versa). Symptom: fewer permutations than n!.
- **Advancing the index instead of tracking used elements.** Fine for subsets, wrong here. Subsets move forward with `i + 1` because order doesn't matter; permutations need to reach back and pick any unused index.
- **Reaching for `itertools.permutations`.** It works and it's one line, but interviewers tend to treat that as declining to answer. When it's genuinely allowed, the reviewer usually says so explicitly.

## What comes after this problem

Permutations is a gateway problem, and the four that follow it in the same family are where the pattern actually gets tested:

- **Permutations II (47)** — same output, but the input can contain duplicates. The fix is to sort first and skip `nums[i] == nums[i-1]` when the previous duplicate hasn't been used in the current path. Same template, one extra condition.
- **Next Permutation (31)** — modifies the array in place to produce the next lexicographic arrangement. Different technique entirely, but it's the reason sorting matters in the duplicate case.
- **Permutation Sequence (60)** — asks for only the k-th permutation, so you skip the full enumeration and compute it with factorials.
- **Subsets / Combination Sum** — the sibling patterns. Same tree, different `get_edges` and different leaf conditions.

If you can write Permutations, Permutations II, and Subsets from the same template without looking, you've learned a pattern. If you can only write Permutations, you've learned a problem — and LeetCode has thousands of those.

That distinction is the whole sales pitch behind pattern-based platforms, and it's worth being specific rather than just nodding along. AlgoMonster's curriculum is text and diagram based with no video, so if you learn by watching someone code, this isn't the format for you. What it does have is a decision-flowchart approach to "which pattern does this problem want," which is the skill that decides whether you spend five minutes or twenty-five on an unfamiliar question. 👉 [Browse the AlgoMonster pattern library](https://algo.monster?fpr=li69)

## What AlgoMonster currently costs

AlgoMonster spans 500+ lessons across 6 courses, and the free tier is enough to read through a few pattern write-ups and decide whether the text-first format suits how you study. The paid plans are annual or lifetime — there's no monthly option, which is worth knowing before you look.

Pricing on the platform moves with promotions, so treat the numbers below as what the checkout page currently shows rather than a fixed list price:

| Plan | What you get | Price | Billing | Access |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | — | [Start free](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; company questions updated monthly; bonus Foundation and Build-an-LLM courses | $5.84/mo | Billed annually ($69.99/year), cancel or upgrade anytime | [Get Pro Yearly](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan, plus lifetime updates | $89 | One-time payment | [Get Pro Lifetime](https://algo.monster?fpr=li69) |

> AlgoMonster runs frequent sales, and the annual figure in particular has been reported anywhere from around $89 to well over $200 at different points. Check the live price at checkout rather than trusting a screenshot or a review post.

Two more things worth flagging honestly. First, there's no monthly billing and no formal free trial beyond the free tier, so the free content is your preview. Second, the platform is text and diagrams — no video walkthroughs — and the system design coverage is introductory rather than something you'd rely on for a senior-level loop. If you need either of those, budget for a second resource.

## Which plan makes sense

The annual plan is the straightforward pick if you have an interview loop in the next six to eight weeks. At the current annual price you're paying roughly what a couple of months of LeetCode Premium costs, for a curriculum with an endpoint.

Lifetime is the better call if you expect to do this again. Engineers interview every two to three years, and at $89 one-time the maths isn't close — you'd hit the break-even point on your first renewal cycle, and the plan includes all future content at no extra cost. That includes courses AlgoMonster adds later, which is a real consideration given they've already shipped a Build-an-LLM course on top of the algorithm material.

If you're still deciding, read a few free lessons first. The format question is the one that matters: AlgoMonster's lessons read like a well-organised textbook with interactive diagrams, which is either exactly what you want or exactly what you don't.

## Quick answers

**Is `itertools.permutations` acceptable?** Only if the interviewer explicitly permits library calls. It's a one-liner and it demonstrates nothing about your ability to build a recursion tree. Keep it as a brute-force baseline in a comparison, not as your submission.

**Do I need to sort the input?** Not in LeetCode 46, where all elements are distinct. Sorting is required in Permutations II so duplicate values sit adjacent and the skip condition can be expressed as `nums[i] == nums[i-1]`.

**Why does my output contain duplicates?** You're almost certainly not distinguishing identical values. The standard fix is the ordering rule: among equal elements, always take the leftmost unused one first.

**Is recursion depth a problem?** With `n <= 6` on the platform, no. In an interview variant where n is larger, depth is O(n) and still not the bottleneck — the n! output is.

**Can I solve it iteratively?** Yes. Start with a list containing one empty permutation, then for each number in the input, insert it into every position of every permutation built so far. It's clear to read and it avoids recursion, though the memory profile is worse since you hold partial permutations at every stage.

The part that matters is what you carry away from LeetCode 46. Spend the time to internalise the choose–recurse–unchoose skeleton now, and Permutations II, Subsets, Combination Sum, and Letter Combinations of a Phone Number all become variations on a single page you already know. Skip that step, and you'll search "permutations leetcode" again in three months — which, conveniently, is roughly how long it takes for the template to fade if you never actually learned it. 👉 [Learn the pattern once instead](https://algo.monster?fpr=li69)
