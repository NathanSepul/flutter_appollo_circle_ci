===========================
Appollo with CI/CD CircleCI
===========================

This is an example project, with the objective to show how to use CircleCI with `Appollo <https://github.com/Appollo-CLI/Appollo>`_ for releasing iOS apps with a CI solution.  

.. image:: /.images/workflow.jpg 
    :align: center
    :width: 70%

-------------
Prerequisites
-------------

To Follow this tutorial you will need :

* An appollo account linked to your Apple Developer Account. `Learn how to setup Appollo <https://appollo.readthedocs.io/en/master/tutorial/2_configure_app_store_connect.html>`_.
* A `Flutter <https://docs.flutter.dev/get-started/install>`_ project located at the root of your git repository.

---------------------------
Configuration of repository
---------------------------

^^^^^^^^^^^^^^^^^^^^^^^^^^
Creation of workflows file
^^^^^^^^^^^^^^^^^^^^^^^^^^

To work properly you need to create this folder at the root of your project 

.. code-block::

    mkdir .circleci/

Inside .circleci you will create a config.yml file. This is where we will add the actions.
Here is an example :

.. code-block:: yaml

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
        docker:
          - image: cirrusci/flutter
        steps:
          - checkout
          - run: flutter test

      build-ipa:
        docker:
          - image: cimg/python:3.11.2
        steps:
          - checkout
          - run: pip3 install  --no-input appollo
          - run: appollo signin --email '$EMAIL_APPOLO' --password '$PASSWORD_APPOLLO'
          - run: appollo build start --build-type=ad-hoc '$APPLICATION_KEY'
          - run: appollo signout

      deploy:
        docker:
          - image: cimg/python:3.11.2
        steps:
          - checkout
          - run: pip3 install --no-input appollo 
          - run: appollo signin --email '$EMAIL_APPOLO' --password '$PASSWORD_APPOLLO'
          - run: appollo build start --build-type=publication '$APPLICATION_KEY'
          - run: appollo signout



In this exemple we have 3 variables:

* *EMAIL_APPOLO* is the email to connect to your account on appollo
* *PASSWORD_APPOLLO* is the password to connect to your account on appollo
* *APPLICATION_KEY* is the key off your application. 

Here is the `documentation <https://circleci.com/docs/env-vars/>`_ for variables on CircleCI

|

.. note:: If you forgot the application's Appollo key you can use this following command :  `appollo app ls`


-------------------------
Configuration of CircleCI
-------------------------

^^^^^^^^^^^^^^^^^^^^^^
Connection to CircleCI
^^^^^^^^^^^^^^^^^^^^^^

After connected your account (GitHub, GitLab or Bitbucket) you can select your organization and chose the project your want set up.  

.. image:: /.images/setUp.jpg
    :align: center
    :width: 50%

When you had selected your project you can chose the *'Fastest'* solution and specify the branch

.. image:: /.images/modal.jpg
    :align: center
    :width: 50%

^^^^^^^^^^^^^^^^^^
Self-hosted runner
^^^^^^^^^^^^^^^^^^

To use CircleCI there are 2 possibilities, use the  CircleCI's runner (paid solution) or use the self-hosted runners (free solution).
In this tutorial we will use the sel-hosted runner.

The first step is agree the `CircleCI Runner Terms <https://circleci.com/legal/runner-terms>`_ in organization settings **>** Self-Hosted Runners.
When you add agree the usage terms you can access to *Self-Hosted Runners* on the left and create ressource class by following the tutorial 

.. image:: /.images/createressourceclass.jpg
    :align: center
    :width: 50%

|

Now that you have create your self-hosted runner you can update your workflows with runner informations with this type of structure for each job :

.. code-block:: yaml

    test:
      machine: true
      resource_class: namespace/resource_class_label
      steps:
        - checkout
        -  run: flutter test'
        
-----
Usage
-----

Now that all is configured you don't need to do anything else. The previously made worflow is called on each push no matter the branch. 
However the last job is call only if there are a push on ``production`` branch and the second job isn't call in this case.

^^^^^^^^^^^^^^^^
View the actions
^^^^^^^^^^^^^^^^

When you push your code you can show the workflow executed or in execution in the section ``Dashboard`` on the left side of CircleCI

If the unit tests have been successfully passed and the build ipa succeeded you get back the url to the IPA, either to download it, or to install it if opened (in safari) from an iOS device.

Finally if the push was on the ``production`` branch the workflow will publish your app on the App Store directly. You can then either test your application through testflight or submit the latest version to Apple.

And that's it with this tutorial you had learn how to use Appollo with Circle .

-------------
Documentation
-------------
We propose 3 others examples of solution with other CI tools:

* `GitHub Actions <https://github.com/NathanSepul/flutter_ci_appollo>`_
* `GitLab Ci <https://gitlab.com/NathanSepul/flutter_ci_appollo>`_
* `Bitbucket Pipelines <https://bitbucket.org/appollo-ci-cd/flutter_appollo_ci>`_
