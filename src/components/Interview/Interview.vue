<template>
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
        <button class="btn btn-green">Test Salesforce Connection</button>
      </div>
    </div>
    <div class="content">
      <div class="speech-controls">
        <button class="btn btn-mic" @click="startSpeechFlow">
          🎤
        </button>
      </div>

    </div>
  </div>

</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'https://unpkg.com/axios/dist/esm/axios.min.js';
import './interview.css';

const API_URL = 'http://localhost:5038/';
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


const recognizing = ref(false);
const agentResponse = ref('');
const userTranscript = ref('');


const synth = window.speechSynthesis;
let recognition;



const refreshData = async () => {
  loading.value = true;
  try {
    await Promise.all([
      getJobs(),
      getOpportunityDiscussed(),
      
    ]);
  } finally {
    loading.value = false;
  }
};

const getJobs = async () => {
  try {
    const response = await axios.get(`${API_URL}getJobs`);
    console.log('Jobs API response:', response.data);
    jobs.value = response.data.records || [];
    console.log('Job Public Name:', jobs.value.length > 0 ? jobs.value[0].Job_Public_Name__c : 'No jobs');
  } catch (error) {
    console.log('Error fetching jobs:', error);
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
    console.log('Error fetching opportunity discussed:', error);
  }
};

const speak = (text) => {
  return new Promise((resolve) => {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.onend = resolve;
    synth.speak(utterance);
  });
};

const initSpeechRecognition = () => {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  const recog = new SpeechRecognition();
  recog.lang = 'en-US';
  recog.interimResults = false;
  recog.maxAlternatives = 1;
  return recog;
};

const recordUserResponse = () => {
  return new Promise((resolve, reject) => {
    recognition = initSpeechRecognition();
    recognition.onstart = () => recognizing.value = true;

    recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript;
      userTranscript.value = transcript;
      resolve(transcript);
    };

    recognition.onerror = (event) => {
      console.log('Speech recognition error:', event.error);
      reject(event.error);
    };

    recognition.onend = () => recognizing.value = false;
    recognition.start();
  });
};

// Replace these with real logic as needed

const processUserAnswer = async (answer) => {
  
  try {
   
    return await analyzeAnswerWithGemini('Internal_Q_1__c',opportunityDiscussed.value.Internal_Answer_1__c,answer);
  } catch (error) {
    //console.error('Error analyzing with Gemini:', error);
    return `There was an error procesing your answer`;
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
        updateInternalAnswer(answer);
    }

    if (field=="Internal_Q_2__c"){
        analysisInternal_Q_2__c.value = response.data.analysis;
    }

    if (field=="Internal_Q_3__c"){
        analysisInternal_Q_3__c.value = response.data.analysis;
    }

    return response.data.analysis;
    
  } catch (error) {
    console.log('Error analyzing with Gemini:', error);
    alert('Error analyzing with Gemini. Please try again.');
  }
};

const updateInternalAnswer = async (answer) => {
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


const processAgentResponse = async (userText) => {
  return `Here's a follow-up question based on "${userText}"`;
};

const startSpeechFlow = async () => {
  try {
    const question = opportunityDiscussed.value?.Internal_Q_1__c;
    if (!question) return alert('No question available.');

    await speak(question); // Speak question
    const userInput = await recordUserResponse(); // Record user answer
    const processedUserInput = await processUserAnswer(userInput); // Process input
    const agentReply = await processAgentResponse(processedUserInput); // Get agent reply

    agentResponse.value = agentReply;
    await speak(agentReply); // Speak reply
  } catch (error) {
    console.log('Speech flow error:', error);
  }
};

onMounted(async () => {
  await refreshData();
});
</script>

<style>
</style>