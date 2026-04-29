<template>
  <form @submit.prevent="submitSearch">
    <input
      v-model="city"
      type="text"
      placeholder="Enter city name"
    />
    <button type="submit">Search</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      city: ''
    }
  },
  methods: {
    submitSearch() {
      if (!this.city.trim()) return

      this.$emit('search', this.city)
      this.city = ''
    }
  }
}
</script>

<style scoped>
form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input {
  padding: 10px;
}

button {
  padding: 10px;
}
</style>