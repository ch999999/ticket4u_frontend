<template>
  <div class="chatbot-container">
    <v-card class="chatbot">
      <!-- Chat Header -->
      <v-toolbar class="chatbot-header" dark flat>
        <v-toolbar-title>Ticket4U Chatbot</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn icon @click="$emit('close')">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-toolbar>

      <!-- Chat Content -->
      <v-card-text class="chat-content" ref="chatMessagesRef">
        <div 
          v-if="chatMessages.length>0"
          v-for="(message, index) in chatMessages"
          :key="index"
          :class="message.sender === 'user' ? 'user-message' : 'bot-message'"
        >
          <span v-if="message.sender === 'user'">{{ message.text }}</span>
          <div v-else v-html="parseMarkdown(message.text)"></div>
        </div>
        <div v-else>
          <v-card-text class="text-center text-h5">Welcome! I'm the Ticket4U chatbot, here to guide you in your movie selection and booking process! </v-card-text>
        </div>
        <div v-if="querying" class="loading-message">
          <v-progress-circular indeterminate color="white" size="20"></v-progress-circular>
          <span>Typing...</span>
        </div>
      </v-card-text>

      <!-- Input Area -->
      <v-card-actions class="input-area">
        <v-text-field
          v-model="question"
          placeholder="Write a message..."
          @keyup.enter="submitQuery"
          dense
          outlined
          hide-details
        ></v-text-field>
        <v-btn
          color="white"
          @click="submitQuery"
          class="send-btn"
          :loading="querying"
          :disabled="!isValidQuestion || querying"
        >
          <v-icon icon="mdi-send" end></v-icon>
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
import { ref, watch, computed } from "vue";
import axios from "axios";
import MarkdownIt from "markdown-it";

export default {
  name: "Chatbot",
  setup() {
    const question = ref("");
    const querying = ref(false);
    const chatMessages = ref([]);
    const chatMessagesRef = ref(null);

    const API_BASE_URL = "https://vertex-api-service-1015935499035.asia-southeast1.run.app";

    const md = new MarkdownIt({
      html: true,
      linkify: true,
      typographer: true,
    }).disable("image");

    const isValidQuestion = computed(() => question.value.trim().length > 0);

    const parseMarkdown = (text) => md.render(text);

    const submitQuery = async () => {
      if (!isValidQuestion.value || querying.value) return;

      querying.value = true;
      const trimmedQuestion = question.value.trim();
      chatMessages.value.push({ sender: "user", text: trimmedQuestion });
      question.value = "";

      try {
        const response = await axios.get(`${API_BASE_URL}/query`, {
          params: { question: trimmedQuestion },
        });
        chatMessages.value.push({
          sender: "ai",
          text: response.data.answer || "No response available.",
        });
      } catch (error) {
        console.error("Error querying document:", error);
        chatMessages.value.push({
          sender: "ai",
          text: "Sorry, something went wrong. Please try again.",
        });
      } finally {
        querying.value = false;
      }
    };

    // Scroll to bottom of chat messages when a new message is added
    watch(
      chatMessages,
      () => {
        //Original code
        setTimeout(() => {
          if (chatMessagesRef.value) {
            chatMessagesRef.value.scrollTop = chatMessagesRef.value.scrollHeight;
          }
        }, 0);

        //ISaac code also no work
        // nextTick(() => {
        //   if (chatMessagesRef.value) {
        //     chatMessagesRef.value.scrollTop = chatMessagesRef.value.scrollHeight;
        //   }
        // });
      },
      { deep: true }
    );

    return {
      question,
      querying,
      chatMessages,
      chatMessagesRef,
      submitQuery,
      parseMarkdown,
      isValidQuestion,
    };
  },
};
</script>

<style scoped>
.chatbot-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  max-width: 2000px;
}

.chatbot {
  max-height: 90vh;
  max-width: 2000px;
  width: 1000px;
  height: 650px;
  min-width: 500px;
  min-height: 500px;
  resize: both;
  overflow: auto;
  border-radius: 20px;
  background: #2c2c2e;
  display: flex;
  flex-direction: column;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
  position: relative;
}

.chatbot-header {
  background: linear-gradient(45deg, #6a11cb, #2575fc);
}

.chat-content {
  padding: 20px;
  overflow-y: auto;
  flex-grow: 1;
  font-size: 1.1rem;
  background-color: #1f1f21;
}

.user-message {
  background: #6a11cb;
  color: white;
  text-align: right;
  margin: 12px 0;
  padding: 12px 16px;
  border-radius: 16px;
  align-self: flex-end;
  font-size: 1.1rem;
}

.bot-message {
  background: #3c3c3e;
  color: white;
  text-align: left;
  margin: 12px 0;
  padding: 12px 16px;
  border-radius: 16px;
  align-self: flex-start;
  font-size: 1.1rem;
}

.loading-message {
  display: flex;
  align-items: center;
  color: white;
  font-size: 1.1rem;
}

.input-area {
  display: flex;
  align-items: center;
  padding: 16px;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  background-color: #1f1f21;
}

.v-text-field {
  flex-grow: 1;
  margin-right: 12px;
  font-size: 1rem;
}

.send-btn {
  background-color: #6a11cb;
  color: white;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s ease-in-out;
}

.send-btn:hover {
  transform: scale(1.1);
}

.send-btn v-icon {
  font-size: 28px;
}
</style>
