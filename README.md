
# owasp-orb
A Circle CI orb using [OWASP Dependency Check](https://jeremylong.github.io/DependencyCheck/) to check for components with known security-vulnerablities. Supported variants:

  * [Gradle plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/index.html)
  * [Maven plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/index.html)
  * [Command Line Tool](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/index.html)

## Usage
Import the orb

```yaml
orbs:
  owasp: entur/owasp@0.0.x
```
where `x` is the latest version from [the orb registry](https://circleci.com/orbs/registry/orb/entur/owasp).

### Default executor
To use the default executor, [Docker Hub credentials](https://circleci.com/docs/2.0/private-images/) must be set as the environment variables `$DOCKERHUB_LOGIN` and `$DOCKERHUB_PASSWORD`.

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

```groovy
plugins {
    id 'org.owasp.dependencycheck' version '5.2.4'
}

dependencyCheck {
    analyzedTypes = ['jar'] // the default artifact types that will be analyzed.
    format = 'ALL' // CI-tools usually needs XML-reports, but humans needs HTML.
    failBuildOnCVSS = 7 // Specifies if the build should be failed if a CVSS score equal to or above a specified level is identified.
    suppressionFiles = ["$projectDir/dependencycheck-base-suppression.xml"] // specify a list of known issues which contain false-positives
}
```

#### Details
The default OWASP plugin task is `dependencyCheckAnalyze`, for using other tasks, add a `task` parameter as so:

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/gradle_owasp_dependency_check:
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
    <version>5.2.4</version>
    <configuration>
        <format>all</format>
        <failBuildOnCVSS>7</failBuildOnCVSS>
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

#### Details

The default OWASP plugin task is `check`, for using other tasks, add a `task` parameter as so:

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/maven_owasp_dependency_check:
          executor: java_11
          task: aggregate
          context: global
```

## Command Line Tool
Configure a job

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/commandline_owasp_dependency_check:
          executor: java_11
          context: global
```
#### Details
The default OWASP arguments is `--scan ./`, for using other commands, add an `arguments` parameter as so:

```yaml
workflows:
  version: 2.1
  build:
    jobs:
      - owasp/commandline_owasp_dependency_check:
          executor: java_11
          arguments: "--scan ./ --failOnCVSS 7 --suppression ./dependency-check-suppressions.xml"
          context: global
```

See the [arguments page](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/arguments.html) for further details. Note that `--format`, `--data` and `--noupdate` arguments are already appended by this orb (updating the database is performed in an individual previous step).

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

### Data directory
Use the orb parameter `cve_data_directory` to configure non-standard data directory. Note that for Gradle builds this is necessary for plugin version <= `5.1.0`.

Configuration examples (using default directories):

#### Gradle

```
dependencyCheck {
    data {
        // must correspond with CircleCI-configuration
        directory = System.properties['user.home'] + "/.gradle/dependency-check-data" 
    }
}
```

for `cve_data_directory` parameter value `~/.gradle/dependency-check-data`.

#### Maven 

```xml
<configuration>
    <!-- must correspond with CircleCI-configuration -->
    <dataDirectory>${user.home}/.m2/repository/org/owasp/dependency-check-data</dataDirectory>
</configuration>
```

for `cve_data_directory` parameter value `~/.m2/repository/org/owasp/dependency-check-data`.

## Further reading
See the [orb](/src/@orb.yml) source or [CircleCI orb registry](https://circleci.com/orbs/registry/orb/entur/owasp) for further details.
