---
date: 2018-03-29T20:57:54+08:00
title: Infra flight rules 
weight: 10
---

## What is the goal of Chaos Monkey?
Inspired by [PRINCIPLES OF CHAOS ENGINEERING](http://principlesofchaos.org/) and by my work in distributed system, with a focus on Spring Boot, I wanted to test the resulting applications better and especially during operation.

After writing many unit and integration tests, a code coverage from 70% to 80%, this unpleasant feeling remains, how our baby behaves in production?<br><br>
Many questions remain unanswered:
- Will our fallbacks work?
- How does the application behave with network latency?
- What if one of our services breaks down?
- Service Discovery works, but is our Client-Side-Load-Balancing also working?

As you can see, there are many more questions and open topics you have to deal with.

That was my start to take a deep dive into Chaos Engineering and I started this little project to share my thoughts and experience.

### inspired by Chaos Engineering at Netflix

This project provides a Chaos Monkey for Spring Boot and will try to attack your running [Spring Boot](https://projects.spring.io/spring-boot/) App.

>Chaos Engineering is the discipline of experimenting on a distributed system
in order to build confidence in the system’s capability
to withstand turbulent conditions in production.
> *principlesofchaos.org*

### Check your resilience

1. Are your services already _resilient_ and can handle _failures_? Don´t start a chaos experiment if not!
2. Check your monitoring and check if you can see the overall state of your system. There are many great tools out there to get a pleasant feeling about your entire system.
3. Define a metric to check a steady state about your service and of course your entire system. Start small with a service that is not so critical.
4. Of course, you can start in production, but keep in mind...

### How does it work?
Spring Boot Chaos Monkey is a small library which you can integrate as a dependency into your existing application. As long as you don't use your application with the profile **"chaos-monkey"**, nothing will happen.

As you can see, you don't have to change the source code!

If Spring Boot Chaos Monkey is on your classpath and activated with profile name "chaos-monkey", it will automatically scan your application for all classes annotated with any of the following Spring annotations:

- @Controller
- @RestController
- @Service
- @Repository

By configuration you define which assaults and watcher are activated, per default only the @Service watcher and the latency assault are activated.

#### Example - single Spring Boot application
Let's say you built a standalone Spring Boot application. For example, there is a service annotated with Spring @Service annotation and some other components. Now we want to attack our service component.

Let´s activate Chaos Monkey for Spring Boot, only 2 steps are required.

1. Added Chaos Monkey for Spring Boot to your dependencies.

```xml
<dependency>
  <groupId>spring.boot.chaos.monkey</groupId>
  <artifactId>spring-boot-chaos-monkey</artifactId>
</dependency>
```
2. Start your app with profile "chaos-monkey"

```sh
java -jar your-app.jar --spring.profiles.active=chaos-monkey
```

Chaos Monkey for Spring Boot will attack your @Service classes and will randomly add some latency to all `public` methods.

There are some more assaults and watcher, that can attack your app.

### Watcher
A watcher is a Chaos Monkey for Spring Boot component, that will scan your app for a specific type of annotation.

Following Spring annotation are supported:
- @Controller
- @RestController
- @Service
- @Repository

With the help of [Spring AOP](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html), Chaos Monkey recognizes the execution of a public method and will either not execute any action or start one of its assaults. You can customize the behave by configuration.

### Assaults

Following assaults are actual provided:
- Latency Assault
- Exception Assault
- AppKiller Assault

You can customize the behave by configuration.

### Configuration

| Property        | Description                | Values  | Default |
| ------------- |------------------| -----:|----:|
| chaos.monkey.assaults.level | How many requests are to be attacked.<br> 1 each request, 5 each 5th request is attacked | 1-10 | 5
|chaos.monkey.assaults.latencyRangeStart | Minimum latency in ms added to the request| Integer.MIN_VALUE, Integer.MAX_VALUE  | 3000
|chaos.monkey.assaults.latencyRangeEnd | Maximum latency in ms added to the request| Integer.MIN_VALUE, Integer.MAX_VALUE  | 15000
|chaos.monkey.assaults.latencyActive | Latency assault active| TRUE or FALSE | TRUE
|chaos.monkey.assaults.exceptionsActive | Exception assault active| TRUE or FALSE | FALSE
|chaos.monkey.assaults.killApplicationActive | AppKiller assault active| TRUE or FALSE | FALSE
chaos.monkey.watcher.controller | Controller watcher active| TRUE or FALSE | FALSE
chaos.monkey.watcher.restController | RestController watcher active| TRUE or FALSE | FALSE
chaos.monkey.watcher.service | Service watcher active| TRUE or FALSE | TRUE
chaos.monkey.watcher.repository | Repository watcher active| TRUE or FALSE | FALSE

## What are "flight rules"?

A guide for astronauts  about what to do when things go wrong.

>Flight Rules are the hard-earned body of knowledge recorded in manuals that list, step-by-step, what to do if X occurs, and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures. [...]

— Chris Hadfield, An Astronaut's Guide to Life.


### With Blobs

#### Cannot find blob named XXX
	 Building a release from directory '/Users/yremmet/DEV/bosh/mongodb-bosh-release':
	 Cannot find blob named 'create-replset/6ff7a835c431e94d47476107f8c93c2d5c27ed62' with SHA1 '0692dd85bc1486d8b1b1d076f1a43911de02aa8f'

	 Exit code 1

When creating a new release, this is propably due to some sha mismatch after you recated your bosh environment.
Delete `.dev_builds` and try again.

#### Missing Blob
	  - Constructing packages from directory:
      - Reading package from '/workspace/releases/osb-kafka-deployment/packages/zookeeper':
          Collecting package files:
            Missing files for pattern 'zookeeper/zookeeper-3.4.10.tar.gz'
You propably forgot a step when adding the new blob. 

1. `bosh add-blob`   
To add blob to local blob store.
2. `bosh upload-blobs`   
To upload local blobs to the remote blob store
3. `git commit`
To add the blob ids to 

### With deployments

#### Job not found in template table
	L Error: Job 'node_exporter' not found in Template table
This often occurs when an incorrect version of the release is deployed. Check the if the version you are deploing contains the mentioned job.   
If your deployment is pointing to the version `latest`, make sure the director doesn't have a higher version of your release than you create.

### Colocated job is already added to instance group
	L Error: Colocated job ‘node_exporter’ is already added to the instance group ‘postgres’.
Check the runtime configuration. The mentioned job will run on all VMs.

### Bad file descriptor
Trying to upload release and getting: 

```shell
Uploading release file:
  Performing request POST 'https://172.16.106.4:25555/releases?':
    Performing POST request:
      Seeking to beginning of seekable request body during attempt 0: seek /root/.bosh/tmp/bosh-platform-disk-TarballCompressor-CompressSpecificFilesInDir232448384: bad file descriptor
```

Actual problem: environment variables for Bosh director not set:

```shell
Exit code 1
root@host:/workspace/releases/osb-bosh-postgresql# bosh vms
Expected non-empty Director URL

Exit code 1
Check the runtime configuration. The mentioned job will run on all VMs
```
Solution: login

### Credhub cannot generate password
```shell
Preparing deployment: Preparing deployment (00:00:01)
                   L Error: Config Server failed to generate value for '/Bosh Lite Director/kubo/kubo-admin-password' with type 'password'. Error: ''
Task 15 | 21:30:09 | Error: Config Server failed to generate value for '/Bosh Lite Director/kubo/kubo-admin-password' with type 'password'. Error: ''
````

It turns out that the path for credhub is not allowed to contain spaces:

````shell
horst:kubo-deployment jhiemer$ credhub generate -n '/Bosh Lite Director/kubo/kubo-admin-password' -t 'password'
Credential names may only include alpha, numeric, hyphen, underscore, and forward-slash characters. Please update and retry your request.
````
Works:

````shell
horst:kubo-deployment jhiemer$ credhub generate -n '/BoshLiteDirector/kubo/kubo-admin-password' -t 'password'
id: f829e692-d98e-400e-8b14-9ac20d558c81
name: /BoshLiteDirector/kubo/kubo-admin-password
type: password
value: *******************
version_created_at: 2017-12-30T21:50:33Z
````
### Script not working (e.g. pre-start script)
```shell
Updating instance my-instances: my-instances/0d08eec9-9d4e-4507-b4fe-ba0b7b20f628 (0) (canary) (00:00:14)
L Error: Action Failed get_task: Task 3a65c031-3bd6-4cae-44cc-ab7cdc58e871 result: 1 of 2 pre-start scripts failed. Failed Jobs: my-cool-job. Successful Jobs: bosh-dns.
````

In most cases, this means you missed to switch to bash at the start of your script. Add the following to the beginning of your script and all works out well...

```shell
#!/usr/bin/env bash
````
### Getting blob from inner blobstore (though it is a package..)
```shell
-- Failed downloading 'erlang/dbcae8592bdecbf4d89b02e2bc18222ff3a41bdd' (sha1=b9c568a1dfde2860501f36e6174da2b58f4f483d)


-- Failed downloading 'rabbitmq-server/12e135fad7a59bfcf017ab297403ebe1587c81c8' (sha1=bdef8803fc5f4094c2bbde9bb94cc9731c030095)


-- Failed downloading 'rabbitmq-common/9232e8ac16db2e9a2fa8d65aaca25c3c69b1b17f' (sha1=7b7a3f080eaaa3d6cde0cd7fc00631761cc058d0)


-- Failed downloading 'rabbitmq-server/696215db5554d02c647acbd8c7f5b6f6ce8509c0' (sha1=49f96f0678fcddcc08ab72c277dbc3fa930ddb12)


-- Failed downloading 'smoke-tests/1d60f612faac462576d37c879b7d552523ea47fe' (sha1=c3b6288109951727b9b60fbaf8ff7db2e02d0e80)


Building a release from directory '/Users/johanneshiemer/Development/osb-framework/bosh/osb-bosh-rabbitmq':
  - Downloading blob '35aeb965-ca9e-4f75-787f-463d968cde3a' with digest string 'b9c568a1dfde2860501f36e6174da2b58f4f483d':
      Getting blob from inner blobstore:
        Getting blob from inner blobstore:
          NoSuchKey: The specified key does not exist.
        status code: 404, request id: B9589CF4A8670C06, host id: PIFDeFSFJvPbVtouTlVuavI4L36MOH4RyamftqfsYc4SLSdDqrcTmzq65HAefhVPKWjKZTW8Q+E=
  - Downloading blob 'bbfaf028-2252-44ee-749d-09f0369ff53c' with digest string 'bdef8803fc5f4094c2bbde9bb94cc9731c030095':
      Getting blob from inner blobstore:
        Getting blob from inner blobstore:
          NoSuchKey: The specified key does not exist.
        status code: 404, request id: 249B4CB372E6CABB, host id: Mcw5ivUreKfyJQIuKpYIYvKoVgS6s0FXTx8YPHHy1mbHgL9Kwyjs60l8qPmePxA1ZCQD2E2Zd/I=
  - Downloading blob 'e1cd036e-168f-4587-4072-18e182785bd2' with digest string '7b7a3f080eaaa3d6cde0cd7fc00631761cc058d0':
      Getting blob from inner blobstore:
        Getting blob from inner blobstore:
          NoSuchKey: The specified key does not exist.
        status code: 404, request id: 4D433D8E6790E85A, host id: VZP2hUiY1yxD+7/XleE73FvKEbBuZNyYJQwIyGS3wrPYqDNoado4D9BHurB01KIWPVy4NXXNNTs=
  - Downloading blob '2ed93f00-e3fa-499c-53e0-82373d6b2d05' with digest string '49f96f0678fcddcc08ab72c277dbc3fa930ddb12':
      Getting blob from inner blobstore:
        Getting blob from inner blobstore:
          NoSuchKey: The specified key does not exist.
        status code: 404, request id: 376EC8EE47564623, host id: bJmKhs3zlt2r++Nrams7NOZ2YXafj9/g7fmmx6wzLlETuEQIrnZrwGY/Aj5mIQaAumLWvrukAew=
  - Downloading blob '9d79b054-48f6-40f5-5934-ac62dae19ef9' with digest string 'c3b6288109951727b9b60fbaf8ff7db2e02d0e80':
      Getting blob from inner blobstore:
        Getting blob from inner blobstore:
          NoSuchKey: The specified key does not exist.
        status code: 404, request id: 4823E8F84D6B904C, host id: 1+8idwXlGDNLnB6nQ7TWDSeu1vtb2JyTucP5QnbCM/tl5H/DW+RSJql7HlHfp3eslO+SiY40n6o=

Exit code 1
```

Yes this might be an inner blobstore. And it is the blobstore of your release, which you can find under `.final_builds` in the root of your release.

Just run:

`rm -r .final_builds/`

And you should be able to run `bosh create-release --force` again

> I´m still working on this page and the documentation!