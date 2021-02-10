Example Voting App
=========

A simple distributed application running across multiple Docker containers.

Getting started
---------------

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). 

Contribution guidelines for newly joined developers
---------------------------------------------------

### Branching Strategy

What Is Branching Strategy:

The purpose of a branching strategy is to increase code stability and avaoid unnecessary conflicts.

* From which branch you should cut your feature branch?
* After code completion, in which branch you should be raising a PR (Pull Request) for code review and testing?
* After completing review and testing, in which branch should that feature branch be merged?

Why Branching Strategy Matters?

* You should not be merging buggy code to the production branch. 
* While merging to production branch you should encounter minimal merge conflicts. 
* The branch from which you're cutting your feature branch should be production-stabele.

One of the best brancing strategy that is being used in most places is the `master`, `develop`, and `feature` branch. 

#### master

* We can call it the production branch. Well tested, stable code lies here.
* It's the branch from which the last release should have gone and the next release should go. 
* It should only accept merges from the develop branch.
* We can have pipelines for releasing this branch (whenever this branch encounters a new merge, that pipeline autometically build and roll out the new release for production server.)

#### develop

* The branch at a level lower to `master`
* A developer who starts working on a feature cuts a fresh branch from this branch.
* Post completion of development/test/review they would raise a PR to the same branch, as this is the branch that is going to be released in the next relese.
* At the time of release a merge goes from this branch to the `master` branch and the `master` is the one that is release. 

#### feature

* A branch cut from `develop` to work upon a feature is planned in the next release.
* Generally a single developer works on `feature` branch.  

### Commit Guidelines

There are few common rules on how to write git commit messages. 

* Short (72 chars or less) summary in more detailed explanatory text. 
* The blank line separating the summary from the body is critical (unless you omit the body entirely).
* Write your commit message in the imperative: "Fix bug" and not "Fixedbug" or "Fixes bug."
* Describe what was done and why, but not how.

### Pre-Commit Hooks

The pre-commit script is executed every time you run git commit before Git asks the developer for a commit message or generates a commit object. You can use this hook to inspect the snapshot that is about to be committed. 

* Navigate to your local Git repository and go into the folder .git\hooks
* Create a file.
* Write a script to find the jira id 

This will be run whenever you add a new commit. 

### Git Rebase

Rebase is a utility that specializes in integrating changes from one branch onto another.
From a content perspective, rebasing is changing the base of your branch from one commit to another making it appear as if you'd created your branch from a different commit.
Internally, Git accomplishes this by creating new commits and applying them to the specified base. It's very important to understand that even though the branch looks the same, it's composed of entirely new commits.

### Git Cherry Pick

As we all know git cherry-pick is a powerful command that enables arbitary git commits to be picked by reference and appended to the current working HEAD. Cherry pick is the act of picking a commit from a branch and applying it to another. git cherry-pick can be useful for undoing changes. For example, say a commit is accidently made to the wrong branch. You can switch to the correct branch and cherry-pick the commit to where it should belong.

Cherry pick can be used during team collaboration, bug hotfixing, and undoing changes and restoring lost commits. 

How to use git cherry pick:
```bash
git cherry-pick commitSha
``` 

### Monorepo
 
Monorepo is to store all code in a single version control system (VCS) repository. The alternative, of course, is to store code split into many different VCS repositories, usually on a service/application/library basis.

More at https://medium.com/@mattklein123/monorepos-please-dont-e9a279be011b

## Linux Containers

The Linux stack uses Python, Node.js, .NET Core (or optionally Java), with Redis for messaging and Postgres for storage.

> If you're using [Docker Desktop on Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows), you can run the Linux version by [switching to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers), or run the Windows containers version.

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

## Windows Containers

An alternative version of the app uses Windows containers based on Nano Server. This stack runs on .NET Core, using [NATS](https://nats.io) for messaging and [TiDB](https://github.com/pingcap/tidb) for storage.

You can build from source using:

```
docker-compose -f docker-compose-windows.yml build
```

Then run the app using:

```
docker-compose -f docker-compose-windows.yml up -d
```

> Or in a Windows swarm, run `docker stack deploy -c docker-stack-windows.yml vote`

The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).


Run the app in Kubernetes
-------------------------

The folder k8s-specifications contains the yaml specifications of the Voting App's services.

First create the vote namespace

```
$ kubectl create namespace vote
```

Run the following command to create the deployments and services objects:
```
$ kubectl create -f k8s-specifications/
deployment "db" created
service "db" created
deployment "redis" created
service "redis" created
deployment "result" created
service "result" created
deployment "vote" created
service "vote" created
deployment "worker" created
```

The vote interface is then available on port 31000 on each host of the cluster, the result one is available on port 31001.

Architecture
-----

![Architecture diagram](architecture.png)

* A front-end web app in [Python](/vote) or [ASP.NET Core](/vote/dotnet) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) or [NATS](https://hub.docker.com/_/nats/) queue which collects new votes
* A [.NET Core](/worker/src/Worker), [Java](/worker/src/main) or [.NET Core 2.1](/worker/dotnet) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) or [TiDB](https://hub.docker.com/r/dockersamples/tidb/tags/) database backed by a Docker volume
* A [Node.js](/result) or [ASP.NET Core SignalR](/result/dotnet) webapp which shows the results of the voting in real time


Notes
-----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple 
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to 
deal with them in Docker at a basic level. 
