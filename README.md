# PCF-Training-02-14-2019
Interesting topics for Devs and Ops who use and/or support PCF

- cf update-cli plugin
- 

# [Spring-Boot Actuators](https://docs.pivotal.io/pivotalcf/2-4/console/using-actuators.html)


When running a Spring Boot App, do you see the Spring Boot symbol on Apps Manager?


![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/Do-you-see-this.png)


If you don't, you will by the end of this section.

**1. Let's start by cloning a demo Spring Boot App:**

```
$ cd /work                                              # my home/working directory
$ git clone https://github.com/rm511130/spring-music
$ cd spring-music
$ ./gradlew clean assemble
$ cat manifest.yml                                      # just to be acquainted with what it contains

---
applications:
- name: spring-music
  memory: 1G
  random-route: true
  path: build/libs/spring-music-1.0.jar
```

**2. Let's make sure your are using a recent version of my CF CLI:**

```
$ cf -v
cf version 6.42.0+0cba12168.2019-01-10
```

If you are lagging behind on versions, you can use the `cf update-cli` plugin to catch-up:

```
$ cf add-plugin-repo CF-Community https://plugins.cloudfoundry.org
$ cf install-plugin -r CF-Community "update-cli"
$ cf update-cli
```

**3. Make sure you are logged into a PCF environment as a Space Developer.**
 
```
$ cf api api.system.pcf4u.com --skip-ssl-validation

$ cf login -u rmeira@pivotal.io
API endpoint: https://api.system.pcf4u.com

Password> 
Authenticating...
OK
Targeted org demo
Targeted space demo
                
API endpoint:   https://api.system.pcf4u.com (API version: 2.120.0)
User:           rmeira@pivotal.io
Org:            demo
Space:          demo
```

So now I'm logged in as rmeira@pivotal.io and I'm targetting the "demo" ORG and "demo" SPACE.

Your USERNAME and targeted ORG and SPACE will obviously be different and pertain to the PCF environment you have access to.

```
$ cf space-users demo demo
Getting users in org demo / space demo as rmeira@pivotal.io

SPACE MANAGER
  admin
  rmeira@pivotal.io

SPACE DEVELOPER
  admin
  rmeira@pivotal.io

SPACE AUDITOR
  No SPACE AUDITOR found
Ralph-Meira-MacBook-pro-9:spring-music ralphmeria$ cf space-users demo demo
Getting users in org demo / space demo as rmeira@pivotal.io

SPACE MANAGER
  admin
  rmeira@pivotal.io

SPACE DEVELOPER
  admin
  rmeira@pivotal.io

SPACE AUDITOR
  No SPACE AUDITOR found
  ```
  
  We now know for sure that rmeira@pivotal.io has the **Space Developer Role** on the Demo/Demo ORG and SPACE.
  
  This is important for the Spring Boot Actuators to work. If you do not have the Space Developer Role, ask your ORG manager for help.
  
**4. Let's make sure our code has the correct lines**

```
$ cat build.gradle
```

![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/springbootlines.png)
  
  
You also need this file (if you normally need to --skip-ssl-validation)  
  
```
$ cat application.properties

management.cloudfoundry.skip-ssl-validation=true
```
  
