<template>

  <section class="experience-details">

    <h1>{{ data.name }}</h1>

    <div class="destination-details">

      <img :src="`/images/${data.image}`" />

      <p class="description">{{ data.description }}</p>

    </div>

  </section>

</template>

<script setup>
import dataSource from '@/data.json'
import { ref, computed, watch } from 'vue'
import { useRoute } from 'vue-router'
const route = useRoute()
const data = ref(null)
const id = computed(() => route.params.id)
const slug = computed(() => route.params.experienceSlug)

function fetchData() {
  const destination = dataSource.destinations.find(
    (destination) => destination.id === Number(id.value)
  )
  const experience = destination.experiences.find(
    (experience) => experience.slug === slug.value
  )
  data.value = experience
}

watch(slug, fetchData, { immediate: true })
</script>

