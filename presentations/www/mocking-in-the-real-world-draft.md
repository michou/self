# Mocking the real world for fun and profit - scratchpad

- introduction
    - Mihai Balan, Quality Engineer in Test (SET) @ Adobe Identity Services
    - tonight I will tell you a couple of stories about mocking; hope to hear your stories about that afterwards as well (beer and pizza on the house)
    - first of all, a short show of hands, how many of you **never** used mocking when testing a product / service / whatever
        - Wikipedia's definition of [mocking](https://en.wikipedia.org/wiki/Mock_object):
            > In **object-oriented programming**, mock objects are simulated objects that mimic the behavior of real objects in controlled ways. A programmer typically creates a mock object to test the behavior of some other object, in much the same way that a car designer uses a crash test dummy to simulate the dynamic behavior of a human in vehicle impacts.
        - that definition is rather restrictive, but if you think about it, we use "almost-but-not-quite-real" objects all the time when testing -- be it "mock test data" that barely passes our validations, or dummy credit cards, test files, etc.
    - hoping I convinced you that _everybody does it_, I'm going to tell you three stories of mocking:
        1. the textbook mocking
        2. the "When I'm alone at night, I Google myself" mocking
        3. the "Nu răspunzi la SMS" mocking
- textbook mocking
    + the "textbook definition" for mocking singles out OOP since polimorphism usually makes it very easy to swap components that share an interface
    + in our case, we were testing a brand new micro service built with Java (OOP all the things) and Spring (DI all the things)
        * DI is even nicer (or not, depending on who you're asking) as it allows further decoupling between resource usage and creation
        * this means that where a developer sees
        ```java
@AutoWired
private CacheService redisCache;
        ```
        I see
        ```java
@AutoWired
private CacheService redisCache; // definitely **NOT** a Redis cache
        ```
    + going back to our story, brand new micro-service + Spring = some <3 for Integration in the Small tests
        * we wanted to test across the whole stack, but without going through the Internet
        * there are some tools built on top of Spring that enable this kind of testing, `mockMVC` being one of them
        * so, how did we do it?
            - mock all the things, then mock some more
                + mocked all the stuff that we didn't care about via `mockito`
                + built very shallow mocks for the stuff we cared about (Redis, MySQL, etc.)
        * was it worth it?
            - we caught some timing-sensitive bugs that we would've had a hard time catching otherwise
            - we were able to have a viable test suite run a build time
- "Nu raspunzi la SMS" mocking
    + another cool feature we worked on allows users to use SMS codes to reset their account password
        * this is usually done by your service using another service to send an SMS with a known payload (e.g. verification code)
    + for feature developement all worked fine and dandy, but we wanted to make sure the changes we made to our system didn't negatively impact the performance of the system as a whole
        * we wanted to get accurate performance data, so NOP-ing the code in the server-side was not an option
        * a load test was born
            - but unlike another load test that sent emails, we would've liked better if the bill wouldn't cover a small family car
            - we wrote a mock service, that exposed the same APIs and only responded with success, very quickly
            - packed it up, put it up on AWS, and started the load test
                + after a certain point, we noticed the throughput stopped increasing, although we hadn't yet reach the performance goals
                + on the server side all looked good - no timeouts, no CPU or memory stress
                + turns out, the mock itself was actually topping up
                    * we spun up some more machines, hid all of them behind a LB and restarted the test
                + in the new clustermock configuration, all went well
    + was it worth it?
        * definitely, a lot of $$$ worth-it
        * good opportunity to practice our Ops-fu
        * paved the way for other glorious mocks
- "When I'm alone at night, I Google myself" mocking
    - another feature that we worked on consisted in allowing users to log in using their Google accounts; we already implemented support for facebook
    - the problem was, depending on whether a certain social account was never before used or not, different code paths were taken inside our product
    - testing for facebook was rather simple - facebook gives you an API to create test users scoped to a certain application
        + Google, not so much
        + also, Google is rather anal both when it comes to creating a bunch of accounts (unless you also have a bunch of SIM cards laying around) and when it comes to logging in from all parts of the world (or just AWS' datacenters, I'm not 100% sure)
    - also, on the other hand, facebook was notoriously slow (even on API calls) and fickle with it UIs (and sometimes API responses, too)
    - and thus, we mocked (using python and Elastic BeanStalk)
    - we created the bare minimum web-app (and services) that would allow us to mimic Google convincingly
        + implemented the UI, with a lot of reverse engineering
        + for the APIs it was a lot easier, as they (1) have it documented and (2) it more or less respects the OAuth2 standard
        + product wise, the changes were minimal
            * we had to be able to specify the enpoints somehow, anyway
            * most of the overhead came from being able to switch between real & mocked on the fly
    - was it worth it?
        + we were able to test a lot easier some workflows that would've required jumping through a lot of hoops
        + for common flows, test run times dropped to about 50% (facebook vs. Google mock)
        + also, it was fun :)


- conclusions
    - mocking can be useful in non-standard situations as well
        - however when you can, use libraries and don't reinvent the wheel (Mockito rocks, btw.)
    - often times, effectively mocking requires some collaboration from the development team as well
        - but more often than not, the results outweight the time investment
        - e.g. in the Google / SMS mock cases it could've been done by simply spoofing the DNS on the server machines, but by tradind off some added flexibility (e.g. running mocked & non-mocked side-by-side)
    * think about improving testability as a whole