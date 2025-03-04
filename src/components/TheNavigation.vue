<template>

  <div id="nav">

    <router-link id="logo" to="/">Travel App</router-link>

    <router-link
      v-for="destination in data"
      :key="destination.id"
      :to="{
        name: 'destination.show',
        params: { id: destination.id, slug: destination.slug },
      }"
    >
       {{ destination.name }}
    </router-link>

  </div>

</template>

<script setup>
import { ref } from 'vue'
import dataSource from '@/data.json'

const data = ref(dataSource.destinations)
</script>

<!-- <style>
#nav .vue-school-active-link{
    color:red
}
</style> -->

