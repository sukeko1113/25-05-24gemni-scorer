<script setup>
import { ref } from 'vue';
// GoogleGenerativeAI を @google/generative-ai からインポートします
import { GoogleGenerativeAI } from "@google/generative-ai";

const impressionText = ref('');
const score = ref(null);
const feedback = ref('');
const isLoading = ref(false);
const errorMessage = ref('');

async function scoreImpression() {
  // 入力チェック
  if (!impressionText.value.trim()) {
    errorMessage.value = '感想文を入力してください。';
    return;
  }

  isLoading.value = true;
  errorMessage.value = ''; // エラーメッセージをリセット
  score.value = null;      // 結果をリセット
  feedback.value = '';     // フィードバックをリセット

  const apiKeyFromEnv = import.meta.env.VITE_GEMINI_API_KEY;
  if (!apiKeyFromEnv) {
    errorMessage.value = 'APIキーが設定されていません。.envファイルを確認してください。';
    isLoading.value = false;
    return;
  }

  // GoogleGenerativeAI のインスタンスを作成
  const genAI = new GoogleGenerativeAI(apiKeyFromEnv);

  try {
    // モデル名を "gemini-1.5-flash-latest" に変更
    const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash-latest" });

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

    console.log("Gemini API Response:", text); // デバッグ用にAPIの生レスポンスをコンソールに出力

    let parsedResponse;
    try {
      // APIレスポンスが ```json ... ``` のようにマークダウンで囲まれている場合を考慮
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
      // APIからのレスポンスの形式が期待通りでなかった場合
      console.error("Invalid response format from API:", parsedResponse);
      errorMessage.value = "APIから予期しない形式の応答がありました。";
      // throw new Error("Invalid response format from API"); // エラーを投げる代わりにメッセージ表示
    }

  } catch (error) {
    console.error("Error scoring impression with Gemini API:", error);
    errorMessage.value = 'Gemini APIとの通信中にエラーが発生しました。';
    if (error.message) {
      // エラーメッセージに詳細が含まれている場合、それを表示
      // GoogleGenerativeAIFetchError の場合、error.message にはAPIからのエラー詳細が含まれることが多い
      errorMessage.value += ` (${error.message})`;
    }
  } finally {
    isLoading.value = false;
  }
}
</script>

<template>
  <div class="container mx-auto p-4 pt-6 md:p-6 lg:p-12">
    <div class="max-w-2xl mx-auto bg-white shadow-lg rounded-lg p-6">
      <header class="mb-6 text-center">
        <h1 class="text-3xl font-bold text-gray-800">📝 感想文採点アプリ</h1>
        <p class="text-sm text-gray-500">25-05-24gemni-scorer</p>
      </header>

      <div class="space-y-4">
        <textarea
          v-model="impressionText"
          placeholder="ここに感想文を入力してください..."
          rows="10"
          class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 ease-in-out text-gray-700"
          :disabled="isLoading"
        ></textarea>

        <button
          @click="scoreImpression"
          :disabled="isLoading"
          class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition duration-150 ease-in-out disabled:opacity-50 disabled:cursor-not-allowed"
        >
          <span v-if="isLoading">
            <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white inline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            採点中...
          </span>
          <span v-else>採点する</span>
        </button>
      </div>

      <div v-if="errorMessage" class="mt-6 p-4 bg-red-100 border border-red-400 text-red-700 rounded-md">
        <p><strong>エラー:</strong> {{ errorMessage }}</p>
      </div>

      <div v-if="score !== null && !errorMessage" class="mt-6 p-6 bg-green-50 border border-green-300 rounded-lg shadow">
        <h2 class="text-2xl font-semibold text-gray-800 mb-3">採点結果</h2>
        <div class="bg-white p-4 rounded-md shadow-sm border border-gray-200">
            <p class="text-xl"><strong>点数:</strong> <span class="font-bold text-indigo-600">{{ score }}</span> 点</p>
            <p class="mt-2"><strong>フィードバック:</strong></p>
            <p class="text-gray-700 whitespace-pre-wrap">{{ feedback }}</p>
        </div>
      </div>
    </div>
     <footer class="text-center mt-8 text-sm text-gray-500">
      <p>Gemini APIを利用した感想文採点デモ</p>
    </footer>
  </div>
</template>

<style scoped>
/* Tailwind CSS を利用するため、<style scoped> 内のカスタムCSSは最小限に */
.whitespace-pre-wrap {
  white-space: pre-wrap; /* 改行やスペースを保持しつつ、必要に応じて折り返す */
}
/* より洗練されたUIのために、グローバルCSSやTailwindのユーティリティクラスを積極的に使用することを推奨します。 */
/* この例ではTailwindをCDN経由で読み込んでいることを前提としています。 */
/* public/index.html に <script src="https://cdn.tailwindcss.com"></script> を追加してください。 */
</style>
