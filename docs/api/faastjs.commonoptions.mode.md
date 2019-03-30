---
id: faastjs.commonoptions.mode
title: CommonOptions.mode property
hide_title: true
---
[faastjs](./faastjs.md) &gt; [CommonOptions](./faastjs.commonoptions.md) &gt; [mode](./faastjs.commonoptions.mode.md)

## CommonOptions.mode property

Specify invocation mode. Default: `"auto"`<!-- -->.

<b>Signature:</b>

```typescript
mode?: "https" | "queue" | "auto";
```

## Remarks

Modes specify how invocations are triggered. In https mode, the functions are invoked through an https request or the provider's API. In queue mode, a provider-specific queue is used to invoke functions. Queue mode adds additional latency and (usually negligible) cost, but may scale better for some providers. In auto mode the best default is chosen for each provider depending on its particular performance characteristics.

The defaults are:

- aws: `"auto"` is `"https"`<!-- -->. In https mode, the AWS SDK api is used to invoke functions. In queue mode, an AWS SNS topic is created and triggers invocations. The AWS API Gateway service is never used by faast, as it incurs a higher cost and is not needed to trigger invocations.

- google: `"auto"` is `"https"`<!-- -->. In https mode, a PUT request is made to invoke the cloud function. In queue mode, a PubSub topic is created to invoke functions.

- local: The local provider ignores the mode setting and always uses an internal asynchronous queue to schedule calls.

Size limits are affected by the choice of mode. On AWS the limit is 256kb for arguments and return values in `"queue"` mode, and 6MB for `"https"` mode. For Google the limit is 10MB regardless of mode. In Local mode messages are sent via node's IPC and are subject to OS IPC limits.

Note that no matter which mode is selected, faast.js always creates a queue for sending back intermediate results for bookeeping and performance monitoring.