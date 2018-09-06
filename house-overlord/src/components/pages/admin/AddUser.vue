<template>
  <div>
    <div class="main-nav">
      <app-nav></app-nav>
    </div>
    <h2>Add a user</h2>
    <div class="side">
      <h3>Existing users</h3>
      <ul>
        <li v-for="(item, index) in userlist" v-bind:key="index">
          {{ item.username }}
        </li>
      </ul>
    </div>
    <div class="content">
      <div v-if="buttons">
        Please select the type of user you want to add
        <button v-on:click="doorUser()">Door user</button>
        <button v-on:click="sensorUser()">Sensor user</button>
        <button v-on:click="systemUser()">System user</button>
      </div>
      <div v-if="dooruser">
        <table class="center">
          <tr>
            <td class="tabLabel">
              Username (if 'burner' it is onetime use)
            </td>
            <td>
              <input v-model="username">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Password
            </td>
            <td>
              <input type="password" v-model="pass1">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Confirm password
            </td>
            <td>
              <input type="password" v-model="pass2">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Keycode
            </td>
            <td>
              <input v-model="keycode">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Role
            </td>
            <td v-for="item in doorRoles" v-bind:key="item">
              <input type="radio" :id="item" :value="item" v-model="role">
              <label for="item">{{ item }}</label>
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Enabled
            </td>
            <td>
              <input type="checkbox" id="checkbox" v-model="enabled">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Valid from
            </td>
            <td>
              <div style="position: relative">
                <date-picker v-model="startDateObject" :config="config"></date-picker>
              </div>
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              Valid until
            </td>
            <td>
              <div style="position: relative">
                <date-picker v-model="endDateObject" :config="config"></date-picker>
              </div>
            </td>
          </tr>
          <tr>
            <th colspan="2"  class="middle">
              <br>
              Select doors that apply
            </th>
          </tr>
          <tr v-for="item in doorlist" v-bind:key="item">
            <td class="tabLabel">
              <label >{{ item }}</label>
            </td>
            <td>
              <input type="checkbox" :id="item" :value="item" v-model="enableddoorlist">
            </td>
          </tr>
          <tr>
            <td colspan="2" class="middle">
              <button v-on:click="addUser(JSON.stringify({'username': this.username, 'password': this.pass1, 'role': this.role, 'keycode': this.keycode, 'enabled': this.enabled, 'timeStart': this.startDateObject, 'timeEnd': this.endDateObject, 'doorlist': this.enableddoorlist}))">Submit</button>
            </td>
          </tr>
        </table>
      </div>
      <div v-if="sensoruser">
        Username:
        <input v-model="username">
        Password
        <input type="password" v-model="pass1">
        Confirm password
        <input type="password" v-model="pass2">
        Role:
        <div v-for="item in sensorRoles" v-bind:key="item">
          <input type="radio" :id="item" :value="item" v-model="sensorrole">
          <label for="item">{{ item }}</label>
        </div>
        Allowed sites:
        <select v-model="sensorsite" multiple>
          <option disabled value="">Select allowed sites</option>
          <option v-for="item in sites" v-bind:key="item" >{{ item }}</option>
        </select>
        <button v-on:click="addUser(JSON.stringify({'username': this.username, 'password': this.pass1, 'role': { 'role': this.sensorrole, 'sites': this.sensorsite }}))">Submit</button>
      </div>
      <div v-if="systemuser">
        Role:
        <div v-for="item in systemRoles" v-bind:key="item">
          <input type="radio" :id="item" :value="item" v-model="sensorrole">
          <label for="item">{{ item }}</label>
        </div>
        <button v-on:click="addUser(JSON.stringify({'username': this.username, 'password': this.pass1, 'role': this.systemrole }))">Submit</button>
      </div>
      <div v-if="resp != ''">
        <br><br>
        User
        {{ resp.data.Message }}
      </div>
    </div>
  </div>
</template>

<script>
import { getUsers, getDoors, postUserData, getSites } from '../../../../utils/door-api'
import 'bootstrap/dist/css/bootstrap.css'
import datePicker from 'vue-bootstrap-datetimepicker'
import AppNav from '../../AppNav'
import 'eonasdan-bootstrap-datetimepicker/build/css/bootstrap-datetimepicker.css'
export default {
  name: 'updateuser',
  data () {
    return {
      sensorrole: '',
      sensorsite: [],
      systemrole: '',
      buttons: true,
      dooruser: false,
      sensoruser: false,
      systemuser: false,
      doorlist: [],
      userlist: [],
      message: '',
      keycode: '',
      resp: '',
      enableddoorlist: [],
      config: {
        format: 'ddd, MMM DD YYYY, HH:mm'
      },
      username: '',
      pass1: '',
      pass2: '',
      role: '',
      doorRoles: ['admin', 'user'],
      sensorRoles: ['admin', 'sensuser'],
      systemRoles: ['python', 'sensor'],
      sites: [],
      roles: ['admin', 'user', 'sensor', 'python', 'sensuser'],
      endDateObject: '',
      startDateObject: '',
      enabled: '',
      frontend: ''
    }
  },
  components: {
    datePicker,
    AppNav
  },
  methods: {
    doorUser () {
      this.dooruser = true
      this.systemuser = false
      this.sensoruser = false
      this.frontend = 'doors'
    },
    systemUser () {
      this.systemuser = true
      this.dooruser = false
      this.sensoruser = false
      this.frontend = null
    },
    sensorUser () {
      this.sensoruser = true
      this.dooruser = false
      this.systemuser = false
      this.frontend = 'sensors'
      this.sitesRoles()
    },
    postData (payload) {
      postUserData(payload).then((ret) => {
        this.resp = ret
      })
      // this.$router.push('/users')
    },
    getDoors () {
      getDoors().then((ret) => {
        this.doorlist = ret
      })
    },
    getUsers () {
      getUsers().then((ret) => {
        this.userlist = ret
      })
    },
    addUser (data) {
      if (this.pass1 === this.pass2) {
        this.postData(data)
        this.getUsers()
      } else {
        this.resp = {'data': {'Status': 'Error', 'Message': 'Passwords do not match'}}
      }
    },
    addSensorUser () {
      if (this.pass1 === this.pass2) {
        this.postData(JSON.stringify({'username': this.username, 'password': this.pass1, 'role': { 'role': this.sensorrole, 'sites': this.sensorsite }}))
        this.getUsers()
      } else {
        this.resp = {'data': {'Status': 'Error', 'Message': 'Passwords do not match'}}
      }
    },
    addSystemUser () {
      if (this.pass1 === this.pass2) {
        this.postData(JSON.stringify({'username': this.username, 'password': this.pass1, 'role': this.systemrole }))
        this.getUsers()
      } else {
        this.resp = {'data': {'Status': 'Error', 'Message': 'Passwords do not match'}}
      }
    },
    sitesRoles() {
      getSites().then((ret) => {
        this.sites = ret
      })
    }
  },
  mounted () {
    this.getDoors()
    this.getUsers()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.radio {
  text-align: left;
}

td {
  padding: 5px;
}

.middle {
  text-align: center;
}

.justify {
  margin-left: auto;
  margin-right: auto;
  width: 30%;
  text-align: right;
}
/*span {
  vertical-align: top;
}*/

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  vertical-align: top;
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b283;
}
</style>
