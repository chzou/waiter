Waiter's behavior is customized through parameters specified by the HTTP client.

The basic parameters that Waiter supports are:

|Parameter|Required?|Default Value|Description|
|---------|---------|-------------|-----------|
|`X-Waiter-Cmd`|**Yes**|n/a|The shell command to run. Your application must listen on a port chosen by Waiter. The port your service should listen in is set via the `$PORT0` environment variable.|
|`X-Waiter-Cpus`|**Yes**|n/a|The number of cores your service needs.|
|`X-Waiter-Health-Check-Url`|No|"/status"|Specifies an HTTP endpoint that Waiter will hit to determine if your app is healthy. This should return `200 OK`.|
|`X-Waiter-Mem`|**Yes**|n/a|The amount of memory your service needs (MB). You will still need to explicitly include any memory flags (e.g. `-Xmx`) in your command.|
|`X-Waiter-Name`|No|empty string|A name for your service. Use only letters, numbers, dashes, underscores, and dots.|
|`X-Waiter-Run-As-User`|No|user who is making the request|The user that the service should be run as.|
|`X-Waiter-Version`|**Yes**|n/a|Specify a version to associate with your service (e.g. "a0b1c2d3e4f5" or "my-version-name").|

Additional (optional) parameters that can be set:

|Parameter|Default Value|Valid Values|Description|Guidance|
|---------|-------------|------------|-----------|--------|
|`X-Waiter-Authentication`|standard|disabled or standard|The authentication mechanism to use for incoming requests.|By default, Waiter authenticates incoming requests using the standard protocol (e.g. Kerberos). If you would prefer that Waiter not authenticate incoming requests, set this flag to disabled.|
|`X-Waiter-Backend-Proto`|http|http or https|The backend connection protocol to use.|By default, Waiter connects to backend instances using the HTTP protocol. If you would prefer that Waiter use HTTPS, feel free to set this parameter to https.|
|`X-Waiter-Blacklist-On-503`|true|true or false|If an instance returns 503, whether or not the instance will be blacklisted.|By default, Waiter avoids instances that are returning a 503 error code in responses, which typically indicates the server is too busy. If you would prefer that Waiter not do this, feel free to disable this feature.|
|`X-Waiter-Cmd-Type`|"shell"|"shell"|Provides an extension point for supporting different types of commands in the future.|Feel free to add new command types to suit your needs.|
|`X-Waiter-Concurrency-Level`|1|1-10000|The number of simultaneous requests to an individual backend instance.|Increasing this value will likely lead to better performance and better resource utilization. Avoid queuing your requests on your backend instances by not increasing concurrency level above what an individual instance can handle. The shorter your requests, the more benefit you will get out of a higher concurrency level.|
|`X-Waiter-Distribution-Scheme`|"balanced"|"balanced" or "simple"|The strategy used by Waiter routers to distribute instances amongst themselves. A "simple" distribution, achieved using consistent hashing, will minimize instances being assigned to multiple routers due to scaling. A "balanced" distribution ensures even splitting of instances across routers after the initial consistent hash distribution.|If you see too many blacklisted instances due to 503 responses, it may be a good idea to try out "simple" distribution.|
|`X-Waiter-Env`|none|A JSON map from string keys to string values|The keys and values correspond to environment variable names and values which will get passed to the command.|Use as needed.|
|`X-Waiter-Expired-Instance-Restart-Rate`|0.1|(0-1]|The max percentage of total expired instances to recycle at the same time (e.g. if the service has 10 expired instances and a `X-Waiter-Expired-Instance-Restart-Rate` of 0.1, Waiter will only restart 1 at a time).|The default is reasonable for most cases.|
|`X-Waiter-Grace-Period-Secs`|30|1-3600 (1 hour)|The amount of time to wait after a service starts before considering a health check failure to be an error.|Set this value to startup time of your service. If your service starts up within 30 seconds, don't bother with changing this value.|
|`X-Waiter-Idle-Timeout-Mins`|4320 (3 days)|1-10080 (1 week)|The number of minutes of no activity to wait before shutting down a service completely.|Setting a lower idle timeout is strongly encouraged and frees cluster resources for other services to use.|
|`X-Waiter-Instance-Expiry-Mins`|7200 (5 days)|>0|The amount of time to allow a particular instance of your app to run before restarting it. Waiter ensures another instance is available before killing the expiring instance such that service is unaffected.|You can set this value to be as large as you like.|
|`X-Waiter-Jitter-Threshold`|0.5|[0,1)|How big the difference between ideal and current number of instances must be for Waiter to scale your service.|The default is reasonable for most cases.|
|`X-Waiter-Max-Instances`|500|1-500|The maximum number of instances that Waiter should not scale above.|Setting a max can be helpful for preventing unexpected scaling.|
|`X-Waiter-Max-Queue-Length`|1000|>0|The number of requests to queue before rejecting requests with a 503 error code.|Lowering this value to something reasonable for your architecture is encouraged as it allows you to fail faster. Don't increase the value without a good reason.|
|`X-Waiter-Metadata-*`|n/a|any string|A metadata field. Metadata fields allow you to store arbitrary strings along with your service description.|Allows additional information to be stored with a service to help identify the service.|
|`X-Waiter-Metric-Group`|"other"|[A-Za-z0-9-_]+|Metric groups allow services to be grouped together for the purpose of metric collection.|If you have `:statsd` enabled in your [config file](../config-full.edn), then specify a metric group. If you have multiple services, choose the same metric group if you'd like combined metrics, otherwise choose independent metric groups.|
|`X-Waiter-Min-Instances`|1|1 or 2|The minimum number of instances that Waiter should keep running at all times.|Go with the default.|
|`X-Waiter-Permitted-User`|user who is making the request|any user|The user that is authorized to make requests to your service|Use as needed.|
|`X-Waiter-Ports`|1|[1-10]|The number of ports needed by the service.|Use as needed, only restriction is that the first port must always be used for a web server responding to healthchecks and incoming web requests.|
|`X-Waiter-Queue-Timeout`|300000 (5 mins)|>0|The maximum time, in milliesconds, allowed spent by a request waiting for an available service backend.|Don't change it unless you have a good reason to do so.|
|`X-Waiter-Restart-Backoff-Factor`|2|>=1|The factor by which to multiple the amount of time between consecutive start-up failures. The base is 1 second.|The default is reasonable for most cases.|
|`X-Waiter-Scale-Down-Factor`|0.1|(0-1]|The percentage amount, per second, to decrease the current number of instances toward the target number of instances. See [How Waiter Autoscaling Works](autoscaling.md).|Don't change it unless you have a good reason to do so.|
|`X-Waiter-Scale-Factor`|1|(0-1]|The ratio of desired number of backends to number of outstanding requests. See [How Waiter Autoscaling Works](autoscaling.md).|Don't change it unless you have a good reason to do so, although values lower than 1 promote more efficient usage of resources if you don't mind waiting.|
|`X-Waiter-Scale-Up-Factor`|0.01|(0-1]|The percentage amount, per second, to increase the current number of instances toward the target number of instances. See [How Waiter Autoscaling Works](autoscaling.md).|Don't change it unless you have a good reason to do so.|
|`X-Waiter-Timeout`|900000 (15 mins)|>0|The socket timeout, in milliesconds, of the connection between Waiter and a service backend.|Don't change it unless you have a good reason to do so.|
