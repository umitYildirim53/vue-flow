<script setup>
import { computed, reactive } from "vue";

const destinations = reactive([
  {
    id: "1223dasda",
    name: "1223dasda",
  },
  {
    id: "54646erter",
    name: "54646erter",
  },
]);

const destinationId = computed(() => {
  return this.$route.params.id;
});
const destination = computed(() => {
  return destinations.find((destination) => destination.id === destinationId);
});
</script>

<template>
  <div>
    <h1>{{ destination.name }}</h1>
  </div>
</template>
