<template>
  <div className="p-6 bg-gray-900 shadow rounded-lg">
    <div class="wrapper">
      <h1 className="text-xl border-b">Database Checker</h1>
      <p>Checkout user <b>{{ city == "" ? "in database" : cityName }}</b></p>
      <input class="test" v-model="city" placeholder="Enter city "></input>
      <button v-if="!city == ''" className="m-2 p-2 text-black font-medium rounded-lg transition ease-in-out delay-50 bg-cyan-500 shadow-lg hover:text-white hover:-translate-y-0.3 hover:bg-indigo-500 duration-150 shadow-indigo-500/50 hover:shadow-cyan-500/50" @click="">
        Submit
      </button>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      city: ""
    }
  },
  computed: {
    cityName() {
      return "`" + this.city + "`"
    }
  }
}
</script>


<style scoped>
@tailwind base;
@tailwind components;
@tailwind utilities;

.wrapper {
  width: 400px;
  height: 300px;
  border: 3px solid red;
  border-radius: 50px;
}
.wrapper h1 {
  margin-top: 50px;
}

.wrapper input {
  margin-top: 30px;
  padding: 5px 8px;
  background: transparent;
  border: 0;
  border-bottom: 2px solid #110813;
  font-size: 0.9em;
  outline: none;
}

.wrapper input:focus {
  border-bottom-color: #6e2d7d;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
button {
  outline: none;
}
</style>
