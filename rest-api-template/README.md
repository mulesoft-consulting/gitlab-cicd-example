```
A generic REST API Template for experience, process, and System APIs. All new REST API projects must be created from this template
```

# Building blocks

### Parent pom

Parent POM is published into the GitLab Maven repository and consumed as a dependency into the REST API Implementations.
- See [Exchange](https://anypoint.mulesoft.com/exchange/1646e1ef-5dd6-4ed5-973a-0ca5ce210da2/api-template---parent-pom/)
- Parent POM is maintained in the following repository - [Parent POM](https://gitlab.com/rail-delivery-group-mulesoft/mule-project-templates/api-parent-pom)

### JSON Logger

> See [Exchange](https://anypoint.mulesoft.com/exchange/1646e1ef-5dd6-4ed5-973a-0ca5ce210da2/json-logger/)

### Common Error Handler

> See [Exchange](https://anypoint.mulesoft.com/exchange/1646e1ef-5dd6-4ed5-973a-0ca5ce210da2/common-error-handler/)

### API Autodiscovery

> Is preconfigured with API Autodiscovery and only the API ID needs to be provided.

### Preconfigured API KIT Router

> Uses the API Kit router which has been configured with the necessary information

### Setup secure properties

- See [here](https://docs.mulesoft.com/mule-runtime/4.3/secure-configuration-properties) for details on how to encrypt secure properties.
- Ensure any secure properties are also updated to the artifact.json to ensure these are safely hidden in cloudhub.


------

> How to Use?

If using GitLab
1. Create new project from Template, and select this API template as the base.
2. Provide relevant Project Name, URL etc. and create project.
3. Once the project successfully creates, use the Git clone URL to clone the repository to local.
4. Open Anypoint Studio - Import - Anypoint Studio - Anypoint Studio project from File System
5. Set project root as the cloned local repo from Step 1
6. Prior to commencing any development, ensure that you are at the develop branch.
```
git checkout -b develop
# If working on a new Feature...
git checkout -b feature-{JIRA Number}
```

> Test

1. Right-click project root, select Run As - Run As Configurations...
2. Select the project from 'Mule projects to launch'
3. Add '-Dmule.env=dev -Dmule.key=mulelongkeyof12characters' to Program arguments (Under (x)-Arguments) tab
4. Click Run, and wait for the project to the build and deployed successfully
5. Check at `http://localhost:8091/api/bye`, You should see `[{"message": "Good Bye!"}]`

> Modify the new Project created from Template

1. Open pom.xml - Update project groupId [Not parent pom groupId]
2. In pom.xml - Update project artifactId [Not parent pom artifactId]
3. Right-click api.xml from Project explorer [Project root - src/main/mule - api.xml], then add the project raml as click on  Anypoint Platform - Import from Design Center - choose the project raml {target-api}.raml [e.g.: customers.raml]
This will generate customer.xml
4. In order to refer the *global error handler* , remove all on `error propogates` in ***{target-api}-main flow***
	1. Go to xml view and add `<error-handler ref="globalError_Handler" />` before `</flow>` [see in api.xml for reference]
```
		<apikit:router config-ref="api-config" />
		<error-handler ref="globalError_Handler" />
	</flow>
```
5. Delete Global Element of {target-api}.xml, Router configuration[{target-api}-config] [e.g.: customers-config]  
6. Open global.xml,
	1. Rename `Router configuration` name to `{target-api}-config` [e.g.: customers-config]
	2. Select api definition to {target-api}.raml [e.g.: customers.raml]
	3. Edit API Autodiscovery configuration to have the 'Flow Name' as `{target-api}-main` [e.g.: customers-main]
7. Delete api.xml and api-impl.xml as you will now need to implement for {target-api}.raml
8. Update the ***api.id*** in all {env}.yaml files.

> Deployments

2. Open terminal/Command Prompt at the local project folder [e.g.: eapi-customers]
3. Push the project source code to the newly created feature branch
```
# stage all changes for commit. Alternatively, specify specific changes ( '.' includes all changed files)
git add .
# Commit changes to local - Include commit Message which should always include a JIRA number and a short description
git commit -m "Initial commit"
# Push changes to remote repository
git push -u origin feature-{JIRA-number}
```
4. Code get pushed to Gitlab[^1]
5. When feature branch gets merged to Develop, then and you should see the application automatically deployed on to the target Develop environment in a few minutes



-----------

> Modify the Template

1. Modify files as needed
2. Increment project version in pom.xml
3. Update in [Exchange](https://anypoint.mulesoft.com/exchange/1646e1ef-5dd6-4ed5-973a-0ca5ce210da2/rest-api-template)
4. Push latest code to [Gitlab](https://gitlab.com/rail-delivery-group-mulesoft/rest-api-template)

------

> Update [Parent POM][1]

 - Modify pom.xml to have the latest parent pom version

```
	<parent>
		<groupId>uk.co.example</groupId>
		<artifactId>api-parent-pom</artifactId>
		<version>${parent.pom.version}</version>
	</parent>
```

 - Increment project version in pom.xml

------

> Publish the Template to Exchange

1. Right-click project root from Project explorer, then Anypoint Platform - Publish to Exchange
2. Confirm the values auto-generated on the window (Project type = Template) and then Finish
3. The new version of the template will be published and available on the template


> Notes

  [1]: https://anypoint.mulesoft.com/exchange/1646e1ef-5dd6-4ed5-973a-0ca5ce210da2/api-template---parent-pom/
  [^1]: Please make careful decisions whether to push to `develop` branch or `feature-branch`
