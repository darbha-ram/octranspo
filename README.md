# octrans
Spring Boot Application to track OC Transpo Bus Cancellations
(c) 2019- Ram Darbha. All Rights Reserved.

OC Transpo is the public transit system of Ottawa, Canada.

This application uses JDBC to interface to an Azure SQL DB containing data about cancelled OC Transpo buses.  (The Azure DB is populated in realtime by another Azure application that is triggered by tweets from the OC Transpo account, outside the scope of this project.)  This application presents a REST API in the frontend to allow a user to determine cancellations of specific bus routes,  the most or least cancelled buses.  Each API call can be restricted to AM or PM or all day, and to a specific time interval (fenced by start date and end date).  Both start and end date parameters are optional.  If only start date is given, cancellation on or after that date are returned.  If only end date is given, cancellation up to and including that date are returned.  If both dates are given, cancellations in that time interval are returned.  If they are both unspecified, query is executed over all the DB contents from time of inception (August 2018) to the latest available data.  Note, start and end date must be in the format "yyyy-mm-dd", with leading zeroes as needed, e.g. `2018-2-17` is invalid.

## REST API

Below are examples of API invocations and corresponding responses.  Note the API only supports HTTP GET, that is PUT, POST, DELETE are not supported.

Base URL: `/ocapi/v1/`

### Specified Bus

To find all cancellations of bus route 265:
 
`ocapi/v1/bus/265/all`

To find morning cancellations of bus route 265:
 
`ocapi/v1/bus/265/am`

To find evening cancellations of bus route 265:
 
`ocapi/v1/bus/265/pm`

To find all evening cancellations between 1-Nov-18 and 31-Dec-18:
 
`ocapi/v1/bus/265/pm?sd=2018-01-21&ed=2018-06-28`

For all above invocations, the response body is a JSON array, each element of which has information fields about a specific cancellation of the requested route: bus number, bus name, startdate, start location and delay to the next bus.

    [
      {
        "busnumber": 256,
        "busname": "Bridlewood",
        "starttime": "2018-10-09T16:36:00",
        "startloc": "Mackenzie King bridge",
        "nextminutes": 9
      },
      {
        "busnumber": 256,
        "busname": "Bridlewood",
        "starttime": "2018-10-01T15:35:00",
        "startloc": "Mackenzie King bridge",
        "nextminutes": 15
      }
    ]

### Most Cancellations

To find 8 routes with the most cancellations:
 
`ocapi/v1/most/8/all`

To find 6 routes with the most morning cancellations:
 
`ocapi/v1/most/6/am`

To find 10 routes with the most afternoon cancellations until 31-Dec-18:
 
`ocapi/v1/most/10/pm?ed=2018-12-31`


### Least Cancellations

To find 5 routes with the least cancellations:
 
`ocapi/v1/least/5/all`

To find 3 routes with the least morning cancellations starting 1-Nov-18:
 
`ocapi/v1/least/3/am?sd=2018-11-01`



## Dev/Build Environment

- Spring Tool Suite 3.9.7
- Gradle 5.1.1
- JDK 1.8

## To Build and Deploy

This project is setup to build with gradle.

    $ git clone 
    $ cd octranspo
    $ ./gradlew build
    $ java -jar build/libs/ramd-octrans-0.2.0.jar

After service is started, issue HTTP request to port 8080, e.g. `http://localhost:8080/ocapi/v1/bus/95/all`.

## Troubleshooting

### Eclipse dependencies not found
If Eclipse failing to find dependencies even though `build.gradle` is up to date, it is likely because Eclipse `.classpath` is out of date.  To update Eclipse project:

    $ ./gradlew cleanEclipse
    $ ./gradlew eclipse

### HTTP Requests returning 404
If a class has been annotated `@RestController` but HTTP Requests still fail, it is likely because that class isn't being found by Spring.  Recommended Spring Boot approach is to have the top-level application (`@SpringBootApplication`) class in a non-default package (e.g. `app`), the controller and other classes in sub-packages (e.g. `app.controller`, `app.model` etc.).  If that's not feasible, alternately, the top-level class can be explicitly annotated by pointing it to class(es) to scan:

    @SpringBootApplication
    @ComponentScan(basePackageClasses = ApiV1Controller.class)
    public class Application {
