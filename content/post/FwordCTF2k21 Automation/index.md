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
draft: false

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

# FwordCTF2k21 Automation scripts
  

## Introduction
Last August our team organized FwordCTF2k21 and it was a successful event. And in this article, I am going to talk about all the automation scripts we used to assure that everything went accordingly from health checks to interacting with the participants.

Note: this article is for documentation purposes and all the links will be below.


### Summury

[1- Tasks Health check](##1)
[2- Discord Bot](##2)
[3- Certificates script](##3)
[4- Conclusion](##4)

# <a name="1"></a> 
## Tasks-health-check
So for the health checks and server monitoring, we used newRelic mainly and it was very helpful and it saved us many times but we wanted something that notifies us in discord since all the admins were mostly active in there so I wrote a simple script that when a task is down it will automatically send a message pinging all the admin with information about which task had problems

the script takes a JSON file with a certain structure for every task and tests the task status depending on the task's category.

the JSON file:
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/Untitled.png?raw=true)

The alert message will be this form:\
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/1.png?raw=true)
\
the scripts start by checking the task's type and according to that it will either try to send a request and wait for the correct response if it is a web task or it will try to connect with pwn tools if it's a service the if there is no response it will send the message above to Discord It contains the task name and category and the server and port that it's running on.

# <a name="2"></a> 
## Discord Bot
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/Fbot.png?raw=true)

Now for the discord bot, we wanted it to do two main tasks which are giving the welcome flag to the participants and announcing all the first blood. the first task was pretty much straight forward I just used the discord bot library. the bot has basics commands like help and ping to test the latency and of course the flag command to give the flags for the participants.
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/flag1.png?raw=true)

It listens for a private message and if it finds the word flag it will send it.
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/flag2.png?raw=true)

For the second task, I found a lot of solutions but the easiest for me at least was to use selenium to scrape the data needed for the first blood. the bot at first start with collecting all the tasks names when lunched and then every 3 minutes it checks for first blood if there is one marks the task as solved and sends a message to the discord server containing the team that had the first blood and the task name:
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/firstblood.png?raw=true)

btw credit to the CSICTF admin for his amazing article it helped me a lot(link below)
# <a name="3"></a> 
## Certificates script
In this year edition, we had more than 1000 participants and more than 400 teams that made it to the scoreboard so no way we were going to do the certificates manually and send them for the teams so I opted to write a script that generate all the certificates and send them via email to the teams.

So to generate the certificates a started with a template done by our talented designer Aptx and I used pillow library in python to put the team name, points, and ranking and the result was a pdf containing the certificate that will be attached to the email.
**Template**
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/cet1.png?raw=true)

**Result**
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/cert2.png?raw=true)

Now for the emails, I opted to use Sendgrid because I was most familiar with it and I worked with it before in the FwordCTF first edition

for those who don't know it Sendgrid is a platform used mainly for email marketing you can customize the emails and use the API key to use it in an automation script.

here are some stats about the emails we sent during the period of FwordCTF
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/sendgrid.png?raw=true)

For the emails template, we used one for the invitation before the CTF and one for sending the certificates.
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/mailtemplate.png?raw=true)
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/mail.png?raw=true)
# <a name="4"></a> 
## Conclusion
I know these are simple scripts but it helped us manage the CTF and made it a lot easier than doing all these tasks manually, these script had a lot of issues like for the bot if anything happens the data will be lost and it will starts the first blood from he brggind and also the health check can have a lot of flase alerts s for net year edition we set the goal to combine all these tasks in one bot that it will manage everything from the beginning of the CTF with no human interaction it will be more optimized and it will help us focus more interacting with the participants and doing more challenges.

In the next article, we will break down the main infrastructure and all its components and we will talk in detail about all the challenges and problems we faced and the solution we went with.
