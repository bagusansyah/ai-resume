<template>
  <div class="container">
    <h1>AI Resume Analyzer</h1>
    <p>Upload your resume and let AI provide feedback like a professional recruiter.</p>

    <!-- Upload Section -->
    <div class="upload-box">
      <input type="file" @change="handleFileUpload" accept="application/pdf" />
      <button @click="analyzeCV" :disabled="!selectedFile || isLoading">
        {{ isLoading ? 'Analyzing…' : 'Analyze Resume' }}
      </button>
    </div>

    <!-- Error Message -->
    <div v-if="errorMessage" class="error">
      {{ errorMessage }}
    </div>

    <!-- Result Section -->
    <div v-if="result" class="result-box">
      <h2>AI Resume Analysis Results</h2>

      <div class="score">
        <h3>Resume Score: <span>{{ result.score }} / 100</span></h3>
      </div>

      <div class="section">
        <h3>Detected Skills</h3>
        <ul>
          <li v-for="(skill, index) in result.detected_skills" :key="index">{{ skill }}</li>
        </ul>
      </div>

      <div class="section">
        <h3>Missing Keywords</h3>
        <ul>
          <li v-for="(keyword, index) in result.missing_keywords" :key="index">{{ keyword }}</li>
        </ul>
      </div>

      <div class="section">
        <h3>Improvement Suggestions</h3>
        <ul v-if="getSuggestions().length > 0">
          <li v-for="(suggestion, index) in getSuggestions()" :key="index">{{ suggestion }}</li>
        </ul>
        <p v-else style="color: gray; font-style: italic;">No improvement suggestions were provided by the AI.</p>
      </div>

      <!-- Debugging Section (only to inspect raw AI output) -->
      <div class="section debug-box">
        <details>
          <summary>🛠️ View Raw Data from AI (Debug)</summary>
          <pre>{{ result }}</pre>
        </details>
      </div>
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
    errorMessage.value = 'An error occurred while contacting the server.';
    console.error(error);
  } finally {
    isLoading.value = false;
  }
}

</script>

<style scoped>
.container { max-width: 800px; margin: 0 auto; padding: 20px; font-family: sans-serif; }
.upload-box { margin: 20px 0; padding: 20px; border: 2px dashed #ccc; text-align: center; }
button { padding: 10px 20px; margin-top: 10px; cursor: pointer; background-color: #4CAF50; color: white; border: none; border-radius: 5px; }
button:disabled { background-color: #ccc; cursor: not-allowed; }
.result-box { margin-top: 30px; padding: 20px; border: 1px solid #eee; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.score span { font-size: 24px; font-weight: bold; color: #2196F3; }
.section { margin-top: 20px; }
.error { color: red; margin-top: 10px; }
ul { list-style-type: disc; margin-left: 20px; line-height: 1.5; }
.debug-box { margin-top: 40px; padding: 10px; background-color: #f9f9f9; border: 1px solid #ddd; border-radius: 5px; }
.debug-box summary { cursor: pointer; font-weight: bold; color: #666; }
.debug-box pre { white-space: pre-wrap; font-size: 12px; margin-top: 10px; color: #d63384;}
</style>