The complete details on how to activate Spring Boot Actuators on Maven and Gradle projects can be read [here](https://docs.pivotal.io/pivotalcf/2-4/console/spring-boot-actuators.html).

**5. Time to Build and Push**

```
$ cd /work/spring-music                   # just making sure I'm in the right directory
$ ./gradlew clean assemble
$ cf push
```

You should see an output similar to the example shown below:

```
Pushing from manifest to org demo / space demo as rmeira@pivotal.io...
Using manifest file /work/spring-music/manifest.yml
Getting app info...
Updating app with these attributes...
  name:                spring-music
  path:                /work/spring-music/build/libs/spring-music-1.0.jar
  command:             JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" && CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE -totMemory=$MEMORY_LIMIT -loadedClasses=20325 -poolType=metaspace -stackThreads=250 -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher
  disk quota:          1G
  health check type:   port
  instances:           1
  memory:              1G
  stack:               cflinuxfs2
  routes:
    spring-music-optimistic-bilby.apps.pcf4u.com

Updating app spring-music...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 517.65 KiB / 517.65 KiB [===========================================================================================] 100.00% 1s

Waiting for API to complete processing files...

Stopping app...

Staging app and tracing logs...
   Downloading nodejs_buildpack...
   Downloading staticfile_buildpack...
   Downloading java_buildpack_offline...
   Downloading cppcms-buildpack...
   Downloading ruby_buildpack...
   Downloaded staticfile_buildpack
   Downloaded ruby_buildpack
   Downloading php_buildpack...
   Downloading go_buildpack...
   Downloaded cppcms-buildpack
   Downloaded java_buildpack_offline
   Downloaded nodejs_buildpack
   Downloading python_buildpack...
   Downloading dotnet_core_buildpack...
   Downloading binary_buildpack...
   Downloaded dotnet_core_buildpack
   Downloaded go_buildpack
   Downloaded python_buildpack
   Downloaded php_buildpack
   Downloaded binary_buildpack
   Cell 11c6701a-565d-47d9-a5b1-4fd0b656c5db creating container for instance 952e5d7c-8ad1-4040-a391-273d1b372c05
   Cell 11c6701a-565d-47d9-a5b1-4fd0b656c5db successfully created container for instance 952e5d7c-8ad1-4040-a391-273d1b372c05
   Downloading app package...
   Downloading build artifacts cache...
   Downloaded build artifacts cache (132B)
   Downloaded app package (40.6M)
   -----> Java Buildpack v4.16.1 (offline) | https://github.com/cloudfoundry/java-buildpack.git#41b8ff8
   -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.16.0_RELEASE.so (found in cache)
   -----> Downloading Open Jdk JRE 1.8.0_192 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_192.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.0s)
          JVM DNS caching disabled in lieu of BOSH DNS caching
   -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-3.13.0_RELEASE.tar.gz (found in cache)
          Loaded Classes: 19546, Threads: 250
   -----> Downloading Client Certificate Mapper 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.8.0_RELEASE.jar (found in cache)
   -----> Downloading Container Security Provider 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.16.0_RELEASE.jar (found in cache)
   -----> Downloading Spring Auto Reconfiguration 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-2.5.0_RELEASE.jar (found in cache)
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading droplet...
   Uploading build artifacts cache...
   Uploaded build artifacts cache (132B)
   Uploaded droplet (87.3M)
   Uploading complete
   Cell 11c6701a-565d-47d9-a5b1-4fd0b656c5db stopping instance 952e5d7c-8ad1-4040-a391-273d1b372c05
   Cell 11c6701a-565d-47d9-a5b1-4fd0b656c5db destroying container for instance 952e5d7c-8ad1-4040-a391-273d1b372c05

Waiting for app to start...

name:              spring-music
requested state:   started
routes:            spring-music-optimistic-bilby.apps.pcf4u.com
last uploaded:     Wed 13 Feb 15:54:56 EST 2019
stack:             cflinuxfs2
buildpacks:        client-certificate-mapper=1.8.0_RELEASE container-security-provider=1.16.0_RELEASE
                   java-buildpack=v4.16.1-offline-https://github.com/cloudfoundry/java-buildpack.git#41b8ff8 java-main
                   java-opts java-security jvmkill-agent=1.16.0_RELEASE open-jd...

type:            web
instances:       1/1
memory usage:    1024M
start command:   JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1
                 -Djava.io.tmpdir=$TMPDIR
                 -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext
                 -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" &&
                 CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE
                 -totMemory=$MEMORY_LIMIT -loadedClasses=20325 -poolType=metaspace -stackThreads=250 -vmOptions="$JAVA_OPTS") &&
                 echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" &&
                 MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.
                 org.springframework.boot.loader.JarLauncher
     state     since                  cpu      memory         disk           details
#0   running   2019-02-13T20:55:38Z   215.9%   183.1M of 1G   170.8M of 1G   

type:            task
instances:       0/0
memory usage:    1024M
start command:   JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1
                 -Djava.io.tmpdir=$TMPDIR
                 -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext
                 -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" &&
                 CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE
                 -totMemory=$MEMORY_LIMIT -loadedClasses=20325 -poolType=metaspace -stackThreads=250 -vmOptions="$JAVA_OPTS") &&
                 echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" &&
                 MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/.
                 org.springframework.boot.loader.JarLauncher

There are no running instances of this process.

```

**6. Log into Apps Manager and look for your Spring-Music App**

You may not initially see the Spring Boot App Symbol, but you can activate the detection by hitting the Spring-Music App URL:

![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/SpringBootDetected.png)


**7. Time to investigate what Apps Manager has to offer when running Spring Boot Apps**

- There's an older but very pertinent tutorial for you to peruse [here](https://content.pivotal.io/blog/using-spring-boot-actuator-integrations-with-pivotal-cloud-foundry-111).

- The [Pivotal documentation on configuring Spring Boot Actuators](https://docs.pivotal.io/pivotalcf/2-4/console/spring-boot-actuators.html) also provides very interesting details.

- (a) Application Health

![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/AppHealth.png)

- (b) Heap Dump

![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/heapdump.png)

- (c) Heap Dump Analysis

Download and install a Heap Memory Dump Analyzer such as https://www.eclipse.org/mat/

![](https://github.com/rm511130/PCF-Training-02-14-2019/blob/master/images/memoryAnalyzer.png)







  
  
  
