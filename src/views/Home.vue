<template>

  <div class="home">

    <h1>All Destination</h1>

    <div v-if="destinations" class="destinations">

      <router-link
        v-for="destinations in destinations"
        :key="destinations.id"
        :to="{
          name: 'destination.show',
          params: { id: destinations.id, slug: destinations.slug },
        }"
      >

        <h2>{{ destinations.name }}</h2>

        <img :src="`/images/${destinations.image}`" :alt="destinations.name" />

      </router-link>

    </div>

  </div>

</template>

<script setup>
import { ref } from 'vue'
import dataSource from '@/data.json'

const destinations = ref(dataSource.destinations)
</script>

