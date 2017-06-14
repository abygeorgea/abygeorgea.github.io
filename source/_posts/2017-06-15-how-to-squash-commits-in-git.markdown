---
layout: post
title: "How to Squash Commits in Git"
date: 2017-06-15 05:46:39 +1000
comments: true
categories: Git
keywords: squash commits, Git , Rebase
description: How to squash commits in Git using rebase
---


Very often we will be committing smaller pieces of work in our local machine as we go. However before we push them to a centralized repository, we may have to combine these small commits to single large commit, which makes sense for rest of the team. I will explain how this can be achieved by using interactive rebasing.

To start with, let us assume the initial commits history look like below. It have 4 minor commits done to the same file. 
![Initial Commit Structure]({{site.images_dir}}/2017/06/15/HowToSquashCommits_image1.png)

Now we need to squash last for commits into a single commit. The command required for that is as below. This tells git to rebase head with previous 4 commits in an interactive mode.

		$ git rebase -i HEAD~4


This will pop up another editor with details of last 4 commits and some description about possible actions on this. Initially, all of them will have a default value of PICK. Since we are trying to squash commits together, we can select one of the commits as PICK and rest all needs to be changed as SQUASH. Save and close the editor once all changes are made.

![Interactive Rebasing]({{site.images_dir}}/2017/06/15/HowToSquashCommits_image2.png)

After this, another popup will appear with comments given for each of the commits. We can comment out unnecessary comments by using `#` and also modify required comments as we need. In below screen, I have modified comments for the first commit and commented out rest all. Save and close the editor once all changes are made.

![Selecting comments]({{site.images_dir}}/2017/06/15/HowToSquashCommits_image3.png)

Now Git will continue rebasing and it will squash all commits as selected in the previous step.

![Git rebase]({{site.images_dir}}/2017/06/15/HowToSquashCommits_image4.png)

If we look at commit history, we can see that commits are now squashed to single commit.

![Squashed Result]({{site.images_dir}}/2017/06/15/HowToSquashCommits_image5.png)