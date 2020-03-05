# DLB.EventBus.Transport

## Use

### Register Kafka transport and subscribers
```csharp
public void ConfigureServices(IServiceCollection services)
{
            var eventBusSettings = GetSettings();

            services.AddTransport(x =>
            {
                x.UseKafka(opt =>
                {
				    // If you wish define your custom kafka settings
                    //cnf.MainConfig = new Confluent.Kafka.ClientConfig(new Dictionary<string, string>());
                    opt.Servers = eventBusSettings.Servers;
                    opt.SSLCertificatePath = eventBusSettings.SSLCeriticatePath;
                    opt.SecurityProtocol = KafkaSecurityProtocol.PlainText;
                });

                x.OnLogError += X_OnLogError;

            }).RegisterSubscriber<HelloIntegrationEventHandler>();

            services.AddControllers();
}
```

### Subscriber class

With DLB.EventBus.Transport.Kafka you can subscribe to multiple events in the same topic. Only you need define the class and implement interface ITransportSubscriber and set topic name abstract property.

When you are subscribe to multiple events, the handler who was execute it's determinate with name of the event (ClassName).

In the below example you can implements many ITransportSubscriber in the same class to received multiple events in the same  topic.

```csharp
public class HelloIntegrationEventHandler : ITransportSubscriber<HelloIntegrationEvent>
{
        private readonly ILogger<HelloIntegrationEventHandler> logger;

        public HelloIntegrationEventHandler(ILogger<HelloIntegrationEventHandler> logger)
        {
            this.logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public string Topic => "hello_world";

        public Task HandleAsync(HelloIntegrationEvent @event, IHandleContext context, object sender)
        {
            this.logger.LogInformation(context.MessageContext.GetId());
            this.logger.LogInformation($"Msg value: {@event.Value}");

            return Task.CompletedTask;
        }
}

public class HelloIntegrationEvent
{
	public string Value { get; set; }
}
```

### Publisher

DLB.EventBus.Transport.Kafka allow to set PartitionKey when you publish a new message in a topic. Only you need inject class 'ITransportPublisher':

```csharp
public HelloWorldController(
                               ILogger<HelloWorldController> logger,
                               ITransportPublisher transportPublisher)
        {
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
            _transportPublisher = transportPublisher ?? throw new ArgumentNullException(nameof(transportPublisher));
        }
```

###Publish message with PartitionKey

```csharp
[HttpGet("{id}")]
        public async Task<string> Get(string id)
        {
            var integrationEvent = new HelloWorldIntegrationEvent()
            {
                Id = id,
                Value = "Hello world!",
            };

            // Send mensaje to Kafka with specific partition key.
            var result = await _transportPublisher.PublishAsync(helloWorldTopic, integrationEvent, p => p.Id);

            return $"Success: {result.Succeeded}";
        }
```

###Publish message without PartitionKey

```csharp
public async Task<string> PublishWithoutPartitionKey(string name)
        {
            var integrationEvent = new HelloNewSchemaIntegrationEvent()
            {
                CustomerName = name,
                Counter = 1,
            };

            // Send mensaje to Kafka without specific partition key.
            var result = await _transportPublisher.PublishAsync(helloWorldTopic, integrationEvent);

            return $"Success: {result.Succeeded}";
        }
```