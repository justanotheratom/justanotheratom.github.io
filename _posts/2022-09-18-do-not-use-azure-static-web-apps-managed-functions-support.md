---
title: Do not use Azure Static Web Apps Managed Functions Support
---

 

Azure Static Web Apps provide some nifty features for hosting [Backend APIs
](https://learn.microsoft.com/en-us/azure/static-web-apps/apis-overview)-
AuthN/AuthZ and Routing. However, the [Free Tier
](https://azure.microsoft.com/en-gb/pricing/details/app-service/static/#pricing)only
supports Managed Azure Functions APIs. While Managed Functions APIs are pretty
easy to get started with, I ran into some blockers due to which I had to pursue
other free routes for my hobby project.

 

I could not access Diagnostics logs from my Azure Function
----------------------------------------------------------

Documentation states that you can access logs by turning on Application Insights
on your Static Web App. The Portal does have an Application Insights Setting,
but it simply does not seem to stay enabled.

 

Managed Functions cannot integrate with Azure SignalR
-----------------------------------------------------

Azure Functions can be [configured
](https://learn.microsoft.com/en-us/azure/azure-signalr/concept-upstream)to
receive messages and events from the Azure SignalR Service. This requires
retrieving an [API Key
](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-signalr-service-trigger?tabs=in-process&pivots=programming-language-csharp#signalr-service-integration)from
the Function App and supplying it to SignalR Service. This API Key is not
visible for Managed Functions. So, I don’t think this integration is possible.

 

References

-   My failed [Pull Request
    ](https://github.com/justanotheratom/TicTacToe/pull/6)trying to integrate
    Managed Functions with SignalR Service.
