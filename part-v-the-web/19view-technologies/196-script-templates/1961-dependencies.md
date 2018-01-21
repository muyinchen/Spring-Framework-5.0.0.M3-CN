### 19.6.1Dependencies

To be able to use script templates integration, you need to have available in your classpath the script engine:

* [Nashorn](http://openjdk.java.net/projects/nashorn/)Javascript engine is provided builtin with Java 8+. Using the latest update release available is highly recommended.

* [Rhino](http://docs.oracle.com/javase/7/docs/technotes/guides/scripting/programmer_guide/#jsengine)Javascript engine is provided builtin with Java 6 and Java 7. Please notice that using Rhino is not recommended since it does not support running most template engines.

* [JRuby](http://jruby.org/)dependency should be added in order to get Ruby support.

* [Jython](http://www.jython.org/)dependency should be added in order to get Python support.

You should also need to add dependencies for your script based template engine. For example, for Javascript you can use[WebJars](http://www.webjars.org/)to add Maven/Gradle dependencies in order to make your javascript libraries available in the classpath.

