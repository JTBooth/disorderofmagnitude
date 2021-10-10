---
layout: post
title: "Longest Substring Without Repeating Characters: Explained"
categories: leetcode
permalink: /leetcode-non-repeating-substring
---

This is my ruby solution to [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters), a Leetcode medium. My solution is, I believe, optimal in complexity, but not especially fast practically. I'll explain the pieces independently, note some assumptions I'm making, derive the complexity, and discuss takeaways for future problems.

# The Problem
...is to find the longest substring of the input string that doesn't have any repeated characters. We're going to do this using a sliding window, with left and right marked by "pointers" that we slide down the string. These algorithms are nice because the complexity is very easy to derive, and is usually linear, as long as the pointers only move right and always move right.

# The Solution
```ruby
require 'set'

# @param {String} s
# @return {Integer}
def length_of_longest_substring(s)
    if s.length == 0 then return 0 end
    if s.length == 1 then return 1 end
        
    left_ptr = 0;
    right_ptr = -1;
    window = Set[]
    max_len = 1
        
    while right_ptr < s.length - 1 do
        if window.include?(s[right_ptr + 1]) then
            window.delete(s[left_ptr])
            left_ptr += 1
        else
            right_ptr += 1
            window.add(s[right_ptr])
            max_len = [max_len, window.length].max
        end
    end
    return max_len
end
```

## Base Cases
```ruby
...
    if s.length == 0 then return 0 end
    if s.length == 1 then return 1 end
...
```
This is an interviewing habit. There tend to be tricky edge cases around very short inputs. Sometimes you can come up with a parsimonious solution that handles them elegantly without a specific callout. But, this is tricky, and even if you manage it, anyone reading the program is going to have to convince themselves of the same thing. Let's just save ourselves the trouble.

## Setup
```ruby
...
    left_ptr = 0;
    right_ptr = 0;
    window = Set[s[0]]
    max_len = 1
...
```
The string we're considering starts at left_ptr and ends at right_ptr, having length 1 when `left_ptr == right_ptr`. We initialize the window accordingly, containing the first character of the string. Note that our base cases save us from worrying about whether s[0] is an out of bounds error. `max_len` is initialized to 1, the current length of `window`.

## The Loop
```ruby
...
    while right_ptr < s.length - 1 do ... end
...
```
This is the ugliest part of the code. Basically, the loop body can involve looking one space to the right of right_ptr's current position. When it does so, we're done, the window has covered every possible long string, so the loop can end. 

### The oh-heck condition
```ruby
...
    if window.include?(s[right_ptr + 1]) then
        window.delete(s[left_ptr])
        left_ptr += 1
    else
...
```
We check if we're allowed to expand the window. Here, we can't. So, we move the start of the window forward. Since the window's contents always match the span of characters between the pointers, without duplicates, we know that moving left_ptr to the right of a character removes it from the window.

### The horray condition
```ruby
...
    else
            right_ptr += 1
            window.add(s[right_ptr])
            max_len = [max_len, window.length].max
        end
    end
...
```
If we're not not allowed to expand the window, we're allowed to expand the window, so we do. It now might be the longest it's ever been, so, we check that, and if it is, we write it down.

Then, when the loop ends, we return.

# Discussion
## Assumptions
There's a neat thing called an invariant. It's basically something that, first, you promise will always be true, and make sure is always true, about your code. Then, you can use the assumption that the invariant is true throughout your code. My main invariant here is that the contents of the set `window` always exactly match the substring between left_ptr and right_ptr. This is why we can just do `window.include?(...)` to check if the window can expand.

You might also worry that left_ptr might end up to the right of right_ptr. This actually happend! If two of the same character are right next to each other ("bb"), at some point, both pointers will be on the first "b", and the window will be ("b"). Then, we can't advance right_ptr, so we advance left_ptr, and window becomes the empty set. When the window is empty, it will never include anything, so the next thing we do will always be to advance right_ptr. 

## Complexity
The cool thing about sliding window techniques is that, for a string of length n, we can only "advance a pointer" 2n times, and thankfully 2n == n. So, if we can make sure all our pointer advance steps have constant cost, and we're sure advance a pointer every loop, we know our complexity is linear. We're making good use of Set here, which uses a hashmap to give `add`, `delete` and `include?` in constant time. Hashmaps are black magic that break all the normal rules of of complexity analysis by assuming that integers are big and you don't get unlucky, which nothing else gets to do. But they're allowed in this game for some reason. So, of course, you should use them whenever you even suspect they might be useful, when playing the "complexity" game. Ruby is a sensible language that tracks `.length` for you automatically, so, everything is constant. Hooray, O(n).

## Things I Would Improve
I don't like that there are so many +1 and -1 in the code. Whenever I see one I'm worried that it's wrong. I expect that by using the usual string slicing convention, where the length of a slice is equal to the difference between the endpoints, we can eliminate these. But, I'm happy with the fact that base cases are handled separately, and I wouldn't try too hard to write a version that didn't need them unless I was entering a beauty pageant.

# Takeaways
So, what from this problem will I be bringing to future problems?
- I'll be on the lookout for sliding-window solutions, even ones where the slide step might not be constant time.
- It's nice to note what invariants you're using. I think there's some connection between an invariant and what arguments make sense for a recursive call. I find recursion easier to understand but much harder to write - it would have been a nightmare for this problem.
- When in doubt, cheat - use hashmaps.
