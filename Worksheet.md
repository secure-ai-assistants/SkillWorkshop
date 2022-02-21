# Part One: Get to Grips with Skills and ASK
## Create the skill and provision resources 
Because the underlying technical implementation of an Alexa skill is quite complicated, we're going to start with a template from Amazon and use this to explore how skills work and tinker with some of the functionality that's available. In this first part of the hands-on exercise, we're going to clone the skill template that Amazon provides, set up the cloud resources we need in order to run the skill, and make sure that all the components are plugged into each other.

* Go to the Alexa Skills Kit at https://developer.amazon.com/alexa/console/ask and log in with an Amazon account.

* Select "create skill" and enter the name that people will call your skill by (this is what you’ll ask Alexa for when you test the skill).

* Choose the custom skill model and "Alexa-Hosted (Python)" resources, then click “create skill” and choose "High-Low Game Skill". We’ll use this as a template and add some new features to it.

ASK will now create a basic conversation model for the skill and provision a Lambda function for your skill. This may take a few minutes. The Lambda function will be hosted and run on AWS, but you can edit and deploy it from within ASK so you don't need to switch between the two. When the provisioning is done it should place you into the ASK development environment. Time to get hacking!

## Test the Skill and See What it Does
Okay, so we cloned this skill but what does it actually do? We'll have a look at the code soon, but for now let's just run it with the Alexa simulator and see what happens. You'll need to use this tool later on to test and debug the changes you make to your skill.

* Go to the “Test” tab at the top of the development environment. You will need to grant access to your microphone when prompted if you want to interact with the skill via speech. You can also type your commands to Alexa - if you recall the request lifecycle, this just skips out the recording and transcription steps and feeds your typed request straight into the natural language processing model.

This part of ASK simulates using an Alexa device like the Echo, including any visual display elements that might be included in a response. It also shows you the JSON data that's being sent back and forth when you use Alexa. When you build Alexa responses later in the workshop, these will be translated directly into these JSON objects that you see now.

* Try out some of the commands you've been using with Alexa over the past few weeks to get an idea of how the simulator works. You can see the JSON data that is being sent between ASK and AWS Lambda in the boxes on the main part of the screen. Note that you don't need to prefix commands to the simulator with "Alexa" - everything here is obviously for Alexa!

* Start your skill by asking something like "open [skill name]". With this and all of the other prompts you can experiment with different phrasings to see how Alexa interprets them. Play the game for as long as you want, making note of the choices it offers you and how it responds to your input.
 
You could also test the skill using Alexa on your phone–it will automatically appear under "Dev" in the "Skills and Games" -> "Your Skills" section of the app–but being able to see the JSON request and response objects in the simulator will come in very handy later on. If there's a problem with your skill's code it might not _say_ anything, but you'll still be able to check the response to see what Alexa thinks is happening and (hopefully) diagnose any problems.

## Get to Know the Skills Kit Interface - Intents, Slots, and Utterances 
Now th fun begins! We're going to explore the conversation model for the skill. As covered in the lecture, these are formed of three main components: utterances, intents, and slots. Intents define the states your skill can be in; utterances control what users can say to enter these states; and slots allow users to pass additional information to your skill along with an intent (e.g. the number they would like to guess).

* The conversation model can be found under the “Build” tab (along the top of the screen). On the left hand menu go to "Interaction Model"-> "Intents" 

You can see a range of intents here, and most of them are present by default - every skill will accept the "Help", "Stop", "Cancel", and "NavigateHome" intents. Several others are commonly used by a range of skills and are build into the ASK (here "Yes", "No", and "Fallback"). The main intent that forms the entry point into the skill when it gets invoked is "NumberGuessIntent".

* If you click on this intent you can see the different utterances that will trigger it. Saying any of these phrases will send this intent to the skill. Add in a new utterance (e.g. "I choose {number}").

The NumberGuessIntent has a single slot called "number", and you can see more details about this further down the page – it has the type AMAZON.NUMBER (this is the default way of representing numbers with Alexa) and can have only one value. Alexa needs to know what kind of information will be in the slot so that it can correctly deduce what intent the user wants (i.e. your skill won’t have to deal with something like “how about apples”). This means that such an utterance would be captured instead by the FallbackIntent, which fires whenever the user says something that isn't captured by another intent. We can also see that this utterance will not prompt Alexa to ask the user for confirmation (you might want to do so before making a purchase, for instance) 
    
## Get to Know the Skills Kit Interface - the Lambda Function
Next up we're going to take a look at the code that makes up our lambda function. This will run every time that somebody talks to the skill (i.e. multiple times each interaction). If you haven't had much experience with coding and programming languages this might be a bit daunting, but we'll go through what's happening step by step.

* Switch to the “Code” tab. This will open the code that runs on AWS Lambda. Take a look at each of the three files in the left-hand menu.
 
As you might have guessed, we are interested in "lambda-function.py". The second file, "requirements.txt", tells the server what other software is needed to run the Lambda Function, and "utils.py" contains some helper code that is called from the main python document. When you’re done exploring, make sure you switch back to "lambda-function.py".

This file contains the code needed to respond to requests by people playing the game. Structurally, the code is divided into functions. In the Python programming language these start with “def” (as they define the function) followed by the name of the function and any information it requires to run.Each intent in the conersation model is given to a seperate "handler" function; the function that deals with players guessing numbers in the game is called "number_guess_handler" and takes some information called "handler_input" - see line 147. But how does the code _know_ which function to run for each intent? If you look above many of the functions in the document you can see an annotation that starts with an “@” symbol. This contains a reference to the intent that should trigger this function:  is_intent_name("NumberGuessIntent").

If you look at the code underneath that makes up the function, the following is happening:
1. We access the number that the player is trying to guess. In order to retain it between invocations of the skill, this has previously been stored as a "session variable". Thus we can get to it by reading the session state (line 151).
2. We access the number that the player actually guessed by reading the “number” slot that is part of the NumberGuess intent (line 152).
3. We compare these numbers and choose some appropriate words to say back (lines 157-l170).
4. If the player won by guessing correctly then we increment the games played counter and record that the game has ended (lines 172-173).
5. We choose what to say to nudge the user if they don’t say anything for a while, sometimes called a re-prompt (lines179).
6. We take all of this and use it to build it into a response "object", which is the counterpart to the request object that we are provided when the user talks to the skill. The response builder presents a helpful API that takes the information we pass it and translates it into some JSON that we then "return" to end the function.

* Choose one of the dialogue responses in the source code and make a change to it (just make sure that what Alexa will say stays between the double quotes!). Go back to the testing tab and play the game again to check that your alterations work as you expect them to. You will need to save and deploy your changes (top right buttons) before they take effect. 

Every time the skill is run, it creates a log of the request and stores it in another Amazon product called Cloudwatch. You can check the Cloudwatch log for your skill by selecting “EU” from the CloudWatch Logs button in the test tab. The log at the top of the list will be for the most recent time you used the skill. If your skill has a problem then you can use these logs to learn more about what went wrong.

* Take a look at the most recent log and see what it contains. 

Because the skill retains its state between requests you might want to end the session and clear out anything that’s stored when you're testing it (this is what allows us to keep track of the number the user is trying to guess). You can do this by saying "stop", which will trigger the AMAZON.StopIntent.
