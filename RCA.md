# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output

target proxy: failed to solve: process "/bin/sh -c apt-get update &&     apt-get install -y nginx curl htop vim git" did not complete successfully: exit code: 100


### How I diagnosed it

* I broke the RUN row in the proxy dockerfile into multiple RUN lines- to isloate the failing package.

* I identified that the nginx package was causing the build failure.

### Root cause
nginx apt installation pops a prompt regarding overriding or keeping the existing nginx conf file- the failure is caused because the apt install is ran in an noninteractive environment.

#9 10.16 *** nginx.conf (Y/I/N/O/D/Z) [default=N] ? dpkg: error processing package nginx-common (--configure):

### Fix

passing the argument for the prompt in the dockerfile RUN line- keeping the original existing nginx conf file instead of overriding it.

RUN DEBIAN_FRONTEND=noninteractive apt-get install --option "Dpkg::Options::=--force-confdef" --option "Dpkg::Options::=--force-confold"

---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output

The test runner crashed displaying:

Testing proxied backend /api/info...
TEST RUNNER CRASHED: HTTP Error 502: Bad Gateway

### How I diagnosed it

* I tried accessing the proxy backend by entering the following url in chrome: http://localhost:8081/api/info

* I viewed the nginx container logs and viewed the following error:

    [error] 7#7: *7 connect() failed (111: Connection refused) while connecting to upstream, client: 172.20.0.1, server: localhost, request: "GET /api/info HTTP/1.1", upstream: "http://127.0.0.1:8080/api/info", host: "localhost:8081"

* I then ran the following connectivity checkup command from within the proxy container:
    curl http://127.0.0.1:8080/api/info
    and got the following error:
    curl: (7) Failed to connect to localhost port 8080 after 12 ms: Couldn't connect to server

### Root cause
Eventhough the containers are both in the same default docker network, they are inaccessible to one another.

### Fix
deploying the containers on a new docker network-and provide them with hostnames so they would be able to resolve each other's ip addresses via docker native dns translation.

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

the test-runner still crashes displaying the same error:

Testing proxied backend /api/info...
TEST RUNNER CRASHED: HTTP Error 502: Bad Gateway

### How I diagnosed it
* I manually checked the proxied backend /api/info and it worked successfully

* I reset the test-runner container and got a successful result with the following message: ALL TESTS PASSED!

* I ran docker compose logs- to view all of the containers logs- and i saw that the test container attempted to query the backend before the app was listening on the 8080 port.

### Root cause
race conditions- the tester container attempts to query the backend before it is done running the app.

### Fix, and why I chose it over the alternatives
Fix:
    - I configured health checks to the nginx and backend containers

    - I configured that each container will not initialize until its container dependencies are initialized and healthy

Reason:

    - configuring health checks might offer an easier way in the future to check the app's health and availability

    - each container's initalization time might change over time due to multiple reasons-therefore it is better to offer a generic solution non-dependent on the initialization time.
---

## Defect 4: CI pipeline fails

### Symptom / error output
* when viewing the github page there is an X indicating that the workflow failed on the last commit pushed

### How I diagnosed it
* I inspected the pipeline's run details and noticed the following error:
    Run ./scripts/run-tests.sh
    /home/runner/work/_temp/90a68932-127b-4852-a034-4ea8318d36b6.sh: line 1: ./scripts/run-tests.sh: Permission denied
    Error: Process completed with exit code 126.

* I modified the workflow in order to add a debug stage to view the file's permissions on the filesystem:
    command: Run ls -l ./scripts/run-tests.sh
    output: -rw-r--r-- 1 runner runner 499 Oct 20 19:24 ./scripts/run-tests.sh

### Root cause

the file does not have executable permissions, therefore when the runner attempts to execute it a permission error is displayed.

### Fix

setting the executable permissions via git:

git update-index --chmod=+x scripts/run-tests.sh
---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

1) 
    - I removed the unecessary packages: htop, vim, git
    - I changed the base image to alpine

    image size before: 121MB
    image size after: 7.8MB

2) 
    - I modified the dockerfile to run the container with the nginx user and to expose port 8081 instead of 80
    - I modified the nginx.conf to listen in an unprivilliged port -port 8081
    - I modified the docker compose file to bind the ports 8081:8081

3) 
    - Isolating the backend so that access is only through proxy
        * I created a private internal docker network
        * I detached the backend container from the public net
        * I attached the backend and the proxy containers to the private-net

    
    - Adding best practice nginx conf settings


    what i hoped to do but did not have enough time:
    - Encrypting proxy communication using ssl
        * modifying nginx conf so that every remote connection only uses ssl only
        * modifying nginx conf so that test-runner is able to access proxy via http

        That way the test-runner doesn't need any modification-and can still check container's health via http protocol on the internal network.



---

## Anything you would do differently with more time
- Explore additional ways to make the images lighter.
- Find some other docker compose settings to make the containers more secure.
- Try to understand the nginx configuration.