<template>
  <div class="ai-theme-wrapper">
    <div class="container glass-panel">
      <div class="header-section">
        <h1 class="gradient-text">AI Resume Analyzer</h1>
        <p class="subtitle">Upload your resume and let our neural network evaluate your profile.</p>
      </div>

      <div class="upload-box" :class="{ 'is-loading': isLoading, 'has-file': selectedFile }">
        <input 
          type="file" 
          id="cv-upload" 
          @change="handleFileUpload" 
          accept="application/pdf" 
          class="hidden-input"
        />
        <label for="cv-upload" class="upload-label">
          <div class="upload-icon">
            <svg v-if="!selectedFile" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
            </svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="#10b981">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
          </div>
          <span class="file-name">{{ selectedFile ? selectedFile.name : 'Drag & Drop or Click to Upload PDF' }}</span>
        </label>
        
        <button class="action-btn" @click="analyzeCV" :disabled="!selectedFile || isLoading">
          <span v-if="!isLoading">Analyze Resume</span>
          <span v-else class="loader-text">
            <span class="spinner"></span> Analyzing...
          </span>
        </button>
      </div>

      <transition name="fade">
        <div v-if="errorMessage" class="error-banner glow-red">
          ⚠️ {{ errorMessage }}
        </div>
      </transition>

      <transition name="slide-up">
        <div v-if="result" class="result-wrapper">
          <div class="score-card glass-panel glow-blue">
            <h3>AI Match Score</h3>
            <div class="score-circle">
              <span class="score-number">{{ result.score }}</span>
              <span class="score-max">/ 100</span>
            </div>
          </div>

          <div class="data-grid">
            <div class="data-card glass-panel fade-in-stagger-1">
              <h3><span class="icon">⚡</span> Detected Skills</h3>
              <ul class="chip-list">
                <li v-for="(skill, index) in result.detected_skills" :key="index" class="chip">{{ skill }}</li>
              </ul>
            </div>

            <div class="data-card glass-panel fade-in-stagger-2">
              <h3><span class="icon">🎯</span> Missing Keywords</h3>
              <ul class="chip-list missing">
                <li v-for="(keyword, index) in result.missing_keywords" :key="index" class="chip outline">{{ keyword }}</li>
              </ul>
            </div>
          </div>

          <div class="suggestion-card glass-panel fade-in-stagger-3">
            <h3><span class="icon">💡</span> Improvement Suggestions</h3>
            <ul v-if="getSuggestions().length > 0" class="styled-list">
              <li v-for="(suggestion, index) in getSuggestions()" :key="index">{{ suggestion }}</li>
            </ul>
            <p v-else class="empty-state">No improvement suggestions were provided by the AI.</p>
          </div>

          <div class="debug-box glass-panel fade-in-stagger-4">
            <details>
              <summary>🛠️ View Raw JSON Data (Debug)</summary>
              <pre>{{ result }}</pre>
            </details>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const selectedFile = ref(null);
const isLoading = ref(false);
const result = ref(null);
const errorMessage = ref('');

const handleFileUpload = (event) => {
  selectedFile.value = event.target.files[0];
  result.value = null;
  errorMessage.value = '';
};

const getSuggestions = () => {
  if (!result.value) return [];
  return result.value.improvement_suggestions || 
         result.value.suggestions || 
         result.value.saran || 
         result.value.saran_perbaikan || 
         [];
};

