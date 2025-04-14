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
        
        <!-- Main content with tabs -->
        <div class="main-tabs-container">
          <div class="main-tabs">
            <div 
              class="main-tab" 
              :class="{ active: mainActiveTab === 'questions' }"
              @click="mainActiveTab = 'questions'"
            >
              Interview Questions
            </div>
            <div 
              class="main-tab" 
              :class="{ active: mainActiveTab === 'analysis' }"
              @click="mainActiveTab = 'analysis'"
            >
              Analysis
            </div>
          </div>
          
          <!-- Interview Questions Tab -->
          <div class="main-tab-content" :class="{ active: mainActiveTab === 'questions' }">
            <!-- Internal Question Section -->
            <div class="question-section">
              <h4>First Question:</h4>
              <div class="question-with-controls">
                <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_1__c }}</p>
                <p v-else>Loading question...</p>
                <button 
                  class="btn btn-audio" 
                  @click="speakText(candidateName+', '+opportunityDiscussed.Internal_Q_1__c)"
                  :disabled="!opportunityDiscussed || isGeneratingSpeech"
                  title="Read question aloud"
                >
                  <span v-if="isGeneratingSpeech">🔊 ...</span>
                  <span v-else>🔊</span>
                </button>
              </div>
              <audio ref="audioPlayer" style="display: none;"></audio>
              <div class="input-group">
                <textarea 
                  v-model="internalAnswer" 
                  :placeholder="opportunityDiscussed ? 'Share with us your best answer' : 'Loading...'"
                  :disabled="!opportunityDiscussed"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(1)"
                    :disabled="!opportunityDiscussed || isTranscribing || (isRecording && currentQuestionNumber !== 1)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 1 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 1">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="analyzeAnswerWithGemini('Internal_Q_1__c',opportunityDiscussed.Internal_Q_1__c,internalAnswer)"
                    :disabled="!opportunityDiscussed"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 1" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div>

            <!-- <div class="question-section">
              <h4>AI Question:</h4>
              <div class="question-with-controls">
                <p v-if="analysisInternal_Q_1__c">{{ analysisInternal_Q_1__c }}</p>
                <p v-else>Loading question...</p>
                <button 
                  class="btn btn-audio" 
                  @click="speakText(candidateName+', '+analysisInternal_Q_1__c)"
                  :disabled="!opportunityDiscussed || isGeneratingSpeech"
                  title="Read question aloud"
                >
                  <span v-if="isGeneratingSpeech">🔊 ...</span>
                  <span v-else>🔊</span>
                </button>
              </div>
              <audio ref="audioPlayer" style="display: none;"></audio>
              <div class="input-group">
                <textarea 
                  v-model="aianswer1" 
                  :placeholder="opportunityDiscussed ? 'Share with us your best answer' : 'Loading...'"
                  :disabled="!opportunityDiscussed"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(0)"
                    :disabled="!opportunityDiscussed || isTranscribing || (isRecording && currentQuestionNumber !== 1)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 1 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 1">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="analyzeAnswerWithGemini('OD_openai_answer_2__c',analysisInternal_Q_1__c,aianswer1)"
                    :disabled="!opportunityDiscussed"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 1" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div> -->
            

            <!-- Internal Question 2 Section -->
            <!-- <div class="question-section">
              <h4>Second Question:</h4>
              <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_2__c }}</p>
              <p v-else>Loading question...</p>
              <div class="input-group">
                <textarea 
                  v-model="internalAnswer2" 
                  :placeholder="opportunityDiscussed ? 'Share with us your best answer' : 'Loading...'"
                  :disabled="!opportunityDiscussed"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(2)"
                    :disabled="!opportunityDiscussed || isTranscribing || (isRecording && currentQuestionNumber !== 2)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 2 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 2">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="analyzeAnswerWithGemini('Internal_Q_2__c',opportunityDiscussed.Internal_Q_2__c,internalAnswer2)"
                    :disabled="!opportunityDiscussed"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 2" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div> -->

            <!-- Internal Question 3 Section -->
            <!-- <div class="question-section">
              <h4>Third Question:</h4>
              <p v-if="opportunityDiscussed">{{ opportunityDiscussed.Internal_Q_3__c }}</p>
              <p v-else>Loading question...</p>
              <div class="input-group">
                <textarea 
                  v-model="internalAnswer3" 
                  :placeholder="opportunityDiscussed ? 'Share with us your best answer' : 'Loading...'"
                  :disabled="!opportunityDiscussed"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(3)"
                    :disabled="!opportunityDiscussed || isTranscribing || (isRecording && currentQuestionNumber !== 3)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 3 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 3">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="saveThirdAnswerAndGenerateNext"
                    :disabled="!opportunityDiscussed"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 3" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div> -->

            <!-- Fourth Question Section (Generated) -->
            <div class="question-section" v-if="fourthQuestion || isGeneratingQuestion">
              <h4>Fourth Question:</h4>
              <div v-if="isGeneratingQuestion" class="generating-indicator">
                <div class="spinner"></div>
                <p>Generating follow-up question based on your answers...</p>
              </div>
              <p v-else>{{ fourthQuestion }}</p>
              <div class="input-group" v-if="!isGeneratingQuestion">
                <textarea 
                  v-model="internalAnswer4" 
                  placeholder="Share with us your best answer"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(4)"
                    :disabled="isTranscribing || (isRecording && currentQuestionNumber !== 4)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 4 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 4">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="saveFourthAnswer"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 4" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div>

            <!-- Fifth Question Section (Generated) -->
            <div class="question-section" v-if="fiftQuestion || isGeneratingQuestion">
              <h4>Fifth Question:</h4>
              <div v-if="isGeneratingQuestion" class="generating-indicator">
                <div class="spinner"></div>
                <p>Generating follow-up question based on your answers...</p>
              </div>
              <p v-else>{{ fiftQuestion }}</p>
              <div class="input-group" v-if="!isGeneratingQuestion">
                <textarea 
                  v-model="internalAnswer5" 
                  placeholder="Share with us your best answer"
                  class="expandable-textarea"
                  rows="3"
                  @input="autoGrow($event.target)"
                ></textarea>
                <div class="input-controls">
                  <button 
                    class="btn btn-recording" 
                    @click="toggleRecording(5)"
                    :disabled="isTranscribing || (isRecording && currentQuestionNumber !== 5)"
                    :class="{ 'recording': isRecording && currentQuestionNumber === 5 }"
                    title="Record your answer"
                  >
                    <span v-if="isRecording && currentQuestionNumber === 4">⏹️</span>
                    <span v-else>🎤</span>
                  </button>
                  <button 
                    class="btn btn-green" 
                    @click="saveFourthAnswer"
                  >
                    Save Answer
                  </button>
                </div>
                <div v-if="isTranscribing && currentQuestionNumber === 5" class="transcribing-indicator">
                  Converting speech to text...
                </div>
              </div>
            </div>

          </div>
          
          <!-- Analysis Tab -->
          <div class="main-tab-content" :class="{ active: mainActiveTab === 'analysis' }">
            <div class="question-section">
              <h4>First Question Analysis:</h4>
              <p>{{ analysisInternal_Q_1__c }}</p>
            </div>

            <div class="question-section">
              <h4>Second Question Analysis:</h4>
              <p v-if="opportunityDiscussed">{{ analysisInternal_Q_2__c }}</p>
            </div>

            <div class="question-section">
              <h4>Third Question Analysis:</h4>
              <p v-if="opportunityDiscussed">{{ analysisInternal_Q_3__c }}</p>
            </div>
            
            <div class="question-section" v-if="fourthQuestionAnalysis">
              <h4>Fourth Question Analysis:</h4>
              <p>{{ fourthQuestionAnalysis }}</p>
            </div>
          </div>
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


    <div class="modal recording-modal" :style="{ display: showRecordingModal ? 'block' : 'none' }">
      <div class="modal-content">
        <div class="modal-header">
          <h3 v-if="recordingModalQuestionNumber">Recording Answer for Question {{ recordingModalQuestionNumber }}</h3>
           <h3 v-else>Recording Answer</h3>
          <span class="close" @click="cancelRecordingModal">&times;</span>
        </div>
        <div class="modal-body">
          <div v-if="micAccessStatus === 'requesting'" class="status-message">
             <div class="spinner-small"></div> Requesting microphone access... Please allow access in your browser.
          </div>
          <div v-if="micAccessStatus === 'denied'" class="status-message error">
            Microphone access denied. Please check your browser's site settings and allow microphone access for this page.
          </div>
           <div v-if="micAccessStatus === 'error'" class="status-message error">
            Error accessing microphone: {{ modalErrorMessage }}
          </div>

          <div v-if="micAccessStatus === 'granted'">
            <div class="recording-indicator" :class="{ 'is-recording': isModalRecording }">
              <div class="mic-icon">🎤</div>
              <div class="pulsing-circle circle-1"></div>
              <div class="pulsing-circle circle-2"></div>
              <div class="pulsing-circle circle-3"></div>
            </div>
            <p v-if="isModalRecording" class="status-message">Recording...</p>
             <div v-else-if="isTranscribing" class="status-message">
               <div class="spinner-small"></div> Processing audio...
             </div>
            <p v-else class="status-message">Ready to record.</p>
          </div>
        </div>
        <div class="modal-footer">
           <button
             v-if="micAccessStatus === 'granted' && !isModalRecording && !isTranscribing"
             class="btn btn-green"
             @click="startRecordingInModal"
             :disabled="isTranscribing"
           >
             Start Recording
           </button>
          <button
             v-if="micAccessStatus === 'granted' && isModalRecording"
             class="btn btn-orange"
             @click="stopRecordingFromModal"
             :disabled="isTranscribing"
           >
             Stop Recording
           </button>
          <button
            class="btn btn-gray"
            @click="cancelRecordingModal"
            :disabled="isTranscribing"
          >
            Cancel
          </button>
        </div>
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
const audioPlayer = ref(null);
const isGeneratingSpeech = ref(false);
const isRecording = ref(false);
const isTranscribing = ref(false);
const mediaRecorder = ref(null);
const audioChunks = ref([]);
const currentQuestionNumber = ref(null);
const audioStream = ref(null);
const mainActiveTab = ref('questions');
const internalAnswer4 = ref('');
const fourthQuestion = ref('');
const fourthQuestionAnalysis = ref('');
const isGeneratingQuestion = ref(false);
const fiftQuestion = ref('');
const internalAnswer5 = ref('');
const showRecordingModal = ref(false);
const recordingModalQuestionNumber = ref(null); // To know which question the modal is for
const micAccessStatus = ref('idle'); // 'idle', 'requesting', 'granted', 'denied', 'error'
const isModalRecording = ref(false); // Separate state for modal's visual recording status
const modalErrorMessage = ref('');
const candidateName = ref('');
const aianswer1 = ref('');


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
      candidateName.value = response.data.records[0].TR1__Candidate__r.Name|| '';
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

