import express from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import { GoogleGenerativeAI } from '@google/generative-ai';

// Load environment variables
dotenv.config();

// Initialize Express App
const app = express();
const port = 3001;

// Initialize Google Gemini Client
if (!process.env.GEMINI_API_KEY) {
  throw new Error("GEMINI_API_KEY is not defined in the environment variables.");
}
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

// Middleware
app.use(express.json());

// For development, allow requests from the Vite client
const corsOptions = {
  origin: 'http://localhost:5173', // Adjust if your client runs on a different port
};
app.use(cors(corsOptions));


// API Route for code generation
app.post('/api/generate', async (req, res) => {
  const { prompt } = req.body;

  if (!prompt) {
    return res.status(400).json({ error: 'Prompt is required.' });
  }

  try {
    const model = genAI.getGenerativeModel({ 
        model: "gemini-1.5-flash",
        generationConfig: { responseMimeType: "application/json" }
    });
    
    const systemInstruction = `You are an expert full-stack software development team. Your goal is to generate a complete, production-ready, single-file codebase for a full-stack application based on the user's request.

    Your Process:
    1.  **Project Plan:** Create a concise plan outlining the application's features.
    2.  **Frontend Code:** Write the code for the frontend using React and Tailwind CSS in a single, complete JSX file.
    3.  **Backend Code:** Write the code for the backend using Node.js with the Express framework in a single file. Create a simple in-memory data store if needed.
    4.  **QA Review:** Provide a brief summary confirming the code has been reviewed for quality.

    You MUST provide the final output in a single JSON object with four keys: "plannerOutput", "frontendCode", "backendCode", and "qaOutput".
    The code in "frontendCode" and "backendCode" must be complete and runnable as single files.`;

    const fullPrompt = `Generate a full-stack application for the following requirement: "${prompt}"`;
    
    const result = await model.generateContent([systemInstruction, fullPrompt]);
    const response = result.response;
    const text = response.text();
    
    // The response text is a JSON string, so we send it directly
    res.setHeader('Content-Type', 'application/json');
    res.send(text);

  } catch (error) {
    console.error('Error generating code with Gemini API:', error);
    res.status(500).json({ error: 'Failed to generate code. Please check the server logs.' });
  }
});

// Start the server
app.listen(port, () => {
  console.log(`Server listening at http://localhost:${port}`);
});
