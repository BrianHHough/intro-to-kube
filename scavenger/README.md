# Scavenger Hunt

🎉 Woohoo!!!! 🎉 We are thrilled to welcome you to our DevOps team at StartupCo! 🚀 Your expertise and skills will undoubtedly bring a fresh perspective to our innovative environment.

Your next ramp-up task is going to be an exciting one! Get ready for a Kubernetes Scavenger Hunt 🕵️‍♂️🔍, where you'll embark on an adventure to explore the resources within our existing cluster. This engaging exercise will enable you to dive deeper into Kubernetes and discover its hidden gems, all while having a ton of fun! Let the Kubernetes Scavenger Hunt begin! May the pods be ever in your favor! 🌟

## Setup

Create K8s resources by running

```
./scavenger-setup.sh
```

Add add your answers to `scavenger.json`

We have GitHub Actions configured to report your score to our scavenger hunt. Just create your own branch prefixed with `scavenger-` and push to it. [Then you can see your results here](https://github.com/abanuelo/intro-to-kube/actions/workflows/scavenger.yml).

## Mark I - new beginngs

0. What are the total number of pods?
1. How many namespaces are there?
2. How many nodes do we have?
3. How many failing pods are there?
4. How many deployments do we have?
5. For the deployments you discovered, how many ReplicaSets are there in total?

## Mark II - not so new beginnings

6. Locate a postgres pod within our cluster. Why is it failing? Please provide the variable name(s) that is missing.
7. Locate a failed nginx pod within our cluster. Why is it failing? Please copy and paste the specific `kubectl` log you find. Please escape `"` with `\"`.

## Mark III - fixing our bugs

8. For the postgres pod, can you fix the `./k8s/postgres-pod.yml` file to spin up the image correctly? Please add the environment variable needed with value "supersecret". Added your lines of code.
9. For the failed nginx pod, can you fix the `./k8s/nignx-pod.yml` file to spin up the image correctly. Add the lines of code you added/corrected and please escape `"` with `\"`.
