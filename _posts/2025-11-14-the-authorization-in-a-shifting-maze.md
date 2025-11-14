---
layout: post
title: "Authorization in a Shifting Maze"
date: 2025-11-14 02:00:00 -0000
categories: ai, authorization, mcp, a2a, braindump
tags: en-US
permalink: /posts/authorization-in-a-shifting-maze
---

Everyone’s hitting the same wall with OPA. It starts great, externalizing policies, central control, policy-as-code. But once you go past a few dozen rules, things start to feel heavy. Latency creeps in. Your rules grow from clean to unreadable. You add a “deny all” just to keep things safe and now you’re chasing ghosts in staging. And the worst part is, once you finally get a policy to say “yes” or “no”, you still don’t know who else that rule applies to. You know it worked, but not why, or who’s living in the same permission shadow.

So people start looking elsewhere. ReBAC pops up in the conversation. SpiceDB, ORY, the whole Zanzibar-inspired crowd. The promise is seductive, represent relationships as first-class citizens, turn permissions into something you can query and explain. Who can read this doc? Just ask. Who has access to a project? Query the graph. It feels closer to how we think about access, and a lot closer to how auditors ask questions.

But it’s not easy. These systems aren’t a library you import. They’re infrastructure. They need care. You model your world in a schema, sync changes, run migrations. And when your policies go beyond simple relationships like time windows, content rules, anomaly triggers, you start stitching back in custom logic, just like before. Still, it’s better than OPA in some cases. You trade raw flexibility for visibility, and a lot of teams are happy to make that trade.

And then comes AI. Not the fancy demos, the messy, real stuff. Assistants calling APIs on behalf of users. Agents chaining requests. Data flying across embeddings, vectors, context windows. Your policies weren’t written for this. They were written for API calls, for users clicking buttons, not for autonomous workflows making decisions three layers deep.

The enforcement point is gone. There’s no gateway in the loop. The agent is inside. It can read a tool manifest, call whatever it wants. If you don’t bind its permissions explicitly, it’ll just act. Worse, sometimes it doesn’t even know what it’s doing. It’s reacting, generating, inferring. The idea of deterministic authz starts to bend.

So now we’re staring at a future with protocols like MCP and A2A trying to formalize how these agents act. They’re early, experimental, mostly scaffolding. But they’re pointing at a real need, to give agents context-aware, time-bound, minimal access. To make authorization dynamic, explainable, and enforced at the function level. To do for agents what we tried to do for users a decade ago.

We’re in that awkward phase again. The old tools are creaking. The new ones aren’t mature. Everyone’s rewriting the same policies, debugging the same permission holes, patching over the same visibility gaps. It feels like the ground is shifting, but no one’s quite sure where it’s going to settle.

It’s a good time to be paying attention. Better time to be building. More soon.