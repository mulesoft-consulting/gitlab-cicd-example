# gitlab-cicd-example

This repository details the CI/CD configurations and usage guidelines to support Mule projects on GitLab. 

## High level software development lifycycle
From a MuleSoft AnyPoint platform perspective, the CI/CD process and tooling will leverage the following MuleSoft components.
1.	Design Center - To support API design specifications.
2.	Exchange - To support collaboration and discovery (reuse) of existing assets.
3.	Studio - To support API development and Unit Testing.
4.	Maven and the Mule Maven Plugin- To automate the build of Anypoint artifacts, and automate deployment to CloudHub environments via AnyPoint Runtime Manager (Note: The Mule Maven Plugin leverages Cloudhub platform APIs under the hood.)


## Project Templates
GitLab supports the ability to define project templates which in turn can be utilized to support development activities. The intention is for this to work as part of the development activity to instantiate projects based on a pre-configured Mule Template. The setup of project templates within GitLab is managed as part of the administration - https://docs.gitlab.com/ee/user/admin_area/custom_project_templates.html
 
1. Initialize project from template
There are different ways of initializing projects based on templates (e.g. using Anypoint Studio, downloading the template from Exchange etc.), however, this is the recommended way as this automatically instantiates the GIT repository and associated branches. 
 
## API parent POM
The API Parent POM contains all Maven build configuration that is not specific to an API, packaged into a reusable parent POM called api-parent-pom.xml - In terms of setting up MuleSoft projects, the API parent POM should be published to a central repository and inherited as a dependency. The attached api-parent-pom folder contains an example API parent POM which can be published into the GitLab artifact repository, and inherited as part of subsequent builds.  

Key configuration encapsulated in the parent POM
•	Common Mule Maven dependencies for REST APIs
•	Build and Test plugin configurations, including code coverage
•	Deploy configurations to deploy assets to Cloudhub
The parent POM is published to the GitLab Maven repository and inherited as a dependency to individual Mule API projects.


## Version Control
The proposed solution utilizes GitFlow as the version control management strategy. Further details around GitFlow can be reviewed in the link below which provide a good perspective and understanding of the GitFlow strategy - https://nvie.com/posts/a-successful-git-branching-model/ It is important to get an understanding of this approach as this forms a key part of the underlying CI/CD configurations. 
 
## Build and Deployment Automation
The build and deployment automation process is automated using GitLab CI pipeline configurations. The base configuration is incorporated into the project template so that this capability is available to projects to utilize and update as required. 
The GitLab CI pipeline is configured as per the GitLab documentation - https://docs.gitlab.com/ee/ci/yaml/ 




3.1 Pipeline Stages
The following pipeline stages exist within the GitLab CI. Note that the underlying pipeline configuration is managed centrally and imported into the relevant project pipelines - https://gitlab.com/rail-delivery-group-mulesoft/mule-project-templates/cicd-base
This is managed per business group and is currently configured for the Travel information Services Business Group. 
For each project, as per the underlying project template, contains the following code which imports the pipeline configuration and applies to the project. This enables pipeline updates to be applied more easily when changes occur. 
See the comments within the project when instantiating a project to ensure that project-specific changes (e.g. for external-facing applications, API versions) are managed appropriately. 
Where diffrerent worker configurations are required for different environments, this can be managed using Project variables - e.g. where UAT needs more Cloudhub Workers - See https://gitlab.com/rail-delivery-group-mulesoft/journey-planning-iptis-sa/-/settings/ci_cd as a reference - in this case, new variables for CH_WORKERS have been set up for UAT and PROD to ensure different deployment configuration in these environments. 
include:
  - project: 'rail-delivery-group-mulesoft/mule-project-templates/cicd-base'
    ref: master
    file: '/travel-info-services/.gitlab-ci.yml'

