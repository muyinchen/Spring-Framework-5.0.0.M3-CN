## 19.6Script templates

It is possible to integrate any templating library running on top of a JSR-223 script engine in web applications using Spring. The following describes in a broad way how to do this. The script engine must implement both`ScriptEngine`and`Invocable`interfaces.

It has been tested with:

* [Handlebars](http://handlebarsjs.com/)running on[Nashorn](http://openjdk.java.net/projects/nashorn/)

* [Mustache](https://mustache.github.io/)running on[Nashorn](http://openjdk.java.net/projects/nashorn/)

* [React](https://facebook.github.io/react/)running on[Nashorn](http://openjdk.java.net/projects/nashorn/)

* [EJS](http://www.embeddedjs.com/)running on[Nashorn](http://openjdk.java.net/projects/nashorn/)

* [ERB](http://www.stuartellis.eu/articles/erb/)running on[JRuby](http://jruby.org/)

* [String templates](https://docs.python.org/2/library/string.html#template-strings)running on[Jython](http://www.jython.org/)



