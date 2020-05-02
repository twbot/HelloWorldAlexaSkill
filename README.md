<fs x-large>Writing the code for Alexa</fs>

-----------------------------------------------------------------------------------------------------------------------------

**Author:** Tristan Brodeur 

**Email:** brodeurtristan@gmail.com

**Date:** Last modified on 12/29/16 

**Keywords:** Alexa, Amazon Skills Kit</fs>

-----------------------------------------------------------------------------------------------------------------------------

<fs large>Overview</fs>

***Video Demonstration***

<a href="https://www.youtube.com/watch?v=m8i3vgwMH3k"><img src="./video.png" 
alt="Video Demonstration" width="200" height="200" border="10" /></a>

-----------------------------------------------------------------------------------------------------------------------------

1. <fs large>Create a Lambda Function</fs>

To create a lambda function, follow this tutorial: [Creating a lambda function](https://github.com/twbot/LambdaFunctionAlexa)

-----------------------------------------------------------------------------------------------------------------------------

2. <fs large>Add the code</fs>


```python
def lambda_handler(event, context):
	#Checks to make sure application id is same as alexa skill (links)
	if (event['session']['application']['applicationId'] !=
		#Change to application id listed under your alexa skill in the developer portal
		"amzn1.ask.skill.XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"):
		raise ValueError("Invalid Application ID")

	if event["session"]["new"]:
		on_session_started({"requestId": event["request"]["requestId"]}, event["session"])
		#check event types and call appropriate response
		if event["request"]["type"] == "LaunchRequest":
			return on_launch(event["request"], event["session"])
		elif event["request"]["type"] == "IntentRequest":
			return on_intent(event["request"], event["session"])
		elif event["request"]["type"] == "SessionEndedRequest":
			return on_session_ended(event["request"], event["session"])
####################################################################
def on_session_started(session_started_request, session):
	print "Starting new."
####################################################################
def on_launch(launch_request, session):
	return get_welcome_response()
####################################################################
def on_intent(intent_request, session):
	intent = intent_request["intent"]
	intent_name = intent_request["intent"]["name"]

	if intent_name == "Hello":
		return say_hello()
	elif intent_name == "AMAZON.HelpIntent":
		return get_welcome_response()
	elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
		return handle_session_end_request()
	else:
		raise ValueError("Invalid intent")
####################################################################
def on_session_ended(session_ended_request, session):
	print "Ending session."
####################################################################
def say_hello():
	session_attributes = {}
	card_title = "Hello World"
	speech_output = "Hello World"
	reprompt_text = ""
	should_end_session = True

	return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
####################################################################
def build_response(session_attributes, speechlet_response):
	return {
		"version": "1.0",
		"sessionAttributes": session_attributes,
		"response": speechlet_response
    }			
####################################################################
def build_speechlet_response(title, output, reprompt_text, should_end_session):
	#return data in json format to alexa skills kit
	return {
		"outputSpeech": {
     			"type": "PlainText",
     			"text": output
     		},
     		"card": {
     			"type": "Simple",
     			"title": title,
     			"content": output
     		},
     		"reprompt": {
     			"outputSpeech": {
     				"type": "PlainText",
     				"text": reprompt_text
     			}
     		},
     		"shouldEndSession": should_end_session
     	}
####################################################################			
```

******Make sure to change the application id listed in lambda_handler() to your own id once you create the Alexa Skill with Alexa Skills Kit******</fc>

-----------------------------------------------------------------------------------------------------------------------------

The handler function takes two parameters:

  * event: will contain an object with information about the request.

  * context: will represent the state of the Lambda function (for
  * example, how much time is left before the function times out). The
  * context parameter is also used for sending a response back to the
  * caller.

In this tutorial, we’ll run the web service on AWS Lambda, a service
that executes code in response to events, thus saving the developer
the trouble of maintaining a server. Lambda also links seamlessly to
the Alexa Skills Kit, which makes it an excellent tool for running the
code for Alexa Skills.

We will implement the server functionality as a python module, so to
complete the tutorial, you’ll need a basic understanding of python2.7.

Every request from the Alexa Skills Kit has a type, passed in event.request.type.

In all, there are three request types, all of which the service needs to
respond to separately:

  * Launch request: Sent when the user launches the Skill. The
  * example code calls the helper function onLaunch to initialize the
  * help function.

  * SessionEndedRequest: Sent when the user stops using the skill
  * by saying “exit”, by not responding for a while, or if an error occurs.
  * The example code calls the helper function onSessionEnded,
  * which is currently just a placeholder and doesn’t do anything.

  * IntentRequest: Sent when the user speaks a command that
  * maps to an intent. The example code calls the helper function
  * onIntent.

Amazon’s documentation explains that:

“[A]n intent represents a high-level action that fulfills a user’s
spoken request. Intents can optionally have arguments called
slots that collect additional information needed to fulfill the
user’s request.”

When a user speaks a command, Alexa converts it to text and
compares it to a list of phrases the application understands. Each
phrase is mapped to an intent definition so that your code doesn’t
have to worry about parsing the free-form input from the user but can
rely on the intents to organize its communication with the user.

-----------------------------------------------------------------------------------------------------------------------------

You’ll soon see how to specify the intents for your skill, but first, let’s
take a look at the onIntent function to see how Alexa handles
the intents.

```python
def on_intent(intent_request, session):
	intent = intent_request["intent"]
	intent_name = intent_request["intent"]["name"]

	if intent_name == "Hello":
		return say_hello()
	elif intent_name == "AMAZON.HelpIntent":
		return get_welcome_response()
	elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
		return handle_session_end_request()
	else:
		raise ValueError("Invalid intent")
```

As you can see, most of the function is an if..else structure that
compares the intent name from the request to a set of intents the
skill accepts. Depending on the intent name, it then calls a matching function.

For example, when Alexa receives a request with the intent name
Hello, it calls the corresponding say_hello() function.

-----------------------------------------------------------------------------------------------------------------------------

Looking at the say_hello function, 

```python
def say_hello():
	session_attributes = {}
	card_title = "Hello World"
	speech_output = "Hello World"
	reprompt_text = ""
	should_end_session = True

	return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
```

we can see that the function returns a build_response function that will pass arguments defined in the say_hello function.

-card_title: name of the card that will display on the Alexa app on your smartphone

-speech_output: what the Echo or Echo Dot will ouput once the function build_response function is called

-should_end_session: tells Alexa whether to end the skill session after a response has been outputed. This should be set to True in this case. If a reprompt text is defined, usually should_end_session would be set to false, however all we are doing with this function is asking for a simple output.

-----------------------------------------------------------------------------------------------------------------------------

After the code is added, [configure your lambda function with the Alexa Skills Kit.](https://github.com/twbot/CreatingAnAlexaSkill)