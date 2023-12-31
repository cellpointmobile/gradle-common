# gradle-common

Contains common files which support your Gradle build process.

## Purpose 

To reduce configuration efforts and automatically update build scripts of many java-projects. 

It supposes that you attach the repository to your project as submodule, and then you will be able to 
use .gradle files directly from 'src' directory.

Current Gradle project and appropriate 'build.gradle' files are used for testing purposes here. 
Do not use them in your project.   

## After checking out a branch that is already configured to use this submodule

If the build cannot find gradle files from "addons" directory, you need to run this command:

```bash
git submodule update --init
```

This command will download appropriate GIT repository and place it into "addons" directory.


## How to configure your project for using this submodule

### for all project types

1) Attach submodule to your project (from root project directory):

```bash
git submodule add --force https://github.com/cellpointmobile/gradle-common.git addons
```

It creates *addons* directory that contains last state of "main" branch of "gradle-common" repository. 


2) You must define **ext.addonsPath** variable, and attach .gradle scripts, add plugin "com.gorylenko.gradle-git-properties"
   into your **root** build.gradle. The root build.gradle may looks like:
 
```groovy
plugins {
  // declare versions of used plugins and attach them into gradle classpath
  id "com.gorylenko.gradle-git-properties" version "2.4.1" apply false
  id 'com.github.spotbugs' version '5.0.14' apply false // remove if you don`t want to use spotbugs  
}

ext.addonsPath = "${projectDir}/addons/src"
apply from: "${addonsPath}/git.gradle"
apply from: "${addonsPath}/common.gradle"
apply from: "${addonsPath}/project_info.gradle" // optional
apply from: "${addonsPath}/compose.gradle" // optional
```

3) assign properties with credentials in gradle.properties file:

```properties
cpmArtifactoryReadPassword=xxxxxxxxxxx
```

4) If you use *cellpointdigital/github-reusable-workflows/.github/workflows/gradle-build.yml@main* common reusable flow,
  please add **checkout_submodules: 'true'** to its configuration, example:

```yaml
jobs:
   gradle-build-server:
      uses: cellpointdigital/github-reusable-workflows/.github/workflows/gradle-build.yml@main
      with:
         java-version: 17
         artifact_path: server/build/docker
         artifact_name: server
         gradle-arguments: :server:build :server:dockerPrepare --stacktrace
         checkout_submodules: 'true'
      secrets: inherit
```

### for liquibase

replace liquibase/build.gradle with these only 2 lines:

```groovy
apply from: "${addonsPath}/project_info.gradle"
apply from: "${addonsPath}/liquibase.gradle"
```


### for maven libraries

Configuration for libraries is not fully completed and tested, but you can use all .gradle files.

```groovy
plugins {
    id 'java-library'
    id 'maven-publish'
}

java {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

test {
    useJUnitPlatform()
}

configurations {
    testImplementation.extendsFrom compileOnly
}

apply(from: rootProject.file('addons/repositories.gradle'))
apply(from: rootProject.file('addons/jacoco.gradle'))
apply(from: rootProject.file('addons/publishing.gradle'))

publishing {

    // getting the name for the maven artifact and publishing tasks:
    def projName = project.name.substring(project.name.lastIndexOf('.') + 1)
    def forTaskName = projName.replace("-","").toUpperCase()
    publications.register(forTaskName, MavenPublication) {
        // you may replace groupId, artifactId, version as you need
        groupId = rootProject.name  
        artifactId = projName
        version = project.version
        
        from components.java
    }

}

dependencies {
    // any
}
```

## Security

These scripts MUST not have credentials in any form, because they locate in public GitHub repository.

To get credentials they use gradle properties and environment variables.
You need to define properties in **your gradle.properties** file or set environment variables.
Environment variables take precedence over properties.

Used properties and environment variables:

| property | env variable | descrition                                                                                       | default                                             |
|-|-|-|-|
|cpmArtifactoryReadUsername|CPM_ARTIFACTORY_READ_USERNAME| username for readonly access to common maven repository.| 'cellpointmobileread'                               | 
|cpmArtifactoryReadPassword|CPM_ARTIFACTORY_READ_PASSWORD| password for readonly access to common maven repository.| ''                                                  |
|cpmArtifactoryWriteUsername|CPM_ARTIFACTORY_WRITE_USERNAME| username for write access to common maven repository.| 'github'                                            |
|cpmArtifactoryWritePassword|CPM_ARTIFACTORY_WRITE_PASSWORD| password for write access to common maven repository. <br/> This password usually assigned by CI | 'jenkinspasswordplaceholder' used as a placeholder |   



## working with submodule

See:
https://git-scm.com/book/en/v2/Git-Tools-Submodules
https://stackoverflow.com/questions/1777854/how-can-i-specify-a-branch-tag-when-adding-a-git-submodule

- If you have deleted or changed content of "addons" directory, you can restore its state with the command:

```bash
git submodule update -f addons 
```

- Getting the latest state of addons replacing all changes in '/addons' folder that you may have done:

```bash
git submodule update -f --remote addons  # (1) 
git add addons  # (2)   
```

1) getting the latest state from "addons" repository.
2) index git-sha of "addons" repository, to link current state of the project source-code with state of "addons" repository.

You may use gradle task for that too: 
```bash
./gradlew updateAddons
```

- Removing submodule from the project:

```bash
git submodule deinit --force addons  # (1)
git rm -f addons   # (2)
```
1) removes reference to "addons" submodule from .git/config file and removes content of "addons" directory.
2) removes "addons" directory from GIT-index.


### Remarks

According to the GitHub documentation, "git@github.com:" in submodule URL will be changed to
"https://github.com/" automatically, if SSL certificate is not used, and in that case we must use PAT authorization.
But because of standard GitHub credentials always have SSL client certificate, the GitHub workflow action 
cannot get to the repo. Public repositories in GitHub are accessible only by https protocol. 

Show indexed submodule directories:
```bash
git ls-files --stage | grep 160000
```

Show last commit in "addons" submodule:
```bash
pushd addons; git show --summary; popd
```
