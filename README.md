# Creating a Flocker Cluster using Docker and Ansible

At [Valtech Canada](https://www.valtech.com/)  we're always looking for new challenges and ways to improve our technology stack which is why I set my self onto the task of building a [Jenkins](https://jenkins.io/index.html) cluster using Docker's [Multi-Host Networking](https://docs.docker.com/engine/userguide/networking/get-started-overlay/)  , [Swarm](https://docs.docker.com/swarm/overview/) and [Compose](https://docs.docker.com/compose/overview/). The ultimate goal is to be able to run everything inside containers, which of course demands the ability o run a Jenkins Master and as many slaves as possible inside containers. So let me state my goals in a concise way:

    1 - The Jenkins Master can run in any node of the Swarm cluster and the deployment must be durable.
    2 - Jenkins slaves can run in any node of the Swarm Cluster and they must provide out of the box support for building Docker images as well as for running containers.

Goal #1 is very important and it's what this article is mainly about and there will be a follow up article on #2 since that goal by itself it's quite lengthy and not that simple. 

Let's break goal #1 into pieces:
    
    1 - The Jenkins Master can run in any node of the Swarm Cluster
    2 - The deployment of the Jenkins Master must be durable

The first item does not take much thinking, nor much time to implement, since all we need is a properly configured Swarm Cluster and that's about it. But having a durable deployment of the Jenkins master, well that's where the fun begins :).

But first, what does *durable deployment* means in this specific context? For me, it means that:

    1 - In the case of failure, like when a Swarm Node crashes, there should not be no data loss (and possible no data corruption). 
    2 - It should be as easy as possible to relocate the Jenkins Master to a different node on the Swarm Cluster while keeping all existing build configurations, jobs definitions, etc.

If you've been into Docker long enough you will know that those two statements above are not easy to deal with, specially in the context of a Swarm Cluster. Why? To start with we need to make sure that the Jenkins installation folder it's externalized, i.e it's placed inside a data volume so that changes made to the Jenkins configuration can survive across containers. But this is not good enough in the case of Swarm Cluster since the container might be scheduled for execution in any node of the Swarm Cluster. 

Luckily for us support for volume plugins was added as an [experimental feature in Docker 1.7.0](https://github.com/docker/docker/blob/master/CHANGELOG.md#170-2015-06-16) and made it to production grade [at version 1.8.0](https://github.com/docker/docker/blob/master/CHANGELOG.md#180-2015-08-11) . So, how exactly does volume plugins comes into rescue? Let's take a look at the [Volume Plugin documentation](https://docs.docker.com/engine/extend/plugins_volume/):

    Docker Engine volume plugins enable Engine deployments to be integrated with external storage systems, such as Amazon EBS, and enable data volumes to persist beyond the lifetime of a single Engine host.

which is a way of saying that data volumes are no longer bounded to a single host and could actually be reused across different hosts. Then all we need is a volume plugin that can follow the Jenkins Master container wherever the later gets deployed. Right? You can find a list of existing volume plugins [here](https://docs.docker.com/engine/extend/plugins_volume/) which I went through more than once and after weighting several factors, such as pricing, performance and project activity, I narrowed it down to two elements:

- [Flocker by ClusterHQ](https://clusterhq.com/flocker/introduction/)
- [OpenStorage](https://github.com/libopenstorage/openstorage)

Unfortunately I've not being able to use OpenStorage as much I'd love to mainly because of [this issue](https://github.com/libopenstorage/openstorage/issues/119) and also because I really wanted to give it a try running a Swarm Cluster on AWS (even after some asshole stole my AWS credentials I accidentally committed to GitHub and I ended up with a $2400 bill on my AWS account. But that's another story for different post). This leaves us with FlockerHQ.

# Enter Flocker

Somewhere alone this tool [introductory page](https://clusterhq.com/flocker/introduction/) you can read:

    Flocker manages Docker containers and data volumes together. When you use Flocker to manage your stateful microservice, your volumes will follow your containers when they move between different hosts in your cluster.

Et voilà! That's precisely what we need!! But, there is always a but, 








