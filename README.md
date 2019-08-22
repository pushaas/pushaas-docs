<div align="center">
  <img width="200px" src="./pushaas.png">
</div>
<div align="center">

# PushaaS docs

</div>

[PushaaS](https://github.com/pushaas) is a project that aims to offer [server-push](https://en.wikipedia.org/wiki/Push_technology) functionality as a [service](https://docs.tsuru.io/stable/services/index.html) for the [Tsuru](https://tsuru.io/) PaaS, based on tools built around the [Push Stream](https://www.nginx.com/resources/wiki/modules/push_stream/) module for [Nginx](https://www.nginx.com/).

The system that implements the [Push Service](#push-service) can also be deployed manually outside of Tsuru, as a standalone server-push system.


## Concepts

Lets just clarify what we call "Push Service" and "PushaaS".

### Push Service

The **Push Service** is a system (composed of 4 components detailed later) that provides the server-push functionality for client applications. It's components operate on a pub-sub model that allows:
  - subscribers (usually users consuming some web application) to listen for messages on a channel in order to receive real-time data
  - publishers (usually applications or users that can produce content for the web application) to publish messages on specified channels, to be delivered to the subscribers

The main component is the Nginx Push Stream module, a battle-tested and very performant module where subscribers and publishers connect to consume and produce messages. Other tools were built around it to expand its functionality, ease publishing and provide stats and authentication for publishers. These tools together compose what we call **Push Service**.

### PushaaS

While an instance of the Push Service is useful on its own and can be manually made available to client applications, it doesn't abstract away from the application developer the provisioning of the infrastructure needed to run a Push Service instance.

The **PushaaS** is a service to provision automatically **Push Service** instances for client applications. Thus, the application developer can easily provision a Push Service instance for each application that needs the server-push functionality. Very much like you would do with a Database-as-a-Service system.

The PushaaS talks to an infrastructure provider and runs the components of a Push Service instance, and make them available for client applications. Most importantly, PushaaS does this as a service for the Tsuru open-source PaaS. So applications running on top of a Tsuru cluster can be bound to a Push Service instance and use it to implement server-push functionality in an easy and scalable fashion.


## Components

This project documents all the pieces that compose the PushaaS ecosystem. All projects in the ecosystem can be found at https://github.com/pushaas.

- Push Service: is composed of 4 components that collaborate to implement the service. The components can be deployed by the [`pushaas`](#component-pushaas) being called as a Tsuru service. If you are not using Tsuru, you can still use these 4 components to have an instance of a Push Service. You can deploy them manually, or using containers, or some other tool. [`push-service`](https://github.com/pushaas/push-service) contains a sample setup with Docker Compose that can be used as a reference.
  - <span name="component-push-stream">[`push-stream`](https://github.com/pushaas/push-stream)</span>
  - <span name="component-push-api">[`push-api`](https://github.com/pushaas/push-api)</span>
  - <span name="component-push-redis">[`push-redis`](https://github.com/pushaas/push-redis)</span>
  - <span name="component-push-agent">[`push-agent`](https://github.com/pushaas/push-agent)</span>

- PushaaS
  - <span name="component-pushaas">[`pushaas`](https://github.com/pushaas/pushaas)</span>
- Libraries
  - [`push-api-client-javascript`](https://github.com/pushaas/push-api-client-javascript): a JavaScript library to make calls to an instance of the [`push-api`](#push-api). To be used by client applications
- Demo applications
  - [`push-stream-demo-app`](https://github.com/pushaas/push-stream-demo-app)
  - [`push-service-demo-app`](https://github.com/pushaas/push-service-demo-app)
- Infrastructure configuration
  - [`pushaas-aws-ecs-config`](https://github.com/pushaas/pushaas-aws-ecs-config)


## Architecture

TODO


## Current State and Limitations

- no scaling commands, though architecture supports it
- provisioners: ECS
