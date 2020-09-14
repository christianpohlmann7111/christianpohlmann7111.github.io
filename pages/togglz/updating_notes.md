---
title: Updating Notes
keywords: updating
permalink: updating_notes.html
---
This page contains some notes on updating Togglz to newer versions. Although we aim to keep backwards compatibility with older version, in some cases it may be necessary to take care of something when updating. So please have a look at the sections on this page before updating.

## Updating to Togglz 2.6.x from 2.5.x
### Java 8 required
From now on Togglz will require Java 8 or newer. If you are still on Java 7, you have to use an older version.

### Spring 4.x required
Togglz now depends on Spring Framework 4.x and dropped support for version 3.x.

### Spring Boot 2.0
The Togglz Spring Boot integration now requires Spring Boot 2.0 or newer. 
If you are still using Spring Boot 1.x, you will have to use the `togglz-legacy-spring-boot-starter` module.

#### Spring Boot 2.0
```xml
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-spring-boot-starter</artifactId>
  <version>2.6.1.Final</version>
</dependency>
```

#### Spring Boot 1.x
```xml
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-legacy-spring-boot-starter</artifactId>
  <version>2.6.1.Final</version>
</dependency>
```

## Updating to Togglz 2.5.0 from 2.4.0
### Spring Configuration
A Spring Boot application that includes the Togglz starter will automatically be able to pick up feature configuration from the Spring `Environment`. When migrating from older versions the "enabled" flag has to be inserted after the feature name. Example, in 2.4.0:

```
togglz.features.FOO=true
```

becomes in 2.5.0:
```
togglz.features.FOO.enabled=true
```

## Updating to Togglz 2.3.0 from 2.2.0
### Dropped support for Java 6
Togglz 2.3.0 requires Java 7 or newer. Today you have to pay Oracle to even get updates for Java 7, so dropping Java 6 support is a reasonable decision. :)

### Removed Seam Security module
The Seam Security module was a constant source of pain during the release process, because Seam Security isn't available in the Maven Central repository. As Red Hat has halted the active development of Seam, we decided to remove the module. If you need it, you can use an older version of the module or copy the corresponding classes into your application's code base. The source of the module is still in the Togglz source code repository.

## Updating to Togglz 2.2.0 from 2.1.0
### Split of Spring module
In previous Togglz versions the Spring module had a dependency on `spring-web`. This caused problems for people wanting to use Togglz in Spring projects which aren't web applications.

Therefore the `togglz-spring` module has been split into `togglz-spring-core` and `togglz-spring-web`. What does this mean for you? If you are updating from a previous version of Togglz, you will have to use this Maven dependency for the Spring module after updating the version:

```xml
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-spring-web</artifactId> <!-- was 'togglz-spring' before -->
  <version>${togglz.version}</version>
</dependency>
```

## Updating to Togglz 2.1.0 from 2.0.0
### Hash calculation for gradual rollout strategy
The *Gradual Rollout* activation strategy in Togglz 2.0.0 used a simple hash value of the user's name to determine if a feature should be active or not. This has changed in 2.1.0. Since this version the default hash value will be created from the username AND the feature name to provide a better distribution of hash values.

What does this mean for you? This means that if you are currently using the gradual rollout strategy and are updating to 2.1.0 or newer, the users for which the features are active will be different after the update. So please plan your update accordently.

## Updating to Togglz 2.0.0 from 1.1.x
This section described the process of migrating applications that are using Togglz 1.1.x to the new Togglz major version 2.0.0.

### New feature state data structure
Togglz 2.0.0 uses a revised data structure for the feature state. In Togglz 1.1.x the feature state contained a list of user names that could be used to restrict a feature to a certain subset of all users. But feedback from the community showed that there is a demand for more complex strategies for activating features than just the user names. Therefore the data structure has been refactored. Instead of a simple user list, the feature state now allows to select an *activation strategy* and an arbitrary list of custom parameters for that strategy.

As the feature state is persisted and restored by the state repositories, this change affects all repository implementations that Togglz provides and also all custom implementations users have created.

The following sections will describe the changes to the standard state repositories.

Please note that all repositories automatically migrate the persisted feature state to the new format. However, you should double check your application after the update to make sure that everything worked as expected.

#### FileBasedStateRepository
The implementation of `FileBasedStateRepository` has been updated to support the new feature state data structure. Feature state persisted with Togglz 1.1.x represents it like this:

```
FEATURE_ONE = true
FEATURE_ONE.users = chkal, john
```

The repository will still understand this format. But after updating a feature state, the new format will be used:
```
FEATURE_TWO = true
FEATURE_TWO.strategy = username
FEATURE_TWO.param.users = chkal, john
```

As the migration is handled automatically, you typically don't have to manually perform any steps for updating to Togglz 2.0.0. However you should backup your properties file before doing the update, because after the format has been migrated, Togglz 1.1.x won't be able to read the file any more. So keeping a backup is a good idea so you can go back to Togglz 1.1.x in case of problems.

#### JDBCStateRepository
The JDBC-based state repository has been updated to use a new table structure for storing the feature state. In Togglz 1.1.x the feature state was persisted like this:

| FEATURE_NAME | FEATURE_ENABLED | FEATURE_USERS  |
|--------------|-----------------|----------------|
| SOME_FEATURE | 1               | chkal, john    |


In Togglz 2.0.0 the table structure has been adapted to support activation strategies. The new structure looks like this:


| FEATURE_NAME | FEATURE_ENABLED | STRATEGY_ID	| STRATEGY_PARAMS  |
|--------------|-----------------|--------------|------------------|
| SOME_FEATURE | 1               | username     | chkal, john      |


The repository will automatically check the table format during startup. If the old table format is detected, it will be automatically migrated to the new format. The table migration has been tested with various databases. However, you should do a backup of the table before updating to Togglz 2.0.0, because earlier versions of Togglz won't be able to read the new table structure any more.

#### Custom repositories
If you have implemented a custom state repository for your application, you will also have to update the implementation. Have a look at the existing repositories to get an idea of how this can be done.

### Feature enum's isActive() method
Feature enums typically implement a method called `isActive()`. In Togglz 1.1.x this method was enforced by the `Feature` interface. With Togglz 2.0.0 this method was removed from the interface. Therefore if you placed a `@Overrides` annotation on the method, you will have to remove this to satisfy the compiler.

### Deprecated UntypedFeature
The class `UntypedFeature` has been deprecated. If you used this class before, it is recommended to migrate your code to use `NamedFeature` instead.

### Deprecated org.togglz.LOCAL_FEATURE_MANAGER
Togglz 1.1.x allowed to disable the bootstrapping of a local feature manager by setting the servlet context parameter `org.togglz.LOCAL_FEATURE_MANAGER` to `false`. This parameter has been deprecated in Togglz 2.0.0. Although this parameter will still work for some time, you are encouraged to use `org.togglz.FEATURE_MANAGER_PROVIDED` instead.

If you have this entry in your `web.xml`:

```xml
<context-param>
  <param-name>org.togglz.LOCAL_FEATURE_MANAGER</param-name>
  <param-value>false</param-value>
</context-param>
```

You should change this to:
```xml
<context-param>
  <param-name>org.togglz.FEATURE_MANAGER_PROVIDED</param-name>
  <param-value>true</param-value>
</context-param>
```

Please note the inverted logic. To disable the bootstrapping, you had to set the old parameter to `false`. To get the same effect, you will have to set the new parameter to `true`.

