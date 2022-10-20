:sd_hide_title:

Distributed Apps
================

.. toctree::

.. topic:: Distributed Applications

   Deploy and Advertise a container running inside a managed namespace

Configure a virtual k8s cluster, deploy and scale a containerized :term:`workload` from a private :term:`registry`.

A service with one or more containers with configurable number of replicas may be deployed on a selection of Regional Edge sites or customer sites and advertised within the cluster where is it deployed, on the Internet, or on other sites using TCP or HTTP or HTTPS load balancer.

https://github.com/richardjortega/ks8-blue-green

`F5 Distributed Cloud documentation <https://docs.cloud.f5.com/docs/ves-concepts/dist-app-mgmt>`_

..  contents:: The following topics will be covered in this lab.
    :local:
    :backlinks: none
    :depth: 1

Create Virtual k8s
------------------

.. collapse:: Step 1

    .. tabs::

       .. group-tab:: UI

          #. Click :menuselection:`Applications --> Virtual K8s`, and then click :bdg-primary:`Add Virtual K8s` 

             .. image:: images/distributedappclickaddvirtualk8s.png
                :class: no-scaled-link
                :width: 100%

          #. **Name** the Virtual K8s object, then under :guilabel:`Virtual Sites` click |add-item|, select **shared/agility-k8s-vsite**, and click :bdg-primary:`Save and Exit`

             .. image:: images/distributedappclickvirtualk8ssettings2.png
                :class: no-scaled-link
                :width: 100%

             .. warning:: Virtual K8s "|create-in-progress|" may take five or more minutes to complete. :fa:`coffee`

             .. image:: images/distributedappclickvirtualk8screate-in-progress.png
                :class: no-scaled-link
                :width: 100%

       .. group-tab:: vesctl

          :file:`virtual_k8s.json`
  
          .. literalinclude:: ../manifests/virtual_k8s.json
             :language: json

          .. code-block:: console

             $ vesctl configuration create virtual_k8s -i virtual_k8s.json

Deploy Workload
^^^^^^^^^^^^^^^

.. collapse:: Step 2

    .. tabs::

       .. group-tab:: UI

          #. When Virtual K8s shows as "|ready|", click the virtual site to display details.

             .. image:: images/distributedappclickvirtualk8sready.png
                :class: no-scaled-link
                :width: 100%

          #. Click :guilabel:`Workloads` in the properties tab, and then click :bdg-primary:`Add VK8s Workload`

             .. image:: images/apps-vk8s-add-vk8s-workload.png
                :class: no-scaled-link
                :width: 100%

       .. group-tab:: vesctl

          :file:`virtual_k8s.json`

          .. literalinclude:: ../manifests/virtual_k8s.json
             :language: json

          .. code-block:: console

             $ vesctl configuration create virtual_k8s -i virtual_k8s.json

Container Service
^^^^^^^^^^^^^^^^^

#. Provide a :guilabel:`Name`, then under :guilabel:`Select Type of Workload` select **Service**, and click :guilabel:`Configure`.

   .. image:: images/vk8s-workload-create-workload-configure.png
      :class: no-scaled-link
      :width: 100%

#. In :guilabel:`Containers` section click |add-item|

   .. image:: images/6add_container.png
      :class: no-scaled-link
      :width: 100%

#. Complete the :guilabel:`Container Configuration` section by providing a **Name** and details for which image to use, then :bdg-primary:`Add Item`

   * **Name**: vk8s-container 
   * **Image Name**: coleman.azurecr.io/f5xcdemoapp
   * **Container Registry**: Private Registry
   * **Private Registry**: shared/azure-registry

   .. image:: images/7container_config.png
      :class: no-scaled-link
      :width: 100%

Associate to Virtual Site
^^^^^^^^^^^^^^^^^^^^^^^^^

#. Within the :guilabel:`Deploy Options` section, set :guilabel:`Where to Deploy the workload` to **Customer Virtual Sites**, then click **Configure**.

   .. image:: images/8deploy_options.png
      :class: no-scaled-link
      :width: 100%

#. Select **shared/agility-k8s-vsite** under :guilabel:`List of Virtual Sites to Deploy`, then :bdg-primary:`Apply`

   .. image:: images/9select_customer_site.png
      :class: no-scaled-link
      :width: 100%

