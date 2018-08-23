<template>
  <div>
    <div class="main-nav">
      <app-nav></app-nav>
    </div>
    <div class="content">
       <h2>Currently Allowed Users</h2>
       <div v-for="item in allowedusers" v-bind:key="item">
         <li>{{ item.door }} has the following user(s) enabled:
           <div v-for="x in item.users" v-bind:key="x">
              {{ x }}
           </div>
         </li>
       </div>
     </div>
  </div>
</template>

<script>
import { getAllowedUsers } from '../../../../utils/door-api'
import AppNav from '../../AppNav'
export default {
  name: 'allowedusers',
  data () {
    return {
      allowedusers: []
    }
  },
  components: {
    AppNav
  },
  methods: {
    getAllowedUsers () {
      getAllowedUsers().then((ret) => {
        this.allowedusers = ret
      })
    }
  },
  mounted () {
    this.getAllowedUsers()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