const analyzeCV = async () => {
  if (!selectedFile.value) return;

  isLoading.value = true;
  errorMessage.value = '';

  const formData = new FormData();
  formData.append('file', selectedFile.value);

  try {
    const response = await axios.post('http://localhost:8000/analyze-cv/', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    if (response.data.error) {
      errorMessage.value = response.data.error;
    } else {
      result.value = response.data;
    }
  } catch (error) {
    errorMessage.value = 'Neural network connection failed. Please check the server.';
    console.error(error);
  } finally {
    isLoading.value = false;
  }
}
</script>

<style scoped>
/* Web3 / AI Theme Variables */
:root {
  --bg-primary: #050505;
  --bg-secondary: #0a0a0a;
  --accent-blue: #3b82f6;
  --accent-purple: #8b5cf6;
  --accent-teal: #14b8a6;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --glass-bg: rgba(20, 20, 25, 0.6);
  --glass-border: rgba(255, 255, 255, 0.08);
}

* {
  box-sizing: border-box;
}

/* Base Layout */
.ai-theme-wrapper {
  min-height: 100vh;
  background: radial-gradient(circle at top center, #1e1e2e 0%, #050505 100%);
  color: #f8fafc;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  padding: 40px 20px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

.container {
  width: 100%;
  max-width: 850px;
  padding: 40px;
  border-radius: 24px;
}

/* Glassmorphism Utilities */
.glass-panel {
  background: rgba(20, 20, 25, 0.6);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
}

.glow-blue {
  box-shadow: 0 0 40px rgba(59, 130, 246, 0.15), inset 0 0 20px rgba(59, 130, 246, 0.05);
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.glow-red {
  box-shadow: 0 0 20px rgba(239, 68, 68, 0.2);
  border: 1px solid rgba(239, 68, 68, 0.5);
}

/* Typography */
.header-section {
  text-align: center;
  margin-bottom: 40px;
}

.gradient-text {
  font-size: 2.8rem;
  font-weight: 800;
  margin-bottom: 10px;
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%, #8b5cf6 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.subtitle {
  color: #94a3b8;
  font-size: 1.1rem;
}

/* Upload Area */
.upload-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
  border: 2px dashed rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background: rgba(0, 0, 0, 0.2);
}

.upload-box:hover, .upload-box.has-file {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.05);
}

.upload-box.is-loading {
  animation: pulse-border 2s infinite;
}

.hidden-input {
  display: none;
}

.upload-label {
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.upload-icon svg {
  width: 60px;
  height: 60px;
  color: #94a3b8;
  transition: all 0.3s ease;
}

.upload-box:hover .upload-icon svg {
  color: #3b82f6;
  transform: translateY(-5px);
}

.file-name {
  font-size: 1.1rem;
  font-weight: 500;
  color: #e2e8f0;
}

/* Buttons */
.action-btn {
  margin-top: 30px;
  padding: 14px 32px;
  font-size: 1.1rem;
  font-weight: 600;
  color: white;
  background: linear-gradient(135deg, #3b82f6, #8b5cf6);
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.4);
  width: 100%;
  max-width: 300px;
}

.action-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(139, 92, 246, 0.6);
}

.action-btn:disabled {
  background: #334155;
  color: #94a3b8;
  box-shadow: none;
  cursor: not-allowed;
  transform: none;
}

.loader-text {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255,255,255,0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
}

/* Results Section */
.result-wrapper {
  margin-top: 40px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.score-card {
  text-align: center;
  padding: 30px;
  border-radius: 16px;
  background: linear-gradient(180deg, rgba(30, 41, 59, 0.8) 0%, rgba(15, 23, 42, 0.8) 100%);
}

.score-card h3 {
  margin: 0 0 15px 0;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 2px;
  font-size: 0.9rem;
}

.score-circle {
  display: inline-flex;
  align-items: baseline;
  justify-content: center;
}

.score-number {
  font-size: 4rem;
  font-weight: 800;
  background: linear-gradient(135deg, #10b981, #3b82f6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.score-max {
  font-size: 1.5rem;
  color: #64748b;
  margin-left: 5px;
}

.data-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

@media (max-width: 600px) {
  .data-grid { grid-template-columns: 1fr; }
}

.data-card, .suggestion-card, .debug-box {
  padding: 24px;
  border-radius: 16px;
}

.data-card h3, .suggestion-card h3 {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  gap: 10px;
  color: #e2e8f0;
}

/* Chips / Tags */
.chip-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding: 0;
  list-style: none;
  margin: 0;
}

.chip {
  background: rgba(59, 130, 246, 0.15);
  color: #60a5fa;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.9rem;
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.chip.outline {
  background: transparent;
  color: #f43f5e;
  border-color: rgba(244, 63, 94, 0.4);
}

/* Lists */
.styled-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.styled-list li {
  position: relative;
  padding-left: 24px;
  margin-bottom: 12px;
  line-height: 1.6;
  color: #cbd5e1;
}

.styled-list li::before {
  content: '→';
  position: absolute;
  left: 0;
  color: #8b5cf6;
  font-weight: bold;
}

.empty-state {
  color: #64748b;
  font-style: italic;
  text-align: center;
  padding: 20px 0;
}

/* Error Banner */
.error-banner {
  margin-top: 20px;
  padding: 15px;
  background: rgba(127, 29, 29, 0.2);
  color: #fca5a5;
  border-radius: 12px;
  text-align: center;
  font-weight: 500;
}

/* Debug Box */
.debug-box {
  background: rgba(0, 0, 0, 0.3);
  border-color: rgba(255, 255, 255, 0.05);
}

.debug-box summary {
  cursor: pointer;
  color: #94a3b8;
  font-family: monospace;
}

.debug-box pre {
  margin-top: 15px;
  color: #10b981;
  font-family: 'Fira Code', monospace;
  font-size: 0.85rem;
  overflow-x: auto;
}

/* Animations */
@keyframes spin {
  to { transform: rotate(360deg); }
}

@keyframes pulse-border {
  0% { border-color: rgba(59, 130, 246, 0.4); box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.2); }
  50% { border-color: rgba(139, 92, 246, 1); box-shadow: 0 0 20px 0 rgba(139, 92, 246, 0.4); }
  100% { border-color: rgba(59, 130, 246, 0.4); box-shadow: 0 0 0 0 rgba(59, 130, 246, 0); }
}

/* Vue Transitions */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active {
  transition: all 0.6s cubic-bezier(0.16, 1, 0.3, 1);
}
.slide-up-enter-from {
  opacity: 0;
  transform: translateY(30px);
}

/* Staggered Fade-ins for UI Polish */
.fade-in-stagger-1 { animation: slideUpFade 0.6s ease-out 0.1s both; }
.fade-in-stagger-2 { animation: slideUpFade 0.6s ease-out 0.2s both; }
.fade-in-stagger-3 { animation: slideUpFade 0.6s ease-out 0.3s both; }
.fade-in-stagger-4 { animation: slideUpFade 0.6s ease-out 0.4s both; }

@keyframes slideUpFade {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>