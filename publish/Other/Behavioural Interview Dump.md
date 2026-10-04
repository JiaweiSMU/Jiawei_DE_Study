### Behavioural interviews
Each question tests a quality:
- **Leadership:** how proactive you are
- **Resilience:** how you respond to challenges or failure (e.g. "proudest accomplishment" should show an obstacle overcome)
- **Teamwork**
- **Influence / persuasion**
- **Ethical / moral conflicts**
**Preparation**
1. Brain-dump experiences that demonstrate each quality.
2. Pick at least 2 stories per quality.
3. Shape each story as STAR:
    - **Situation:** 2–3 sentences of context
    - **Task:** what you were asked to do
    - **Action:** what you did, including trade-offs considered
    - **Result:** the outcome and what you learned
Show, don't tell: don't claim you have leadership; tell the story that demonstrates it.

### Leadership
#### Story 1
> [!NOTE] Title
> Contents

**Situation**
At Munich Re, I asked a colleague a question and she had to check a dashboard to answer it. After she applied the filters, it took over a minute to load. I asked whether it usually took that long.
**Task**
It wasn't my dashboard and nobody had asked me to fix it. But it was slowing her down every time she used it, so I asked if I could look into why in my spare time.
**Action**
I found the cause in the data model. It was a snowflake schema: dimension tables were chained to other dimension tables, so every filter had to pass through several joins before it reached the data. I went back to the same data in Databricks and denormalised the dimensions there, so the joins happened once, upstream. Then I loaded it into Power BI as a star schema, where each dimension connects directly to the facts. I used Performance Analyzer to time the same filters on the old and new versions.
**Result**
Filter response time dropped by about 70%. What I took from it is that you don't need to own something to improve it; you need to notice the problem and ask.
#### Story 2
- school project 

### Resilience
#### Story 1
**Situation.** In university I failed my algorithms mid-term badly.
**Task.** I needed to pass the module, which meant doing much better in the final.
**Action.** I looked at how I'd been studying and saw the problem: I was memorising solutions without understanding why they worked. So I changed my approach. I wrote my own notes and broke each algorithm into steps I could follow. Then I tested myself by explaining each concept to someone else. Wherever I couldn't explain it clearly, I'd found a gap, and I went back to it.
**Result.** I passed the module. More useful than the grade, I found out how I learn best, and I still do it. These days I explain a new concept in my own words to an AI assistant and ask it to point out what I got wrong or left out. It's the same method, with a listener that's always available.
#### Story 2
**Situation.** When I joined the Singapore Tourism Board, it was my first full-time data engineering job, and I was asked to design the end-to-end pipeline myself before bringing it to the team.
**Task.** I'd never designed a production system before, so it was daunting. I needed a design I could explain and defend.
**Action.** I started with what was around me. I studied how the team had built their existing pipelines and asked them why they'd chosen each approach, so I understood the trade-offs and not just the outcome. I read engineering blogs from large companies to see how others weigh the same choices. And I talked to the project managers and business analysts about the quirks in the data that the design would have to handle.
**Result.** I presented the design to the team, and after some changes it was approved. It's the design I went on to build. The biggest thing I learned was not to overcomplicate. For example, I'd planned to write a tracker file after every ID I pulled, so a restarted job could resume where it stopped. Then I realised the whole pull takes under 15 minutes. So I write the tracker once at the end and retry only the failures, and if the job dies, it just starts again.
### Teamwork


### Influence / Persuasion

### Ethical / Moral conflicts