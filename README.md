# caf-component-cloud-pipelines

---
## Introduction
#### TODO: add an introduction here

---
## General Pipeline Requirements
1. All git repositories which will be managed by a pipeline must contain a seed pipeline yaml file
     - this seed pipeline file's only purpose is to install the dso platform by running a `make configure`, and
     - call the intended pipeline yaml file that was just pulled down dynamically.
     - this seed pipeline file is the entry point to integration with this dso platform pipeline code
2. Pipelines with stages must be used.
     - for example in AWS a Pipeline resource cannot be called with arguments, thus AWS Codebuild is called first
       which accepts ARGS and it is used to trigger a Pipeline, where that Pipeline's configuration was defined at the
       time of its creation by Terraform
3. All pipeline yaml and the shell scripts that they call must be client and application agnostic.
4. When a pipeline yaml file requires the use of scripts as opposed to native yaml functions:
   
     - these yaml files should only call 1 shell script with arguments,
   
     - where these arguments are populated by global environment variables that are either supplied,
   
       - directly by the configuration of the pipeline, or
   
       - shell script above that call of the script that derives the needed values and sets that as global
         environment variables.
5. The intent of this project is to keep libraries of pipeline yaml and shell logic structured, organize, dry, and
composable.
   
    - In the current directory and file structure there are examples of will be real world implementations of shell script 
      that is organized.
    - Any additional of native yaml functions is required to all be structured and organized following the current
      patterns
6. All functions with the directory `local` in the tree are only permitted to use local scoped variables which in bash
can only be set inside of functions.
   - the only files containing functions that may exist must be named `functions.sh` and nested in a meaningful directory
     path following the current pattern
   - no script inside of `functions.sh` files may contain logic outside a function except for: `set -ex` 
   - a `function.sh` file inside a directory where that same directory contains other directories is implied to be
     common to its parent directory
     - a `function.sh` file in a nested directory related to a technology group or tool is to be scoped to its nearest
       parent directory
7. Global function files, in-line script within pipeline yaml files, or Spec Scripts where Spec Scripts provide a script
   called by a pipeline yaml file are the only place where global environment variables may be used or created.

