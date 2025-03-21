<template>
    <!-- Add your template here based on your needs -->
    <!-- Example minimal template: -->
    <div id="app">
    <div class="header">
      <div class="logo">THEIA</div>
      <div class="nav">
        <a href="#">Home</a>
        <a href="#">Products</a>
        <a href="#">Pricing</a>
        <a href="#">About</a>
        <a href="#">Jobs</a>
      </div>
      <div class="buttons">
        <button class="btn btn-orange">Get Started</button>
        <button class="btn btn-green" @click="refreshData">Test Salesforce Connection</button>
      </div>
    </div>
    <div class="content">
      <!-- Job Section -->
      <div v-if="loading" class="loading">Loading data...</div>
      <div v-else>
        <!-- Main Job Card - This displays the job name from Salesforce -->
        <div class="job-card" v-if="jobs.length > 0">
          <h3>{{ jobs[0].Job_Public_Name__c }}</h3>
        </div>
        
        <!-- Internal Question Section -->
        <div class="question-section">
          <h4>Internal Question:</h4>
          <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_1__c }}</p>
          <p v-else>Loading internal question...</p>
          <div class="input-group">
            <input 
              type="text" 
              v-model="internalAnswer" 
              :placeholder="opportunityDiscussed ? (opportunityDiscussed.Internal_Answer_1__c || 'Enter your answer...') : 'Loading...'"
              :disabled="!opportunityDiscussed"
            >
            <button 
              class="btn btn-green" 
              @click="analyzeAnswerWithGemini('Internal_Q_1__c',opportunityDiscussed.Internal_Q_1__c,internalAnswer)"
              :disabled="!opportunityDiscussed"
            >
              Save Answer
            </button>
          </div>
        </div>

        <!-- Internal Question 2 Section -->
        <div class="question-section">
          <h4>Internal Question 2:</h4>
          <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_2__c }}</p>
          <p v-else>Loading internal question...</p>
          <div class="input-group">
            <input 
              type="text" 
              v-model="internalAnswer2" 
              :placeholder="opportunityDiscussed ? (opportunityDiscussed.Internal_Answer_1__c || 'Enter your answer...') : 'Loading...'"
              :disabled="!opportunityDiscussed"
            >
            <button 
              class="btn btn-green" 
              @click="analyzeAnswerWithGemini('Internal_Q_2__c',opportunityDiscussed.Internal_Q_2__c,internalAnswer2)"
              :disabled="!opportunityDiscussed"
            >
              Save Answer
            </button>
          </div>
        </div>

        <!-- Internal Question 3 Section -->
        <div class="question-section">
          <h4>Internal Question 3:</h4>
          <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_3__c }}</p>
          <p v-else>Loading internal question...</p>
          <div class="input-group">
            <input 
              type="text" 
              v-model="internalAnswer3" 
              :placeholder="opportunityDiscussed ? (opportunityDiscussed.Internal_Answer_1__c || 'Enter your answer...') : 'Loading...'"
              :disabled="!opportunityDiscussed"
            >
            <button 
              class="btn btn-green" 
              @click="analyzeAnswerWithGemini('Internal_Q_3__c',opportunityDiscussed.Internal_Q_3__c,internalAnswer3)"
              :disabled="!opportunityDiscussed"
            >
              Save Answer
            </button>
          </div>
        </div>

        <div class="question-section">
          <h4>Analysis 1:</h4>
          <p>{{ analysisInternal_Q_1__c }}</p>
          
        </div>

        <div class="question-section">
          <h4>Analysis 2:</h4>
          <p v-if="opportunityDiscussed">{{ analysisInternal_Q_2__c }}</p>
          
        </div>

        <div class="question-section">
          <h4>Analysis 3:</h4>
          <p v-if="opportunityDiscussed">{{ analysisInternal_Q_3__c }}</p>
          
        </div>

        <!-- AI Analysis Section with Tabs -->
        <div class="analysis-section">
          <div class="tabs">
            <div 
              class="tab" 
              :class="{ active: activeTab === 'analysis' }"
              @click="activeTab = 'analysis'"
            >
              AI Analysis
            </div>
            <div 
              class="tab" 
              :class="{ active: activeTab === 'prompts' }"
              @click="activeTab = 'prompts'"
            >
              Manage Prompts
            </div>
          </div>

          <!-- Analysis Tab -->
          <div class="tab-content" :class="{ active: activeTab === 'analysis' }">
            <div class="form-group">
              <label for="prompt-select">Select Prompt Template:</label>
              <select id="prompt-select" v-model="selectedPromptId" @change="onPromptSelect" required>
                <option value="">-- Select a prompt --</option>
                <option v-for="prompt in prompts" :key="prompt.id" :value="prompt.id">
                  {{ prompt.name }}
                </option>
              </select>
            </div>
            <div v-if="selectedPrompt" class="form-group">
              <label>Selected Prompt:</label>
              <p>{{ selectedPrompt.template }}</p>
            </div>
            <button 
              class="btn btn-orange" 
              @click="analyzeWithGemini"
              :disabled="!opportunityDiscussed || !jobs.length || !selectedPromptId"
            >
              Analyze with AI
            </button>
            <div v-if="analysis" class="analysis-result">
              {{ analysis }}
            </div>
            
            <!-- Next Interview Question Display Section -->
            <div v-if="selectedPromptId === 'generate_next_question' && analysis" class="next-question-section">
              <h4>Next Interview Question:</h4>
              <div class="next-question-display">
                {{ extractedQuestion() }}
              </div>
            </div>
          </div>

          <!-- Prompts Management Tab -->
          <div class="tab-content" :class="{ active: activeTab === 'prompts' }">
            <button class="btn btn-green" @click="showAddPromptModal">Add New Prompt</button>
            
            <div class="prompt-list">
              <div v-for="prompt in prompts" :key="prompt.id" class="prompt-item">
                <h4>{{ prompt.name }}</h4>
                <p>{{ prompt.template.substring(0, 100) }}{{ prompt.template.length > 100 ? '...' : '' }}</p>
                <div class="prompt-actions">
                  <button class="btn btn-blue" @click="editPrompt(prompt)">Edit</button>
                  <button 
                    class="btn btn-gray" 
                    @click="deletePrompt(prompt.id)"
                    :disabled="prompt.id === 'custom'"
                  >
                    Delete
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Add/Edit Prompt Modal -->
    <div class="modal" :style="{ display: showModal ? 'block' : 'none' }">
      <div class="modal-content">
        <div class="modal-header">
          <h3>{{ editingPrompt ? 'Edit Prompt' : 'Add New Prompt' }}</h3>
          <span class="close" @click="closeModal">&times;</span>
        </div>
        <div class="form-group">
          <label for="prompt-name">Prompt Name:</label>
          <input type="text" id="prompt-name" v-model="promptForm.name" placeholder="Enter a name for this prompt">
        </div>
        <div class="form-group">
          <label for="prompt-template">Prompt Template:</label>
          <textarea id="prompt-template" v-model="promptForm.template" placeholder="Enter the prompt template text" rows="6"></textarea>
        </div>
        <button class="btn btn-green" @click="savePrompt">Save Prompt</button>
      </div>
    </div>
  </div>

  </template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'https://unpkg.com/axios/dist/esm/axios.min.js';

