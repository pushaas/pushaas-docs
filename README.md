<div align="center">
  <img width="200px" src="./pushaas.png">
</div>
<div align="center">

# PushaaS docs

</div>

[PushaaS](https://github.com/pushaas) is a project that aims to offer [server-push](https://en.wikipedia.org/wiki/Push_technology) functionality as a [service](https://docs.tsuru.io/stable/services/index.html) for the [Tsuru](https://tsuru.io/) PaaS, based on tools built around the [Push Stream](https://www.nginx.com/resources/wiki/modules/push_stream/) module for [Nginx](https://www.nginx.com/).

The system that implements the [Push Service](#push-service) can also be deployed manually outside of Tsuru, as a standalone server-push system.


## Concepts

Lets just clarify what "Push Service" and "PushaaS" are.

### Push Service

The **Push Service** is a system (composed of 4 components detailed later) that provides the server-push functionality for client applications. It's components operate on a pub-sub model that allows:
  - subscribers (usually users consuming some web application) to listen for messages on a channel in order to receive real-time data
  - publishers (usually applications or users that can produce content for the web application) to publish messages on specified channels, to be delivered to the subscribers

The main component is the Nginx Push Stream module, a battle-tested and very performant module where subscribers and publishers connect to consume and produce messages. Other tools were built around it to expand its functionality, ease publishing and provide stats and authentication for publishers. These tools together compose the **Push Service**.

### PushaaS

While an instance of the Push Service is useful on its own and can be manually made available to client applications, it doesn't abstract away from the application developer the provisioning of the infrastructure needed to run a Push Service instance.

The **PushaaS** is a service to provision automatically **Push Service** instances for client applications. Thus, the application developer can easily provision a Push Service instance for each application that needs the server-push functionality. Very much like you would do with a Database-as-a-Service system.

The PushaaS talks to an infrastructure provider and runs the components of a Push Service instance, and make them available for client applications. Most importantly, PushaaS does this as a service for the Tsuru open-source PaaS. So applications running on top of a Tsuru cluster can be bound to a Push Service instance and use it to implement server-push functionality in an easy and scalable fashion.


## Components

This documentation covers all the pieces that compose the PushaaS ecosystem. All projects in the ecosystem can be found at https://github.com/pushaas.

While reading this list of components, check the [Architecture](#architecture) for a visual representation.

- PushaaS
  - <span name="component-pushaas">[`pushaas`](https://github.com/pushaas/pushaas)</span>: implements an HTTP API that will be called by Tsuru clients in order to create instances of the Push Service and bind them to applications running on Tsuru. PushaaS relies on some infrastructure provider that will be called to provision the components of the Push Service. While the provider is configurable, currently the only implemented provider is Amazon Elastic Container Service (Amazon ECS). The images used to deploy each specific Push Service component can be customized.

- Push Service: is composed of 4 components that collaborate to implement the service. The components can be automatically deployed by the [`pushaas`](#component-pushaas) being called as a Tsuru service. If you don't use Tsuru, you can still use these 4 components to have an instance of a Push Service. You can deploy them manually, or using containers, or some other tool. [`push-service`](https://github.com/pushaas/push-service) contains a sample setup with Docker Compose that can be used as a reference on how to connect them together.
  - <span name="component-push-stream">[`push-stream`](https://github.com/pushaas/push-stream)</span>: is just an Nginx instance with the Push Stream module installed. The subscribers connect to `push-stream` instances to listen for messages, and messages published by the publishers are sent to them via the `push-stream`. The subscribing route should be open to users of your application, while the publishing route should only be open to the [`push-agent`](#component-push-agent). You can customize the Nginx configuration, the repository just provides the default one.
  - <span name="component-push-api">[`push-api`](https://github.com/pushaas/push-api)</span>: the `push-api` is the publishing interface for message publishers. Instead of calling directly the [`push-stream`](#component-push-stream) to publish content, the `push-api` provides a convenient wrapper with extra functionality, authentication, an admin application and stats about channels and messages.
  - <span name="component-push-redis">[`push-redis`](https://github.com/pushaas/push-redis)</span>: is just a Redis instance with persistence enabled (to store channels). It acts as a middleware between the [`push-api`](#component-push-api) and the [`push-stream`](#component-push-stream), which receives publishing from the [`push-agent`](#component-push-agent)
  - <span name="component-push-agent">[`push-agent`](https://github.com/pushaas/push-agent)</span>: the `push-agent` connects to the [`push-redis`](#component-push-redis) and listens for messages sent from the [`push-api`](#component-push-api). When a message is received, the `push-agent` publishes that message to publishing route of [`push-stream`](#component-push-stream). The `push-agent` should be deployed together with the `push-stream`, with a 1-1 relationship.

- Libraries
  - [`push-api-client-javascript`](https://github.com/pushaas/push-api-client-javascript): a JavaScript library to make calls to an instance of the [`push-api`](#component-push-api) in order to publish messages on channels. To be used by client applications.

- Demo applications
  - [`push-stream-demo-app`](https://github.com/pushaas/push-stream-demo-app): a client-side web application useful to test and demonstrate the features of the Nginx Push Stream module. Can be pointed either to a [`push-stream`](#component-push-stream) instance as well as to any Nginx instance with the module installed.
  - [`push-service-demo-app`](https://github.com/pushaas/push-service-demo-app): a full-stack web application ready to be run on Tsuru to test and demonstrate the Push Service. Requires an instance of the Push Service to be created and bound via [`pushaas`](#component-pushaas). Or can also be run pointing to any Server Push instance, like the one provided by [`push-service`](https://github.com/pushaas/push-service).

- Infrastructure configuration
  - [`pushaas-aws-ecs-config`](https://github.com/pushaas/pushaas-aws-ecs-config): contains a demo configuration of a full ecosystem on AWS containing: a [VPC](https://docs.amazonaws.cn/en_us/vpc/latest/userguide/what-is-amazon-vpc.html), a Tsuru cluster running on EC2 machines, an [ECS cluster](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html) running a [`pushaas`](#component-pushaas) instance, as well as subnets, security groups, service discovery and other configurations. The `pushaas` instance running on this configuration can be called to provision Push Service instances as [ECS services](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) on the ECS cluster, and bound to applications running on the Tsuru cluster.


## Architecture

TODO


## Current State and Limitations

- no scaling commands, though architecture supports it
- provisioners: ECS
