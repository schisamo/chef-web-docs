=====================================================
Install Chef Compliance Server
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/install_chef_compliance.rst>`__

.. tag chef_automate_mark

.. image:: ../../images/a2_docs_banner.svg
   :target: https://automate.chef.io/docs

.. danger:: This documentation covers an outdated version of Chef Automate. See the `Chef Automate site <https://automate.chef.io/docs/quickstart/>`__ for current documentation. The new Chef Automate includes newer out-of-the-box compliance profiles, an improved compliance scanner with total cloud scanning functionality, better visualizations, role-based access control and many other features.

.. end_tag

.. tag EOL_compliance_server

.. warning:: The standalone Chef Compliance server is deprecated. The standalone Chef Compliance server's end-of-life date is December 31, 2018. `Chef Automate 2 <https://automate.chef.io/>`__ has all of the functionality of Chef Compliance Server and also includes newer out-of-the-box compliance profiles, an improved compliance scanner with total cloud scanning functionality, better visualizations, role-based access control and many other features not found in Chef Compliance Server.

.. end_tag

To install the Chef Compliance server in a standalone on-premises configuration, first review the prerequisites and then follow the steps below. Chef Compliance may also be installed as an Amazon Machine Images (AMI) from the AWS Marketplace.

.. note:: .. tag chef_subscriptions

          This feature is included as part of the Chef Automate license agreement and is `available via subscription <https://www.chef.io/pricing/>`_.

          .. end_tag

Prerequisites
=====================================================
The Chef Compliance server has the following prerequisites:

* An x86_64 compatible system architecture; Red Hat Enterprise Linux and CentOS may require updates prior to installation
* A resolvable hostname that is specified using a FQDN or an IP address
* A connection to Network Time Protocol (NTP) to prevent clock drift
* Optional. A local user account under which services will run, a local user account for PostgreSQL, and a group account under which services will run. See /install_server_pre.html#uids-and-gids for more information

The following prerequisites apply to every installation of the Chef Compliance server.

Platforms
-----------------------------------------------------
The following table lists the Foundational platforms for the Chef Compliance server:

.. list-table::
   :widths: 280 100 120
   :header-rows: 1

   * - Platform
     - Architecture
     - Version
   * - CentOS
     -
     - ``6.x``, ``7.x``
   * - Red Hat Enterprise Linux
     -
     - ``6.x``, ``7.x``
   * - Ubuntu
     -
     - ``12.04 LTS``, ``14.04 LTS``, ``16.04 LTS``

Untested Platforms
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Although you can audit many kinds of servers, the Chef Compliance server itself is not tested by Chef Software on the following platforms:

* Any Linux or UNIX distribution that is not listed as a supported platform
* Microsoft Windows
* 32-bit architectures

Standalone
=====================================================
The standalone installation of Chef Compliance server creates a working installation on a single server. This installation is also useful when you are installing Chef Compliance in a virtual machine, for proof-of-concept deployments, or as a part of a development or testing loop.

To install the Chef Compliance server:

#. Download the package from https://downloads.chef.io/compliance/.
#. Upload the package to the machine that will run the Chef Compliance server, and then record its location on the file system. The rest of these steps assume this location is in the ``/tmp`` directory
#. Install the Chef Compliance package on the server, using the name of the package provided by Chef. These commands require ``root`` privileges.

   For Red Hat and CentOS 6:

   .. code-block:: bash

      $ rpm -Uvh /tmp/chef-compliance-<version>.rpm

   For Ubuntu:

   .. code-block:: bash

      $ dpkg -i /tmp/chef-compliance-<version>.deb

   After a few minutes, the Chef Compliance will be installed.

#. Run the following to start all of the services:

   .. code-block:: bash

      $ chef-compliance-ctl reconfigure

   .. note:: .. tag chef_license_reconfigure_compliance

             Starting with Chef Compliance 1.1.9, the `Chef MLSA </chef_license.html>`__ must be accepted when reconfiguring the product. If the Chef MLSA has not already been accepted, the reconfigure process will prompt for a ``yes`` to accept it. Or run ``chef-compliance-ctl reconfigure --accept-license`` to automatically accept the license.

             .. end_tag

