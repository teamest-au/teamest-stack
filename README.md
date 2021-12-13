# Teamest Stack

This repository aims to describe the architecture of the Teamest application as well as providing configuration files to run it locally or in production.

The following diagram shows a high level view of the interactions between each of the components in the Teamest stack.

![Diagram that shows the interaction between all components of the teamest stack.](docs/microservice_diagram.png?raw=true "Microservice Diagram")

## Component Types

### Rabbit Queue / Exchange

Teamest currently makes use of [RabbitMQ](https://www.rabbitmq.com/) for asynchronous communication between services. Each brown block in the diagram represents a RabbitMQ exchange. Green arrows represent publishing messages to the exchanges.

Rabbit is predominantly used in two ways:

1. **Eventing**: Each subscriber creates and maintains it's own Rabbit queue linked to the exchange, creating it on service start and destroying it on service finish. Each message is received by all active subscribers. Event subscriptions are represented by yellow arrows on the diagram.
2. **Jobs**: Each type of subscriber creates a single queue shared by all instances. This single queue is not destroyed. Each message is only processed once. Job subscriptions are represented by red arrows on the diagram.

### Workers

Worker components are represented by green blocks on the diagram. They are driven by one or more rabbit queues and perform non-time-critical and/or long-running tasks.

Workers are currently written in nodejs and use the [simple-rabbitmq](https://github.com/danielemery/simple-rabbitmq) package to consume and/or push messages.

### Internal APIs

Internal APIs are represented by purple blocks on the diagram. They expose an rpc interface defined using [dumb-node-rpc](https://github.com/danielemery/dumb-node-rpc) allowing workers and/or external apis to interact with them. Typically each of them have their own mysql database for persistence.

### External APIs / BFFs (Backend-for-frontends)

BFFs are represented by blue blocks on the diagram. They expose a rest and/or websocket interface to clients. They interact with internal APIs to service requests from clients and may also subscribe to rabbit eventing queues in order to notify clients over websockets.

BFFs are currently written in node and use [koa](https://koajs.com/) for the rest api surface.

### Clients

Clients are browser single page applications represented by pink blocks on the diagram. They make rest calls to external APIs and may also subscribe to external api websockets to receive real-time events.

Clients are currently written using [React](https://reactjs.org/) bootstrapped with [create-react-app](https://create-react-app.dev). [docker-cra](https://github.com/danielemery/docker-cra) is used to bundle the built client and serve the assets.

### External Services

External services are represented by yellow ovals on the diagram.

## Scraping

One of the core functionalities of Teamest is to regularly scrape various sources to keep match fixtures up to date.

1. [scraper-worker](https://github.com/teamest-au/scraper-worker) is run periodically to scrape sources and emit all scraped results to a rabbitmq exchange.
2. [season-data-worker](https://github.com/teamest-au/season-data-worker) listens to scraped seasons, saves them to the team season store, and if they have changed, emits change events to another rabbitmq exchange.

## Calendar Generation

Based on user configuration calendars will be generated and then updated whenever the scraper detects a change to fixtures affecting a user.

## Shared Types

All types used by multiple services are defined in the [models](https://github.com/teamest-au/models) repository and exposed via npm.

## Internal Services

Internal services provide http apis accessible only within the internal network. Each service is scaffolded with a contract repository which uses [dumb-node-rpc](https://github.com/danielemery/dumb-node-rpc) to generate a client and server npm package.

The server npm package is consumed by another repository which provides the implementation and the docker configuration details.

### Internal Season Service

The internal season service is responsible for managing season data, it provides internal endpoints to perform CRUD over season data.
