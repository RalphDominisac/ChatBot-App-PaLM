# AI-Powered Chatbot on Cloud Run Using Google's PaLM API

## Project Overview

This project demonstrates the creation of an AI-powered chatbot using Google’s PaLM API, deployed on Cloud Run. The goal is to build a scalable and intelligent chatbot that can handle natural language conversations by leveraging Google's generative AI capabilities. The project integrates a Python-based Flask web application, which communicates with the PaLM API through Vertex AI. It is then containerized with Docker and deployed on Cloud Run for a fully managed, scalable environment.

## PaLM API

The **Pathway Language Model (PaLM)** API enables access to Google's advanced language models, capable of performing tasks like text generation, translation, summarization, and answering queries. PaLM is a robust tool for building intelligent, responsive chatbots.

## Vertex AI SDK for Python

The **Vertex AI SDK for Python** simplifies interaction with Vertex AI, streamlining the process of training models, making predictions, and deploying AI models. It provides an easy way to build and deploy AI-driven applications using Google Cloud’s infrastructure.

## Project Structure

- **index.html**: Frontend for the chatbot, where users input queries.
- **app.py**: Python Flask backend that interacts with the PaLM API.
- **Dockerfile**: Containerizes the application for deployment.
- **requirements.txt**: Lists the required Python packages.
- **test.py**: A simple script to interact with the PaLM API in a terminal-based environment.

## Getting Started

### Step 1: Install Dependencies

Make sure Python 3.8+ is installed. Install the required dependencies using the following command:

```bash
pip install -r requirements.txt
```

`requirements.txt` includes:

```
google-cloud-aiplatform >= 1.31.0
flask
google-cloud-logging
```

### Step 2: Set Up PaLM API Access

Before you can use the PaLM API, ensure that you have set up your Google Cloud project and enabled the Vertex AI API. You will need to authenticate using a service account key, which can be set up using the following command:

```bash
gcloud auth activate-service-account --key-file=YOUR-SERVICE-ACCOUNT-KEY.json
```

### Step 3: Flask Application Backend

#### app.py

This Flask app serves as the backend of the chatbot. It initializes the PaLM chat model and handles user queries. Here’s the relevant code:

```python
from flask import Flask, render_template, request, jsonify
import vertexai
from vertexai.language_models import ChatModel
import os

app = Flask(__name__)
PROJECT_ID = "your-project-id"  # Replace with your Google Cloud project ID
LOCATION = "us-central1"  # Replace with the appropriate location

vertexai.init(project=PROJECT_ID, location=LOCATION)

def create_session():
    chat_model = ChatModel.from_pretrained("chat-bison@001")
    chat = chat_model.start_chat()
    return chat

def response(chat, message):
    parameters = {
        "temperature": 0.2,
        "max_output_tokens": 256,
        "top_p": 0.8,
        "top_k": 40
    }
    result = chat.send_message(message, **parameters)
    return result.text

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/palm2', methods=['GET', 'POST'])
def vertex_palm():
    user_input = request.args.get('user_input', '')
    chat_model = create_session()
    content = response(chat_model, user_input)
    return jsonify(content=content)

if __name__ == '__main__':
    app.run(debug=True, port=8080, host='0.0.0.0')
```

### Step 4: Frontend (HTML)

The `index.html` file is the user interface of the chatbot. It contains a simple input field and a button to send queries to the chatbot:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>TechTrapture Chatbot</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
      /* CSS styles for the chatbot UI */
      body {
        font-family: Arial, sans-serif;
        background-color: rgb(205, 204, 211);
      }
      .chat-container {
        background-color: #fff;
        border-radius: 10px;
        max-width: 600px;
      }
      .header {
        background-color: #0b041e;
        color: white;
        padding: 20px;
      }
      .chat-form {
        display: flex;
        justify-content: space-between;
        margin: 0 20px;
      }
      #user-input {
        flex-grow: 1;
        border: 1px solid #ccc;
        padding: 10px;
      }
      #submitBtn {
        background-color: rgb(13, 2, 28);
        color: white;
        cursor: pointer;
      }
      #result {
        padding: 20px;
        background-color: #f8f8f8;
      }
    </style>
  </head>
  <body>
    <div class="chat-container">
      <h1 class="header">Chatbot App - PaLM</h1>
      <form class="chat-form" id="chat-form">
        <input
          type="text"
          id="user-input"
          name="user_input"
          placeholder="Your query..."
        />
        <button type="submit" id="submitBtn">Send</button>
      </form>
      <div id="result"></div>
    </div>
    <script>
      window.onload = function () {
        document
          .getElementById("chat-form")
          .addEventListener("submit", function (event) {
            event.preventDefault();
            let userInput = document.getElementById("user-input").value;
            fetch(`/palm2?user_input=${encodeURIComponent(userInput)}`)
              .then((response) => response.json())
              .then(
                (data) =>
                  (document.getElementById("result").innerHTML = data.content)
              )
              .catch((error) =>
                console.error("Error fetching PaLM response:", error)
              );
          });
      };
    </script>
  </body>
</html>
```

### Step 5: Dockerizing the Application

#### Dockerfile

To containerize the application, use the following Dockerfile:

```Dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

#### Build and Run the Docker Container

```bash
docker build -t chatbot-app .
docker run -p 8080:8080 chatbot-app
```

### Step 6: Deploy to Cloud Run

Once your container is ready, deploy the app to Google Cloud Run:

```bash
gcloud builds submit --tag gcr.io/your-project-id/chatbot-app
gcloud run deploy chatbot-app --image gcr.io/your-project-id/chatbot-app --platform managed
```

## How to Test the Chatbot

To test locally, simply run the Flask application:

```bash
python app.py
```

Then, visit `http://localhost:8080/` in your browser. You can now enter a query and get a response from the chatbot.

## Conclusion

This project demonstrates the integration of Google’s PaLM API with a Flask web application, deployed on Cloud Run. It shows how to create scalable, intelligent chatbots and manage them in a cloud environment. By using Docker for containerization and Cloud Run for deployment, this solution provides high availability and auto-scaling capabilities, essential for real-world AI applications.
