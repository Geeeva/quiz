<template>
  <div class="h-full d-flex justify-center items-center flex-col">
    <section
      class="quiz-item d-flex justify-start"
      v-if="showQuizResults === false"
      :class="quizItemIndex === step - 1 ? 'active' : ''"
      v-for="(quizItem, quizItemIndex) in quizItems"
      :key="quizItemIndex"
    >
      <div v-if="step == quizItem.id">
        <h2>{{ quizItem.question }}</h2>
        <div
          class="pb-2"
          type="radio"
          v-for="(
            availableAnswer, availableAnswerIndex
          ) in quizItem.availableAnswers"
          :key="availableAnswerIndex"
        >
          <label for="quizItemIndex" class="quizItemIndex">
            <input
              :disabled="filledAnswers[quizItemIndex] !== undefined"
              :id="availableAnswerIndex"
              :name="quizItemIndex"
              type="radio"
              :value="availableAnswer"
              v-model="filledAnswers[quizItemIndex]"
              @input="selectedAvailableAnswerHandler($event, quizItemIndex)"
            />
            {{ availableAnswer }}</label
          >
        </div>

        <div
          class="mt-4 pl-4 correct-answer"
          v-if="filledAnswers[quizItemIndex] === quizItem.correctAnswer"
        >
          You have selected the correct answer!
        </div>
        <div
          class="mt-4 pl-4 wrong-answer"
          v-if="
            filledAnswers[quizItemIndex] !== quizItem.correctAnswer &&
            filledAnswers[quizItemIndex] !== undefined
          "
        >
          Sorry, the correct answer is
          <div class="span-wrapper">
            <span>{{ quizItem.correctAnswer }}</span
            >!
          </div>
        </div>
        <div class="mt-4 pl-4" v-if="filledAnswers[step - 1] === undefined">
          {{ warning }}
        </div>
      </div>
    </section>
    <div class="arrow-buttons-wrapper d-flex justify-end content-center">
      <img
        v-if="step != 1 && showQuizResults === false"
        @click.prevent="prevStep()"
        src="~/assets/images/arrow-up.svg"
        alt="arrow-up"
      />
      <img
        class="ml-1"
        v-if="step !== noOfSteps && showQuizResults === false"
        @click.prevent="
          filledAnswers[step - 1] === undefined ? setWarning() : nextStep()
        "
        src="~/assets/images/arrow-down.svg"
        alt="arrow-down"
      />
    </div>
    <div
      class="submit mt-6"
      v-if="step == noOfSteps && showQuizResults === false"
      @click.prevent="submitQuizAnswers"
    >
      Submit answers
    </div>
    <section
      v-if="showQuizResults"
      class="score-results-wrapper d-flex flex-col items-center w-full"
    >
      <div class="score-results">
        <p class="pl-4">
          You have answered {{ correctAnswersScorePerGameSession }} question(s)
          correctly.
        </p>
        <p class="pl-4">
          You were better then {{ worsePlayersinPercentage }} of all quizers!
        </p>
      </div>
      <NuxtLink to="/" class="redo mt-6" @click="redoQuiz()"
        >Take one more chance</NuxtLink
      >
    </section>
  </div>
</template>

<script setup>
import nuxtStorage from "nuxt-storage";
import quizData from "../assets/quiz-data.json";

const quizItems = ref(quizData);

const step = ref(quizItems.value[0].id);
const noOfSteps = ref(quizItems.value.length);
const warning = ref("");
const filledAnswers = ref([]);
const correctAnswersScorePerGameSession = ref(0);
const worsePlayersinPercentage = ref("100%");
const showQuizResults = ref(false);

function selectedAvailableAnswerHandler(e, quizItemIndex) {
  filledAnswers.value[quizItemIndex] = e.target.value;
  quizItems.value.forEach((item) => {
    if (item.correctAnswer === e.target.value) {
      correctAnswersScorePerGameSession.value++;
    }
  });
}

function nextStep() {
  step.value++;
  warning.value = "";
}

function prevStep() {
  step.value--;
  warning.value = "";
}

function setWarning() {
  warning.value = "Please select an answer!";
}

function submitQuizAnswers() {
  showQuizResults.value = true;
  let scoresData = {};

  scoresData = nuxtStorage.localStorage.getData("scoresData", scoresData);

  let totalNumberOfPlayers;
  let totalNumberOfWorsePlayers = 0;
  worsePlayersinPercentage.value = "100%";
  let correctAnswersScore = correctAnswersScorePerGameSession.value;

  if (scoresData === null) {
    scoresData = { 0: 0, 1: 0, 2: 0, 3: 0, 4: 0 };
    scoresData[correctAnswersScore] = 1;
    totalNumberOfPlayers = 1;
  } else {
    totalNumberOfPlayers = (obj) =>
      Object.values(scoresData).reduce((a, b) => a + b, 0);
    for (const key in scoresData) {
      if (key < correctAnswersScore) {
        totalNumberOfWorsePlayers = totalNumberOfWorsePlayers + scoresData[key];
      }
    }

    scoresData[correctAnswersScore] = scoresData[correctAnswersScore] + 1;
    worsePlayersinPercentage.value = `${Math.round(
      (totalNumberOfWorsePlayers / totalNumberOfPlayers()) * 100
    )}%`;
  }
  nuxtStorage.localStorage.setData("scoresData", scoresData);
}

function redoQuiz() {
  showQuizResults.value = false;
  step.value = 1;
  filledAnswers.value = [];
  correctAnswersScorePerGameSession.value = 0;
}
</script>
