workflow for creating a docker environment
------------------------------------------

Please read the tutorial under knowhow_example_repo/jobs/docker first.  This explains the jobs used here.  Here is a summary of what this workflow does:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create 10 docker containers running knowhow-agent with this template
* configure networking on each container
