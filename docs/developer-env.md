# Developer Environment

If you are developing Kolla on an existing OpenStack cloud
that supports Heat, then follow the Heat template [README][].
Otherwise, follow the instructions below to manually create
your Kolla development environment.

[README]: https://github.com/stackforge/kolla/tree/version-m3/devenv/README.md

In order to run Kolla, it is mandatory to run a version of
`docker-compose` that includes pid: host support.  One of the
authors of Kolla has a pull request outstanding that the
docker-compose maintainers have said they would merge shortly.

The pull request is:

    https://github.com/docker/compose/pull/1011

Until then, it must be retrieved via git and installed:

    git clone http://github.com/sdake/fig
    cd fig
    sudo pip install -e .
    sudo pip install -U docker-py
    sudo pip install -e .
    sudo pip install six==1.7.3

The docker-compose version available via the sdake repository has been
rebased on to a master version of docker-compose which requires the
docker API 1.18.  the docker API 1.18 is not available in distro
packaging and is only available by building from source.  Docker also
distributes pre-built binaries for docker.  It is recommended to just run
the docker provided binaries rather then building from source:

Further complicating matters, docker requires the replace_execdriver
branch from:

    https://github.com/LK4D4/docker/tree/replace_execdriver

See:

    https://github.com/docker/docker/issues/11760
    https://github.com/docker/compose/issues/812

If a version of Docker other than the replace_exec driver branch is currently
running on your system, stop it:

    sudo systemctl stop docker
    sudo killall -9 docker

Next, download and run the Docker 1.6 lk4d4 (Docker Inc Employee) built binary:

    Login to dropbox and download https://www.dropbox.com/s/vyz79t4r7nicltc/docker-1.6.0-rc2?dl=0
    mv docker-1.6.-rc2 docker
    sudo ./docker -d &

Finally stop libvirt on the host machine.  Only one copy of libvirt may be
running at a time.

    service libvirt stop

The basic starting environment will be created using `docker-compose`.
This environment will start up the openstack services listed in the
compose directory.

To start, setup your environment variables.

    $ cd kolla
    $ ./tools/genenv

The `genenv` script will create a compose/openstack.env file
and an openrc file in your current directory. The openstack.env
file contains all of your initialized environment variables, which
you can edit for a different setup.

Next, run the start script.

    $ ./tools/start

The `start` script is responsible for starting the containers
using `docker-compose -f <osp-service-container> up -d`.

If you want to start a container set by hand use this template

    $ docker-compose -f glance-api-registry.yml up -d

# Debug

All Docker commands should be run from the directory of the Docker binary,
by default this is `/`.

You can follow a container's status by doing

    $ sudo ./docker ps -a

If any of the containers exited you can check the logs by doing

    $ sudo ./docker logs <glance-api-container>
    $ docker-compose logs <glance-api-container>

If you want to start a individual service like `glance-api` by hand, then use
this template.  This is a good method to test and troubleshoot an individual
container.

    $ sudo ./docker run --name glance-api -d \
             --net=host
             --env-file=openstack.env kollaglue/fedora-rdo-glance-api:latest
