title: Create a smart wcf client wrapper
date: 2015-11-12 18:57:25
categories:
- Coding
tags:
- Windows
- WCF
- .NET
---

## A good start for wcf client programing.

In the last article we presented a solution for calling WCF services that needed only a single line of code and no using statements.  But it had many limitations including

- Reliance on a static class
- Testability was low for any code using it
- Reliance on the WCF client rather than the contract
- Not optimized for multiple calls
In reality the previous series of articles where all about setting up the infrastructure to support the final solution.  All this infrastructure will be hidden from code when we are done.  Let’s get started.

## Service Template
Ultimately we will be using T4 to generate service clients but the templates are complex and we do not yet even have an idea of what we’re building.  For this article we will build the final service client by hand.  In subsequent articles we will convert the code to a template. 

The service client should meet the following goals

- Creatable using a standard new operator with no requirements for specifying any WCF endpoint information.
- Implement the WCF contract interface so instances can be used as parameters such as in an IoC.
- Instances do not need to be disposed.
- Should allow multiple calls to be optimized through the same client.
- Extensible to allow callers to adjust the client if needed.

## ServiceChannelClient

To start we will create a new abstract class that will serve as the base class for service clients.  While we could reuse the ServiceClientWrapper of previous articles that would expose too much WCF infrastructure.  Instead the class will wrap calls to `ServiceClientWrapper` and `ServiceClientFactory` from the previous articles.  All the functionality is protected since this is an abstract class.

