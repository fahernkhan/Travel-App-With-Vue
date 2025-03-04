<template>

  <section v-if="data" class="destination">

    <h1 class="destination-name">{{ data.name }}</h1>

    <div class="destination-details">

      <img class="image" :src="`/images/${data.image}`" :alt="data.name" />

      <p class="description">{{ data.description }}</p>

    </div>

  </section>

  <section v-if="data" class="experiences">

    <h2>Top Experiences in {{ data.name }}</h2>

    <div class="cards">

      <router-link
        v-for="experience in data.experiences"
        :key="experience.slug"
        :to="{
          name: 'experience.show',
          params: { slug: experience.slug, id: route.params.id },
        }"
      >

        <ExperienceCard :experience="experience" />

      </router-link>

    </div>

  </section>

</template>

<script setup>
import ExperienceCard from '@/components/ExperienceCard.vue'
import router from '@/router'
import { ref, computed, watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const slug = computed(() => route.params.slug)

const data = ref(null)

async function fetchData() {
  try {
    const res = await fetch(
      `https://travel-dummy-api.netlify.app/${slug.value}.json`
    )
    if (res.status === 200) {
      data.value = await res.json()
    }
  } catch (error) {
    console.log(error)
  }
}

watch(slug, fetchData, { immediate: true })
</script>

