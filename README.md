# proj-mag


```bash
npx create-next-app@latest docforge
cd docforge
npm install lucide-react framer-motion clsx tailwind-merge react-dropzone
```

Update src/app/globals.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --background-start: #0B0E14;
  --background-end: #150F1D;
  --accent-purple: #6B46C1;
}

body {
  background: radial-gradient(circle at top left, var(--background-start), var(--background-end));
  background-attachment: fixed;
  color: white;
  min-height: 100vh;
}

/* Custom scrollbar for the terminal/logs view */
::-webkit-scrollbar {
  width: 8px;
}
::-webkit-scrollbar-track {
  background: rgba(0,0,0,0.1);
}
::-webkit-scrollbar-thumb {
  background: rgba(255,255,255,0.1);
  border-radius: 4px;
}
```

Create src/components/ProjectWorkspace.tsx
```tsx
'use client'
import { useState } from 'react';
import Sidebar from './Sidebar';
import Stage1Upload from './stages/Stage1Upload';
import Stage2Knowledge from './stages/Stage2Knowledge';
// ... import other stages

export default function ProjectWorkspace() {
  const [currentStage, setCurrentStage] = useState(1);

  // This function will eventually call your Python backend endpoints
  const handleNextStage = async (stageData: any) => {
    // Example: if (currentStage === 1) await fetch('/api/process-docs', ...)
    setCurrentStage(prev => prev + 1);
  };

  return (
    <div className="flex h-screen w-full">
      {/* Left Sidebar tracking progress */}
      <div className="w-64 border-r border-white/10 bg-black/20 backdrop-blur-md">
        <Sidebar currentStage={currentStage} />
      </div>

      {/* Main Content Area */}
      <div className="flex-1 p-10 overflow-y-auto relative">
        <div className="max-w-4xl mx-auto">
          {currentStage === 1 && <Stage1Upload onComplete={handleNextStage} />}
          {currentStage === 2 && <Stage2Knowledge onComplete={handleNextStage} />}
          {/* Add your other stages here based on the plan */}
          
          {currentStage > 2 && (
             <div className="text-center mt-20 text-slate-400">
               <h2 className="text-2xl text-white mb-4">Context Synthesis</h2>
               <p>Connecting to backend API...</p>
             </div>
          )}
        </div>
      </div>
    </div>
  );
}
```

Create src/components/Sidebar.tsx

```tsx
import { CheckCircle2, Circle, Dot } from 'lucide-react';
import clsx from 'clsx';

const STAGES = [
  { id: 1, title: 'Document Upload', subtitle: 'Initialize raw knowledge' },
  { id: 2, title: 'Knowledge Selection', subtitle: 'Filter and refine inputs' },
  { id: 3, title: 'Context Configuration', subtitle: 'Set references' },
  { id: 4, title: 'Documentation', subtitle: 'Generate tech specs' },
  { id: 5, title: 'Development Plan', subtitle: 'Review and execute' },
];

