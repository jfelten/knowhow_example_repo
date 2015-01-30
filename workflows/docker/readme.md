workflow for creating a docker environment
------------------------------------------

Please read the tutorial under knowhow_example_repo/jobs/docker first.  This explains the jobs used here.  Here is a summary of what this workflow does:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create 10 docker containers running knowhow-agent with this template
* configure networking on each container

###Step 1 Create the environment file
Environment files define and designate hosts that run agents within your system.  All environment files must be named environmnet.json and may be placed anywhere under the environment directory.  For this example we have created an environment file: knowhow_repo:///environments/docker/environment.json that defines 11 hosts: 1 docker hsot + 10 containers.  We definedthat the docker host in on localhost.  All other hosts get created via this workflow.


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

###Step 2 Define the workflow
A knowhow workflow is a collection of tasks.  Each task wraps a job and defines an execution type:  series or paralell.  Series tasks are run one at a time in the order specified.  Paralell tasks are run concurrently.  In this example we will run everything as a series task.  Here is a walkthrough of the tasks: 

* Task 1 On host localhost install docker and everything else it needs using job: knowhow-example:///jobs/docker/installDocker.json
* Task 2 On host localhost create a centos7 template that has knowhow installed using job: knowhow-example:///jobs/docker/createKHTemplate.json
* Task 3 On host localhost commit the template to the local docker repository using job: knowhow-example:///jobs/docker/commitTemplate.json
* Task 4 On host localhost create the containers worker1-worker10 in series using job: knowhow-example:///jobs/docker/createHost.json
* Task 5 On host localhost add DNS entires for worker1-worker10 in series using job: knowhow-example:///jobs/docker/addDNSEntry.json
