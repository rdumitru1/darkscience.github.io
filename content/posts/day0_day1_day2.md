+++
title = 'What is all about Day0, Day1 and Day2'
date = 2024-10-09T18:21:24+02:00
draft = false
+++

I recently come accross the Day 2 operations when discussing about joining a new project, and I realized that I did not encountered terminology until now.
So I invested some time to understand what it means, and in short Day 2 operations refers to the ongoing tasks and responsibilities involved in managing and maintaining the reliability, availability, and performance of ongoing software development and production environments.
Day 2 operations keep runtime from becoming downtime.

The below information is what I gathered from different sources.

In IT, the terms Day 0/Day 1/Day 2 refer to different phases of the software life cycle.

**Day 0** it represents the design phase, during which project requirements are specified, and the architecture of the solution is decided.

**Day 1** involves developing and deploying software that was designed in the Day 0 phase.
In this phase we create not only the application itself, but also its infrastructure, network, external services and implement the initial configuration of it all.

**Day 2** is the time when the product is shipped or made available to the customer. Here, most of the effort is focused on maintaining, monitoring and optimizing the system.
Analyzing the behavior of the system and reacting correctly are of crucial importance, as the resulting feedback loop is applied until the end of the application’s life

## Day 0 - boring, but essential
**Day 0 is often overlooked** because it can be boring, but that doesn’t diminish its importance. A successful software product is the result of a thorough planning and design process.
It is necessary to carefully plan the architecture of a system or app and the resources needed (CPUs, storage space, RAM) to get it up and running.
Secondly, you should define measurable milestones leading to the realization of a project goal. Each milestone should have a precise date. This helps measure the progress of the project and determine if you are running late with the schedule. All project time estimates should be based on the probability and not be merely best-case scenarios.

## Day 1 - where things get creative
For developers and project owners, **Day 1 is definitely the most interesting phase**. The initial design is brought to life and an infrastructure is created based on the project’s specifications.
In order to become truly cloud-native, best practices must be observed. Guidelines such as [Twelve-Factor Apps methodology](https://en.wikipedia.org/wiki/Twelve-Factor_App_methodology) can be followed.
Using cloud means that you should stick to an important practice of [the DevOps lifecycle phases:](https://www.simform.com/blog/devops-lifecycle/) Continuous Integration/Continuous Delivery.
There are several classes of tools that can be used during **Day 1**. They can be grouped by the problems they solve. “Infrastructure-as-a-code” **(Terraform)** should be mentioned at the top of the list. IaaC manages the operations environment just as the applications or code are. This approach allows DevOps best practices, including version control, virtualized tests, and continuous monitoring to be applied to the code driving the infrastructure.
The second class of tools is related to the container orchestration systems that automate the deploying and managing of containers. Kubernetes and its implementations, such as Google Kubernetes Engine and Amazon’s Elastic Kubernetes Service, are essential.
There are a variety of tools to make CI/CD processes more efficient and easier to manage. You can choose from cloud-native tools, such as:
* [Github Actions](https://github.com/features/actions)
* [Gitlab ci](https://docs.gitlab.com/ee/ci/)
* [Jenkins](https://www.jenkins.io/)
* [Azure DevOps](https://azure.microsoft.com/en-us/products/devops/)

## Day 2 - daily operations routine
Once the software is ready, it goes live and customers start using it. **Day 2 begins here**, and elements including software maintenance and customer support are introduced.
The software itself is going to evolve in order to adapt to the changing needs and customer requirements. **During Day 2, the main focus is on establishing a feedback loop.** We monitor how the app is working, gather feedback from users and send it to the development team, which will implement it in the product and release a new version.
**Application performance monitoring (APM)** class groups software that helps IT admins monitor app performance and thus provide a high-quality user experience. Among many solutions on the market, we can name:
* [Datadog](https://www.datadoghq.com/)
* [Dynatrace](https://www.dynatrace.com/)
* [Prometheus](https://prometheus.io/docs/introduction/overview/)
