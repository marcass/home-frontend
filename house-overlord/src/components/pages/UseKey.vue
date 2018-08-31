<template>
  <div>
    <div class="main-nav">
      <app-nav></app-nav>
    </div>
    <div class="main-footer">
      <h3>Door's status</h3>
      <ul v-for="item in doorsstatus" v-bind:key="item">
        <li>
          <div class="open" v-if="item.status === 'open'">
            {{ item.door }} is currently {{ item.status }} at {{ item.time }}
          </div>
          <div class="closed" v-if="item.status === 'closed'">
            {{ item.door }} is currently {{ item.status }} at {{ item.time }}
          </div>
          <div class="unknown" v-if="item.status === 'unknown'">
            {{ item.door }} is currently {{ item.status }} at {{ item.time }}
         <!-- <br> -->
          </div>
        </li>
      </ul>
      <ul>
        <li>
          <button v-on:click="getDoorStatus()">Update status</button>
        </li>
      </ul>
    </div>
    <div class="content">
      <br>
      <select v-model="doorselected">
        <option disabled value="">Select a door to open</option>
        <option v-for="item in doorlist" v-bind:key="item">{{ item }}</option>
      </select>
    </div>
    <div class="content">
      <br>
      <h3>Input your keycode,<br> then click #</h3>
      <keyboard layouts="123A|456B|789C|*0{#:enter}D" v-model="keycode" @enter="postkey"></keyboard>
      <!-- <p> Keycode = {{ keycode }}</p> -->
      <div v-if="keysent">
        <div class="statusgood" v-if="resp.pin_correct === 1">
         <p>Status</p>
        </div>
        <div class="statusbad" v-else>
         <p>Status</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { postKeycode, getDoors, getADoorStatus, getDoorStatus } from '../../../utils/door-api'
import keyboard from 'vue-keyboard'
import AppNav from '../AppNav'
export default {
  name: 'updateuser',
  data () {
    return {
      doorlist: [],
      keysent: false,
      keycode: '',
      doorselected: '',
      doorstatus: [],
      doorsstatus: [],
      resp: []
    }
  },
  components: {
    keyboard,
    AppNav
  },
  methods: {
    postkey () {
      postKeycode(JSON.stringify({'door': this.doorselected, 'pincode': this.keycode})).then((ret) => {
        this.resp = ret
      })
      this.keycode = ''
      this.keysent = true
    },
    getDoors () {
      getDoors().then((ret) => {
        this.doorlist = ret
      })
    },
    getADoorStatus (door) {
      getADoorStatus(door).then((ret) => {
        this.doorstatus = ret
      })
      return this.doorstatus.status
    },
    getDoorStatus () {
      getDoorStatus().then((ret) => {
        this.doorsstatus = ret
        return this.doorsstatus
      })
    }
  },
  mounted () {
    this.getDoors()
    this.getDoorStatus()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div.statusgood {
    background-color: green;
    width: 50px;
    margin:0 auto;
}
div.statusbad {
    background-color: red;
    margin:0 auto;
    width: 50px;
}
.closed {
    border-radius: 25px;
    background-color: #9fdf9f;
}
.open {
    border-radius: 25px;
    background-color: #ff8080;
}
.unknown {
    border-radius: 25px;
    background-color: #d1e0e0;
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b283;
}
</style>
