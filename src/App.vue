<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useVirtualList, useInfiniteScroll } from '@vueuse/core'

const movies: any = ref([])
const page = ref(0)
const apiKey =
  'eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxYjZlNzdkODRjMjZjZjIzMjY0YjdmYzM3YTljNDJiNiIsInN1YiI6IjY1Y2ZhYzBhNjBjNzUxMDE3YjY5ZTRmMyIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.gx2SDexc2pCDYip897p87yEruKWq7i8tKDYg9kFcxQ4'

const movieList = computed(() => {
  return movies.value.map((movie: any) => {
    return {
      ...movie,
      release_date: convertDate(movie.release_date)
    }
  })
})

const { list, containerProps, wrapperProps } = useVirtualList(movieList, {
  itemHeight: 380,
  overscan: 5
})

const fetchMovies = () => {
  if (page.value !== 500) {
    page.value++
  }

  const options = {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${apiKey}`
    }
  }

  fetch(`https://api.themoviedb.org/3/movie/now_playing?language=en-US&page=${page.value}`, options)
    .then((response) => response.json())
    .then((response) => {
      movies.value.push(...response.results)
    })
    .catch((err) => console.error(err))
}

const convertDate = (date: string) => {
  var dob = new Date(date)
  var dobArr = dob.toDateString().split(' ')
  return dobArr[1] + ' ' + dobArr[2] + ', ' + dobArr[3]
}

useInfiniteScroll(
  containerProps.ref,
  () => {
    fetchMovies()
  },
  { distance: 10 }
)

onMounted(() => {
  fetchMovies()
})
</script>

<template>
  <div v-bind="containerProps" class="movies-container">
    <div v-bind="wrapperProps" class="movies-wrapper">
      <div
        v-for="{ index, data } in list"
        :id="`container-${index}`"
        :key="index"
        class="movie-item"
      >
        <img :src="`https://image.tmdb.org/t/p/w500${data.poster_path}`" />
        <div>
          <h4>{{ data.title }}</h4>
          <h5>Release dated: {{ data.release_date }}</h5>
          <p>{{ data.overview }}</p>
        </div>
      </div>
    </div>
    <a class="absolute top-0 left-0" href="#container-118">goto</a>
  </div>
  <!-- <div v-bind="containerProps" class="movies-container">
    <div class="movies-wrapper">
      <div v-for="(movie, i) in movieList" :key="i" class="movie-item">
        <img :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`" />
        <div>
          <h4>{{ movie.title }}</h4>
          <h5>Release dated: {{ movie.release_date }}</h5>
          <p>{{ movie.overview }}</p>
        </div>
      </div>
    </div>
  </div> -->
</template>

<style lang="scss" scoped>
.movies-container {
  @apply mx-auto py-5 h-screen;

  .movies-wrapper {
    @apply max-w-2xl mx-auto;

    .movie-item {
      @apply p-5 flex h-[360px] mb-5 bg-white shadow-lg border rounded-sm gap-5 overflow-hidden;

      img {
        @apply w-1/3 rounded-sm;
      }

      h4 {
        @apply text-xl font-medium;
      }

      h5 {
        @apply mb-5 text-xs;
      }
    }
  }
}
</style>
