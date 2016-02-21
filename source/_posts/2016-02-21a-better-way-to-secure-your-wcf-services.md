title: a better way to secure your wcf services
date: 2016-02-21 11:15:06
categories:
- coding
tags:
- windows
- wcf
- .net
---
## using inspectors, you could add headers to the wcf message, and then you could do authentication.
{% codeblock lang:csharp %}
/// <summary>
/// Inspect client messages : add GUID in headers
/// </summary>
internal class CProcessAuthenticationClientInspector : IClientMessageInspector
{

    #region IClientMessageInspector Membres

    public void AfterReceiveReply(ref System.ServiceModel.Channels.Message reply, object correlationState)
    {
    }

    public object BeforeSendRequest(ref System.ServiceModel.Channels.Message request, System.ServiceModel.IClientChannel channel)
    {
        request.Headers.Add(MessageHeader.CreateHeader("ProcessAuth", "http://schemas.YOURCOMPANY.com/YOURAPPID", CProcessAuthenticationBehavior._authToken));
        return null;
    }

    #endregion
}

/// <summary>
/// Inspect server messages : Check GUID
/// </summary>
internal class CProcessAuthenticationDispatchInspector : IDispatchMessageInspector
{

    #region IDispatchMessageInspector Membres

    public object AfterReceiveRequest(ref Message request, System.ServiceModel.IClientChannel channel, System.ServiceModel.InstanceContext instanceContext)
    {
        Guid token = OperationContext.Current.IncomingMessageHeaders.GetHeader<Guid>("ProcessAuth", "http://schemas.YOURCOMPANY.com/YOURAPPID");
        if (token != CProcessAuthenticationBehavior._authToken)
            throw new Exception("Invalid process");
        return null;
    }

    public void BeforeSendReply(ref Message reply, object correlationState)
    {

    }

    #endregion
}

/// <summary>
/// Add inspectors on both client and server messages
/// </summary>
public class CProcessAuthenticationBehavior : IEndpointBehavior
{
    /// <summary>
    /// Authentification token known by both sides of the pipe
    /// </summary>
    internal static Guid _authToken = Guid.NewGuid();

    #region IEndpointBehavior Membres

    public void AddBindingParameters(ServiceEndpoint endpoint, System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
    {
    }

    public void ApplyClientBehavior(ServiceEndpoint endpoint, System.ServiceModel.Dispatcher.ClientRuntime clientRuntime)
    {
        clientRuntime.MessageInspectors.Add(new CProcessAuthenticationClientInspector());
    }

    public void ApplyDispatchBehavior(ServiceEndpoint endpoint, System.ServiceModel.Dispatcher.EndpointDispatcher endpointDispatcher)
    {
        endpointDispatcher.DispatchRuntime.MessageInspectors.Add(new CProcessAuthenticationDispatchInspector());
    }

    public void Validate(ServiceEndpoint endpoint)
    {
    }

    #endregion
}
{% endcodeblock %}

And then you just need to add your endpoint behaviour to your endpoint on both sides :

client :
{% codeblock lang:csharp %}
    ChannelFactory<TInterface> factory;
    factory = new ChannelFactory<TInterface>(BuildLocalBinding(), "net.pipe://localhost/foo");
    factory.Endpoint.Behaviors.Add(new CProcessAuthenticationBehavior());
{% endcodeblock %}

server :
{% codeblock lang:csharp %}
    ServiceHost svcHost = new System.ServiceModel.ServiceHost(imlpementationType);
    svcHost.AddServiceEndpoint(interfaceType, binding, "net.pipe://localhost/foo");
    svcHost.Description.Endpoints[0].Behaviors.Add(new CProcessAuthenticationBehavior());
    Well... this may be done in config, but I'll let you dig :)
{% endcodeblock %}
