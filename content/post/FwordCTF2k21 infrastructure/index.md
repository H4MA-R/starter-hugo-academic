---
title: FwordCTF2k21 Automation scripts
subtitle: FwordCTF2k21

# Summary for listings and search engines
summary: FwordCTF2k21 Automation scripts
# Link this post with a project
projects: []

# Date published
date: "2021-10-13T00:00:00Z"

# Date updated
lastmod: "2021-10-13T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: [**Unsplash**](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/162443729_121061223368199_7099046546114959220_n.png?raw=true)'
  focal_point: ""
  placement: 2
  preview_only: false

authors:
- admin

tags:
- Article

categories:
- Scripts
---
In the first article we talked ablout the various automation scripts we used in FwordCTF 2k21 and this one we will be covering all the details about the infrastruction we worked  with during the whole ctf.
The ctf was awesome, it was an amazing experience, even better than the first edition even though it was really stressful for the first 3 hours at least because of a tiny issue that had a lot of damage we will talk about it later 

So let's start with some stats about our event. as we mentioned before, in this year's edition we had 1925 registered users and 1001 teams 
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/ctfd.png?raw=true)
 in addition to that we had a lot more traffic than the previous year and here are some stats from cloud flare:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/CloudFlare.png?raw=true)
As you see the traffic was huge and the participants didn't make it easy at all the DDOS attacks where remarkable xD thank god CloudFlare was in the rescue we forgot to take screenshots but we had around 40K threats on cloudflare stats but as KAHLA said in last year in his article:
"hackers can’t be hacked easily :p" Ahmed Belkahla Sep 22, 2020
So as I said a lot of user and heavy traffic needed a strong infrastructure to handle all that so will talk in details about the architecture and the diffrent componement and services in it.
**The main infrastructure**
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/infra.png?raw=true)
So in the schema above, it shows all the components of Fword CTF infrastructure and bellow are all the details:
-   Microsoft Azure as a cloud platform
-   Servers:
		-   2 * CTFD Instance: Ubuntu Server 20.04 LTS - Gen2 (**Standard E2s v3 - 2 vcpus, 16 GiB memory**) : a docker swarm Master Node + 1 Worker node
		-   Redis caching instance: Ubuntu Server 20.04 LTS - Gen2 (**Standard_D2s_v3 - 2 vcpus, 8 GiB memory**)
		-   MySQL server: Ubuntu Server 20.04 LTS - Gen2 (**Standard_D2s_v3 - 2 vcpus, 8 GiB memory**)
		-	2 * Ubuntu servers for the automation scripts:  Ubuntu Server 20.04 LTS - Gen2 (**Standard_DS1_v2 - 1 vcpu, 3.5 GiB memory**)
-	Azure LoadBalancer
-	CloudFlare
-	AWS S3 bucket to serve the files to download.
-	Sendgrid free account (25k free emails): We used Sendgrid for sending the certificates after the CTF.
Nowm let's break down each componement and why have we gone with that specific option.
For the cloud platform we have gone with microsoft for one and only one reason.
![enter image description here](https://i.imgflip.com/5rkonm.jpg)
Microsoft Azure was the only solution because we had the 100$ credit we get as students but we can't deny it was easy to use and it had a lot of usefull features
For the CTFd instances we initilly wanted to go with 4/8GiB servers with 2 replicas in each node (you can read about how we used docker swarm in last year article) but due to Microsoft Azure for students limitation we only could have the total of of 4 cores per account and we needed the ctfd nodes to be in the same account so finaly we opted for 2 16GiB 2 cores servers with 4 replicas in each node and of course we needed Azure LoadBalancer to load the traffic between the two servers
For the redis caching server and the MySQL server we gone with a 2 core and 8GiB servers to make sure everything went smothly 
Now for the discord bot and utomation script we gone for two 1 vcpu, 3.5 GiB, we know we could run them in our PC's but having a good internet connection was a must and the speeds in the servers were awsome.
As you noticed in the beggining of the article we used CloudFlre cdn as it is free nd easy to use.
For the static files storge we used AWS S3 bucket the same aas the last year
Now for SendGrid we tslked in details on why we gone with it in the automtion article
**Hosting the tasks**
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-24%2000_33_04-tasks6%20-%20Microsoft%20Azure.png?raw=true)
For tasks we used 6 Azure VMs: Ubuntu Server 20.04 LTS - Gen2 (**Standard E2s v3 - 2 vcpus, 16 GiB memory**), and every tsks was in a septrte docker container.
and for the deployment we used the same scripts as last year edition.
**managment and monitooring**
for the servers management we used termius like last because its really awsome and made our life easier we had seprate groups for the tasks servers and the infra servers
Main group:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-22%2001_10_28-Termius.png?raw=true)
Sub groups:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-22%2001_10_46-Termius.png?raw=true)
infra:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-22%2001_10_55-Termius.png?raw=true)
tasks:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-22%2001_11_08-Termius.png?raw=true)

now for the servers monitoring as we talked in the previous article we made our custome scripts to alerts us if anything goes wrong plus we used newRelic
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-10-22%2001_14_21-.png?raw=true)
New relic is a platform used for monitoring and you can customise ow you sow the dat it was awsome and feutures rich it alerted us when a server had  heavy load or if any problem accured in addition we could always monitor the servers stats like cpu and memory usage:
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20infrastructure/2021-08-28%2009_21_39-New%20Relic%20Navigator%20_%20New%20Relic%20One.png?raw=true)
