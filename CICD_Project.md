- The requirement is to simulate a full CI/CD Cycle 
- It's preferable to create a separate namespace for each of the tools used 

### SCM 
- You can use `Gitea` or `Gogs`as a place to store the code
- Gitea should also have a container registry for storing container images 

### Jenkins CI 
- Use jenkins to generate new container images from the source code of whatever language you prefer (Java etc)
- Push the image to the container registry
- Push the image tag to the repository on Gitea (Think of using **YQ** to do this step)

### ArgoCD 
- Should be configured to watch the repository on Gitea
- Once a new image tag is pushed, ArgoCD should reflect the changes on the kubernetes cluster

** BONUS - think of using an image scanning tool like trivy to validate images contain no vulnerabilites before pushing the image to the registry