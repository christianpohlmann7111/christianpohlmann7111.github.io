---
title: Example
keywords: example
datatable: true
permalink: example.html
sidebar: mydoc_sidebar
folder: togglz
---

```java
public enum MyFeatures implements Feature {

    @Label("First Feature")
    FEATURE_ONE,

    @Label("Second Feature")
    FEATURE_TWO;

    public boolean isActive() {
        return FeatureContext.getFeatureManager().isActive(this);
    }

}
```

```java
public void someBusinessMethod() {

  if( MyFeatures.FEATURE_ONE.isActive() ) {
    // do new exciting stuff here
  }

  [...]

}
```