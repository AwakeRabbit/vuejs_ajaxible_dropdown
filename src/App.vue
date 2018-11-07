<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <br>
    <label for="">Editable: </label>
    <input type="checkbox" v-model="editable">
    <input type="text" v-model="current" placeholder="Add new item" @keyup.enter="addItem"/>
    <br><br><br>
    <Dropdown :data="items" v-model="val" :editable="editable" :on-key-pause="(str) => ajaxSimulation(str)"/>
    <br>  <br><br>
    <br>
    <label for="">Choosed: </label><p>"{{val ? items.filter(i => i.Value === val)[0] ? items.filter(i => i.Value === val)[0].Text : '' : ''}}"</p>
  </div>
</template>

<script>
import Dropdown from './components/Dropdown.vue'

export default {
  name: "app",
  data: () => ({
    current: "",
    items: [],
    val: null,
    editable: false,
  }),
  components: {
    Dropdown
  },
  methods: {
    addItem: function() {
      this.items.push({Text: this.current, Value: this.items.length});
      this.current = "";
    },
    alert: function(text) {
      window.alert(text)
    },
    ajaxSimulation: function (str) {
        setTimeout(() => {this.items = [];
          if (str.length > 2) {
            for (let i = 0; i < 10; this.items.push({Value: ++i, Text: str + i}));
          }
        }, 4000)
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
