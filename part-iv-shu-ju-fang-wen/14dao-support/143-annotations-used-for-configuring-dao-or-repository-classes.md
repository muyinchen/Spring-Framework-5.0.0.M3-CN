## 14.3Annotations used for configuring DAO or Repository classes

The best way to guarantee that your Data Access Objects \(DAOs\) or repositories provide exception translation is to use the`@Repository`annotation. This annotation also allows the component scanning support to find and configure your DAOs and repositories without having to provide XML configuration entries for them.

```java
@Repository
public class SomeMovieFinder implements MovieFinder {
	// ...
}
```

Any DAO or repository implementation will need to access to a persistence resource, depending on the persistence technology used; for example, a JDBC-based repository will need access to a JDBC`DataSource`; a JPA-based repository will need access to an`EntityManager`. The easiest way to accomplish this is to have this resource dependency injected using one of the`@Autowired,`,`@Inject`,`@Resource`or`@PersistenceContext`annotations. Here is an example for a JPA repository:

```java
@Repository
public class JpaMovieFinder implements MovieFinder {

	@PersistenceContext
	private EntityManager entityManager;

	// ...

}
```

If you are using the classic Hibernate APIs than you can inject the SessionFactory:

```java
@Repository
public class HibernateMovieFinder implements MovieFinder {

	private SessionFactory sessionFactory;

	@Autowired
	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	// ...

}
```

Last example we will show here is for typical JDBC support. You would have the`DataSource`injected into an initialization method where you would create a`JdbcTemplate`and other data access support classes like`SimpleJdbcCall`etc using this`DataSource`.

```java
@Repository
public class JdbcMovieFinder implements MovieFinder {

	private JdbcTemplate jdbcTemplate;

	@Autowired
	public void init(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}

	// ...

}
```

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png "\[Note\]") |
| :--- |
| Please see the specific coverage of each persistence technology for details on how to configure the application context to take advantage of these annotations. |



