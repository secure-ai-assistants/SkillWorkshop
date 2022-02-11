# Part One: Get to Grips with Skills and ASK
## Create the skill and provision resources 
* Go to the Alexa Skills Kit at https://developer.amazon.com/alexa/console/ask and log in with an Amazon account

* Select "create skill" and enter the name that people will call your skill by (this is what you’ll ask Alexa for when you test the skill)

* Choose the custom skill model and "Alexa-Hosted (Python)" resources, then click “create skill” and choose "High-Low Game Skill". We’ll use this as a template and add some new features to it

ASK will now create a basic conversation model for the skill and provision a Lambda function for your skill. This may take a few minutes. It should then place you into the ASK development environment. Time to get hacking!

## Test the Skill and See What it Does 
* Go to the “Test” tab at the top of the development environment. You will need to grant access to your microphone when prompted if you want to interact with the skill via speech
  * (you can also type your commands to Alexa - if you recall the request lifecycle, this just skips out the recording and transcription steps and feeds your typed request straight into the natural language processing model)

This part of ASK simulates using an Alexa device (like the Echo) or using Alexa on your phone. A key benefit is that you can see the request and response objects at each stage of the conversation. This can help you to understand what your skill is doing and diagnose any problems. 

* Start the skill by asking something like "open [skill name]". With this and all of the other prompts you can experiment with different phrasings to see how Alexa interprets them. Note that you don't need to prefix commands to the simulator with "Alexa" - everything here is obviously for Alexa!

Play the game for as long as you want, making note of the choices it offers you and how it responds to your input. You can see the JSON data that is being sent between ASK and AWS Lambda in the boxes on the right hand side of the screen.  

## Get to Know the Skills Kit Interface - Intents, Slots, and Utterances 
First, we’ll take a look at the conversation model.

* This is found under the “Build” tab (along the top of the screen). On the left hand menu go to "Interaction Model"-> "Intents" 

You can see a range of intents here, most of them present by default. The main intent that forms the entry point into the skill when it gets invoked is "Number Guess Intent"

* If you click on this intent you can see some different utterances for it. Saying any of these utterances will send this intent to the skill. Add in a new utterance (e.g. "I choose {number}")

You can see more details about this number slot further down the page – it has the type AMAZON.NUMBER (the default way of representing numbers with Alexa) and can have only one value. We need to tell Alexa what kind of information will be in the slot so that it can correctly deduce what intent the user wants (I.e. your skill won’t have to deal with something like “I choose apples”). We can see that this utterance will not prompt Alexa to ask the user for confirmation (you might want to do so before making a purchase, for instance) 
    
## Get to Know the Skills Kit Interface - the Lambda Function 
* Switch to the “Code” tab. This will open the code that runs on AWS Lambda. This code is run every time that a user says anything to the skill (I.e. this will run multiple times during a single play of the game) 

This looks very in-depth, but don’t panic (at least, not yet!). The left-hand menu shows us that the Lambda function is comprised of three files ("lambda-function.py", "requirements.txt", and "utils.py"). We are interested in the first of these. The second tells the server what other software is needed to run the Lambda Function, and the third contains some code that is called from the main python document. You can double click on its name to open it and look, but it’s not very interesting.

When you’re done exploring, make sure you switch back to "lambda-function.py". This file contains the code needed to respond to requests by people playing the game. Structurally, the code is divided into functions. In the Python programming language these start with “def” (as they define the function) followed by the name of the function and any information it requires to run. For example, the function that deals with players guessing numbers in the game is called "number_guess_handler" and takes some information called "handler_input" - see line 147 

If you look at the code underneath that makes up the function, the following is happening:
1. We access the number that the player is trying to guess by reading the session state (l151) 
2. We access the number that the player actually guessed by reading the “number” slot that is part of the NumberGuess intent (l152) 
3. We compare these numbers and choose some appropriate words to say back (l157-l170) 
4. If the player won by guessing correctly then we increment the games played counter and record that the game has ended (l172-173) 
5. We choose what to say to nudge the user if they don’t say anything for a while (l179) 
6. We take all of this and bundle it into a response object (the counterpart to the request object that we are provided when the user talks to the skill) 
7. But how does the code know which function to run for each intent? If you look above many of the functions in the document you can see an annotation that starts with an “@” symbol. Part of this refers to the intent that should trigger this function:  is_intent_name("NumberGuessIntent") 

* Choose one of the dialogue responses in the document and make a change to it (just make sure that what Alexa will say stays between the double quotes!). Go back to the testing tab and play the game again to check that your alterations work as you expect them to. You will need to save and deploy your changes (top right buttons) before they take effect. 

Every time the skill is run, it creates a log of the request and stores it in another product called Cloudwatch. You can check the Cloudwatch log for your skill by selecting “EU” from the CloudWatch Logs button in the test tab. The log at the top of the list will be for the most recent time you used the skill. If your skill has a problem then you can use these logs to learn more about what went wrong.

* Take a look at the most recent log and see what it contains. 

Because the skill retains its state between requests you might want to end the session and clear out anything that’s stored when you're testing it. You can do this by saying "stop" and triggering the AMAZON.StopIntent.

# Part Two: Modifying the Skill
We're now going to start modifying the skill to make use of some personal data. If you end up changing code and want a reference to check against, you can find the original source file at https://github.com/alexa-samples/skill-sample-python-highlowgame/blob/master/lambda/py/lambda_function.py

## Adding Some Data Collection – Checking and asking for permissions 