#Default Variables
variables:
  # This can be updated as part of manually triggering the pipeline and specifying this variable as part of the manual trigger. 
  DEPLOY_VERSION: $CI_COMMIT_TAG
  
  # For external facing applications, update this to ext-rdg so that the DLB mapping rules allow this connectivity. 
  APP_NAME_PREFIX: "rdg"

  # Default application verion. Update based on versioning strategy and changes that are considered breaking changes. 
  APP_VERSION: "v1"

  #These variables represent the default variables to be used as part of the CI/CD pipeline. In terms of environment specific values (e.g. UAT, PROD needing different numbers / types of workers, please update the Project CI/CD environment variables)
  CH_WORKER_TYPE: "MICRO"
  CH_WORKERS: "1"

  # These variables are used as part of creating a release, as part of the maven:prepare/maven:perform release steps. In addition the release branch is created based on the RELEASE_VERSION variable as well. Please update these variables as part of a new release being created. Note - these do no need to be used for minor release version updates within a release branch - this should be managed within the project POM file version configurations. 
  RELEASE_VERSION: "1.0.0"
  NEXT_DEV_VERSION: "1.1.0-SNAPSHOT"
The CI-CD base configuration contains the following
3.1.1 Development Pipeline
This pipeline is used as part of the development cycle, through development, unit testing, and functional testing. Once the development and testing are at a stage where the code can move into User Acceptance, and subsequent non functional testing activities, a release is created using the Release step of this pipeline.
 
Stage	Details	Script
build-develop	AUTOMATED - Build Mule application, run unit tests and report results. As part of this, the scripts execute the Mule Maven plugin to 
•	build the Mule application artifact
•	run all configured unit tests 
•	check that unit test coverage is at least 80% (configured as part of the Parent POM)	build-develop:
  stage: build
  script:
    - echo "Building Mule application, and Executing Unit Tests"
    - mvn $MAVEN_CLI_OPTS clean package test -Dmule.key=${MULE_KEY}
  environment:
    name: DEV
  artifacts:
    reports:
      junit:
        - target/surefire-reports/TEST-*.xml
  only:
    - develop
    - /^feature-.*$/

build-release	AUTOMATED - Build Mule application, run unit tests and report results. As part of this, the scripts execute the Mule Maven plugin to 
•	build the Mule application artifact
•	run all configured unit tests 
•	check that unit test coverage is at least 80% (configured as part of the Parent POM)
•	deploy newly build artifact to the GitLab artifact management repository (release branch only)	build-release:
  stage: build
  script:
    - echo "Building Mule application, and Executing Unit Tests"
    - mvn $MAVEN_CLI_OPTS clean package test deploy -Dmule.key=${MULE_KEY}
  environment:
    name: DEV
  artifacts:
    reports:
      junit:
        - target/surefire-reports/TEST-*.xml
  only:
      - /^release-.*$/

deploy-snapshot	•	DEV environment: AUTOMATED - This stage deploys the snapshot application to the development environment automatically. 
•	TEST environment: MANUAL - There is a manual job available to be triggered to deploy to the Test environment. This is manual by design, based on an agreement with the testing team, and is used to ensure that deployment tasks do not impact any on-going testing activities. 	deploy-snapshot-dev:
  stage: deploy-snapshot
  variables:
    APP_NAME: "${APP_NAME_PREFIX}-${CI_PROJECT_NAME}-${APP_VERSION}-dev"
    # These variables are business group specific - These are controlled within the parent GitLab Group.
    CH_CLIENT_ID: $CH_TRAVELINFOSERVICES_DEV_CLIENT_ID
    CH_CLIENT_SECRET: $CH_TRAVELINFOSERVICES_DEV_CLIENT_SECRET
    CH_BGID: $CH_TRAVELINFOSERVICES_BG_ID
    CH_BG_NAME: $CH_TRAVELINFOSERVICES_BG_NAME
    CH_ENV: ${CH_ENV_DEV}
  environment:
    name: DEV
  only:
    - develop
    - /^release-.*$/
  script:
    - echo "Deploying the project with maven to $CH_ENV_DEV environment"
    - mvn $MAVEN_CLI_OPTS deploy -P cloudhub -DmuleDeploy -DskipTests -Dmule.env=dev -Dcloudhub.skipDeploymentVerification=true $MAVEN_DEPLOY_PROPS

