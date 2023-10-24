# gradle-common

Contains common files which support your Gradle build process.

## Purpose 

To reduce configuration efforts and automatically update build scripts of many java-projects. 

It supposes that you attach the repository to your project as submodule, and then you will be able to 
use .gradle files directly from 'src' directory.

Current Gradle project and appropriate 'build.gradle' files are used for testing purposes here. 
Do not use them in your project.   

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


## Usage

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

ext {
  addonsPath = "${projectDir}/addons/src"
}
apply from: "${addonsPath}/git.gradle"
apply from: "${addonsPath}/common.gradle"
apply from: "${addonsPath}/project_info.gradle" // optional
apply from: "${addonsPath}/compose.gradle" // optional
```

3) assign properties with credentials in gradle.properties file:

```properties
cpmArtifactoryReadPassword=xxxxxxxxxxx
```

### for maven libraries


### for liquibase

Add next lines

```groovy
apply from: "${rootProject.ext.addonsPath}/project_info.gradle"
apply from: "${rootProject.ext.addonsPath}/liquibase.gradle"
```

## working with submodule

See:
https://git-scm.com/book/en/v2/Git-Tools-Submodules
https://stackoverflow.com/questions/1777854/how-can-i-specify-a-branch-tag-when-adding-a-git-submodule

- If you have deleted or changed content of "addons" directory, you can restore the state with the command:

```bash
git submodule update -f addons 
```

- Getting the latest state of addons replacing all changes in '/addons' folder that you may have done:

```bash
git submodule update -f --remote addons
```

- Removing submodule from the project:

```bash
git submodule deinit --force addons
git rm -f addons
```

"git submodule deinit" removes reference to "addons" submodule from .git/config file and removes content of "addons" directory.
"git rm addons" removes "addons" directory from GIT-index. 

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
