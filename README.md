CrewAI Full-Stack Generator
This is a production-ready, full-stack application that leverages the Google Gemini API to generate complete frontend and backend code based on user prompts. It simulates a CrewAI-style workflow with distinct AI agents to deliver a complete codebase.

Tech Stack
Frontend: React, Vite, Tailwind CSS

Backend: Node.js, Express

API: Google Gemini (@google/generative-ai)

Getting Started
Follow these instructions to get the project running locally for development and testing.

Prerequisites

Node.js (v18 or later)

npm or yarn

A Google Gemini API Key. You can obtain one from Google AI Studio.

1. Clone the Repository

Clone this repository to your local machine:

git clone <repository-url>
cd <repository-directory>

2. Project Setup

The project is structured as a monorepo with client and server directories.

A. Set up the Server:

Navigate to the server directory:

cd server

Install server dependencies:

npm install

Create a .env file in the server directory and add your Gemini API key:

GEMINI_API_KEY=your_google_gemini_api_key_here

B. Set up the Client:

Navigate to the client directory from the root:

cd ../client

Install client dependencies:

npm install

3. Running the Application

You need to run both the server and the client in separate terminals.

Start the Server: In the server directory, run:

node server.js

The server will start on http://localhost:3001.

Start the Client: In the client directory, run:

npm run dev

The client development server will start, typically on http://localhost:5173. Open this URL in your browser.

4. How to Use

Enter a description of the application you want to build in the text area.

Click "Build Application".

The AI agents will process the request, and the generated code for the frontend and backend will be displayed.

Deployment
To deploy this application, you would typically:

Build the Client: In the client directory, run npm run build. This creates a dist folder with static assets.

Deploy the Server: Deploy the server directory to a platform like Heroku, Vercel, or AWS.

Ensure the GEMINI_API_KEY environment variable is set in your deployment environment.

Configure the server to serve the static files from the client's dist folder.

CORS Configuration: For production, update the corsOptions in server/server.js to allow requests only from your frontend's domain.

