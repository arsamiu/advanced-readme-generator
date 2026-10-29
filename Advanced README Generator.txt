import React, { useState, useEffect, useRef } from 'react';
import { 
  Settings, Download, Copy, LayoutTemplate, Github, Code, Check, 
  ChevronDown, ChevronUp, AlignLeft, List, Terminal, Activity, FileJson, 
  Loader2, Sparkles, BookOpen, Shield
} from 'lucide-react';

// --- Constants & Data ---

const AVAILABLE_BADGES = [
  { id: 'react', name: 'React', url: 'https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB' },
  { id: 'next', name: 'Next.js', url: 'https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white' },
  { id: 'vue', name: 'Vue.js', url: 'https://img.shields.io/badge/vuejs-%2335495e.svg?style=for-the-badge&logo=vuedotjs&logoColor=%234FC08D' },
  { id: 'angular', name: 'Angular', url: 'https://img.shields.io/badge/angular-%23DD0031.svg?style=for-the-badge&logo=angular&logoColor=white' },
  { id: 'svelte', name: 'Svelte', url: 'https://img.shields.io/badge/svelte-%23f1413d.svg?style=for-the-badge&logo=svelte&logoColor=white' },
  { id: 'node', name: 'Node.js', url: 'https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white' },
  { id: 'python', name: 'Python', url: 'https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54' },
  { id: 'javascript', name: 'JavaScript', url: 'https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E' },
  { id: 'typescript', name: 'TypeScript', url: 'https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white' },
  { id: 'html5', name: 'HTML5', url: 'https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white' },
  { id: 'css3', name: 'CSS3', url: 'https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white' },
  { id: 'tailwind', name: 'Tailwind CSS', url: 'https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white' },
  { id: 'docker', name: 'Docker', url: 'https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white' },
  { id: 'aws', name: 'AWS', url: 'https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white' },
  { id: 'gcp', name: 'Google Cloud', url: 'https://img.shields.io/badge/GoogleCloud-%234285F4.svg?style=for-the-badge&logo=google-cloud&logoColor=white' },
  { id: 'azure', name: 'Azure', url: 'https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white' },
  { id: 'mongodb', name: 'MongoDB', url: 'https://img.shields.io/badge/MongoDB-%234ea94b.svg?style=for-the-badge&logo=mongodb&logoColor=white' },
  { id: 'postgresql', name: 'PostgreSQL', url: 'https://img.shields.io/badge/postgresql-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white' },
  { id: 'redis', name: 'Redis', url: 'https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white' },
  { id: 'firebase', name: 'Firebase', url: 'https://img.shields.io/badge/firebase-%23039BE5.svg?style=for-the-badge&logo=firebase' },
  { id: 'supabase', name: 'Supabase', url: 'https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white' },
  { id: 'git', name: 'Git', url: 'https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white' },
  { id: 'linux', name: 'Linux', url: 'https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black' },
];

const LICENSES = ['MIT', 'GPLv3', 'Apache 2.0', 'ISC', 'Unlicense'];
const STATS_THEMES = ['radium', 'dark', 'dracula', 'tokyonight', 'github_dark', 'nord', 'transparent'];

// --- Reusable UI Components ---

const Toggle = ({ enabled, onChange, label }) => (
  <label className="flex items-center cursor-pointer select-none">
    <div className="relative">
      <input type="checkbox" className="sr-only" checked={enabled} onChange={onChange} />
      <div className={`block w-10 h-6 rounded-full transition-colors duration-300 ${enabled ? 'bg-blue-600' : 'bg-gray-700'}`}></div>
      <div className={`absolute left-1 top-1 bg-white w-4 h-4 rounded-full transition-transform duration-300 shadow-sm ${enabled ? 'transform translate-x-4' : ''}`}></div>
    </div>
    {label && <span className="ml-3 text-sm font-medium text-gray-300">{label}</span>}
  </label>
);

