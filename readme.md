# Datadog Cluster Agent : Horizontal Pod Autoscaling Exercice

# Introduction merge conflict test

The aim of this exercice is to setup HPA with the Datadog Cluster Agent. During this exercice you'll be guided through the following steps, and will have to troubleshoot various parts of the given manifests in order to get HPA running.
1. Setting up the Node Agent
2. Creating a Redis ReplicaSet
3. Setting up the Datadog Cluster Agent
4. Setting up the HPA Manifest
5. Stress your Redis instance

To get started, clone this repo locally.

# 1. Setting up the Node Agent

Checkout the `hpa1` branch by executing `git checkout hpa1`

Given the Datadog documentation and the files in the `/node-agent` folder, run a Datadog Node Agent Daemonset.

Please ensure that you create a Secret to store the `DD_API_KEY` environment value, with the key of `api-key`

*Hint : RBAC*

**Objective:** Ensure that you get the Datadog Agent reporting in your account.

# 2. Setting up a Redis ReplicaSet

Checkout the `hpa2` branch by executing `git checkout hpa2`

a. Using the data in `/redis`, run a Redis ReplicaSet with 1 replicas and the necessary pod annotations for the Datadog Agent to extract Redis metrics and logs.

b. Create a service for the Redis pod. *Hint: Expose*

c. How many Redis pods do you have running ?

**Objective:** Get Redis metrics flowing into your Datadog Account

# 3. Setting up the Datadog Cluster Agent

Checkout the `hpa3` branch by executing `git checkout hpa3`

a. Apply the RBAC configuration using the manifest in the `/dca/rbac` folder

b. Create the services described in the `/dca/services` folder

c. Analyzing the DCA manifest in `/dca`, edit the necessary files to get the DCA working. *Hint: secret*

**Objective:** View the Cluster Agent in your Container's Live View


# 4. Setting up the HPA manifest

Checkout the `hpa4` branch by executing `git checkout hpa4`

a. Apply the RBAC configuration using the manifest in the `/hpa/rbac` folder

b. Using `hpa-manifest` as an example, create a Horizontal Pod Autoscaler for the Redis ReplicaSet you built, that will scale yours pods to 5 replicas if the amount of connections to the redis instance is higher than 20.

c. Ensure that your Horizontal Pod Autoscaler is working, use `kubectl describe hpa redisext`. *HINT: Make sure that the DCA has list and watch rights to pod autoscalers, use SEDOCS for help*

**Objective:** Get the Horizontal Pod Autoscaler running without errors

# 5. Stress your Redis instance

Checkout the `master` branch by executing `git checkout master`

a. Exec into your Redis pod
b. Execute 100K requests from 30 clients on your Redis instance. *HINT: `redis-benchmark`*
c. Check the value of `redis.net.clients` in your Datadog account.
d. How many Redis pods do you have now?

**Objective:** Scale up your redis pods
