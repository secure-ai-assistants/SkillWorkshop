# Adding Some Data Collection – Being Sneaky 

We’re going to add a special message at the end of the game if it’s the user’s birthday today, but we’re not going to use the standard API to do it. Sometimes the easiest way to get information is just to ask for it... 

* Switch back to the build tab and bring up the list of intents. Add a new intent to capture the user’s birthday.

You’ll need to consider what utterances a user might use to tell the skill their date of birth, and what slot type would be appropriate for capturing birthdates.

* Take a look through the list of slot types and choose one. If you’re feeling confident you could also make a custom slot type to deal with this 

* Save and build the voice model (at the top of the page) before returning to the Code tab 

Now we need to make a new intent handler for the sneaky birthday intent. Adapt the code that defines the LaunchRequest to make a definition of your new intent handler (this should be placed after the end of LaunchRequest).

<details>
  <summary>Sample code if you want something to refer to or check against</summary>
  
```
@sb.request_handler(can_handle_func=lambda input: is_intent_name("your intent name")(input)) 
def sneaky_bday_handler(handler_input): 
```
</details>

* Using the handler for NumberGuessIntent as a guide, define a variable that captures the users date of birth from the request object. Copying the way that the skill stores the number of games played, also store the birthday in the session object

<details>
  <summary>Sample code if you want something to refer to or check against</summary>
  
```
bday = handler_input.request_envelope.request.intent.slots["slot name"].value

attr = handler_input.attributes_manager.persistent_attributes
attr['bday'] = bday
handler_input.attributes_manager.session_attributes = attr
 ```
</details>

* Using one of the other intent handlers as a guide, have the intent response say something to the user along the lines of “say yes to start the game” so that they will subsequently trigger the AMAZON.YesIntent. Include a reprompt so that Alexa keeps the skill open after the user replies.

* In LaunchIntent, make the spoken response ask the user to say their birthday 

To correctly interpret dates we need to import, you guessed it, `date` from the `datetime` module. Amazon will supply us the date from the slot as a string formatted using the international standard for date formats (the ISO format) that takes the form "YYYY-MM-DD". Python has a special function for interpreting dates in this format, e.g. `date.fromisoformat("2012-10-13")`. This is important because when you get the current date with Python (using `date.today()`) then they need to be in the same format so that you can compare them.

* Import `date` at the top of the source document

*    In NumberGuessIntent, find the part of the if statement where the user’s guess matches the secret number - we'll say happy birthday if the user won the game. In the appropriate part of the if statement, access the session attribute for birthday you stored earlier, convert it from the ISO format, and assign it to a variable.

<details>
  <summary>Sample code if you want something to refer to or check against</summary>
  
```
bday = date.fromisoformat(session_attr['bday'])
```
</details>

Compare the user’s birthday to today’s date using an `if` statement, and, if they are the same, modify the speech_text to wish them happy birthday! 


<details>
  <summary>Sample code if you want something to refer to or check against</summary>
  
```
extra = ""
if bday.month == n.month and bday.day == n.day:
    extra = "Oh and by the way, happy birthday! "
        
speech_text = (
    "Congratulations. {} is the correct guess. "
    "You guessed the number in {} guesses. {}"
    "Would you like to play a new game?".format(
    guess_num, session_attr["no_of_guesses"], extra))
```
Notice how `extra` is normally blank - if it gets inserted into `speech_text` when it's empty then it won't affect the spoken response (obvious if you thnk about it!).
</details>

Note: you may want to test this out in an easier to access part of the skill until you get it working. Trying to debug logic that you can only reach after ~10 requests is going to get very tedious very quickly 

You might find the documentation for the date object useful both for accessing individual parts of a date, as well as for getting the current date: https://docs.python.org/3/library/datetime.html#datetime.date 

Take a look at how the dates the user speaks to the skill get represented by Alexa. Can you figure out what’s going on here? You can check the documentation here: https://docs.aws.amazon.com/lex/latest/dg/built-in-slot-date.html 
