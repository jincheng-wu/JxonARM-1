# JxOnArm

# Introduction of P3.1

P3.1 is a simple test for lighthouse docking with Tekton. The process of testing is to change the code repository and automatically trigger the pipeline. 

## preparation

* Deploy kubernetes cluster with one master and two nodes.
* Deploy the basic components, such as helm3, local docker registry
* Build the image of lighthouse for arm64 architecture. The reference link is as follows: https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/Build%20lighthouse-0.0.748%20on%20Arm64.md
* Build the image of tekton for arm64 architecture. The reference link is as follows: https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/Build%20tekton-v0.15.2%20on%20Arm64.md
* Build the image of kaniko for arm64 architecture. The reference link is as follows: https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/Build%20kaniko-v0.20.0%20on%20Arm64.md
* Build the image of bucketrepofor arm64 architecture. The reference link is as follows: https://github.com/yyunk/jenkins-x-arm-support/blob/master/doc/Build%20bucketrepo-0.1.12%20on%20Arm64.md

## Operation process

step1: 

According to the official lighthouse documentation, install and deploy the lighthouse. The reference link is as follows https://github.com/jenkins-x/lighthouse#installing. (2020.12.29)

step2:

Deploy the tekton according to the charts provided by the jenkins-x team. The process  as follows:

1. Clone the charts from https://github.com/jenkins-x-charts/tekton

2. Modify the values.yaml to your images

3. Run command `helm install` to deploy the tekton

Besides, you can install and deploy the tekton according to the official tekton documentation. The reference link is as follows https://github.com/tektoncd/pipeline/blob/master/docs/install.md (2020.12.29)

step3:

After the two are deployed, refer to the official lighthouse tutorial to configure the docking. https://github.com/jenkins-x/lighthouse/blob/master/docs/install_lighthouse_with_tekton.md (2020.12.29)

## Testing process

The process as follows:

1. Create a test code repository, such as golang web hello-world.
2. Write a pipeline, the process is to pull the code, compile the code, use kaniko to build a container image and push it to a local docker registry
3. Set up webhook in the code repository
4. Change the configmap of plugins and config of lighthouse

The specific steps refer to https://github.com/jenkins-x/lighthouse/blob/master/docs/install_lighthouse_with_tekton.md (2020.12.29)

So far, the configuration has been completed. Perform corresponding operations on the code repository to trigger the pipeline. If the pipeline can run successfully, the P3.1 is finished.

The golang demo pipeline link is as follows https://github.com/jincheng-wu/JxOnArm/tree/main/P3.1%20Simple%20tekton%20CI/tekton-pipeline .

If the pipeline run successfully, the container images will be pushed to the designated docker registry. Then, we can pull it and run it to test weather the image can be used.

# Introduction of P3.2

We now have an t1 environment composed of lighthouse, tekton, kaniko, nexus. We want to use this environment deploy another same t2 environment. Then, we will use t2 environment to run an simple golang demo like P3.1. P3.2 consists of two parts, CI and CD.

P3.1 and P3.2 are mainly different in the following aspects:

* We run pipeline with shell script in it. There are some differences between different components to compile and build container images. We want to write a reusable pipeline. So we use shell script to realize it.
* In P3.1, we use local docker registry, in p3.2, we use nexus docker registry.
* P3.1 just build the container image(CI phase), do not deploy it. P3.2 contains the deploy phase(CD phase).

## Specific process of CI

Write CI pipelines, with these, the arm images of lighthouse, tekton, knaiko, bucketrepo are automatically constructed.

### lighthouse

1. The first step of the pipeline is to pull the corresponding code from github. But after the code of the official repository is pulled down, it is not suitable for the machine of arm64 architecture to directly compile build the container image. 

   Therefore, we fork a source code, and then change some of it's contents, such as dockerfile and makefile. We use this repository as the main code repository of this project. Such as https://github.com/yyunk/lighthouse/tree/modify-0.0.748 (fork from lighthouse-tag-0.0.748 )

2. Write general shell script to reuse in pipeline. The pipeline's process is to pull the code, compile the code, use kaniko to build a container image. 

   We add a configuration file to achieve this goal.The additional profile contains the path of binary file, the path of dockerfile, the name of container image, they are split by commas. The last but not the least, the path of binary file in the dockerfile, it is in a separate row.

   The docker registry url and image tag are in pipeline parameters. The example of additional profile's link as follows: https://github.com/yyunk/lighthouse/blob/modify-0.0.748/arm_config