// Text-to-speech function
const speakText = async (text) => {
  if (!text || isGeneratingSpeech.value) return;
  
  try {
    isGeneratingSpeech.value = true;
    
    const response = await axios.post(`${API_URL}text-to-speech`, {
      text: text,
      voice: 'coral',
      instructions: 'Speak in a clear and engaging tone.'
    });
    
    if (response.data.success && response.data.audioUrl) {
      // Play the audio
      audioPlayer.value.src = `${API_URL.replace(/\/$/, '')}${response.data.audioUrl}`;
      audioPlayer.value.onloadedmetadata = () => {
        audioPlayer.value.play();
      };
    } else {
      console.error('Failed to generate speech');
    }
  } catch (error) {
    console.error('Error generating speech:', error);
    alert('Error generating speech. Please try again.');
  } finally {
    isGeneratingSpeech.value = false;
  }
};

// Speech-to-text functions
const startRecording = async () => {
  try {
    console.log('Starting recording for question', currentQuestionNumber.value);
    
    // Request audio-only permission
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: true,
      video: false
    });
    console.log('Got audio stream');
    
    // Store stream in separate variable
    audioStream.value = stream;
    
    // Create simple MediaRecorder (browser will pick the appropriate MIME type)
    mediaRecorder.value = new MediaRecorder(stream);
    audioChunks.value = [];
    
    // Set up event handlers
    mediaRecorder.value.ondataavailable = (event) => {
      console.log('Data available from recorder, size:', event.data.size);
      if (event.data.size > 0) {
        audioChunks.value.push(event.data);
      }
    };
    
    mediaRecorder.value.onstop = async () => {
      console.log('MediaRecorder stopped event triggered');
      await transcribeAudio();
    };
    
    // Start recording with 200ms interval for chunks
    mediaRecorder.value.start(200);
    console.log('MediaRecorder started');
    isRecording.value = true;
  } catch (error) {
    console.error('Error starting recording:', error);
    alert('Unable to access microphone. Please make sure your browser has permission to use the microphone.');
    isRecording.value = false;
    currentQuestionNumber.value = null;
  }
};