const API_URL = 'http://localhost:5038/';

// Reactive state
const jobs = ref([]);
const opportunityDiscussed = ref(null);
const internalAnswer = ref('');
const internalAnswer2 = ref('');
const internalAnswer3 = ref('');
const analysisInternal_Q_1__c = ref('');
const analysisInternal_Q_2__c = ref('');
const analysisInternal_Q_3__c = ref('');
const loading = ref(true);
const analysis = ref('');
const activeTab = ref('analysis');
const prompts = ref([]);
const selectedPromptId = ref('');
const selectedPrompt = ref(null);
const showModal = ref(false);
const editingPrompt = ref(null);
const promptForm = ref({
  name: '',
  template: ''
});

// Computed property to extract the question from analysis
const extractedQuestion = () => {
  if (!analysis.value || selectedPromptId.value !== 'generate_next_question') {
    return '';
  }
  
  // Try to extract just the question part, assuming it might be prefixed with explanatory text
  const text = analysis.value;
  
  // Look for question patterns
  const questionMatches = text.match(/(?:Question:|Next Question:|Interview Question:)?\s*(.+\?)/i);
  if (questionMatches && questionMatches[1]) {
    return questionMatches[1].trim();
  }
  
  // If no question mark is found, return the whole text as the question
  return text;
};

// Methods
const getJobs = async () => {
  try {
    const response = await axios.get(`${API_URL}getJobs`);
    console.log('Jobs API response:', response.data);
    jobs.value = response.data.records || [];
    console.log('Job Public Name:', jobs.value.length > 0 ? jobs.value[0].Job_Public_Name__c : 'No jobs');
  } catch (error) {
    console.error('Error fetching jobs:', error);
    jobs.value = [];
  }
};

