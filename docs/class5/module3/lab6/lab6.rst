Step 10 - Deploy NAP with a CI/CD toolchain
###########################################

In this lab, we will deploy deploy NAP with a CI/CD pipeline. NAP is tied to the app, so when DevOps commits a new app (or a new version), the CI/CD pipeline has to deploy a new NAP component in front. In order to avoid repeating what we did previously, we will use a Signature package update as a trigger.

.. note:: When a new signature package is available, the CI/CD pipeline will build a new version of the Docker image and run it in front of Arcadia Application

**This is the workflow we will run (the steps to run are later in this page)**

    #. Check if a new Signature Package is available
    #. Simulate a Commit in GitLab (Goal is to simulate a full automated process checking Signature Package date every day)
    #. This commit triggers a webhook in Gitlab CI
    #. Gitlab CI runs the pipeline
    
        #. Build a new Docker NAP image with a new tag ``date of the signature package``
        #. Destroy the previous running NAP container
        #. Run a new NAP container with this new Signature Package

.. note:: Goal of this lab is not to learn how to do it, but understand how I did it.

**Check the Gitlab CI file**

.. code-block:: yaml

    stages:
        - Build_image
        - Push_image
        - Run_docker

    before_script:
        - docker info

    Build_image:
        stage: Build_image
        script:
            - TAG=`yum info app-protect-attack-signatures | grep Version | cut -d':' -f2`
            - echo $TAG
            - docker build -t 10.1.20.7:5000/app-protect:`echo $TAG` .
            - echo export TAG=`echo $TAG` > $CI_PROJECT_DIR/variables
        artifacts:
            paths:
            - variables

    Push_image:
        stage: Push_image
        script:
            - source $CI_PROJECT_DIR/variables
            - echo $TAG
            - docker push 10.1.20.7:5000/app-protect:`echo $TAG`

    Run_docker:
        stage: Run_docker
        script:
            - source $CI_PROJECT_DIR/variables
            - echo $TAG
            - ansible-playbook -i hosts playbook.yaml --extra-var dockertag=`echo $TAG`



.. note:: The challenge here was to retrieve the date of the package and tag the image with this date in order to have one image per signature package date. This is useful if you need to roll back to a previous version of the signatures.

**Simulate an automated task detecting a new Signature Package has been release by F5**

Steps:

    #.  RDP to the Jumphost and open ``Edge Browser``
    #.  Open ``Gitlab``

        #. If Gitlab is not available (502 error), restart the GitLab Docker container. SSH to the GitLab VM and run ``docker restart gitlab`` 
    #.  In GitLab, open ``NGINX App Protect / nap-docker-signature`` project

        .. image:: ../pictures/lab6/gitlab_project_updated.png
           :align: center
           :scale: 50%

    #.  SSH from Jumpbox commandline ``ssh ubuntu@10.1.1.8`` (or WebSSH) to ``CICD server (runner, Terraform, Ansible)``

        #. Run this command in order to determine the latest Signature Package date: ``yum info app-protect-attack-signatures``
        #. You may notice the version date. In my case, when I write this lab ``2020.06.30`` was the most recent version of the signatures package. We will use this date as a Docker tag, but this will be done automatically by the CI/CD pipeline.

        .. image:: ../pictures/lab6/yum-date.png
           :align: center
           :scale: 50%




**Trigger the CI/CD pipeline**

Steps :

    #. In GitLab, click on ``Repository`` and ``Tags`` in the left menu
    #. Create a new tag and give it a name like ``Sig-<version date>`` where ideally ``<version_date>`` should be replaced by the package version information found in the result of the ``yum info`` step above. But it does not matter, you can put anything you want in this tag.
    #. Click ``Create tag``
    #. At this moment, the ``Gitlab CI`` pipeline starts
    #. In Gitlab, in the ``signature-update`` repository, click ``CI / CD`` > ``Pipelines``

       .. image:: ../pictures/lab6/github_cicd.png
          :align: center   

    #. Enter into the pipeline by clicking on the ``running or passed`` button. And wait for the pipeline to finish. You can click on every job/stage to check the steps

       .. image:: ../pictures/lab6/github_pipeline.png
          :align: center 
    
    #. Check if the new image created and pushed by the pipeline is available in the Docker Registry.
        #. In ``Edge Browser`` open bookmark ``Docker Registry UI`` which is inside the ``extras`` bookmark folder
        #. Click on ``App Protect`` Repository
        #. You can see your new image with the tag ``2020.06.30`` - or any other tag based on the latest package date.

        .. image:: ../pictures/lab6/registry-ui.png
           :align: center 

    #. Connect in SSH from Jumpbox commandline ``ssh ubuntu@10.1.1.12`` to the Docker App Protect + Docker repo VM, and check the signature package date running ``docker logs app-protect``
    
    .. code-block:: bash
       
       2021/02/24 13:59:24 [notice] 13#13: APP_PROTECT { "event": "configuration_load_success", "software_version": "3.332.0", "user_signatures_packages":[],"attack_signatures_package":{"revision_datetime":"2021-01-28T20:04:14Z","version":"2021.01.28"},"completed_successfully":true,"threat_campaigns_package":{}}


.. note:: Congratulations, you ran a CI/CD pipeline with a GitLab CI.

