<template>
  <div class="d-flex quiz-container">
    <div class="flex-basis-5-12 left-part h-screen d-flex justify-center content-center">
      <img
        src="~/assets/images/quiz.svg"
        alt="quiz"
      />
    </div>
    <div class="flex-basis-7-12 right-part items-center justify-center h-screen pl-10 pr-10">
      <QuizQuestions />
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: "quiz",
});
</script>