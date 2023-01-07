# Inventory Model
# REF https://www.ibm.com/cloud/blog/implement-a-ci-cd-pipeline-in-ibm-cloud-with-tekton-and-argo-cd
Introduction to the inventory model, it's role and purpose in the CI/CD process.

- [Inventory Structure and Content](#inventory-structure-and-content)
- [Inventory Entry Format](#inventory-entry-format)
- [Inventory Workflow](#inventory-workflow)
  - [CI writes](#ci-writes-to-inventory)
  - [Promotion](#promotion)
- [Use Cases](#Use-cases)

## Inventory Structure and content

Inventory model keeps track of the following information:

- Details of the built artifact (pipeline run, commit SHA).
- Signature of the built artifact.
- The artifact that is to be deployed.

Based on this data, we can keep track of related evidence during the artifact build and deployment lifecycle.

### Branches

The Inventory is stored in a git repo as git helps to keep track of the changes ensuring auditability.

The main branch (`master`) is updated by the CI pipeline, while other branches are used for the various target deployment environments. 

### Content

The Inventory contains an Inventory entry for every artifact that participates in a deployment. 

One Inventory entry points to a single artifact.

Inventory entries are JSON files, [see the format below](#inventory-entry-format). The file name is the entry name, it can be structured in folders. 

## Inventory Entry Format

The `Entry` type represents the schema of the Inventory entry (using typescript syntax but it's fairly easy to translate it to eg. JSON Schema):

```ts

interface Entry {
  repository_url: string;
  artifact: string;
  build_number: number;
  commit_sha: string;
  name: string;
  pipeline_run_id: string;
  version: string;
  signature: string;
}

```

##### Example 

```

{
  "repository_url": "https://us-south.git.cloud.ibm.com/user/java-spring-app.git",
  "artifact": "binary-20210318042338.tar.gz",
  "build_number": 2,
  "commit_sha": "218913eb78b3c1b8cde2573a66998ac04848f529",
  "name": "java-spring-app",
  "pipeline_run_id": "96fc6b5e-d491-43bc-ada5-e03e17538c0a",
  "version": "v1",
  "signature": "3cb49a1f235e81a11a6abab55713172a" 
}


```


## Inventory Workflow

### CI writes to Inventory

The master branch is populated from CI builds. The last commit in the master branch has all the metadata related to the build and its output.


### Promotion

Promoting to a target branch happens by creating a pull request. The PR contents will fill the Change Request fields for change management service if they are integrated. When the PR is reviewed, the promotion PR can be merged.

## Use cases

#### 1. First update to master branch

> 0. Set up inventory. 
> 1. CI pipelines fill the `master` branch with Inventory entries
> 1. CD pipeline starts:  
>    - Pipeline will tag the latest commit with the Pipeline Run ID 
>    - Pipeline picks up the content of the `master` branch from that tag 
>    - Since this is a first promotion to a new target, deployment will contain all items
>      and will attempt to deploy.

#### 2. Rollback

> 1. If there a problem, a deployed application has to be rolled back to a previous version on `prod`
> 1. DevOps pick a point in the Inventory to roll back to.
> 1. All the commits are reverted until that point and promoted as a PR
> 1. The Rollback Promotion PR get merged
> 1. CD pipeline starts:
>    - Pipeline picks up the contents of the prod branch from the tag
>      and will attempt to deploy.
