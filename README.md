Error in annotation processors for mongodb data example

Currently, the annotation processors are specified as so

```xml
  <annotationProcessorPaths combine.children="append">
    <path>
      <groupId>io.micronaut</groupId>
      <artifactId>micronaut-http-validation</artifactId>
      <version>${micronaut.version}</version>
    </path>
    <path>
      <groupId>io.micronaut.data</groupId>
      <artifactId>micronaut-data-processor</artifactId>
      <version>${micronaut.data.version}</version>
    </path>
    <path>
      <groupId>io.micronaut.data</groupId>
      <artifactId>micronaut-data-document-processor</artifactId>
      <version>${micronaut.data.version}</version>
    </path>
  </annotationProcessorPaths>
```

This works, as can be shown by running a mongo database and sending a request via CuRL:

```shell
docker run -d --rm -p 27017:27017 --name mongodb mongo
./mvnw mn:run
```

```shell
curl -i -X POST -H "Content-Type: application/json" -d '{"name":"apple"}' http://localhost:8080/fruits
curl http://localhost:8080/fruits
```

## What is the problem?

If we switch the processor order to:

```xml
  <annotationProcessorPaths combine.children="append">
    <path>
      <groupId>io.micronaut</groupId>
      <artifactId>micronaut-http-validation</artifactId>
      <version>${micronaut.version}</version>
    </path>
    <path>
      <groupId>io.micronaut.data</groupId>
      <artifactId>micronaut-data-document-processor</artifactId>
      <version>${micronaut.data.version}</version>
    </path>
    <path>
        <groupId>io.micronaut.data</groupId>
        <artifactId>micronaut-data-processor</artifactId>
        <version>${micronaut.data.version}</version>
    </path>
  </annotationProcessorPaths>
```

Then we get a NPE during compilation:

```shell
❯ ./mvnw clean mn:run
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< example.micronaut:micronautguide >------------------
[INFO] Building micronautguide 0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ micronautguide ---
[INFO] Deleting /Users/tim/Code/GitHub/bloidonia/micronaut-data-mongodb-synchronous-maven-java-processor-order/target
[INFO]
[INFO] >>> micronaut-maven-plugin:3.2.1:run (default-cli) > process-classes @ micronautguide >>>
[INFO]
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ micronautguide ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Using 'UTF-8' encoding to copy filtered properties files.
[INFO] Copying 2 resources
[INFO]
[INFO] --- maven-compiler-plugin:3.10.1:compile (default-compile) @ micronautguide ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 5 source files to /Users/tim/Code/GitHub/bloidonia/micronaut-data-mongodb-synchronous-maven-java-processor-order/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.440 s
[INFO] Finished at: 2022-03-31T16:11:07+01:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.10.1:compile (default-compile) on project micronautguide: Fatal error compiling: java.lang.ExceptionInInitializerError: NullPointerException -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
```

The error seems to be in the data processor

```shell
Caused by: java.lang.NullPointerException
    at java.util.stream.Collectors.lambda$toMap$68 (Collectors.java:1658)
    at java.util.stream.ReduceOps$3ReducingSink.accept (ReduceOps.java:169)
    at java.util.ArrayList$ArrayListSpliterator.forEachRemaining (ArrayList.java:1655)
    at java.util.stream.AbstractPipeline.copyInto (AbstractPipeline.java:484)
    at java.util.stream.AbstractPipeline.wrapAndCopyInto (AbstractPipeline.java:474)
    at java.util.stream.ReduceOps$ReduceOp.evaluateSequential (ReduceOps.java:913)
    at java.util.stream.AbstractPipeline.evaluate (AbstractPipeline.java:234)
    at java.util.stream.ReferencePipeline.collect (ReferencePipeline.java:578)
    at io.micronaut.data.processor.visitors.finders.Restrictions.<clinit> (Restrictions.java:65)
    at io.micronaut.data.processor.visitors.finders.AbstractCriteriaMethodMatch.<clinit> (AbstractCriteriaMethodMatch.java:91)
    at io.micronaut.data.processor.visitors.finders.FindMethodMatcher.match (FindMethodMatcher.java:58)
    at io.micronaut.data.processor.visitors.finders.AbstractPatternMethodMatcher.match (AbstractPatternMethodMatcher.java:63)
    at io.micronaut.data.processor.visitors.RepositoryTypeElementVisitor.visitMethod (RepositoryTypeElementVisitor.java:247)
    at io.micronaut.annotation.processing.visitor.LoadedVisitor.visit (LoadedVisitor.java:169)
    at io.micronaut.annotation.processing.TypeElementVisitorProcessor$ElementVisitor.visitExecutable (TypeElementVisitorProcessor.java:483)
    at io.micronaut.annotation.processing.TypeElementVisitorProcessor$ElementVisitor$1.accept (TypeElementVisitorProcessor.java:377)
    at io.micronaut.annotation.processing.SuperclassAwareTypeVisitor.visitDeclared (SuperclassAwareTypeVisitor.java:80)
    at io.micronaut.annotation.processing.SuperclassAwareTypeVisitor.visitDeclared (SuperclassAwareTypeVisitor.java:57)
```

As an aside, if the data processor is omitted, then we get a different failure in Maven, which doesn't seem to use the transitive processor at all 