<script setup lang="ts">
import chatsM1 from "./GeneratedAnswers_test_GPT-3.5 + SimpleDbRetriever + Advanced System Message.json";
import chatsM2 from "./GeneratedAnswers_test_GPT-3.5 + SubquestionRetriever.json";
import chatsM3 from "./GeneratedAnswers_test_GPT-3.5 Only.json";
import chatsM4 from "./GeneratedAnswers_test_Llama-2 Only.json";
import chatsM5 from "./GeneratedAnswers_test_Llama-2 + SimpleDbRetriever + Advanced System Message.json";
import chatsM6 from "./GeneratedAnswers_test_Llama-2-ft + SimpleDbRetriever + Advanced System Message_filtered.json";
import chatsM7 from "./GeneratedAnswers_test_Llama-2-ft Only_filtered.json";
import { computed, ref, watch } from "vue";
import Context from "./Components/Context.vue";
import Chat from "./Components/Chat.vue";
import { saveAs } from "file-saver";

const MODELCOUNT = 7;

const allChats = [
  chatsM1,
  chatsM2,
  chatsM3,
  chatsM4,
  chatsM5,
  chatsM6,
  chatsM7,
];

const chatReduction = 2;

const chats = allChats.map((modelChats) =>
  modelChats.filter((_, index) => index % chatReduction === 0)
);

const modelWithChatContext = chats[0];

let sortedChats = chats.sort(() => 0.5 - Math.random());
const chatIndex = ref(0);
const modelIndex = ref(0);
const pageIndex = ref(0);

const progress = computed(() => {
  const totalChatCount = sortedChats[0].length;

  return (
    "Chat: " +
    (chatIndex.value + 1) +
    " / " +
    totalChatCount / chatReduction +
    " | " +
    "Frage: " +
    (pageIndex.value + 1) +
    " / " +
    sortedChats[modelIndex.value][chatIndex.value].qaPairs.length +
    " | " +
    "Model: " +
    (modelIndex.value + 1) +
    " / " +
    MODELCOUNT
  );
});

const showHelp = ref(true);

const initLocalStorage = () => {
  const chatsString = localStorage.getItem("chats");
  if (chatsString) {
    sortedChats = JSON.parse(chatsString).sort(() => 0.5 - Math.random());
  } else {
    localStorage.setItem("chats", JSON.stringify(chats));
  }
};
initLocalStorage();

const currentScores = ref([
  [0, 0],
  [0, 0],
]);

const currentQaPair = computed(() => {
  return sortedChats[modelIndex.value][chatIndex.value].qaPairs[
    pageIndex.value
  ];
});

const currentContext = computed(() => {
  console.log(
    sortedChats[modelIndex.value][chatIndex.value].qaPairs[pageIndex.value]
      .context
  );
  if (
    !sortedChats[modelIndex.value][chatIndex.value].qaPairs[pageIndex.value]
      .context
  ) {
    return modelWithChatContext[chatIndex.value].qaPairs[pageIndex.value]
      .context;
  }
  return sortedChats[modelIndex.value][chatIndex.value].qaPairs[pageIndex.value]
    .context;
});

const currentChat = computed(() => {
  const chat = sortedChats[modelIndex.value][chatIndex.value].qaPairs;

  const messages: string[] = [];

  chat.forEach((pair) => {
    messages.push(`S: ${pair.question}`);
    messages.push(`A: ${pair.answer.map((answer) => answer.text)}`);
  });

  return messages;
});

const download = () => {
  const exportData = JSON.stringify(sortedChats);
  const blob = new Blob([exportData], { type: "text/plain;charset=utf-8" });
  saveAs(blob, "chats.json");
};

const saveScores = () => {
  currentScores.value.forEach((score, index) => {
    sortedChats[modelIndex.value][chatIndex.value].qaPairs[
      pageIndex.value
    ].answer[index].trust = score[0];
    sortedChats[modelIndex.value][chatIndex.value].qaPairs[
      pageIndex.value
    ].answer[index].helpfulness = score[1];
  });

  const exportData = JSON.stringify(sortedChats);
  localStorage.setItem("chats", exportData);
};

const nextPage = () => {
  saveScores();
  console.log(
    modelIndex.value,
    MODELCOUNT - 1,
    modelIndex.value < MODELCOUNT - 1
  );
  if (modelIndex.value < MODELCOUNT - 1) {
    modelIndex.value += 1;
    return;
  }
  modelIndex.value = 0;
  sortedChats = sortedChats.sort(() => 0.5 - Math.random());

  if (
    sortedChats[modelIndex.value][chatIndex.value].qaPairs.length >
    pageIndex.value + 1
  ) {
    pageIndex.value += 1;
    return;
  }
  pageIndex.value = 0;
  console.log(sortedChats[modelIndex.value].length, chatIndex.value + 1);
  if (sortedChats[modelIndex.value].length > chatIndex.value + 1) {
    chatIndex.value += 1;
    return;
  }
  chatIndex.value = 0;
};

