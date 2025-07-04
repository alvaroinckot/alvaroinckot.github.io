---
layout: post
title:  "Skip the Puzzle, Review the Pull Request"
date:   2025-05-29 01:00:00 -0300
categories: [braindump]
tags: [en-US]
permalink: /posts/skip-the-puzzle-review-the-pull-request
---

Generative copilots can already spit out textbook LeetCode answers on demand, which means the signal in those questions is evaporating. What remains scarce is the ability to read unfamiliar code, spot hidden risks, and articulate respectful feedback that guides it toward production quality. In other words, review skill is becoming the clearest proxy for day-zero impact on a codebase, and companies that keep filtering only for algorithm speed will keep mistaking autocomplete for engineering judgment.

This is not about handing candidates a random diff and asking for a style nitpick. A thoughtful review exercise can act as a safe wrapper around a genuine business puzzle: think of a simplified service the team owns, a flaw that recently made it to staging, and a follow-up request from product that forces a design rethink. Give the candidate the context, the failing test, and the freedom to propose a patch or a rewrite. You will see whether they weigh tradeoffs, align with conventions, and offer incremental steps the team could actually merge, all of which map directly to what the role needs on week one. Yes, it costs staff time to craft and maintain these scenarios, but the payoff is hiring engineers who walk into onboarding already fluent in how the company writes software.

A recent side project drove this home for me. I was sketching a model to price collectible RPG characters (post coming soon.) and let o3 suggest a feature set and algorithm lineup. The reply read like a drinking fountain menu: token count, embedding vectors, gradient boosting, everything evenly chilled. None of it captured the lore quirks that drive bids, quirks I recognized only after years of raiding dungeons and reading forum drama. When I embedded those domain signals the model finally sang. A review-first interview can surface that sort of tacit knowledge because the candidate explains the why behind each line, not only the what.

Until a true general intelligence can blend raw computation with the lived nuance of every market on earth, we should make the most of this gap. Shift interview energy toward code and pull request review, and invest the effort to craft realistic scenarios that mirror your production beating heart. **The next generation of agentic AI coders will pair brilliantly with humans who excel at critique and context, and those are exactly the engineers you will attract.** While we do not have AGI, this meantime would be good to do it.