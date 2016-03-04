## Azure web job.
 SDK: `Microsoft.Azure.WebJobs` `Microsoft.Azure.WebJobs.Extensions`
 
### Sample:
entry point
``` cs
public static void Main()
        {
            var config = new JobHostConfiguration();

            if (config.IsDevelopment)
            {
                config.UseDevelopmentSettings();
            }
             
            config.UseTimers(); // timer trigger which is defined in SDK extensions.

            JobHost host = new JobHost(config);
            host.RunAndBlock();
        }
```
QueueTrigger
``` cs
   public static void BindToQueueXXXXXX([QueueTrigger("xxxx")] XObject obj, TextWriter log)
        {
           // do stuff here
            log.Write("balabala ");
        }
```
QueueTrigger
``` cs
   public static void BindToBlobsXXXXX([BlobTrigger("xxxx")] XObject obj, TextWriter log)
        {
           // do stuff here
            log.Write("balabala ");
        }
```
# DSL in C# Try/NoException

``` cs
   Func<Action,Action> NoException = (action) =>
   {
       Action wapper = () =>
       {
           try
           {
               action.Invoke();
           }
           catch (Exception ex)
           {
 
               Console.WriteLine(ex);
           }
       };
 
       return wapper;
   };
```
# Docker
## play with boot2docker 
```
boot2docker delete
boot2docker init

boot2docker start

docker run hello-world
```
# Linqpad
[Some wonderful examples](http://linqpadsamples.codeplex.com/)

# Toolbox
`Linqpad` `C#` `F#` 
