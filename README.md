# DDD Strategic Design with Spring Boot
Application to demonstrate Domain Driven Design Context Mapping patterns based on various Spring Boot applications.
Please bear in mind that the application itself is kept simplistic, in order to isolate the focus on the Context Mapping
Patterns. This is the reason why you will find some logic in Controllers that should be placed in other classes in a real-world
application or the reason why I use database IDs for a general purpose.

Also keep in mind that some of the Context Mapping Patterns _not_ best practices but are things that are found in existing, historically grown applications. Context Mapping is mostly a way to look at _existing_ solutions and this code example is an existing solution that has some intentionally built in issues.

A good starting point for your analysis is the CreditApplicationController in credit-application as it implements the
main workflow.


## Prerequisites
- You need Java 1.8 to build and run the applications
- Maven v3.5.0 (or just use the included `mvnw` or `mvnw.cmd` Maven wrapper)
- A basic installation of Redis must be installed and running (redis-server)
- *Optional.* [Cloud Foundry CLI](https://github.com/cloudfoundry/cli/releases) if you want to run the applications in Cloud Foundry

## How to run and install the example

There is no "one-stop" build and install script as of yet so you will have to take a few easy manual steps that you should
*run in this specific order*:

### Create the backing services

1. Create the redis-server

        cf create-service p-redis shared-vm redis-pubsub

2. Create the customer MySQL database

        cf create-service p-mysql 512mb mysql-customer

3. Create the customer-contact MySQL database

        cf create-service p-mysql 512mb mysql-customer-contact

4. Create the mysql-credit-application MySQL database

        cf create-service p-mysql 512mb mysql-credit-application

### Build and deploy the applications

1. Build the scoring-shared-kernel module

        ./mvnw install --projects scoring-shared-kernel

2. Build the customer application

        ./mvnw package --projects customer

3. Run the customer application

        cf push -f customer/manifest.yml

4. Build the scoring application

        ./mvnw package --projects scoring

5. Run the scoring application

        cf push -f scoring/manifest.yml --no-start
        cf set-env mploed-scoring creditAgencyServer https://mploed-credit-agency.local.pcfdev.io/
        cf start mploed-scoring

6. Build the credit-agency application

        ./mvnw package --projects credit-agency

7. Build the credit-agency application

        cf push -f credit-agency/manifest.yml

8. Build the customer-contact application

        ./mvnw package --projects customer-contact

9. Run the customer-contact application

        cf push -f customer-contact/manifest.yml

10. Build the credit-application while the customer application is running (this executes jaxb2:generate against the customers.wsdl)

        ./mvnw package --projects credit-application -Dcustomer.wsdl.endpoint=http://mploed-customer.local.pcfdev.io/ws/customers.wsdl

11. Run the credit-application application

        cf push -f credit-application/manifest.yml --no-start
        cf set-env mploed-credit-application scoringServer https://mploed-scoring.local.pcfdev.io/
        cf set-env mploed-credit-application customerServer https://mploed-customer.local.pcfdev.io/
        cf start mploed-credit-application

12. Start entering data at https://mploed-credit-application.local.pcfdev.io

## URLs and Ports
Each of the modules is it's own Spring Boot Application which can be accessed as follows:

<table>
    <tr>
        <th>Name</th>
        <th>Application / Enpoint Type</th>
        <th>Port</th>
        <th>URL</th>
    </tr>
    <tr>
        <td>Credit Application</td>
        <td>Web App</td>
        <td>9090</td>
        <td>http://localhost:9090/</td>
    </tr>
    <tr>
        <td>Customer</td>
        <td>WSDL Endpoint</td>
        <td>9091</td>
        <td>http://localhost:9091/ws/ or http://localhost:9091/ws/customers.wsdl for the wsdl</td>
    </tr>
    <tr>
        <td>Credit Agency</td>
        <td>REST Endpoint</td>
        <td>9092</td>
        <td>http://localhost:9092/personRating</td>
    </tr>
    <tr>
        <td>Scoring</td>
        <td>RMI Endpoint</td>
        <td>1199</td>
        <td>http://localhost:1199/scoringService</td>
    </tr>
    <tr>
        <td>Customer Contact</td>
        <td>No active server endpoint, listens to Redis on the following topics: customer-created-events, credit-application-approved-events</td>
        <td>No open port</td>
        <td>No available URL for access</td>
    </tr>
</table>
