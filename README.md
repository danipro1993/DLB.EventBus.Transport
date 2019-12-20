# DLB.EventBus.Transport

## Use

### Register Kafka transport and subscribers
```csharp
public void ConfigureServices(IServiceCollection services)
{
            var eventBusSettings = GetSettings();

            services.AddTransport(x =>
            {
                opt.UseKafka(cnf =>
                {
                    cnf.MainConfig.BootstrapServers = eventBusSettings.Servers;
                    cnf.MainConfig.SslCaLocation = eventBusSettings.SSLCeriticatePath;
                });

                opt.DefaultGroup = "default_group"; // Optional
                opt.OnLog += Transport_OnLog;
                opt.OnLogError += Transport_OnLogError;

            }).RegisterSubscriber<HelloIntegrationEventHandler>();

            services.AddControllers();
}
```

### Subscriber class

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
