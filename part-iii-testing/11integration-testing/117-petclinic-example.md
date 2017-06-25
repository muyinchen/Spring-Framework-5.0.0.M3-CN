## 11.7 PetClinic Example

The PetClinic application, available on[GitHub](https://github.com/spring-projects/spring-petclinic), illustrates several features of the_Spring TestContext Framework_in a JUnit 4 environment. Most test functionality is included in the`AbstractClinicTests`, for which a partial listing is shown below:

```java
import static org.junit.Assert.assertEquals;
// import ...

@ContextConfiguration
public abstract class AbstractClinicTests extends AbstractTransactionalJUnit4SpringContextTests {

	@Autowired
	protected Clinic clinic;

	@Test
	public void getVets() {
		Collection<Vet> vets = this.clinic.getVets();
		assertEquals("JDBC query must show the same number of vets",
			super.countRowsInTable("VETS"), vets.size());
		Vet v1 = EntityUtils.getById(vets, Vet.class, 2);
		assertEquals("Leary", v1.getLastName());
		assertEquals(1, v1.getNrOfSpecialties());
		assertEquals("radiology", (v1.getSpecialties().get(0)).getName());
		// ...
	}

	// ...
}
```

Notes:

* This test case extends the`AbstractTransactionalJUnit4SpringContextTests`class, from which it inherits configuration for Dependency Injection \(through the`DependencyInjectionTestExecutionListener`\) and transactional behavior \(through the`TransactionalTestExecutionListener`\).

* The`clinic`instance variable — the application object being tested — is set by Dependency Injection through`@Autowired`semantics.

* The`getVets()`method illustrates how you can use the inherited`countRowsInTable()`method to easily verify the number of rows in a given table, thus verifying correct behavior of the application code being tested. This allows for stronger tests and lessens dependency on the exact test data. For example, you can add additional rows in the database without breaking tests.

* Like many integration tests that use a database, most of the tests in`AbstractClinicTests`depend on a minimum amount of data already in the database before the test cases run. Alternatively, you might choose to populate the database within the test fixture set up of your test cases — again, within the same transaction as the tests.

The PetClinic application supports three data access technologies: JDBC, Hibernate, and JPA. By declaring`@ContextConfiguration`without any specific resource locations, the`AbstractClinicTests`class will have its application context loaded from the default location,`AbstractClinicTests-context.xml`, which declares a common`DataSource`. Subclasses specify additional context locations that must declare a`PlatformTransactionManager`and a concrete implementation of`Clinic`.

For example, the Hibernate implementation of the PetClinic tests contains the following implementation. For this example,`HibernateClinicTests`does not contain a single line of code: we only need to declare`@ContextConfiguration`, and the tests are inherited from`AbstractClinicTests`. Because`@ContextConfiguration`is declared without any specific resource locations, the_Spring TestContext Framework_loads an application context from all the beans defined in`AbstractClinicTests-context.xml`\(i.e., the inherited locations\) and`HibernateClinicTests-context.xml`, with`HibernateClinicTests-context.xml`possibly overriding beans defined in`AbstractClinicTests-context.xml`.

```java
@ContextConfiguration
public class HibernateClinicTests extends AbstractClinicTests { }
```

In a large-scale application, the Spring configuration is often split across multiple files. Consequently, configuration locations are typically specified in a common base class for all application-specific integration tests. Such a base class may also add useful instance variables — populated by Dependency Injection, naturally — such as a`SessionFactory`in the case of an application using Hibernate.

As far as possible, you should have exactly the same Spring configuration files in your integration tests as in the deployed environment. One likely point of difference concerns database connection pooling and transaction infrastructure. If you are deploying to a full-blown application server, you will probably use its connection pool \(available through JNDI\) and JTA implementation. Thus in production you will use a`JndiObjectFactoryBean`or`<jee:jndi-lookup>`for the`DataSource`and`JtaTransactionManager`. JNDI and JTA will not be available in out-of-container integration tests, so you should use a combination like the Commons DBCP`BasicDataSource`and`DataSourceTransactionManager`or`HibernateTransactionManager`for them. You can factor out this variant behavior into a single XML file, having the choice between application server and a 'local' configuration separated from all other configuration, which will not vary between the test and production environments. In addition, it is advisable to use properties files for connection settings. See the PetClinic application for an example.

