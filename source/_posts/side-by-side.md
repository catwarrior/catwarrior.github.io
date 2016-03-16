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
## Redis cache
For better performance
``` cs
private static ConnectionMultiplexer connection;
private static ConnectionMultiplexer Connection 
{
    get
    {
        if(connection == null || !connection.IsConnected)
        {
            connection = ConnectionMultiplexer.Connect("yhdcache0.redis.cache.windows.net,ssl=true,password=...");
        }
        return connection;
    }
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

`emacs` `autohotkey`

# .Net
[Common question for C#/.NET](http://www.c-sharpcorner.com/UploadFile/1492b1/C-Sharp-and-Asp-Net-question-and-answersall-in-one/)

[Awesome dotnet](https://github.com/catwarrior/awesome-dotnet)

# Nuget
ex:
`Install-Package Microsoft.AspNet.WebApi -Version 5.2.2`
`Update-Package Microsoft.AspNet.WebApi -reinstall`

# Fsharp
`paket` `fake` `Argu - cli argument parsing`
https://github.com/fsprojects
http://fssnip.net/

# Install all stuff with one click
```bash
START http://boxstarter.org/package/nr/url?https://raw.githubusercontent.com/catwarrior/garage/master/install.txt
```
