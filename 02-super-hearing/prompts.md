# 02 · Super Hearing — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session you built a context file and met the
company.

You still do not know what actually went wrong.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.
Use the rook-wiki connector to find the Customer interviews database and read every interview in it. These are four conversations with the people who run the console for our responders. Tell me what they're unhappy about, group it, tell me how many of the four raised each thing, and quote one line for each so I can hear how they actually said it.

### 2.
Show themes that are not mentioned here but might be important and also tell me why you think this might be important and why they are not being mentioned in the first answer

### 3.
Are there any themes that are unique to each interview that are not mentioned by others? quote one line for each

### 4.
Show me the analysis based on the ones that are too busy and ones that have no calls at all, also quote one line for each.

### 5.
Use the rook-database connector to read the support_tickets table. Same treatment as before: group them, tell me how many are in each group, and quote one line from each.

### 6.
Why are some tickets not closed?

### 7.
Combine the interviews anlaysis with ticket analysis, i want to know which ticket themes are confirmed by interviews, and vice versa.

### 8.
Any themes that are unique in ticket and interview analysis?

### 9.
You've now read both folders. Where do they disagree? What's loud in the interviews but rare in the tickets, and what's all over the tickets that nobody brought up in the interviews?

### 10.
How do i tell what is true when both source of truths contradict each other?

### 11.
Check the data before the release, what does it tell us? Any trends?

### 12.
Now compare pre-release with post-release data

### 13.
based on what we discussed with only the 'interviews', what is the trouble with 4.2?

And based on what we discussed with only 'ticket analysis', what is the trouble with 4.2?

### 14.
i only want one sentence for each

### 15.
then after anlyzing both, what is the problem in 1 sentence