// const stopRecording = () => {
//   console.log('Stopping recording...');
//   if (mediaRecorder.value && mediaRecorder.value.state !== 'inactive') {
//     mediaRecorder.value.stop();
//     console.log('MediaRecorder stopped');
    
//     // Stop all tracks to release the microphone
//     if (audioStream.value) {
//       audioStream.value.getTracks().forEach(track => {
//         track.stop();
//         console.log('Track stopped');
//       });
//       audioStream.value = null;
//     }
//   }
//   isRecording.value = false;
// };

// const transcribeAudio = async () => {
//   if (audioChunks.value.length === 0) {
//     console.error('No audio chunks to transcribe');
//     return;
//   }
  
//   try {
//     isTranscribing.value = true;
    
//     // Create audio blob - let the browser determine the best format
//     const audioBlob = new Blob(audioChunks.value);
//     console.log('Created audio blob of size:', audioBlob.size, 'bytes');
    
//     if (audioBlob.size < 100) {
//       console.error('Audio blob too small, likely no audio recorded');
//       alert('No audio was recorded. Please try again and speak clearly.');
//       isTranscribing.value = false;
//       return;
//     }
    
//     // Create form data
//     const formData = new FormData();
//     formData.append('audio', audioBlob, 'recording.webm');
    
//     console.log('Sending audio for transcription, size:', audioBlob.size);
    
