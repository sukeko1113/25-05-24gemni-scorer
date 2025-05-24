<script setup>
import { ref } from 'vue';

const impressionText = ref('');
const score = ref(null);
const feedback = ref('');
const isLoading = ref(false);
const errorMessage = ref('');

// App.vue の <script setup> 内
// ... (他のimport文)
// import { GoogleGenerativeAI } from "@google/generative-ai"; // npm install @google/generative-ai

// ... (他のref)
// const API_KEY = "YOUR_GEMINI_API_KEY"; // ここに実際のAPIキーを設定 (非推奨)

async function scoreImpression() {
  // ... (入力チェックなど)
  isLoading.value = true;
  // ...

  // GoogleGenerativeAI のインスタンスを作成 (APIキーを環境変数などから読み込むのが望ましい)
  // const genAI = new GoogleGenerativeAI(import.meta.env.VITE_GEMINI_API_KEY);
  // .env ファイルに VITE_GEMINI_API_KEY="YOUR_API_KEY" を記述し、
  // npm install @google/generative-ai を実行してください。

  // ★★★ 重要: APIキーの取り扱い ★★★
  // フロントエンドにAPIキーを直接記述するのはセキュリティリスクがあります。
  // 小規模なテストや学習目的以外では、バックエンド経由での呼び出しを検討してください。
  // ここではデモのため、直接記述する形に近い方法を示しますが、
  // Viteの環境変数 (import.meta.env.VITE_GEMINI_API_KEY) の利用を推奨します。
  // .env ファイルを作成し、 VITE_GEMINI_API_KEY=あなたのAPIキー を記述してください。

  const apiKeyFromEnv = import.meta.env.VITE_GEMINI_API_KEY;
  if (!apiKeyFromEnv) {
      errorMessage.value = 'APIキーが設定されていません。';
      isLoading.value = false;
      return;
  }
  const genAI = new GoogleGenerativeAI(apiKeyFromEnv);

  try {
    const model = genAI.getGenerativeModel({ model: "gemini-pro" }); // または適切なモデル

    const prompt = `
      以下の感想文を評価し、採点してください。
      評価基準は以下の通りです。
      - 表現力 (0-30点)
      - 内容の深さ (0-40点)
      - 独自性 (0-30点)
      合計100点満点で採点し、具体的なフィードバックも提供してください。

      出力形式は以下のJSON形式でお願いします。
      {
        "score": <合計点数>,
        "feedback": "<具体的なフィードバック>"
      }

      感想文:
      ${impressionText.value}
    `;

    const result = await model.generateContent(prompt);
    const response = await result.response;
    const text = response.text();

    console.log("Gemini API Response:", text); // デバッグ用

    // JSON形式のレスポンスをパース
    // APIからの出力が常に期待通りとは限らないため、エラーハンドリングをしっかり行う
    let parsedResponse;
    try {
        // APIのレスポンスが ```json ... ``` のようにマークダウンで囲まれている場合を考慮
        const jsonMatch = text.match(/```json\s*([\s\S]*?)\s*```/);
        if (jsonMatch && jsonMatch[1]) {
            parsedResponse = JSON.parse(jsonMatch[1]);
        } else {
            // そのままJSONとしてパースしてみる
            parsedResponse = JSON.parse(text);
        }
    } catch (e) {
        console.error("Failed to parse Gemini API response:", e);
        errorMessage.value = "APIからの応答の解析に失敗しました。形式が正しくない可能性があります。";
        isLoading.value = false;
        return;
    }


    if (parsedResponse && typeof parsedResponse.score === 'number' && typeof parsedResponse.feedback === 'string') {
      score.value = parsedResponse.score;
      feedback.value = parsedResponse.feedback;
    } else {
      throw new Error("Invalid response format from API");
    }

  } catch (error) {
    console.error("Error scoring impression with Gemini API:", error);
    errorMessage.value = 'Gemini APIとの通信中にエラーが発生しました。';
    // エラーオブジェクトに詳細が含まれている場合、それを表示することも検討
    if (error.message) {
        errorMessage.value += ` (${error.message})`;
    }
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