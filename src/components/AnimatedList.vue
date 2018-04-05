<template>
  <div class="container">
    <div class="pagination">
      <button 
        :disabled="currentPage === 1 || loadingData" 
        @click.prevent="changePage(currentPage - 1)">
        Prev
      </button>
      <button 
        :disabled="currentPage === $options.maxPages || loadingData" @click.prevent="changePage(currentPage + 1)">
        Next
      </button>
    </div>
    <div class="list-container">
      <transition-group
        @before-enter="beforeEnter"
        @enter="enter"
        @after-enter="afterEnter"
        @before-leave="beforeLeave"
        @leave="leave"
        @afterLeave="afterLeave"
        tag="div">
        <div 
          class="list-item" 
          v-for="(person, index) in people" 
          :key="person.name + index"
          :data-index="index">
          {{ person.name }}
        </div>
      </transition-group>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { TweenLite } from 'gsap'
const HTTP = new axios.create({
  baseURL: 'https://swapi.co/api/'
})
export default {
  data: () => ({
    people: [],
    peopleToUpdate: [],
    currentPage: 1,
    loadingData: false,
    animating: false
  }),
  watch: {
    animating(val) {
      if (!val && this.peopleToUpdate.length > 0) {
        this.people = this.peopleToUpdate
        this.$options.peopleLength = this.people.length                
        this.peopleToUpdate = []
      }
    }
  },
  methods: {
    changePage(page) {
      if (page < 0) page = 1
      else if (page > this.$options.maxPages) page = this.$options.maxPages
      this.updateList(page)
    },
    async updateList(page) {
      this.loadingData = true
      this.people = []
      const { results } = await this.getPeople(page)
      // Check if animation is still in progress
      if (this.animating) {
        // Add results to waiting array
        this.peopleToUpdate = results
      } else {
        // Animation has finished, assign new results and length
        this.people = results
        this.$options.peopleLength = results.length        
      }
      this.currentPage = page
      this.loadingData = false
    },
    async getPeople(page = 1) {
      const { data } = await HTTP.get('people', { params: { page } })
      return data
    },
    beforeEnter(el) {
      // Set opacity to none and move the initial position of the item by 50%
      el.style.opacity = 0
      el.style.left = '50%'
      // Switch on the animating flag
      if (!this.animating) {
        this.animating = true
      }
    },
    enter(el, done) {
      const delay = (el.dataset.index >= 10 ? 10 : el.dataset.index) * 100
      setTimeout(() => {
        // For half a second animate opacity and left property and call done on complete
        TweenLite.to(el, 0.5, {
          opacity: 1,
          left: 0,
          onComplete: done
        })
      }, delay)
    },
    afterEnter(el) {
      // We only want to execute this part of code after last item entered
      if (+el.dataset.index === this.$options.peopleLength - 1) {
        this.animating = false
      }
    },
    beforeLeave() {
      // Switch on animating flag
      if (!this.animating) {
        this.animating = true
      }
    },
    leave(el, done) {
      // Get a delay num
      const delay = el.dataset.index * 50
      setTimeout(() => {
        // For half a second animate opacity and left property and call done on complete
        TweenLite.to(el, 0.5, {
          opacity: 0,
          left: '50%',
          onComplete: done
        })
      }, delay)
    },
    afterLeave(el) {
      if (+el.dataset.index === this.$options.peopleLength - 1) {
        this.animating = false
      }
    }
  },
  async created() {
    this.loadingData = true    
    const { results, count } = await this.getPeople()
    this.people = results
    this.$options.peopleLength = results.length
    this.$options.maxPages = Math.ceil(count / 10)
    this.loadingData = false    
  }
}
</script>

<style>
.container {
  width: 50%;
  height: 100vh;
}
.list-container {
  border: 1px solid black;
  height: 100%;
}

.list-item {
  text-align: center;
  padding: 12px 0;
  margin: 20px;
  position: relative;
  background-color: #ccc;
}
.pagination {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.pagination button {
  padding: 8px 12px;
  border: 1px solid orange;
  cursor: pointer;
  transition: 0.25s all;
  background: transparent;
}

.pagination button:hover {
  background-color: orange;
}
</style>