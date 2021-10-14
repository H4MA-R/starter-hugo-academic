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
  

## Introduction
Last August our team organized FwordCTF2k21 and it was a successful event. And in this article, I am going to talk about all the automation scripts we used to assure that everything went accordingly from health checks to interacting with the participants.

Note: this article is for documentation purposes and all the links will be below.


### Summury

[1- Tasks Health check](##1)\
[2- Discord Bot](##2)\
[3- Certificates script](##3)\
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
the scripts start by checking the task's type using the data in the JSON file
```python=
def check(task):
	if task["type"] == "web":
		link = task['server']+":"+task["port"]
		try:
			r = requests.get(link)
			if r.status_code == 200:
				if task["out"] in r.text:
					return 1
				else:
					return 0
			else:
				return 0
		except:
			return 0
	elif task["type"] == "service":
		try:
			r = remote(task["server"],task["port"])
			r.send(task["in"])
			data = r.recv()
			if data in task["out"]:
				return 1
			return 0
		except:
			return 0
	pass
```

then according to the  type it will use either pwntools or python requests to test the task.

for the web tasks it just send a GET request and if the status code is 200 it checks the output if it is like it suppose to do.
Now for the services tasks it uses pwntools library and connect to server and if the response has the output in the jason file then everything is ok.

# <a name="2"></a> 
## Discord Bot
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/Fbot.png?raw=true)

Now for the discord bot, we wanted it to do two main tasks which are giving the welcome flag to the participants and announcing all the first blood. the first task was pretty much straight forward I just used the discord bot library. the bot has basics commands like help and ping to test the latency and of course the flag command to give the flags for the participants.
```python=
@bot.command()
async def flag(ctx):
    await ctx.send("Noooo not here the good stuff is a secret :shushing_face: \nhit me up in my dms(message the bot privately)")
```
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/flag1.png?raw=true)

It listens for a private message and if it finds the word flag it will send it.
```python=
@bot.listen()
async def on_message(message):
    if "flag" == message.content.lower():
        try:
            if message.channel.id == message.author.dm_channel.id:
                #await message.channel.send('**THE CTF HASN\'t STARTED YET**')
                await message.channel.send('**HAVE FUN**\nFwordCTF{Welcome_To_FwordCTF_2021}')
                await bot.process_commands(message)
        except:
            pass
```
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/flag2.png?raw=true)

