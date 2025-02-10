<template>
  <div class="about">
    <ParticipationForm />

  </div>
</template>

<script setup>
import ParticipationForm from "@/components/ParticipationForm.vue";
</script>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
