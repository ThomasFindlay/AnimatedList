<template>
  <div>

  </div>
</template>

<script>
import axios from 'axios'
const HTTP = new axios.create({
  baseURL: 'https://swapi.co/api/'
})
export default {
  data: () => ({
    people: []
  }),
  methods: {
    async getPeople(page = 1) {
      return await HTTP.get('people', {
        page
      })

      this.people = result.data.results
      this.$options.count = result.data.count
      console.log(result);
      console.log(this.people);
      console.log(this.$options.count);
    }
  },
  created() {
    this.getPeople()
  }
}
</script>

<style scoped>

</style>