<template>
  <div class="content">
    <div class="main-nav">
      <app-nav></app-nav>
    </div>
    <h2>Select user to update/delete</h2>
    <select v-model="username">
      <option disabled value="">Select user</option>
      <option v-for="item in this.userlist" v-bind:key="item.username">{{ item.username }}</option>
    </select>
    <button v-on:click="amendUser(username)">Do stuff to this user</button>
    <br><br>
    <div v-if="disp">
      <h4>Username: {{ this.userData.username }}</h4>
      <table class="center">
        <tr>
          <td>
            <button v-on:click="amendKeycode()">Change keycode</button>
            <button v-on:click="amendDoors()">Change door permissions</button>
            <button v-on:click="amendValiddates()">Change dates allowed</button>
          </td>
        </tr>
        <tr>
          <td>
            <button v-on:click="amendEnabled()">Enable or disable</button>
            <button v-on:click="amendPass()">Change password</button>
            <button v-on:click="amendRole()">Change role</button>
          </td>
        </tr>
        <tr>
          <td colspan="3">
            <button v-on:click="sendDelete(userData.username)">Delete user</button>
          </td>
        </tr>
      </table>
      <br><br>
      <div v-if="edType == 'role'">
        Current role is: {{ userData.role }}
        <br>
        Role:
        <div v-for="item in roles" v-bind:key="item">
          <input type="radio" :id="item" :value="item" v-model="role">
          <label for="item">{{ item }}</label>
        </div>
        Allowed sites:
        <select v-model="allowedsites" multiple>
          <option disabled value="">Select allowed sites</option>
          <option v-for="item in sites" v-bind:key="item" >{{ item }}</option>
        </select>
        <!-- <table class="center">
          <tr>
            <td class="tabLabel">
              <label for="admin">Admin</label>
            </td>
            <td>
              <input type="radio" id="admin" value="admin" v-model="role">
            </td>
          </tr>
          <tr>
            <td class="tabLabel">
              <label for="user">User</label>
            </td>
            <td>
              <input type="radio" id="user" value="user" v-model="role">
            </td>
          </tr>
        </table>
        <br> -->
        <button v-on:click="changeRole()">Change role</button>
      </div>
      <div v-if="edType == 'key'">
        Keycode: <input v-model="key" :placeholder="this.userData.keycode">
        <button v-on:click="changeattr(userData.username, 'keycode', key)">Change keycode</button>
      </div>
      <div v-if="edType == 'doors'">
        <table class="center">
          <tr>
            <th>
              Currently enabled doors
            </th>
          </tr>
          <tr v-for="item in this.userData.doors" v-bind:key="item">
            <td>
              {{ item }}
            </td>
          </tr>
          <tr>
            <th>
              Check required doors:
            </th>
          </tr>
          <tr v-for="x in doorlist" v-bind:key="x">
            <td>
              <input type="checkbox" :id="x" :value="x" v-model="doors">
              <label >{{ x }}</label>
            </td>
          </tr>
          <tr>
            <td>
              <button v-on:click="changeattr(userData.username, 'doors', doors)">Change doors</button>
            </td>
          </tr>
        </table>
      </div>
      <div v-if="edType == 'dates'" style="position: relative">
        Valid from: <date-picker v-model="newStart" :config="config" :placeholder="String(this.userData.times.start)"></date-picker>
        <br>
        <button v-on:click="changeattr(userData.username, 'timeStart', newStart)">Change start date</button>
        <br><br>
        Expires: <date-picker v-model="newEnd" :config="config" :placeholder="String(this.userData.times.end)"></date-picker>
        <br>
        <button v-on:click="changeattr(userData.username, 'timeEnd', newEnd)">Change end date</button>
      </div>
      <div v-if="edType == 'enabled'">
        Enabled: <input type="checkbox" id="checkbox" v-model="userData.enabled">
        <br>
        <button v-on:click="changeattr(userData.username, 'enabled', userData.enabled)">Change status</button>
      </div>
      <div v-if="edType == 'pass'">
          New password: <input type="password" v-model="pass1">
          Confirm password: <input type="password" v-model="pass2">
          <br>
          <button v-on:click="changePass(pass1, pass2)">Change password</button>
      </div>
    </div>
    <div v-if="response != ''">
      {{ response.data.Message }}
    </div>
  </div>
