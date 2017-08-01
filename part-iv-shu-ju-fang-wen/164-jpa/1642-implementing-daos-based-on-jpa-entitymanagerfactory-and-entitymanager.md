### 16.4.2Implementing DAOs based on JPA: EntityManagerFactory and EntityManager

| ![](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/note.png.pagespeed.ce.9zQ_1wVwzR.png) |
| :--- |
| Although`EntityManagerFactory`instances are thread-safe,`EntityManager`instances are not. The injected JPA`EntityManager`behaves like an`EntityManager`fetched from an application server’s JNDI environment, as defined by the JPA specification. It delegates all calls to the current transactional`EntityManager`, if any; otherwise, it falls back to a newly created`EntityManager`per operation, in effect making its usage thread-safe. |

It is possible to write code against the plain JPA without any Spring dependencies, by using an injected`EntityManagerFactory`or`EntityManager`. Spring can understand`@PersistenceUnit`and`@PersistenceContext`annotations both at field and method level if a`PersistenceAnnotationBeanPostProcessor`is enabled. A plain JPA DAO implementation using the`@PersistenceUnit`annotation might look like this:

```java
public class ProductDaoImpl implements ProductDao {

	private EntityManagerFactory emf;

	@PersistenceUnit
	public void setEntityManagerFactory(EntityManagerFactory emf) {
		this.emf = emf;
	}

	public Collection loadProductsByCategory(String category) {
		EntityManager em = this.emf.createEntityManager();
		try {
			Query query = em.createQuery("from Product as p where p.category = ?1");
			query.setParameter(1, category);
			return query.getResultList();
		}
		finally {
			if (em != null) {
				em.close();
			}
		}
	}
}
```

The DAO above has no dependency on Spring and still fits nicely into a Spring application context. Moreover, the DAO takes advantage of annotations to require the injection of the default`EntityManagerFactory`:

```java
<beans>

	<!-- bean post-processor for JPA annotations -->
	<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

As an alternative to defining a`PersistenceAnnotationBeanPostProcessor`explicitly, consider using the Spring`context:annotation-config`XML element in your application context configuration. Doing so automatically registers all Spring standard post-processors for annotation-based configuration, including`CommonAnnotationBeanPostProcessor`and so on.

```java
<beans>

	<!-- post-processors for all standard config annotations -->
	<context:annotation-config/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

The main problem with such a DAO is that it always creates a new`EntityManager`through the factory. You can avoid this by requesting a transactional`EntityManager`\(also called "shared EntityManager" because it is a shared, thread-safe proxy for the actual transactional EntityManager\) to be injected instead of the factory:

```java
public class ProductDaoImpl implements ProductDao {

	@PersistenceContext
	private EntityManager em;

	public Collection loadProductsByCategory(String category) {
		Query query = em.createQuery("from Product as p where p.category = :category");
		query.setParameter("category", category);
		return query.getResultList();
	}
}
```

The`@PersistenceContext`annotation has an optional attribute`type`, which defaults to`PersistenceContextType.TRANSACTION`. This default is what you need to receive a shared EntityManager proxy. The alternative,`PersistenceContextType.EXTENDED`, is a completely different affair: This results in a so-called extended EntityManager, which is_not thread-safe_and hence must not be used in a concurrently accessed component such as a Spring-managed singleton bean. Extended EntityManagers are only supposed to be used in stateful components that, for example, reside in a session, with the lifecycle of the EntityManager not tied to a current transaction but rather being completely up to the application.

**Method- and field-level Injection**

Annotations that indicate dependency injections \(such as`@PersistenceUnit`and`@PersistenceContext`\) can be applied on field or methods inside a class, hence the expressions_method-level injection_and_field-level injection_. Field-level annotations are concise and easier to use while method-level allows for further processing of the injected dependency. In both cases the member visibility \(public, protected, private\) does not matter.

What about class-level annotations?

On the Java EE platform, they are used for dependency declaration and not for resource injection.

The injected`EntityManager`is Spring-managed \(aware of the ongoing transaction\). It is important to note that even though the new DAO implementation uses method level injection of an`EntityManager`instead of an`EntityManagerFactory`, no change is required in the application context XML due to annotation usage.

The main advantage of this DAO style is that it only depends on Java Persistence API; no import of any Spring class is required. Moreover, as the JPA annotations are understood, the injections are applied automatically by the Spring container. This is appealing from a non-invasiveness perspective, and might feel more natural to JPA developers.