---
## General organizational pattern of directories and files
``` text
pipeline_provider ....................... # this directory to hold the providers of pipeline tools such as aws, azure,
                                          # gcp, and github
  aws ................................... # this directory holds directories where the name starts with the pipeline
                                          # tool and ends with the cloud they interact with
    codebuild ........................... # this directory contains reuseable pipeline yaml and the scripts they depend
                                          # on
      scripts ........................... # this directory contains global/local functions & scripts where the name maps
                                          # 1 to 1 with with the pipeline.yml in the specs that calls it
        global .......................... # this director contains collections of functions grouped by a tools that will
                                          # be called by spec-scripts that set or depend upon global vars provided by
                                          # the pipeline that runs them
          pipeline ...................... # this directory contains collections of global functions specific to running
                                          # in AWS codebuild for example as implied by its location in the directory
                                          # tree where it is intended to be called by spec-scripts that run in an AWS
                                          # pipeline stage
            common ...................... # this directory contains only 1 functions.sh file with functions that are
                                          # common to AWS Codebuild running in AWS Pipeline stages
              function.sh ............... # this directory contains global functions that are common to any pipeline
                                          # stage or action, such as a deploy stage that can deploy to any environment
                                          # type further these functions are specific to pipeline and not for example to
                                          # terraform logic that would run in a pipeline but be grouped in the same
                                          # parent directory under another directory named terraform
          example_tool .................. # there could be many additional directories located in:
                                          # `pipeline_provider/aws/codebuild/scripts/global`
                                          # such as terraform, terrgrunt, etc.
                                          # each of these as implied by the directory tree would hold at least a common
                                          # sub-directory with logic related directly to the tool and could hold
                                          # non-common directories that may be more specific
          functions.sh .................. # this functions.sh file directly under the directory global would be for
                                          # functions that are common to any tool that may live under global
        local ........................... # this directory contains collections of function called only by global or
                                          # spec-scripts only local scoped variables in a function are permitted which
                                          # would be passed in as ARGS to the shell functions further this directory
                                          # could hold functions grouped by a specific class of technology such as tools,
                                          # cloud resource groups such as aws cli commands for ECS, etc.
          pipeline ...................... # this directory contains collections of directories that hold functions
                                          # specific to running in pipeline stages as defined by the directory path.
                                          # the nested functions that live here are intended to be called by global
                                          # functions or spec-scripts that run in pipeline stages
            common ...................... # this directory contains only 1 functions.sh file which is common to all
                                          # pipeline stages
              functions.sh .............. # this functions.sh file contains local functions that are common to any
                                          # pipeline stage
          functions.sh .................. # this functions.sh file contains local functions that are common to anything
                                          # run by codebuild
        spec-scripts .................... # this directory contains 2 directories called actions and events
                                          # each of these 2 directories will contain scripts called directly by a
                                          # pipeline.yml, which then calls global functions which finally use local
                                          # functions the directory events is for a pipeline stage that is triggered by
                                          # an event such as a PR or merge, where as actions is for a pipeline stage
                                          # that performs an action such as a deploy or update of properties but was in
                                          # most cases was triggered by another stage
          actions ....................... # this directory contains collections of global scoped scripts that would be 
                                          # called directly from a pipeline yaml file that has the same name
                                          # an example would be: simulated-merge.yml would have one line of script that
                                          # calls the script, simulated-merge.sh
                                          # action are pipeline stages that are not triggered by events such as git
                                          # webhook events or cloudtrail events and in most cases called by other
                                          # pipeline stages
            pipeline .................... # this directory will hold scripts that are common pipeline actions such as a 
                                          # simulated-merge.sh or collections of the scripts grouped by a technology
                                          # such as the Launch DSO Platform where for example the DSO Platform will need
                                          # to install its depedencies
              dso_platform .............. # this directory holds global scripts called directly by a pipeline stage from
                                          # a pipeline yaml file with the same name that is specific to dso platform
                                          # actions such as install-dependencies.sh
                install-dependencies.sh.. # script called by install-dependencies.yml from a pipeline stage that
                                          # installs the dso platform dependencies
              simulated-merge.sh ........ # script called by simulated-merge.yml from a pipeline stage that commonly
                                          # simulates a merge of feature branch into main
          events ........................ # this directory contains collections of global scoped scripts that would be
                                          # called directly from a pipeline yamle file that has the same name
                                          # an example would be: push-event.yml would have one line of script that
                                          # calls the script, push-event.sh
                                          # events are pipeline stages that are triggered by events such as git webhook
                                          # events or cloudtrail events
            pipeline .................... #
              push-event.sh ............. #
              pull-request-event.sh ..... #
              merge-event.sh ............ #            
      specs ............................. # contains pipeline yaml filesthat live under the directories actions or events
                                          # that map 1 to 1 with scripts that live in:
                                          # `pipeline_providers/aws/aws_pipeline-to-aws_cloud/codebuild/common/scripts`
                                          # these are the pipeline yaml files that will be called by AWS codebuild stages
        actions ......................... # this directory contains collections of global scoped pipeline yaml files that
                                          # would be called directly from a codebuild stage that would then call a
                                          # single script where that scripts file name maps 1 to 1 with this pipeline
                                          # yaml file's name
                                          # action are pipeline stages that are not triggered by events from git
                                          # webhooks or cloudtrail for example, but usually by other pipeline stages
          pipeline ...................... # this directory will hold pipeline yaml files that are common pipeline stage
                                          # actions such as a simulated merge 
            simulated-merge.yml ......... # this is a pipeline yaml file that is common to all pipelines where it would
                                          # be required to simulate a merge
                                          # this pipeline yaml file would call the 1 to 1 mapped spec-script called:
                                          # simulated-merge.sh
          dso-platform .................. # this directory will hold pipeline yaml files that are dso-platform pipeline
                                          # stage actions such as a install dependencies
            install-dependencies.yml .... # this is a pipeline yaml file that is needed to install the dso platform's
                                          # dependencies
        events .......................... # this directory contains collection of pipeline yaml files grouped by a
                                          # technology group such as common pipeline events like: push, pull request, or
                                          # merge
                                          # these pipeline yaml files would be directly called by the pipeline stage
          pipeline ...................... #
            push-event.yml .............. #
            pull-request-event.yml ...... #
            merge-event.yml ............. #                       
    templates ........................... # this directory contains very high level templates that would be common to
                                          # anything under the aws folder in such as the pipline seed yaml that would be
                                          # committed to each git repository that would then call additional libary 
                                          # pipeline yaml files that are pulled in dynamically
      buildspec-seeds ................... # this directory contains seed pipeline yaml files that bootstrap this project
                                          # with the full pipeline framework
```
