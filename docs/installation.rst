User setup
==========

Install ``pulpcore``
--------------------

Follow the `installation
instructions <https://docs.pulpproject.org/en/3.0/nightly/installation/instructions.html>`__
provided with pulpcore.

Install ``pulp_rpm``
--------------------

Users should install from **either** PyPI or source or use ansible-pulp installer.

.. _ansible-installation:

Install with Ansible-pulp
*************************

With the use of the ``ansible-pulp`` installer you need to download a supportive
role before you run installer.

Only Fedora 29+ and CentOS 7 (with epel repository and python36) are supported.

.. code-block:: bash

   git clone https://github.com/pulp/ansible-pulp.git
   cd ansible-pulp
   ansible-galaxy install pulp.pulp_rpm_prerequisites -p ./roles/

Then use role you downloaded **before** ansible-pulp installer roles.

.. code-block:: yaml

   ---
   - hosts: all
     vars:
       pulp_secret_key: secret
       pulp_default_admin_password: password
       pulp_install_plugins:
         pulp-rpm:
           app_label: "rpm"
     roles:
       - pulp.pulp_rpm_prerequisites
       - pulp-database
       - pulp-workers
       - pulp-resource-manager
       - pulp-webserver
       - pulp-content
     environment:
       DJANGO_SETTINGS_MODULE: pulpcore.app.settings

Now you can run installer against your desired host following instructions
in the ansible-pulp installer.


Install ``createrepo_c`` from source
************************************

``pulp_rpm`` depends on a Python package named ``createrepo_c``, which is compiled from a C
library. Unfortunately, this package is currently only available as a Python "source distribution",
meaning that it must be compiled on your own machine. But, luckily, you won't have to do that yourself!
Simply install the build dependencies on your machine and the build process itself will happen behind
the scenes when you install the package.

If you are on Fedora, install the build dependencies with this command:

.. code-block:: bash

   sudo dnf install -y gcc make cmake bzip2-devel expat-devel file-devel glib2-devel libcurl-devel libxml2-devel python3-devel rpm-devel openssl-devel sqlite-devel xz-devel zchunk-devel zlib-devel

If you are on Ubuntu or Debian, install the build dependencies and build ``createrepo_c`` with these commands:

.. code-block:: bash

   sudo apt install -y gcc make cmake libbz2-dev libexpat1-dev libmagic-dev libglib2.0-dev libcurl4-openssl-dev libxml2-dev libpython3-dev librpm-dev libssl-dev libsqlite3-dev liblzma-dev zlib1g-dev
   pip install createrepo_c --install-option="--" --install-option="-DWITH_ZCHUNK:BOOL=OFF"


Install ``pulp_rpm`` from source
********************************

.. code-block:: bash

   sudo -u pulp -i
   source ~/pulpvenv/bin/activate
   git clone https://github.com/pulp/pulp_rpm.git
   cd pulp_rpm
   pip install -e .

Install ``pulp-rpm`` From PyPI
******************************

.. code-block:: bash

   sudo -u pulp -i
   source ~/pulpvenv/bin/activate
   pip install pulp-rpm

Make and Run Migrations
***********************

.. code-block:: bash

   django-admin makemigrations rpm
   django-admin migrate rpm

Run Services
------------

.. code-block:: bash

   django-admin runserver 24817
   gunicorn pulpcore.content:server --bind 'localhost:24816' --worker-class 'aiohttp.GunicornWebWorker' -w 2
   sudo systemctl restart pulp-resource-manager
   sudo systemctl restart pulp-worker@1
   sudo systemctl restart pulp-worker@2