//     // Send to backend with timeout
//     const response = await axios.post(`${API_URL}speech-to-text`, formData, {
//       headers: {
//         'Content-Type': 'multipart/form-data'
//       },
//       timeout: 60000 // 60 second timeout (speech processing can take time)
//     });
    
//     console.log('Transcription response:', response.data);
    
//     if (response.data.success && response.data.text) {
//       // Update the appropriate answer field based on which question was recorded
//       if (currentQuestionNumber.value === 1) {
//         internalAnswer.value = response.data.text;
//         console.log('Updated question 1 answer:', internalAnswer.value);
//       } else if (currentQuestionNumber.value === 2) {
//         internalAnswer2.value = response.data.text;
//         console.log('Updated question 2 answer:', internalAnswer2.value);
//       } else if (currentQuestionNumber.value === 3) {
//         internalAnswer3.value = response.data.text;
//         console.log('Updated question 3 answer:', internalAnswer3.value);
//       } else if (currentQuestionNumber.value === 4) {
//         internalAnswer4.value = response.data.text;
//         console.log('Updated question 4 answer:', internalAnswer4.value);
//       }
//     } else {
//       console.error('Failed to transcribe speech:', response.data);
//       alert('Failed to transcribe speech. Please try again.');
//     }
//   } catch (error) {
//     console.error('Error transcribing speech:', error);
//     alert(`Error transcribing speech: ${error.message}. Please try again.`);
//   } finally {
//     isTranscribing.value = false;
//     currentQuestionNumber.value = null;
//     audioChunks.value = [];
//   }
// };

// const toggleRecording = (questionNumber) => {
//   if (isRecording.value && currentQuestionNumber.value === questionNumber) {
//     stopRecording();
//   } else {
//     currentQuestionNumber.value = questionNumber;
//     startRecording();
//   }
// };

const toggleRecording = (questionNumber) => {
  // This function now just opens the modal
  openRecordingModal(questionNumber);
  if(questionNumber == 0){
    speakText(candidateName.value+', '+analysisInternal_Q_1__c.value);
  }else{
    speakText(candidateName.value+', '+opportunityDiscussed.value.Internal_Q_1__c);
  }
  
};


