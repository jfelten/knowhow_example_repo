Tutorial on to to build an army of knowhow agents using Docker
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

In this job start up a bash shell to a centos7 image that we pull from Docker's standard library.  We then install the centos EPEL yum repository, nodejs, npm, and lastly knowhow via npm.  Note: we need to install make because it is needed by node-gyp.  We reinstall npm with npm because the npm rpm on EPEL is outdated.

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
    
###Clone the template to create a container

    {
        "id": "create a new KH docker container",
        "working_dir": "./",
        "options": {
            "timeoutms": 360000
        },
        "files": [],
        "script": {
        "env": {
            "CONTAINER_NAME": "container201",
            "DOMAIN": "zenzic.com",
            "GATEWAY": "10.10.1.1",
            "TEMPLATE_NAME": "knowhow"
        },
        "commands": [
        {
            "command": "docker stop ${CONTAINER_NAME} || :"
        },
        {
            "command": "docker rm ${CONTAINER_NAME} || :"
        },
        {
            "command": "docker run -d --name=${CONTAINER_NAME} --hostname=${CONTAINER_NAME}.${DOMAIN} ${TEMPLATE_NAME} startKHAgent --user=root"
        },
        {
            "command": "PID=`docker inspect --format='{{.State.Pid}}' ${CONTAINER_NAME}`"
        },
        {
            "command": "sleep 5"
        },
        {
            "command": "nsenter -t ${PID} -n /usr/sbin/ip route del default"
        },
        {
            "command": "nsenter -t ${PID} -n /usr/sbin/ip route add default via ${GATEWAY} dev eth0"
        }
        ]
        }
    }

### (OPTIONAL) Add a dns record for each container
Docker is easy to learn, but I found networking with Docker to be painful.  Out of the box Docker installs a virtual bridge called docker0, which handles connectivity via NAT.  This didn't work for me since I wanted all containers accessible on my normal network.  Since I already had a netowrk bridge I configured Docker to connect to it using a subset of my internal network.  I tried using pipework to assign addresses via dhcp, but that didn't work because the watered-down Docker networking wasn't able to renogiate a dhcp lease when it expired.  In the end I let docker assign a static IP and then used the following job to add a DNS record on my internal server.

    {
      "id": "add dns record for a new container",
      "working_dir": "./",
      "options": {
        "timeoutms": 360000
      },
      "files": [],
      "script": {
        "env": {
          "DNS_SERVER": "10.10.1.10",
          "DNS_ZONE": "zenzic.com",
          "CONTAINER_NAME": "container201",
          "UPDATE_FILE": "dns_update",
          "KEY_NAME": "/etc/rndc.key"
        },
        "commands": [
          {
            "command": "IP_ADDRESS=`docker inspect --format='{{.NetworkSettings.IPAddress}}' ${CONTAINER_NAME}`"
          },
          {
            "command": "rm -f ${UPDATE_FILE}"
          },
          {
            "command": "echo \"server ${DNS_SERVER}\" > ${UPDATE_FILE}"
          },
          {
            "command": "echo \"zone ${DNS_ZONE}\" >> ${UPDATE_FILE}"
          },
          {
            "command": "echo \"update delete ${CONTAINER_NAME}.${DNS_ZONE}. A\" >> ${UPDATE_FILE}"
          },
          {
            "command": "echo \"update add ${CONTAINER_NAME}.${DNS_ZONE}. 86400 A ${IP_ADDRESS}\" >> ${UPDATE_FILE}"
          },
          {
            "command": "echo 'show' >> ${UPDATE_FILE}"
          },
          {
            "command": "echo 'send' >> ${UPDATE_FILE}"
          },
          {
            "command": "nsupdate -k ${KEY_NAME} -v ${UPDATE_FILE}"
          }
        ]
      }
    }

