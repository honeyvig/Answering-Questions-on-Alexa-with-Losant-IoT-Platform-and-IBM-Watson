# Answering-Questions-on-Alexa-with-Losant-IoT-Platform-and-IBM-Watson
To build an Alexa skill that answers questions by integrating the Losant IoT platform and IBM Watson, you'll need to follow a few key steps:

    Set Up Losant IoT Platform: This is where the IoT data from sensors or devices will be stored and processed.
    Create an Alexa Skill: You'll write an Alexa skill that communicates with the Losant platform to retrieve the data.
    Integrate with IBM Watson: IBM Watson's Natural Language Understanding (NLU) can process the questions from Alexa and trigger the required actions based on user queries.

Step 1: Set Up Losant IoT Platform

    Sign up and create an account on the Losant IoT Platform: Losant IoT.
    Create an Application in Losant to manage the data flow.
    Set up devices (sensors, or other IoT devices) that will send data to your Losant application. For instance, you could simulate a smart device like a temperature sensor.

Step 2: Set Up IBM Watson

    Sign up and create an account on IBM Cloud: IBM Cloud.
    Create an IBM Watson NLU service instance: IBM Watson NLU.
    Get the API Key and URL for the NLU service to use later in your Alexa skill.

Step 3: Create an Alexa Skill

To create the Alexa skill, you'll use the Alexa Skills Kit (ASK), which provides tools and resources for creating voice-driven Alexa skills.

    Set Up Your Alexa Developer Account: Visit the Alexa Developer Console and create a new skill.

    Create Intent Schema: Define the type of questions Alexa will answer, such as "What's the temperature?" or "What's the status of the sensor?"

Here is a simple example of an Intent Schema (in JSON format):

{
  "intents": [
    {
      "name": "GetTemperatureIntent",
      "slots": []
    },
    {
      "name": "GetSensorStatusIntent",
      "slots": []
    }
  ]
}

    Sample Code for Alexa Skill (Node.js):

This Node.js code for your Alexa skill will integrate IBM Watson NLU and Losant IoT to answer questions like "What's the temperature?" or "What is the sensor status?".
Step 4: Code for Alexa Skill

Here’s a sample code using Node.js, Alexa Skills Kit SDK, Losant, and IBM Watson.

const Alexa = require('ask-sdk-core');
const axios = require('axios');

// IBM Watson NLU API credentials
const watsonApiKey = '<YOUR_WATSON_API_KEY>';
const watsonUrl = '<YOUR_WATSON_API_URL>';

// Losant API credentials
const losantAccessToken = '<YOUR_LOSANT_ACCESS_TOKEN>';
const losantDeviceId = '<YOUR_LOSANT_DEVICE_ID>';

// Helper function to query IBM Watson NLU
const queryWatson = async (text) => {
  try {
    const response = await axios.post(
      `${watsonUrl}/v1/analyze`,
      {
        text: text,
        features: {
          entities: {},
          keywords: {}
        }
      },
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Basic ${Buffer.from('apikey:' + watsonApiKey).toString('base64')}`
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Error querying Watson:', error);
    throw new Error('Could not process the request');
  }
};

// Helper function to get data from Losant
const getLosantData = async () => {
  try {
    const response = await axios.get(
      `https://api.losant.com/applications/<YOUR_APP_ID>/devices/${losantDeviceId}/state`,
      {
        headers: {
          'Authorization': `Bearer ${losantAccessToken}`
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Error fetching data from Losant:', error);
    throw new Error('Could not retrieve sensor data');
  }
};

// Alexa skill handler for GetTemperatureIntent
const GetTemperatureIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetTemperatureIntent';
  },
  async handle(handlerInput) {
    const losantData = await getLosantData();
    const temperature = losantData.state.temperature; // Assuming temperature is part of the sensor state

    return handlerInput.responseBuilder
      .speak(`The current temperature is ${temperature} degrees.`)
      .getResponse();
  }
};

// Alexa skill handler for GetSensorStatusIntent
const GetSensorStatusIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'GetSensorStatusIntent';
  },
  async handle(handlerInput) {
    const losantData = await getLosantData();
    const status = losantData.state.status; // Assuming status is part of the sensor state

    return handlerInput.responseBuilder
      .speak(`The sensor status is currently ${status}.`)
      .getResponse();
  }
};

// Alexa skill handler for default intent (questions from user)
const DefaultIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest';
  },
  async handle(handlerInput) {
    const userQuery = handlerInput.requestEnvelope.request.intent.slots.query.value;
    
    // Use Watson NLU to process the user's query
    const watsonResponse = await queryWatson(userQuery);
    const mainEntity = watsonResponse.entities ? watsonResponse.entities[0] : null;

    let responseMessage = 'Sorry, I could not understand your question.';

    if (mainEntity) {
      // Process entity and make a response based on Watson's analysis
      if (mainEntity.type === 'keyword' && mainEntity.text === 'temperature') {
        const losantData = await getLosantData();
        const temperature = losantData.state.temperature;
        responseMessage = `The current temperature is ${temperature} degrees.`;
      }
    }

    return handlerInput.responseBuilder
      .speak(responseMessage)
      .getResponse();
  }
};

// Lambda handler for Alexa Skill
exports.handler = Alexa.SkillBuilders.custom()
  .addRequestHandlers(
    GetTemperatureIntentHandler,
    GetSensorStatusIntentHandler,
    DefaultIntentHandler
  )
  .lambda();

Step 5: Explanation of the Code

    IBM Watson NLU:
        The queryWatson function uses the IBM Watson Natural Language Understanding API to analyze user queries, extracting entities or keywords to understand what the user is asking for (e.g., asking for temperature or sensor status).

    Losant IoT:
        The getLosantData function makes a request to the Losant IoT platform to retrieve the state of a sensor device (e.g., temperature, status).

    Alexa Intent Handlers:
        The GetTemperatureIntentHandler fetches the current temperature from Losant and responds to the user.
        The GetSensorStatusIntentHandler retrieves the current sensor status and responds.
        The DefaultIntentHandler handles general queries, uses Watson to analyze them, and responds appropriately (e.g., answering temperature-related queries).

    Deployment:
        Deploy the Alexa skill to the Alexa Developer Console.
        Configure the Endpoint to point to your Lambda function that includes the code above.
        Test the Alexa skill with queries like "What's the temperature?" or "What's the sensor status?"

Step 6: Test and Launch

    Test the Alexa Skill: Use an Alexa-enabled device or the Alexa app to test the skill.
    Refine the Skill: Refine your skill to handle more questions or interactions based on the Watson NLU analysis.

Conclusion:

By integrating IBM Watson and Losant IoT with an Alexa skill, you can create an intelligent voice assistant that answers questions by fetching real-time data from IoT devices. Watson helps process the natural language queries, and Losant handles the IoT data collection and processing. This solution allows for effective, scalable interaction with your IoT-enabled devices.
