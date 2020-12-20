<template>
  <div class="home">
    <MovieFetcher/>
  </div>
</template>

<script>
// @ is an alias to /src
import MovieFetcher from '@/components/movieFetcher.vue'

export default {
  name: 'Home',
  components: {
    MovieFetcher
  }
}
</script>
