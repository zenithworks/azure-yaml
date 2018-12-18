# Stage triggers and conditions

Pipelines provide a well-defined, repeatable way to continuously push changes to production while minimizing human intervention. However, even the simplest of pipelines can be demanding in terms of the control over orchestration like how the deployment is started,  the sequence in which targets are deployed, the targets that are deployed etc. Stage triggers and conditions help users to enable this by providing constructs to define the flow of execution.


## Stage Triggers
These are events on which a stage condition evaluation would start. The execution flow of stages is very much similar to that of [job](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=vsts&tabs=schema#job). I will take the example of a ring based geographic deployment to explain the basic flow. 

![IMG: Ring based deployment](https://github.com/zenithworks/azure-pipelines-yaml/blob/master/capture20181218115033741.png)

#### 1. On Pipeline Trigger:
*Eg. I want to begin  'Dev' stage execution as soon as a new run is triggered*
For easy on boarding, by default all stages will start on a pipeline trigger. The user however is free to override this by using dependsOn/manual keywords as mentioned below.

#### 2. Sequential:
*E.g. I want to begin 'Test' stage execution as soon as 'Dev' stage completes.*
Use 'dependsOn' keyword to specify a stage dependency:

```yaml
    - stage: Test
      dependsOn: Dev
 ```

#### 3. Fan-out:
*E.g. I want to begin 'Ring1a'(Africa), 'Ring1b'(Australia) as soon as 'Ring0'(internal users) is complete.*  

```yaml
    - stage: Ring1a
      dependsOn:
      - Ring0
    ...
    - stage: Ring1b
      dependsOn:
      - Ring0
```


#### 4. Fan-in:
*E.g. I want to begin 'Ring2'(Europe) as soon as both 'Ring1a'(Africa) and  'Ring1b'(Australia) are complete.* 

```yaml
    - stage: Ring2
      dependsOn:
      - Ring1a
      - Ring1b 
 ```

#### 5. Manual: 
*E.g. I do not want to start 'Ring-0'(internal users) automatically. I want to start it manually only after deployment to 'PPE' stage is successful and I have verified the artifacts and other configurations.* 

```yaml
- stage: Ring0
  manual: true  # stage would be started manually
  dependsOn: PPE
```


## Stage Conditions

These are conditions which need to pass for a stage to be executed. The stage conditions can be based on:
1.  Status of a 'dependsOn' stage.
2. Branch of a resource
3. Tags on a resource
4. Path of the changes done in a resource
5. version of a resource

#### 1. Status of 'dependsOn' Stage:
Similar to [job conditions](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/multiple-phases?tabs=yaml&view=vsts#conditions), a stage can run based on execution status of one or many 'dependsOn' stages.

```yaml
    stages:
    - stage: PPE
       jobs:
       - script: echo Hello from PPE
    - stage: RootCauseAnalysis
      dependsOn: PPE
      condition: failed()
      steps:
      - script: echo RootCauseAnalysis runs when PPE fails
```

```yaml
    - stage: X
      dependsOn: A, B
      condition: failed('A')
      steps:
      - script: echo X stage runs only when A fails
```

#### 2. Branch of a resource:
*E.g. All deployments to Ring0 should contain resources from 'releases' branch only.* 
The 'condition' construct  can be used to define this(no separate YAML construct required). However, resource branch metadata, if applicable, should be present as a variable during pipeline execution. Branch meta data is applicable for repositories, pipelines, build resources.
```yaml
resources:
  repositories:
  - repository: zenithWorksCore
    type: github
    name: zenithworks/core
stages:
- stage: RingO
  dependsOn: PPE
  condition: |
    and(
      succeeded(),    
      startswith(variables['resource.zenithWorksCore.branch'], 'refs/heads/releases/')
      )
```

#### 3. Tag on a resource:
*E.g. Resources being deployed to Ring0 should have 'QA-passed' tag.*
All the tags present on a resource should be available as a variable during pipeline execution. Tag meta data should be available for repositories, pipelines, build resources. Since there can be multiple tags on a resource, list of tags can be an array or 

```yaml
resources:
  pipelines:
  - pipeline: zenithWorksCore
    project: zenithWorks
    source: ZenithWorksCore-CI
stages:
- stage: RingO
  dependsOn: PPE
  condition: |
    and(
      succeeded(),    
      in(variables['resource.zenithWorksCore.tag'], 'QA-passed')
      )
```

```yaml
resources:
  containers:
  - container: zenithWorksCore
    type: Docker
    connection: myDockerRegistry
    image: zenithWorksCore
stages:
- stage: RingO
  dependsOn: PPE
  condition: |
    and(
      succeeded(),    
      in(variables['resource.zenithWorksCore.tag'], 'QA-passed')
      )
```

####Multiple conditions:
```yaml

```
