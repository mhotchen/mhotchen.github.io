---
layout: post
title: "Effective code reviews"
---

As a team lead one of the areas I think my team stood out in was code reviews. They're thorough, effective, and we've developed a system with great exposure and awareness of what's going on for the whole team without distractions.

## Workflow

We use git-flow, but the primary benefit comes from feature branches. This allows commits to be built however the developer decides (whether squashing or not) and then start a code review when they want to merge their branch back in to the main branch (develop, master, default, whatever). If you're not using git-flow or not using branches then the code review could be done on a squashed commit. The code to review should have few changes, preferably less than 500 changed lines.

## Tooling

Tooling helps code reviews a lot. We have static analyzers picking up on coding standards, potential security issues, and technical debt through things like cyclomatic complexity, duplicate lines, etc. Most of this is handled with SonarQube, with an additional commercial security static code analyzer for good measure. We also have builds running on push to verify the code works and all tests pass. By using these tools the reviewers are free to look for more in depth things like potential bugs and architectural choices.

For the actual code review we use Bitbucket. This is integrated with our builds and SonarQube so we can see the results before reviewing the code.

## People

We break our teams up in to sprint groups, with each sprint group being up to 7 developers and QA. My team is 5 developers and QA so it's small enough that we include every person in the team on the code review. That includes QA. This gives the team a lot of exposure to what's going on, and it stops people from being bias with who they pick for their code reviews.

What we noticed after introducing that was that only a couple of people were consistent in reviewing, and some reviews might be open for several days before someone even looks at it, even if it was a single line change. To mitigate this we decided to dedicate 2 of the developers to being "merge masters": someone who is responsible for reviewing the code and getting it merged. This responsibility rotates each week, and the merge master puts any open code reviews as a higher priority than their own work.

## Reviews

If anyone happens to be free and wants to review code then they're free to do so, but it's the responsibility of the merge masters to get code reviewed quickly and effectively. Because branches are small and these people know they can take their time since it's their role for that week, we aim for ruthlessly detailed code reviews. This does occasionally bring up stylistic choices, opinions, and flame wars, but after some time (usually a couple of months) people come to terms with that and no feelings are hurt any more. The team actually end up trusting each other a lot more, and become a much more cohesive unit because things are reasoned about and understanding is shared. If things get messy enough we'll usually try to settle it with further research or face-to-face discussions (usually as an entire team again).

You need to create an atmosphere of letting people take their time reviewing, and also of understanding that reviewers are trying to help. Once those hurdles are passed then the quality of code begins to rise dramatically from every person on the team. The same patterns of comments stop coming up as everyone learns from each other, so the comments become about new things, new ideas, and so the code continues to improve, as do the individuals.

## Approval and QA

We say that two developers need to approve before a branch can be tested by QA. Once these two approvals are made then QA will test the branch passing or failing as necessary. Any changes to the code automatically remove all approvals in Bitbucket (with the right setting) so the branch goes back to the review phase (normally with little/no comments this time around) and then back to QA once reviewing is complete. Once QA approves the code then the developer is free to merge it in to the main branch (upon which it's automatically deployed to our staging environment for UAT).

## Conclusion

My team have given very positive feedback about this workflow and outside of the classic things around architecture, testing, etc. this has been one of the greatest contributions to a high quality codebase (and high quality team) that I've seen. When we initially did fairly sloppy and ad-hoc reviewing the benefits were almost nil but there was an increase in friction to releasing your code. Now there's still the friction, but it brings a lot of benefits.
