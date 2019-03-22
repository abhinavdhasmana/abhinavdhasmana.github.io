---
layout: post
title:  "How to conduct technical interviews"
tags: [Interview]
summary: Tech interviews are broken. Let's do our best to fix it
author: Abhinav Dhasmana
---

Over a period of time I have given and taken several technical interviews. Here are some of my learnings
![](/images/blog/interview/1.jpeg){: .center-image }

Of course we want to hire the best person and we are driven to hire “A Players”. However this thought process can lead to the point where the interviewer want to prove that he is a “A player”. This defeats the whole purpose of the interview.

* * *

> The idea of the interview is to know what the candidate knows and not how awesome you are.

* * *

**Introduce yourself before you ask “Tell me something about yourself”:** More often than not, this is the very first question I ask as an ice breaker. However, I always introduce myself and set up a pace and context on what I am looking for in the introduction. The good part is that this can be tailored based on the skill/experience of the interviewee.

For example, when interviewing someone relatively junior, I would focus more on technical details as I am interested in hands on coding knowledge and less on architecture design.

```
Hi!! I am Abhinav and I work with <team name> team.
Over a period of years, I worked in various technologies like RoR,
NodeJS, Go and C#.I am also learning React these days and it has
been a fun journey so far.

Currently I am working on <interesting problem> and we are solving
<this particular problem overview> which is really really exciting.
```

If, I am interviewing some one with more experience, it would go like this

```
Hi!! I am Abhinav and I work with team <team name>.
Over a period of years, I worked in various technologies
like RoR, NodeJS, Go and C#.

I really enjoy working on RoR and NodeJS. Most of the time we end up using
both for creating micro-services to arrive at the right solution.
We try very hard to follow Single Responsibility Principle for our services.
```

The first response shows that we all learn new things all the time and its a great company where one can learn and grow. Quiet a few times the discussion starts right from here where candidate starts talking about React with Redux/Angular or how they solved a problem with simple jQuery.

The second response is far more biased. Essentially, I am looking for a person with a strong view on things that they have been doing. Some in-depth on why and how they choose technology to solve a problem.

If I do not start with my introduction, the interviewee can either go too wide (Covering his 10 years of experience) or too much in depth(Just talking about his current project). To do justice to time, I have to interrupt the person and ask a follow up question or worst loose interest and time.

**What’s the most interesting problem that you have solved:** This is my first question after the introduction. The reason is best explained by Elon Musk in the quote below:

> “When interviewing someone to work for the company, I ask them to tell me about the problems they worked on and how they solved them, and if someone was really the person that solved it, they would be able to answer on multiple levels, they would be able to go down to the brass tacks, and if they weren’t they will get stuck.”

> “And then you can say this person wasn’t really the person that solved it, because anyone that struggled hard with the problem never forgets it.”

If the problem is “good” enough, this would be the only question I ask during the whole interview time. If not, I either ask to pick some other problem. If that fails as well, I generally go for some standard interview questions based on what role we are hiring for.

**Don’t come with domain/tool/technology specific questions and solutions:**

![](/images/blog/interview/2.png){: .center-image }

In case I have to ask the question, I never ask domain specific questions unless its an exact match between the question and what the interviewee has done (which is rare). This is also the most common source of dissatisfaction with interviewees if they do not get selected.

Lets take a simple example of a leaderboard. You are a gaming company with few million players. You want to maintain a leaderboard for all the players. You give the same problem to the interviewee. Lets consider that the candidate does not know about redis and its sorted set and have only worked with mySQL. It would be unfair if one would expect the person to come up with a solution and implementation of sorted set in those 30–40 minutes. A better approach would be to see what we can do if we only had mySQL and what would you do to solve it and focus on the approach. The best approach would be to ask a more appropriate question which matches the skills of the individual.

Another example which is common these days is designing newsfeed. Its not a bad question to ask because we see this everyday in FB and Twitter. Where this starts going bad is where people start saying now scale it to FB level. There are two things that are wrong with it. First there are very very very few companies that operate at that level and if you company is one of them, then it did not became overnight. There would have been several mistakes that would be done before getting things right. Its unfair to expect to get to the “right” solution in that short stressed interview time frame. Second, if I have not done something before, I google and read about it. Listen to some talks on how other people have done it and then take architecture decisions. Making those decisions on the fly with specific technologies ( redis /kafka /rabbitMQ etc) is again a knowledge that can be acquired over a period of time and is not the true reflection of the interviewee potential.

**Don’t have more than one interviewer in a session:** This might be a little controversial but in my experience this has more flaws than gains. Lets see how this plays up. Interviewee explain his problem and how she overcame it. Now both the interviewer could have slightly different interpretation and can ask completely different follow up questions. It becomes far more difficult for the interviewee to switch context between two and to move to a next round, she would have to satisfy two different individual who might not be in a sync about the problem.

I would prefer to have two set of interviews rather than one interview with two people.

**Change the problem exactly once:** Every now and then it happens that I have to give a problem because the candidate challenging problem is too domain specific/ less interesting. If I see the candidate is not making progress fast enough as I would like, I would find a logical end to the problem, so that I do not affect the confidence, and move to a next problem. I explain this new problem in more details than the first one. My aim is to get as much information as I can from the interviewee rather than confirming from my first interview question that candidate does not know enough.

On another extreme are people who catch up too quickly to the problem. Either they are smart or have solved similar problem before, I would switch to a different problem and see how they perform.

**Some obvious points:**

*   Be polite even if the candidate is not the right fit.
*   Exact solution is not important. Approach to the solution is far more important.
*   Drop the relevant hints during the whole problem solving session.
*   Don’t look at the resume and pick one technology/problem you know well and grill on it because “its there in the resume”

Do you have other good/bad experiences which are not covered? Lets share and learn!!