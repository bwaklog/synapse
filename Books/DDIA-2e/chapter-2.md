**Materializing and Updating Timelines**

The bluesky blog on lossy timeline generation[^1] and explaining with improvement in tail latency, reducing p99 workload for fanout delivery of posts by 90% and p99 for full post fanout duration reduced

**Average, Medians and Percentiles**
Someone brought up trimmed means (instead of arithmetic mean itself) and how they use tm90, etc at work for understanding the average user experience while removing the outliers that sit in the tail latency measurements, that could affect the calculations.

Found the Frequency Tails[^2] article by Brendan Gregg really nice.

[^1]: When Imperfect Systems are Good, Actually: Bluesky's Lossy Timelines · Jaz's Blog [jazco.dev](https://jazco.dev/2025/02/19/imperfection/)
[^2]: Frequency Trails [www.brendangregg.com](https://www.brendangregg.com/frequencytrails.html)
