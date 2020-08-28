**Parent POM source code**

Parent POM.xml for all new templates/projects
> Note: This asset is deployed to the GitLab Artifact repository and inherited into individual projects. 

- All new templates/projects must use the project pom
```
    <parent>
        <groupId>uk.co.company</groupId>
        <artifactId>api-parent-pom</artifactId>
        <version>1.0.0</version>
    </parent>
```

**How to modify Parent POM?**

*  Clone the repository and and make neceessary changes. Publish to exchange by maven deploy and update code repository

`mvn deploy -DdeployToExchange=true`
