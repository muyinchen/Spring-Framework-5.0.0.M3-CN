## 17.3Using Marshaller and Unmarshaller

Spring’s OXM can be used for a wide variety of situations. In the following example, we will use it to marshal the settings of a Spring-managed application as an XML file. We will use a simple JavaBean to represent the settings:

```java
public class Settings {

	private boolean fooEnabled;

	public boolean isFooEnabled() {
		return fooEnabled;
	}

	public void setFooEnabled(boolean fooEnabled) {
		this.fooEnabled = fooEnabled;
	}
}
```

The application class uses this bean to store its settings. Besides a main method, the class has two methods:`saveSettings()`saves the settings bean to a file named`settings.xml`, and`loadSettings()`loads these settings again. A`main()`method constructs a Spring application context, and calls these two methods.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import javax.xml.transform.stream.StreamResult;
import javax.xml.transform.stream.StreamSource;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.oxm.Marshaller;
import org.springframework.oxm.Unmarshaller;

public class Application {

	private static final String FILE_NAME = "settings.xml";
	private Settings settings = new Settings();
	private Marshaller marshaller;
	private Unmarshaller unmarshaller;

	public void setMarshaller(Marshaller marshaller) {
		this.marshaller = marshaller;
	}

	public void setUnmarshaller(Unmarshaller unmarshaller) {
		this.unmarshaller = unmarshaller;
	}

	public void saveSettings() throws IOException {
		FileOutputStream os = null;
		try {
			os = new FileOutputStream(FILE_NAME);
			this.marshaller.marshal(settings, new StreamResult(os));
		} finally {
			if (os != null) {
				os.close();
			}
		}
	}

	public void loadSettings() throws IOException {
		FileInputStream is = null;
		try {
			is = new FileInputStream(FILE_NAME);
			this.settings = (Settings) this.unmarshaller.unmarshal(new StreamSource(is));
		} finally {
			if (is != null) {
				is.close();
			}
		}
	}

	public static void main(String[] args) throws IOException {
		ApplicationContext appContext =
				new ClassPathXmlApplicationContext("applicationContext.xml");
		Application application = (Application) appContext.getBean("application");
		application.saveSettings();
		application.loadSettings();
	}
}
```

The`Application`requires both a`marshaller`and`unmarshaller`property to be set. We can do so using the following`applicationContext.xml`:

```java
<beans>
	<bean id="application" class="Application">
		<property name="marshaller" ref="castorMarshaller" />
		<property name="unmarshaller" ref="castorMarshaller" />
	</bean>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller"/>
</beans>
```

This application context uses Castor, but we could have used any of the other marshaller instances described later in this chapter. Note that Castor does not require any further configuration by default, so the bean definition is rather simple. Also note that the`CastorMarshaller`implements both`Marshaller`and`Unmarshaller`, so we can refer to the`castorMarshaller`bean in both the`marshaller`and`unmarshaller`property of the application.

This sample application produces the following`settings.xml`file:

```java
<?xml version="1.0" encoding="UTF-8"?>
<settings foo-enabled="false"/>
```



