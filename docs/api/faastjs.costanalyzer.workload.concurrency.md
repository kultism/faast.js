---
id: faastjs.costanalyzer.workload.concurrency
title: CostAnalyzer.Workload.concurrency property
hide_title: true
---
[faastjs](./faastjs.md) &gt; [CostAnalyzer](./faastjs.costanalyzer.md) &gt; [Workload](./faastjs.costanalyzer.workload.md) &gt; [concurrency](./faastjs.costanalyzer.workload.concurrency.md)

## CostAnalyzer.Workload.concurrency property

The amount of concurrency to allow. Concurrency can arise from multiple repetitions of the same configuration, or concurrenct executions of different configurations. This concurrency limit throttles the total number of concurrent workload executions across both of these sources of concurrency. Default: 64.

<b>Signature:</b>

```typescript
concurrency?: number;
```