Service Advertisement
^^^^^^^^^^^^^^^^^^^^^

#. Within the :guilabel:`Advertise Options` section, set :guilabel:`Options to Advertise the Workload` to **Advertise in Cluster**, then select **Configure**.

   .. image:: images/10select_advertise_options.png
      :class: no-scaled-link
      :width: 100%

#. Set :guilabel:`Select Port to Advertise` to **3000**, select :guilabel:`Application Protocol` to **HTTP**, and click :bdg-primary:`Apply`

   .. image:: images/11set_advertise_port.png
      :class: no-scaled-link
      :width: 100%

#. The :guilabel:`Deploy Options` dialogue is dismissed, and click :bdg-primary:`Apply` to complete the :guilabel:`Containers` dialogue.

   .. image:: images/apply-vk8s-workload.png
      :class: no-scaled-link
      :width: 100%

#. The :guilabel:`Containers` dialogue is now dismissed, to finalize the :guilabel:`Workload`, Click :bdg-primary:`Save and Exit`

   .. image:: images/create-workload-save-and-exit.png
      :class: no-scaled-link
      :width: 100%

#. The workload has been added. The vsite that vk8s is deployed on consists of 3 sites, so there are 3 pods in total.

   .. image:: images/12verify_3_workload_sites_pods.png
      :class: no-scaled-link
      :width: 100%

Advertise on the Internet
-------------------------

In order to view the kubernetes workload with a browser, create an HTTP-LB to advertise the site on the internet.

Create Origin Pool
^^^^^^^^^^^^^^^^^^

#. Navigate the left-side menu to :menuselection:`Manage --> Load Balancers --> Origin Pools`, then click :bdg-primary:`Add Origin Pool`.

   .. image:: images/m-origin-pool.png
      :class: no-scaled-link
      :width: 100%

#. Enter a **Name**, set the :guilabel:`Port` value to *3000*, and under :guilabel:`Origin Servers` click |add-item|

   .. image:: images/m-origin-pool-name.png
      :class: no-scaled-link
      :width: 100%

#. Complete the :guilabel:`Origin Server` section with the values below, click :bdg-primary:`Apply`, and :bdg-primary:`Save and Exit` on subsequent screen to complete the origin pool creation.

   * :guilabel:`Select Type of Origin Server`: **K8s Service Name of Origin Server on given Sites**
   * :guilabel:`Service`: **Service Name**
   * :guilabel:`Service Name`: **vk8s-workload.<namespace>**
      * .. attention::
           Supply the configured workload name from previous steps along with the student namespace.
   * :guilabel:`Site or Virtual Site`: **Virtual Site**
   * :guilabel:`Virtual Site`: **shared/agility-k8s-vsite**
   * :guilabel:`Select Network on the site`: **vK8s Networks on Site**

   .. image:: images/m3-add-origin-server.png
      :class: no-scaled-link
      :width: 100%

Create HTTP Load-Balancer
^^^^^^^^^^^^^^^^^^^^^^^^^

#. Navigate the left-side menu to :menuselection:`Manage --> Load Balancers --> HTTP Load Balancers`, then click :bdg-primary:`Add HTTP Load Balancer`.

   .. image:: images/m-add-http.png
      :class: no-scaled-link
      :width: 100%

#. Add the following values, and click :bdg-primary:`Save and Exit`

   * :guilabel:`Name`: app-http-lb
   * :guilabel:`Domains and LB Type`: Use the assigned {namespace}.lab-app.f5demos.com
   * :guilabel:`Load Balancer Type`: **HTTP**
   * :guilabel:`Automatically Manage DNS Records`: Make sure this is checked
   * :guilabel:`Origins`: Click |add-item|, and select **app-origin-pool**

   .. image:: images/m-http-name.png
      :class: no-scaled-link
      :width: 100%

#. It may take a minute :fa:`coffee` for the :guilabel:`DNS Info` to display **VIRTUAL_HOST_READY**

   .. image:: images/m-http-status.png
      :class: no-scaled-link
      :width: 100%

#. Open a browser tab and navigate to the configured DNS name `http://busy-parrot.lab-app.f5demos.com/`. Refresh your browser a few times and notice what happens to the country name.

   .. image:: images/m-http-page.png
      :class: no-scaled-link
      :width: 100%

