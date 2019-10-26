<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Welcome file</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#multibranch-pipeline-for-application-deployment">Multibranch Pipeline for application deployment</a>
<ul>
<li><a href="#table-of-contents">Table of Contents</a></li>
<li><a href="#configuration">Configuration</a></li>
<li><a href="#process-flow">Process Flow</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="multibranch-pipeline-for-application-deployment">Multibranch Pipeline for application deployment</h1>
<h2 id="table-of-contents">Table of Contents</h2>
<p>[TOC]</p>
<hr>
<h3 id="introduction-to-multi-branch-pipeline">Introduction to Multi Branch pipeline</h3>
<p>The Multibranch Pipeline project type enables you to implement different Jenkinsfiles for different branches of the same project. In a Multibranch Pipeline project, Jenkins automatically discovers, manages and executes Pipelines for branches which contain a Jenkinsfile in source control.</p>
<p>A jenkins Pipeline is a combination of plugins that support the integration and implementation of continuous delivery pipelines using Jenkins. A pipeline has an extensible automation server for creating simple or complex delivery <strong>pipelines as code</strong> via pipeline DSL (Domain-specific Language).This Jenkins pipeline is developed on the basis on the process flow defined in the [<strong>CI-CD Process Commit Flow</strong>] (<a href="https://www.lucidchart.com/invitations/accept/4d4414cb-2069-471d-ac4e-5679f456884c">https://www.lucidchart.com/invitations/accept/4d4414cb-2069-471d-ac4e-5679f456884c</a>) and [<strong>CI-CD Process Pull Request Flow</strong>] (<a href="https://www.lucidchart.com/invitations/accept/a08bdf0c-177d-49c1-813a-ff557944f1ab">https://www.lucidchart.com/invitations/accept/a08bdf0c-177d-49c1-813a-ff557944f1ab</a>) documents. This pipeline is stored alongside the application code in the form of “<strong>Jenkinsfile</strong>”. The pipeline is designed to be generic so that the same pipeline can be used to deploy multiple applications inside the Merchant e-Solution with/without minute changes. Any failure in the current stage of the jenkins pipeline will be marked as a complete failure of the entire pipeline and the process gets terminated at that moment itself.</p>
<p><strong>1. Declarative: Checkout SCM</strong> - This stage will determine the branch from which the commmit /PR has been created and clone the source code from Bitbuket for the project and is proivded by the Pipeline configuration of Jenkins.</p>
<p><strong>2. Preparation</strong> - It will detemine all the variables values that will be used in the current build process for a project.  It will ouput all the consolidated values of the variables in Jenkins<br>
<strong>Console Output</strong> for debugging and tracking purposes.</p>
<p><strong>3. Build</strong> - This is the stage to compile that application and run all unit tests associated with the application.  If the compilation or unit tests fails, all subsequent stages are stopped and email notification will send to the developer to troubleshoot the errors in the code.</p>
<p><strong>4. Static Analysis &amp; Code Coverage</strong> - This stage uses a selected SonarQube scanner with a pre-defined quality profile in a specified SonarQube server to scan the code and associated libraies to arrive the static analysis result.  It will then use the quality gate defined in the SonarQube server to evaluate if the project passes the quality control (Code coverage &gt; 80%).  If the code quality fails the quality gate,  all subsequent stages are stopped and email notification will send to the developer to troubleshoot  the errors in the code.</p>
<p><strong>5. Repository Tagging &amp; Artifact Building &amp; Artifact Publishing</strong> - This stage will only gets once the pull request is raised by the developer from feature branch to develop branch and in develop branch. This stage will docerize the java application and then upload it to the docker hub / Docker registry. If any errors encoundered in this stage , all subsequent stages will be stopped and an email notification to the reviewer to reject the pull request.</p>
<p><strong>6. Team Specific Environment Artifact Deployment</strong> - This stage will only gets once the pull request is raised by the developer from feature branch to develop branch and in develop branch. As the name indicates, this stage downloads the docker image of the application from the docker hub / docker registry and is then deployed to the kubernetes environment. Any failure in the current stage will stop all the subsequent stages and email notification is send to the reviewer to reject the pull request.</p>
<p><strong>7. Sanity &amp; Smoke Test Execution</strong> - This stage will only gets once the pull request is raised by the developer from feature branch to develop branch and in develop branch. This stage will execute the Sanity and Smoke tests and provide the results. Any failure in this stage will stop the subsequent stages and the email notification is send to the reviewer to reject the pull request.</p>
<p><strong>8. Manual &amp; Regression Tests Execution</strong> - This stage will only gets once the pull request is raised by the developer from feature branch to develop branch and in develop branch. This stage will execute the Manual and Regression tests and provide the results. Any failures in the stage will stop the subsequent stages and the email notification is send to the reviewer to reject the pull request.</p>
<p><strong>9. Declarative: Post Actions</strong> - This stage will send email notification to corresponding team with build status.</p>
<h3 id="pre-requisites">Pre-requisites</h3>
<p>This pipeline uses several Jenkins plugins. The required Jenkins plugins are,</p>
<ul>
<li>Pipeline - It includes a suite of plugins to allow orchestration of complex automation.</li>
<li>Multibranch Pipeline - This plugin allows the use of multibranch pipeline in the jenkins.</li>
<li>SonnarQube Scanner - It allows an easy integration of SonarQube, the open source platform for Continuous Inspection of code quality.</li>
<li>Email Extension Plugin - It is a replacement for Jenkins’ email publisher.  It allows to configure every aspect of email notifications: when an email is sent, who should receive it and what the email says.</li>
<li>BitBucket Plugin - It integrates with BitBucket to trigger build with pull request.</li>
<li>Artifactory Plugin - It allows deploying maven artifacts and build info to Artifactory.</li>
<li>Pipeline Utility Steps - It allows utility steps for pipeline jobs.</li>
<li>Pipeline Stage View Plugin -  It includes an extended visualization of Pipeline build history on the index page of a flow project, under Stage View.</li>
<li>Credentials Plugin - This plugin allows to store the credentials in the Jenkins</li>
<li>Blue Ocean Plugin - This plugin provides visual enhancements to the jenkins pipeline flow.</li>
<li>SSH Agent Plugin - It allows configurer to provide SH credentials to builds via a ssh-agent.</li>
<li>Maven Integration - It integrates with Pipeline, configures maven environment to use within a pipeline job by calling sh mvn or bat mvn. The selected maven installation will be configured and prepended to the path.</li>
</ul>
<h2 id="configuration">Configuration</h2>
<p>To incorporate the pipeline into MeS project, there are two major tasks to make it works.  Following sub sections outline the steps corresponding to each of these two major tasks</p>
<h3 id="bitbucket-configuration">Bitbucket configuration</h3>
<p>1.Git clone the bitbucket repository for the application using the command git clone    <a href="https://bitbucket.org/merchante-solutions/">https://bitbucket.org/merchante-solutions/</a>&lt;application_repository&gt;.git    to your local machine.<br>
2.Copy the Jenkinsfile_Commit and Jenkinsfile_PR to the folder<br>
3. Push the entire folder bach to the repository.<br>
4. Now we need to create the webhooks for commit and pull request. For that go to the settings -&gt; webhooks<br>
5. Create a new webhook for the commit pipeline. For that click on <em><strong>Add Webhook</strong></em> and type a name for the hook.<br>
6. In the URL , enter <em><strong>JENKINS_URL/multibranch-webhook-trigger/invoke?token=[Trigger token]</strong></em> (Trigger token should be unique for each jenkins job).<br>
7. In the trigger section, select all those related to the <em><strong>Repository</strong></em> section. Mark the webhook status as <em><strong>Active</strong></em> and save.<br>
8. Repeat the steps for the Pull request webhook too. In the trigger section , select those related to <em><strong>Pull Request</strong></em> and provide a    new <em><strong>Trigger token</strong></em> for the URL. Mark the status as <em><strong>Active</strong></em>    and save the webhook.</p>
<h3 id="jenkins-configuration">Jenkins Configuration</h3>
<ol>
<li>Open the jenkins console , click on <em><strong>New Item</strong></em> and select the <em><strong>MultiBranch Pipeline</strong></em>. Provide the name of the pipeline as “Commit” pipeline. (This pipeline gets executed when the developer commits the code to the bitbucket repository)</li>
<li>Provide the bitbucket repository and the credentials in the <em><strong>Branch Sources</strong></em> section. Select the <em><strong>Behaviour</strong></em> as <em><strong>All Branches</strong></em></li>
<li>Provide the <em><strong>Build Configuration</strong></em> as <em><strong>Jenkinsfile</strong></em> and the provide the <em><strong>Jenkinsfile_Commit</strong></em> in the <em><strong>Script Path</strong></em>.</li>
<li>Under the <em><strong>Scan Multibranch Pipeline Triggers</strong></em> , Check the <em><strong>Scan by webhook</strong></em> and provide the unique token provided during creating the webhook for Commit and then save the pipeline.</li>
<li>The Jenkins pipeline scans for <em><strong>Jenkinsfile_Commit</strong></em> in all the branches of the bitbucket repository and a test build will be executed. Later try commiting something to the bitbucket repository and you can see this jenkin pipeline gets automatically triggered.</li>
<li>Create another jenkins multibranch pipeline for the <em><strong>Pull Request</strong></em>.Provide the <em><strong>Build Configuration</strong></em> as <em><strong>Jenkinsfile</strong></em> and the provide the <em><strong>Jenkinsfile_PR</strong></em> in the <em><strong>Script Path</strong></em></li>
<li>Under the <em><strong>Scan Multibranch Pipeline Triggers</strong></em> , Check the <em><strong>Scan by webhook</strong></em> and provide the unique token provided during creating the webhook for Pull request and then save the pipeline.</li>
<li>The Jenkins pipeline scans for <em><strong>Jenkinsfile_PR</strong></em> in all the branches of the bitbucket repository and a test build will be executed. Later try creating a pull requst in the bitbucket repository and you can see this jenkin pipeline gets automatically triggered.</li>
</ol>
<h2 id="process-flow">Process Flow</h2>
<h3 id="ci-cd-process-commit-flow"><strong>CI-CD Process Commit Flow</strong></h3>
<p>When the developer pushes the code to the bit bucket repository , the webhook triggers the Jenkins multibranch pipeline. The branch name from which the commit happened will be automatically determined from the webhook and build stage is executed where the build happens using the maven plugin. Once the build stage is success ,the Static Analysis &amp; Code Coverage stage gets executed to determine the code coverage. All the codes with code coverage less than 80 % is considered a failure. If the Static Analysis &amp; Code Coverage is success , then the component tests are executed. If there is any failure in any of the 4 stages , then an email notification is send to the developer to recheck the code that is commited to the bit bucket repository. If all the 4 stages are executed successfully , then another email notification is send to the developer that the code is now ready for the pull request.</p>
<h3 id="ci-cd-process-pull-request-flow"><strong>CI-CD Process Pull Request Flow</strong></h3>
<p>When the developer raises the pull request to merge the feature branch with the develop branch , another webhook will trigger the multibranch jenkins pipeline.The branch name will be automatically determined from the webhook for the build process to happen. The build stage will then execute the build process of the code uploaded by the developer using the maven plugin. Post successful buld , the Static Analysis &amp; Code Coverage stage will be executed and code coverage is determined. This stage will be succesful only when the code coverage criteria is above 80% . Then the Component Test stage will be executed to test the working of the code in virtualized environment. Post success of this stage , the Repository Tagging &amp; Artifact Building &amp; Artifact Publishing stage will be executed , where the code gets dockerized and is uploaded to the Docker registry/Docker hub. After this stage , Team Specific Environment Artifact Deployment stage gets executed where the application gets finally deployed to the kubernetes cluster. Post successful deployment, Sanity &amp; Smoke Test Execution stage gets executed where the integration and functional tests get executed.Post succesful stage , Manual &amp; Regression Tests Execution stage gets executed where the manual and regression tests get executed. If there is any failure in any of the stage, the entire pipeline will be marked as a failure and email notification is send to the reviewer to reject the pull request. If all the stages are executed successfully, another email communication is send to the reviewer to approve the pull request and then the entire pipeline will execute once again in the develop branch.</p>

    </div>
  </div>
</body>

</html>
