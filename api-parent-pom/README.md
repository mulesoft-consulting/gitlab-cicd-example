**Parent POM for Mule Applications**

Parent POM.xml for all new templates/projects.
- This asset encapsuates common configurations and dependencies across Mule APIs and applications. This asset also contains the configuration used to deploy Mule applications to Cloudhub using the Mule Maven Plugin.
- Where specific requirements for API templates or other organization specific conisdeations is required, please modify this asset as required.
- The Parent POM configuration currently assumes the use of GitLab Packages as a maven repository but this can be updated as required.
- The variables used in this template are in line with the configuration documents used to configure GitLab CICD. 
- All new templates/projects must use the project pom. Note the version of this resource should be the latest relevant version.
```
    <parent>
        <groupId>com.example</groupId>
        <artifactId>api-parent-pom</artifactId>
        <version>1.0.0</version>
    </parent>
```
