Ladies and Gentlemen, here are the latest performance benchmarks on our Open Source Firebase project - a realtime, decentralized, offline-first, [graph](https://github.com/amark/gun/wiki/graphs) database!

![](https://gun.eco/see/ie6.png)

Thank you.

We're getting even better numbers on other devices:

 - Android phone, **~5M ops/sec**.
 - Macbook Air, Chrome, **~30M ops/sec**.
 - Macbook Pro, Chrome Canary, **~80M ops/sec**.
 - Lenovo netbook, IE6, **~100K ops/sec**.

Compare to Redis at 0.5M ops/sec (cached reads, Macbook Air), even with pipeline optimizations turned on, here: https://redis.io/topics/benchmarks .

## How did we accomplish 100,000 ops/sec in IE6?

This test ran on the cheapest Windows laptop available on the market. A $150 2GB Atom CPU which was loaded up with IETester to do some performance benchmarking. These numbers represent a breakthrough in performance not possible with other databases. A technical deep dive is highlighted on the [Read the Source](https://www.youtube.com/watch?v=70dn1oZQFCk) podcast:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/70dn1oZQFCk/0.jpg)](https://www.youtube.com/watch?v=70dn1oZQFCk)

For those who want the short summary:

1. Use PTSD (Performance Testing Speed Development, like TDD) to micro-optimize your code without getting lost in micro-optimizations. This [tech talk](https://youtu.be/BEqH-oZ4UXI) is a must watch - [@alexhultman](https://github.com/alexhultman), author of the high performance C++ [uWebSockets](https://github.com/uWebSockets/uWebSockets) library praised it [here](https://github.com/amark/gun/issues/261#issuecomment-262959696).
2. Reduce function calls and prioritize hot code paths as the first if statements.
3. Do work only once, then use centralized caching and cache updating to skip ever doing work again.

Resulting caveats:

 - These performance benchmarks rarely have cache misses, but replicates other database benchmarks (like Redis) which do the same. Measuring disk I/O is not very interesting.
 - Take all performance testing benchmarks with a huge grain of salt. That is why we're working on [panic](https://github.com/gundb/panic-server), a distributed testing runner.

Check out our main [GitHub repo](https://github.com/amark/gun) to learn more or hop on the [gitter](https://gitter.im/amark/gun) to chat! 