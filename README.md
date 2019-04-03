# owasp-orb
A Circle CI orb executing OWASP dependency analysis for Gradle projects.

## Usage
Import the orb

```yaml
orbs:
  owasp: entur/owasp@0.0.2
```

Then configure a job

```yaml
workflows:
  version: 2
  build:
    jobs:
      - owasp/owasp_dependency_check_analyze:
          executor: java_11
          context: global
```

Then add [OWASP Dependency Plugin](https://github.com/jeremylong/DependencyCheck) to your gradle build:

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
        directory = System.properties['user.home'] + "/.owasp-dependency-check"
    }
}
```

where the data directory must correspond to the orb job parameter `cve_data_directory` (default value is `~/.owasp-dependency-check` like in the configuration above). 

## Details
Supported jobs are

 * owasp_dependency_check_analyze for single-module projects
 * owasp_dependency_check_aggregate for multi-module projects 

The OWASP plugin updates its database every 4 hours or so, and the database is cached by the orb.

See the [orb](/src/@orb.yml) for further details.
