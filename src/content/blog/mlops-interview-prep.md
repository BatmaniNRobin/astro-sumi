---
title: "MLOps Interview Prep Guide"
description: "Interview Prep for a Mid-level DevOps / Machine Learning Operations Engineer (MLOps)"
pubDate: 2026-05-12
updatedDate: 2026-05-12 # optional
tags: ["mlops", "tech", "leetcode", "computer science", "devops", "career"]
draft: false # drafts are visible in dev, dropped from builds
---

# Introduction

LeetCode. Data Structures and Algorithms. My nemesis, we meet again. Yup, you still need it for DevOps. You still need it. This post won’t just cover LeetCode and great resources to use to practice it, it’s meant to outline what additional and useful information one would find useful as well when getting ready to interview around.

## Data Structures and Algorithms

### LeetCode

After nearly 3 years it returns. Let’s get right into it. The best advice is to just start doing them. PRACTICE PRACTICE PRACTICE. Below I’ll be going a bit more into a technique I apply to break down questions into what the likely solution will be but it becomes significantly easier to identify once you can finally stop thinking about the algorithms and can do those without thinking, and actually address the important parts of the question at hand. I found this from the pirate king.

Great resources for practicing that I have found are below:

1. The Pirate Kingdom, a fellow engineer in the industry, has this great cheat sheet and simple breakdown of the most commonly used leetcode algorithms:
https://www.piratekingdom.com/leetcode/cheat-sheet

2. The goto for learning how to even think about algorithms again, the goat, NeetCode. Neetcode has both free and paid options but offers courses, youtube video breakdowns of HUNDREDS of questions and both their greedy and optimal solutions alongside their Big (O) breakdown for space and time. All in all just one of the absolute best resources. 

https://neetcode.io/roadmap

### Cheat Sheet

```python
if input == array:
	- hashmap
	- sort
	- two pointers
if input == sorted_array:
	- two pointers
	- binary search
if input == binary_tree:
	- DFS (preorder, inorder, or postorder)
	- BFS
if input == matrix or graph:
	- DFS (recursion, stack)
	- BFS (queue)
if input == linked_list:
	- dummy node
	- two pointers
	- fast, slow pointers

(harder, more rare to find these until you get to mediums)
if input == max/min subarray:
	- sliding window
	- dynamic programming
if recursion == banned:
  - stack (deque)
if kth largest or smallest:
  - heap
  - quickselect
```

## The Job Prep

### DevOps

- k8s
- docker
- cicd
- iac
- security and optimizations for the above
- link to github

### MLOps

- rag
- bedrock
- litellm
- openwebui
- agentic workloads
- optimization of models, infra, resources for rag and context to work better