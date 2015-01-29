# porcupine
Threading, Resiliency and Monitoring for Java EE 7

##Installation

```
        <dependency>
            <groupId>com.airhacks</groupId>
            <artifactId>porcupine</artifactId>
            <version>NEWEST_VERSION</version>
            <scope>compile</scope>
        </dependency>
```

##Conventional Usage

```
@Stateless
public class MessagesService {

    @Inject
    @Dedicated
    ExecutorService light;

    @Inject
    @Dedicated
    ExecutorService heavy;
```

##Statistics and monitoring

###Exposure

	@Path("statistics")
	@RequestScoped
	@Produces({MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON})
	public class StatisticsResource {

	    @Inject
	    Instance<List<Statistics>> statistics;

    	@GET
	    public List<Statistics> expose() {
	        return this.statistics.get();
    	}

	}
	
###Sample output

XML statistics

```
<statistics>
	<pipelineName>light</pipelineName>
	<remainingQueueCapacity>100</remainingQueueCapacity>
	<completedTaskCount>1</completedTaskCount>
	<activeThreadCount>0</activeThreadCount>
	<largestThreadPoolSize>1</largestThreadPoolSize>
	<currentThreadPoolSize>1</currentThreadPoolSize>
	<totalNumberOfTasks>1</totalNumberOfTasks>
	<maximumPoolSize>48</maximumPoolSize>
	<rejectedExecutionHandlerName>ExecutorServiceExposer$$Lambda$3/1562863014</rejectedExecutionHandlerName>
	<rejectedTasks>0</rejectedTasks>
</statistics>
<statistics>
	<pipelineName>heavy</pipelineName>
	<remainingQueueCapacity>16</remainingQueueCapacity>
	<completedTaskCount>0</completedTaskCount>
	<activeThreadCount>1</activeThreadCount>
	<largestThreadPoolSize>1</largestThreadPoolSize>
	<currentThreadPoolSize>1</currentThreadPoolSize>
	<totalNumberOfTasks>1</totalNumberOfTasks>
	<maximumPoolSize>8</maximumPoolSize>
	<rejectedExecutionHandlerName>CallerRunsPolicy</rejectedExecutionHandlerName>
	<rejectedTasks>0</rejectedTasks>
</statistics>
```
JSON statistics
```
[{
        "pipelineName": "light",
        "remainingQueueCapacity": 100,
        "completedTaskCount": 1,
        "activeThreadCount": 0,
        "largestThreadPoolSize": 1,
        "currentThreadPoolSize": 1,
        "totalNumberOfTasks": 1,
        "maximumPoolSize": 48,
        "rejectedExecutionHandlerName": "ExecutorServiceExposer$$Lambda$3/1562863014",
        "rejectedTasks": 0
    },
    {
        "pipelineName": "heavy",
        "remainingQueueCapacity": 16,
        "completedTaskCount": 1,
        "activeThreadCount": 0,
        "largestThreadPoolSize": 1,
        "currentThreadPoolSize": 1,
        "totalNumberOfTasks": 1,
        "maximumPoolSize": 8,
        "rejectedExecutionHandlerName": "CallerRunsPolicy",
        "rejectedTasks": 0
    }]
```
##Configuration

```
@Specializes
public class CustomExecutorConfigurator extends ExecutorConfigurator {

    @Override
    public ExecutorConfiguration defaultConfigurator() {
        return super.defaultConfigurator();
    }

    @Override
    public ExecutorConfiguration forPipeline(String name) {
        if ("heavy".equals(name)) {
            return new ExecutorConfiguration.Builder().
                    corePoolSize(4).
                    maxPoolSize(8).
                    queueCapacity(16).
                    keepAliveTime(1).
                    callerRunsPolicy().
                    build();
        }
        return super.forPipeline(name);
    }

}
```