For the second task, I found a lot of solutions but the easiest for me at least was to use selenium to scrape the data needed for the first blood. 
The bot at first start with collecting all the tasks names when lunched and storing all the data in an array.
```python=
@bot.event
async def on_ready():
    await bot.change_presence(activity=discord.Activity(url="https://fword.tech/"))
    print('The Bot is Ready')
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')

    usernameStr = '###CTFd_CREDS###'
    passwordStr = '###CTFd_CREDS###'

    browser = webdriver.Chrome(ChromeDriverManager().install(), options=options )

    print('loading')
    browser.get(('https://ctf.fword.tech/login'))
    print('loaded')
    time.sleep(20)
    username = browser.find_element_by_id('name')
    username.send_keys(usernameStr)
    password = browser.find_element_by_id('password')
    password.send_keys(passwordStr)

    submitBtn = browser.find_element_by_class_name('btn')
    submitBtn.click()
    browser.get(f'https://ctf.fword.tech/api/v1/challenges')
    html = browser.page_source
    time.sleep(2)
    html = html[html.index('{'):html.rindex('}')+1]
    y = json.loads(html)
    for i in range(len(y['data'])):
        ch[y['data'][i]['id']] = {'name':y['data'][i]['name'],'solved': False}
    print(ch)
    await firstBlood.start()
```
And then every 3 minutes a bot tasks loop checks for first blood if there is one marks the task as solved and sends a message to the discord server containing the team that had the first blood and the task name:
```python=
@tasks.loop(seconds=180)
async def firstBlood():

    allSolved = True
    keys = dict.keys(ch)
    for i in keys:
        if not(ch[i]['solved']):
            allSolved = False

    if(allSolved):
        return
    
    channel = bot.get_channel(int('###CHANNEL_ID###'))

    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')

    usernameStr = '###CTFd_CREDS###'
    passwordStr = '###CTFd_CREDS###'

    browser = webdriver.Chrome(ChromeDriverManager().install(), options=options )

    print('loading')
    browser.get(('https://ctf.fword.tech/login'))
    print('loaded')

    username = browser.find_element_by_id('name')
    username.send_keys(usernameStr)
    password = browser.find_element_by_id('password')
    password.send_keys(passwordStr)

    submitBtn = browser.find_element_by_class_name('btn')
    submitBtn.click()


    for i in keys:
        
        if(ch[i]['solved']):
            continue

        browser.get(f'https://ctf.fword.tech/api/v1/challenges/{i}/solves')
        html = browser.page_source
        time.sleep(2)
        html = html[html.index('{'):html.rindex('}')+1]
        y = json.loads(html)
            
        try: y['data']
        except:
            print('key error')
            continue

        if(y['data']==[]):
            continue

        if(y['data']==[]):
            print(f'no data for {ch[i]}')
            continue

        ch[i]['solved'] = True
        # print(f'`First blood for challenge: {ch[i]["name"]} goes to {y["data"][0]["name"]}`')
        print("sending")
        await channel.send(f'```css\n🩸 First blood for .{ch[i]["name"]} goes to [{y["data"][0]["name"]}]```')
        print(ch)
    browser.close()
    print('Completed!')
```
\
![enter image description here](https://github.com/H4MA-R/starter-hugo-academic/blob/master/content/post/FwordCTF2k21%20Automation/firstblood.png?raw=true)

btw credit to the CSICTF admin for his amazing article it helped me a lot(link below)
# <a name="3"></a> 
## Certificates script
In this year edition, we had more than 1000 participants and more than 400 teams that made it to the scoreboard so no way we were going to do the certificates manually and send them for the teams so I opted to write a script that generate all the certificates and send them via email to the teams.

So to generate the certificates I started with a template done by our talented designer Aptx and I used pillow library in python to put the team name, points, and ranking and the result was a pdf containing the certificate that will be attached to the email.
```python=
im = Image.open(r'Certificate.png')
d = ImageDraw.Draw(im)
text_color = (150, 192, 14)
font = ImageFont.truetype("NeusaNextPro-Light.ttf", 250)
w, h = d.textsize(i['name'], font=font)
location = ((2700-w)/2, 1000)
d.text(location, i['name'], fill = text_color, font = font)
#write score
text_color = (255, 255, 255)
font = ImageFont.truetype("NeusaNextPro-Regular.ttf", 100)
w, h = d.textsize(str(i['score']), font=font)
location = ((4250-w)/2, 1650)
d.text(location, str(i['score']), fill = text_color, font = font)
#write positions
text_color = (255, 255, 255)
font = ImageFont.truetype("NeusaNextPro-Regular.ttf", 100)
w, h = d.textsize(str(i['pos'])+"#", font=font)
location = ((1100-w)/2, 1650)
d.text(location, str(i['pos'])+"#", fill = text_color, font = font)
im.save("certificate_" + str(i['pos']) +".pdf")
```
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
```python=
sg = sendgrid.SendGridAPIClient(api_key="###SENDGRID_API_KEY###")
        from_email = Email("contact@fword.tech")
        x = id_list.index(i['members'][0]['id'])
        to_email = To(mail_list[x])
        mail = Mail(from_email, to_email)
        mail.dynamic_template_data = {'team' : i['name']}
        mail.template_id = "###SENDGRID_TEMPLATE_ID###"
        #with open("certificate_" + i +".pdf", 'rb') as sigf:
        #    sig = sigf.read()
        sig = open("certificate_" + str(i['pos']) +".pdf", "rb").read()
        encoded = base64.b64encode(sig).decode()
        attachment = Attachment()
        attachment.file_content = FileContent(encoded)
        attachment.file_type = FileType('pdf')
        attachment.file_name = FileName("certificate_" + str(i['pos']) +".pdf")
        attachment.disposition = Disposition('attachment')
        attachment.content_id = ContentId('Example Content ID')
        mail.attachment = attachment
        response = sg.client.mail.send.post(request_body=mail.get())
```

The code attache the certificate and fill the placeholders thens end it using sendfrid api the team mail that is stored in an excel sheet
# <a name="4"></a> 
## Conclusion
I know these are simple scripts but it helped us manage the CTF and made it a lot easier than doing all these tasks manually, these script had a lot of issues like for the bot if anything happens the data will be lost and it will starts the first blood from he brggind and also the health check can have a lot of flase alerts s for net year edition we set the goal to combine all these tasks in one bot that it will manage everything from the beginning of the CTF with no human interaction it will be more optimized and it will help us focus more interacting with the participants and doing more challenges.

In the next article, we will break down the main infrastructure and all its components and we will talk in detail about all the challenges and problems we faced and the solution we went with.

### Links: 
CSICTF bot: https://medium.com/csictf/kuwu-a-discord-utility-bot-for-ctf-s-5727ea5a6019 \
All the source code: https://github.com/H4MA-A/FwordCTF
