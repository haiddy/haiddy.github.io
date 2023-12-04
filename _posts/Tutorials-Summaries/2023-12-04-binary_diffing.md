---
title: "APKs Analysis"
classes: wide
ribbon: Red
header:
  teaser: /assets/images/binary_diff/wallpaper.jpg
description: " APKs analysis tips and tools "
categories:
  - Tutorials Summaries
toc: true

---

## Overview
I was working on a ransome sample when I had a problem with that sample and decided to change it to another version. then it came to my mind to use bindiff to know the differences and make this overview article about what I got :)

## BinDiff

bindiff is a tool and an IDA plugin that can get us the matched and the unmatched functions between 2 samples or files.
we can export comments or the renamaing of the functions if we already passed those steps before.

using it as a tool or as a plugin has the same functionality but here I'm using the plugin.

there are many other choices but I'm just using Bindiff

## How to use

first, you need to get the IDA database for one file of them and save it.

then open the second file on IDA and press ctrl+6

 ![](/assets/images/binary_diff/bindiff.PNG)

 here choose Diff Database then the idb/i64 file we saved before.

 4 sub-windows will pop up after that.
 
 2 for unmatched, 1 for the matched, and the last for some statistics

 ![](/assets/images/binary_diff/windows.PNG)

 if we scroll down to the statistics window we can see the similarity is 52% "Looks like there are a lot of differences :'("

 ![](/assets/images/binary_diff/similarity.PNG)

 primary unmatched is the functions in the current file that there is no match for it

 while secondary unmatch is the opposite

 currently, I only care about the matched since those are what I already analyzed before so I just get to the matched functions sub-window.

 in this sub-window there are even statistics about how similar they are, what changed, the effective address for both, and the names in both files.

 so we want to check a function how to do so?

 just right click on the function then View flow graphs

 ![](/assets/images/binary_diff/graph.png)

this will open a workspace in bindiff 

![](/assets/images/binary_diff/comparsion.PNG)

for example I chose this random function to see.

on the very left and right sides those are the addresses of the functions in IDA.

now let's pick up 2 functions that are less similar

![](/assets/images/binary_diff/less_similar.PNG)

here we can see there was a block added, let's zoom in to see the details

![](/assets/images/binary_diff/corresponding.PNG)

the line and the corresponding one are both highlighted if changed but of course you need to read the changes your self and better through IDA

since I am not analyzing the ransome I will stop here and not going to go deeper in the details.

