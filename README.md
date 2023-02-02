<h1>Appollo with CI/CD CircleCI</h1>

This is an example project, with the objective the show how to use the [Circle Ci](https://circleci.com) with [Appollo](https://github.com/Appollo-CLI/Appollo "The easy way to setup, build & release flutter apps for iOS on Linux, Windows and MacOS").  
<br>
![workflow](/.images/workflow.jpg "workflow")

<h2>Prerequisites</h2>

To Follow this tutorial you will need :
- [Appollo](https://github.com/Appollo-CLI/Appollo) configured on the futur runner machine
- A repository GitHub, GitLab or Bitbucket with a project [Flutter](https://docs.flutter.dev/get-started/install) directly in root repository

<h2>Configuration of repository</h2>

<h3>Creation of workflows file</h3>
To work properly you need to create this folder at the root of project 

```
mkdir .circleci/
```

<br>
Inside .circleci you will create config.yml file and insert your jobs.

<br>  
Exemple of workflow with Appollo

```YAML 
version: 2.1
workflows:
  testing:
    jobs:
      - test
      - build-ipa:
          requires:
            - test
          filters:
            branches:
              ignore: 
                - production

      - deploy:
          requires:
            - test
          filters:
            branches:
              only: production
      
jobs:
  test:
    machine: true
    resource_class: <namespace/resource_class_label>
    steps:
      - checkout
      - run: flutter test

  build-ipa:
    machine: true
    resource_class: <namespace/resource_class_label>
    steps:
      - checkout
      - run: appollo signin --email <email> --password <password>
      - run: appollo build start --build-type=ad-hoc 882
      - run: appollo signout

  deploy:
    machine: true
    resource_class: <namespace/resource_class_label>
    steps:
      - checkout
      - run: appollo signin --email <email> --password <password>
      - run: appollo build start --build-type=publication <application_key>
      - run: appollo signout

```

In this exemple we have 4 parametres:
- <*namespace/resource_class_label*> is the name space and the resource class you will create in next step.
- <*email*> is the email to connect to your account on appollo
- <*password*> is the password to connect to your account on appollo
- <*application_key*> is the key off your application. 

<br>  
:warning: **If you are using Gitlab's runners** : You should add a job to add appollo ! 

```
pip install appollo 
```

<br>
If you forgot the application's key you can use this following command : 

```
appollo app ls
```


<h2>Configuration of CircleCI</h2>

<h3>Connection to CircleCI</h3>

After connected your account (GitHub, GitLab or Bitbucket) you can select your organization and chose the project your want set up.  
![set up project](/.images/setUp.jpg "set up project")

When you had selected your project you can chose the *'Fastest'* solution and specify the branch
![set up project modal](/.images/modal.jpg "set up project modal")

<h3>Self-hosted runner</h3>

To use CircleCI there are 2 possibilities, use the  CircleCI's runner (paid solution) or use the self-hosted runners (free solution).
In this tutorial we will use the sel-hosted runner.

The first step is agree the [CircleCI Runner Terms](https://circleci.com/legal/runner-terms) in organization settings **>** Self-Hosted Runners.
When you add agree the usage terms you can access to *Self-Hosted Runners* on the left and create ressource class by following the tutorial 
![create ressource class](/.images/createressourceclass.jpg "create ressource class").

Now that you have create your self-hosted runner you can update your config.yml with the namespace and resource class label.

<h2>Usage</h2>

Now that all is configured you doens't need to do anything else.  
The previous worflow is call on each push no matter the branch because we specify *on: ['push']*.  
However the last job is call only if there are a push on *production* branch and the second job isn't call in this case.

<h3>View the actions</h3>

When you push your code you can show the workflow executed or in execution in the section *Dashboard* on the left side of CircleCI

If the unit tests has been successfully passed and the build ipa successed too you got te url to get the IPA, either to download it, or to install it if opened from an iOS device.

Finally if the push was on *production* branch the workflow will publish your app on App Store.

And that's it with this tutorial you learn how to use Appollo with Github Actions.

<h2>Documentation</h2>
We purpose 3 others exemple of solution :

- [GitHub Actions](https://github.com/NathanSepul/flutter_ci_appollo)
- [GitLab Ci](https://gitlab.com/NathanSepul/flutter_ci_appollo)
- [Bitbucket Pipelines](https://bitbucket.org/appollo-ci-cd/flutter_appollo_ci)