const openRecordingModal = async (questionNumber) => {
  console.log('Opening recording modal for question:', questionNumber);
  recordingModalQuestionNumber.value = questionNumber;
  currentQuestionNumber.value = questionNumber; // Keep track of the target question
  micAccessStatus.value = 'requesting';
  isModalRecording.value = false;
  modalErrorMessage.value = '';
  audioChunks.value = []; // Clear previous chunks
  showRecordingModal.value = true;

  // Try to get microphone access immediately
  try {
    console.log('Requesting microphone access...');
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: true,
      video: false
    });
    console.log('Microphone access granted.');
    audioStream.value = stream; // Store the stream
    micAccessStatus.value = 'granted';
    // Don't start recording automatically, wait for button click
  } catch (error) {
    console.error('Error getting microphone access:', error);
    micAccessStatus.value = 'denied';
     if (error.name === 'NotAllowedError' || error.name === 'PermissionDeniedError') {
        modalErrorMessage.value = "Permission denied. Please allow microphone access in browser settings.";
        micAccessStatus.value = 'denied';
     } else if (error.name === 'NotFoundError' || error.name === 'DevicesNotFoundError') {
         modalErrorMessage.value = "No microphone found. Please ensure a microphone is connected and enabled.";
         micAccessStatus.value = 'error';
     } else {
         modalErrorMessage.value = `An unexpected error occurred: ${error.name}`;
          micAccessStatus.value = 'error';
     }
     // Don't close the modal, show the error message
  }
};

const startRecordingInModal = () => {
  if (micAccessStatus.value !== 'granted' || !audioStream.value) {
    console.error('Cannot start recording: Mic access not granted or stream not available.');
    modalErrorMessage.value = "Microphone access issue. Please try cancelling and reopening the recording window.";
    micAccessStatus.value = 'error';
    return;
  }

  try {
    console.log('Starting recording from modal for question', currentQuestionNumber.value);

    // Create MediaRecorder using the existing stream
    mediaRecorder.value = new MediaRecorder(audioStream.value);
    audioChunks.value = []; // Reset chunks just in case

    mediaRecorder.value.ondataavailable = (event) => {
      console.log('Data available from recorder, size:', event.data.size);
      if (event.data.size > 0) {
        audioChunks.value.push(event.data);
      }
    };

    mediaRecorder.value.onstop = async () => {
      console.log('MediaRecorder stopped event triggered (from modal)');
      isModalRecording.value = false; // Update modal visual state
      // Transcription will handle closing the modal
      await transcribeAudio();
    };

     mediaRecorder.value.onerror = (event) => {
       console.error('MediaRecorder error:', event.error);
       modalErrorMessage.value = `Recording error: ${event.error.name}. Please try again.`;
       micAccessStatus.value = 'error'; // Indicate an error state in the modal
       isModalRecording.value = false;
       isRecording.value = false; // Also update main state
       // Optionally stop tracks here if needed
       if (audioStream.value) {
         audioStream.value.getTracks().forEach(track => track.stop());
         audioStream.value = null;
       }
    };

    mediaRecorder.value.start(200); // Start recording with chunks
    console.log('MediaRecorder started');
    isModalRecording.value = true; // Update modal visual state
    isRecording.value = true; // Update the main recording state
  } catch (error) {
    console.error('Error starting MediaRecorder:', error);
     modalErrorMessage.value = `Failed to start recording: ${error.message}.`;
     micAccessStatus.value = 'error';
     isModalRecording.value = false;
     isRecording.value = false;
  }
};


const stopRecording = () => {
  console.log('Stopping recording logic...');
  if (mediaRecorder.value && mediaRecorder.value.state !== 'inactive') {
     try {
        mediaRecorder.value.stop(); // This triggers the onstop event
        console.log('MediaRecorder stop() called.');
     } catch (error) {
       console.error("Error stopping MediaRecorder:", error);
       // Handle cases where stopping fails, maybe force close?
       isModalRecording.value = false; // Still update UI
       isRecording.value = false;
     }
  } else {
     console.log('MediaRecorder not active or not initialized.');
     isModalRecording.value = false; // Ensure UI state is correct
     isRecording.value = false;
  }

  // Stop tracks *after* recorder is stopped or if it wasn't running
  if (audioStream.value) {
    audioStream.value.getTracks().forEach(track => {
      track.stop();
      console.log('Audio track stopped');
    });
    audioStream.value = null; // Release the stream reference
  }

  // Note: isRecording/isModalRecording are set false here and/or in onstop
  // Let the onstop handler deal with transcription
  isRecording.value = false; // Set main state immediately
};

const stopRecordingFromModal = () => {
    stopRecording(); // Call the core stop logic
    // The onstop handler will manage transcription and modal closing
};