Scale Deployment
----------------

Modify Virtual K8s Deployment to Scale Replicas.

Edit JSON
^^^^^^^^^

#. Navigate the left-side menu to :menuselection:`Applications --> Virtual K8s --> virtual-k8s`, click :guilabel:`Deployments`, :guilabel:`Actions`, |three-dots| then click :guilabel:`Edit`.

   .. image:: images/14edit_deployment.png
      :class: no-scaled-link
      :width: 100%

#. Enable |edit-mode|, and expand the ``spec`` section by clicking |out-arrows|

   .. image:: images/15modify_deployment_spec.png
      :class: no-scaled-link
      :width: 100%

#. Change **replicas: 1** to **replicas: 3** and click :bdg-primary:`Save`

   .. image:: images/set-three-replicas-save.png
      :class: no-scaled-link
      :width: 100%

#. After a few moments, the number of **Running Pods** will increase to 9.

   .. image:: images/16review_scaled_deployment.png
      :class: no-scaled-link
      :width: 100%

View Results with kubectl
-------------------------

Install kubectl
^^^^^^^^^^^^^^^

Insert instructions to install kubectl

Download kubeconfig
^^^^^^^^^^^^^^^^^^^^

#. Navigate to :menuselection:`Applications --> Virtual K8s`, click |three-dots|, and then click |download-kubeconfig-button|

   .. image:: images/distributedappclickvirtualk8kubeconfig.png
      :class: no-scaled-link
      :width: 100%

#. `Follow the kubernetes.io guide to install the kubeconfig <https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/>`_

kubectl commands
^^^^^^^^^^^^^^^^

Run the following commands and view the vk8s configuration.

**View Nodes**

.. code-block:: console

   $ kubectl get nodes
   NAME                                                 STATUS   ROLES        AGE   VERSION
   agility-vpc-site-one-agility-vpc-site-one-1w2h       Ready    ves-master   28s   v1.21.7-vesdev
   agility-vpc-site-three-agility-vpc-site-three-xn79   Ready    ves-master   32s   v1.21.7-vesdev
   agility-vpc-site-two-agility-vpc-site-two-j735       Ready    ves-master   33s   v1.21.7-vesdev
   
**View pods**

.. code-block:: console
 
   $ kubectl get pods
   NAME                             READY   STATUS    RESTARTS   AGE
   vk8s-workload-574ffc5cdd-sb5bm   2/2     Running   0          2m40s
   vk8s-workload-64f8f87976-kh8zz   2/2     Running   0          2m37s
   vk8s-workload-67b54bd74b-bqdx8   2/2     Running   0          2m41s
   $ kubectl describe pod <podname>
   
**View deployment**

.. code-block:: console

   $ kubectl get deployment vk8s-workload
   NAME            READY   UP-TO-DATE   AVAILABLE   AGE
   vk8s-workload   3/1     3            3           4m43s

**View service**

.. code-block:: console

   $ kubectl get svc vk8s-workload
   NAME            TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)    AGE
   vk8s-workload   ClusterIP   192.168.167.169   <none>        3000/TCP   8m33s

**View all resources in the namespace**

.. code-block:: console

   $ kubectl get all
   NAME                                 READY   STATUS    RESTARTS   AGE
   pod/vk8s-workload-574ffc5cdd-sb5bm   2/2     Running   0          9m18s
   pod/vk8s-workload-64f8f87976-kh8zz   2/2     Running   0          9m15s
   pod/vk8s-workload-67b54bd74b-bqdx8   2/2     Running   0          9m19s

   NAME                    TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)    AGE
   service/vk8s-workload   ClusterIP   192.168.167.169   <none>        3000/TCP   9m21s

   NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/vk8s-workload   3/1     3            3           9m22s

   NAME                                       DESIRED   CURRENT   READY   AGE
   replicaset.apps/vk8s-workload-574ffc5cdd   1         1         1       9m22s
   replicaset.apps/vk8s-workload-64f8f87976   1         1         1       9m22s
   replicaset.apps/vk8s-workload-67b54bd74b   1         1         1       9m22s

**View the output of the deployment in yaml format**

.. code-block:: console

   $ kubectl get deployment -o yaml


