===================
Running simulations
===================

The Neuromorphic Computing Platform of the `Human Brain Project`_ contains two very different neuromorphic hardware
systems - NM-PM-1 ("physical model") and NM-MC-1 ("many core") - but has a single interface.

Jobs are written as Python scripts using the PyNN API, submitted to a queue server, then executed on one of the
neuromorphic systems. On job completion, the user may retrieve the results of the simulation/emulation.

There are several ways of interacting with the queue server. This document describes the web interface and the Python
client.


Using the web interface
=======================

TODO

Note: for using the web interface an account on the HBP unified portal will be needed when the integration of the access software has been completed. That account grants also access e.g. to the `HBP Collaboration Server`.


Using the Python client
=======================

The Python client allows scripted access to the Platform. The same client software is used both by end users for
submitting jobs to the queue, and by the hardware systems to take jobs off the queue and to post the results.


Installing the Python client
----------------------------

Download the nmpi_client package from ???? (need to put a tarball somewhere). We strongly recommend you use
virtualenv or Anaconda. The client works with Python 2.7 and Python 3.3 or newer.

::

  $ tar xzf nmpi_client-0.1.0.tar.gz
  $ cd nmpi_client-0.1.0
  $ python setup.py install


Format of a job
---------------

A job for the HBP Neuromorphic Computing Platform consists of:

  * an experiment description
  * input data
  * hardware platform configuration
  * a project name

Experiment description
``````````````````````

The experiment description must take the form of a Python script using the PyNN API. You must provide one of:

  * a single script, uploaded as part of the job submission
  * the file path of a single script located on your local machine
  * the URL of a public Git repository
  * the URL of a zip or tar.gz archive

In the latter two cases, the respository or archive must contain a top-level file named :file:`run.py`.
**The script must accept a single command-line argument**, the name of the backend (e.g. "nest", "spinnaker"). Any other parameters or
data files needed by the script should be provided as input files.

Input data
``````````

Input data are specified as a list of data items. Each data item is the URL of a data file that should be downloaded
and placed in the job working directory. If your input data files are contained within your Git repository or zip
archive, you do not need to specify them here.

Hardware platform configuration
```````````````````````````````

Here you must choose the hardware system to be used ("NM-PM1" for the Heidelberg system, "NM-MC1" for the
Manchester (SpiNNaker) system, or "sandbox" to test your script using the PyNN "mock" backend.) and specify any
specific configuration options for the hardware system you have chosen.

.. todo:: add options for running with ESS

.. todo:: list configuration options for NM-PM1 and NM-MC1 systems

Project
```````

Projects are a way of controlling who can access job results. Each job *must* be associated with a project.


Configuring the client
----------------------

Before using the Neuromorphic Computing Platform you must have an account: contact Andrew Davison for this.
To interact with the Platform, you first create a :class:`Client` object, passing it your username and password

.. code-block:: python

    import nmpi

    c = nmpi.Client(username="myusername", password="abc123")

.. todo:: allow a configuration file (".nmpirc"?) for putting username, password in


Creating a new project
----------------------

Before submitting jobs, you must create at least one project. Each project must have a unique name,
containing only letters, numbers, underscores or hyphens.
We suggest using a "namespace" approach, e.g. prefix all project names with the name of your
university or laboratory.

.. code-block:: python

    c.create_project("unic-testproject")

You can also specify a longer name, which need not be unique, and may include spaces and punctuation,
and a paragraph-length description of the project.

.. code-block:: python

    c.create_project("unic-synfire",
                     full_name="Synfire Chain Network",
                     description="Simulations of a synfire chain network")


.. todo:: what happens if a project with that name already exists?


Submitting a job
----------------

Simple example: a single file on your local machine, no input data or parameter files.

.. code-block:: python

    job_id = c.submit_job(source="/Users/andrew/dev/pyNN_0.7/examples/IF_cond_exp.py",
                          platform="NM-PM1",
                          project="unic-testproject")


A more complex example: the experiment and model description are contained in a Git repository. The input to the
network is an image file taken from the internet.

.. code-block:: python

    job_id = c.submit_job(source="https://github.com/apdavison/nmpi_test",
                          platform="NM-MC1",
                          project="unic-testproject",
                          inputs=["http://aloi.science.uva.nl/www-images/90/90.jpg"])


Monitoring job status
---------------------

.. code-block:: python

    >>> c.job_status(job_id)
    u'submitted'



Retrieving the results of a job
-------------------------------

.. code-block:: python

    >>> job = c.get_job(job_id)
    >>> from pprint import pprint
    >>> pprint(job)
    {u'experiment_description': u'https://github.com/apdavison/nmpi_test',
     u'hardware_config': u'',
     u'hardware_platform': u'NM-MC1',
     u'id': 19,
     u'input_data': [{u'id': 34,
                      u'resource_uri': u'/api/v1/dataitem/34',
                      u'url': u'http://aloi.science.uva.nl/www-images/90/90.jpg'}],
     u'log': u'',
     u'output_data': [{u'id': 35,
                      u'resource_uri': u'/api/v1/dataitem/35',
                      u'url': u'http://example.com/my_output_data.h5'}],
     u'project': u'/api/v1/project/1',
     u'resource_uri': u'/api/v1/queue/19',
     u'status': u'finished',
     u'timestamp_completion': u'2014-08-13T21:02:37.541732',
     u'timestamp_submission': u'2014-08-13T19:40:43.964541',
     u'user': u'/api/v1/user/apdavison'}



Error messages
--------------

TODO


.. _`Human Brain Project`: http://www.humanbrainproject.eu
.. _`HBP Collaboration Server`: https://collaboration.humanbrainproject.eu