#. Access the Chef Compliance setup wizard.  Build the URL by prepending ``https://`` and appending ``/#/setup`` to the IP address or public hostname that was automatically assigned when the Amazon Machine Images (AMI) was launched.  For example, ``https://<fqdn>/#/setup``.

#. Run through the setup wizard, and then log in to the Chef Compliance web interface at ``https://<fqdn>`` using a current Chrome or Firefox browser. Internet Explorer is currently not supported.

AWS Marketplace
=====================================================
See the `AWS Marketplace documentation </aws_marketplace.html#chef-compliance-ami>`__ for instructions on deploying the Compliance server Amazon Machine Image (AMI).

Configure SSL
=====================================================
The Chef Compliance server can be configured to use SSL certificates by adding the following settings in the server configuration file located at ``/etc/chef-compliance/chef-compliance.rb``:

.. list-table::
   :widths: 200 300
   :header-rows: 1

   * - Setting
     - Description
   * - ``ssl['certificate']``
     - The SSL certificate used to verify communication over HTTPS.
   * - ``ssl['certificate_key']``
     - The certificate key used for SSL communication.

and then setting their values to define the paths to the certificate and key.

For example:

.. code-block:: ruby

   ssl['certificate']  = "/etc/pki/tls/certs/your-host.crt"
   ssl['certificate_key']  = "/etc/pki/tls/private/your-host.key"

Save the file, and then run the following command:

.. code-block:: bash

   $ sudo chef-compliance-ctl reconfigure

Configuring WinRM
=====================================================

Chef Compliance also allows you to configure SSH and WinRM through the user interface.
In the case of WinRM, you will need to perform additional steps from the command line to enable and verify the WinRM configuration of a node.

#. From CMD, start the WinRM service and load the default WinRM configuration.

   .. code-block:: none

      c:\ winrm quickconfig


#. Verify whether a listener is running, and which ports are used. The default ports are 5985 for HTTP, and 5986 for HTTPS.

   .. code-block:: none

      c:\ winrm enumerate winrm/config/listener


#. Enable basic authentication on the WinRM service. Run the following command to check whether basic authentication is allowed.

   .. code-block:: none

      c:\ winrm get winrm/config/service

#. Run the following command to enable basic authentication on the WinRM service.

   .. code-block:: none

      c:\ winrm set winrm/config/service/auth @{Basic="true"}

   .. note:: PowerShell requires a slightly modified syntax:
      ``PS:\ winrm set winrm/config/service/auth '@{Basic="true"}'``

#. Run the following command to allow transfer of unencrypted data on the WinRM service.

   .. code-block:: none

      c:\ winrm set winrm/config/service @{AllowUnencrypted="true"}

#. Enable Unencrypted client connections for the test winrm identity command to work.

   .. code-block:: none

      c:\ winrm set winrm/config/client @{AllowUnencrypted="true"}

#. Run the following command to test the connection to the WinRM service.

   .. code-block:: none

      c:\ winrm identify -r:http://NODE:5985 -auth:basic -u:USERNAME -p:PASSWORD -encoding:utf-8



Logging
=====================================================
.. tag compliance_logging

The default directory where Chef Compliance is saving logs is: ``/var/log/chef-compliance/``. It contains a directory for each service, but you can follow all the logs with this command:

.. code-block:: ruby

   sudo chef-compliance-ctl tail

This command can also be run for an individual service by specifying the name of the service in the command. For example:

.. code-block:: bash

   sudo chef-compliance-ctl tail core

.. end_tag

Troubleshooting
=====================================================
The following section describes troubleshooting tips for Chef Compliance installations.

Setup Button Does Not Work
-----------------------------------------------------
**Reason**

For security purposes the Chef Compliance setup wizard has a one hour timeout.

**Resolution**

#. Login using SSH to access the Chef Compliance instance.

   .. code-block:: bash

      $ ssh -i /path/to/ssh_key.pem <user>@<instance IP address>

#. Reconfigure Chef Compliance to restart the 60 minute setup timer.

   .. code-block:: bash

      $ chef-compliance-ctl reconfigure
