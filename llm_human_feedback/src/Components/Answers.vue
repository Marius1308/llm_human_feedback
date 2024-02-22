<script setup lang="ts">
import chats from "./chats.json";
import { computed, ref, watch } from "vue";
import Context from "./Components/Context.vue";
import Chat from "./Components/Chat.vue";

const chatIndex = ref(0);
const pageIndex = ref(0);

const currentScores = ref([
  [0, 0],
  [0, 0],
]);

const currentQaPair = computed(() => {
  return chats[chatIndex.value]["qaPairs"][pageIndex.value];
});

const currentChat = computed(() => {
  const chat = chats[chatIndex.value]["qaPairs"];

  const messages: string[] = [];

  chat.forEach((pair) => {
    messages.push(`S: ${pair.question}`);
    messages.push(`A: ${pair.answer.map((answer) => answer.text)}`);
  });

  return messages;
});

const nextPage = () => {
  if (chats[chatIndex.value].qaPairs.length > pageIndex.value + 1) {
    pageIndex.value += 1;
  } else if (chats.length > chatIndex.value + 1) {
    chatIndex.value += 1;
  }
};

watch(
  currentQaPair,
  () => {
    currentScores.value = [];

    for (let i = 0; i < currentQaPair.value["answer"].length; i++) {
      currentScores.value.push([0, 0]);
    }
  },
  { immediate: true }
);

const evalFilled = computed(() => {
  return currentScores.value.every((score) => {
    console.log(score[0] > 0 && score[1] > 0);
    return score[0] > 0 && score[1] > 0;
  });
});

const contextLarge = ref(true);
</script>

<template>
  <div class="answer">
    <div
      class="sentence-row"
      v-for="(sentence, index) in currentQaPair['answer']"
      :key="index"
    >
      <div class="sentence">
        {{ sentence["text"] }}
      </div>
      <div class="score" :data-color="currentScores[index][0]">
        <span class="scoreOption">
          <label>Belegt</label>
          <input
            type="radio"
            :name="`trust-${index}`"
            :value="5"
            v-model="currentScores[index][0]"
          />
        </span>
        <span class="scoreOption">
          <label>Teilweise belegt</label>
          <input
            type="radio"
            :name="`trust-${index}`"
            :value="4"
            v-model="currentScores[index][0]"
          />
        </span>
        <span class="scoreOption">
          <label>Allgemeinwissen</label>
          <input
            type="radio"
            :name="`trust-${index}`"
            :value="3"
            v-model="currentScores[index][0]"
          />
        </span>
        <span class="scoreOption">
          <label>Falschaussage</label>
          <input
            type="radio"
            :name="`trust-${index}`"
            :value="2"
            v-model="currentScores[index][0]"
          />
        </span>
        <span class="scoreOption">
          <label>Quatsch</label>
          <input
            type="radio"
            :name="`trust-${index}`"
            :value="1"
            v-model="currentScores[index][0]"
          />
        </span>
      </div>
      <div class="score" :data-color="currentScores[index][1]">
        <span class="scoreOption">
          <label>Hilfreich</label>
          <input
            type="radio"
            :name="`helpfulness-${index}`"
            :value="5"
            v-model="currentScores[index][1]"
          />
        </span>
        <span class="scoreOption">
          <label>Eingeschränk</label>
          <input
            type="radio"
            :name="`helpfulness-${index}`"
            :value="4"
            v-model="currentScores[index][1]"
          />
        </span>
        <span class="scoreOption">
          <label>Unklar</label>
          <input
            type="radio"
            :name="`helpfulness-${index}`"
            :value="3"
            v-model="currentScores[index][1]"
          />
        </span>
        <span class="scoreOption">
          <label>Unhilfreich</label>
          <input
            type="radio"
            :name="`helpfulness-${index}`"
            :value="2"
            v-model="currentScores[index][1]"
          />
        </span>
        <span class="scoreOption">
          <label>Unsinn</label>
          <input
            type="radio"
            :name="`helpfulness-${index}`"
            :value="1"
            v-model="currentScores[index][1]"
          />
        </span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.answer {
  grid-area: answer;
  background-color: #f0f0f0;
  padding: 20px;
  font-size: 16px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  overflow-y: auto;
}

.sentence-row {
  width: 100%;
  display: flex;
  gap: 10px;
}

.sentence {
  font-size: 20px;
  flex: 1;
  background-color: gray;
  padding: 10px;
}

.score {
  font-size: 12px;
  flex: 0;
  background-color: gray;
  padding: 10px;

  &[data-color="1"] {
    background-color: red;
  }
  &[data-color="2"] {
    background-color: orange;
  }
  &[data-color="3"] {
    background-color: yellow;
  }
  &[data-color="4"] {
    background-color: lightgreen;
  }
  &[data-color="5"] {
    background-color: green;
  }
}

.scoreOption {
  display: flex;
  gap: 5px;

  & > label {
    flex: 1;
  }
}
</style>
