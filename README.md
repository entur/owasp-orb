# owasp-orb
A Circle CI orb executing OWASP dependency analysis for Gradle and Maven projects.

## Usage
Import the orb

```yaml
orbs:
  owasp: entur/owasp@0.0.3
```

## Gradle

Configure a job

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/gradle_owasp_dependency_check:
          executor: java_11
          context: global
```

Then add [OWASP Gradle Plugin](https://github.com/jeremylong/DependencyCheck) to your gradle build:

```
plugins {
    id 'org.owasp.dependencycheck' version '4.0.2'
}


dependencyCheck {
    analyzedTypes = ['jar'] // the default artifact types that will be analyzed.
    format = 'ALL' // CI-tools usually needs XML-reports, but humans needs HTML.
    failBuildOnCVSS = 0 // Specifies if the build should be failed if a CVSS score equal to or above a specified level is identified.
    suppressionFiles = ["$projectDir/dependencycheck-base-suppression.xml"] // specify a list of known issues which contain false-positives

    data {
        directory = System.properties['user.home'] + "/.owasp-dependency-check" // must correspond with CircleCI-configuration
    }
}
```

where the data directory __must correspond__ to the orb job parameter `cve_data_directory` (default value is `~/.owasp-dependency-check` like in the configuration above).


#### Details
The default OWASP plugin task is `dependencyCheckAnalyze`, for using other tasks, add a `task` parameter as so:

```
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/owasp_dependency_check:
          executor: java_11
          context: global
          task: dependencyCheckAggregate
```

where task is one of [dependencyCheckAnalyze](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration.html), [dependencyCheckAggregate](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration-aggregate.html), [dependencyCheckUpdate](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration-update.html), and [dependencyCheckPurge](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration-purge.html).

## Maven 
Configure a job

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/maven_owasp_dependency_check:
          executor: java_11
          context: global
```

Then add [OWASP Maven Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/index.html) to your Maven build:

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>5.0.0</version>
    <configuration>
        <format>all</format>
        <outputDirectory>target/owasp-reports</outputDirectory>
        <failBuildOnCVSS>7</failBuildOnCVSS>
        <dataDirectory>~/.owasp-dependency-check</dataDirectory>
    </configuration>
    <executions>
        <execution>
            <goals>
                <!-- run only using explicit command -->
                <goal>none</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```

where the data directory __must correspond__ to the orb job parameter `cve_data_directory` (default value is `~/.owasp-dependency-check` like in the configuration above).

## Caching
The OWASP plugin checks for updates to its database every four hours, and the database is cached by the orb like so:

 * Year
 * Quarter (12 weeks)
 * Month (4 weeks)
 * Week
 * Day
 * 12 hours
 * 4 hours

So for each working day, the first builds (in the morning) will check for updates, and last for four hours with potential cache refreshes every four clock hours (at 9, 13, 17, 21 and so on). In other words, the OWASP plugin will check for updates whenever four hours have passed, and will be able to persist those updates to CircleCI cache in maximum four hours - a compromise between time spent saving cache and time spent checking for updates.

See the [orb](/src/@orb.yml) source or [CircleCI orb registry](https://circleci.com/orbs/registry/orb/entur/owasp) for further details.