deploy-snapshot-test:
  stage: deploy-snapshot
  variables:
    APP_NAME: "${APP_NAME_PREFIX}-${CI_PROJECT_NAME}-${APP_VERSION}-test"
    # These variables are business group specific - These are controlled within the parent GitLab Group.
    CH_CLIENT_ID: $CH_TRAVELINFOSERVICES_TEST_CLIENT_ID
    CH_CLIENT_SECRET: $CH_TRAVELINFOSERVICES_TEST_CLIENT_SECRET
    CH_ENV: ${CH_ENV_TEST}
    CH_BGID: $CH_TRAVELINFOSERVICES_BG_ID
    CH_BG_NAME: $CH_TRAVELINFOSERVICES_BG_NAME
  environment:
    name: TEST
  # The stage has been agreed to be manual to reduce impact on the Testing Teams.
  when: manual
  only:
    - develop
    - /^release-.*$/
  script:
    - echo "Deploying the project with maven to TEST environment"
    - mvn $MAVEN_CLI_OPTS deploy -P cloudhub -DmuleDeploy -DskipTests -Dmule.env=test -Dcloudhub.skipDeploymentVerification=true $MAVEN_DEPLOY_PROPS
    

release	MANUAL - This stage automatically manages versioning relating to release artifacts. As part of this, the script creates the release version and the next development iteration version, based on the variables (RELEASE_VERSION and NEXT_DEV_VERSION). 
In addition, the script tags the dev branch with the release and creates a new release branch automatically to manage release specific activities. 
The Maven release:prepare and release:perform stages are used to automate much of the activities relating to code release management. 
Before the release is created, please review and update the RELEASE_VERSION and the NEXT_DEV_VERSION in the project pipeline configuration. 
Note the initial values of these should be both minor versions based, with patch versions reserved for cases where either - 
•	multiple release versions need to be created due to changes as part of UAT and non-functional testing
•	issues that need to be addressed as a patch release in production. 	release:
  stage: create-release
  when: manual
  only:
      - develop
  before_script:
    # Allow push/merge/commit action on git action(s)
    # https://forum.gitlab.com/t/getting-mvn-release-to-work-with-gitlab-ci/4904/2
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$GITLAB_SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - echo "$GITLAB_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - git config --global user.email "gitlab-mule@raildeliverygroup.co.uk"
    - git config --global user.name "Auto Gitlab Pipeline"
    - git remote set-url --push origin "${CI_REPOSITORY_URL}"
  script:
    # The following script is specific to performing the Maven Release - including updating release and development versions, as well as creating the release branch as per GitFlow model.
    - git checkout -B "$CI_COMMIT_REF_NAME"
    - mvn $MAVEN_CLI_OPTS release:prepare release:perform release:clean -DscmCommentPrefix="Auto-Release [skip ci]" -DreleaseVersion=$RELEASE_VERSION -DdevelopmentVersion=$NEXT_DEV_VERSION -Darguments="-Dmule.skip.deploy=true -Dmaven.test.skip=true -DskipTests"
    # We are unable to auto create the branch using GIT here due to issues with permissions on the CI deploy keys so have opted for the API route.
    - curl --request POST --header "Private-Token:${GITLAB_PRIVATE_TOKEN}" https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/repository/branches -d branch=release-$RELEASE_VERSION -d ref=rc$RELEASE_VERSION


