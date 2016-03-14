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
``` cs
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
```

And then you just need to add your endpoint behaviour to your endpoint on both sides :

client :
``` cs
    ChannelFactory<TInterface> factory;
    factory = new ChannelFactory<TInterface>(BuildLocalBinding(), "net.pipe://localhost/foo");
    factory.Endpoint.Behaviors.Add(new CProcessAuthenticationBehavior());
```

server :
``` cs
    ServiceHost svcHost = new System.ServiceModel.ServiceHost(imlpementationType);
    svcHost.AddServiceEndpoint(interfaceType, binding, "net.pipe://localhost/foo");
    svcHost.Description.Endpoints[0].Behaviors.Add(new CProcessAuthenticationBehavior());
    Well... this may be done in config, but I'll let you dig :)
```

## even more
``` cs
  using System;
    using System.Linq;
    using System.ServiceModel;
    using System.ServiceModel.Channels;
    using System.ServiceModel.Description;
    using System.ServiceModel.Dispatcher;

    internal class SecurityInspectors : IClientMessageInspector, IDispatchMessageInspector
    {
        static readonly string HEADER_NAME = "token";
        static readonly string HEADER_NS = "token-ns";
        void IClientMessageInspector.AfterReceiveReply(ref Message reply, object correlationState)
        {
            Verify(reply);
        }
        object IClientMessageInspector.BeforeSendRequest(ref Message request, IClientChannel channel)
        {
            // chanlenging provider
            ApplyChallenge(ref request);
            return null;
        }

        object IDispatchMessageInspector.AfterReceiveRequest(ref Message request, IClientChannel channel, InstanceContext instanceContext)
        {
            Verify(request);
            return null;
        }

        void IDispatchMessageInspector.BeforeSendReply(ref Message reply, object correlationState)
        {
            // chanlenging client
            ApplyChallenge(ref reply);
        }

        private static void Verify(Message message)
        {
            if (!message.Headers.Any(h => h.Name == HEADER_NAME && h.Namespace == HEADER_NS))
                throw new AppAuthenticationException(new FaultReason("API was called by an untrust application"));

            if (message.Headers.GetHeader<string>(HEADER_NAME, HEADER_NS) != HashSalt())
                throw new AppAuthenticationException(new FaultReason("API was called by an untrust application"));
        }

        private static void ApplyChallenge(ref Message message)
        {
            var token = MessageHeader.CreateHeader(HEADER_NAME, HEADER_NS, HashSalt());
            if (message.Headers.Any(h => h.Name == token.Name))
            {
                message.Headers.RemoveAll(token.Name, token.Namespace);
            }
            message.Headers.Add(token);
        }

        private static string HashSalt()
        {
            /// do some magic here to generate a device specific ex： the hash of st
            return GetSalt() + "sff";
        }

        private static string GetSalt()
        {
            return "aaaaabb";
        }
    }

    public class AppAuthenticationException : FaultException
    {
        public AppAuthenticationException(FaultReason reason = null, FaultCode code = null)
            : base(reason, code)
        {
        }
    }

    /// <summary>
    /// This is a wcf operation contract attribute.
    /// It could be attached to an operation contract, will apply two way authentication, and make sure
    /// the wcf service and the client application is both trusted.
    /// for invalid access, there will be a exception `AppAuthenticationException` thrown.
    /// </summary>
    public class AuthenticationRequiredAttribute : Attribute, IEndpointBehavior, IOperationBehavior
    {
        public AuthenticationRequiredAttribute()
        {
#if DEBUG
#else
            var cert1 = System.Reflection.Assembly.GetEntryAssembly().ManifestModule.GetSignerCertificate();
            var cert2 = this.GetType().Assembly.ManifestModule.GetSignerCertificate();
            if (cert2 == null || 
                cert2 == null || 
                cert1.GetCertHashString() != cert2.GetCertHashString())
                throw new AppAuthenticationException();
#endif
        }
        #region IEndpointBehavior Members  
        public void AddBindingParameters(ServiceEndpoint endpoint, BindingParameterCollection bindingParameters) { }
        public void ApplyClientBehavior(ServiceEndpoint endpoint, ClientRuntime clientRuntime)
        {
            clientRuntime.MessageInspectors.Add(new SecurityInspectors());
        }
        public void ApplyDispatchBehavior(ServiceEndpoint endpoint, EndpointDispatcher endpointDispatcher)
        {
            endpointDispatcher.DispatchRuntime.MessageInspectors.Add(new SecurityInspectors());
        }
        public void Validate(ServiceEndpoint endpoint) { }
        #endregion

        #region IOperationBehavior Members  
        public void AddBindingParameters(OperationDescription operationDescription, BindingParameterCollection bindingParameters) { }
        public void ApplyClientBehavior(OperationDescription operationDescription, ClientOperation clientOperation)
        {
            clientOperation.Parent.MessageInspectors.Add(new SecurityInspectors());
        }
        public void ApplyDispatchBehavior(OperationDescription operationDescription, DispatchOperation dispatchOperation)
        {
            dispatchOperation.Parent.MessageInspectors.Add(new SecurityInspectors());
        }
        public void Validate(OperationDescription operationDescription) { }
        #endregion
    }
```

usage, just add attribute on the contract, no changes on the client.

``` cs
[ServiceContract]
interface IService
{
    [OperationContract]
    [AuthenticationRequired]
    string SayHelloTo(string name);
}
```
