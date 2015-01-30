workflow for creating a docker environment
------------------------------------------

Please read the tutorial under knowhow_example_repo/jobs/docker first.  This explains the jobs used here.  Here is a summary of what this workflow does:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create 10 docker containers running knowhow-agent with this template
* configure networking on each container
* 

###Step 1 Create the environment file
Environment files define and designate hosts within your system.  All environment files must be named environmnet.json and may be placed anywhere under the environment directory.  For this example we have created an environment file: knowhow_repo:///environments/docker/environment.json that defines 11 hosts: 1 docker hsot + 10 containers.  We definedthat the docker host in on localhost.  All other hosts get created via this workflow.


    {
      "id": "docker environment",
      "agents": {
        "docker_host": {
          "user": "root",
          "host": "localhost",
          "port": 3000
        },
        "worker1": {
          "user": "root",
          "host": "worker1",
          "port": 3000
        },
        "worker2": {
          "user": "root",
          "host": "worker2",
          "port": 3000
        },
        "worker3": {
          "user": "root",
          "host": "worker3",
          "port": 3000
        },
        "worker4": {
          "user": "root",
          "host": "worker4",
          "port": 3000
        },
        "worker5": {
          "user": "root",
          "host": "worker5",
          "port": 3000
        },
        "worker6": {
          "user": "root",
          "host": "worker6",
          "port": 3000
        },
        "worker7": {
          "user": "root",
          "host": "worker7",
          "port": 3000
        },
        "worker8": {
          "user": "root",
          "host": "worker8",
          "port": 3000
        },
        "worker9": {
          "user": "root",
          "host": "worker9",
          "port": 3000
        },
        "worker10": {
          "user": "root",
          "host": "worker10",
          "port": 3000
        }
      }
    }
