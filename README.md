# Toy Example

## Folder Organization and Fault Description

This toy example is organized as follows:

* `toyproject` folder contains sources (folder `src`) and binaries (folder `bin`)
* `myclasses.txt` is a file that lists the toyproject classes.

The Java class `ResourceManager` (`ResourceManager.java`) contains a bug:

```java
    /**
     * Tells if it is possible to use the resource.
     *
     * @return true if the resource is not locked, false otherwise
     */
    public boolean canUseResource(){
        // bug! Should be: !resource.isLocked();
        return resource.isLocked();
    }
```

The return statement of method `canUseResource()` does not match what the Javadoc `@return` tag states. Randoop, by itself, is not able to detect such bug. However, when Jdoctor specifications are supplied to Randoop, it will be able to detect the bug.

## Prerequisites

You need Jdoctor and Randoop jars to try this example.

- Clone this repository: `git clone https://github.com/ariannab/toyproject`

- Download Jdoctor: `wget link/to/toradocu.jar`

- Download Randoop: `wget https://github.com/randoop/randoop/releases/download/v4.0.3/randoop-all-4.0.3.jar`

## Automatic Fault Detection with Jdoctor

Follow the steps to try it yourself.

1. Compile the example sources:

`mkdir toyproject/bin/ && javac toyproject/src/* -d toyproject/bin/`

2. Generate the Jdoctor specifications over the example class:

`java -jar toradocu-1.0-all.jar --target-class ResourceManager --source-dir toyproject/src/ --class-dir toyproject/bin/ --randoop-specs toy-specs.json`

`toy-specs.json` contains the specifications generated by JDoctor over class ResourceManager.

3. Run Randoop on the toyproject sources:

`java -classpath randoop-all-4.0.3.jar:toyproject/bin randoop.main.Main gentests --classlist=toyproject/myclasses.txt --time-limit=60`

Randoop runs for 60 seconds on the classes in `myclasses.txt` and then prints the message: "No error-revealing tests to output".

4. Now feed Randoop with the specifications generated at step 2:

`java -classpath randoop-all-4.0.3.jar:toyproject/bin randoop.main.Main gentests --classlist=toyproject/myclasses.txt --time-limit=60 --specifications=toy-specs.json --stop-on-error-test`

Randoop stops almost immediately because an error-revealing test is produced. You should see `ErrorTest0.java` in the current folder.

5. Compile and then execute the error test generated at step 4:

`javac -classpath randoop-all-4.0.3.jar ErrorTest0.java -sourcepath toyproject/src/`

`java -classpath .:randoop-all-4.0.3.jar:toyproject/bin/ org.junit.runner.JUnitCore ErrorTest0`

You should get the following error:
```
java.lang.AssertionError: Post-condition: true if the resource is not locked, false otherwise.
    at org.junit.Assert.fail(Assert.java:88)
    at org.junit.Assert.assertTrue(Assert.java:41)
```
Which means Randoop fed with the Jdoctor specifications was able to detect the bug on the postcondition.
