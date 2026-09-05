Source: Building an Autonomous Engineering Org
https://www.youtube.com/watch?v=whue9_YquGA

So, I've spent the last couple of years transforming Block's 3500 person engineering org into an autonomous one.
0:088 secondsAnd this is a challenge that most tech companies are actively trying to solve or will be in the very near future. So, today I'll share our path to getting
0:1616 secondsthere. Our agentic coding journey started very early. We were building goose, our internal coding agent, even before the LLM supported tool calling.
0:2727 secondsWe work with Anthropic as design partners for the initial release of MCP and Goose became the reference implementation for the MCP client. So
0:3636 secondsinternally our most curious engineers were some of the very first in the industry to use these types of coding agents.
0:4444 secondsAfter a couple of months about 90% of our engineers were now using tools like goose and Claude Code regularly to
0:5252 secondsgenerate code. So on paper it looked like we were all in on AI but our CEO was convinced that engineering wasn't
0:5959 secondsusing AI at all like they couldn't be right as far as he was concerned because we weren't shipping any faster.
1:071 minute, 7 secondsWell I had the numbers both the metrics and the token bills. So I knew that engineering was in fact using AI but he
1:141 minute, 14 secondswas right. The features certainly weren't making it to our customers any faster. So I began to dig into this a
1:211 minute, 21 secondsbit. I think of AI enablements in three phases. Experimentation, adoption, and impact. I'd say that we surpassed the
1:301 minute, 30 secondsexperimentation phase as 90% of our engineering or was using AI, but they still were kind of using it inside of
1:381 minute, 38 secondsthe IDE, you know, asking questions or writing boilerplate code. And I knew that if we really wanted to see impactful outcomes, then we needed a way
1:471 minute, 47 secondsto integrate AI into how we build and ship.
1:521 minute, 52 secondsI'd spent the first half of 2025 leading AI enablement for our entire company. So 12,000 employees across every function,
2:012 minutes, 1 secondmarketing, design, finance, legal. Now our CTO tasked me with building an
2:072 minutes, 7 secondsagentic engineering or okay sure. But what does that actually mean? There's no playbook for any of this stuff, right?
2:152 minutes, 15 secondsAnd I know because I went looking at your blogs hoping that you all had it all figured out, but I only saw a bunch
2:232 minutes, 23 secondsof posts saying how you all were making it up as you went along. So I did the same, right? In the simplest of terms, I
2:312 minutes, 31 secondsdefined an agentic engineering org as one where engineers leverage AI agents as their primary means of producing
2:382 minutes, 38 secondsengineering outcomes. So that meant that our engineers needed to treat agents as core members of their engineering
2:462 minutes, 46 secondsworkflow. Not just using AI to help them write code here and there, but actually working with agents, decomposing
2:542 minutes, 54 secondsproblems, delegating work, reviewing and verifying what was done. And we wanted them directing the work as their default
3:023 minutes, 2 secondsway of operating. But of course, the vast majority of our engineers were not here. So to see where we needed to go, I
3:103 minutes, 10 secondscame up with the maturity model. This model measures the engineers relationship with AI agents. So how they
3:173 minutes, 17 secondsthink and delegate and orchestrate. And while I had some form of this model in Q3 of last year, Steve Yegge's Gastown
3:263 minutes, 26 secondsarticle helped me reorganize it into a better model. So stage zero is where the engineer doesn't use AI tools in their
3:343 minutes, 34 secondsworkflow at all. Stage one, they use AI to autocomplete, but they've never used it in agent mode, maybe. Uh, stage two
3:443 minutes, 44 secondsis where engineers are chatting with agents, but not using it to produce any PRs. Stage three, engineers are delegating tasks to agents and reviewing
3:533 minutes, 53 secondsthe output. Those at stage four are running multiple agents in parallel. And then stage five is that final boss where
4:004 minutesengineers are delegating complete tasks to agents and the agent is able to produce shippable results without the human necessarily needing to guide it.
4:094 minutes, 9 secondsSo I'd say by the end of the first half the bulk of our engineers were between stages one and two and I needed to get
4:164 minutes, 16 secondsthem to stage five. Now, I wasn't quite sure how to get 3500 engineers to that level, especially when one, this is all
4:264 minutes, 26 secondshighly experimental, right? There's no playbook again for any of this. Um, two, things were moving so fast that what
4:344 minutes, 34 secondsmight be a best practice this week could be outdated when a new tool or a new model drops next week. And honestly,
4:414 minutes, 41 secondsthis was leading to AI fatigue. And then three, people were already feeling turned off by the top down pressure from
4:494 minutes, 49 secondsleadership to essentially AI or die. So I thought about the 1/9/90 rule where in
4:564 minutes, 56 secondsdigital communities about 1% create, 9% interact, and 90% passively consume. And this maps almost perfectly to how
5:055 minutes, 5 secondsengineers adopt AI. So, you'll have a small percentage that'll deep dive and start creating aic patterns and discovering useful techniques for
5:135 minutes, 13 secondsworking with agents. You'll have some that might tweak the agents MD file here or there, but most people aren't going
5:215 minutes, 21 secondsto spend the extra cycles to figure this stuff out. So, I realized that if my AI strategy depends on every individual
5:305 minutes, 30 secondsleveling themselves up, I'm never going to see that broad impact. So, I leaned into this model and I used it to my
5:385 minutes, 38 secondsadvantage. Instead of focusing on all 3,500 engineers, I decided to focus on forming the 1%, the power users from our
5:485 minutes, 48 secondsmost critical teams. So, I started an AI champions program where I handpicked about 50 engineers across the company
5:565 minutes, 56 secondswho could pioneer what aic engineering looks like for their team. Now, this was not a call for volunteers. All right? I
6:036 minutes, 3 secondsneeded to be strategic about the selection. I needed engineers who were willing to dedicate at least 30% of
6:116 minutes, 11 secondstheir time to investing in AI enablement. I needed those who wouldn't be discouraged by the non-deterministic
6:186 minutes, 18 secondsnature of AI and give up on it all when it didn't work out of the box, which it often did not, right? So I spent the
6:256 minutes, 25 secondsweek talking to tech leads and managers to find the 50 engineers who could represent our most critical repositories.
6:346 minutes, 34 secondsNow remember at this point most of our engineers were between stages one and two where they're chatting with AI but
6:416 minutes, 41 secondsnot really using it to produce PRs. So I wanted to get the AI champions to stage three. This was about June 2025. So the
6:506 minutes, 50 secondsmodels were decent enough to write a feature for you, right? the chances were pretty high that the code wouldn't conform to your team's conventions and
6:586 minutes, 58 secondsstandards. So developers didn't trust the agents enough to delegate work to it yet. So the first area that we focused
7:057 minutes, 5 secondson was making our repos AI ready. My theory here was if I can get engineers
7:127 minutes, 12 secondsto embed AI directly into their repos, then not only would the agents perform better, but the entire team would benefit, not just the 1%.
7:227 minutes, 22 secondsAnd the repos are a central point of reference for every engineer that's contributing code. So to make their
7:307 minutes, 30 secondsrepos AI friendly, they added assets like context and rules files so that agents could reliably understand and
7:377 minutes, 37 secondsnavigate the codebase as well as contribute to it.
7:427 minutes, 42 secondsNow when building the AI champions I made sure that we had folks from every corner of block engineering square cash
7:497 minutes, 49 secondsapp Afterpay TIDAL across front end backend mobile data infra right um we
7:567 minutes, 56 secondscovered repos of all shapes and sizes the nasty big legacy massive monor repos the smaller services mobile apps right
8:058 minutes, 5 secondsand that mix let us pressure test patterns across very different engineering realities so that we could
8:128 minutes, 12 secondsquickly see what actually scales. As expected, monor repos came with their own challenges. But my JVM devs were already strong in inheritance patterns.
8:238 minutes, 23 secondsSo we set shared context and rules at the root then layered more specific ones at the service levels. We also learned
8:328 minutes, 32 secondsfast that what worked for web didn't necessarily work for mobile. So Android and iOS even needed different approaches at times.
8:418 minutes, 41 secondsSo instead of forcing a one-sizefits-all solution, each champion figured out what worked for their repo. And then teams
8:488 minutes, 48 secondswith similar shapes and sizes naturally converged on the same tools and patterns. And honestly, engineers love
8:578 minutes, 57 secondsthis, right? We let them choose what made sense for their repo instead of pushing a top- down mandate on them. So
9:059 minutes, 5 secondswe ended up with a standard set of components that would make a repo AI friendly but again customized for each team's needs. Some context files like
9:149 minutes, 14 secondsAGENTS.md or CLAUDE.md to provide the agent with repo guidance rules files to provide the agent with some guard rails.
9:239 minutes, 23 secondsrepeatable workflows like slash commands and later agent skills um an enabled AI co-reviewer preferably with some
9:329 minutes, 32 secondsinstructions on what matters and what it should review and then AI attribution on PRs.
9:399 minutes, 39 secondsSo, at this point, we've done a lot of great work and technically we do have the agents writing PRs, but I had a
9:469 minutes, 46 secondscouple of concerns, right? Not many outside of the AI champions had gotten to this level where they were delegating work to agents. And the idea is to have
9:559 minutes, 55 secondsthe work of the 1% lift everyone up. So, we weren't there yet. And then even with the champions, they were delegating, but
10:0410 minutes, 4 secondsthey were still kind of babysitting the process. So, I wanted to explore if we could delegate work to the agents in a
10:1110 minutes, 11 secondsmore hands-off approach and make it easier for others outside of the champions program to do so as well. So,
10:1910 minutes, 19 secondsthere's three common places where engineers receive requirements for work.
10:2310 minutes, 23 secondsIssue trackers like Jira or linear GitHub issues and informally in Slack. I wanted to be able to delegate work to
10:3110 minutes, 31 secondsthe agents from any of these places and the champions implemented all three. So true story,
10:3810 minutes, 38 secondswe were in Slack one day. An engineer saw a bug with the product and wrote, you know, like, "Hey, I saw this bug.
10:4410 minutes, 44 secondsAnybody else seen it?" So engineer 2 responds to that and goes, "No, I hadn't
10:5010 minutes, 50 secondsseen that one." Engineer 3 asks Goose right there in Slack and says, "Hey Goose, have you seen this bug before?
10:5810 minutes, 58 secondsLike can you can you go and check if this is a bug?" So Goose goes to the repo. It like pulls the files down.
11:0511 minutes, 5 secondslike, "Yep, this is a bug. Here it is right here. But here's also three options of how you might implement a
11:1211 minutes, 12 secondssolution." Right? Right there in Slack code snippets. And so engineer one goes, "I like option one." Engineer two goes,
11:1911 minutes, 19 seconds"Yeah, yeah, me too." So engineer three says, "Hey goose, go implement option one. Who's does so?" Returns with a link
11:2611 minutes, 26 secondsto the PR. Right? So the entire cycle from discussion, diagnosis, issue creation, alignment, and the fix took
11:3411 minutes, 34 secondslike five minutes all right there in Slack. Very cool party trick by the way.
11:3911 minutes, 39 secondsUm engineers were also able to assign linear and Jira tickets as well as GitHub issues to an agent and have the
11:4611 minutes, 46 secondsagent implement the work end to end. So now the agents have become a part of the sprint. This blew everyone's minds. The
11:5511 minutes, 55 secondsfirst time we did this, the team ran out of work and had to pull in more tickets like twice, right? So, of course,
12:0212 minutes, 2 secondsengineering managers, product teams, they loved this flow. And the beauty about these flows is that all engineers
12:1012 minutes, 10 secondsdidn't necessarily have to learn a new skill. The champions had already laid the foundation so that agents were able to work well within these repos. And
12:1812 minutes, 18 secondsthey worked well. All of this worked well because it made delegation feel native to how people already work. So
12:2712 minutes, 27 secondsnow I felt comfortable claiming that we were truly delegating work to the agents. At this point it had been 3
12:3512 minutes, 35 secondsmonths since launching the champions program and we were seeing great results. AI authored code was up by 69%.
12:4212 minutes, 42 secondsReported time savings increased 37% and automated PRs increased 21 times.
12:4912 minutes, 49 secondsWe were ready to pursue stage four, multi- aent parallelism. And with our setup, honestly, moving to stage four
12:5712 minutes, 57 secondswas almost free. We were pretty good at delegating work to the agents now. But this stage introduced several new challenges that we needed to solve.
13:0713 minutes, 7 secondsEngineers are now tripling, quadrupling the number of PRs that they're producing, but the PRs are stuck waiting
13:1313 minutes, 13 secondsfor code reviews, right? So people struggle to stay on top of code reviews even before we added AI to the mix. So
13:2113 minutes, 21 secondsyou know that it's bad now. And I'll be honest, this isn't a totally solved problem, but I'll share how we did stop
13:2913 minutes, 29 secondsthe bleeding a bit. We absolutely had to get the bots to help us review PRs. So earlier in the process, this was
13:3613 minutes, 36 secondsoptional, mostly because the AI co- reviewers sucked so badly and we were just pissing the engineers off by having
13:4313 minutes, 43 secondsthem use them. But now with the repo readiness work that the champions had done coupled with better models and
13:5013 minutes, 50 secondstools, shout out to Codex. Uh we were getting much better results here. So we enabled Codex on all the repos. And we
14:0014 minutesalso created an autofix loop where if it identifies issues, another agent will automatically fix those issues and commit them to the PR. So now at least
14:0914 minutes, 9 secondspeople were no longer complaining about not wanting to review sloppy PRs from bots. Right? By the time they got to them, they were in pretty good shape.
14:1814 minutes, 18 secondsAnother issue, specifically when running multiple agents in parallel, was that the agents were bumping into each other as they were trying to work and more
14:2714 minutes, 27 secondsimportantly, our machines couldn't handle the load anymore. Like engineers laptops were running out of memory, the CPU was choking, you know. So, we
14:3514 minutes, 35 secondsinvested in dedicated cloud workspaces where each agent ran in its own isolated environment. And this allowed us to
14:4314 minutes, 43 secondseasily run them in parallel and from anywhere. So at this point the engineers are cooking. They got four or five
14:5114 minutes, 51 secondsagents running at any given time and that number is growing. So a small group of engineers including many of the AI
14:5914 minutes, 59 secondschampions started building our own orchestrator to coordinate all of these agents, Builderbot.
15:0715 minutes, 7 secondsAnd we needed Builderbot to get us to an autonomous engineering org.
15:1315 minutes, 13 secondsWe realized that if we're trying to build anything close to an autonomous engineering or the agents need to understand where everything lives and
15:2115 minutes, 21 secondswhat depends on what. So we built a company world modeled based on the entirety of our 25,000
15:3115 minutes, 31 secondsrepo codebase, right? And this was a machine readable view of every single service and how they all connect. And
15:4015 minutes, 40 secondsthis allowed orchestrators and the agents that they delegated to to pull context in as needed and understand the
15:4715 minutes, 47 secondslandscape as they implement. And this enabled multiple agents to explore different parts of the system in parallel. Each building their own
15:5615 minutes, 56 secondsunderstanding and then come back together and have the orchestrator put all of this together, right? put in a plan that spans across multiple code
16:0516 minutes, 5 secondsbases which was especially useful for us as we were building offerings that spans multiple products. Now with this we had
16:1416 minutes, 14 secondssuccessfully reached stage five where engineers are delegating complete tasks to agents. The agent is able to produce shippable results without human
16:2216 minutes, 22 secondshandholding. In fact, this wasn't even limited to engineers anymore. Anyone at the company could at Builderbot in Slack
16:3116 minutes, 31 secondsand have it fix a bug or implement a new feature. They didn't even need GitHub.
16:3816 minutes, 38 secondsThis felt like a dream until it became a nightmare.
16:4716 minutes, 47 secondsYou know, of course, all layoffs are tough, but this one felt different. I had so many questions. Was this my fault?
16:5616 minutes, 56 secondsDid enabling employees to do the most incredible work of their careers ultimately result in their dismissal?
17:0617 minutes, 6 secondsJust the day before, I felt so proud. I was in awe of the way we were working, feeling quite accomplished that I
17:1417 minutes, 14 secondssuccessfully built an autonomous engineering or to what end?
17:2117 minutes, 21 secondsI guess I'll leave you all with a few questions. What are we doing? Where are we heading?
17:3017 minutes, 30 secondsAnd are we sure that it's where we want to end up?
