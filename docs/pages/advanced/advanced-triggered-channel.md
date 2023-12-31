---
title: Triggered Channels
keywords: interlok
tags: [advanced]
sidebar: home_sidebar
permalink: advanced-trigged-channel.html
---

There are some subtle differences between a [triggered-channel][] and a normal channel; the consumers inside each workflow should really be based around things that actively poll (i.e. [AdaptrisPollingConsumer][] implementations) rather than consumers that are passively receiving messages (like a [jms-queue-consumer][] or the like, if you need JMS behaviour, use a concrete [JmsPollingConsumer][]). The polling implementation for each consumer should be a [one-time-poller][] rather than one of the other implementations.

This type of channel also handles errors and events slightly differently. By default, the channel will supply its own message error handling implementation, rather than using the Adapter's (in this case a [triggered-retry-message-error-handler][], infinite retries at 30 second intervals). The trigger itself could be anything you want, it has a consumer/producer/connection element, so you could listen for an HTTP request, or use a [triggered-jmx-consumer][] which registers itself as a standard MBean, so you can trigger it remotely via jconsole or the like.

## Example ##

If for instance, an adapter is running on a remote machine, and you don't have the capability to login to the filesystem and retry failed messages then you could use a [triggered-channel][] to copy all the files from the _bad_ directory into a _retry_ directory so that the configured [FailedMessageRetrier][] is triggered. This is quite a marginal use case; if you have the type of failure where messages can be automatically retried without manual intervention then a normal [retry-message-error-handler][] will probably be a better bet.

```xml
<channel class="triggered-channel">
  <unique-id>RETRY_FAILED_MESSAGES</unique-id>
  <trigger>
    <connection class="jms-connection">
       ... config skipped for brevity
    </connection>
    <consumer class="jms-topic-consumer">
      <topic>retry-failed-messages</topic>
    </consumer>
  </trigger>
  <workflow-list>
    <standard-workflow>
      <consumer class="fs-consumer">
        <base-directory-url>/path/to/bad/directory</base-directory-url>
        <poller class="triggered-one-time-poller"/>
        <create-dirs>true</create-dirs>
      </consumer>
      <producer class="fs-producer">
        <base-directory-url>/path/to/retry/directory</base-directory-url>
        <create-dirs>true</create-dirs>
      </producer>
    </standard-workflow>
  </workflow-list>
</channel>
```
- The trigger is any message received on the topic `retry-failed-messages`
- When a trigger message is received it will
    - start the channel
    - files will be copied from the bad directory to the retry directory
    - stop the channel


[triggered-channel]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-triggered/3.11-SNAPSHOT/com/adaptris/core/triggered/TriggeredChannel.html
[AdaptrisPollingConsumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisPollingConsumer.html
[jms-queue-consumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/jms/PtpConsumer.html
[one-time-poller]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-triggered/3.11-SNAPSHOT/com/adaptris/core/triggered/OneTimePoller.html
[triggered-retry-message-error-handler]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-triggered/3.11-SNAPSHOT//com/adaptris/core/triggered/RetryMessageErrorHandler.html
[triggered-jmx-consumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-triggered/3.11-SNAPSHOT//com/adaptris/core/triggered/JmxConsumer.html
[FailedMessageRetrier]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/FailedMessageRetrier.html
[retry-message-error-handler]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/RetryMessageErrorHandler.html
[JmsPollingConsumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/jms/JmsPollingConsumer.html