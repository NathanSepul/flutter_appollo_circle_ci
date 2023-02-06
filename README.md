<h1>Appollo with CI/CD CircleCI</h1>

This is an example project, with the objective to show how to use CircleCI with [Appollo](https://github.com/Appollo-CLI/Appollo "The easy way to setup, build & release flutter apps for iOS on Linux, Windows and MacOS")
for releasing iOS apps with a CI solution.  
<br>
![workflow](/.images/workflow.jpg "workflow")

<h2>Prerequisites</h2>

To Follow this tutorial you will need :
- An appollo account linked to your Apple Developer Account. [Learn how to setup Appollo.](https://appollo.readthedocs.io/en/master/tutorial/2_configure_app_store_connect.html)
- A [Flutter](https://docs.flutter.dev/get-started/install) project located at the root of your git repository.

<h2>Configuration of repository</h2>

<h3>Creation of workflows file</h3> 

To work properly you need to create this folder at the root of your project 

```
mkdir .circleci/
```
<br>

Inside .circleci you will create a config.yml file.
This is where we will add the actions.
Here is an example :

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
      - run: pip3 install -y Appollo
      - run: appollo signin --email <email> --password <password>
      - run: appollo build start --build-type=ad-hoc 882
      - run: appollo signout

  deploy:
    machine: true
    resource_class: <namespace/resource_class_label>
    steps:
      - checkout
      - run: pip3 install -y Appollo
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

> **_NOTE:_** If you forgot the application's Appollo key you can use this following command :  `appollo app ls`

<br>

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

Now that all is configured you don't need to do anything else. The previously made worflow is called on each push no matter the branch. 
However the last job is call only if there are a push on `production` branch and the second job isn't call in this case.

<h3>View the actions</h3>

When you push your code you can show the workflow executed or in execution in the section `Dashboard` on the left side of CircleCI

If the unit tests have been successfully passed and the build ipa succeeded you get back the url to the IPA, either to download it, or to install it if opened (in safari) from an iOS device.

Finally if the push was on the `production` branch the workflow will publish your app on the App Store directly. You can then either test your application through testflight or submit the latest version to Apple.

And that's it with this tutorial you had learn how to use Appollo with Circle .

<h2>Documentation</h2>
We purpose 3 others exemple of solution :

- [GitHub Actions](https://github.com/NathanSepul/flutter_ci_appollo)
- [GitLab Ci](https://gitlab.com/NathanSepul/flutter_ci_appollo)
- [Bitbucket Pipelines](https://bitbucket.org/appollo-ci-cd/flutter_appollo_ci)
