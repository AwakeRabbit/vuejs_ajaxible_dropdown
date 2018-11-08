<template>
  <div class="dropdown" :class="{expanded, loading}">
    <input type="text"  :readonly="!(expanded && editable)" @click="showList" @keyup="checkList" ref="inp" />
    <div class="fake-background" @click="expanded = false"></div>
      <ul>
        <li v-for="i in items.filter(i => i.hidden !== true)" @click="setValue(i.Value)" :key="i.Value" v-if="items.filter(i => i.hidden !== true).length > 0">
            
            {{i.Text}}
        </li>
        <li class="empty" v-if="items.length == 0 || items.filter(i => i.hidden !== true).length == 0">No data</li>
      </ul>
  </div>
</template>

<script>
export default {
  name: 'Dropdown',
  props: {
    data: {
      type: Array,
      default: []
    },
    editable: {
      type: Boolean,
      default: false
    },
    timeout: {
      type: Number,
      default: 1000
    },
    onKeyPause: {
      type: Function,
      default: null
    }

  },
  data: () => ({ 
    Text: '', 
    Value: null, 
    expanded: false, 
    loading: false, 
    items: [], 
    lastKeyUp: false 
  }),

  methods: {
    setValue: function(val) {
      this.expanded = false;
      let item = this.items.filter(i => i.Value === val)[0]
      if (!item) return
      this.Value = item.Value 
      this.Text = item.Text
      this.$refs.inp.value = this.Text
      this.$emit("input", this.Value)
    },
    checkList: function(event) {
      let word = this.$refs.inp.value.trim()

      if (word == '') {
        return false;
      } 
      if (this.onKeyPause != null) {
        if (event.keyCode == 13) {
            this.loading = true
            this.onKeyPause(word)
            this.lastKeyUp = 0
        } else {
            this.lastKeyUp = new Date()
            setTimeout(() => {
              if (this.expanded) { 
                if (word == this.$refs.inp.value.trim() && (new Date() - this.lastKeyUp) >= this.timeout && this.lastKeyUp != 0) {
                    this.loading = true
                    this.onKeyPause(this.$refs.inp.value.trim())
                    this.lastKeyUp = 0
                  }
              } else {
                this.lastKeyUp = 0
              }
            } , this.timeout)
        }
        
      }
      this.items = this.items.map( i =>  ({...i, hidden: (i.Text.indexOf(this.$refs.inp.value.trim()) < 0)}))
    },
    showList: function() {
        if (!this.expanded) this.items = this.items.map(i => ({...i, hidden: false}))
        this.expanded = true
    }
  },
  mounted: function () {
    window.addEventListener('keyup', function(e) {
        if (e.keyCode == 27) {
            let dd =  document.querySelector('.dropdown.expanded .fake-background') 
            if (dd) {
               dd.click()
            }
        }
    }) 
  },
  watch: {
    data: function(newVal, oldVal) {
      this.loading = false
      this.items = newVal || []
      
      if (this.editable === false && this.Value === null && this.Text === '' && newVal.length > 0) {
          this.setValue(newVal[0].Value)
      }
  }}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
$green: #42b983;
a {
  color: $green;
}
.dropdown {
  position: relative;
  display: inline-block;
  input {
    border: 1px solid #42b983;
    border-width: 0 0 1px;
    padding: 5px;
    cursor: pointer;
    user-select: none;
    outline: none;
  }
  .fake-background {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba($color: #000000, $alpha: 0);
    z-index: 9999;
  }
  ul {
    position: absolute;
    top: 26px;
    left: 0;
    height: 0;
    opacity: 0;
    overflow-y: hidden;
    z-index: 10000;
    background: #fff;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    min-width: 100%;
    text-align: left;
    list-style: none;
    padding: 0;
    margin: 0;
    overflow-y: auto;
    max-height: 200px;
    user-select: none;
    transition: all ease-in .3s;
  }
  li {
    cursor: pointer;
    padding: 8px 10px;
    max-width: 200px;
    font-size: 13px;
    line-height: 1em;
    word-wrap: break-word;
    &:hover {
      background: lighten($color: $green, $amount: 40%);
    }

    &.empty {
      color: #999;
    }
  }
  &.expanded {
    input {
      z-index: 10000;
    }
    .fake-background{
      display: block;
    }
    ul {
      overflow-y: auto;
      opacity: 1;
      height: auto;
    }
  }
  &.loading {
    &:after {
      display: block;
      position: absolute;
      top: 5px;
      right: 5px;
      width: 10px;
      height: 10px;
      border: solid 1px;
      border-color: $green #fff;
      box-shadow: 0 0 3px transparentize($color: $green, $amount: .2);
      content: '';
      border-radius: 50%;
      animation: rotateLoading .5s linear infinite 0s ; 
    }
  }
}
@keyframes rotateLoading {
  0% {transform: rotate(0deg);}
  100% {transform: rotate(180deg);}
}
</style>
