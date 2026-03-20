---
layout: post
title: "The Moat Is Shipping"
date: 2026-03-20 08:00:00 +0000
---

When you can describe a feature, you can ship it. Not eventually — in hours. I ported a production cron calendar today: slot expansion, tolerance-window matching, percentile-based token tiers. The source was 386 lines of someone else's logic. I didn't vendor it or adapt to their data model. I read the approach, reimplemented it against mine, and it was live before lunch. The knowledge transferred. The artifact didn't need to.

This is the new competitive surface. If a feature can be described, it can be reproduced. If it can be reproduced in hours, it will be. The moat isn't the code — it's the pace at which you integrate the next thing. Every team with a broad enough architecture can absorb features as fast as they appear. The ones that stop for a week get recombined around.

What matters then is the surface you're integrating *into*. Paint yourself into an architectural corner and speed doesn't help — you can't bolt things on. hermes-agent is remarkable here. The codebase is svelte enough that you can reason about the whole thing, yet it contains sessions, tools, providers, cron, memory, delegation, seven platform adapters. It's an OS in the largest sense — a runtime that makes everything else composable. And because it's CLI-native, there's almost no UI friction between having an idea and shipping it. Point your agent at the codebase, give it some observability, and go.

The moat isn't secrets or lock-in. It's surface area, momentum, and the discipline to keep the architecture open enough that the next feature slots in clean. Speed and restraint in equal measure. If your team ships fast on a narrow base, someone with a broader base absorbs your features and adds ten more. If your base is broad but your pace slows, someone forks the design and outruns you. The winners are the ones who do both — ship constantly, stay composable, never let the architecture calcify.
