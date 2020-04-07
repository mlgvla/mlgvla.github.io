---
layout: post
title:      "Dog with a bone ("nbsp" version)"
date:       2020-04-07 14:17:48 -0400
permalink:  dog_with_a_bone_nbsp_version
---


Like a dog with a bone, I just can't let go of a problem, even the smallest one.  I'm working on my CLI project, and the scraping was going beautifully until I came across an annoying problem.  The "p" elements I'm stripping have nested "strong" tags for a label, followed  by the text.  On the website, it appears "**Instrument**: Viola" and it strips to "Instrument: Viola,"   I really just want "Viola" as the value to go into my hash and make my own hash keys.  No problem - `.sub(/^[^:]+:\s*/, "")`, strips all that gunk away, and I'm left with the value to plop into my hash.  Except  ONE of my "p" element text outputs would ALWAYS print with a space before it, completely immune to my Regex:  

" Upon Request."

No chomping, stripping, no ANYTHING would get rid of that space before " Upon."  " Viola" - no problem.  "Viola." " Cello" - no problem.  "Cello." So, I re-inspected the HTML and discovered that for some reason " Upon Request" was marked up as `&nbsp;Upon Request`.  Why?  I have no clue. It didn't fit the pattern, at all.  It doesn't matter.  It IS my job to get rid of it, however.  After trying so many ways to eradicate it and starting to get hungry, I had to look up to those who have tread this path before...

Ok, Google is my friend.  Was it too specific an issue for some wizard on Stack Overflow to have tackled? NO!  After combing through quite a few solutions to the problem, I found one that was simple and completely understandable to me at this stage of the game.  I also learned a little more about Nokogiri in the process.

`nbsp = Nokogiri::HTML("&nbsp;").text`

This takes the HTML-edness away from the non-breaking space and converts to a space character that Ruby can happily deal with. So then,

`string.sub(nbsp, "")`

will completely obliterate any leading HTML non-breaking space characters.

I can't even begin to tell you how great it felt to see that little menacing gap disappear, so now all my values have no leading spaces, and I can format things the way I would like in my user interface.

Ok, NOW I can eat a peanut butter sandwich...






