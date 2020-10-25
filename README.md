# Teamest Stack

This repository aims to describe the architecture of the Teamest application as well as providing configuration files to run it locally or in production.

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