export default function Sidebar({ currentStage }: { currentStage: number }) {
  return (
    <div className="p-6">
      <div className="flex items-center gap-2 mb-10">
        <div className="w-8 h-8 rounded-lg bg-indigo-600 flex items-center justify-center font-bold">
          S
        </div>
        <span className="font-semibold text-lg tracking-wide">Synapse AI</span>
      </div>

      <div className="text-xs font-semibold text-slate-500 mb-4 tracking-wider">
        CURRENT PROJECT
      </div>
      <div className="text-sm font-medium mb-8 pl-2">DocForge Alpha</div>

      <div className="space-y-6">
        {STAGES.map((stage) => {
          const isActive = currentStage === stage.id;
          const isPast = currentStage > stage.id;

          return (
            <div key={stage.id} className="flex gap-4">
              <div className="relative flex flex-col items-center">
                {isPast ? (
                  <CheckCircle2 className="w-5 h-5 text-indigo-400" />
                ) : isActive ? (
                  <Circle className="w-5 h-5 text-indigo-500 fill-indigo-500/20" />
                ) : (
                  <Circle className="w-5 h-5 text-slate-700" />
                )}
                {/* Connecting line */}
                {stage.id !== STAGES.length && (
                  <div className={clsx(
                    "w-[1px] h-full mt-2",
                    isPast ? "bg-indigo-500/50" : "bg-slate-800"
                  )} />
                )}
              </div>
              <div className={clsx("pb-6", isActive ? "opacity-100" : "opacity-40")}>
                <div className={clsx("text-sm font-medium", isActive && "text-indigo-300")}>
                  {stage.title}
                </div>
                <div className="text-xs text-slate-400 mt-1">{stage.subtitle}</div>
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

Create src/components/stages/Stage1Upload.tsx

```tsx
import { useCallback, useState } from 'react';
import { useDropzone } from 'react-dropzone';
import { UploadCloud, FileText, CheckCircle } from 'lucide-react';

export default function Stage1Upload({ onComplete }: { onComplete: (data: any) => void }) {
  const [file, setFile] = useState<File | null>(null);
  const [isProcessing, setIsProcessing] = useState(false);

  const onDrop = useCallback((acceptedFiles: File[]) => {
    if (acceptedFiles.length > 0) {
      setFile(acceptedFiles[0]);
    }
  }, []);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({ onDrop });

  const handleProcess = () => {
    setIsProcessing(true);
    // Simulate backend processing time before moving to next stage
    setTimeout(() => {
      setIsProcessing(false);
      onComplete({ fileName: file?.name });
    }, 2000);
  };

  return (
    <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
      <h1 className="text-3xl font-bold mb-2">Document Upload</h1>
      <p className="text-slate-400 mb-8">Upload raw knowledge base materials to begin initialization.</p>

      <div className="bg-white/[0.03] border border-white/10 rounded-2xl p-6 backdrop-blur-sm">
        <h3 className="text-sm font-semibold mb-4">Select Files</h3>
        
        <div 
          {...getRootProps()} 
          className={`border-2 border-dashed rounded-xl p-10 flex flex-col items-center justify-center cursor-pointer transition-colors
            ${isDragActive ? 'border-indigo-500 bg-indigo-500/10' : 'border-slate-700 hover:border-slate-500 hover:bg-white/[0.02]'}`}
        >
          <input {...getInputProps()} />
          <div className="w-12 h-12 rounded-full bg-indigo-500/20 flex items-center justify-center mb-4">
            <UploadCloud className="w-6 h-6 text-indigo-400" />
          </div>
          <p className="font-medium text-slate-200">Click to upload documents</p>
          <p className="text-sm text-slate-500 mt-1">Supports PDF, DOCX, TXT, CSV</p>
        </div>

        {file && (
          <div className="mt-6">
            <h3 className="text-xs font-semibold text-slate-500 mb-2 uppercase">Selected Files</h3>
            <div className="flex items-center justify-between bg-black/40 border border-white/5 rounded-lg p-3">
              <div className="flex items-center gap-3">
                <FileText className="w-5 h-5 text-blue-400" />
                <span className="text-sm">{file.name}</span>
              </div>
              <span className="text-xs text-slate-500">
                {(file.size / 1024 / 1024).toFixed(2)} MB
              </span>
            </div>

            <div className="mt-6 flex justify-end">
              <button 
                onClick={handleProcess}
                disabled={isProcessing}
                className="bg-indigo-600 hover:bg-indigo-500 text-white px-6 py-2.5 rounded-lg font-medium text-sm transition-all flex items-center gap-2"
              >
                {isProcessing ? 'Processing...' : 'Process Documents'}
              </button>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```


Create a new file at src/components/stages/Stage2Knowledge.tsx

```tsx
import { useState } from 'react';
import { Database, UploadCloud, FileText, CheckCircle2, Search, ArrowRight, FileType2 } from 'lucide-react';
import clsx from 'clsx';

// Mock data representing backend RAG/FAISS results
const NAS_RESULTS = [
  { id: 'nas-1', name: 'auth-system-requirements.pdf', relevancy: 96, size: '1.2 MB' },
  { id: 'nas-2', name: 'legacy-db-schema-v3.pdf', relevancy: 82, size: '3.4 MB' },
];

export default function Stage2Knowledge({ onComplete }: { onComplete: (data: any) => void }) {
  const [activeMode, setActiveMode] = useState<'upload' | 'nas'>('nas');
  
  // NAS State
  const [isSearching, setIsSearching] = useState(false);
  const [selectedNasDoc, setSelectedNasDoc] = useState<string | null>(null);

  // Upload State
  const [uploadedDoc, setUploadedDoc] = useState<File | null>(null);
  const [isConverting, setIsConverting] = useState(false);

  // Handlers
  const handleNasSearch = () => {
    setIsSearching(true);
    setTimeout(() => setIsSearching(false), 1500); // Simulate FAISS search delay
  };

  const handleFileUpload = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files && e.target.files[0]) {
      setUploadedDoc(e.target.files[0]);
      setIsConverting(true);
      // Simulate Word to PDF conversion time
      setTimeout(() => setIsConverting(false), 2000);
    }
  };

  const handleContinue = () => {
    const selectedData = activeMode === 'nas' 
      ? NAS_RESULTS.find(d => d.id === selectedNasDoc)
      : { name: uploadedDoc?.name, converted: true };
      
    onComplete({ stage2Data: selectedData });
  };

  const canContinue = (activeMode === 'nas' && selectedNasDoc) || (activeMode === 'upload' && uploadedDoc && !isConverting);

  return (
    <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
      <h1 className="text-3xl font-bold mb-2">Knowledge Selection</h1>
      <p className="text-slate-400 mb-8">Select the primary requirement document to guide the development plan.</p>

      {/* Mode Selection Tabs */}
      <div className="flex gap-4 mb-8">
        <button
          onClick={() => setActiveMode('nas')}
          className={clsx(
            "flex-1 flex items-center justify-center gap-3 py-4 rounded-xl border transition-all",
            activeMode === 'nas' 
              ? "bg-indigo-500/20 border-indigo-500 text-indigo-300" 
              : "bg-white/[0.02] border-white/5 text-slate-400 hover:bg-white/[0.05]"
          )}
        >
          <Database className="w-5 h-5" />
          <span className="font-medium">Fetch from NAS</span>
        </button>
        
        <button
          onClick={() => setActiveMode('upload')}
          className={clsx(
            "flex-1 flex items-center justify-center gap-3 py-4 rounded-xl border transition-all",
            activeMode === 'upload' 
              ? "bg-indigo-500/20 border-indigo-500 text-indigo-300" 
              : "bg-white/[0.02] border-white/5 text-slate-400 hover:bg-white/[0.05]"
          )}
        >
          <UploadCloud className="w-5 h-5" />
          <span className="font-medium">Upload Document</span>
        </button>
      </div>

      {/* Main Content Area */}
      <div className="bg-white/[0.03] border border-white/10 rounded-2xl p-6 backdrop-blur-sm min-h-[320px]">
        
        {/* --- NAS MODE --- */}
        {activeMode === 'nas' && (
          <div className="animate-in fade-in duration-300">
            <div className="flex gap-3 mb-6">
              <div className="relative flex-1">
                <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-500" />
                <input 
                  type="text" 
                  placeholder="Query NAS repository..." 
                  className="w-full bg-black/40 border border-white/10 rounded-lg py-2.5 pl-10 pr-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200"
                />
              </div>
              <button 
                onClick={handleNasSearch}
                className="bg-white/10 hover:bg-white/20 px-6 rounded-lg text-sm font-medium transition-colors"
              >
                Search
              </button>
            </div>

            {isSearching ? (
              <div className="flex flex-col items-center justify-center py-12 text-slate-500">
                <div className="w-6 h-6 border-2 border-indigo-500 border-t-transparent rounded-full animate-spin mb-3"></div>
                <p className="text-sm">Running FAISS vector search...</p>
              </div>
            ) : (
              <div className="space-y-3">
                <p className="text-xs font-semibold text-slate-500 uppercase mb-3">Top Relevant Results</p>
                {NAS_RESULTS.map((doc) => (
                  <div 
                    key={doc.id}
                    onClick={() => setSelectedNasDoc(doc.id)}
                    className={clsx(
                      "flex items-center justify-between p-4 rounded-xl border cursor-pointer transition-all",
                      selectedNasDoc === doc.id 
                        ? "bg-indigo-500/10 border-indigo-500/50" 
                        : "bg-black/20 border-white/5 hover:bg-white/[0.04]"
                    )}
                  >
                    <div className="flex items-center gap-4">
                      <div className={clsx(
                        "w-10 h-10 rounded-lg flex items-center justify-center",
                        selectedNasDoc === doc.id ? "bg-indigo-500/20" : "bg-white/5"
                      )}>
                        <FileText className={clsx("w-5 h-5", selectedNasDoc === doc.id ? "text-indigo-400" : "text-slate-400")} />
                      </div>
                      <div>
                        <p className="text-sm font-medium text-slate-200">{doc.name}</p>
                        <p className="text-xs text-slate-500">{doc.size} • Vector Match</p>
                      </div>
                    </div>
                    
                    <div className="flex items-center gap-4">
                      <div className="flex flex-col items-end">
                        <span className="text-xs text-slate-400 mb-1">Relevancy</span>
                        <div className="flex items-center gap-2">
                          <div className="w-16 h-1.5 bg-black/50 rounded-full overflow-hidden">
                            <div 
                              className="h-full bg-emerald-500 rounded-full" 
                              style={{ width: `${doc.relevancy}%` }}
                            />
                          </div>
                          <span className="text-xs font-medium text-emerald-400">{doc.relevancy}%</span>
                        </div>
                      </div>
                      {selectedNasDoc === doc.id && <CheckCircle2 className="w-5 h-5 text-indigo-500" />}
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {/* --- UPLOAD MODE --- */}
        {activeMode === 'upload' && (
          <div className="animate-in fade-in duration-300">
            {!uploadedDoc ? (
              <div className="border-2 border-dashed border-slate-700 hover:border-slate-500 bg-black/20 rounded-xl p-12 flex flex-col items-center justify-center text-center relative transition-colors">
                <input 
                  type="file" 
                  accept=".docx,.doc,.pdf"
                  onChange={handleFileUpload}
                  className="absolute inset-0 w-full h-full opacity-0 cursor-pointer" 
                />
                <div className="w-12 h-12 rounded-full bg-indigo-500/20 flex items-center justify-center mb-4">
                  <FileType2 className="w-6 h-6 text-indigo-400" />
                </div>
                <p className="font-medium text-slate-200 mb-1">Upload requirement document</p>
                <p className="text-sm text-slate-500">Word (.docx) files will be auto-converted to PDF for processing.</p>
              </div>
            ) : (
              <div className="bg-black/20 border border-white/5 rounded-xl p-6">
                <div className="flex items-center justify-between mb-6">
                  <div className="flex items-center gap-4">
                    <div className="w-12 h-12 rounded-lg bg-indigo-500/20 flex items-center justify-center">
                      <FileText className="w-6 h-6 text-indigo-400" />
                    </div>
                    <div>
                      <p className="text-sm font-medium text-slate-200">{uploadedDoc.name}</p>
                      <p className="text-xs text-slate-500">{(uploadedDoc.size / 1024 / 1024).toFixed(2)} MB</p>
                    </div>
                  </div>
                  <button 
                    onClick={() => setUploadedDoc(null)}
                    className="text-xs text-slate-500 hover:text-white transition-colors"
                  >
                    Remove
                  </button>
                </div>

                <div className="bg-black/40 rounded-lg p-4 flex items-center justify-between">
                  <span className="text-sm text-slate-400">PDF Conversion Status</span>
                  {isConverting ? (
                    <div className="flex items-center gap-2 text-indigo-400 text-sm font-medium">
                      <div className="w-4 h-4 border-2 border-indigo-400 border-t-transparent rounded-full animate-spin"></div>
                      Converting...
                    </div>
                  ) : (
                    <div className="flex items-center gap-2 text-emerald-400 text-sm font-medium">
                      <CheckCircle2 className="w-4 h-4" />
                      Ready for processing
                    </div>
                  )}
                </div>
              </div>
            )}
          </div>
        )}
      </div>

      {/* Stage Action Footer */}
      <div className="mt-8 flex justify-end">
        <button 
          onClick={handleContinue}
          disabled={!canContinue}
          className={clsx(
            "px-6 py-2.5 rounded-lg font-medium text-sm transition-all flex items-center gap-2",
            canContinue 
              ? "bg-indigo-600 hover:bg-indigo-500 text-white shadow-lg shadow-indigo-500/25" 
              : "bg-white/5 text-slate-500 cursor-not-allowed"
          )}
        >
          Confirm Knowledge Source
          <ArrowRight className="w-4 h-4" />
        </button>
      </div>
    </div>
  );
}
```


Create a new file at src/components/stages/Stage3Context.tsx

```tsx
import { useState, useEffect } from 'react';
import { Link, Globe, Search, ArrowRight, CheckCircle2, AlertCircle, TerminalSquare } from 'lucide-react';
import clsx from 'clsx';

const MOCK_SUGGESTIONS = [
  'https://github.com/your-org/legacy-auth-service',
  'https://github.com/your-org/docforge-frontend',
  'https://confluence.internal.com/spaces/engineering/arch-v2',
  'https://gitlab.com/core-infra/database-schemas'
];

export default function Stage3Context({ onComplete }: { onComplete: (data: any) => void }) {
  const [activeMode, setActiveMode] = useState<'provide' | 'search'>('provide');
  
  // Provide URL State
  const [manualUrl, setManualUrl] = useState('');
  const [showSuggestions, setShowSuggestions] = useState(false);
  
  // Search State
  const [searchQuery, setSearchQuery] = useState('');
  const [isSearching, setIsSearching] = useState(false);
  const [searchResult, setSearchResult] = useState<{ url: string, success: boolean } | null>(null);

  const filteredSuggestions = MOCK_SUGGESTIONS.filter(s => 
    s.toLowerCase().includes(manualUrl.toLowerCase()) && manualUrl.length > 0
  );

  const handleAutoSearch = () => {
    if (!searchQuery) return;
    setIsSearching(true);
    setSearchResult(null);
    
    // Simulate backend search with 60% success rate as requested
    setTimeout(() => {
      setIsSearching(false);
      const isSuccess = Math.random() <= 0.60;
      setSearchResult({
        success: isSuccess,
        url: isSuccess 
          ? `https://github.com/auto-resolved/${searchQuery.replace(/\s+/g, '-').toLowerCase()}`
          : ''
      });
    }, 2000);
  };

  const handleContinue = () => {
    const finalUrl = activeMode === 'provide' ? manualUrl : searchResult?.url;
    onComplete({ contextUrl: finalUrl });
  };

  const canContinue = 
    (activeMode === 'provide' && manualUrl.length > 5) || 
    (activeMode === 'search' && searchResult?.success);

  return (
    <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
      <h1 className="text-3xl font-bold mb-2">Context Configuration</h1>
      <p className="text-slate-400 mb-8">Establish the primary reference repository or documentation URL.</p>

      {/* Mode Selection */}
      <div className="flex gap-4 mb-8">
        <button
          onClick={() => { setActiveMode('provide'); setSearchResult(null); }}
          className={clsx(
            "flex-1 flex items-center justify-center gap-3 py-4 rounded-xl border transition-all",
            activeMode === 'provide' 
              ? "bg-indigo-500/20 border-indigo-500 text-indigo-300" 
              : "bg-white/[0.02] border-white/5 text-slate-400 hover:bg-white/[0.05]"
          )}
        >
          <Link className="w-5 h-5" />
          <span className="font-medium">Provide a URL</span>
        </button>
        <button
          onClick={() => setActiveMode('search')}
          className={clsx(
            "flex-1 flex items-center justify-center gap-3 py-4 rounded-xl border transition-all",
            activeMode === 'search' 
              ? "bg-indigo-500/20 border-indigo-500 text-indigo-300" 
              : "bg-white/[0.02] border-white/5 text-slate-400 hover:bg-white/[0.05]"
          )}
        >
          <Globe className="w-5 h-5" />
          <span className="font-medium">Auto-Search URL</span>
        </button>
      </div>

      <div className="bg-white/[0.03] border border-white/10 rounded-2xl p-6 backdrop-blur-sm min-h-[250px] relative">
        
        {/* --- PROVIDE MODE --- */}
        {activeMode === 'provide' && (
          <div className="animate-in fade-in duration-300 relative">
            <label className="text-sm font-medium text-slate-300 mb-2 block">Reference URL</label>
            <div className="relative">
              <TerminalSquare className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-slate-500" />
              <input 
                type="text" 
                value={manualUrl}
                onChange={(e) => {
                  setManualUrl(e.target.value);
                  setShowSuggestions(true);
                }}
                onFocus={() => setShowSuggestions(true)}
                onBlur={() => setTimeout(() => setShowSuggestions(false), 200)}
                placeholder="https://github.com/..." 
                className="w-full bg-black/40 border border-white/10 rounded-xl py-3 pl-11 pr-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200 transition-colors"
              />
            </div>
            
            {/* Live Autocomplete Dropdown */}
            {showSuggestions && filteredSuggestions.length > 0 && (
              <div className="absolute top-full left-0 w-full mt-2 bg-slate-900 border border-white/10 rounded-xl overflow-hidden z-10 shadow-xl">
                {filteredSuggestions.map((suggestion, idx) => (
                  <div 
                    key={idx}
                    onClick={() => {
                      setManualUrl(suggestion);
                      setShowSuggestions(false);
                    }}
                    className="px-4 py-3 hover:bg-indigo-500/20 cursor-pointer text-sm text-slate-300 flex items-center gap-3 border-b border-white/5 last:border-0"
                  >
                    <Link className="w-4 h-4 text-indigo-400" />
                    {suggestion}
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {/* --- SEARCH MODE --- */}
        {activeMode === 'search' && (
          <div className="animate-in fade-in duration-300">
            <label className="text-sm font-medium text-slate-300 mb-2 block">Project Context Query</label>
            <div className="flex gap-3">
              <input 
                type="text" 
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
                placeholder="e.g., 'Core Authentication Service'" 
                className="flex-1 bg-black/40 border border-white/10 rounded-xl py-3 px-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200"
              />
              <button 
                onClick={handleAutoSearch}
                disabled={isSearching || !searchQuery}
                className="bg-indigo-600 hover:bg-indigo-500 disabled:bg-slate-800 disabled:text-slate-500 text-white px-6 rounded-xl text-sm font-medium transition-colors flex items-center gap-2"
              >
                {isSearching ? <div className="w-4 h-4 border-2 border-white/50 border-t-transparent rounded-full animate-spin" /> : <Search className="w-4 h-4" />}
                Search
              </button>
            </div>

            {/* Search Results */}
            {searchResult && (
              <div className={clsx(
                "mt-6 p-4 rounded-xl border animate-in slide-in-from-bottom-2",
                searchResult.success ? "bg-emerald-500/10 border-emerald-500/20" : "bg-rose-500/10 border-rose-500/20"
              )}>
                {searchResult.success ? (
                  <div className="flex items-start gap-3">
                    <CheckCircle2 className="w-5 h-5 text-emerald-400 mt-0.5" />
                    <div>
                      <p className="text-sm font-medium text-emerald-300">Context URL Found</p>
                      <a href={searchResult.url} className="text-sm text-slate-300 hover:text-white hover:underline mt-1 break-all block">
                        {searchResult.url}
                      </a>
                    </div>
                  </div>
                ) : (
                  <div className="flex items-start gap-3">
                    <AlertCircle className="w-5 h-5 text-rose-400 mt-0.5" />
                    <div>
                      <p className="text-sm font-medium text-rose-300">Search Failed</p>
                      <p className="text-sm text-slate-400 mt-1">Could not automatically resolve a URL for this query. Please switch to "Provide a URL" and enter it manually.</p>
                    </div>
                  </div>
                )}
              </div>
            )}
          </div>
        )}
      </div>

      <div className="mt-8 flex justify-end">
        <button 
          onClick={handleContinue}
          disabled={!canContinue}
          className={clsx(
            "px-6 py-2.5 rounded-lg font-medium text-sm transition-all flex items-center gap-2",
            canContinue 
              ? "bg-indigo-600 hover:bg-indigo-500 text-white shadow-lg shadow-indigo-500/25" 
              : "bg-white/5 text-slate-500 cursor-not-allowed"
          )}
        >
          Initialize Synthesis
          <ArrowRight className="w-4 h-4" />
        </button>
      </div>
    </div>
  );
}
```

Create a new file at src/components/stages/ContextSynthesis.tsx

```tsx
import { useEffect, useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { Terminal, Check, Loader2 } from 'lucide-react';

const SYNTHESIS_STEPS = [
  "Initializing isolated workspace sandbox...",
  "Parsing and indexing requirement documents...",
  "Extracting core architectural entities...",
  "Vectorizing contextual codebase references...",
  "Querying FAISS knowledge base for patterns...",
  "Resolving cross-repository dependencies...",
  "Formulating proposed system architecture...",
  "Generating foundational data models...",
  "Planning API route handlers and services...",
  "Drafting initial test coverage specifications...",
  "Finalizing context blueprint for Phase 2."
];

export default function ContextSynthesis({ onComplete }: { onComplete: () => void }) {
  const [currentStep, setCurrentStep] = useState(0);

  useEffect(() => {
    if (currentStep < SYNTHESIS_STEPS.length) {
      // Simulate processing time for each step (between 600ms and 1500ms)
      const delay = Math.floor(Math.random() * 900) + 600; 
      const timer = setTimeout(() => setCurrentStep(prev => prev + 1), delay);
      return () => clearTimeout(timer);
    } else {
      // All steps complete, wait 1.5s then trigger transition to Phase 2
      const timer = setTimeout(onComplete, 1500);
      return () => clearTimeout(timer);
    }
  }, [currentStep, onComplete]);

  return (
    <motion.div 
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
      className="fixed inset-0 z-50 bg-[#0B0E14]/95 backdrop-blur-xl flex flex-col items-center justify-center p-8"
    >
      <div className="max-w-3xl w-full">
        <div className="flex items-center gap-4 mb-12">
          <div className="w-12 h-12 rounded-xl bg-indigo-500/20 flex items-center justify-center border border-indigo-500/30">
            <Terminal className="w-6 h-6 text-indigo-400" />
          </div>
          <div>
            <h2 className="text-2xl font-bold text-white tracking-wide">Context Synthesis</h2>
            <p className="text-indigo-400 text-sm font-mono mt-1">Orchestrating backend analysis agents...</p>
          </div>
        </div>

        <div className="space-y-4 font-mono text-sm">
          <AnimatePresence>
            {SYNTHESIS_STEPS.map((step, index) => {
              if (index > currentStep) return null;

              const isComplete = index < currentStep;
              const isActive = index === currentStep;

              return (
                <motion.div
                  key={index}
                  initial={{ opacity: 0, x: -20 }}
                  animate={{ opacity: 1, x: 0 }}
                  className={`flex items-start gap-4 p-3 rounded-lg ${isActive ? 'bg-white/5 border border-white/10' : ''}`}
                >
                  <div className="mt-0.5">
                    {isComplete ? (
                      <Check className="w-4 h-4 text-emerald-500" />
                    ) : isActive ? (
                      <Loader2 className="w-4 h-4 text-indigo-400 animate-spin" />
                    ) : null}
                  </div>
                  
                  <span className={`${isComplete ? 'text-slate-500' : isActive ? 'text-indigo-300' : 'text-slate-600'}`}>
                    [{String(index + 1).padStart(2, '0')}/11]
                  </span>
                  
                  <span className={`${isComplete ? 'text-slate-300' : isActive ? 'text-white' : 'text-slate-600'}`}>
                    {step}
                  </span>
                </motion.div>
              );
            })}
          </AnimatePresence>
        </div>
      </div>
    </motion.div>
  );
}
```


Update ProjectWorkspace.tsx

```tsx
// Inside src/components/ProjectWorkspace.tsx
import { useState } from 'react';
import Stage3Context from './stages/Stage3Context';
import ContextSynthesis from './stages/ContextSynthesis';
// ... other imports

export default function ProjectWorkspace() {
  const [currentStage, setCurrentStage] = useState(1);
  const [isSynthesizing, setIsSynthesizing] = useState(false);

  const handleNextStage = async (stageData: any) => {
    if (currentStage === 3) {
      // Trigger the full screen animation before moving to stage 4
      setIsSynthesizing(true);
    } else {
      setCurrentStage(prev => prev + 1);
    }
  };

  const handleSynthesisComplete = () => {
    setIsSynthesizing(false);
    setCurrentStage(4); // Move to Phase 2 (Documentation)
  };

  return (
    <>
      <div className="flex h-screen w-full">
        {/* ... Sidebar and layout code ... */}
        
        <div className="flex-1 p-10 overflow-y-auto relative">
          <div className="max-w-4xl mx-auto">
            {/* ... previous stages ... */}
            {currentStage === 3 && <Stage3Context onComplete={handleNextStage} />}
            {currentStage === 4 && <div>Stage 4: Documentation (Coming Next)</div>}
          </div>
        </div>
      </div>

      {/* Render the full-screen overlay when triggered */}
      {isSynthesizing && <ContextSynthesis onComplete={handleSynthesisComplete} />}
    </>
  );
}
```

Create a new file at src/components/stages/Stage4Documentation.tsx

```tsx
import { useState } from 'react';
import { FileText, Download, RefreshCcw, ArrowRight, SkipForward, CheckCircle2, AlertCircle } from 'lucide-react';
import clsx from 'clsx';

// Mock generated documentation content
const MOCK_MARKDOWN = `# Technical Specification: DocForge Alpha

## 1. System Architecture
The system utilizes a microservices architecture pattern...

## 2. Data Models
- \`User\`: Handles authentication and profile data.
- \`Project\`: Stores workspace configurations and state.
- \`Document\`: Represents uploaded or NAS-fetched artifacts.

## 3. API Endpoints
- \`POST /api/v1/process-docs\`
- \`GET /api/v1/context-search\`
- \`POST /api/v1/generate-plan\`
`;

export default function Stage4Documentation({ onComplete }: { onComplete: () => void }) {
  const [status, setStatus] = useState<'idle' | 'generating' | 'success' | 'error'>('idle');
  
  const handleGenerate = () => {
    setStatus('generating');
    // Simulate API call to LLM for document generation
    setTimeout(() => {
      // 90% chance of success for demonstration
      if (Math.random() > 0.1) {
        setStatus('success');
      } else {
        setStatus('error');
      }
    }, 3000);
  };

  const handleDownload = () => {
    // Simple implementation to download the mock text as a .md file
    const element = document.createElement("a");
    const file = new Blob([MOCK_MARKDOWN], {type: 'text/markdown'});
    element.href = URL.createObjectURL(file);
    element.download = "tech_specs.md";
    document.body.appendChild(element);
    element.click();
    document.body.removeChild(element);
  };

  return (
    <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
      <div className="flex items-center justify-between mb-2">
        <h1 className="text-3xl font-bold">Documentation</h1>
        <button 
          onClick={onComplete}
          className="text-sm text-slate-400 hover:text-white flex items-center gap-2 transition-colors"
        >
          Skip this step
          <SkipForward className="w-4 h-4" />
        </button>
      </div>
      <p className="text-slate-400 mb-8">Generate comprehensive technical specifications based on the synthesized context.</p>

      <div className="bg-white/[0.03] border border-white/10 rounded-2xl p-6 backdrop-blur-sm min-h-[400px] flex flex-col">
        
        {/* IDLE STATE */}
        {status === 'idle' && (
          <div className="flex-1 flex flex-col items-center justify-center text-center">
            <div className="w-16 h-16 rounded-full bg-indigo-500/20 flex items-center justify-center mb-6">
              <FileText className="w-8 h-8 text-indigo-400" />
            </div>
            <h3 className="text-lg font-medium text-white mb-2">Ready to Generate</h3>
            <p className="text-sm text-slate-400 max-w-sm mb-8">
              The AI agents will draft architecture documents, data models, and API schemas based on your Phase 1 inputs.
            </p>
            <button 
              onClick={handleGenerate}
              className="bg-indigo-600 hover:bg-indigo-500 text-white px-8 py-3 rounded-xl font-medium transition-all shadow-lg shadow-indigo-500/25"
            >
              Generate Technical Specs
            </button>
          </div>
        )}

        {/* GENERATING STATE */}
        {status === 'generating' && (
          <div className="flex-1 flex flex-col items-center justify-center text-center">
            <div className="relative w-16 h-16 mb-6">
              <div className="absolute inset-0 border-4 border-indigo-500/20 rounded-full"></div>
              <div className="absolute inset-0 border-4 border-indigo-500 border-t-transparent rounded-full animate-spin"></div>
              <FileText className="absolute inset-0 m-auto w-6 h-6 text-indigo-400 animate-pulse" />
            </div>
            <h3 className="text-lg font-medium text-white mb-2">Drafting Documentation...</h3>
            <p className="text-sm text-slate-400">Please wait while the AI models format the specifications.</p>
          </div>
        )}

        {/* ERROR STATE */}
        {status === 'error' && (
          <div className="flex-1 flex flex-col items-center justify-center text-center">
            <div className="w-16 h-16 rounded-full bg-rose-500/20 flex items-center justify-center mb-6">
              <AlertCircle className="w-8 h-8 text-rose-400" />
            </div>
            <h3 className="text-lg font-medium text-white mb-2">Generation Failed</h3>
            <p className="text-sm text-slate-400 max-w-sm mb-8">
              There was a timeout communicating with the backend agents. Please try again.
            </p>
            <button 
              onClick={handleGenerate}
              className="bg-white/10 hover:bg-white/20 text-white px-8 py-3 rounded-xl font-medium transition-all flex items-center gap-2"
            >
              <RefreshCcw className="w-4 h-4" />
              Retry Generation
            </button>
          </div>
        )}

        {/* SUCCESS STATE */}
        {status === 'success' && (
          <div className="flex-1 flex flex-col animate-in fade-in duration-500">
            <div className="flex items-center justify-between mb-4">
              <div className="flex items-center gap-2 text-emerald-400">
                <CheckCircle2 className="w-5 h-5" />
                <span className="font-medium text-sm">Generation Complete</span>
              </div>
              <div className="flex gap-3">
                <button 
                  onClick={handleGenerate}
                  className="p-2 text-slate-400 hover:text-white bg-white/5 hover:bg-white/10 rounded-lg transition-colors tooltip-trigger"
                  title="Regenerate"
                >
                  <RefreshCcw className="w-4 h-4" />
                </button>
                <button 
                  onClick={handleDownload}
                  className="flex items-center gap-2 px-3 py-2 text-sm font-medium text-indigo-300 bg-indigo-500/10 hover:bg-indigo-500/20 border border-indigo-500/20 rounded-lg transition-colors"
                >
                  <Download className="w-4 h-4" />
                  Download .md
                </button>
              </div>
            </div>

            {/* Document Preview Window */}
            <div className="flex-1 bg-black/50 border border-white/5 rounded-xl p-6 overflow-y-auto font-mono text-sm text-slate-300 whitespace-pre-wrap">
              {MOCK_MARKDOWN}
            </div>
          </div>
        )}
      </div>

      {/* Stage Action Footer */}
      <div className="mt-8 flex justify-end">
        <button 
          onClick={onComplete}
          disabled={status === 'idle' || status === 'generating'}
          className={clsx(
            "px-6 py-2.5 rounded-lg font-medium text-sm transition-all flex items-center gap-2",
            (status === 'success' || status === 'error')
              ? "bg-indigo-600 hover:bg-indigo-500 text-white shadow-lg shadow-indigo-500/25" 
              : "bg-white/5 text-slate-500 cursor-not-allowed"
          )}
        >
          Continue to Development Plan
          <ArrowRight className="w-4 h-4" />
        </button>
      </div>
    </div>
  );
}
```


update  ProjectWorkspace.tsx

```tsx
// Inside ProjectWorkspace.tsx, add the import:
import Stage4Documentation from './stages/Stage4Documentation';

// In the render block:
{currentStage === 4 && <Stage4Documentation onComplete={() => setCurrentStage(5)} />}
{currentStage === 5 && <div>Stage 5: Development Plan (Coming Next)</div>}
```

Create a new file at src/components/stages/Stage5Development.tsx

```tsx
import { useState, useEffect, useRef } from 'react';
import { Terminal, Play, GitBranch, GitPullRequest, RotateCcw, CheckCircle2, FileCode2, Github } from 'lucide-react';
import clsx from 'clsx';

// Simulated terminal output sequence
const BUILD_LOGS = [
  "[SYSTEM] Initializing secure development sandbox...",
  "[GIT] Created new branch: feature/auto-generated-specs",
  "[AGENT] Analyzing Technical Specifications (tech_specs.md)...",
  "[AGENT] Generating src/lib/db/schema.ts...",
  "[INFO] Successfully wrote 124 lines to schema.ts",
  "[AGENT] Scaffolded API routes in src/app/api/v1/*",
  "[AGENT] Generating Next.js frontend components...",
  "[WARN] Missing explicit type for 'ProjectConfig', applying fallback interface.",
  "[INFO] Wrote Stage1Upload.tsx, Stage2Knowledge.tsx...",
  "[TEST] Running preliminary static analysis...",
  "[TEST] Static analysis passed. 0 critical vulnerabilities.",
  "[GIT] Staging 14 modified files...",
  "[GIT] Committing: 'feat: AI generated base orchestration flow'",
  "[GIT] Pushing to remote repository...",
  "[SYSTEM] Development cycle complete. Ready for review."
];

export default function Stage5Development({ onReset }: { onReset: () => void }) {
  const [status, setStatus] = useState<'plan' | 'developing' | 'complete'>('plan');
  const [logs, setLogs] = useState<string[]>([]);
  const terminalEndRef = useRef<HTMLDivElement>(null);

  // Auto-scroll terminal
  useEffect(() => {
    if (terminalEndRef.current) {
      terminalEndRef.current.scrollIntoView({ behavior: 'smooth' });
    }
  }, [logs]);

  // Simulate streaming terminal logs
  useEffect(() => {
    if (status === 'developing') {
      let currentIndex = 0;
      setLogs([]); // Clear logs on start
      
      const streamInterval = setInterval(() => {
        if (currentIndex < BUILD_LOGS.length) {
          setLogs(prev => [...prev, BUILD_LOGS[currentIndex]]);
          currentIndex++;
        } else {
          clearInterval(streamInterval);
          setTimeout(() => setStatus('complete'), 1000);
        }
      }, 800); // Add a new log every 800ms

      return () => clearInterval(streamInterval);
    }
  }, [status]);

  return (
    <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
      <h1 className="text-3xl font-bold mb-2">Development Execution</h1>
      <p className="text-slate-400 mb-8">Review the execution plan and trigger the autonomous development agents.</p>

      {/* PLAN VIEW */}
      {status === 'plan' && (
        <div className="bg-white/[0.03] border border-white/10 rounded-2xl p-6 backdrop-blur-sm animate-in fade-in">
          <h3 className="text-sm font-semibold text-slate-300 mb-4 uppercase tracking-wider">Proposed Development Plan</h3>
          
          <div className="grid grid-cols-2 gap-4 mb-8">
            <div className="bg-black/40 border border-white/5 rounded-xl p-4">
              <div className="flex items-center gap-3 mb-2">
                <FileCode2 className="w-5 h-5 text-indigo-400" />
                <span className="font-medium text-slate-200">Files to Generate</span>
              </div>
              <p className="text-2xl font-bold text-white">14</p>
            </div>
            <div className="bg-black/40 border border-white/5 rounded-xl p-4">
              <div className="flex items-center gap-3 mb-2">
                <GitBranch className="w-5 h-5 text-emerald-400" />
                <span className="font-medium text-slate-200">Target Branch</span>
              </div>
              <p className="text-lg font-mono text-white mt-1">feature/auto-generated-specs</p>
            </div>
          </div>

          <div className="flex justify-end">
            <button 
              onClick={() => setStatus('developing')}
              className="bg-indigo-600 hover:bg-indigo-500 text-white px-8 py-3 rounded-xl font-medium transition-all flex items-center gap-2 shadow-lg shadow-indigo-500/25"
            >
              <Play className="w-4 h-4 fill-current" />
              Start Development
            </button>
          </div>
        </div>
      )}

      {/* TERMINAL & COMPLETE VIEW */}
      {(status === 'developing' || status === 'complete') && (
        <div className="bg-[#0A0D14] border border-white/10 rounded-2xl overflow-hidden flex flex-col h-[500px] shadow-2xl animate-in zoom-in-95 duration-500">
          
          {/* Terminal Header */}
          <div className="bg-white/5 border-b border-white/10 px-4 py-3 flex items-center gap-4">
            <div className="flex gap-2">
              <div className="w-3 h-3 rounded-full bg-rose-500/80"></div>
              <div className="w-3 h-3 rounded-full bg-amber-500/80"></div>
              <div className="w-3 h-3 rounded-full bg-emerald-500/80"></div>
            </div>
            <div className="flex items-center gap-2 text-slate-400 text-xs font-mono bg-black/40 px-3 py-1 rounded-md">
              <Terminal className="w-3 h-3" />
              agent-orchestrator.py
            </div>
          </div>

          {/* Terminal Body */}
          <div className="flex-1 p-4 overflow-y-auto font-mono text-sm">
            {logs.map((log, i) => (
              <div key={i} className="mb-2 flex gap-3">
                <span className="text-slate-600 select-none">{`>`}</span>
                <span className={clsx(
                  log.includes('[ERROR]') && "text-rose-400",
                  log.includes('[WARN]') && "text-amber-400",
                  log.includes('[SUCCESS]') || log.includes('passed') && "text-emerald-400",
                  log.includes('[GIT]') && "text-blue-400",
                  (!log.includes('[ERROR]') && !log.includes('[WARN]') && !log.includes('[SUCCESS]') && !log.includes('passed') && !log.includes('[GIT]')) && "text-slate-300"
                )}>
                  {log}
                </span>
              </div>
            ))}
            
            {status === 'developing' && (
              <div className="flex gap-3 mt-4">
                <span className="text-slate-600 select-none">{`>`}</span>
                <span className="w-2 h-4 bg-indigo-500 animate-pulse"></span>
              </div>
            )}
            <div ref={terminalEndRef} />
          </div>

          {/* Success Overlay Panel */}
          {status === 'complete' && (
            <div className="bg-indigo-950/40 border-t border-indigo-500/30 p-6 backdrop-blur-md animate-in slide-in-from-bottom-full duration-500">
              <div className="flex items-center justify-between">
                <div>
                  <div className="flex items-center gap-2 text-emerald-400 font-medium mb-1">
                    <CheckCircle2 className="w-5 h-5" />
                    Code Generation Successful
                  </div>
                  <p className="text-sm text-slate-400 font-mono">
                    <span className="text-emerald-400">+ 842 lines</span> / <span className="text-rose-400">- 12 lines</span>
                  </p>
                </div>
                
                <div className="flex gap-3">
                  <button className="bg-[#24292F] hover:bg-[#24292F]/80 text-white px-4 py-2.5 rounded-lg text-sm font-medium transition-colors flex items-center gap-2">
                    <Github className="w-4 h-4" />
                    View Pull Request
                  </button>
                  <button 
                    onClick={onReset}
                    className="bg-indigo-600 hover:bg-indigo-500 text-white px-4 py-2.5 rounded-lg text-sm font-medium transition-colors flex items-center gap-2 shadow-lg shadow-indigo-500/25"
                  >
                    <RotateCcw className="w-4 h-4" />
                    Start New Project
                  </button>
                </div>
              </div>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```


ProjectWorkspace.tsx

```tsx
// Inside src/components/ProjectWorkspace.tsx
import { useState } from 'react';
import Sidebar from './Sidebar';
import Stage1Upload from './stages/Stage1Upload';
import Stage2Knowledge from './stages/Stage2Knowledge';
import Stage3Context from './stages/Stage3Context';
import ContextSynthesis from './stages/ContextSynthesis';
import Stage4Documentation from './stages/Stage4Documentation';
import Stage5Development from './stages/Stage5Development';

export default function ProjectWorkspace() {
  const [currentStage, setCurrentStage] = useState(1);
  const [isSynthesizing, setIsSynthesizing] = useState(false);

  const handleNextStage = async (stageData?: any) => {
    if (currentStage === 3) {
      setIsSynthesizing(true);
    } else {
      setCurrentStage(prev => prev + 1);
    }
  };

  const handleSynthesisComplete = () => {
    setIsSynthesizing(false);
    setCurrentStage(4);
  };

  const resetWorkspace = () => {
    setCurrentStage(1);
    // Here you would also clear any global state/context if you add state management later
  };

  return (
    <>
      <div className="flex h-screen w-full bg-transparent">
        <div className="w-72 border-r border-white/10 bg-black/20 backdrop-blur-md">
          <Sidebar currentStage={currentStage} />
        </div>

        <div className="flex-1 p-10 overflow-y-auto relative">
          <div className="max-w-4xl mx-auto mt-10">
            {currentStage === 1 && <Stage1Upload onComplete={handleNextStage} />}
            {currentStage === 2 && <Stage2Knowledge onComplete={handleNextStage} />}
            {currentStage === 3 && <Stage3Context onComplete={handleNextStage} />}
            {currentStage === 4 && <Stage4Documentation onComplete={handleNextStage} />}
            {currentStage === 5 && <Stage5Development onReset={resetWorkspace} />}
          </div>
        </div>
      </div>

      <AnimatePresence>
        {isSynthesizing && <ContextSynthesis onComplete={handleSynthesisComplete} />}
      </AnimatePresence>
    </>
  );
}
```

Create a new file at src/components/NewProjectModal.tsx

```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { X, FolderPlus, ArrowRight } from 'lucide-react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  onInitialize: (name: string, description: string) => void;
}

export default function NewProjectModal({ isOpen, onClose, onInitialize }: ModalProps) {
  const [name, setName] = useState('');
  const [description, setDescription] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (name.trim()) {
      onInitialize(name, description);
    }
  };

  return (
    <AnimatePresence>
      {isOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center px-4">
          {/* Backdrop */}
          <motion.div 
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={onClose}
            className="absolute inset-0 bg-[#0B0E14]/80 backdrop-blur-sm"
          />

          {/* Modal Content */}
          <motion.div 
            initial={{ opacity: 0, scale: 0.95, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.95, y: 20 }}
            className="relative w-full max-w-md bg-[#121620] border border-white/10 rounded-2xl shadow-2xl overflow-hidden"
          >
            <div className="px-6 py-5 border-b border-white/10 flex items-center justify-between bg-white/[0.02]">
              <div className="flex items-center gap-3">
                <div className="w-8 h-8 rounded-lg bg-indigo-500/20 flex items-center justify-center">
                  <FolderPlus className="w-4 h-4 text-indigo-400" />
                </div>
                <h2 className="text-lg font-semibold text-white">Initialize Workspace</h2>
              </div>
              <button 
                onClick={onClose}
                className="text-slate-400 hover:text-white transition-colors"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <form onSubmit={handleSubmit} className="p-6">
              <div className="space-y-5">
                <div>
                  <label className="block text-sm font-medium text-slate-300 mb-1.5">Project Name</label>
                  <input 
                    type="text" 
                    value={name}
                    onChange={(e) => setName(e.target.value)}
                    placeholder="e.g., DocForge Alpha"
                    autoFocus
                    className="w-full bg-black/40 border border-white/10 rounded-xl py-2.5 px-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200 transition-colors"
                  />
                </div>
                
                <div>
                  <label className="block text-sm font-medium text-slate-300 mb-1.5">Description (Optional)</label>
                  <textarea 
                    value={description}
                    onChange={(e) => setDescription(e.target.value)}
                    placeholder="Briefly describe the goal of this project..."
                    rows={3}
                    className="w-full bg-black/40 border border-white/10 rounded-xl py-2.5 px-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200 transition-colors resize-none"
                  />
                </div>
              </div>

              <div className="mt-8 flex items-center justify-end gap-3">
                <button 
                  type="button"
                  onClick={onClose}
                  className="px-4 py-2 text-sm font-medium text-slate-400 hover:text-white transition-colors"
                >
                  Cancel
                </button>
                <button 
                  type="submit"
                  disabled={!name.trim()}
                  className="bg-indigo-600 hover:bg-indigo-500 disabled:bg-white/5 disabled:text-slate-500 text-white px-5 py-2.5 rounded-lg text-sm font-medium transition-all flex items-center gap-2 shadow-lg shadow-indigo-500/25"
                >
                  Create Project
                  <ArrowRight className="w-4 h-4" />
                </button>
              </div>
            </form>
          </motion.div>
        </div>
      )}
    </AnimatePresence>
  );
}
```

Replace the contents of src/app/page.tsx

```tsx
'use client'
import { useState } from 'react';
import { Plus, Search, Layers, Clock, Settings, LogOut } from 'lucide-react';
import clsx from 'clsx';
import ProjectWorkspace from '@/components/ProjectWorkspace';
import NewProjectModal from '@/components/NewProjectModal';

// Mock data for existing projects shown on the dashboard
const RECENT_PROJECTS = [
  { id: 1, name: 'DocForge Alpha', desc: 'AI orchestration for software requirements', phase: 'Phase 2', stage: 'Stage 5', updated: '2 hours ago' },
  { id: 2, name: 'Legacy Migration', desc: 'Refactoring the old billing service', phase: 'Phase 1', stage: 'Stage 2', updated: '1 day ago' },
  { id: 3, name: 'Mobile App API', desc: 'GraphQL endpoints for iOS client', phase: 'Completed', stage: 'Deployed', updated: '3 days ago' },
];

export default function Home() {
  const [activeView, setActiveView] = useState<'dashboard' | 'workspace'>('dashboard');
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  // This would hold your active project data in a real app
  const [currentProject, setCurrentProject] = useState<{name: string} | null>(null);

  const handleCreateProject = (name: string, description: string) => {
    setIsModalOpen(false);
    setCurrentProject({ name });
    setActiveView('workspace');
  };

  const handleReturnToDashboard = () => {
    setActiveView('dashboard');
    setCurrentProject(null);
  };

  // If we are in a project, render the multi-stage orchestration flow
  if (activeView === 'workspace') {
    return (
      <div className="relative">
        {/* We pass a custom header/back button to overlay on top of the workspace if needed */}
        <button 
          onClick={handleReturnToDashboard}
          className="absolute top-6 right-8 z-50 text-sm text-slate-400 hover:text-white transition-colors flex items-center gap-2 bg-black/20 px-4 py-2 rounded-lg border border-white/5 backdrop-blur-md"
        >
          Return to Dashboard
        </button>
        <ProjectWorkspace />
      </div>
    );
  }

  // Otherwise, render the Home Dashboard
  return (
    <div className="min-h-screen flex flex-col items-center pt-24 pb-12 px-6">
      
      {/* Top Navigation Bar (Mockup) */}
      <nav className="fixed top-0 w-full border-b border-white/5 bg-[#0B0E14]/80 backdrop-blur-xl z-40 px-6 py-4 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div className="w-8 h-8 rounded-lg bg-indigo-600 flex items-center justify-center font-bold text-white shadow-lg shadow-indigo-500/20">
            S
          </div>
          <span className="font-semibold text-lg tracking-wide text-white">Synapse AI</span>
        </div>
        <div className="flex items-center gap-6">
          <div className="flex items-center gap-4 text-sm font-medium text-slate-400">
            <span className="hover:text-white cursor-pointer transition-colors">Projects</span>
            <span className="hover:text-white cursor-pointer transition-colors">Integrations</span>
            <span className="hover:text-white cursor-pointer transition-colors">Settings</span>
          </div>
          <div className="w-px h-5 bg-white/10" />
          <div className="w-8 h-8 rounded-full bg-gradient-to-tr from-indigo-500 to-purple-500 border border-white/20" />
        </div>
      </nav>

      <div className="w-full max-w-5xl animate-in fade-in slide-in-from-bottom-4 duration-700">
        
        {/* Dashboard Header */}
        <div className="flex items-end justify-between mb-12">
          <div>
            <h1 className="text-4xl font-bold text-white mb-3 tracking-tight">Welcome back, Developer</h1>
            <p className="text-slate-400">Manage your AI-orchestrated workspaces and projects.</p>
          </div>
          <button 
            onClick={() => setIsModalOpen(true)}
            className="bg-indigo-600 hover:bg-indigo-500 text-white px-6 py-3 rounded-xl font-medium transition-all flex items-center gap-2 shadow-lg shadow-indigo-500/25"
          >
            <Plus className="w-5 h-5" />
            New Project
          </button>
        </div>

        {/* Search and Filter */}
        <div className="flex items-center gap-4 mb-8">
          <div className="relative flex-1 max-w-md">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-500" />
            <input 
              type="text" 
              placeholder="Search projects..." 
              className="w-full bg-white/[0.02] border border-white/10 rounded-xl py-2.5 pl-10 pr-4 text-sm focus:outline-none focus:border-indigo-500 text-slate-200 transition-colors"
            />
          </div>
        </div>

        {/* Projects Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {RECENT_PROJECTS.map((project) => (
            <div 
              key={project.id}
              onClick={() => {
                setCurrentProject({ name: project.name });
                setActiveView('workspace');
              }}
              className="bg-white/[0.02] border border-white/10 rounded-2xl p-6 hover:bg-white/[0.04] hover:border-white/20 transition-all cursor-pointer group flex flex-col"
            >
              <div className="flex items-start justify-between mb-4">
                <div className="w-10 h-10 rounded-xl bg-indigo-500/10 flex items-center justify-center text-indigo-400 group-hover:scale-110 transition-transform">
                  <Layers className="w-5 h-5" />
                </div>
                <div className="flex items-center gap-1 text-xs font-medium text-slate-500 bg-black/40 px-2.5 py-1 rounded-md">
                  <Clock className="w-3 h-3" />
                  {project.updated}
                </div>
              </div>
              
              <h3 className="text-lg font-semibold text-slate-200 mb-2">{project.name}</h3>
              <p className="text-sm text-slate-400 mb-6 flex-1">{project.desc}</p>
              
              <div className="flex items-center justify-between border-t border-white/5 pt-4">
                <div className="flex flex-col">
                  <span className="text-xs text-slate-500 uppercase font-semibold tracking-wider mb-1">Status</span>
                  <span className={clsx(
                    "text-sm font-medium",
                    project.phase === 'Completed' ? "text-emerald-400" : "text-indigo-400"
                  )}>
                    {project.phase} • {project.stage}
                  </span>
                </div>
              </div>
            </div>
          ))}
        </div>

      </div>

      <NewProjectModal 
        isOpen={isModalOpen} 
        onClose={() => setIsModalOpen(false)} 
        onInitialize={handleCreateProject}
      />
    </div>
  );
}
```

run using 

```bash
npm run dev
```


PYTHON BACKEND


Install Dependencies
```bash
pip install fastapi uvicorn python-multipart
```


Create main.py

```python
from fastapi import FastAPI, UploadFile, File, Form
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse
import asyncio
import random

app = FastAPI(title="DocForge API")

# Allow your Next.js frontend to communicate with this API
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"], # Update if your frontend runs on a different port
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# --- STAGE 1 & 2: Document Processing ---
@app.post("/api/v1/process-docs")
async def process_documents(file: UploadFile = File(...)):
    # Here you would actually process the file, extract text, etc.
    await asyncio.sleep(2) # Simulating backend work (e.g., parsing, chunking)
    return {"status": "success", "filename": file.filename, "extracted_chunks": 42}

@app.get("/api/v1/context-search")
async def search_nas_context(query: str = ""):
    # Here you would query your FAISS vector database
    await asyncio.sleep(1.5)
    return {
        "results": [
            {"id": "nas-1", "name": "auth-system-requirements.pdf", "relevancy": 96, "size": "1.2 MB"},
            {"id": "nas-2", "name": "legacy-db-schema-v3.pdf", "relevancy": 82, "size": "3.4 MB"},
        ]
    }

# --- STAGE 3: Context URLs ---
@app.post("/api/v1/resolve-url")
async def resolve_url(query: dict):
    # Simulates the 60% chance of finding a URL automatically
    await asyncio.sleep(2)
    is_success = random.random() <= 0.60
    url = f"https://github.com/auto-resolved/{query.get('term', 'repo').replace(' ', '-').lower()}" if is_success else ""
    return {"success": is_success, "url": url}

# --- STAGE 4: Documentation Generation ---
@app.post("/api/v1/generate-specs")
async def generate_specifications(data: dict):
    # Here you would call your LLM (OpenAI, Anthropic, local, etc.)
    await asyncio.sleep(3)
    
    markdown_content = """# Technical Specification: DocForge Alpha
## 1. System Architecture
The system utilizes a microservices architecture pattern...

## 2. API Endpoints
- `POST /api/v1/process-docs`
- `GET /api/v1/context-search`
"""
    return {"status": "success", "markdown": markdown_content}

# --- STAGE 5: Live Terminal Streaming (SSE) ---
async def log_generator():
    """Yields logs one by one to simulate a live terminal stream."""
    logs = [
        "[SYSTEM] Initializing secure development sandbox...",
        "[GIT] Created new branch: feature/auto-generated-specs",
        "[AGENT] Analyzing Technical Specifications (tech_specs.md)...",
        "[AGENT] Generating src/lib/db/schema.ts...",
        "[INFO] Successfully wrote 124 lines to schema.ts",
        "[GIT] Committing: 'feat: AI generated base orchestration flow'",
        "[SYSTEM] Development cycle complete. Ready for review."
    ]
    
    for log in logs:
        yield f"data: {log}\n\n"
        await asyncio.sleep(0.8) # Wait between logs

@app.get("/api/v1/develop/stream")
async def stream_development():
    # Returns a Server-Sent Events (SSE) stream
    return StreamingResponse(log_generator(), media_type="text/event-stream")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```


In src/components/stages/Stage1Upload.tsx

```tsx
const handleProcess = async () => {
    if (!file) return;
    setIsProcessing(true);
    
    try {
      // 1. Prepare the file for upload
      const formData = new FormData();
      formData.append('file', file);

      // 2. Call the Python API
      const response = await fetch('http://localhost:8000/api/v1/process-docs', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) throw new Error('Upload failed');
      
      const data = await response.json();
      
      // 3. Move to next stage on success
      onComplete({ fileName: data.filename, chunks: data.extracted_chunks });
      
    } catch (error) {
      console.error("Error processing document:", error);
      // Handle error state in UI here
    } finally {
      setIsProcessing(false);
    }
  };
```

In src/components/stages/Stage4Documentation.tsx

```tsx
const [generatedMarkdown, setGeneratedMarkdown] = useState('');

  const handleGenerate = async () => {
    setStatus('generating');
    
    try {
      const response = await fetch('http://localhost:8000/api/v1/generate-specs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ context: "Pass Phase 1 & 2 data here eventually" })
      });

      if (!response.ok) throw new Error('Generation failed');
      
      const data = await response.json();
      setGeneratedMarkdown(data.markdown); // Store the real backend text
      setStatus('success');
      
    } catch (error) {
      console.error("Error generating specs:", error);
      setStatus('error');
    }
  };
```


In src/components/stages/Stage5Development.tsx

```tsx
// Replace the old setTimeout useEffect with this EventSource approach
  useEffect(() => {
    if (status === 'developing') {
      setLogs([]); // Clear previous logs
      
      // Connect to the Python SSE endpoint
      const eventSource = new EventSource('http://localhost:8000/api/v1/develop/stream');

      // Listen for incoming log lines
      eventSource.onmessage = (event) => {
        setLogs(prev => [...prev, event.data]);
        
        // Check if this is the final log to trigger the completion state
        if (event.data.includes("Development cycle complete")) {
          eventSource.close();
          setTimeout(() => setStatus('complete'), 1000);
        }
      };

      // Handle connection errors
      eventSource.onerror = (error) => {
        console.error("EventSource failed:", error);
        eventSource.close();
        setLogs(prev => [...prev, "[ERROR] Connection to development agent lost."]);
        // Optionally setStatus('error') if you add an error state
      };

      // Cleanup listener when component unmounts or status changes
      return () => {
        eventSource.close();
      };
    }
  }, [status]);
```