3.1.1 Release Pipeline
This pipeline is used as part of the release cycle, to manage changes to release branches and deploy release artifacts to the relevant environments, up to production. 
Once the release is created, if changes need to be made, ensure that 
1.	changes are made to the application POM version and this is updated sequentially to the next patch version - once checked in, this will create a new development pipeline, which in turn will publish a newly versioned artifact to the Maven artifact repository. 
2.	Once the change has been made and tested, once this needs to be deployed to UAT etc., create a tag with the patch version in GitLab which in turn will trigger a new release pipeline. 
The retrieval of release artifacts stores the artifact in the GitLab runner - this cache typically expires after 2 weeks so if a deploy pipeline is created and deployed to UAT, and subsequently after 2 weeks deployed to production, this might need a refresh of the artifact retrieval stage. alternatively, the artifact cache time can be increased for the runner by updating the pipeline configuration. 
Stage	Details	Script
get-release-artifacts	This phase retrieves release artifacts that have previously been deployed to the Maven artifact repository in GitLab. Artifacts are not built as part of this pipeline, rather the stage relies on either a release tag or a DEVELOP_VERSION to be specified as part of manually triggering the release pipeline. 	get-release-artifacts:
  stage: get-release-artifacts
  script:
    - echo "Building Mule application, and Executing Unit Tests"
    - mkdir -p deployArtifacts
    - mvn $MAVEN_CLI_OPTS dependency:get -Dartifact=uk.co.nationalrail.travelinfoservices:${CI_PROJECT_NAME}:${DEPLOY_VERSION}:jar:mule-application -DremoteRepositories=gitlab-maven::default::https://gitlab.com/api/v4/groups/rail-delivery-group-mulesoft/-/packages/maven -Ddest=deployArtifacts/${CI_PROJECT_NAME}-${DEPLOY_VERSION}-mule-application.jar -Dtransitive=false
  only:
    - tags

deploy-release-test	MANUAL - This stage deploys the release artifact to the TEST environment. 	deploy-release-test:
  stage: deploy-release
  script:
    - echo "Deploy to TEST Env"
  variables:
    APP_NAME: "${APP_NAME_PREFIX}-${CI_PROJECT_NAME}-${APP_VERSION}-test"
    CH_CLIENT_ID: $CH_TRAVELINFOSERVICES_TEST_CLIENT_ID
    CH_CLIENT_SECRET: $CH_TRAVELINFOSERVICES_TEST_CLIENT_SECRET
    CH_ENV: ${CH_ENV_TEST}
    CH_BGID: $CH_TRAVELINFOSERVICES_BG_ID
    CH_BG_NAME: $CH_TRAVELINFOSERVICES_BG_NAME
  environment:
    name: TEST
  when: manual
  only:
    - tags
  script:
    - echo "Deploying the project with maven to TEST environment"
    - mvn $MAVEN_CLI_OPTS deploy -P cloudhub -DmuleDeploy -DskipTests -Dmule.env=test $MAVEN_DEPLOY_PROPS -Dcloudhub.skipDeploymentVerification=true -Dmule.deploy.artifact=deployArtifacts/${CI_PROJECT_NAME}-${DEPLOY_VERSION}-mule-application.jar -Dmaven.install.skip=true

deploy-release-uat	MANUAL - This stage deploys the release artifact to the UAT environment. 	deploy-release-uat:
  stage: deploy-release
  script:
    - echo "Deploy to UAT Env"
  variables:
    APP_NAME: "${APP_NAME_PREFIX}-${CI_PROJECT_NAME}-${APP_VERSION}-uat"
    CH_CLIENT_ID: $CH_TRAVELINFOSERVICES_UAT_CLIENT_ID
    CH_CLIENT_SECRET: $CH_TRAVELINFOSERVICES_UAT_CLIENT_SECRET
    CH_ENV: ${CH_ENV_UAT}
    CH_BGID: $CH_TRAVELINFOSERVICES_BG_ID
    CH_BG_NAME: $CH_TRAVELINFOSERVICES_BG_NAME
  environment:
    name: UAT
  when: manual
  only:
    - tags
    # Uncomment on exception only. UAT should always be built from a release branch.
    # - develop
  script:
    - echo "Deploying the project with maven to UAT environment"
    - mvn $MAVEN_CLI_OPTS deploy -P cloudhub -DmuleDeploy -DskipTests -Dmule.env=uat $MAVEN_DEPLOY_PROPS -Dcloudhub.skipDeploymentVerification=true -Dmule.deploy.artifact=deployArtifacts/${CI_PROJECT_NAME}-${DEPLOY_VERSION}-mule-application.jar -Dcloudhub.monitoringEnabled=true -Dmaven.install.skip=true
