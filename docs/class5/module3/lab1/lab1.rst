Step 5 - Build your first NAP (NGINX App Protect) Docker image
##############################################################

In this lab, we will build manually our first NAP Docker image via command line.

**Follow the step below to build the Docker image:**

   #. SSH from jumphost commandline ``ssh ubuntu@10.1.1.12`` (or WebSSH and ``cd /home/ubuntu/``) to Docker App Protect + Docker repo VM
   #. Run the command ``docker build -t app-protect:nosig .`` <-- Be careful, there is a "." (dot) at the end of the command
   #. Wait until you see the message: ``Successfully tagged app-protect:nosig``

   .. note:: This command execute the Dockerfile below

   .. code-block:: bash

         # For CentOS 7:
         FROM centos:7.4.1708

         # Download certificate and key from the customer portal (https://my.f5.com)
         # and copy to the build context:
         COPY nginx-repo.crt nginx-repo.key /etc/ssl/nginx/

         # Install prerequisite packages:
         RUN yum -y install wget ca-certificates epel-release

         # Add NGINX Plus repo to Yum:
         RUN wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-7.repo

         # Install NGINX App Protect:
         RUN yum -y install app-protect \
            && yum clean all \
            && rm -rf /var/cache/yum \
            && rm -rf /etc/ssl/nginx

         # Forward request logs to Docker log collector:
         RUN ln -sf /dev/stdout /var/log/nginx/access.log \
            && ln -sf /dev/stderr /var/log/nginx/error.log

         # Copy configuration files:
         COPY nginx.conf log-default.json /etc/nginx/
         COPY entrypoint.sh  /root/

         CMD ["sh", "/root/entrypoint.sh"]



**When Docker image is built :**

   1. Check if the app-protect Docker image is available locally by running ``docker images``

   .. image:: ../pictures/lab1/docker_images.png
      :align: center

|

   2. Run a container with this image ``docker run -dit --name app-protect -p 80:80 -v /home/ubuntu/nginx.conf:/etc/nginx/nginx.conf app-protect:nosig``
   3. Check that the Docker container is running ``docker ps``

   .. image:: ../pictures/lab1/docker_run.png
      :align: center

|

   4. Check the signature package date included in this image (by default) ``docker logs app-protect -f`` wait while the docker image boots and you would see the log below.


   .. code-block:: bash
      
      2021/02/16 14:40:52 [notice] 13#13: APP_PROTECT { "event": "configuration_load_success", "software_version": "3.332.0", "user_signatures_packages":[],"attack_signatures_package":{"revision_datetime":"2019-07-16T12:21:31Z"},"completed_successfully":true,"threat_campaigns_package":{}}


**It's time to test your lab**

   #. In Edge Browser, click on the bookmark ``Arcadia NAP Docker``
   #. Navigate in the app, and try some attacks like injections or XSS - I let you find the attacks :)
   #. You will be blocked and see the default Blocking page

.. code-block:: html

   The requested URL was rejected. Please consult with your administrator.

   Your support ID is: 14609283746114744748

   [Go Back]

.. note:: Did you notice the blocking page is similar to ASM and Adv. WAF ?

**Video of this lab (force HD 1080p in the video settings)**

.. warning :: You can notice some differences between the video and the lab. When I did the video, the dockerfile was different. But the concept remains the same.

.. raw:: html

    <div style="text-align: center; margin-bottom: 2em;">
    <iframe width="1120" height="630" src="https://www.youtube.com/embed/hltSycmXFU0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    </div>