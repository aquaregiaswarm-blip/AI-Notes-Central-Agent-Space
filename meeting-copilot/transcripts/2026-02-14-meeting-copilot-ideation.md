# Meeting Copilot — Ideation Session Transcript
**Date:** 2026-02-14
**Participants:** Jonathan Gough, Roy Bales, Clarice (AI)
**Channel:** Clarice's Hangout (Telegram)

---

**Jonathan Gough:**
I have an idea for an application that I would like you to help Roy and I flush out. We're not building anything yet. Your job is to ask us questions about the application. How it would work? Think about functionality that we would need functionality that we would not need. That way we can fully scope out if we want to do this and how we would do it. So the idea is application for a computer. I'm thinking if it's a Mac and electron app, I'm not sure what the equivalent would be for a PC or Linux. Is there an equivalent? We want the app to record speech to text. We want to turn the app on during a conversation, for example, a Microsoft teams meeting a WebEx, a zoom meeting and record the audio. It would then turn that into text. We would need to find a way in parallel be sending the transcript of the full conversation every every minute or so to an llm and have that llm a reasoning model. I'm thinking be considering the context of the call itself. It would be considering the client that is on the call. It will be considering what we as an organization can offer them with the purpose of returning to the user ideas, questions, bullet point notes in real time. For example, Roy and I are talking to Delta airlines about AI ops project running on IBM I series looking for anomalies. This app would be prompting us with questions to ask about how the hardware is managed. What software is being run? Have they considered XYZ? It would not bring back things that had already been brought up. However, it would take those things into account potentially go out to the internet and gain context about details that are being discussed. Bring them back and provide those as notes, point of reference and also potentially put those into a question. Keep this as an idea. Ask Roy and I no less than 10 clarifying questions. Not all at once but one by one.

---

**Jonathan Gough:**
The first iteration would be for Roy and I. The persona is a high-end consultant speaking to clients a mid to high level thought leader within within an organization talking to peers.

---

**Roy Bales:**
In a perfect world it would be part of the meeting (most of which are teams based) as an agent etc and be able to align who is speaking context, but being able to capture both speakers and microphone would be needed for our MVP even if it doesn't know who is speaking.

---

**Jonathan Gough:**
Collecting all meeting audio and doing speech to text is a great first start. Diarization would be an upgrade.

---

**Roy Bales:**
Overlay or sidebar would be my preference, ideally I could use it during an in-person meeting as well.

---

**Jonathan Gough:**
I'm thinking the app would have running notes on the left hand side and on the right hand side it would have questions and prompts that would show up. It would have the ability to X out something that you wouldn't want to ask. A checkbox to say that you know it already or to acknowledge that you asked the question or got an answer to that question inadvertently. The idea being that you would be able to clear the screen of things you didn't want in your view while designating them as positive or negative. I think there would be an option to see the transcript live if you wanted to.

---

**Jonathan Gough:**
We would have a pre-populated list of vendors, products and services. We would also have a list of clients and a history of summarized notes in the database. This would be like a front end to our notes on all of our opportunities and all of our meetings and everything that we're doing. That way, if we're having the second call the third call the fourth call. We have a summarized context of what their current stack is, what the expectations are, etc.

---

**Jonathan Gough:**
Yes, we would have a list based on the conversations and what we know of their current internal stack for the client. That list is going to be populated based on our historical notes. Pre-populated list of vendors will likely include things they already have in house. In addition to things that they do not have. We are a value-added reseller and sell almost everything. Therefore, there will be overlap and there will be mismatches. This scope at this point does not include external research.

---

**Jonathan Gough:**
Yes, I think an optional drop down for a meeting type is a welcome addition. It is not mandatory. The model would be able to intuit based on the conversation what's happening. If it's the first of its kind, like there's no context, we might want to wait until the conversation has elapsed for a minute or so before trying to provide suggestions. Quick question for you. How many more questions do we have? These are all great questions.

---

**Jonathan Gough:**
Once again, great question. Based on the meeting type that would be selected if selected, a set of after meeting actions would take place by default. If it's a client meeting, it would create artifacts for our records. It would update internal notes. It would create action items and it would create a stock set of meeting summary notes that we could send out as an email. It would not send any emails however. It would simply provide formatted texts that could be copy pasted. Maybe one day exported to an email client.

---

**Roy Bales:**
I'm a fan of middle ground, trust but verify.

---

**Roy Bales:**
Internal tool only for now, only Jonathan and I will use/have access and we can add extra security later.

---

**Jonathan Gough:**
This is going to be a local app to start with. We want this tight individual closed loop. V1 is going to help us understand better what it is that we need want and how well this will work. That being said, we would like you to ask us questions to better flush out the architecture. Or actually we would like you to offer us suggestions on how we would architect this for a local deployment only. What do we need to think about? How complex would it get? What's the easy button? Not so easy button and the super hard button and the insurmountable Hill.

---

**Jonathan Gough:**
I have a clarifying question. When I clicked open the application an electron application, will that automatically start up the vector DB? The sqlite DB is using electron. Is that going to manage the back end part of this for me or am I going to have to set that up on my own? I want to make this simple, less headache to start stop etc. if we have too many moving pieces for a V1, it'll be delicate. It'll break. We'll never use it. Help me understand the practical side of using it. That will inform how I think about it.

---

**Jonathan Gough:**
We want to run the speech to text either locally or pointing at an external endpoint. That endpoint may be a cloud API or we would specify a specific API endpoint that we are self-hosting. Notifications need to be silent. If a lot of things come back and pop up on the screen they need to auto scroll up. That's the purpose of having a check mark or an x to clear things out. Which reminds me we may need to limit the number of things that come back all at once.

---

**Jonathan Gough:**
I think a max of five per cycle is appropriate. I think there should be a setting where we can change it. I think being able to pause speech to text is also a v1 feature. We should also be recording and capturing the full text transcript as one of the output artifacts.

---

**Roy Bales:**
Let's add patterns to v2.

---

**Jonathan Gough:**
For V1 let's say it is configurable. The specification would be using the standard OpenAI format. That would allow us to use Ollama, LiteLLM, vLLM, or any other OpenAI API in a cloud or neocloud.

---

**Roy Bales:**
Can we do a combination, store them flat as well as in a format that can be searchable within the app? App should auto update but with human in the loop.

---

**Jonathan Gough:**
It should be captured in the notes for current meeting as a correction. It should be flagged as a discrepancy and let the user decide what goes in the summary record. Don't touch the earlier meeting notes.

---

**Jonathan Gough:**
I like the auto prep mode approach.

---

**Jonathan Gough:**
Yes, we can view and edit and update client attributes. I want all of that to be updated organically from meeting notes, however we should have the option to add to it in case we get info not in a meeting.

---

**Roy Bales:**
You've asked FAR more than 10 questions.

---

**Jonathan Gough:**
Write this up as 1. a detailed requirements document 2. An application summary of features and functionality 3. create a new directory for transcripts and put the entire transcript of this conversation in it as a md file.

---

*End of transcript.*
