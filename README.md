
# Payara embedded Lucene lookup failure reproducer

This project contains a simple EAR composed of an EJB and a WAR.
The EAR includes the library Lucene, which contains a Multi-Release JAR since JDK19.

## Requirements

- Java Development Kit 21
- Apache Maven 3.9 or later

## How to test

By default, the project uses Payara 6.2023.9. Ensure you are using the JDK21 by checking `mvn --version`.

Simply use the command `mvn clean package` and the tests will show the crash:

```
Caused by: java.lang.LinkageError: MemorySegmentIndexInputProvider is missing in Lucene JAR file
        at org.apache.lucene.store.MMapDirectory.lookupProvider(MMapDirectory.java:437)
        at java.base/java.security.AccessController.doPrivileged(AccessController.java:319)
        at org.apache.lucene.store.MMapDirectory.doPrivileged(MMapDirectory.java:395)
        at org.apache.lucene.store.MMapDirectory.<clinit>(MMapDirectory.java:448)
        at org.apache.lucene.store.FSDirectory.open(FSDirectory.java:161)
        at org.apache.lucene.store.FSDirectory.open(FSDirectory.java:156)
        at ch.astorm.ejb.SimpleBean.initLucene(SimpleBean.java:34)
        at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
        at java.base/java.lang.reflect.Method.invoke(Method.java:580)
        at com.sun.ejb.containers.interceptors.BeanCallbackInterceptor.intercept(InterceptorManager.java:1022)
        at com.sun.ejb.containers.interceptors.CallbackChainImpl.invokeNext(CallbackChainImpl.java:72)
        at com.sun.ejb.containers.interceptors.CallbackInvocationContext.proceed(CallbackInvocationContext.java:204)
        ... 55 more
Caused by: java.lang.ClassNotFoundException: org.apache.lucene.store.MemorySegmentIndexInputProvider
        at com.sun.enterprise.loader.ASURLClassLoader.findClassData(ASURLClassLoader.java:807)
        at com.sun.enterprise.loader.ASURLClassLoader.findClass(ASURLClassLoader.java:689)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:593)
        at com.sun.enterprise.loader.CurrentBeforeParentClassLoader.loadClass(CurrentBeforeParentClassLoader.java:83)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:526)
        at java.base/java.lang.Class.forName0(Native Method)
        at java.base/java.lang.Class.forName(Class.java:534)
        at java.base/java.lang.Class.forName(Class.java:513)
        at java.base/java.lang.invoke.MethodHandles$Lookup.findClass(MethodHandles.java:2869)
        at org.apache.lucene.store.MMapDirectory.lookupProvider(MMapDirectory.java:422)
```

This works fine with the JDK17: `JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/ mvn clean package`.

Here is some of the output in the logs when the test succeeds.

```
### [INIT] WAR initialized
### [CREATE] Response: <html><head></head><body>10 leafs created</body></html>
### [QUERY][WAR] Response: <html><head></head><body>Leaf 4</body></html>
### [CLOSE] WAR destroyed
```

## Test with GlassFish

```
mvn clean package -Dappserver.groupId=org.glassfish.main.extras \
                  -Dappserver.artifactId=glassfish-embedded-all \
                  -Dappserver.version=7.0.9
```