deploy-prod	MANUAL - This stage deploys the release artifact to the PROD environment. 	deploy-prod:
  stage: deploy-prod
  variables:
    APP_NAME: "${APP_NAME_PREFIX}-${CI_PROJECT_NAME}-${APP_VERSION}"
    CH_CLIENT_ID: $CH_TRAVELINFOSERVICES_PROD_CLIENT_ID
    CH_CLIENT_SECRET: $CH_TRAVELINFOSERVICES_PROD_CLIENT_SECRET
    CH_ENV: ${CH_ENV_PROD}
    CH_BGID: $CH_TRAVELINFOSERVICES_BG_ID
    CH_BG_NAME: $CH_TRAVELINFOSERVICES_BG_NAME
  environment:
    name: PROD
  when: manual
  only:
    - tags
  script:
    - echo "Deploying the project with maven to production environment"
    - mvn $MAVEN_CLI_OPTS deploy -P cloudhub -DmuleDeploy -DskipTests -Dmule.env=prod $MAVEN_DEPLOY_PROPS -Dmule.deploy.artifact=deployArtifacts/${CI_PROJECT_NAME}-${DEPLOY_VERSION}-mule-application.jar -Dcloudhub.monitoringEnabled=true -Dmaven.install.skip=true
  artifacts:
    paths:
    - deployArtifacts/*.jar

3.2 CI/CD Variables
There are different layers of variables managed within GitLab. As part of the RDG CI/CD configuration, the following levels of variables are used: 
See https://gitlab.com/help/ci/variables/README#priority-of-environment-variables for variable priority. 
Level	Details	Additional Info
Group Level variables	Captures variables that are common across APIs - e.g. environment deployment variables, Maven repository credentials etc. 	https://gitlab.com/help/ci/variables/README#group-level-environment-variables

Project Level Variables	These are variables defined at the environment level wihtin the project. The following environments are configured to be used - note this is case sensitive. 
•	DEV
•	TEST
•	UAT
•	PROD
Details captured in these variables include
•	API_ID - API Manager Id for the API - used as part of API Autodiscovery
•	MULE_KEY - Used as part of secure property management.	https://gitlab.com/help/ci/variables/README#create-a-custom-variable-in-the-ui

Gitlab CI defined Variables	CI/CD configuration-specific variables - These include
•	DEPLOY_VERSION: $CI_COMMIT_TAG
Cloudhub environment identifiers. This is currently based on the Travel and Information Services Business Group.
•	CH_ENV_DEV: "DEV"
•	CH_ENV_TEST: "TEST"
•	CH_ENV_UAT: "UAT"
•	CH_ENV_PROD: "PROD"
•	CH_RGN: "eu-west-1"
Common Build artifact generation related variables
•	APP_NAME_PREFIX: "rdg"
•	APP_VERSION: "v1"
Cloudhub Deployment related variables
•	CH_WORKER_TYPE: "MICRO”
•	CH_WORKERS: "1"	https://gitlab.com/help/ci/variables/README#gitlab-ciyml-defined-variables

Pre-defined Variables	Used as part of CI/CD scripts	https://gitlab.com/help/ci/variables/README#predefined-environment-variables

3.3 Deploy Keys
As part of the release process, deploy keys are used to support the maven release plugin and push changes to the code repositories. As part of this, each project needs to enable a pre-defined SSH public key. Details on where this is done is available here - https://docs.gitlab.com/ee/user/project/deploy_keys/#how-to-enable-deploy-keys
For each project, this needs to be enabled - specifically under ‘Privately accessible deploy keys’ - Deploy Key is rdg-mule-ssh-deploy-key-pub.
If you need to refer to the public key, this is available here - https://gitlab.com/neildesouza_rdg/rdg-common-files/-/blob/master/ssh-deploy-key-rsa.pub
Ensure, that once the key has been enabled, you ensure that the key has write access to the repository - this can be validated by clicking in to Edit (using the button at the side of Enable in the below screenshot) 
 
