# Part Two: Modifying the Skill
We're now going to start modifying the skill to make use of some personal data. If you end up changing code and want a reference to check against, you can find the original source file at https://github.com/alexa-samples/skill-sample-python-highlowgame/blob/master/lambda/py/lambda_function.py

## Adding Some Data Collection – Checking and asking for permissions 

At present the skill doesn't use any user data. We’re going to update it to greet the user by name to make the experience more personal. The first thing we need to do is to tell ASK that our skill might need access to the user's name.

* Switch back to the build tab and bring up the permissions menu (under "Tools"). Take a look at the different permissions available and enable access to "Given Name".

As the suggested by the presence of the permissions menu, skills need to be granted permission in order to access personal data. It might be that someone runs the skill without having granted those permissions, so when your skill runs it needs to check whether this has been granted. This information is contained within the "context" data as part of the request (remember back to the lecture). In the code we can access it as `handler_input.request_envelope.context.system.user.permissions`

* Switch to the code tab and go to the function that handles LaunchRequest (i.e. when the skill is opened).
* To make it easier to use and refer to this part of the context data, we’re going to assign it to a variable within the launch request handler. This will give us a short 'pointer' that we can use without having to type the unwieldy path: `system = handler_input.request_envelope.context.system`. Put this somewhere near the top of LaunchRequest.
* Now we can check if we have any permissions. If the object that’s supposed to contain them is empty (represented in Python as “None”) then we have not been granted any permissions: `if system.user.permissions is None:` (notice how we refer to our new variable `system` to shortcut the long line of text above)

Whatever we put next will run if the `if` condition is true (that is, if the list of permissions the skill is granted is empty). If this is the case then we want to prompt the user to grant us the necessary permissions to the skill. Using the response at the end of LaunchRequest as a template, we will build a response that includes some spoken dialogue as well as a card that will appear in the alexa app (or on screen for devices like the Echo Show) that tells the user what they need to do.

* Copy the three lines at the bottom of LaunchRequest that set (1) the speech text; (2) use the response builder; and (3) return the response object. Alter them to make the skill say something along the lines of the above. Make sure that these lines of code are indented under the `if` statement (this is how Python will know that they are associated).

You'll notice the `format` command at the end of the speech text. This will insert whatever is within the parentheses of `.format()` where there's two curly braces `{}` in the string. The code you copied uses this to insert the name of the skill, stored in the cunningly named varieble `SKILL_NAME`. This is set at the top of the source document, and the idea is that if you ever want to rename your skill, then you only need to do this once in the code. For now you can use it or remove it and just type the skill name in - up to you.

We also wanted to show a card. The card template we’re going to use is specifically designed to ask for permissions. To create it we need to specify which permissions we’re asking for; different permissions are specified based on where the information falls in the overall data model. Given name data is stored within a user profile, so the permission is specified as `["alexa::profile:given_name:read"]` (while this uses colons to seperate different levels of an object instead of the dots we used above, this is because the permissions definition is not python code - it's a string! Amazon is free to do whatever they want here, so long as they know how to interpret it later.)

The code that defines the permissions card isn't included in the project by default - you might never use it, and it takes up additional space and processing power to include it just in case. This is conventionally done at the top of the document, so that we include everything we need before we use it. You'll notice that there are already several of these `import` statements already there.

* At the top of the source document add the following line to `import` the permissions consent card: `from ask_sdk_model.ui.ask_for_permissions_consent_card import AskForPermissionsConsentCard`
* Create a variable to hold your `AskForPermissionsConsentCard`. You'll need to pass in the permission you want to ask for in parentheses (by including extra strings here you could ask for more permissions if you wanted to).
* Modify your line in LaunchRequest that uses the response builder. Instead of sending a reprompt, we want to send a card, so change the `ask` command into `set_card` and pass in the name of your permissions card variable.

<details>
  <summary>Sample code if you want something to refer to or check against</summary>
  
```
speech_text = "{} needs to know your first name to provide a more personal experience. To continue, please visit your Alexa app to give permission.".format(SKILL_NAME) 
card = AskForPermissionsConsentCard(["alexa::profile:given_name:read"])) 
handler_input.response_builder.speak(speech_text).set_card(card) 
return handler_input.response_builder.response
```
</details>

Finally we put these together and send back the response. Notice how we can keep adding things to the response builder to create the user experience we want (you can see the use of reprompt at other places in the skill, which will cause alexa to say the specified text if the user doesn’t respond after around 8 seconds). 

Before we test this, we need to make sure we reference the extra code required to specify the permission request card. This goes at the top of the file 

from ask_sdk_model.ui.ask_for_permissions_consent_card import AskForPermissionsConsentCard 

Okay! Save and deploy the code, and then open the skill using the Alexa Simulator in the test tab. You might find that you want to keep this open in another browser tab to make it easier to switch back and forth between the simulator and the code. 

Run the skill – it should prompt you to give it permissions with speech and a card. You can see the JSON description of the response the skill sent back in the right hand box. This should match the response detailed in the Cloudwatch logs 

You can grant your skill permission in the Alexa app or at https://alexa.amazon.co.uk 