</template>

<script>
import { getUsers, getDoors, putUserData, putAllUserData, deleteDoorUser, userData, getSites } from '../../../../utils/door-api'
import 'bootstrap/dist/css/bootstrap.css'
import datePicker from 'vue-bootstrap-datetimepicker'
import AppNav from '../../AppNav'
import 'eonasdan-bootstrap-datetimepicker/build/css/bootstrap-datetimepicker.css'
export default {
  name: 'updateuser',
  data () {
    return {
      allowedsites: [],
      sites: [],
      roles: ['admin', 'user', 'sensuser'],
      doorlist: [],
      userlist: [],
      userData: '',
      username: '',
      key: '',
      newStart: '',
      newEnd: '',
      doors: [],
      enabled: '',
      role: '',
      pass1: '',
      pass2: '',
      disp: false,
      edType: '',
      config: {
        format: 'ddd, MMM DD YYYY, HH:mm'
      },
      response: ''
    }
  },
  components: {
    datePicker,
    AppNav
  },
  methods: {
    sitesRoles() {
      getSites().then((ret) => {
        this.sites = ret
        this.sites.push('doors')
      })
    },
    sendData (payload) {
      putAllUserData(payload).then((ret) => {
        this.response = ret
      })
    },
    amendKeycode () {
      this.edType = 'key'
    },
    amendDoors () {
      this.edType = 'doors'
    },
    amendValiddates () {
      this.edType = 'dates'
    },
    amendEnabled () {
      this.edType = 'enabled'
    },
    amendPass () {
      this.edType = 'pass'
    },
    amendRole () {
      this.sitesRoles()
      this.edType = 'role'
    },
    amendUser (username) {
      userData(username).then((ret) => {
        this.userData = ret
        // console.log(ret)
        // Change date format so it can be read
        this.userData.times.start = new Date(ret.times.start)
        this.userData.times.end = new Date(ret.times.end)
      })
      this.disp = true
      this.response = ''
      this.edType = ''
    },
    sendDelete (payload) {
      deleteDoorUser(payload).then((ret) => {
        this.response = ret
      })
      this.response = ''
      this.edType = ''
      // refesh list of users
      this.getUsers()
    },
    changePass () {
      if (this.pass1 === this.pass2) {
        this.changeattr(this.userData.username, 'password', this.pass1)
      } else {
        this.response = {'Status': 'Error', 'Message': 'Passwords do not match'}
      }
    },
    changeRole () {
      this.changeattr(this.userData.username, 'role', {'role': this.role, 'sites': this.allowedsites})
    },
    changeattr (userin, attr, val) {
      const payload = JSON.stringify({'username': userin, [attr]: val})
      console.log(payload)
      putUserData(payload, attr).then((ret) => {
        this.response = ret
        this.disp = false
        console.log(this.response)
      })
    },
    getDoors () {
      getDoors().then((ret) => {
        this.doorlist = ret
      })
    },
    getUsers () {
      getUsers().then((ret) => {
        this.userlist = ret.map(function (el) {
          var o = Object.assign({}, el)
          o.startDateObject = new Date(o.times.start)
          o.endDateObject = new Date(o.times.end)
          return o
        })
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
h1, h2 {
  font-weight: normal;
}

td {
  padding: 5px;
  text-align: center;
}

div.col-sm-3 {
  text-align: left;
    padding-top: 40px;
}

div.col-lg-7 {
  clear: both;;
  margin: 15px;
}

div.col-lg-3 {
  clear: right;
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

.radio {
  text-align: left;
}

th {
  text-align: center;
}
</style>
