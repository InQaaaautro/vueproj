<template>
  <div id="app">
    <h1>Add event</h1>
    <label for="addingEvent">Name event: </label><input id="addingEvent" v-model="nameEvent" />
    <button @click="addEvent">ADD</button>
    <data-picker
        :events="events"
        @get-date="getDate"/>
  </div>
</template>

<script>

import DataPicker from './components/DataPicker.vue'

export default {
  name: 'App',
  data() {
    return {
      dateEvent: '',
      nameEvent: '',
      events: [
        {date: '07.09.2022', name: 'test1', id: '4082022test1'},
        {date: '05.09.2022', name: 'test2', id: '4082022test2'},
        {date: '05.08.2022', name: 'another month', id: '40821022test2'},
        {
          date: '09.09.2022',
          name: 'much_longer______________________________________________test2',
          id: '99082022test2'
        }
      ]
    }
  },
  methods: {
    addEvent() {
      this.events.push({
        date: this.dateEvent,
        name: this.nameEvent,
        id: Math.random()
      });
      this.dateEvent = '';
      this.nameEvent = '';
    },
    getDate(dateEventPick) {
      this.dateEvent = dateEventPick;
    }
  },
  components: {
    DataPicker
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
