---
title: Quickstart
keywords: quickstart
permalink: quickstart.html
sidebar: mydoc_sidebar
folder: togglz
---

This small tutorial will give an overview on how to get started with Togglz.

## Installation

First add following dependencies to your project:
```xml
<!-- Togglz for Servlet environments (mandatory) --> 
<dependency> <groupId>org.togglz</groupId> 
  <artifactId>togglz-servlet</artifactId> 
  <version>2.6.1.Final</version> 
</dependency> 

<!-- CDI integration (optional) --> 
<dependency> <groupId>org.togglz</groupId> 
  <artifactId>togglz-cdi</artifactId> 
  <version>2.6.1.Final</version> 
</dependency> 

<!-- Spring integration (optional) --> 
<dependency> <groupId>org.togglz</groupId> 
  <artifactId>togglz-spring-web</artifactId> 
  <version>2.6.1.Final</version> 
</dependency>
```


If you deploy your application to a Servlet 3.0 container, you won't have to do any further configuration. In case your container doesn't support Servlet 3.0, you have to manually register the required filter. Please refer to [Installation](/documentation/installation.html) for details.

## Configuration

First create your feature enum class:

```java
public enum MyFeatures implements Feature {

    @EnabledByDefault
    @Label("First Feature")
    FEATURE_ONE,
    
    @Label("Second Feature")
    FEATURE_TWO;
    
    public boolean isActive() {
        return FeatureContext.getFeatureManager().isActive(this);
    }
    
}
```

The next step is to create an implementation of `TogglzConfig`:

```java
@ApplicationScoped
public class DemoConfiguration implements TogglzConfig {

    public Class<? extends Feature> getFeatureClass() {
        return MyFeatures.class;
    }

    public StateRepository getStateRepository() {
        return new FileBasedStateRepository(new File("/tmp/features.properties"));
    }

    public UserProvider getUserProvider() {
        return new ServletUserProvider("admin");
    }

}
```

See the following short description for a description of each method:

-   `getFeatureClass()`: This method is used to tell Togglz about your feature enum. Just return the class of the enum here.
-   `getStateRepository()`: This method must return the feature state repository which is responsible to persist the state of features. This example uses the `FileBasedStateRepository` which uses Java property files to save the feature state. But Toggles provides several other implementations. Please refer to [State Repositories](/documentation/repositories.html) for details.
-   `getUserProvider()`: This method returns a provider class that is responsible to tell the FeatureManager which is the _current user_. Togglz provides out-of-the-box integration with many popular security frameworks. Please refer to [User Authentication](/documentation/authentication.html) for details.

## Usage

Using Togglz is very simple. To check if a feature is active just call the `isActive()` method of the corresponding feature enum value:

```java
if( MyFeatures.FEATURE_ONE.isActive() ) {
  // new stuff here
}
```

## Admin Console

Togglz ships with an embedded admin console that allows you to toggle your features:

![](/images/screenshot-admin-console.png)

To enable the embedded Togglz admin console, add the following dependency to your `pom.xml`:

```xml
<!-- Togglz Admin Console -->
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-console</artifactId>
  <version>2.6.1.Final</version>
</dependency>
```

If you deploy your application to a Servlet 3.0 container, you won't have to do any further configuration. If your container doesn't support Servlet 3.0, you will have to manually add the `TogglzConsoleServlet` to your `/WEB-INF/web.xml`. Please refer to [Installation](/documentation/installation.html) for details.

The admin console will be available in a virtual path named `togglz` within you application. So if your application is deployed to a context path named `myapp` within the container, you will have to use the following URL:

[http://localhost:8080/myapp/togglz/](http://localhost:8080/myapp/togglz/)

## JSF integration

To make use of the JSF integration features, add the following module to your pom:

```xml
<!-- JSF integration -->
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-jsf</artifactId>
  <version>2.6.1.Final</version>
</dependency>
```

The JSF integration provides a very simple way to render page content depending on whether a feature is active or not:

```thymeleaftemplatesfragmentexpressions
<h:panelGroup rendered="#{features['FEATURE_ONE']}">
  <!-- Some part of the page required for FEATURE_ONE -->
</h:panelGroup>
```

### Spring Boot Starter <span class="label label-danger">since 2.3.0</span>

If you are using Spring Boot there is an alternative installation approach by using the `togglz-spring-boot-starter` module. This starter will install the common Togglz dependencies and will also perform auto configuration. See the [Spring Boot Starter](/documentation/spring-boot-starter.html) chapter for more info.