const AIButton = ({ onClick, loading }) => (
  <button 
    onClick={onClick}
    disabled={loading}
    className="flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold rounded-md bg-gradient-to-r from-violet-600 to-indigo-600 hover:from-violet-500 hover:to-indigo-500 text-white shadow-md shadow-indigo-500/20 disabled:opacity-70 transition-all border border-indigo-500/50"
    title="Write a short summary and let AI expand it"
  >
    {loading ? <Loader2 className="w-3.5 h-3.5 animate-spin" /> : <Sparkles className="w-3.5 h-3.5" />}
    {loading ? 'Thinking...' : 'Generate with AI'}
  </button>
);

const SectionCard = ({ title, icon: Icon, defaultOpen = true, children, toggleEnabled, onToggle }) => {
  const [isOpen, setIsOpen] = useState(defaultOpen);

  return (
    <div className="bg-[#161b22] rounded-xl border border-[#30363d] overflow-hidden mb-5 shadow-sm transition-all duration-300">
      <div 
        className="px-5 py-4 flex items-center justify-between bg-[#161b22] hover:bg-[#1c2128] transition-colors cursor-pointer select-none border-b border-transparent"
        onClick={() => setIsOpen(!isOpen)}
      >
        <div className="flex items-center gap-3">
          {toggleEnabled !== undefined && (
            <div onClick={(e) => e.stopPropagation()}>
               <Toggle enabled={toggleEnabled} onChange={onToggle} />
            </div>
          )}
          <div className="flex items-center gap-2.5 text-gray-100 font-semibold tracking-wide">
            {Icon && <Icon className="w-4 h-4 text-blue-400" />}
            {title}
          </div>
        </div>
        {isOpen ? <ChevronUp className="w-4 h-4 text-gray-500" /> : <ChevronDown className="w-4 h-4 text-gray-500" />}
      </div>
      {isOpen && (
        <div className="p-5 border-t border-[#30363d] bg-[#0d1117]/30">
          {children}
        </div>
      )}
    </div>
  );
};

// --- Main Application ---

