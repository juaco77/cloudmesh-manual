Cloudmesh Database
==================

Cloudmesh stores its status in a database so that you can easily
remember which services you used where and have an accurate account of
them. We use as a database mongoDB to store this information. To use
cloudmesh you simply need to create and start the database service.

First you need to cahnge the password for mongo in the ``cloudmesh4.yaml`` file
Now you can create an empty database with

.. code:: bash

    cms admin mongo create

Next, you need to start it with the command

.. code:: bash

    cms admin mongo start

Now you can interact with it to find out the status, the stats, and the
database listing with the commands

.. code:: bash

    cms admin mongo status
    cms admin mongo stats
    cms admin mongo list

To stop it from running use the command

.. code:: bash

    cms admin mongo stop


To backup and load from the backup you can use:

.. todo:: the dmin backup needs to be tested.

:: code:: bash

    cms admin mongo save [--file=FILE]
    cms admin mongo load [--file=FILE]


The database will be started based-on the information as specified in
``~/.cloudmesh/cloudmesh4.yaml``. An example is

::

        mongo:
          MONGO_AUTOINSTALL: True
          MONGO_BREWINSTALL: False
          LOCAL: ~/local
          MONGO_HOME: ~/local/mongo
          MONGO_PATH: ~/.cloudmesh/mongodb
          MONGO_LOG: ~/.cloudmesh/mongodb/log
          MONGO_DBNAME: 'cloudmesh'
          MONGO_HOST: '127.0.0.1'
          MONGO_PORT: '27017'
          MONGO_USERNAME: 'admin'
          MONGO_PASSWORD: "my secret password"
          MONGO_DOWNLOAD:
            darwin: https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.4.tgz
            linux: https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.4.tgz
            win32: https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-4.0.4-signed.msi
            redhat: https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/RPMS/mongodb-org-server-4.0.4-1.el7.x86_64.rpm

We also provide a convenient mongoDB install script that downloads the version
defined in the yaml file and installs it in the system with the command.
In case of windows platform, you will have to set the PATH variable
manually after install

.. code:: bash

    cms admin mongo install

Database Decorator
------------------

Cloudmesh comes with a very convenient mechanism to integrate specific
formed dicts into MongoDB. All you have to do is to create a list of
dictionaries with a function, that returns this dictionary and use a
decorator in the function to update the information into the database.

The database decorator automatically replaces an entry in the database
with the dictionary returned by a function.

It is added to a specific MongoDB collection. The location is determined from
the values in the dictionary. The dict must contain a dict named ``cm`` that
containes thre attributes ``kind``, ``cloud``, ``name``:


::

    "cm" : {
        "kind" : "flavor",
        "cloud" : "chameleon",
        "name" : "m1.medium",
    },

WHen such a dict is uploaded with our database decorator that we explain
later, information such as created, updated, and other attributes are added.
Some of the information for these attributes is taken from the ``cloudmesh4
.yaml`` file, while others such as modified, will be updated dynamically.

::

    "cm" : {
        "name" : "m1.medium",
        "created" : "2019-03-25 07:45:46.905623",
        "modified" : "2019-03-25 07:45:46.905623",
        "cloud" : "chameleon",
        "kind" : "flavor",
        "driver" : "openstack",
        "collection" : "chameleon-flavor"
    },

Using this information the object can easily be found in the database by
name, type or cloud or a combination thereof.

.. note:: in a future version the ``cloud`` parameter will be renamed to
          ``service``

The name of the collection is determined from cloud and kind:

``{cloud}-{kind}``

In addition each entry in the collection has a ``name`` that must be
unique in that collection.


In most examples it is best to separate the updload from the native class the
interacts with the service. This way we provide always two classes. One
interacting with the service and the otherone that acts alike for all
provider, while the name of the provider decides which native provider is
used to interactt with the coud services.

Example:

``cloudmesh.example.foo`` contains:

::

    class Provider(object)

        def entries(self):
            return {
             "cm" : {
                "kind" : "flavor",
                "driver" : "openstack",
                "cloud" : "foo",
                "created" : "2019-04-01 15:59:39.815993",
                "name" : "m1.xxxlarge",
                "collection" : "chameleon-flavor",
                "modified" : "2019-04-01 16:01:11.720274"
            },
            

``cloudmesh.example.bar`` contains:

::

    class Provider(object)

        def entries(self):
            return {
             "cm" : {
                "kind" : "flavor",
                "driver" : "openstack",
                "cloud" : "bar",
                "created" : "2019-04-01 15:59:39.815993",
                "name" : "m1.xxxlarge",
                "collection" : "chameleon-flavor",
                "modified" : "2019-04-01 16:01:11.720274"
            },

``cloudmesh.example.provider.foo`` contains:

::

    from cloudmesh.example.foo import Provider as FooProvider
    from cloudmesh.example.foo import Provider as BarProvider

    class Provider(object)

        def __init__(self, provider):
           if provider == "foo":
              provider = FooProvider()
           elif provider == "bar":
              provider = BarProvider()

        @DatabaseUpdate()
        def entries(self):
            provider.entries()

Separating the database and the dictionary creation allows the developer
to implement different providers but only use one class with the same
methods to interact for all providers with the database.
In the combined provider a find function to for example search for
entries by name across collections could be implemented.

Database Access
---------------

In addition to the decorator, we have a very simple database class for
interacting across a number of collections. THis especially is useful
for finding informtion.

::

    self.database = CmDatabase()

Find the entry with the uniqe name CC-Centos

::

    r = self.database.find_name("CC-CentOS7")
    pprint(r)

Find the entries with either CC-CentOS7 or CC-CentOS7-1811

::

    r = self.database.find_names("CC-CentOS7,CC-CentOS7-1811")
    pprint(r)

Find out how many entries exist with the name CC-CentOS7:

::

    r = self.database.name_count("CC-CentOS7")
    pprint(r)

Creating Unique Names
--------------------

Often it is important to craete uniqu names. To support the easy creation
without hazzle, we designed a ``Name`` class, that takes its values from the
cloudmesh ``cmd5`` shell variables. A good example is the following name,
where we like to identify within the name an experiment, a group of resources
within the
experiment, a user running the experiment and a counter. This can be set up
as follows

::

    {experiment}-{group}-{user}-{counter}


The values for them can be set with the cms set function

Thus if you use the name function in your program, you get a very convenient
way of getting a next name. Naturally you could define multiple such names
for different resources and needs

To use it in your program you can say:

::

    from cloumesh.management.configuration.name import Name

    name = Name(
        experiment="exp",
        group="grp",
        user="gregor",
        kind="vm",
        counter=1)

To increase the counter use

::

    name.incr()

To get the name at the current counter value say

::

    str(name) 

or

::

    name.id()

The format can be chaned with ``schema=`` at the initailization. Thus

::

    name = Name(
            user='gregor,
            schema='{user}-{counter}`,
            counter=1)

would create names of the form gergor1, gergor2 and so on.

