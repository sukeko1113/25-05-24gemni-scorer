<script setup>
import { ref } from 'vue';

const impressionText = ref('');
const score = ref(null);
const feedback = ref('');
const isLoading = ref(false);
const errorMessage = ref('');

async function scoreImpression() {
  if (!impressionText.value.trim()) {
    errorMessage.value = '感想文を入力してください。';
    return;
  }

  isLoading.value = true;
  errorMessage.value = '';
  score.value = null;
  feedback.value = '';

  try {
    // --- ここにGemini API呼び出し処理を実装 ---
    // ダミーの採点ロジック (API連携前にテスト用)
    await new Promise(resolve => setTimeout(resolve, 1500)); // 擬似的なAPI遅延
    const dummyScore = Math.floor(Math.random() * 101);
    const dummyFeedback = "素晴らしい感想文です！改善点としては、もう少し具体的なエピソードを加えるとより良くなるでしょう。";

    score.value = dummyScore;
    feedback.value = dummyFeedback;
    // --- Gemini APIからのレスポンスで上記を更新 ---

  } catch (error) {
    console.error("Error scoring impression:", error);
    errorMessage.value = '採点中にエラーが発生しました。もう一度お試しください。';
  } finally {
    isLoading.value = false;
  }
}
</script>

<template>
  <div class="container">
    <h1>📝 感想文採点アプリ (25-05-24gemni-scorer)</h1>

    <textarea
      v-model="impressionText"
      placeholder="ここに感想文を入力してください..."
      rows="10"
      :disabled="isLoading"
    ></textarea>

    <button @click="scoreImpression" :disabled="isLoading">
      {{ isLoading ? '採点中...' : '採点する' }}
    </button>

    <div v-if="errorMessage" class="error-message">
      <p>{{ errorMessage }}</p>
    </div>

    <div v-if="score !== null" class="results">
      <h2>採点結果</h2>
      <p><strong>点数:</strong> {{ score }} 点</p>
      <p><strong>フィードバック:</strong></p>
      <p>{{ feedback }}</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 1rem;
  font-family: sans-serif;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
textarea {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  padding: 0.75rem 1.5rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
button:hover:not(:disabled) {
  background-color: #369f73;
}
.results {
  margin-top: 1rem;
  padding: 1rem;
  border: 1px solid #eee;
  border-radius: 4px;
  background-color: #f9f9f9;
}
.error-message {
  color: red;
  margin-top: 0.5rem;
}
</style>