const previusPage = () => {
  saveScores();
  if (modelIndex.value > 0) {
    modelIndex.value -= 1;
    return;
  }
  if (pageIndex.value > 0) {
    pageIndex.value -= 1;
    modelIndex.value = MODELCOUNT - 1;
    return;
  }
  if (chatIndex.value > 0) {
    chatIndex.value -= 1;
    modelIndex.value = MODELCOUNT - 1;
    pageIndex.value =
      sortedChats[modelIndex.value][chatIndex.value - 1].qaPairs.length - 1;
    return;
  }
};

watch(
  currentQaPair,
  () => {
    currentScores.value = [];

    currentQaPair.value["answer"].forEach((answer) => {
      currentScores.value.push([answer.trust, answer.helpfulness]);
    });
  },
  { immediate: true }
);

const evalFilled = computed(() => {
  return currentScores.value.every((score) => {
    return score[0] > 0 && score[1] > 0;
  });
});

const nextEmptyScore = computed(() => {
  for (let i = 0; i < currentScores.value.length; i++) {
    if (currentScores.value[i][0] === -1) {
      return { sentence: i, category: 0 };
    }
    if (currentScores.value[i][1] === -1) {
      return { sentence: i, category: 1 };
    }
  }
  return { sentence: -1, category: -1 };
});

addEventListener("keydown", (event) => {
  const score = parseInt(event.key);
  if (isNaN(score) || score > 5 || score < 1) return;
  currentScores.value[nextEmptyScore.value.sentence][
    nextEmptyScore.value.category
  ] = score;
});

const contextLarge = ref(true);
</script>

<template>
  <div class="screen">
    <div class="left">
      <Chat
        :current-chat="currentChat"
        :extended="!contextLarge"
        @extend="contextLarge = false"
      />
      <Context
        :text="currentContext"
        :extended="contextLarge"
        @extend="contextLarge = true"
      />
    </div>
    <div class="question">
      {{ currentQaPair.question }}
    </div>
    <div class="progress">{{ progress }}</div>
    <div class="answer">
      <div
        class="sentence-row"
        v-for="(sentence, index) in currentQaPair['answer']"
        :key="index"
      >
        <div class="sentence" :data-next="nextEmptyScore.sentence === index">
          {{ sentence["text"] }}
        </div>
        <div
          class="score"
          :data-color="currentScores[index][0]"
          :data-next="
            nextEmptyScore.sentence === index && nextEmptyScore.category === 0
          "
        >
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
        <div
          class="score"
          :data-color="currentScores[index][1]"
          :data-next="
            nextEmptyScore.sentence === index && nextEmptyScore.category === 1
          "
        >
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
            <label>Wiederholung</label>
            <input
              type="radio"
              :name="`helpfulness-${index}`"
              :value="2"
              v-model="currentScores[index][1]"
            />
          </span>
          <span class="scoreOption">
            <label>Unhilfreich</label>
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

    <div class="navigation">
      <button class="close-help" @click="showHelp = true">Hilfe</button>
      <button class="close-help" @click="download">Download Results</button>
      <div>
        <button @click="previusPage">
          {{ "< Prev" }}
        </button>
        <button :disabled="!evalFilled" @click="nextPage">Next ></button>
      </div>
    </div>
  </div>
  <div v-if="showHelp" class="help-background"></div>
  <div v-if="showHelp" class="help">
    <div>
      Erst einmal danke, dass du uns bei der Evaluation unseres Sprachmodells
      hilst. Hier eine kurze Einführung in die Evaluation mit diesem Tool. Du
      wirst nacheinander verschiedene Fragen sehen und dazu jeweils einen
      Antwort von einem Modell. Es wird verschiedene Antworten zu jeder Frage
      geben, die du nacheinander bewerten sollst. Auf der linken Seite siehst du
      hilfreiche Informationen zum Kontext der Frage und auf der rechten Seite
      die Antwort des Modells. Die Antworten sind in Sätze aufgeteilt, die du
      einzeln bewerten sollst. Jeder Satz hat zwei Bewertungen, eine für den
      Trust und einen für die Helpfullness. Bewerte jeden Satz dabei bitte
      unabhängig von den anderen Sätzen. Eine Ausnahme besteht wenn ein Satz von
      dem Tool auf eine merkwürdige Art getrennt wurde oder wenn der Sinn oder
      die Quelle erst in einem späteren Satz deutlich wird. Dann können diese
      Aspekte in die Bewertung einfließen. Beispiele: Ein Satz besteht nur aus
      "a." und der nächste Satz enthält einen Stichpunkt. Dann gehört "a." zu
      dem Stichpunkt und kann die selbe Bewertung bekommen. Oder ein Satz
      enthält keine Quelle, aber später am Ende des Abschnittes oder am Ende der
      Antwort gibt es eine Quellenangabe, dann kann auch schon ein früherer Satz
      als Belegt gewertet werden. Um einen Satz zu bewerten, klicke auf die
      entsprechende Zahl auf der rechten Seite oder drücke die entsprechende
      Zahl auf deiner Tastatur. Hast du alle Sätze bewertet, kannst du unten auf
      "Next >" klicken, um zur nächsten Frage zu kommen. Dabei werden deine
      Bewertungen automatisch gespeichert. Solltest du dich vertippt haben,
      kannst du die Bewertung eines Satzes auch ändern, indem du auf die
      entsprechende Bewertung klickst. Solltest du Fragen haben, kannst du dich
      gerne an uns wenden. Nun folgen kurze Erklärungen zu den einzelnen Skalen:
    </div>
    <div class="help-list">
      <div><b>Trust:</b></div>
      <div>
        <i>Belegt:</i>
        Der Satz wird durch eine Quellenangabe belegt, die auch im Kontext
        aufgeführt ist. Und die Informationen im Satz passen zum Kontext. (Die
        gegebenen Informationen können auch über die Angaben im Kontext
        hinausgehen, solange sie zu diesem passen (z.B. im Kontext wird ein
        Algorithmus erwähnt und der Satz gibt eine allgemeine Erklärung zu
        diesem Algorithmus))
      </div>
      <div>
        <i>Teilweise belegt:</i>
        Der Satz wird nicht durch eine Quellenangabe belegt, passt jedoch zu den
        Informationen aus dem Kontext. (Wie Belegt, jedoch ohne Quellenangabe)
      </div>
      <div>
        <i>Allgemeinwissen:</i>
        Die Informationen in dem Satz passen nicht zu den Inhalten aus dem
        Kontext, sie sind aber wahr.
      </div>
      <div>
        <i>Falschaussage:</i>
        Der Satz lässt sich widerlegen.
      </div>
      <div>
        <i>Quatsch:</i>
        Der Satz ist offensichtlich falsch, da er z.B. keinen Sinn ergibt,
        widersprüchlich ist oder nicht verständlich ist. (Auch Sätze mit
        falschen Quellenangaben, die nicht im Kontext aufgelistet sind sollten
        als "Quatsch" bewertet werden.)
      </div>
    </div>
    <div class="help-list">
      <div><b>Helpfullness:</b></div>
      <div>
        <i>Hilfreich:</i>
        Der Satz hilft klar bei der Beantwortung der Frage. Ein Satz kann nur
        als hilfreich bewertet werden, wenn zuvor keine wichtigen Teile für die
        Antwort ausgelassen wurden. (z.B. Die Frage besteht aus Teil a und b.
        Der Satz bezieht sich auf Teil b, ohne dass Teil a bereits behandelt
        wurde. Der Satz kann dann maximal als eingeschränkt hilfreich bewertet
        werden.)
      </div>
      <div>
        <i>Eingeschränkt:</i>
        Der Satz hilft nur teilweise bei der Beantwortung der Frage.
      </div>
      <div>
        <i>Unklar:</i>
        Der Satz passt zwar zum Thema der Frage, hilft aber nicht bei der
        Beantwortung der Frage.
      </div>
      <div>
        <i>Wiederholung:</i>
        Der Satz wiederholt hilfreiche oder thematisch passende Informationen
        aus vorherigen Sätzen. Die Information ist also nicht neu.
      </div>
      <div>
        <i>Nicht hilfreich:</i>
        Der Satz ist nicht hilfreich, irreführend oder unverständlich.
      </div>
    </div>
    <button class="close-help" @click="showHelp = false">
      {{ "Verstanden" }}
    </button>
  </div>
