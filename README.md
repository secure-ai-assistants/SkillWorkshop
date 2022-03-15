# Explainability Workshops: From Mobile Ecosystems to AI-Voice Assistants 

This repository contains the resources you will need during the workshop.

## Part 1
For the first part of the session, you can find the voice assistant skill datasets in the `data` folder. These are Excel spreadsheets set up with tables to allow for easier sorting and filtering, but if you have a another preferred way of exploring data then feel free to use that instead.

The full citation for the paper where data originates from is:

J. Edu, X. Ferrer Aran, J. Such and G. Suarez-Tangil, "SkillVet: Automated Traceability Analysis of Amazon Alexa Skills," in _IEEE Transactions on Dependable and Secure Computing_, doi: 10.1109/TDSC.2021.3129116.

## Part 2
For the second part of the session, there are worksheets that step through how to edit a skill in the `guides` folder. Because this involves editing code, it might be challenging if you don't have much technical experience. Work together in your groups to make sure that people with differing levels of experience work together. Don't hesitate to ask for pointers if you get stuck.

In general, diagnosing programming problems follows a loop:
1. Make a change to the code that doesn't do what you want it to
2. Obtain an error message
3. Research what the error message means (e.g. using Google)
4. Change the affected code and try again

Voice assistants are a little tricky in this regard, because when Alexa encounters a problem it will often say something generic like "I'm sorry, the requested skill encountered a problem". In these cases there are two main things you can do to find out what the problem is:
1. Open up Cloudwatch (as described in the worksheet) and see if there is an error in the log
2. Add print statements to your code to see how far the skill gets before encountering an issue. The output of these statements will be visible in Cloudwatch.
