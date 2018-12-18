# Stage triggers and conditions

Pipelines provide a well-defined, repeatable way to continuously push changes to production while minimizing human intervention. However, even the simplest of pipelines can be demanding in terms of the control over orchestration like how the deployment is started,  the sequence in which targets are deployed, the targets that are deployed etc. Stage triggers and conditions help users to enable this by providing constructs to define the flow of execution.


## Stage Triggers
These are events on which a stage condition evaluation would start. The execution flow of stages is very much similar to that of [job](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=vsts&tabs=schema#job). I will take the example of a ring based geographic deployment to explain the basic flow. 

![IMG: Ring based deployment](https://github.com/zenithworks/azure-pipelines-yaml/blob/master/capture20181218115033741.png)

#### 1. On Pipeline Trigger:
*Eg. I want to begin  'Dev' stage execution as soon as a new run is triggered*
All stages will, by default, start on a pipeline trigger. 

#### 2. Sequential:
*Eg. I want to begin 'Test' stage execution as soon as 'Dev' stage completes.*
Use 'dependsOn' keyword to specify a stage dependency:

    - stage: Test
      dependsOn: Dev

#### 3. Fan-out:
*Eg. I want to begin 'Ring-1a'(Africa), 'Ring-1b'(Australia) as soon as 'Ring-0'(internal users) is complete.*  

    - stage: Ring-1a
      dependsOn:
      - Ring-0
    ...
    - stage: Ring-1b
      dependsOn:
      - Ring-0



#### 4. Fan-in:
*Eg. I want to begin 'Ring-2'(Europe) as soon as both 'Ring-1a'(Africa) and  'Ring-1b'(Australia) are complete.* 

    - stage: Ring-2
      dependsOn:
      - Ring-1a
      - Ring-1b 

#### 5. Manual: 
*Eg. I do not want to start 'Ring-0'(internal users) automatically. I want to start it manually only after deployment to 'PPE' stage is successful and I have verified the artifacts and other configurations.* 

    - stage: Ring-0
      manual: true
      dependsOn: PPE



## Stage Conditions

These are conditions which need to pass for a stage to be executed.