</template>

<style scoped>
.screen {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 30px;
  display: grid;
  grid-template-areas:
    "context progress"
    "context question"
    "context answer"
    "context nav";
  grid-template-columns: 1fr 2fr;
  grid-template-rows: 45px auto 1fr 80px;
  column-gap: 20px;
  row-gap: 20px;
  color: black;

  text-align: left;
}

.left {
  grid-area: context;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.progress {
  grid-area: progress;
  background-color: #f0f0f0;
  padding: 10px;
  font-size: 16px;
}

.question {
  grid-area: question;
  background-color: #f0f0f0;
  padding: 20px;
  font-size: 16px;
}

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
  &[data-next="true"] {
    background-color: lightgray;
  }
}

.score {
  font-size: 12px;
  flex: 0;
  background-color: gray;
  padding: 10px;

  &[data-next="true"] {
    background-color: lightgray;
  }

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
.help-background {
  position: absolute;
  inset: 0;
  background-color: black;
  opacity: 0.5;
}
.help {
  white-space: pre-line;
  position: absolute;
  inset: 0;
  width: 70%;
  height: 70%;
  margin: auto;
  background-color: #f0f0f0;
  padding: 20px;
  font-size: 16px;
  color: black;
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 20px;

  overflow: auto;
}

.help-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

button {
  color: black;
  background-color: #a0a0a0;
  border: none;
  &:hover,
  &:focus {
    background-color: #c0c0c0;
  }

  &:active {
    background-color: #d0d0d0;
  }

  &[disabled] {
    background-color: #e0e0e0;
    color: #a0a0a0;
  }
}

.navigation {
  grid-area: nav;
  background-color: #f0f0f0;
  padding: 20px;
  font-size: 16px;
  display: flex;
  justify-content: space-between;
}

.scoreOption {
  display: flex;
  gap: 5px;

  & > label {
    flex: 1;
  }
}
</style>
