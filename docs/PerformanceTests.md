# Performance Tests in YaaS

## Introduction

This document describes the standard performance test process used by the teams for testing the performance of services. In the context of performance tests are defined as follows.

      DEFINITION: Finding out the performance characteristics of a specific service during well defined load scenarios


The fact that performance tests results need to be comparable across all teams makes a standardized performance test approach inevitable. As performance tests are a complex topic teams and the platform overall will benefit from the standardization.

Explicit performance tests as described in this document will not be required anymore once the final continuous deployment/ monitoring pipeline is in place across all teams. The auomated deploymend process will take care that a fraction of real user traffic will be routed to a new service. Only if its performance and other metrics are at least comparable or improved the new service will be kept in production.

This means that performacne testing will become part of the infrastructure and teams will not be required to cover this topic explicitely anymore in future. Until the final deployment pipeline is in place the process described in this document is valid.


## Performance tests


### Test tool 

[Gatling](http://gatling.io/ "Gatling") will be the performance test tool of choice for all teams. This decision is based upon the fact that various teams already gained a considerable amount of knowledge with the tooling.

		HINT: In case a team lacks the knowledge to implement custom tests with Gatling please approach Marek Nawa. He will collect the information and will coordinate knwoledge exchange sessions between teams


### Test Environment

All tests need to be run ***from virtual machine hosted in Monsoon***. The machine is exclusively used for load generation purposes. All tests will be run ***against Stage environment***.

Any tests need to simulate real users' behaviour and therefore need to ***access services via Apigee***. Performance test can be ***run from a given Jenkins CI UI***. Concurrent requests for ***performance tests*** executions ***will only be run sequentially***.

		HINT: In case a team plans to execute performance tests at a given date/ time it needs to align with Marek Nawa  


### Test-case setup

Initially each team needs to select its ***top 5 performance relevant REST endpoints*** and focus its test only on those. Selection criteria may be:

* expected most frequent reads
* expected most frequent writes
* expected business criticality
* ...


### Test strategy

All teams' performance tests need to implement the same load curve and need to apply load for the same amount of time. 

![Load Curve](./images/concurrentusers.png =600x "Load Curve")

A single test run takes 30 minutes. In total >400.000 requests will be sent to the target all simulated users are ramped up.

![Requests per seconds](./images/cummulativerequestsuntilrampupend.png =600x "Requests per seconds")

During the peak load time each of the 900 users will execute 60 requests per minute. All in all this results in 900 requests per second. In case teams decide to implement multiple http calls per scenario *900 * <amount of http>* calls requests will be sent.

![Requests per seconds](./images/rps.png =600x "Requests per seconds")


### Test implementation

Each team is required to re-use the given tests assets:

* BaseSimulation.scala: implement the basic load scenario and must not be changed by teams
* Oauth2Authentication.scala: provides oauth2 authentication mechnanisms and can be used by teams

When a team decides to implement a service load test it:

* Forks this git project (https://github.com/MichaelStephan/defaultsimulation)
* Adds a single class which inherits from BaseSimulation.scala
	
		package defaultsimulation
		
		import io.gatling.core.Predef._
		import io.gatling.http.Predef._
		
		class SchemaRepositoryService extends BaseSimulation {  
		  def getScenarios : List[Scenario] = {
		    List(
		      Scenario(
		        // BEFORE TEST
		        exec { ... },
		       
		        // TEST
		        exec {  ... },
		     
		        // AFTER TEST
		        exec { ... }
		      )
		    )
		  }
		}
		
* As a next step each scenarios test phases need to be implement. Each scenario consists of the following phases:

	* before test: actions required for setup (e.g.: creating data, login via oauth, ...)
	* test: the actual test to be executed (e.g.: fetching data). A test block will be executed endlessly. Between each execution there is a 1 second break
	* after test: actions required for cleanup (e.g. deleting data, logout, ...)
	
It may happen that a scenario does not require a before/ after step, therefore the NOP function can be used (see sample code in git repository). The sample code also shows how to do oauth2 based authentication and how to use the bearer token when doing cass in the actual test phase.

Once all 5 scenarios are implemented a team can test the script directly from commandline which required various environment variabled to be available:

	export BASE_URL (e.g. https://www.facebook.com)
	export OAUTH2AUTHENTICATION_TOKEN_URL (e.g.: https://www.facebook.com/dialog/oauth)
	export TENANT (e.g.: tuorhujoksap)
	export CLIENT_ID (e.g.: ZYePI46H2M48CwBCwAQYbvSRu3zaHexW)
	export CLIENT_SECRET (e.g.: zl4sf6t5p7zkfv4a)


Finally the test can be run with:

	PATH_TO_GATLING_BIN_FOLDER/gatling.sh -sf PATH_TO_IMPLEMENTATION_FOLDER -s NAME_OF_IMPLEMENTATION_CLASS


### Test execution

Currently infrastructure is providing a centralized Jenkins CI installation which tests can be registered to and finally run from.







