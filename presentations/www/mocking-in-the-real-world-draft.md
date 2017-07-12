# Mocking the real world for fun and profit - scratchpad

- introduction
    - Mihai Balan, Quality Engineer in Test (SET) @ Adobe Identity Services
    - tonight I will tell you a couple of stories about mocking; hope to hear your stories about that afterwards as well (beer and pizza on the house)
    - first of all, a short show of hands, how many of you **never** used mocking when testing a product / service / whatever
        - Wikipedia's definition of [mocking](https://en.wikipedia.org/wiki/Mock_object):
            > In **object-oriented programming**, mock objects are simulated objects that mimic the behavior of real objects in controlled ways. A programmer typically creates a mock object to test the behavior of some other object, in much the same way that a car designer uses a crash test dummy to simulate the dynamic behavior of a human in vehicle impacts

        - that definition is rather restrictive, but if you think about it, we use "almost-but-not-quite-real" objects all the time when testing -- be it "mock test data" that barely passes our validations, or dummy credit cards, test files, etc.
    - hoping I convinced you that _everybody does it_, I'm going to tell you three stories of mocking:
        1. the textbook mocking
        2. the "Google all you want" mocking
        3. the "Nu răspunzi la SMS" mocking
- textbook mocking
    - the dictionary definition of mocking singles out OOP since in OOP is usually the easiest to swap an object (or interface) with some other object as long as the new one implements the same interface, either through static or duck typing
        - where developers use polymorphism to create wonderful things, testers can use it for their own nefarious purposes 
    - in our case, one of the components we worked on, was a new (micro) service that, among others, used a database and a cache to store data
    - we were planning to write integration tests, but, since this was a rather isolated part  of the system and pretty much a clean slate, decided to try something new, shiny and fast
        - setup was Java & Spring running (in production) on AWS
        - all hail _Integration in the small_ tests (brief explanation about how IitS tests are above unit tests but below "real" integration tests)
        - in our case, that meant that we would have the framework build all the needed classes and scaffolding and Spring magic for us and we would mock everything else, as much as we could.
            - one of the things that we wanted to "properly mock" was the cache
                - what in reality was actually a REDIS cluster, we replaced with a `ConcurrentHashMap`
                - since it implemented the same interface we were able to very easily bootstrap our micro-service and have it use the Map-based implementation fo the cache
                - for extra easiness, we used MockMVC to wholly bypass the network stack and call Java methods directly
                - this allowed us to cover more cases and catch some timing bugs that would have been difficult to catch otherwise
- "Google all you want" mocking
    -asdf
- "Nu raspunzi la SMS" mocking
    - asdf

- conclusions
    - mocking can be useful in non-standard situations as well
        - however when you can, use libraries and don't reinvent the wheel (Mockito rocks, btw.)
    - often times, effectively mocking requires some collaboration from the development team as well
        - but sometimes, the results outweight the time investment