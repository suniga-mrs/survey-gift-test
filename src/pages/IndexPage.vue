<template>
  <q-page class="row items-center justify-evenly bg-brown-1">
    <survey-form />
  </q-page>
</template>

<script setup lang="ts">
import SurveyForm from 'components/SurveyForm.vue'

defineOptions({
  name: 'IndexPage'
})
</script>
