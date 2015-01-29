Tutorial using Docker
------------------------

[Docker](https://www.docker.com/) is a container based virutalization platform designed to run a single service per instance.  It works very well with knowhow, and the author (jfelten) uses docker to create a pool of workers that are managed with [knowhow-server](https://github.com/jfelten/knowhow-server).  In this exmample we are providing the jobs used to:

* install docker software on a host
* create a knowhow docker template
* commit the template to the docker repository
* create N number of containers running knowhow-agent with this template
* configure networking on each container





