# Writing your own Consumer

Within the Interlok framework, an [AdaptrisMessageConsumer][] is responsible for receiving messages from the target system. You need to decide on how the consumer will be triggered. If the consumer should be timer based (e.g. poll a directory every x seconds), then you can extend [AdaptrisPollingConsumer][] which allows for a pluggable timer mechanism. Other types of consumer should directly extend [AdaptrisMessageConsumerImp][].

- You can get access to the configured [AdaptrisConnection][] instance by using the `retrieveConnection` method.
- Call `this.decode(byte[])` to decode the message with any configured [AdaptrisMessageEncoder][] implementation (optional)

?> **TIP** An example quickstart project for services is available on github : [https://github.com/adaptris/interlok-custom-component-example](https://github.com/adaptris/interlok-custom-component-example)

## Example ##

Our [previous example of a Connection](/pages/developer/developer-connections) defined our connection; now we need to work with the `ClientConnection` interface:

```java
public interface ClientConnection {
  String getMessage(String repository, long timeout) throws IOException, TimeoutException;
}
```


Our [AdaptrisMessageConsumer][] implementation can use `ClientConnection` to receive messages; but because of the timeout parameter/exception, we can assume that the consumer should be timer based.


```java
@XStreamAlias("my-client-consumer")
public class MyClientConsumer extends AdaptrisPollingConsumer {

  private static final TimeInterval DEF_INTERVAL = new TimeInterval(2L, TimeUnit.SECONDS.name());

  @Getter
  @Setter
  private TimeInterval receiveTimeout;

  @Getter
  @Setter
  private String repository;

  public MyClientConsumer() {
  }

  @Override
  protected int processMessages() {
    int msgCount = 0;
    AdaptrisMessageFactory factory = AdaptrisMessageFactory.defaultIfNull(getMessageFactory());
    try {
      ClientConnection conn = retrieveConnection(MyClientConnection.class).createConnection();
      do {
        String data = conn.getMessage(getRepository(), receiveTimeoutMs());
        msgCount ++;
        AdaptrisMessage msg = factory.newMessage(data);
        retrieveAdaptrisMessageListener().onAdaptrisMessage(msg);
        if (!continueProcessingMessages()) {
          break;
        }
      }
      while (true);
    }
    catch (TimeoutException e) {
      log.trace("No More Messages");
    }
    catch (IOException e) {
      log.error(e.getMessage(), e);
    }
    return msgCount;
  }

  long receiveTimeoutMs() {
    return getReceiveTimeout() != null ? getReceiveTimeout().toMilliseconds() : DEF_INTERVAL.toMilliseconds();
  }

  public TimeInterval getReceiveTimeout() {
    return receiveTimeout;
  }

  public void setReceiveTimeout(TimeInterval receiveTimeout) {
    this.receiveTimeout = receiveTimeout;
  }

  @Override
  protected void prepareConsumer() throws CoreException {}
}
```

So, the summary of what we did is as follows :

- We extended [com.adaptris.core.AdaptrisPollingConsumer][AdaptrisPollingConsumer] and implemented the required methods.

?> **NOTE** We have not overridden any lifecycle methods; remember to call the super-class lifecycle methods if lifecycle is required.

- In our example we're going to pull messages from a ficticious "repository", so we have a new field we have named repository; and because repository has public getters and setters this field will be configurable through both the Interlok configuration XML and UI.

?> **TIP** There is also a filter expression available (which may return null).

- We call `retrieveConnection` to find the configured connection object.
- We don't throw an exception; but we log it.
- We return the number of messages processed (which is logged).
- We provide a sensible default for the timeout if it is not configured.
- We use `continueProcessingMessages()` to check if we should process the next message or not

?> **NOTE** This is directly related to [reacquireLockBetweenMessages][] from the parent consumer implementation.

- We use [AdaptrisMessageFactory][] directly as we do not support [AdaptrisMessageEncoder][].
- We trigger the workflow via `retrieveAdaptrisMessageListener().onAdaptrisMessage()`
- An `@XStreamAlias` is added so that we have an alias that we can configure; so now, configuration is `<consumer class="my-client-consumer"/>` rather than the fully qualified classname.



[AdaptrisMessageConsumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisMessageConsumer.html
[AdaptrisMessageConsumerImp]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisMessageConsumerImp.html
[AdaptrisPollingConsumer]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisPollingConsumer.html
[AdaptrisConnection]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisConnection.html
[AdaptrisConnectionImp]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisConnectionImp.html
[AdaptrisMessageEncoder]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisMessageEncoder.html
[AdaptrisMessageFactory]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisMessageFactory.html
[reacquireLockBetweenMessages]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/AdaptrisPollingConsumer.html#setReacquireLockBetweenMessages-java.lang.Boolean-