3. Set the webhook in the forked repo. 

4. Modify the configmap of plugins and config of lighthouse in kubetrnetes cluster.

5. Perform corresponding operations on the code repository to trigger the pipeline. Such as modify the code in the repo.

6. The pipeline will run automatically, if successfully, the container image will be pushed in to docker registry.

### tekton

1. The first step of the pipeline is to pull the corresponding code from github. But after the code of the official repository is pulled down, it is not suitable for the machine of arm64 architecture to directly compile build the container image. 

   Therefore, we fork a source code, and then change some of it's contents, such as dockerfile and makefile. We use this repository as the main code repository of this project. Such as https://github.com/yyunk/pipeline/tree/modify-release-v0.15.x (fork from tekton-pipeline-tag-v0.15)

2. Write general shell script to reuse in pipeline. The pipeline's process is to pull the code, compile the code, use kaniko to build a container image. 

   We add a configuration file to achieve this goal.The additional profile contains the path of binary file, the path of dockerfile, the name of container image, they are split by commas. The last but not the least, the path of binary file in the dockerfile, it is in a separate row.

   The docker registry url and image tag are in pipeline parameters. The example of additional profile's link as follows: https://github.com/yyunk/pipeline/edit/modify-release-v0.15.x/arm_config

3. Set the webhook in the forked repo. 

4. Modify the configmap of plugins and config of lighthouse in kubetrnetes cluster.

5. Perform corresponding operations on the code repository to trigger the pipeline. Such as modify the code in the repo.

6. The pipeline will run automatically, if successfully, the container image will be pushed in to docker registry.

### bucketrepo

1. The first step of the pipeline is to pull the corresponding code from github. But after the code of the official repository is pulled down, it is not suitable for the machine of arm64 architecture to directly compile build the container image. 

   Therefore, we fork a source code, and then change some of it's contents, such as dockerfile and makefile. We use this repository as the main code repository of this project. Such as https://github.com/yyunk/bucketrepo/tree/modify-0.1.12 (fork from bucketrepo-tag-0.1.12)

2. Write general shell script to reuse in pipeline. The pipeline's process is to pull the code, compile the code, use kaniko to build a container image. 

   We add a configuration file to achieve this goal.The additional profile contains the path of binary file, the path of dockerfile, the name of container image, they are split by commas. The last but not the least, the path of binary file in the dockerfile, it is in a separate row.

   The docker registry url and image tag are in pipeline parameters. The example of additional profile's link as follows: https://github.com/yyunk/bucketrepo/blob/modify-0.1.12/arm_config

3. Set the webhook in the forked repo. 

4. Modify the configmap of plugins and config of lighthouse in kubetrnetes cluster.

5. Perform corresponding operations on the code repository to trigger the pipeline. Such as modify the code in the repo.

6. The pipeline will run automatically, if successfully, the container image will be pushed in to docker registry.

### task

* The lighthouse CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/lighthouse_task.yaml
* The tekton CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/tekton_task.yaml
* The kaniko CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/kaniko_task.yaml
* The bucketrepo CI task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/bucketrepo_task.yml

Specifically, if the docker registry which used in kaniko container need login, you shoud mount the docker config. If not, you will have no authority to pull and push images.

## Specific process of CD

On the basis of CI pipelines, writing CD pipeline. So that the components can be deployed in the kubernetes cluster in another namespace. We use an image with helm CLI and kubectl CLI. so we can use helm to deploy the components. 

The pipeline process as follows:

1.  Git clone the charts repo or helm add the charts. 
2. Modify the values.yaml to the images built before or using pre prepared values.yaml
3. Use helm install to deploy the components.

* The lighthouse CD task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/lighthouse_CD_task.yaml

* The tekton CD task is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/tekton_CD_task.yaml

* The bucketrepo CDtask is as follows: https://github.com/jincheng-wu/JxOnArm/blob/main/P3.2%20tekton%20CICD/task/bucketrepo_CD_task.yml

## Test

After the new environment deployed, we will test it.

The test process is as the same as the P3.1 simple demo test. 

## Issues

1. Some resources are global, such as CRD, clusterrole, so if we want to deploy components in two namespace, we should change the charts values.
2. The images which are built by kaniko are not able to use successfully. These storage is less than images which are built by docker, and will miss command line tool such as `git`even if i hava installed it in dockerfile.
