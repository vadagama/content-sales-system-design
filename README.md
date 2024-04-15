# Architecture of a solution for selling content

This document is available in [English](README.md) and [Russian](README_RU.md).

- [Architecture of a solution for selling content](#architecture-of-a-solution-for-selling-content)
  - [Source data](#source-data)
    - [Requirements](#requirements)
    - [Restrictions](#restrictions)
  - [Assumptions](#assumptions)
  - [Stakeholders and a list of their interests](#stakeholders-and-a-list-of-their-interests)
  - [User scenarios](#user-scenarios)
  - [Context Diagram](#context-diagram)
  - [Container diagram](#container-diagram)
    - [Description of container diagram elements](#description-of-container-diagram-elements)
  - [Key Quality Attributes](#key-quality-attributes)
  - [Deployment Diagram](#deployment-diagram)
    - [List of services used](#list-of-services-used)
  - [Security](#security)
    - [Authentication and authorisation](#authentication-and-authorisation)
    - [Cryptography](#cryptography)
    - [Self-testing](#self-testing)
    - [Private subnets](#private-subnets)
    - [Technical auditing](#technical-auditing)


## Source data

### Requirements

This section presents the business requirements that the customer has formulated:

- **REQ-1**: Need to design the architecture of an application that sells content
- **REQ-2**: Showcase: mobile application
- **REQ-3**: App Store, Google Play payment

### Restrictions

This section presents the constraints that the customer has formulated:

- **CON-1**: Number of registered users - 100,000
- **CON-2**: 100 TPS.

## Assumptions

The key assumptions that were identified in the study are described here:

- **ASM-1**: The infrastructure is hosted in the cloud (Amazon Web Services). This is actually not fundamental. The solution can also be deployed On Premise using Open Source (listed in the appendix).
- **ASM-2**: Assume we can leverage a development team (Java - Spring, Javascript - React, Swift, Kotlin) to develop it
- **ASM-3**: Assume that if we have a load of 100 TPS, then the number of catalogue read requests can be 500 RPS (five times more)
- **ASM-4**: Assume that the content catalogue is not very large (1000 items)
- **ASM-5**: Assume an average content size of 10MB.
- **ASM-6**: Assume that after the purchase the content is available to the customer for a certain period of time. After this period, the content is blocked.
- **ASM-7**: Assume that the content is created by employees of our service
- **ASM-8**: Assume that the availability level should be at least 99.95

## Stakeholders and a list of their interests

This section describes the stakeholder groups that can influence the design of the solution and a list of their interests that should be considered when designing the solution

- **SH-1: Customer** - our customer who purchases products
  - User interface simplicity and ease of use
  - No errors in catalogue navigation and content acquisition
  - Application speed
- **SH-2: Administrator** - the employee of the service, who provides processing of orders and content catalogue actualisation
  - Simplicity of the user interface and convenience of work
  - Absence of errors when administering the content catalogue
  - Speed of work of web-application
  - Minimal number of problematic requests from clients
- **SH-3: Content producer** - employee of the service, who is responsible for creating and updating quality content.
  - Convenient user interface for adding content
- **SH-4: Product owner** - service employee who forms the development team, monitors the main metrics of the service and determines priorities
  - Availability of tools for monitoring business metrics
  - Service stability
  - Ability to withstand peak loads (for example, in case of activation of sales)
  - Ability to scale the solution (planned increase in load, number of users, volume of stored information).
- **SH-5: Developer** - an employee of the company who develops the functionality of the service
  - Modern stack
  - Easy to develop and deploy features
  - Easy debugging
- **SH-6: DevSecOps** - an employee of the company, who ensures trouble-free and secure operation of the service.
  - Clear means of monitoring logs and technical metrics
  - Mechanisms for prompt notification of failures and other problems
  - Simple mechanisms for automatic testing and deployment of new features

## User scenarios

Here we describe the main user scenarios that were identified during the study:

- Creating content and populating the catalogue with it
  - **US-1.1**: Content producer logs into Product Management Backoffice
  - **US-1.2**: Content producer uploads new content as a ZIP archive and adds accompanying meta-information (content title, publication date, author, etc.)
- Client acquisition of content
  - **US-2.1**: Customer goes to Google Play or Apple App Store and downloads our mobile app
  - **US-2.2**: Customer installs the app on the device
  - **US-2.3**: Customer signs in to the app using SSO (Google ID or Apple ID account) or manually.
  - **US-2.4**: Customer using search function or content catalogue navigation finds the desired "product".
  - **US-2.5**: Customer opens the product card.
  - **US-2.6**: Customer clicks the "Buy" button.An assistant opens to make the purchase:
  - **US-2.7**: Customer fills in the form for entering personal data (name, e-mail, country, city, etc).
  - **US-2.8**: Customer fills in the form for payment details (Visa, Mastercard, Paypal, etc)
  - **US-2.9**: Customer confirms the entered data and presses the "Pay" button
  - **US-2.10**: Customer receives a receipt for the content payment at the e-mail address he/she specified.
  - **US-2.11**: Once the transaction is processed, the purchased content becomes available to the Customer-user in "My Purchases" for a specified period of time.
  - **US-2.12**: After the time period expires, the purchased content becomes unavailable for viewing.
- Content Admin:
  - **US-3.1**: Administrator logs into Product Management Backoffice
  - **US-3.2**: Administrator can add new content
  - **US-3.3**: Administrator can change the meta information of existing content
  - **US-3.4**: Administrator can remove existing content from the catalogue


## Context Diagram

Below is a simplified diagram of the solution at the business context level. All users interact only with Google or Apple services and use the mail service. Our service is integrated with Google API and Apple API for application deployment, inApp payments implementation and other functions.

![Context diagram!](images/context.png 'Context diagram')

## Container diagram

Below is a diagram of the solution at the level of individual containers (based on the C4Model concept). This diagram shows what elements at the middle level the solution to be designed should consist of and how these elements will interact with each other.

![Container diagram!](images/container.png 'Container diagram')

### Description of container diagram elements

- **Mobile Application iOS** - Apple iOS mobile application available for download from Apple Store
- **Mobile Application Android** - mobile application on the Android platform, available for download from Google Play
- **Google Play API** - API for interacting with Google Play services. The Google Pay API includes.
- **Apple Store API** - API for interacting with Apple Store services. Including Apple Pay.
- **DNS** - domain name service
- **CDN** - Distributed content delivery and distribution service.
- **WAF** - service for detecting and blocking network attacks on the service
- **API Gateway** - gateway providing interaction between client applications and our services (APIs)
- **Auth Service** - service for secure user authentication and authorisation
- **Products API** - software interface of the application providing work with content catalogue
- **Product and Order Management Backoffice** - web-interface providing work with content catalogue, adding new content and updating of already uploaded content, as well as managing customer orders and purchases.
- **S3 Storage** - object storage for static content and historical data
- **Products Database** - Database for storing information about products
- **Orders Database** - database for storing information about orders
- **Products Cache** - intermediate in-memory storage of data on products
- **Orders API** - programme interface of the application providing work with orders
- **Message Broker** - message broker providing asynchronous processing of various events (e.g. sending Push notifications to customers or payment receipts)
- **Reporting** - system for building analytical reports based on data from the database
- **CI/CD** - here it will be possible to store development artefacts, implement code repository, implement CI/CD pipelines
- **Log and tracing** - collection, processing and monitoring of logs

## Key Quality Attributes

Since the architecture to be designed must rely on certain business requirements, constraints and assumptions the key quality attributes are described below.

**Products Service**.

- **QA-1**: Perfomance
  - The service must be able to withstand at least 500RPS (under the assumptions we defined earlier)
- **QA-2**: Availability
  - The service must be available 24/7. Availability Score 99.95.
- **QA-3**: Fault tolerance
  - The service must be able to quickly switch to a working instance in case one of the instances crashes.
  - The service must be able to prevent failures using geographically distributed data centres.

**Orders Service**

- **QA-4**: Security
  - The security of data, payment and order information must be ensured.
  - All information about customer orders and payments must be available for a long period of time and protected from attacks.
- **QA-5**: Reliability
  - The ordering service must be load-resistant (including artificially generated load) and be able to withstand at least 100TPS.
- **QA-6**: Fault tolerance
  - The service must be able to quickly switch to a working instance in case one of the instances crashes.
  - The service must be able to prevent failures using geographically distributed data centres.

**Mobile Applications**

- **QA-7**: Usability
  - Mobile applications should be designed so that the user does not have to get used to the user interface or undergo any training
- **QA-8**: Modifiability
  - Applications should be written using loosely coupled software components to allow for rapid addition of new functionality and modification of existing functionality.
- **QA-9**: Availability
  - Applications should have a high level of availability. For us, this means high quality mobile app code and high availability of Order Service and Product Service

**Backoffice**

- **QA-10**: Usability
  - The web application should be intuitive to the user

## Deployment Diagram

Below is a deployment diagram for publishing a solution to the cloud. This diagram shows how the logical elements of the solution (see Container diagram) land on the PaaS and SaaS services of cloud infrastructure vendor Amazon.

![Deployment diagram!](images/deployment.png 'Deployment diagram')

### List of services used

The following list of infrastructure services is necessary to implement the projected solution in Amazon Cloud.

<table>
  <tr>
   <td><strong>Service name</strong>
   </td>
   <td><strong>Vendor</strong>
   </td>
   <td><strong>Description</strong>
   </td>
   <td><strong>On-Premise Alternative</strong>
   </td>
  </tr>
  <tr>
   <td>WAF
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/network-firewall">AWS Network Firewall</a> is a managed service that provides functionality to protect your network from attacks.
   </td>
    <td><a href="https://shadowd.zecure.org/overview/introduction/">Shadow Daemon</a> - это набор инструментов для обнаружения, регистрации и блокирования атак на веб-приложения.
   </td>
</tr>
  <tr>
   <td>EKS
   </td>
   <td>AWS
   </td>
  <td><a href="https://aws.amazon.com/eks">Amazon EKS</a> is a managed Kubernetes service
   </td>
   <td><a href="https://github.com/kubernetes/kubernetes">Kubernetes</a> is an open-source system for automating deployment, scaling, and management of containerised applications
   </td>
  </tr>
<tr>
   <td>Cognito
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/cognito/">Amazon S3</a> provides a simple secure mechanism for user authentication and authorization (Sign-Up, Sign-In, and Access Control)
   </td>
   <td><a href="https://www.keycloak.org/">Keycloak</a> is an Open Source Identity and Access Management
   </td>
  </tr>
  <tr>
   <td>S3
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/s3">Amazon S3</a> object storage for long-term storage of information and documents
   </td>
      <td><a href="https://ceph.io/en/discover/technology/">Ceph</a> is an open-source, distributed storage system
   </td>
  </tr>
  <tr>
   <td>API Gateway
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/api-gateway">Amazon Api Gateway</a> is designed to manage APIs
   </td>
      <td><a href="https://konghq.com/kong/">Kong</a> the world’s most popular API gateway. Built for microservices and distributed architectures
   </td>
  </tr>
<tr>
   <td>CloudFront
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/cloudfront">Amazon CloudFront</a> a means of distributed delivery and distribution of static content
   </td>
   <td><a href="https://www.jsdelivr.com/">jsDelivr</a> a tool for distributed delivery and distribution of JavaScript content
   </td>
  </tr>
  <tr>
   <td>SNS
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/sns">Amazon SNS</a> managed message broker for pub/sub messaging, SMS, email and push notifications
   </td>
   <td><a href="https://www.rabbitmq.com/">RabbitMQ</a> Additional implementation of functionality for sending emails, SMS, etc. will also be required.
   </td>
  </tr>
  <tr>
   <td>RDS
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/rds/">Amazon RDS</a> is an easily scalable relational database management service
   </td>
   <td><a href="https://www.postgresql.org/">PostgreSQL</a> the World's Most Advanced Open Source Relational Database
   </td>
  </tr>
 <tr>
   <td>ElastiCache for Redis
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/elasticache/redis">Amazon ElastiCache for Redis</a> - Redis in-memory storage managed service
   </td>
   <td><a href="https://redis.io/">Redis </a> is an open source (BSD licensed), in-memory data structure store
   </td>
  </tr>
  <tr>
   <td>DataDog
   </td>
   <td>DataDog
   </td>
   <td><a href="https://www.datadoghq.com/">Datadog</a> SaaS service for monitoring, logging and tracing
   </td>
   <td><a href="https://www.elastic.co/elastic-stack/">ELK Stack</a> Elastic Search, LogStash, Kibana
   </td>
  </tr>
  <tr>
   <td>Tableau
   </td>
   <td>AWS
   </td>
      <td><a href="https://www.tableau.com/">Tableau</a> Analytical report building service
   </td>
   <td><a href="https://www.metabase.com/start/oss/">Metabase </a> is an open-source Business Intelligence(BI) tool
   </td>
  </tr>
  <tr>
   <td>Amazon Managed Streaming for Apache Kafka
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/msk/">Amazon Managed Streaming for Apache Kafka</a> Управляемый сервис Kafka
   </td>
   <td><a href="https://www.metabase.com/start/oss/">Apache Kafka </a> is an open-source distributed event streaming platform
   </td>
  </tr>
</table>

## Security

In order to minimise the risks of leaks of sensitive information about users, their payment data and purchases, as well as to protect the system from various failures during attacks on the service, a number of measures are proposed below.

### Authentication and authorisation

Amazon offers AWS Cognito to implement secure user authentication and authorisation functions (Authentication Service component). This service will be able to be used to authorise both customers and internal users using OpenID Connect, OAuth 2.0 or SAML 2.0. It also helps protect user data from being compromised in hacker attacks.

### Cryptography

Our data must be covered by encryption using 256-bit Secure Socket Layer (SSL) for both mobile client applications and web applications in the back-end.

### Self-testing

You need to ensure regular test runs: port scanning, testing for SQL injection. Netflix (https://github.com/Netflix/SimianArmy) can be used for chaotic testing.

### Private subnets

Private subnets should be used to protect against direct access to data and services from the Internet.

### Technical auditing

Regular external audits of the system should be performed to provide an independent assessment of the security of the system and stored data.
