Tutorial using Docker
------------------------

[Docker](https://www.docker.com/) is a container based virutalization platform designed to run a single service per instance.  It works very well with knowhow, and the author (jfelten) uses docker to create a pool of workers that are managed with [knowhow-server](https://github.com/jfelten/knowhow-server).  In this exmample we are providing the jobs used to:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create N number of containers running knowhow-agent with this template
* configure networking on each container

This tutorial is done with centos7, but can be easily adapted to the linux flavor of your choice.

###Install docker on a centos7 host

The docker software is available in the standard centos yum repo and should be installable out of the box.  This job uses yum to install the software and creates a systemd service.  Lastly it installs [pipework](https://github.com/jpetazzo/pipework), which we didn't end up using, but I left it in because it may be useful to someone else.

    {
      "id": "install docker software",
      "working_dir": "./",
      "options": {
        "timeoutms": 360000
      },
      "files": [],
      "script": {
        "env": {
          "TEST_VAR": "TEST",
          "TEST_VAR2": "TEST2"
        },
      "commands": [
        {
          "command": "yum -y install docker"
        },
        {
          "command": "systemctl enable docker"
        },
        {
          "command": "systemctl start docker"
        },
        {
          "command": "docker pull centos"
        },
        {
          "command": "bash -c \"curl https://raw.githubusercontent.com/jpetazzo/pipework/master/pipework > /usr/local/bin/pipework\""
        }
        ]
      }
    }





