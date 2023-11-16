---
title: Why don't they just make CPUs bigger and add more transistors so we can have 10 GHz?
date: 2022-09-02
tags: ["electronics", "vlsi", "chips", "transistor", "migrated", "quora"]
---

Because they don't work on magic. Some points are as below:

-   It is very hard to build larger chips while maintaining good yeild, i.e., having less defects. When you scale your design up, with more transistors, interconnects, etc., the probability of defects increases. And since the entire chip has to work, your yeild is less as your chip is larger and you can't discard a part of it, so you discard it all.
-   Higher clock frequency will cause more power dissipation, so your chip heats up more, and your electricity bill shoots up.
-   With so many transistors working, power dissipation will increase massively! 
-   Power dissipation is the main reason — you will fry your chip! Dissipating heat from dense pockets in the chips is very challenging.
-   With so many transistors, your capacitance will increase as every transistor has an associated gate capacitance, causing an increase in rise and fall times if the circuits aren't modular enough.
-   If you want faster circuit, you should increase transistor size for a given load capacitance. But increasing size also increases capacitance, so you would need to reduce the transistor count.
-   But the capacitance increases due to interconnects now far overshadow that of transistor, and dominates. And you cannot not have interconnects!
-   It would be very hard to distribute the 10 GHz clock in the circuit, and routing in general, especially when you are adding more transistors.
-   Digital logic operating at 10 GHz would need to switch the states at that speed, which is hard to make.