{% codeblock lang:c# %}
public abstract class ServiceChannelClient<TChannel> where TChannel: class 
{ 
   protected virtual ServiceClientWrapper<TChannel> CreateInstance () 
   { return ServiceClientFactory.CreateAndWrap<TChannel>(); } 

   protected virtual void InvokeMethod ( Action<TChannel> action ) 
   { ServiceClientFactory.InvokeMethod<TChannel>(action, CreateInstance); } 

   protected virtual TResult InvokeMethod<TResult> ( Func<TChannel, TResult> action ) 
   { return ServiceClientFactory.InvokeMethod<TChannel, TResult>(action, CreateInstance); } 
} 
{% endcodeblock %}

Basically the class just wraps the classes from previous articles.  It is about to get more functionality but first let’s implement a WCF service client using the class.

## Typed Service Clients
Creating a class to wrap a WCF service is now pretty straightforward (depending upon the size of the interface).  We need only create a derived type and implement the interface using the methods provided in the base class.
{% codeblock lang:c# %}
public class EmployeeServiceClient : ServiceChannelClient<IEmployeeService> , IEmployeeService 
{ 
   public Employee Get ( int id ) 
   { return InvokeMethod<Employee>(c => c.Get(id)); } 

   public Employee Update ( Employee employee ) 
   { return InvokeMethod<Employee>(c => c.Update(employee)); }  

   public IEnumerable<Employee> GetEmployees () 
   { return InvokeMethod<IEnumerable<Employee>>(c => c.GetEmployees()); } 
}
{% endcodeblock %}
If we wanted we could make all the methods virtual to allow for extensibility.  We could also mark the type as partial to extend its abilities but the above code is sufficient until we move to T4. 

Switching the application over to use the new client is straightforward.

- Remove the service reference code
- Add the service interface assembly to the project
- Replace all the existing calls with the new type
When the service reference code is removed it will remove the configuration entry for the service as well so that information will need to be put back in.  For now ensure the endpoint name matches the class name and that the contract name matches the namespace name of the interface.

{% codeblock lang:xml %}
<!—- Original -–> 
<client>  
    <endpoint address="http://localhost:21054/EmployeeService.svc" 
                 binding="basicHttpBinding"
                 bindingConfiguration="BasicHttpBinding_IEmployeeService" 
                 contract="ServiceReference1.IEmployeeService" 
                 name="BasicHttpBinding_IEmployeeService" />
</client> 

<!—- New -–> 
<client> 
    <endpoint address="http://localhost:21054/EmployeeService.svc" 
                 binding="basicHttpBinding" 
                 bindingConfiguration="BasicHttpBinding_IEmployeeService" 
                 contract="ServiceLib.IEmployeeService"
                 name="EmployeeServiceClient" /> 
</client>
{% endcodeblock %}

The matching of the endpoint name with the client type name is only a convenience.  We can add a constructor that allows the endpoint name to be specified if we want.  Finally we can fix up the calls to the client.

{% codeblock lang:c# %}
//Original 
ServiceClientFactory.InvokeMethod<ServiceReference1.IEmployeeService>(c => 
{ 
   dataGridView1.DataSource = c.GetEmployees(); 
}); 

//New 
dataGridView1.DataSource = Client.GetEmployees(); 

//Poor man's injection 
private IEmployeeService Client 
{ 
    get { return m_client.Value; } 
} 

private readonly Lazy<IEmployeeService> m_client = new Lazy<IEmployeeService>(() => new EmployeeServiceClient());
{% endcodeblock %}

Now that we don’t need to dispose of objects or rely on static classes we can use just the interface which makes testing much easier.

## Multiple Calls
We’re almost done but we want to make one more change.  WCF service calls are not cheap so most calls tend to do several things at once rather than focusing on a single task like standard OOP would have you do.  But there are times when multiple calls are needed and the above code would require 2 client connections to be created.  Take, for example, this code in the sample application that creates 2 connections just to update an employee.

{% codeblock lang:c# %}
private void OnNewEmployee ( object sender, EventArgs e ) 
{ 
   var dlg = new EmployeeForm();  
   if (dlg.ShowDialog(this) != DialogResult.OK) 
      return; 

   //Call the service 
   Client.Update(dlg.Employee); 
   LoadEmployees(); 
} 

private void LoadEmployees ( ) 
{ 
   dataGridView1.DataSource = Client.GetEmployees(); 
}
{% endcodeblock %}

What we want to do is allow the client to tell us they plan to make multiple calls so we can optimize the client creation.  Since the client needs to be able to define the lifetime of the calls it makes sense to use a using statement.  All we need to do is provide a mechanism for creating a type that controls the lifetime of our client.  We can add this directly to the base client.

{% codeblock lang:c# %}
public abstract class ServiceChannelClient<TChannel> : 
     ISupportsPersistentChannel where TChannel: class 
{ 
   protected bool HasOpenChannel 
   { 
      get { return m_channel != null; } 
   }  

   protected virtual void CloseChannel () 
   { 
      var channel = Interlocked.Exchange(ref m_channel, null); 

      if (channel != null)  
          channel.Close(); 
   } 

   protected virtual ServiceClientWrapper<TChannel> CreateInstance () 
   { return ServiceClientFactory.CreateAndWrap<TChannel>(); 
} 
{% endcodeblock %}

Now the base class can have a temporary or permanent connection open depending upon the caller’s needs.  But how do we expose this to the caller?  Introducing the `ISupportsPersistentChannel` interface.  The sole purpose of this interface is to expose Open/Close methods.  Notice the methods are not exposed publicly but only through the interface.  This prevents callers from calling these methods accidentally.  Why is this important?  Why do we need an interface?  It’s all about interchangeability.

## Creating A Persistent Channel
Remember that one of the requirements is that we want to be able to use the service interface directly without regards to the implementation, including mocks.  Therefore persistence cannot require that we use the client class.  Instead we add an extension to the interface which gives us the ability to open a persisted channel irrelevant of the implementation.  The caller code will work the same whether we’re using our client type or a test mock.  Here’s the implementation.

{% codeblock lang:c# %}
public static class EmployeeServiceExtensions 
{ 
   public static IDisposable OpenPersistentChannel ( this IEmployeeService source ) 
   { 
      var batch = source as ISupportsPersistentChannel; 

      if (batch != null) 
      { 
         batch.Open();  

         return new ActionDisposable(batch.Close); 
      }; 
       
      return Disposable.Empty; 
   } 
}
{% endcodeblock %}

The key is that the extension method will look for the `ISupportsPersistentChannel` interface.  For types deriving from `ServiceChannelClient` the interface will exist.  For other types it likely will not.  If the interface does exist then the channel is opened and an instance of a disposable class is returned that will close the channel.  Note the client itself is never returned.  If the interface does not exist then a disposable object is returned that does nothing.  This makes it safe for both production and test code.  Here’s how the caller would use it.

{% codeblock lang:c# %}
private void OnNewEmployee ( object sender, EventArgs e ) 
{ 
   var dlg = new EmployeeForm(); 
   if (dlg.ShowDialog(this) != DialogResult.OK) 
      return; 

   //Call the service 
   using (var proxy = Client.OpenPersistentChannel())   
   { 
      Client.Update(dlg.Employee); 
      LoadEmployees(Client);  
   }; 
} 

private void LoadEmployees ( ) 
{ 
   LoadEmployees(Client); 
} 

private void LoadEmployees ( IEmployeeService service ) 
{ 
   dataGridView1.DataSource = service.GetEmployees(); 
}
{% endcodeblock %}

## Next Steps

We are finally at the point where we have replaced the original service reference logic with a fully testable client that eliminates the dependency on WCF plumbing.  The resultant code is clean and can be mocked.  In the general case of single calls to WCF then the caller does not need to anything special.  For multiple calls the client (can) optimize performance with a `using` statement and it won’t break any test implementations.  The only issue remaining is the amount of code we’d need to generate for each WCF service.  The example client we created can be converted to a T4 template but it is going to require quite a bit of work.  However once we’re done we won’t have to revisit this code ever again.  We’ll pick up T4 next time.

Original from : 
http://blogs.msmvps.com/p3net/2014/03/15/a-smarter-wcf-service-client-part-4/
http://blogs.msmvps.com/p3net/2014/05/18/a-smarter-wcf-service-client-part-5/