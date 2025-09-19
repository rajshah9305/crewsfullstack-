import React, { useState } from 'react';

// --- SVG Icons for Status ---
const CheckCircleIcon = () => ( <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg> );
const WorkingSpinnerIcon = () => ( <svg className="animate-spin h-6 w-6 text-indigo-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle><path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg> );
const PendingIcon = () => ( <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 12h.01M12 12h.01M16 12h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg> );

// --- Helper Components ---
const CodeDisplay = ({ title, code }) => {
  const [copied, setCopied] = useState(false);
  const handleCopy = () => {
    if (!code) return;
    navigator.clipboard.writeText(code).then(() => {
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    }, (err) => {
      console.error('Failed to copy text: ', err);
    });
  };
  return (
    <div className="bg-white rounded-lg shadow-md h-full flex flex-col border border-gray-200">
      <div className="bg-gray-50 px-4 py-2 flex justify-between items-center border-b border-gray-200 rounded-t-lg">
        <h3 className="text-sm font-semibold text-gray-700">{title}</h3>
        <button onClick={handleCopy} className="bg-gray-200 hover:bg-indigo-500 hover:text-white text-gray-600 text-xs font-bold py-1 px-3 rounded-md transition-colors duration-300">{copied ? 'Copied!' : 'Copy Code'}</button>
      </div>
      <div className="p-4 overflow-auto flex-grow bg-gray-50/50 rounded-b-lg"><pre className="text-sm text-gray-800 whitespace-pre-wrap"><code>{code || `// ${title} code will appear here...`}</code></pre></div>
    </div>
  );
};

const AgentCard = ({ agent }) => {
  const statusStyles = {
    pending: { borderColor: 'border-gray-300', icon: <PendingIcon /> },
    working: { borderColor: 'border-indigo-500', icon: <WorkingSpinnerIcon /> },
    complete: { borderColor: 'border-green-500', icon: <CheckCircleIcon /> }
  };
  const { borderColor, icon } = statusStyles[agent.status] || statusStyles.pending;
  return (
    <div className={`bg-white rounded-lg shadow-lg p-4 flex flex-col border-t-4 ${borderColor} transition-all duration-500 transform hover:scale-105`}>
      <div className="flex items-center justify-between mb-3"><h3 className="font-bold text-gray-800">{agent.name}</h3>{icon}</div>
      <div className="bg-gray-100 rounded p-2 overflow-auto h-48 text-xs text-gray-600 border border-gray-200"><pre className="whitespace-pre-wrap font-mono">{agent.output || `Waiting for task...`}</pre></div>
    </div>
  );
};

const AgentWorkflowDisplay = ({ agents }) => (
  <div className="w-full">
    <h2 className="text-2xl font-bold text-center mb-6 text-slate-700">CrewAI Workflow Status</h2>
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      {agents.map(agent => <AgentCard key={agent.id} agent={agent} />)}
    </div>
  </div>
);

// --- Main App Component ---
const initialAgents = [
  { id: 'planner', name: 'Project Planner', status: 'pending', output: '' },
  { id: 'frontend', name: 'Frontend Developer', status: 'pending', output: '' },
  { id: 'backend', name: 'Backend Developer', status: 'pending', output: '' },
  { id: 'qa', name: 'QA Engineer', status: 'pending', output: '' },
];

export default function App() {
  const [prompt, setPrompt] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [agents, setAgents] = useState(initialAgents);
  const [showResults, setShowResults] = useState(false);

  const updateAgentStatus = (id, status, output) => {
    setAgents(prev => prev.map(agent =>
      agent.id === id ? { ...agent, status, output: output !== undefined ? output : agent.output } : agent
    ));
  };
  
  const handleGenerateCode = async () => {
    if (!prompt.trim()) {
      setError('Please enter a description for the application you want to build.');
      return;
    }
    setIsLoading(true);
    setError(null);
    setShowResults(false);
    setAgents(initialAgents);

    // Real-time UI updates for agent workflow
    updateAgentStatus('planner', 'working');

    try {
      const response = await fetch('http://localhost:3001/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ prompt }),
      });

      if (!response.ok) {
        const errData = await response.json();
        throw new Error(errData.error || 'An unknown error occurred.');
      }
      
      const result = await response.json();
      
      updateAgentStatus('planner', 'complete', result.plannerOutput);
      
      updateAgentStatus('frontend', 'working', '// Generating React + Tailwind CSS code...');
      updateAgentStatus('backend', 'working', '// Generating Node.js + Express code...');
      
      // Simulate a small delay to show progress
      await new Promise(res => setTimeout(res, 1000));
      
      updateAgentStatus('frontend', 'complete', result.frontendCode);
      updateAgentStatus('backend', 'complete', result.backendCode);
      
      updateAgentStatus('qa', 'working', '// Reviewing code for quality...');
      await new Promise(res => setTimeout(res, 1000));
      updateAgentStatus('qa', 'complete', result.qaOutput);

      setShowResults(true);

    } catch (err) {
      console.error("API Call Error:", err);
      // Check for a network error and provide a more helpful message.
      if (err instanceof TypeError && (err.message.includes('Failed to fetch') || err.message.includes('Load failed'))) {
        const troubleshootingMessage = `Network Error: Could not connect to the backend server.

Please check the following in your terminal:
1.  Is the backend server running? 
    - Navigate to the 'server' directory.
    - Run the command: node server.js

2.  Did the server start correctly on http://localhost:3001?
    - Look for the "Server listening..." message.
    - Check for any errors, like a missing GEMINI_API_KEY.`;
        setError(troubleshootingMessage);
      } else {
        setError(err.message);
      }
      updateAgentStatus('planner', 'pending'); // Reset on error
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="bg-gray-50 text-slate-800 min-h-screen font-sans flex flex-col p-4 sm:p-6 lg:p-8">
      <header className="text-center mb-10">
        <h1 className="text-4xl sm:text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-sky-500 to-indigo-600">CrewAI Full-Stack Generator</h1>
        <p className="mt-2 text-lg text-slate-600">Describe your application, and let our AI agent crew build it for you.</p>
      </header>

      <main className="flex-grow flex flex-col items-center">
        <div className="w-full max-w-3xl mx-auto mb-8">
          <div className="bg-white rounded-lg shadow-md p-4 border border-gray-200">
            <textarea
              value={prompt}
              onChange={(e) => setPrompt(e.target.value)}
              placeholder="e.g., A simple polling app with real-time vote updates."
              className="w-full h-24 p-3 bg-gray-50 text-slate-800 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition"
              disabled={isLoading}
            />
            <button onClick={handleGenerateCode} disabled={isLoading} className="mt-4 w-full bg-gradient-to-r from-sky-500 to-indigo-600 hover:from-sky-600 hover:to-indigo-700 text-white font-bold py-3 px-4 rounded-lg transition-all duration-300 ease-in-out disabled:opacity-50 disabled:cursor-not-allowed transform hover:scale-105 shadow-lg shadow-indigo-500/20">
              {isLoading ? 'Agents are Working...' : 'Build Application'}
            </button>
          </div>
          {error && (
            <div className="mt-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-md">
              <pre className="whitespace-pre-wrap font-sans text-sm">{error}</pre>
            </div>
          )}
        </div>

        <div className="flex-grow flex flex-col w-full max-w-7xl">
          {(isLoading || showResults) && <AgentWorkflowDisplay agents={agents} />}
          {showResults && !isLoading && (
            <div className="flex-grow grid grid-cols-1 lg:grid-cols-2 gap-6 mt-8">
              <CodeDisplay title="frontend/src/App.jsx" code={agents.find(a => a.id === 'frontend').output} />
              <CodeDisplay title="backend/server.js" code={agents.find(a => a.id === 'backend').output} />
            </div>
          )}
        </div>
      </main>

      <footer className="text-center mt-8 text-gray-500 text-sm"><p>Powered by Google Gemini & CrewAI Principles</p></footer>
    </div>
  );
}

