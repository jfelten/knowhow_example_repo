Tutorial using Docker
------------------------

[Docker](https://www.docker.com/) is a container based virutalization platform designed to run a single service per instance.  It works very well with knowhow, and the author (jfelten) uses docker to create a pool of workers that are managed with [knowhow-server](https://github.com/jfelten/knowhow-server).  In this exmample we are providing the jobs used to:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create N number of containers running knowhow-agent with this template
* configure networking on each container

This tutorial is done with centos7, but can be easily adapted to the linux flavor of your choice.

###Install Docker on a centos7 host

The Docker software is available in the standard centos yum repo and should be installable out of the box.  This job uses yum to install the software and creates a systemd service.  Lastly it installs [pipework](https://github.com/jpetazzo/pipework), which we didn't end up using, but I left it in because it may be useful to someone else.

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
    
###Create a knowhow docker template

In this job start up a bash shell to a centos7 image that we pull from Docker's standard library.  We then install the centos EPEL yum repository, nodejs, npm, and lastly knowhow via npm.  Note: we need to install make because it is needed by node-gyp.

    {
        "id": "create KH docker template",
        "working_dir": "./",
        "options": {
            "timeoutms": 3600000
        },
        "shell": {
            "command": "docker run -i -t --name=${CONTAINER_NAME} --hostname=${CONTAINER_NAME}.${DOMAIN} centos /bin/bash",
            "onConnect": {
                "waitForPrompt": "[#]"
            },
            "onExit": {
                "command": "exit\r\n"
             }
        },
        "files": [],
        "script": {
        "env": {
            "CONTAINER_NAME": "KH_template",
            "DOMAIN": "zenzic.com",
            "EPEL_RPM": "epel-release-7-5.noarch.rpm",
            "EPEL_RPM_LOC": "http://dl.fedoraproject.org/pub/epel/7/x86_64/e/${EPEL_RPM}"
        },
        "commands": [
        {
            "command": "curl -O ${EPEL_RPM_LOC}"
        },
        {
            "command": "rpm -ivh --replacepkgs ${EPEL_RPM}"
        },
        {
            "command": "yum -y install telnet mlocate"
        },
        {
            "command": "yum update -y"
        },
        {
            "command": "rm -rf /usr/lib/node_modules"
        },
        {
            "command": "yum -y erase npm nodejs"
        },
        {
            "command": "yum -y install nodejs npm make"
        },
        {
            "command": "npm install -g npm"
        },
        {
            "command": "npm update -g"
        },
        {
            "command": "npm install knowhow -g"
        }
        ]
    }
    }

###Commit the template to the docker repository

First remove any existing template, and then commit.  We named our repository knowhow and gave it a tag name latest.

        {
            "id": "commits the docker template",
            "working_dir": "./",
            "options": {
                "timeoutms": 360000
            },
            "files": [],
            "script": {
            "env": {
                "REPOSITORY_NAME": "knowhow",
                "TAG_NAME": "latest",
                "TEMPLATE_NAME": "KH_template"
            },
            "commands": [
            {
                "command": "docker rmi ${REPOSITORY_NAME}:${TAG_NAME}"
            },
            {
                "command": "docker commit ${TEMPLATE_NAME} ${REPOSITORY_NAME}:${TAG_NAME}"
            }
            ]
        }
}



