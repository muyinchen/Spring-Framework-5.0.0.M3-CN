## 18.13 Convention over configuration support

For a lot of projects, sticking to established conventions and having reasonable defaults is just what they \(the projects\) need, and Spring Web MVC now has explicit support for_convention over configuration_. What this means is that if you establish a set of naming conventions and suchlike, you can_substantially_cut down on the amount of configuration that is required to set up handler mappings, view resolvers,`ModelAndView`instances, etc. This is a great boon with regards to rapid prototyping, and can also lend a degree of \(always good-to-have\) consistency across a codebase should you choose to move forward with it into production.

Convention-over-configuration support addresses the three core areas of MVC: models, views, and controllers.