const cancelRecordingModal = () => {
  console.log('Cancelling recording modal.');
  if (isModalRecording.value) {
    stopRecording(); // Stop recording if it's active
  } else if (audioStream.value) {
     // If not recording but stream exists, stop tracks
     audioStream.value.getTracks().forEach(track => track.stop());
     audioStream.value = null;
  }

  // Reset states
  showRecordingModal.value = false;
  isModalRecording.value = false;
  isRecording.value = false; // Ensure main state is reset too
  isTranscribing.value = false; // Cancel transcription if it was somehow pending
  micAccessStatus.value = 'idle';
  recordingModalQuestionNumber.value = null;
  currentQuestionNumber.value = null; // Clear the target question
  audioChunks.value = [];
  mediaRecorder.value = null;
};


const transcribeAudio = async () => {
  // ... (keep existing initial checks for audioChunks.length)
   if (audioChunks.value.length === 0) {
    console.error('No audio chunks to transcribe');
    alert('No audio was recorded. Please try again.');
    // Ensure modal closes even if no chunks
    cancelRecordingModal(); // Use cancel to reset everything
    return;
  }

  try {
    isTranscribing.value = true; // Keep main transcribing indicator
    console.log('Modal: Transcription started.'); // Log modal context
    // ... (keep existing blob creation and size check)
     const audioBlob = new Blob(audioChunks.value); // Browser decides type
     console.log('Created audio blob of size:', audioBlob.size, 'bytes');

     if (audioBlob.size < 100) {
       console.error('Audio blob too small, likely no audio recorded');
       alert('No audio was recorded. Please try again and speak clearly.');
       isTranscribing.value = false;
       cancelRecordingModal(); // Reset and close modal
       return;
     }

    // ... (keep existing FormData creation and axios call)
     const formData = new FormData();
     // Try common extensions, 'audio/webm' is often default
     formData.append('audio', audioBlob, `recording-${Date.now()}.webm`);
     console.log('Sending audio for transcription, size:', audioBlob.size);

     const response = await axios.post(`${API_URL}speech-to-text`, formData, {
       headers: {
         'Content-Type': 'multipart/form-data'
       },
       timeout: 60000 // 60 second timeout
     });

    // ... (keep existing success/failure logic for updating textareas)
    console.log('Transcription response:', response.data);

    if (response.data.success && response.data.text) {
      const questionNum = currentQuestionNumber.value; // Use the stored question number
      if (questionNum === 1) internalAnswer.value = response.data.text;
      else if (questionNum === 2) internalAnswer2.value = response.data.text;
      else if (questionNum === 3) internalAnswer3.value = response.data.text;
      else if (questionNum === 4) internalAnswer4.value = response.data.text;
      else if (questionNum === 5) internalAnswer5.value = response.data.text; // Added Q5
      console.log(`Updated question ${questionNum} answer:`, response.data.text);
    } else {
      console.error('Failed to transcribe speech:', response.data);
      alert('Failed to transcribe speech. Please try again.');
    }

  } catch (error) {
    console.error('Error transcribing speech:', error);
    alert(`Error transcribing speech: ${error.message}. Please try again.`);
  } finally {
    isTranscribing.value = false;
    // Reset and close the modal regardless of success/failure
    cancelRecordingModal(); // Use cancel function to ensure cleanup
  }
};

// Add the autoGrow function for dynamically expanding textareas
const autoGrow = (element) => {
  element.style.height = "auto";
  element.style.height = (element.scrollHeight) + "px";
};

// New function to save third answer and generate fourth question
const saveThirdAnswerAndGenerateNext = async () => {
  // First save the third answer
  await analyzeAnswerWithGemini('Internal_Q_3__c', opportunityDiscussed.value.Internal_Q_3__c, internalAnswer3.value);
  
  // Then generate the next question
  await generateNextQuestion();
};

// Function to generate the next question
const generateNextQuestion = async () => {
  try {
    isGeneratingQuestion.value = true;
    
    // Auto-select the "Generate Next Interview Question" prompt
    const nextQuestionPrompt = prompts.value.find(p => p.id === 'generate_next_question');
    if (!nextQuestionPrompt) {
      console.error('Next question prompt not found');
      alert('Unable to generate next question. Prompt template not found.');
      return;
    }
    
    selectedPromptId.value = 'generate_next_question';
    selectedPrompt.value = nextQuestionPrompt;
    
    // Call the existing analyzeWithGemini function
    await analyzeWithGemini();
    
    // Extract the question from the analysis and set it as the fourth question
    fourthQuestion.value = extractedQuestion();
    
    // If successful, scroll to the new question
    if (fourthQuestion.value) {
      setTimeout(() => {
        const fourthQuestionElement = document.querySelector('.question-section:nth-child(4)');
        if (fourthQuestionElement) {
          fourthQuestionElement.scrollIntoView({ behavior: 'smooth' });
        }
      }, 300);
    }
  } catch (error) {
    console.error('Error generating next question:', error);
    alert('Error generating next question. Please try again.');
  } finally {
    isGeneratingQuestion.value = false;
  }
};

