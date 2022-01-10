# DevOps Candidate Test

## Test Instructions

Dear candidate,
Below are the steps for your DevOps Engineer test.
In this test you'll create various files - dockerfiles, k8s manifests, shell scripts - and add them to this repository which will be served to the test reviewer.

- The test requires a working kubernetes cluster - make sure you got a kubeconfig file from your tester, and that you are able to connect to your cluster from your machine
- Note that each part of the test builds upon previous parts, so they should be done in the order they appear.
- Note that some parts of the tests are mandatory and some are bonus. Make sure to complete the mandatory parts before moving on to the bonus.
- Make sure all your files (dockerfiles, k8s manifests, shell scripts) are committed
- Make sure to commit your work incrementally as you advance

### Prerequisites
- Docker
- curl
- kubectl
- helm
- A docker registry to which you can push your docker images. You can create a free account on dockerhub.


### Step 1: Web Service Docker Image

In this step you'll be creating and publishing a docker image for a simple web service.
The service is based on a pre-existing NGINX image, with an additional configuration file provided in this repository.
The service listens on port 1234 and responds to HTTP requests on certain routes.

1. Create a Dockerfile
    - Base image: `openresty/openresty:1.19.9.1-0-buster-fat`
    - Add the `/nginx-config/default.conf` file from this repository to the image under `/etc/nginx/conf.d/default.conf`
2. Build the docker image with a tag of your choice
3. Push the image to your docker registry
4. Validate your work: 
    - Spin up a local container from your docker image
        - Make sure to map the container's port 1234 to some local port on your machine
    - Using curl/postman/browser, check that `http://localhost:<port>/hello` returns "world"

### Step 2: Kubernetes Resources
In this step you'll create kubernetes resource manifests to deploy your web service to a kubernetes cluster.

1. Create the following YAML manifests:
    - Deployment
        - Creates a pod from your docker image
    - Service
        - Exposes the pod under a load-balancer, mapping port 80 to the pod's port 1234
2. Deploy your resources to your kubernetes cluster
3. Validate your work:
    - Obtain the external address of your deployed service
    - Using curl/postman/browser, check that `http://<address>/hello` returns "world"

### Step 3: Helm Chart
In this step you'll create a Helm chart to enable deploying your web service with user-provided configuration.

1. Create a helm chart based on the resource manifest you created at step 2.
    - The chart needs to accept the following inputs from the `values.yaml` file
        - Number of replicas
        - External port number
2. Use helm to deploy your chart to your kubernetes cluster with the following values:
    - Replicas: 4
    - External port number: 5678
3. Validate your work:
    - Obtain the external address of your deployed service
    - Using curl/postman/browser, check that `http://<address>:5678/hello` returns "world"
    - Using `kubectl`, ensure that 4 replicas are in fact running

### Step 4: Performance Test Job
In this step you'll create a simple performance test that will benchmark your deployed service.

1. Create a kubernetes job resource manifest
    - The job should use the dockerized `wrk` load generator to test the `/hello` endpoint
        - Use the `williamyeh/wrk` image
        - Command line options for `wrk` can be found here: https://github.com/wg/wrk#command-line-options
    - The service address and number of threads should be environment variables
    - The duration of the run should be 30 seconds
2. Create a shell script that will be used locally to execute the performance test job
    - The script uses `kubectl` to find the external address and port number of the service
        - Updates the address environment variable of the job manifest accordingly 
    - The script performs a series of load tests, using 1, 2, 10, 50 and 100 threads
    - In each iteration, the script:
        1. Updates the threads environment variable
        2. Runs the performance test job in the k8s cluster
        3. Awaits for the job to complete
        4. Extracts the following values from the `wrk` output
            - Average latency
            - Maximum latency
            - Total requests executed
        5. Appends these results to a file under `./results.txt`
    - When completed, the script prints the results to the local console (stdout)
    - You may use whatever command-line utilities you with to implement the script, such as `jq`, `yq`, `awk`, `sed`, `grep` etc.

### Bonus 1: Pre-Commit Hook
Add a pre-commit hook to your git repository.

The hook should perform the following:
1. Run a [dockerfile linter](https://github.com/hadolint/hadolint#how-to-use) on the dockerfile you created at step 1 and aborts the commit if the linter returns an error
2. Builds the dockerfile and aborts the commit if the build process fails

### Bonus 2: Init Container
Add an init-container to your deployment, which will provide instance-specific information.

1. Add a Volume to your deployment called "startup-details"
2. Mount the volume to the web service's container under `/www` 
3. Create a shell script which 
    1. Creates a JSON file called `info` with the following keys:
        - `podName`: the name of the current pod
        - `startupTime`: the current timestamp
    2. Copies the file to the `/www` directory
4. Create a Dockerfile that creates a docker image for the init container
    1. Executes the shell script
5. Build the init container's docker image and push it to your registry
6. Add the init container to your deployment manifest, mounting the "startup-details" volume under `/www`
7. Validate your work:
    1. Deploy the updated deployment to your kubernetes cluster
    2. Ensure the JSON is accessible via the `/info` endpoint
