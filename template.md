- I have 7+ years of experience building scalable distributed backend applications in Fintech.

- I have built large-scale web applications used by millions of people. I have used SQL and NoSQL DBs, Load Balancers like AWS LB (& API Gateway) & Kong, and distributed message brokers like SQS and Apache Kafka, caching systems like Redis, and monitoring systems like OpenTelemetry, Sentry, Datadog, Prometheus with Grafana.

- I have a clear understanding of lower-level network protocols like TCP, UDP and FTP. And higher-level protocols like HTTP 0.9, 1.0, 1.1, 2 and most recently HTTP/3 (QUIC) and WebSockets.

- Have used DBs like Postgres and MySQL (SQL DBs) for OLTP applications and DynamoDB / MongoDB (NoSQL) for OLAP applications. 

- I have a basic understanding of stacks, queues, trees (especially B-trees, used in DB indexes). I have a basic understanding of algorithms like Binary Search, Sorting,  Path Finding.

- Currently, I am in the process of migrating a monolith application in PHP/Laravel to Go/Gin microservices. 

- I have helped reduce bugs by 30% (at my current org) by tracing with Jaeger,  Sentry, and Datadog. 

- I have a bachelor's degree in Instrumentation Technology Engineering and 7+ years of experience working in Fintech.

I believe I will be a good fit for this role.


Recently I build a framework for Product Activation, called (Product Activation Request). Which is like a glorified feature flag system.
This is not just a project, its a framework to allow developers to activate certain product as part of the system along with some requirements.

like I want user to be able access the `Yield Account` product if they have uploaded `selfie, proof of business and website` and once they are verified they can further access "activate" that "Product" (Hence Product Activation Request).

In this I have also integrated other existing features like `maker-checker check`. i.e once user submits the required information they can further checked twice. (once by maker, later by checker) and then approved. also `maker/checker` and ask for more information about the business until they are setisfied.
And thn they are approved. And only after their approval they can be further `activated` for that product. (this behaviour is inherited from exsiting systems (at the organiation for KYB/KYC using state-machine)). And this is all configurable.

And this also exposes an generic API and Kafka Events. Which can be used and added more custom business logic bu whoever is using it.

Currently, we have release around 4-5 features using this.
- Multi Currency Accounts (owned by me, has reduced onboarding costs from $225 to $175 per account)
- Progressive onboarding (owned by me, has increased onboarding new businesses from 50 per day to 75 per day)
- Pre-EIN number onboarding (owned by me, currently being rolled out for busineses in US)
- Yeild Accounts (owned by other teams)
- Stable coins accounts (owned by other teams)