// Function to save the fourth answer
const saveFourthAnswer = async () => {
  try {
    // Process the fourth answer
    const response = await axios.post(`${API_URL}processInternalAnswerGemini`, {
      jobData: jobs.value[0],
      question: fourthQuestion.value,
      answer: internalAnswer4.value,
      field: 'Fourth_Question'
    });
    
    fourthQuestionAnalysis.value = response.data.analysis;
    alert('Fourth answer saved and analyzed successfully!');
  } catch (error) {
    console.error('Error analyzing fourth answer:', error);
    alert('Error analyzing fourth answer. Please try again.');
  }
};

const generateFifthQuestion  = async () => {
  try {
    // Process the fourth answer
    const response = await axios.post(`${API_URL}generateFifthQuestion`, {
      jobData: jobs.value[0],
      question: fourthQuestion.value,
      answer: internalAnswer4.value,
      analysis1: analysisInternal_Q_1__c.value,
      analysis2: analysisInternal_Q_2__c.value,
      analysis3: analysisInternal_Q_3__c.value,
      question1: internalAnswer.value, 
      question2: internalAnswer2.value,
      question3: internalAnswer3.value
    });
    
    fiftQuestion.value = response.data.analysis;
    alert('fifth question generated!');
  } catch (error) {
    console.error('Error generating fifth question:', error);
    alert('Error generating fifth question. Please try again.');
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
    .question-with-controls {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    
    .btn-audio {
      background-color: #4a90e2;
      color: white;
      border: none;
      border-radius: 50%;
      width: 36px;
      height: 36px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      margin-left: 10px;
      transition: background-color 0.3s;
    }
    
    .btn-audio:hover {
      background-color: #357ac1;
    }
    
    .btn-audio:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }
    
    .input-controls {
      display: flex;
      gap: 8px;
      margin-top: 10px;
    }
    
    .btn-recording {
      background-color: #9c27b0;
      color: white;
      border: none;
      border-radius: 50%;
      width: 36px;
      height: 36px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    
    .btn-recording:hover {
      background-color: #7b1fa2;
    }
    
    .btn-recording.recording {
      background-color: #f44336;
      animation: pulse 1.5s infinite;
    }
    
    .btn-recording:disabled {
      background-color: #ccc;
      cursor: not-allowed;
      animation: none;
    }
    
    .transcribing-indicator {
      margin-top: 8px;
      font-size: 14px;
      color: #666;
      font-style: italic;
    }
    
    @keyframes pulse {
      0% {
        transform: scale(1);
      }
      50% {
        transform: scale(1.1);
      }
      100% {
        transform: scale(1);
      }
    }
    
    /* Add styles for the expandable textarea */
    .expandable-textarea {
      width: 100%;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      resize: none;
      overflow: hidden;
      transition: height 0.2s;
      font-family: inherit;
      font-size: 14px;
      line-height: 1.5;
      max-height: 300px;
    }
    
    .expandable-textarea:focus {
      outline: none;
      border-color: #4a90e2;
      box-shadow: 0 0 3px rgba(74, 144, 226, 0.3);
    }

    /* Main tab styles */
    .main-tabs-container {
      margin-top: 20px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #fff;
      overflow: hidden;
    }
    
    .main-tabs {
      display: flex;
      background-color: #f9f9f9;
      border-bottom: 1px solid #ddd;
    }
    
    .main-tab {
      padding: 12px 20px;
      cursor: pointer;
      font-weight: 500;
      text-align: center;
      flex: 1;
      transition: background-color 0.2s;
    }
    
    .main-tab:first-child {
      border-right: 1px solid #ddd;
    }
    
    .main-tab.active {
      background-color: #fff;
      color: #ff6200;
      border-bottom: 2px solid #ff6200;
      margin-bottom: -1px;
    }
    
    .main-tab:hover:not(.active) {
      background-color: #f0f0f0;
    }
    
    .main-tab-content {
      display: none;
      padding: 20px;
    }
    
    .main-tab-content.active {
      display: block;
    }
    
    /* Question section styling inside tabs */
    .main-tab-content .question-section {
      margin-top: 0;
      margin-bottom: 20px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .main-tab-content .question-section:last-child {
      margin-bottom: 0;
    }
    
    /* Styles for the question generation spinner */
    .generating-indicator {
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 15px;
      margin: 10px 0;
    }
    
    .generating-indicator p {
      margin-top: 10px;
      color: #666;
      font-style: italic;
    }
    
    .spinner {
      width: 40px;
      height: 40px;
      border: 3px solid rgba(0, 0, 0, 0.1);
      border-radius: 50%;
      border-top-color: #ff6200;
      animation: spin 1s ease-in-out infinite;
    }
    
    @keyframes spin {
      to {
        transform: rotate(360deg);
      }
    }


    /* Basic Modal Styling (Adapt as needed) */
.modal {
  display: none; /* Hidden by default */
  position: fixed; /* Stay in place */
  z-index: 1000; /* Sit on top */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgba(0,0,0,0.6); /* Black w/ opacity */
}

.modal-content {
  background-color: #fefefe;
  /* margin: 15% auto;  */
  padding: 0; /* Remove padding, header/body/footer will have it */
  border: 1px solid #888;
  width: 80%; /* Could be more or less, depending on screen size */
  max-width: 500px; /* Maximum width */
  border-radius: 8px;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2),0 6px 20px 0 rgba(0,0,0,0.19);
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 15px 20px;
  background-color: #f1f1f1; /* Light grey header */
  border-bottom: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
  align-items: center;
   border-top-left-radius: 8px;
   border-top-right-radius: 8px;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.2em;
  color: #333;
}

.close {
  color: #aaa;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
}

.modal-body {
  padding: 20px;
  min-height: 150px; /* Ensure minimum space */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.modal-footer {
  padding: 15px 20px;
  background-color: #f1f1f1; /* Light grey footer */
  border-top: 1px solid #ddd;
  text-align: right;
   border-bottom-left-radius: 8px;
   border-bottom-right-radius: 8px;
}

.modal-footer .btn {
  margin-left: 10px;
}

/* Recording Indicator Styles */
.recording-indicator {
  position: relative;
  width: 80px;
  height: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px; /* Space below indicator */
}

.mic-icon {
  font-size: 30px;
  z-index: 5; /* Keep mic icon on top */
}

.pulsing-circle {
  position: absolute;
  border: 2px solid #4CAF50; /* Green color */
  border-radius: 50%;
  opacity: 0; /* Start hidden */
  transform: scale(0.8);
  z-index: 1; /* Behind mic icon */
}

/* Animation for pulsing circles */
@keyframes pulse {
  0% {
    transform: scale(0.8);
    opacity: 0.6;
  }
  50% {
     opacity: 0.1;
  }
  100% {
    transform: scale(1.8); /* Grow larger */
    opacity: 0;
  }
}

/* Apply animation when recording */
.recording-indicator.is-recording .pulsing-circle {
  animation: pulse 2s infinite ease-out;
}

.recording-indicator.is-recording .circle-1 {
  width: 100%;
  height: 100%;
}

.recording-indicator.is-recording .circle-2 {
  width: 100%;
  height: 100%;
  animation-delay: 0.5s; /* Stagger the animations */
}

.recording-indicator.is-recording .circle-3 {
  width: 100%;
  height: 100%;
  animation-delay: 1s; /* Stagger the animations */
}

.status-message {
  margin-top: 10px;
  color: #555;
  font-style: italic;
  display: flex;
  align-items: center;
  justify-content: center;
}
.status-message.error {
  color: #d9534f; /* Red for errors */
  font-weight: bold;
  font-style: normal;
}

/* Small spinner for requesting/processing */
.spinner-small {
  border: 3px solid #f3f3f3; /* Light grey */
  border-top: 3px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 16px;
  height: 16px;
  animation: spin 1s linear infinite;
  margin-right: 8px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Ensure modal buttons are spaced */
.modal-footer .btn + .btn {
    margin-left: 10px;
}
</style>