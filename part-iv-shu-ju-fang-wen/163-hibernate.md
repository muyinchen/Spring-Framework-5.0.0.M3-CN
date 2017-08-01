## 16.3 Hibernate

We will start with a coverage of[Hibernate 5](http://www.hibernate.org/)in a Spring environment, using it to demonstrate the approach that Spring takes towards integrating O/R mappers. This section will cover many issues in detail and show different variations of DAO implementations and transaction demarcation. Most of these patterns can be directly translated to all other supported ORM tools. The following sections in this chapter will then cover the other ORM technologies, showing briefer examples there.

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| As of Spring 5.0, Spring requires Hibernate ORM 4.3 or later for JPA support and even Hibernate ORM 5.0+ for programming against the native Hibernate Session API. Note that the Hibernate team does not maintain any versions prior to 5.0 anymore and is likely to focus on 5.2+ exclusively soon. |



