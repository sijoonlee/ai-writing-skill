source: https://www.youtube.com/watch?v=pqlWNihgdjI

Chapter 1: Four phases, and only 10% to 20% felt
0:011 second[music]
0:1212 secondsMy name is Claire Logori and I'm a senior principal engineer at AWS. I mostly work on Kirao, our agent coding
0:2121 secondsassistant. But today I want to talk about some of the practices we've been seeing inside of Amazon in Amazon teams where we've been seeing really exciting
0:2929 secondsresults of productivity increases that are step function improvements since what what we've been seeing with AI so far.
0:3838 secondsSo I've been working on Agentic AI for over three years now and I've kind of seen the evolution that's happened in
0:4646 secondsour industry when it comes to coding assistance with AI. First, we had this inline code completion helping us to uh
0:5555 secondswrite the next line, maybe the next function. We moved on to chat, asking questions about our code. Everybody started doing vibe coding sometime last
1:031 minute, 3 secondsyear. But now we're starting to see kind of an early adopter phase of what we've been calling frontier development. And
1:111 minute, 11 secondscompletely anecdotally based on my own experience, I've really only felt maybe 10 to 20% more productive with all of
1:201 minute, 20 secondsthese phases that have come before. But now inside of Amazon, we've been running pilots with different teams across the
Chapter 2: What the pilots actually measured
1:261 minute, 26 secondscompany. And we've been seeing a median of 4.5x productivity improvement and sometimes more than 10x. So something
1:351 minute, 35 secondshas really changed here now that we're seeing these step function improvements in productivity.
1:401 minute, 40 secondsAnd I like to um define what we've been calling frontier developers inside of Amazon by three behaviors that I've been seeing. One is hands-off coding.
1:521 minute, 52 secondsFrontier developers write maybe one to two% of the code that they produce. The rest is agents. The second is that they interact with their agents infrequently.
2:032 minutes, 3 secondsThey'll aim to get their coding assistant to run for up to hours at a time without their intervention. And third is that they minimize idle time.
2:132 minutes, 13 secondsThese frontier developers tend to run multiple agents in parallel churning through a backlog of tasks.
2:222 minutes, 22 secondsThe first time that I saw a frontier developer team was the Bedrock mantle team. Bedrock is our model hosting
2:302 minutes, 30 secondsservice. hosts uh LLMs like Claude and GPT. And uh sometime last year, we knew
Chapter 3: Thirty people for 18 months, or six people for 76 days
2:392 minutes, 39 secondsor I say we but the Bedrock team uh knew that they were going to need to build a new inference data plane, but they had
2:472 minutes, 47 secondsestimated it at 30 people over 18 months. this is a big big service and it was going to take time to build the new
2:552 minutes, 55 secondsone, migrate customers over, migrate models over and they decided to take a step back. They took six people and they [snorts] built it in 76 days with Curo.
3:063 minutes, 6 secondsSo this was a huge achievement. This was the first time we've we'd seen anything of the kind inside of Amazon. So this
3:133 minutes, 13 secondswas truly the Pathfinder team that proved that it was possible to get up to 20x improvement. Now they looked at
3:213 minutes, 21 secondscommits and I'll talk about a couple of other ways that we are uh measuring productivity improvements. But there was
3:283 minutes, 28 secondsone problem with this story which was that yes it was built with six people.
3:333 minutes, 33 secondsIt was built with some of the top engineers literally in the company including two distinguished engineers.
3:403 minutes, 40 secondsSo this was not just any team of six people. These were experts in distributed systems, experts at LLMs and their architecture.
Chapter 4: Why that team was not reproducible
3:513 minutes, 51 secondsSo this this story was amazing and it kind of spread like wildfire across Amazon, but it was also very unachievable for a lot of teams. There
3:593 minutes, 59 secondswere a lot of questions about can this actually be reproduced on another team.
4:054 minutes, 5 secondsSo another experiment that I want to talk about is a an experimental sprint that was done in the prime video
4:124 minutes, 12 secondsorganization. They took a 10-day sprint and they did an experiment where they put again six engineers in a room and
4:204 minutes, 20 secondsthey let them go wild with Kira. Uh they brought down the project delivery time
4:274 minutes, 27 secondsestimate from what was going to be 90 weeks down to 24 based on all of the progress they had made in this 10day
4:354 minutes, 35 secondssprint. and they lo they looked at their commit history and they looked at what did they used to do prior to this 10-day
4:434 minutes, 43 secondssprint and how many commits did they produce just in this 10 days. And so this sprint really proved that we can
Chapter 5: A ten day sprint, and its asterisks
4:514 minutes, 51 secondsachieve again at least something close to what the bedrock mantle team had uh had achieved with a different set of
4:594 minutes, 59 secondsengineers. But again, there was a challenge with this story, which was it was six engineers in a room, but they
5:075 minutes, 7 secondshad no on call duties, limited meetings, very few distractions, which we all know are regular in the lives of an engineer.
5:175 minutes, 17 secondsand the senior engineer on the team had spent the previous three weeks creating very detailed small wells scoped tasks
5:255 minutes, 25 secondswith detailed requirements for these [clears throat] six engineers to just go churn on for those two weeks. So this was again not necessarily real life.
5:355 minutes, 35 secondsThis was a structured sprint uh a a point in time that they were able to achieve this. But again the question is
5:425 minutes, 42 secondsis this achievable on real teams on day to for day-to-day work. So Amazon stores
5:495 minutes, 49 secondswhich encompasses uh Amazon.com all of our uh retail websites as well as our physical stores did a more structured
Chapter 6: Fifty ordinary teams, and the real split
5:585 minutes, 58 secondspilot. They watched 50 teams that were totally normal, normal distribution of
6:056 minutes, 5 secondsum early career folks, mid-career senior engineers and that worked on existing systems. Nothing green field like the
6:136 minutes, 13 secondsMantle team got to build from the ground up, but existing systems with existing code bases. And they they watched them
6:216 minutes, 21 secondsfor the better part of last year and they found something super interesting.
6:266 minutes, 26 secondsThey found that there was a big difference in the productivity gains that they saw between half of the teams and the other half. And in this case,
6:356 minutes, 35 secondsthey used a productivity metric of deployment velocity to production. So not just commits, how many commits are
6:426 minutes, 42 secondsthey producing, but how quickly are we getting changes out to customers? How many how quickly are we able to ship things? And they saw that for half of
6:516 minutes, 51 secondsthe teams, they achieved less than 3x increase. And what they found that was the difference between seeing less than
6:596 minutes, 59 seconds3x productivity increase, these teams that saw a median of 4.5x and and in some cases more than 10, was how they
Chapter 7: Same tools, different ways of working
7:077 minutes, 7 secondsuse the tools. 90% of these teams use Curo among other internal tools that we have. And what they found was it wasn't
7:167 minutes, 16 secondsabout the tools, it was about the way that they worked. The teams that achieved step function improvements
7:237 minutes, 23 secondsintentionally changed the way that they worked and the others simply kind of sprinkled Kirao and some of the other tools that we have on top of their
7:327 minutes, 32 secondsexisting way of working. And for me at least, this was the big aha moment that why I hadn't been feeling potentially
7:407 minutes, 40 secondsthe massive gains that productiv that in in productivity that AI has promised, it's about changing the way that we work.
7:497 minutes, 49 secondsSo across this pilot they went and interviewed uh the teams that were involved in the pilot as well as some of these other teams on the bedrock mantle
7:577 minutes, 57 secondsteam on uh prime video and they found five habits and and I use the word habits very specifically because again
8:068 minutes, 6 secondsit's not about that one sprint it's about doing this daytoday and it and what they found when they interviewed with these teams was that it really was
Chapter 8: Habit one: invest in agent context
8:148 minutes, 14 secondshabits that they had to build daytoday when we change our way of working. It's it's hard to build these habits. It
8:228 minutes, 22 secondstakes time to build these habits. So, let's go through each of these one by one. Habit number one is investing in
8:288 minutes, 28 secondsagent context. We have a lot of stuff in our head. We tend to transfer all of that stuff in our head to other people
8:368 minutes, 36 secondsthrough Slack conversations, through onboarding mentors, things like that, through code reviews, through standups
8:448 minutes, 44 secondsand sprint planning. And they had to write all of that down. And the habit that they built was every time the agent
8:528 minutes, 52 secondsmakes a mistake or does something not the way that you would have done it, what am I missing in my skills files?
8:578 minutes, 57 secondsWhat am I missing in my steering files that the agent needed? But then as we know across last year, we saw leaps and
9:069 minutes, 6 secondsbounds in models abilities and their behaviors. uh the sonnet 3.7 in the middle of last year had a lot of quirks
9:149 minutes, 14 secondsthat we had to put a lot of do nots in our uh in our steering files and now we don't have to do that as much with Opus
Chapter 9: Pruning context as models improve
9:219 minutes, 21 seconds4.5 as of last November and then we've had six months more than six months of improvement since then uh with all of
9:299 minutes, 29 secondsthe new versions of models that have come out since then and so the question the new habit again is do I still need this in my steering files or is this
9:379 minutes, 37 secondsjust bloating context The second one is slowing down to speed up. In almost every team that was
9:449 minutes, 44 secondsinterviewed, they reported that their productivity actually went down as they intentionally adopted a new way of working. That's counterintuitive, right?
9:549 minutes, 54 secondsYou have to do intentional engineering work before you're going to see that hockey stick curve in productivity
10:0110 minutes, 1 secondimprovement because we have to do real work in our code bases first for agents to be successful there especially in
10:0810 minutes, 8 secondsbrownfield existing code bases. So they had to build that agent context up. They had to improve existing tools error messages so that the model knew what was
10:1710 minutes, 17 secondsgoing on when it failed. They built new tools, new MCP servers for helping that model to actually get done what it
10:2510 minutes, 25 secondsneeded to get done. A lot of teams ended up restructuring their codebase so that agents could actually navigate it more easily. And I've even seen drastic
Chapter 10: Habit two: slow down to speed up
10:3410 minutes, 34 secondschanges like changing the programming language of the codebase. Um, often I've seen teams struggle with Python, with
10:4110 minutes, 41 secondsJavaScript because they're untyped languages. It's hard to test. There's no compiler errors. So the model kind of
10:4910 minutes, 49 secondsguesses and give it gives it back to you. And so I've seen teams moving to Typescript. Um Rust has become very
10:5610 minutes, 56 secondspopular inside of Amazon. The compiler gives great error messages. Um you don't have to do that, but I've seen a lot of
11:0311 minutes, 3 secondsteams making those intentional changes for the productivity gains that they're able to see.
11:0911 minutes, 9 secondsThe third one is feeding a agents, not babysitting agents. And for me, this was one of those aha moments of why we're
11:1711 minutes, 17 secondsseeing this step function improvement in productivity.
11:2111 minutes, 21 secondsIf you are vibe coding, if you are having a back and forth conversation with your agent all day long, of course
11:2811 minutes, 28 secondsyou're not going to see four to 5x productivity improvements because you are in the loop the entire time. You're
11:3611 minutes, 36 secondsprobably sitting there for 30 seconds to a minute waiting for it to generate code and come back to you with with the code to review.
11:4411 minutes, 44 secondsIf you're sitting there waiting for it, then you can't go off and do other stuff. It's really difficult to run agents in parallel. It's very difficult
11:5311 minutes, 53 secondsto get to to clone yourself into multiple agents. And so if your conversations look a bit like this on
12:0012 minutesthe left, then you're babysitting that agent as opposed to the right side where you're feeding it what it needs to do and how it can self- validate. And
12:1012 minutes, 10 secondsthat's really the key so that agents can self-correct and only come back to you when it meets a certain quality bar, when it when it actually runs and
12:1812 minutes, 18 secondscompiles and passes tests, when it's testable, when it it actually has high coverage. And of course, the next level
12:2512 minutes, 25 secondsis put all of this content into your steering file so it does it every time without you having to prompt it.
12:3312 minutes, 33 secondsThe fourth habit is to make intent explicit. At Amazon, we practice a lot of specri development. We've built that
12:4112 minutes, 41 secondsinto the Kuro product and so it's very natural for Amazon engineers to adopt it in Kuro. Um, what what I've typically
Chapter 11: Feeding agents instead of babysitting them
12:5012 minutes, 50 secondsseen with fibe coding as opposed to frontier engineering is giving a very highlevel prompt, letting the agent
12:5812 minutes, 58 secondsgenerate a ton of code and then having a back and forth conversation saying, "Oh, that's not really what I meant. That you
13:0713 minutes, 7 secondshaven't you haven't exactly gotten the the requirements right. No, I didn't actually want to build it that way.
13:1213 minutes, 12 secondsHere's a technical design." And it is less I find less productive to iterate with the agent on code when the intent
13:2113 minutes, 21 secondsitself was incorrect. So uh often we'll h we'll see Amazon engineers go through
13:2813 minutes, 28 secondsthis process for for ambiguous complex features of writing the specification and in curo of course you don't have to
13:3613 minutes, 36 secondswrite this whole specification you can have the model generate it but it's a lot easier to h to iterate with the
13:4313 minutes, 43 secondsmodel in kind of a back and forth conversation about a document than it is about code that's code changes that are spread across a codebase.
13:5313 minutes, 53 secondsThe fifth one is shift testing left. One of the keys here is to give the agent
Chapter 12: Making intent explicit before writing code
14:0014 minutesthat fast feedback loop because that's what lets it go off for hours at a time and self-correct. The agent is going to
14:0714 minutes, 7 secondsmake mistakes and that's fine. But if you give it the right signals, it can self-correct and it can spend a while
14:1514 minutes, 15 secondsdoing that. So I've seen teams adding llinters, adding unit tests, integration tests, performance tests, security
14:2214 minutes, 22 secondstests. These are all things we all know we should have been doing all along.
14:2614 minutes, 26 secondsThis is good engineering hygiene and practices. But now the ROI is, I think, finally high enough for actually us to
14:3414 minutes, 34 secondsactually invest in it. Um, one thing that I've been seeing a lot of teams do is mock out services. Often with
14:4114 minutes, 41 secondsintegration tests, we would test kind of end to end an entire system, including live services, but we've been investing
14:4814 minutes, 48 secondsa lot in in mock services that run entirely locally with deterministic responses because it lets the agent do
14:5614 minutes, 56 secondseverything locally. Um, doing everything on your laptop without having to spin up a bunch of other services and and
15:0415 minutes, 4 secondsconnect to cloud services makes everything a lot faster because the the more that your agent can get fast
Chapter 13: Shifting testing left with local mocks
15:1215 minutes, 12 secondsfeedback means the more loops that it can can do and the more productive your own agent can be.
15:1915 minutes, 19 secondsSo, across all of these, these are some of the habits we've seen. But of course, I would be remiss if I would tell you if
15:2615 minutes, 26 secondsyou adopt all of these habits, you will achieve Nirvana. You will be the most productive engineering organization uh
15:3415 minutes, 34 secondsthe world has ever seen. Things are still hard. We are still very much in an early adopter phase and teams are still
15:4215 minutes, 42 secondsfiguring it out. So, one thing that we've been seeing across our teams just organizationally is the risk of burnout.
15:4915 minutes, 49 secondsUm I did not coin this term. I forget who did at at what conference, but FOMO is real. We've been seeing engineers
15:5715 minutes, 57 secondsstaying up late late at night um trying to get that perfect prompt that's going to make their agent run for hours overnight so that they wake up in the
16:0616 minutes, 6 secondsmorning with a code change ready. Uh the cognitive load increases as you run these multiple agents in parallel.
16:1316 minutes, 13 secondsYou're constantly shifting between terminal tabs. And then we do see that reviewing AI output is often harder for
Chapter 14: Burnout, FOMO and cognitive load
16:2116 minutes, 21 secondssome than than actually writing it especially early in career. Uh senior engineers have have already spent a
16:2916 minutes, 29 secondslarge portion of their career reviewing others code. But early career engineers h don't have that muscle yet and so
16:3716 minutes, 37 secondsreviewing it can can feel like a lot more cognitive load uh than they're used to in actually writing it. The other one
16:4516 minutes, 45 secondsis organizational change. So it's already hard to change the way we work as engineers. The way that we spend our
16:5216 minutes, 52 secondsentire day completely changes when we're frontier engineers. But also organizations have to change to enable
17:0017 minutesfrontier engineering teams. Um one that I've seen very commonly is accepting slowing down to speed up. And I've been
17:0917 minutes, 9 secondsguilty of this myself. My my fellow leaders have been guilty of of this of saying, well, you have the AI tools now
17:1617 minutes, 16 secondsand the models are so amazing now. Why are you not going faster? Um, and that's because you have to take those two
Chapter 15: What organizations have to change
17:2517 minutes, 25 secondsmonths to invest in your codebase to figure out the best practices for your team to make hard habit changes on your
17:3417 minutes, 34 secondsteam. Um and and if you're constantly expecting shipping features every month because now we have these amazing models and
17:4217 minutes, 42 secondswe're seeing um all of these these companies on X saying how they're shipping 20 PRs a day. Um we have to
17:5217 minutes, 52 secondsslow down to speed up. The second one is actually going too broad in the organization too fast. I think that if
17:5917 minutes, 59 secondswe had um expected all teams in massive organizations to be frontier teams
18:0618 minutes, 6 secondsimmediately, we would not have had the learnings that we had from the pathfinder, from the from the sprint
18:1318 minutes, 13 secondsexperiment, from the pilot uh teams within Amazon. And now the challenge for us is how do we scale it out? And that's
18:2118 minutes, 21 secondswhat 2026 is about for Amazon is how do we scale this out to more and more teams to the next uh 2,000 teams instead of 50
18:3018 minutes, 30 secondsteams. Um and so I think that when you roll it out too quickly, you have a lot of teams who don't know what they're
18:3718 minutes, 37 secondsdoing. You haven't had time to find the best practices for your own organizations, the the context that your
18:4418 minutes, 44 secondsorganization needs. And the last one is that you're going to find new bottlenecks. Previously code writing
18:5118 minutes, 51 secondscode manually was the bottleneck. Um I find that within Amazon we've found um
18:5818 minutes, 58 secondsthe speed of decision- making becomes a new bottleneck. Um the more that you spend reviewing the decision to actually
19:0519 minutes, 5 secondsbuild a new product, the slower it is to build the product now because the code only takes one to two months to write.
19:1319 minutes, 13 seconds[snorts]
19:1319 minutes, 13 secondsum all of the review processes associated with the launch of a product become the bottleneck. When it used to
19:2119 minutes, 21 secondstake 9 to 12 months to build a new product, it didn't matter so much in the in the overall wash of things. If it
19:2919 minutes, 29 secondstook two months to make the decision to build the product and then two months to approve the launch, but now those are the bottlenecks, those are the long
19:3819 minutes, 38 secondspole. And so you find all of these um all of these things that slow you down.
Chapter 16: When decision speed becomes the bottleneck
19:4419 minutes, 44 secondsOften I find that frontier engineering teams spend more time making decisions than they do writing code. And so the
19:5119 minutes, 51 secondsmore that you can make fast decisions, especially ones that are easy to be reversed, the better.
19:5719 minutes, 57 secondsSo my one big takeaway for for everyone here is that frontier engineering is about intentionally changing the way that you work. And that is difficult.
20:0920 minutes, 9 secondsThat takes time. It is forming new habits and a new way of working. And that goes across any engineering team as
20:1820 minutes, 18 secondswell as your organization. Um so I encourage you to think about um how you're interacting with AI tools and how
20:2620 minutes, 26 secondsthat can change to free yourself up from being in the loop. Um thanks. I'm going to I'll hang out uh a little bit if
20:3420 minutes, 34 secondsanyone has questions in the back. Um, but thanks for the time today.