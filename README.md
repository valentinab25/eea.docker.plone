# Plone ready to run Docker image

Docker image for Plone with `plone.recipe.zope2instance` full support
(supports all plone.recipe.zope2instance options as docker environment variables).

This image is generic, thus you can obviously re-use it within
your non-related EEA projects.

### Supported tags and respective Dockerfile links

  - `:4.3.6`, `:latest` (default)

### Base docker image

 - [hub.docker.com](https://registry.hub.docker.com/u/eeacms/plone-instance)

### Source code

  - [github.com](http://github.com/eea/eea.docker.plone)

### Installation

1. Install [Docker](https://www.docker.com/).

## Usage

Most of the configuration of this image is based on the 
[plone.recipe.zope2instance](https://pypi.python.org/pypi/plone.recipe.zope2instance)
recipe package so it is advised that you check it out.

### Run with basic configuration

    $ docker run eeacms/plone-instance:latest

The image is built using a bare `buildout.cfg` file:

    [buildout]
    extends = http://dist.plone.org/release/4.3.6/versions.cfg
    parts = instance
    
    [versions]
    zc.buildout = 2.2.1
    setuptools = 7.0
    
    [instance]
    recipe = plone.recipe.zope2instance
    user = admin:admin
    effective-user = zope-www
    eggs =
      Pillow
      Plone
      plone.app.upgrade

`plone` will therefore run inside the container with the default parameters given
by the recipe, such as listening on `port 8080`.

### Extend configuration through environment variables

Environment variables can be supplied either via an `env_file` with the `--env-file` flag
    
    $ docker run --env-file plone.env eeacms/plone-instance:latest

or via the `--env` flag

    $ docker run --env BUILDOUT_HTTP_ADDRESS="80" eeacms/plone-instance:latest

It is **very important** to know that the environment variables supplied are translated
into buildout configuration. For each variable with the prefix `BUILDOUT_` there will be
a line added to the `[instance]` configuration. For example, if you want to set the
`read-only` attribute to the value `true`, you have to supply an environment variable
in the form `BUILDOUT_READ_ONLY="true"`. When the environment variable is processed,
the prefix is striped, `_` turns to `-` and uppercase turns to lowercase. Also, if the
value is enclosed in quotes or apostrophes, they will be striped. The configuration will
look like

    [instance]
    ...
    read-only = true
    ...

The variables supported are the ones supported by the [recipe](https://pypi.python.org/pypi/plone.recipe.zope2instance),
so check out its documentation for a full list. Keep in mind that this option will trigger
a rebuild at start and might cause a few seconds of delay.

### Use a custom configuration file mounted as a volume

    $ docker run -v /path/to/your/configuration/file:/opt/plone/buildout.cfg eeacms/plone-instance:latest

You are able to start a container with your custom `buildout` configuration with the mention
that it must be mounted at `/opt/plone/buildout.cfg` inside the container. Keep in mind
that this option will trigger a rebuild at start and might cause delay, based on your
configuration. It is unadvised to use this option to install many packages, because they will
have to be reinstalled every time a container is created.


### Extend the image with a custom configuration file

Additionaly, in case you need to considerably change the base configuration of this image,
you can extend it with your configuration. You can write Dockerfile like this

    FROM eeacms/plone-instance:latest
    # Add your configuration file
    COPY path/to/configuration/file /opt/plone/base.cfg
    # Rebuild
    RUN /opt/plone/bin/buildout -c /opt/plone/base.cfg

and then run

   $ docker build -t your-image-name:your-image-tag path/to/Dockerfile

## Persistent data as you wish

The `plone-instance` data is kept in a [data-only container](https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e).
The `data` container keeps the persistent data for a production environment and must be backed up. If you are running in a devel environment,
you can skip the backup and delete the container if you want.

If you have a Data.fs file for your application, you can add it to the `data` container with the following
command:

    $ docker run --rm --volumes-from <name_of_your_data_container> \
      -v /path/to/parent/directory/of/Data.fs/file:/mnt:ro \
      debian /bin/bash -c "cp /mnt/Data.fs /opt/plone/var/filestorage && \
      chown -R 500:500 /opt/plone/var/filestorage"

The command above creates a bare `debian` container using the persistent volumes of your data container.
The parent directory of the `Data.fs` file is mounted as a `read-only` volume in `/mnt`, from where the
`Data.fs` file is copied to the filestorage directory you are going to use (default `/opt/plone/var/filestorage`).
The `data` container must have this directory marked as a volume, so it can be used by the `plone-instance` container,
with a command like:

    $ docker run --volumes-from <name_of_your_data_container> eeacms/zeoserver

The volumes from the `data` container will overwrite the contents of the directories inside the `zeoserver`
container, in a similar way in which the `mount` command works. So, for example, if your data container
has `/opt/zeoserver/var/filestorage` marked as a volume, running the above command will overwrite the
contents of that folder in the `zeoserver` with whatever there is in the `data` container.

The data container can also be easily [copied, moved and be reused between different environments](https://docs.docker.com/userguide/dockervolumes/#backup-restore-or-migrate-data-volumes).

### Docker-compose example

A `docker-compose.yml` file for `plone-instance` using a `data` container:

    plone-app:
      image: eeacms/plone-instance:latest
      user: "500"
      volumes_from:
       - data
       
    data:
      build: data
      volumes:
       - /opt/plone/var/filestorage



## Upgrade

    $ docker pull eeacms/plone-instance:latest


## Supported environment variables ##

As mentioned above, the supported environment variables are derived from the configuration options
from the [recipe](https://pypi.python.org/pypi/plone.recipe.zope2instance). For example, `read-only`
becomes `BUILDOUT_READ_ONLY` and `http-address` becomes `BUILDOUT_HTTP_ADDRESS`.

For variables that support a list of values (such as `eggs`, for example), separate them by space, as
in `BUILDOUT_EGGS="eea.pdf eea.annotator"`.

For complex variables (such as `event-log-custom`, for example), specify new lines with `\n`, as
in BUILDOUT_EVENT_LOG_CUSTOM="<graylog> \n server 123.4.5.6 \n rabbit True \n </graylog>"

Besides the variables supported by the `zope2instance` recipe, you can also use `INDEX` and `FIND_LINKS`
that extend the `[buildout]` tag.

## Copyright and license

The Initial Owner of the Original Code is European Environment Agency (EEA).
All Rights Reserved.

The Original Code is free software;
you can redistribute it and/or modify it under the terms of the GNU
General Public License as published by the Free Software Foundation;
either version 2 of the License, or (at your option) any later
version.


## Funding

[European Environment Agency (EU)](http://eea.europa.eu)