const getOpportunityDiscussed = async () => {
  try {
    const response = await axios.get(`${API_URL}getOpportunityDiscussed`);
    opportunityDiscussed.value = response.data.records[0] || null;
    if (opportunityDiscussed.value) {
      internalAnswer.value = opportunityDiscussed.value.Internal_Answer_1__c || '';
    }
  } catch (error) {
    console.error('Error fetching opportunity discussed:', error);
  }
};

const updateInternalAnswer = async () => {
  try {
    const response = await axios.post(`${API_URL}updateOpportunityDiscussed`, {
      answer: internalAnswer.value
    });
    console.log('Answer updated:', response.data);
    await getOpportunityDiscussed();
    alert('Answer saved successfully!');
  } catch (error) {
    console.error('Error updating answer:', error);
    alert('Error saving answer. Please try again.');
  }
};

const getPrompts = async () => {
  try {
    const response = await axios.get(`${API_URL}prompts`);
    prompts.value = response.data.prompts || [];
  } catch (error) {
    console.error('Error fetching prompts:', error);
    prompts.value = [];
  }
};

const analyzeAnswerWithGemini = async (field,question,answer) => {
  
  try {
    const response = await axios.post(`${API_URL}processInternalAnswerGemini`, {
      jobData: jobs.value[0],
      question: question,
      answer: answer,
      field:field
    });

    console.log(response);

    if (field=="Internal_Q_1__c"){
        analysisInternal_Q_1__c.value = response.data.analysis;
        updateInternalAnswer();
    }

    if (field=="Internal_Q_2__c"){
        analysisInternal_Q_2__c.value = response.data.analysis;
    }

    if (field=="Internal_Q_3__c"){
        analysisInternal_Q_3__c.value = response.data.analysis;
    }
    
  } catch (error) {
    console.error('Error analyzing with Gemini:', error);
    alert('Error analyzing with Gemini. Please try again.');
  }
};


const analyzeWithGemini = async () => {
  if (!selectedPromptId.value) {
    alert('Please select a prompt template');
    return;
  }
  try {
    // Include analysis values if using the generate_next_question prompt
    const requestData = {
      jobData: jobs.value[0],
      opportunityData: opportunityDiscussed.value,
      promptId: selectedPromptId.value
    };
    
    // Add analysis data if using the generate_next_question prompt
    if (selectedPromptId.value === 'generate_next_question') {
      requestData.analysis1 = analysisInternal_Q_1__c.value;
      requestData.analysis2 = analysisInternal_Q_2__c.value;
      requestData.analysis3 = analysisInternal_Q_3__c.value;
    }
    
    const response = await axios.post(`${API_URL}processWithGemini`, requestData);
    analysis.value = response.data.analysis;
  } catch (error) {
    console.error('Error analyzing with Gemini:', error);
    alert('Error analyzing with Gemini. Please try again.');
  }
};

const onPromptSelect = () => {
  if (selectedPromptId.value === 'custom') {
    selectedPrompt.value = null;
  } else {
    const prompt = prompts.value.find(p => p.id === selectedPromptId.value);
    selectedPrompt.value = prompt || null;
  }
};

const refreshData = async () => {
  loading.value = true;
  try {
    await Promise.all([
      getJobs(),
      getOpportunityDiscussed(),
      getPrompts()
    ]);
  } finally {
    loading.value = false;
  }
};

const showAddPromptModal = () => {
  editingPrompt.value = null;
  promptForm.value.name = '';
  promptForm.value.template = '';
  showModal.value = true;
};

const editPrompt = (prompt) => {
  editingPrompt.value = prompt;
  promptForm.value.name = prompt.name;
  promptForm.value.template = prompt.template;
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
};

const savePrompt = async () => {
  if (!promptForm.value.name || !promptForm.value.template) {
    alert('Please fill out all fields');
    return;
  }
  try {
    const url = editingPrompt.value 
      ? `${API_URL}prompts/${editingPrompt.value.id}`
      : `${API_URL}prompts`;
    const method = editingPrompt.value ? axios.put : axios.post;

    await method(url, {
      name: promptForm.value.name,
      template: promptForm.value.template
    });
    
    await getPrompts();
    closeModal();
  } catch (error) {
    console.error('Error saving prompt:', error);
    alert('Error saving prompt. Please try again.');
  }
};

