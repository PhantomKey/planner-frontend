<template>
  <q-layout view="lHh Lpr lFf">
    <q-header style="background-color: white">
      <q-toolbar style = "height: 80px">
        <div style="min-width:33.59px; width:33.59px">
           <q-btn
            flat
            dense
            round
            @click="status = !status; showAllFriends();"
            aria-label="Menu"
            v-if="isLogin()"
            >
            <q-icon name="menu" />
          </q-btn>
        </div>
        <q-toolbar-title @click="home" style="cursor:pointer; color:#e78483;">
          <q-avatar square>
            <img src="~assets/trip_logo2.png"/>
          </q-avatar>
          TRIP PLANNER
        </q-toolbar-title>
        <div v-if="isLogin()" class="q-gutter-sm">
          <!-- <add-friend :showFriends="showAllFriends"></add-friend> -->
          <!-- <logout-button></logout-button> -->
          <friend></friend>
          <profile :name="username"></profile>
        </div>
      </q-toolbar>
    </q-header>
     <q-drawer
      v-model="status"
      bordered
      content-class="bg-grey-2"
    >
      <q-item-label header>Your Friend</q-item-label>
      <div v-if="isLogin()">
      <q-list v-model="friendlist" v-for="i in friendlist" :key="i.id">
        <q-item>
          <q-item-section avatar>
            <q-icon name="school"/>
          </q-item-section>
          <q-item-section>
            <q-item-label>{{i.name}}</q-item-label>
            <q-item-label v-if="!i.isowner" caption>Your friend</q-item-label>
            <q-item-label v-else caption>You</q-item-label>
          </q-item-section>
          <q-btn @click="deleteMember(i.id)">
            remove
          </q-btn>
        </q-item>
      </q-list>
      </div>
      <div v-else>
        You must log in to use this feature.
      </div>
    </q-drawer>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL, colors } from 'quasar'
import AddFriend from '../components/AddFriend.vue'
import LogoutButton from '../components/LogoutButton.vue'
import VueJwtDecode from 'vue-jwt-decode'
import Profile from '../components/Profile.vue'
import Friend from '../components/Friendbutton.vue'
export default {
  name: 'MyLayout',
  mounted() {
        this.$root.$on('component1', () => {
            // your code goes here
            this.showAllFriends()
        })
        this.showUserName()
  },
  data () {
    return {
      friendlist:[],
      leftDrawerOpen: this.$q.platform.is.desktop,
      status: false,
      token:false,
      username:''
    }
  },

  components: {
    'add-friend': AddFriend,
    'logout-button': LogoutButton,
    'profile': Profile,
    'friend':Friend
  },
  methods: {
    openURL,
    home () {
      if (localStorage.token){
        this.$router.push('/Home')
      }else{
        this.$router.push('/')
      }
    },
    checktoken () {
      this.token = localStorage.token ? true: false
  },
  created :function() {
    this.checktoken()
    this.showUserName()
  },
  updated :function() {
    this.checktoken()
  },
  async showAllFriends () {
    this.friendlist=[]
    let headers = {'Authorization': 'JWT '+localStorage.token}
    let value = await this.$http.get('/api/v1/member',{headers})
    this.renderFriends(value)
    // console.log('very good')
  },
  renderFriends (value) {
    for(var i = 0;i < value['data']['id'].length;i++)
      this.friendlist.push({id:value['data']['id'][i],name:value['data']['members'][i],isowner:value['data']['owner'][i]})
    // console.log(this.friendlist)
  },
  isLogin () {
    // console.log(localStorage.token)
    if (localStorage.token) {
      this.showUserName()
      return true
    } else {
      this.friendlist=[]
      return false
    }
  },
  async deleteMember (x) {
    await this.$http.delete('/api/v1/delete_member/'+x)
    this.showAllFriends()
  },
  async showUserName(){
    let user_id= VueJwtDecode.decode(localStorage.token).user_id
    // console.log('mounted')
    // console.log(user_id)
    let response = await this.$http.get('/api/v1/user/'+user_id)
    let username = response.data.user.name
    this.username = username
  }
}
}
</script>

<style scoped>
friend:hover{
  border-bottom: 5px solid #fa928f
}
</style>
