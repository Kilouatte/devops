# TP1 : devops
## Q1.2 : Why do we need a multistage build? And explain each step of this dockerfile.

We use a multistage build to : </br>
- reduce the image size, we copy only necessary files or artifacts into the last stage. </br>
- isolate the build dependencies, all the lib required during the build phase are isolated in the intermediate stage. So the last image include runtime depencencies of the application and nothing else.</br>
- separe the different phases of the build process. It makes it easir to manage and we can understand the Dockerfile in each stage.

## Q1.3 : 

