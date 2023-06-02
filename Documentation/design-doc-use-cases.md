# Summary

Regardless from the Git server, git Reposiory optimization is key for maintaining high standard performance in particular when working with monorepos. This design doc is not strictly related to Gerrit managed repository, but any Git repository independently from the server.

# Background

## Definitions

* *Git repository analysis*: check for number and size of refs, objects, existance and size of blobs in the repository, etc
* *Git repository optimization*: series of activities, to improve performance, carried out on a git reposiroty, for example: GC, repack, etc

## Phase 1: Metrics collection

In the OS community there are currently several tools for one-off analysys of Git repositories ([git-sizer](https://github.com/github/git-sizer), [git-count-objects](https://git-scm.com/docs/git-count-objects), etc).

Some Git server also provide tools to systematically collect metrics which describe the shape of a repository (Gerrit [git-repo-metrics](https://gerrit.googlesource.com/plugins/git-repo-metrics/)).

## Phase 2: Repositiory optimization

The orchestration of optimization activities on different repositories can be done via complex [bash scripts](https://gerrit.googlesource.com/aws-gerrit/+/refs/heads/master/maintenance/git-gc/scripts/utils.sh) scheduled periodically.

Those scripts carry several piece of logic, like:
* making sure only one operation is running at a time
* pick operational parameters
* produce metrics before and after a run

Some Git servers have their own plugins to orchestrate GC runs, for example: ([Gerrrit gc-conductor](https://gerrit.googlesource.com/plugins/gc-conductor/+/refs/heads/master/src/main/resources/Documentation)).

## Issues

* The two phases are not linked at all, while phase #1 should feed phase #2
* Decisions on the operation to perform are taken statically on a case by case, which is not scalable
* Bash scripts implementing a complex logic are fragile
* Git server specific tools are limited in scope
* Metrics are difficult to collect in a systematic way
* In general all the process makes automation difficult

# Primary use cases

* As a Git server admin I want to...
    * manage a single tool for all my Git servers
    * easily configure parameters to trigger a repository optimization cron based
    * easily configure parameters to trigger a repository optimization "threshold" based (for example if repoParamA > X => run GC)
    * monitor the success of the repository optimization activities
    * easily debug issue when a problem accours during a repository optimization activity
    * be alerted when a _crucial_ repository optimization activity fails
    * monitor the shape of my repository

# Secondary use cases

* As a Gerrit admin I want to...
    * see the history of the last X run
    * easily integrate the tool in my existing architecture

# Acceptance criteria

* The repository optimization tool has to be Git server implementation agnostic
* The repository optimization should be automatic after the initial setup
* No optiomizations triggers should be lost in case of temporary system unavailability
* The repository optimization shouldn't affect the normal functionality of the Git server