export default function App() {
  // State: Section Visibility Toggles
  const [sections, setSections] = useState({
    badges: true,
    stats: true,
    features: true,
    installation: true,
    usage: true,
    contributing: true,
    license: true
  });

  // State: GitHub Stats Config
  const [statsConfig, setStatsConfig] = useState({
    profile: true,
    streak: true,
    languages: true,
    theme: 'radium'
  });

  // State: Form Content
  const [formData, setFormData] = useState({
    title: 'Awesome Project',
    description: 'A powerful tool to help developers build amazing applications quickly.',
    githubUsername: 'yourusername',
    repoName: 'awesome-project',
    badges: ['react', 'tailwind', 'typescript'],
    features: '- Fast and responsive UI\n- AI-powered generation\n- Highly customizable',
    installation: 'npm install awesome-project\n# or\nyarn add awesome-project',
    usage: 'import { Awesome } from "awesome-project";\n\nAwesome.init();',
    license: 'MIT',
  });

  // State: App UI Status
  const [aiLoading, setAiLoading] = useState({});
  const [markdown, setMarkdown] = useState('');
  const [copied, setCopied] = useState(false);
  const [activeTab, setActiveTab] = useState('preview'); // 'preview' or 'raw'

  // Update markdown automatically whenever state changes
  useEffect(() => {
    generateMarkdown();
  }, [formData, sections, statsConfig]);

  // --- Handlers ---

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const toggleSection = (sectionName) => {
    setSections((prev) => ({ ...prev, [sectionName]: !prev[sectionName] }));
  };

  const toggleBadge = (badgeId) => {
    setFormData((prev) => {
      const isSelected = prev.badges.includes(badgeId);
      return {
        ...prev,
        badges: isSelected 
          ? prev.badges.filter(id => id !== badgeId)
          : [...prev.badges, badgeId]
      };
    });
  };

  const handleAIGeneration = async (field) => {
    if (!formData[field] || !formData[field].trim()) return;
    
    setAiLoading(prev => ({ ...prev, [field]: true }));
    try {
      let systemPrompt = "";
      if (field === 'description') {
        systemPrompt = "You are an expert technical writer. Expand the user's brief idea into a professional, engaging 1-2 paragraph description for an open-source project GitHub README. Return ONLY the description text, no markdown headers, no introductory text.";
      } else if (field === 'features') {
        systemPrompt = "You are an expert technical writer. Convert the user's text into a professional markdown bulleted list of 4-6 features for a GitHub README. Use relevant emojis for each bullet point. Return ONLY the markdown list, no headers.";
      } else if (field === 'installation') {
        systemPrompt = "You are an expert developer. Write a clear, professional markdown installation guide based on the user's hints. Use standard bash code blocks (```bash). Return ONLY the markdown content, no headers like 'Installation'.";
      } else if (field === 'usage') {
        systemPrompt = "You are an expert developer. Write a clear, professional markdown usage guide or code example based on the user's hints. Use appropriate code blocks. Return ONLY the markdown content, no headers.";
      }

      const payload = {
        contents: [{ parts: [{ text: formData[field] }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] }
      };
      
      const apiKey = ""; // Automatically handled by the environment
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;
      
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      
      const result = await response.json();
      if (result.candidates && result.candidates[0].content?.parts?.[0]?.text) {
        const text = result.candidates[0].content.parts[0].text.trim();
        setFormData(prev => ({ ...prev, [field]: text }));
      }
    } catch (e) {
      console.error("AI Generation failed", e);
    } finally {
      setAiLoading(prev => ({ ...prev, [field]: false }));
    }
  };

  // --- Markdown Generation ---

  const generateMarkdown = () => {
    let md = '';

    // 1. Title & Header
    md += `<h1 align="center">${formData.title}</h1>\n\n`;
    
    // Dynamic Badges (License + Selected)
    if (sections.badges) {
      md += `<p align="center">\n`;
      const licenseBadgeUrl = `https://img.shields.io/badge/license-${encodeURIComponent(formData.license)}-blue.svg`;
      md += `  <img src="${licenseBadgeUrl}" alt="License" />\n`;
      
      const selectedBadges = AVAILABLE_BADGES.filter(b => formData.badges.includes(b.id));
      selectedBadges.forEach(b => {
        md += `  <img src="${b.url}" alt="${b.name}" />\n`;
      });
      md += `</p>\n\n`;
    }

    // 2. Description
    if (formData.description) {
      md += `${formData.description}\n\n`;
    }

    // 3. Table of Contents
    const hasAnySection = sections.features || sections.stats || sections.installation || sections.usage || sections.contributing || sections.license;
    
    if (hasAnySection) {
      md += `## 📖 Table of Contents\n\n`;
      if (sections.features) md += `- [Features](#-features)\n`;
      if (sections.stats && formData.githubUsername) md += `- [GitHub Stats](#-github-stats)\n`;
      if (sections.installation) md += `- [Installation](#-installation)\n`;
      if (sections.usage) md += `- [Usage](#-usage)\n`;
      if (sections.contributing) md += `- [Contributing](#-contributing)\n`;
      if (sections.license) md += `- [License](#-license)\n`;
      md += `\n`;
    }

    // 4. Features
    if (sections.features && formData.features) {
      md += `## ✨ Features\n\n`;
      md += `${formData.features}\n\n`;
    }

    // 5. GitHub Stats
    if (sections.stats && formData.githubUsername) {
      md += `## 📊 GitHub Stats\n\n`;
      md += `<div align="center">\n`;
      if (statsConfig.profile) {
        md += `  <img src="https://github-readme-stats.vercel.app/api?username=${formData.githubUsername}&show_icons=true&theme=${statsConfig.theme}" alt="GitHub Stats" />\n`;
      }
      if (statsConfig.streak) {
         md += `  <img src="https://github-readme-streak-stats.herokuapp.com/?user=${formData.githubUsername}&theme=${statsConfig.theme}" alt="GitHub Streak" />\n`;
      }
      if (statsConfig.languages) {
        md += `  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=${formData.githubUsername}&layout=compact&theme=${statsConfig.theme}" alt="Top Languages" />\n`;
      }
      md += `</div>\n\n`;
    }

    // 6. Installation
    if (sections.installation && formData.installation) {
      md += `## 🚀 Installation\n\n`;
      if (!formData.installation.startsWith('```')) {
          md += `\`\`\`bash\n${formData.installation}\n\`\`\`\n\n`;
      } else {
          md += `${formData.installation}\n\n`;
      }
    }

    // 7. Usage
    if (sections.usage && formData.usage) {
      md += `## 💻 Usage\n\n`;
      if (!formData.usage.startsWith('```')) {
          md += `\`\`\`javascript\n${formData.usage}\n\`\`\`\n\n`;
      } else {
          md += `${formData.usage}\n\n`;
      }
    }

    // 8. Contributing
    if (sections.contributing) {
      md += `## 🤝 Contributing\n\n`;
      md += `Contributions, issues and feature requests are welcome!<br />Feel free to check [issues page](https://github.com/${formData.githubUsername}/${formData.repoName}/issues). \n\n`;
    }

    // 9. License
    if (sections.license) {
      md += `## 📝 License\n\n`;
      md += `This project is [${formData.license}](https://choosealicense.com/licenses/${formData.license.toLowerCase().replace(' ', '-')}/) licensed.\n\n`;
    }

    setMarkdown(md);
  };

  // --- Export Functions ---

  const copyToClipboard = () => {
    navigator.clipboard.writeText(markdown).then(() => {
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    });
  };

  const downloadFile = () => {
    const element = document.createElement("a");
    const file = new Blob([markdown], {type: 'text/plain'});
    element.href = URL.createObjectURL(file);
    element.download = "README.md";
    document.body.appendChild(element); 
    element.click();
    document.body.removeChild(element);
  };

  // --- Simple Markdown Parser for Preview ---
  // Safely parses structural markdown while allowing raw HTML tags generated above to pass through
  const renderSimpleMarkdown = (text) => {
    if (!text) return { __html: '' };
    
    let html = text
      // Headers
      .replace(/^### (.*$)/gim, '<h3>$1</h3>')
      .replace(/^## (.*$)/gim, '<h2>$1</h2>')
      .replace(/^# (.*$)/gim, '<h1>$1</h1>')
      
      // Inline Code
      .replace(/`([^`\n]+)`/g, '<code>$1</code>')
      
      // Code blocks (multi-line)
      .replace(/```(\w*)\n([\s\S]*?)```/gm, '<pre><code class="language-$1">$2</code></pre>')
      
      // Standard Markdown Links (Not inside HTML)
      .replace(/(?<!["']\s*)\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>')
      
      // Bold
      .replace(/\*\*([^*]+)\*\*/g, '<strong>$1</strong>')
      
      // Lists (Very basic interpretation)
      .replace(/^\s*-\s(.*)$/gim, '<li>$1</li>');
      
      // Note: Deliberately avoiding \n\n replacement to prevent messing up the HTML 
      // structures we injected (like <div align="center">). We will use CSS white-space 
      // or handle spacing natively.

    return { __html: html };
  };

  // --- Render ---

  return (
    <div className="min-h-screen bg-[#0d1117] text-gray-300 font-sans flex flex-col selection:bg-blue-500/30">
      
      {/* --- Global CSS for GitHub Markdown Style --- */}
      <style dangerouslySetInnerHTML={{__html: `
        /* Custom Scrollbar for Dark Theme */
        .custom-scrollbar::-webkit-scrollbar { width: 8px; height: 8px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #0d1117; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #30363d; border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #484f58; }

        /* GitHub Markdown Preview Simulation */
        .markdown-body {
          color: #c9d1d9;
          font-family: -apple-system,BlinkMacSystemFont,"Segoe UI",Helvetica,Arial,sans-serif;
          line-height: 1.5;
          word-wrap: break-word;
        }
        .markdown-body h1, .markdown-body h2 {
          border-bottom: 1px solid #21262d;
          padding-bottom: 0.3em;
          margin-top: 24px;
          margin-bottom: 16px;
          font-weight: 600;
          color: #e6edf3;
        }
        .markdown-body h1 { font-size: 2em; }
        .markdown-body h2 { font-size: 1.5em; }
        .markdown-body h3 { font-size: 1.25em; margin-top: 24px; margin-bottom: 16px; font-weight: 600; }
        
        .markdown-body a { color: #58a6ff; text-decoration: none; }
        .markdown-body a:hover { text-decoration: underline; }
        
        .markdown-body pre { 
          background-color: #161b22; 
          border: 1px solid #30363d;
          border-radius: 6px; 
          padding: 16px; 
          overflow: auto;
          margin-top: 0;
          margin-bottom: 16px;
        }
        .markdown-body pre code {
          background-color: transparent;
          padding: 0;
          border: 0;
          font-family: ui-monospace,SFMono-Regular,SF Mono,Menlo,Consolas,Liberation Mono,monospace;
          font-size: 85%;
          color: #e6edf3;
        }
        .markdown-body :not(pre) > code {
          background-color: rgba(110,118,129,0.4);
          padding: 0.2em 0.4em;
          border-radius: 6px;
          font-family: ui-monospace,SFMono-Regular,SF Mono,Menlo,Consolas,Liberation Mono,monospace;
          font-size: 85%;
        }
        
        .markdown-body img { max-width: 100%; box-sizing: content-box; background-color: transparent; }
        .markdown-body ul { padding-left: 2em; margin-top: 0; margin-bottom: 16px; list-style-type: disc;}
        .markdown-body li { word-wrap: break-all; margin-top: 0.25em; }
        .markdown-body p { margin-top: 0; margin-bottom: 16px; white-space: pre-wrap;}
        .markdown-body align-center, .markdown-body div[align="center"] { display: flex; flex-direction: column; align-items: center; gap: 8px;}
        .markdown-body div[align="center"] img { margin-bottom: 8px;}
      `}} />

      {/* --- Top Navigation Header --- */}
      <header className="flex-none bg-[#161b22] border-b border-[#30363d] py-3 px-6 flex items-center justify-between z-10 shadow-sm">
        <div className="flex items-center gap-3">
          <div className="p-2 bg-indigo-500/10 rounded-lg border border-indigo-500/20 shadow-inner">
            <LayoutTemplate className="w-6 h-6 text-indigo-400" />
          </div>
          <div>
            <h1 className="text-xl font-bold text-gray-100 tracking-tight flex items-center gap-2">
              README<span className="text-indigo-400">AI</span>
            </h1>
            <p className="text-[11px] text-gray-400 font-medium tracking-wide uppercase mt-0.5">Advanced Profile Generator</p>
          </div>
        </div>
        
        <div className="flex items-center gap-2 bg-[#0d1117] rounded-lg p-1 border border-[#30363d]">
          <button 
            onClick={() => setActiveTab('preview')}
            className={`px-4 py-1.5 rounded-md text-sm font-medium transition-all duration-200 flex items-center gap-2 ${
              activeTab === 'preview' 
                ? 'bg-[#161b22] text-indigo-400 shadow-sm border border-[#30363d]' 
                : 'text-gray-500 hover:text-gray-300 hover:bg-[#161b22]/50'
            }`}
          >
            <BookOpen className="w-4 h-4" /> Live Preview
          </button>
          <button 
            onClick={() => setActiveTab('raw')}
            className={`px-4 py-1.5 rounded-md text-sm font-medium transition-all duration-200 flex items-center gap-2 ${
              activeTab === 'raw' 
                ? 'bg-[#161b22] text-indigo-400 shadow-sm border border-[#30363d]' 
                : 'text-gray-500 hover:text-gray-300 hover:bg-[#161b22]/50'
            }`}
          >
            <Code className="w-4 h-4" /> Raw MD
          </button>
        </div>
      </header>

      {/* --- Main Workspace (Split Pane) --- */}
      <div className="flex-1 flex flex-col lg:flex-row overflow-hidden">
        
        {/* LEFT PANE: Editor / Configuration */}
        <div className="w-full lg:w-[45%] h-full overflow-y-auto custom-scrollbar border-r border-[#30363d] bg-[#0d1117]">
          <div className="p-6 max-w-2xl mx-auto pb-24">
            
            <div className="mb-6">
              <h2 className="text-2xl font-bold text-gray-100">Setup Project</h2>
              <p className="text-sm text-gray-500 mt-1">Fill in the details or use AI to generate professional content.</p>
            </div>

            {/* Section 1: General Info */}
            <SectionCard title="General Information" icon={AlignLeft}>
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-400 mb-1.5">Project Title</label>
                  <input
                    type="text"
                    name="title"
                    value={formData.title}
                    onChange={handleInputChange}
                    className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-2 text-gray-100 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none"
                    placeholder="e.g., NextGen Framework"
                  />
                </div>
                
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-1.5">GitHub Username</label>
                    <input
                      type="text"
                      name="githubUsername"
                      value={formData.githubUsername}
                      onChange={handleInputChange}
                      className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-2 text-gray-100 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none"
                      placeholder="e.g., octocat"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-1.5">Repository Name</label>
                    <input
                      type="text"
                      name="repoName"
                      value={formData.repoName}
                      onChange={handleInputChange}
                      className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-2 text-gray-100 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none"
                      placeholder="e.g., hello-world"
                    />
                  </div>
                </div>

                <div>
                  <div className="flex items-center justify-between mb-1.5">
                    <label className="block text-sm font-medium text-gray-400">Description</label>
                    <AIButton onClick={() => handleAIGeneration('description')} loading={aiLoading.description} />
                  </div>
                  <textarea
                    name="description"
                    value={formData.description}
                    onChange={handleInputChange}
                    rows="3"
                    className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-3 text-gray-100 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none resize-y"
                    placeholder="Type a short idea and click 'Generate with AI'..."
                  />
                </div>
              </div>
            </SectionCard>

            {/* Section 2: Tech Stack Badges */}
            <SectionCard 
              title="Tech Stack & Badges" 
              icon={LayersIcon} 
              toggleEnabled={sections.badges} 
              onToggle={() => toggleSection('badges')}
            >
              <p className="text-xs text-gray-500 mb-4">Select technologies to display professional Shields.io badges at the top of your README.</p>
              <div className="flex flex-wrap gap-2.5">
                {AVAILABLE_BADGES.map((badge) => {
                  const isSelected = formData.badges.includes(badge.id);
                  return (
                    <button
                      key={badge.id}
                      onClick={() => toggleBadge(badge.id)}
                      className={`
                        px-3 py-1.5 rounded-full text-xs font-medium transition-all duration-200 flex items-center gap-1.5 border
                        ${isSelected 
                          ? 'bg-indigo-500/10 text-indigo-300 border-indigo-500/50 shadow-[0_0_10px_rgba(99,102,241,0.15)]' 
                          : 'bg-[#0d1117] text-gray-400 border-[#30363d] hover:border-gray-600 hover:text-gray-300'
                        }
                      `}
                    >
                      {isSelected && <Check className="w-3 h-3" />}
                      {badge.name}
                    </button>
                  );
                })}
              </div>
            </SectionCard>

            {/* Section 3: GitHub Profile Stats */}
            <SectionCard 
              title="GitHub Dynamic Stats" 
              icon={Activity} 
              toggleEnabled={sections.stats} 
              onToggle={() => toggleSection('stats')}
              defaultOpen={false}
            >
              <div className="space-y-5">
                <div className="bg-[#0d1117] border border-[#30363d] p-4 rounded-lg flex flex-col gap-4">
                  <Toggle 
                    enabled={statsConfig.profile} 
                    onChange={() => setStatsConfig(p => ({...p, profile: !p.profile}))} 
                    label="Show General Profile Stats" 
                  />
                  <Toggle 
                    enabled={statsConfig.streak} 
                    onChange={() => setStatsConfig(p => ({...p, streak: !p.streak}))} 
                    label="Show Contribution Streak" 
                  />
                  <Toggle 
                    enabled={statsConfig.languages} 
                    onChange={() => setStatsConfig(p => ({...p, languages: !p.languages}))} 
                    label="Show Top Languages Card" 
                  />
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-400 mb-1.5">Card Theme</label>
                  <select
                    value={statsConfig.theme}
                    onChange={(e) => setStatsConfig(p => ({...p, theme: e.target.value}))}
                    className="w-full appearance-none bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-2 text-gray-300 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 outline-none cursor-pointer"
                  >
                    {STATS_THEMES.map(theme => (
                      <option key={theme} value={theme}>{theme.charAt(0).toUpperCase() + theme.slice(1).replace('_', ' ')}</option>
                    ))}
                  </select>
                </div>
              </div>
            </SectionCard>

            {/* Section 4: Features */}
            <SectionCard 
              title="Features" 
              icon={List} 
              toggleEnabled={sections.features} 
              onToggle={() => toggleSection('features')}
            >
              <div>
                <div className="flex items-center justify-between mb-1.5">
                  <label className="block text-sm font-medium text-gray-400">List of Features</label>
                  <AIButton onClick={() => handleAIGeneration('features')} loading={aiLoading.features} />
                </div>
                <textarea
                  name="features"
                  value={formData.features}
                  onChange={handleInputChange}
                  rows="5"
                  className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-3 text-gray-100 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none resize-y font-mono text-sm"
                  placeholder="e.g. User Authentication\nReal-time updates"
                />
              </div>
            </SectionCard>

            {/* Section 5: Installation */}
            <SectionCard 
              title="Installation Guide" 
              icon={Terminal} 
              toggleEnabled={sections.installation} 
              onToggle={() => toggleSection('installation')}
              defaultOpen={false}
            >
               <div>
                <div className="flex items-center justify-between mb-1.5">
                  <label className="block text-sm font-medium text-gray-400">Installation Steps</label>
                  <AIButton onClick={() => handleAIGeneration('installation')} loading={aiLoading.installation} />
                </div>
                <textarea
                  name="installation"
                  value={formData.installation}
                  onChange={handleInputChange}
                  rows="4"
                  className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-3 text-gray-300 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none resize-y font-mono text-sm"
                />
              </div>
            </SectionCard>

            {/* Section 6: Usage */}
            <SectionCard 
              title="Usage / Examples" 
              icon={Code} 
              toggleEnabled={sections.usage} 
              onToggle={() => toggleSection('usage')}
              defaultOpen={false}
            >
               <div>
                <div className="flex items-center justify-between mb-1.5">
                  <label className="block text-sm font-medium text-gray-400">Usage Instructions</label>
                  <AIButton onClick={() => handleAIGeneration('usage')} loading={aiLoading.usage} />
                </div>
                <textarea
                  name="usage"
                  value={formData.usage}
                  onChange={handleInputChange}
                  rows="4"
                  className="w-full bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-3 text-gray-300 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 transition-all outline-none resize-y font-mono text-sm"
                />
              </div>
            </SectionCard>

            {/* Section 7: Footer (License, Contributing) */}
            <SectionCard title="Footer Information" icon={Shield} defaultOpen={false}>
               <div className="space-y-5">
                  <Toggle 
                    enabled={sections.contributing} 
                    onChange={() => toggleSection('contributing')} 
                    label="Include 'Contributing' section linking to issues" 
                  />
                  
                  <div className="pt-2 border-t border-[#30363d]">
                    <div className="flex items-center justify-between mb-3">
                      <Toggle 
                        enabled={sections.license} 
                        onChange={() => toggleSection('license')} 
                        label="Include 'License' section" 
                      />
                    </div>
                    {sections.license && (
                      <select
                        name="license"
                        value={formData.license}
                        onChange={handleInputChange}
                        className="w-full appearance-none bg-[#0d1117] border border-[#30363d] rounded-lg px-4 py-2 text-gray-300 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500 outline-none cursor-pointer"
                      >
                        {LICENSES.map((lic) => (
                          <option key={lic} value={lic}>{lic}</option>
                        ))}
                      </select>
                    )}
                  </div>
               </div>
            </SectionCard>

          </div>
        </div>

        {/* RIGHT PANE: Live Preview */}
        <div className="w-full lg:w-[55%] h-full flex flex-col bg-[#0d1117] relative border-l border-black">
          
          {/* Action Tools Overlay */}
          <div className="absolute top-4 right-6 flex items-center gap-3 z-10">
            <button
              onClick={copyToClipboard}
              className={`flex items-center gap-2 px-4 py-2 rounded-lg font-medium text-sm transition-all duration-300 shadow-md
                ${copied 
                  ? 'bg-emerald-500/10 text-emerald-400 border border-emerald-500/50' 
                  : 'bg-[#21262d]/90 hover:bg-[#30363d] text-gray-200 border border-[#484f58] backdrop-blur-sm'
                }
              `}
            >
              {copied ? <Check className="w-4 h-4" /> : <Copy className="w-4 h-4" />}
              {copied ? 'Copied!' : 'Copy'}
            </button>
            
            <button
              onClick={downloadFile}
              className="flex items-center gap-2 px-4 py-2 bg-indigo-600 hover:bg-indigo-500 text-white rounded-lg font-medium text-sm transition-all duration-200 shadow-md shadow-indigo-500/20 border border-indigo-500/50"
            >
              <Download className="w-4 h-4" />
              Download
            </button>
          </div>

          {/* Rendering Container */}
          <div className="flex-1 overflow-y-auto custom-scrollbar p-8 pt-20">
            <div className="max-w-[800px] mx-auto w-full">
              
              {activeTab === 'preview' ? (
                // --- GitHub Styled Markdown Render ---
                <div className="bg-[#0d1117] p-2">
                  <div 
                    className="markdown-body"
                    dangerouslySetInnerHTML={renderSimpleMarkdown(markdown)} 
                  />
                </div>
              ) : (
                // --- Raw Markdown Code ---
                <div className="bg-[#161b22] border border-[#30363d] rounded-xl overflow-hidden shadow-inner">
                   <div className="flex items-center px-4 py-2 bg-[#0d1117] border-b border-[#30363d] text-xs text-gray-500 font-mono tracking-wide">
                    README.md
                  </div>
                  <pre className="p-6 overflow-x-auto custom-scrollbar text-[13px] font-mono text-gray-300 whitespace-pre-wrap break-words leading-relaxed selection:bg-indigo-500/30">
                    {markdown}
                  </pre>
                </div>
              )}

            </div>
          </div>
        </div>

      </div>
    </div>
  );
}

// Icon helper since we are using lucide-react and need a generic layers icon replacement
function LayersIcon(props) {
  return (
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}>
      <polygon points="12 2 2 7 12 12 22 7 12 2" />
      <polyline points="2 17 12 22 22 17" />
      <polyline points="2 12 12 17 22 12" />
    </svg>
  )
}