const deletePrompt = async (id) => {
  if (confirm('Are you sure you want to delete this prompt?')) {
    try {
      await axios.delete(`${API_URL}prompts/${id}`);
      await getPrompts();
    } catch (error) {
      console.error('Error deleting prompt:', error);
      alert('Error deleting prompt. Please try again.');
    }
  }
};

// Lifecycle hook
onMounted(async () => {
  await refreshData();
});
</script>

<style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f9f9f9;
    }
    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px 20px;
      background-color: #fff;
      border-bottom: 1px solid #ddd;
    }
    .header .logo {
      font-size: 24px;
      font-weight: bold;
    }
    .nav {
      display: flex;
      gap: 20px;
    }
    .nav a {
      text-decoration: none;
      color: #333;
      font-weight: 500;
    }
    .nav a:hover {
      color: #007bff;
    }
    .buttons {
      display: flex;
      gap: 10px;
    }
    .btn {
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    .btn-orange {
      background-color: #ff6200;
      color: white;
    }
    .btn-green {
      background-color: #28a745;
      color: white;
    }
    .btn-blue {
      background-color: #007bff;
      color: white;
    }
    .btn-gray {
      background-color: #6c757d;
      color: white;
    }
    .content {
      max-width: 800px;
      margin: 20px auto;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }
    .card-container {
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
    .card {
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #f9f9f9;
    }
    .card h3 {
      margin: 0 0 10px;
      color: #333;
    }
    .card p {
      margin: 0;
      color: #666;
    }
    .job-card {
      margin: 15px 0;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #fff;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }
    .job-card h3 {
      margin: 0 0 10px;
      color: #333;
      text-align: center;
      font-size: 20px;
    }
    .question-section {
      margin-top: 20px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #fff;
    }
    .question-section h4 {
      margin: 0 0 10px;
      color: #333;
    }
    .input-group {
      margin-top: 10px;
    }
    .input-group input {
      width: 100%;
      padding: 8px;
      margin-bottom: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    .loading {
      text-align: center;
      padding: 20px;
      color: #666;
    }
    .analysis-section {
      margin-top: 20px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #f9f9f9;
    }
    .analysis-section textarea {
      width: 100%;
      padding: 8px;
      margin: 10px 0;
      border: 1px solid #ddd;
      border-radius: 4px;
      min-height: 100px;
    }
    .analysis-result {
      margin-top: 15px;
      padding: 15px;
      background-color: #fff;
      border: 1px solid #ddd;
      border-radius: 4px;
      white-space: pre-wrap;
    }
    .next-question-section {
      margin-top: 25px;
      padding: 20px;
      background-color: #f0f8ff;
      border: 1px solid #4682b4;
      border-radius: 8px;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
    }
    .next-question-section h4 {
      margin: 0 0 15px;
      color: #4682b4;
      font-weight: bold;
      text-align: center;
      font-size: 18px;
    }
    .next-question-display {
      padding: 15px;
      background-color: #fff;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 16px;
      font-weight: 500;
      color: #333;
      white-space: pre-wrap;
    }
    select {
      width: 100%;
      padding: 8px;
      margin-bottom: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #fff;
    }
    .tabs {
      display: flex;
      margin-bottom: 15px;
      border-bottom: 1px solid #ddd;
    }
    .tab {
      padding: 10px 15px;
      cursor: pointer;
      border-bottom: 2px solid transparent;
    }
    .tab.active {
      border-bottom: 2px solid #ff6200;
      font-weight: bold;
    }
    .tab-content {
      display: none;
    }
    .tab-content.active {
      display: block;
    }
    .prompt-list {
      margin-top: 15px;
    }
    .prompt-item {
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      margin-bottom: 10px;
      background-color: #fff;
    }
    .prompt-item h4 {
      margin: 0 0 5px;
    }
    .prompt-item p {
      margin: 0;
      color: #666;
      font-size: 14px;
    }
    .prompt-actions {
      display: flex;
      gap: 10px;
      margin-top: 10px;
    }
    .form-group {
      margin-bottom: 15px;
    }
    .form-group label {
      display: block;
      margin-bottom: 5px;
      font-weight: 500;
    }
    .modal {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.5);
      z-index: 1000;
    }
    .modal-content {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: #fff;
      padding: 20px;
      border-radius: 8px;
      width: 80%;
      max-width: 600px;
    }
    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 15px;
    }
    .modal-header h3 {
      margin: 0;
    }
    .close {
      cursor: pointer;
      font-size: 24px;
    }
  </style>