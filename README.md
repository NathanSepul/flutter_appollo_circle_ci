<h1>Appollo with CI/CD Circle CI</h1>

This is an example project, with the objective the show how to use the [Circle Ci](https://circleci.com) with [Appollo](https://github.com/Appollo-CLI/Appollo "The easy way to setup, build & release flutter apps for iOS on Linux, Windows and MacOS").  
![workflow](/.images/workflow.jpg "workflow")

<h2>Prerequisites</h2>

To Follow this tutorial you will need :
- [Appollo](https://github.com/Appollo-CLI/Appollo) configured on the futur runner machine
- A repository GitHub, GitLab or Bitbucket with a project [Flutter](https://docs.flutter.dev/get-started/install) directly in root repository

<h2>Configuration</h2>
When your connect your account (GitHub, GitLab or Bitbucket) you can select the repository you want to monitor.

To use Circle CI there are 2 possibilities, use the GitHub's runner (paid solution) or use the self-hosted runners (free solution).
In this tutorial we will use the sel-hosted runner.

<h3>Self-hosted runner</h3>

To add a self-hosted runner to your repository you should go to settings's repository **>** Actions **>**  Runners  **>** New self-hosted runner button and follow the tutorial for your os. 

<h3>Creation of actions file</h3>

To work properly you need to create these folder at the root of project 

```
mkdir -p .github/worklows/
```
<br>
Inside workflows you will create github_actions.yml file and insert your actions.

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
    resource_class: nathansepul/class_appollo
    steps:
      - checkout
      - run: echo 'test'
      # - run: flutter test

  build-ipa:
    machine: true
    resource_class: nathansepul/class_appollo
    steps:
      - run: echo 'build ipa'
      - checkout
      - run: appollo signin --email <email> --password <password>
      - run: appollo build start --build-type=ad-hoc 882
      - run: appollo signout

  deploy:
    machine: true
    resource_class: nathansepul/class_appollo
    steps:
      - checkout
      - run: echo 'Publication app'
      - run: appollo signin --email <email> --password <password>
      - run: appollo build start --build-type=publication <application_key>
      - run: appollo signout

```

In this exemple we have 4 parametres:
- <*personal_runner_label*> is the self-hosted runner label defined when it was created
- <*email*> is the email to connect to your account on appollo
- <*password*> is the password to connect to your account on appollo
- <*application_key*> is the key off your application. 

<br>
If you forgot the application's key you can use this following command : 

```
appollo app ls
```

<h2>Usage</h2>

Now that all is configured you doens't need to do anything else.  
The previous worflow is call on each push no matter the branch because we specify *on: ['push']*.  
However the last job is call only if there are a push on *production* branch and the second job isn't call in this case.

<h3>View the actions</h3>

When you push your code on Github you can show the workflow executed or in execution in the section *Actions* of the repository
![Go to action](/.images/actions_bar.jpg "Go to action")

If the unit tests has been successfully passed and the build ipa successed too you got te url to get the IPA, either to download it, or to install it if opened from an iOS device.

Finally if the push was on *production* branch the workflow will publish your app on App Store.

And that's it with this tutorial you learn how to use Appollo with Github Actions.

<h2>Documentation</h2>
We purpose 3 others exemple of solution :

- [GitLab Ci](https://gitlab.com/NathanSepul/flutter_ci_appollo)
- [Bitbucket Pipelines](https://bitbucket.org/appollo-ci-cd/flutter_appollo_ci)
- [Circle Ci](https://github.com/NathanSepul/flutter_appollo_circle_ci)
