<template>
    <div>
        <q-btn round
        color="primary"
        icon="add"
        style="right:-95%;"
        size="18.5px"
        @click="openCreatePlanner = true">
        </q-btn>
        <q-dialog v-model="openCreatePlanner" @keyup.enter="clickAddPlanner()">
            <q-card dense style="width:600px;max-width:600px;height:650px;max-height:650px">
              <q-stepper
                v-model="step"
                header-nav
                ref="stepper"
                color="primary"
                animated
                >
                  <q-step
                  :name="1"
                  title="Create Planner"
                  icon="local_activity"
                  :done="done1"
                  style="min-height: 485px;max-height:485px"
                  >
                    <q-card-section class="row items-center">
                      <div class="text-h5" style="margin:0 auto;color:#fa928f">CREATE NEW PLANNER</div>
                    </q-card-section>
                    <q-card-section class="q-gutter-sm" style="padding:16px">
                      <q-input dense bg-color="secondary" ref="input" filled v-model="name" label="Name" style="min-width:100%;max-width:100%"></q-input>
                      <q-input dense bg-color="secondary" filled v-model="startdate" type="date" label="Start Date" stack-label style="width:100%">
                        <template v-slot:prepend>
                          <q-icon name="event" />
                        </template>
                      </q-input>
                      <q-input dense bg-color="secondary" filled v-model="enddate" type="date" label="Start Date" stack-label style="width:100%">
                        <template v-slot:prepend>
                          <q-icon name="event" />
                        </template>
                      </q-input>
                      <q-input dense bg-color="secondary" ref="input" filled v-model="description" label="Description" style="width:100%"></q-input>
                    </q-card-section>
              </q-step>

              <q-step
                :name="2"
                title="Select Friend"
                icon="person_add"
                :done="done2"
                style="min-height: 485px;max-height:485px"
                >
                  <q-card-section class="row items-center" style="padding-bottom:0px">
                      <div class="text-h5" style="margin:0 auto;color:#fa928f;padding-bottom:0px">SELECT FRIEND</div>
                  </q-card-section>
                  <friend-component :needdata="needfriendlist" @thisisdata="preparedata"></friend-component>

              </q-step>

              <template v-slot:navigation>
                <q-card-section position="bottom-right" style="text-align:right">
                  <q-stepper-navigation style="padding-top:0px" v-if="step === 1">
                    <div class="q-gutter-sm">
                      <q-btn v-close-popup flat label="Cancel" outline color="primary" style="text-align:right" @click="resetData()"></q-btn>
                      <q-btn @click="() => { done1 = true; step = 2 }" color="primary" label="Continue" style="text-align:right"/>
                    </div>
                  </q-stepper-navigation>
                  <q-stepper-navigation style="padding-top:0px" v-if="step === 2">
                    <div class="q-gutter-sm">
                      <q-btn flat @click="step = 1" color="primary" label="Back" class="q-ml-sm" />
                      <q-btn color="primary" class="no-shadow" label="Create" style="text-align:right" @click="sentdata()"></q-btn>
                    </div>
                  </q-stepper-navigation>
                </q-card-section>
              </template>

            </q-stepper>
          </q-card>
        </q-dialog>
    </div>
</template>
<script>
import { Notify } from 'quasar'
import FriendComponent from './SelectFriendComponent'
import { constants } from 'crypto';
export default{
  components:{
    'friend-component':FriendComponent
  },
  data() {
    return {
      openCreatePlanner: false,
      name: '',
      startdate: '',
      enddate: '',
      description: '',
      step:1,
      done1:false,
      done2:false,
      done3:false,
      needfriendlist:false,
      selected:[]
    }
  },
  watch:{
    selected:function(){
      this.clickAddPlanner()
    }
  },
  methods:{
    preparedata(value){
      if(!value){
      }else{
        this.selected = value
      }
    },
    resetData() {
      Object.assign(this.$data, this.$options.data.apply(this))
    },
    emitToHome(){
      this.$emit('refreshPlanner')
    },
    sentdata(){
      this.needfriendlist = !this.needfriendlist
    },
    clickAddPlanner() {
      setTimeout(()=>{
      let headers = {'Authorization': 'JWT '+localStorage.token}
      if(this.selected.length !=0){
        this.$http.post('/planner/create_planner', { planner_name: this.name,
          first_date: this.startdate, last_date: this.enddate,
          description: this.description,friendlist:this.selected}, {headers})
          .then(request => this.AddPlannerSuccessfulwithPOST(request))
          .catch((err) => this.AddPlannerFailedwithoutPOST(err))
      }
      },2000)
      this.showLoading()
    },
    AddPlannerSuccessfulwithPOST (req) {
      if (req.data.code === 201) {
        Notify.create({
          message: 'Planner created successfully',
          color: 'positive',
          textColor: 'black',
          timeout: 3000,
          position: 'top-right',
          icon: 'check_circle',
          actions: [{label:'Dismiss',color:'black'}]
        })
        this.openCreatePlanner = false
        this.emitToHome()
        Object.assign(this.$data, this.$options.data.apply(this))
      } else {
        this.AddPlannerFailedwithPOST(req)
      }
    },
    AddPlannerFailedwithPOST (req) {
      Notify.create({
        message: 'Failed to create planner, Reason: '+req.data.message,
        color: 'primary',
        textColor: 'white',
        timeout: 3000,
        position: 'top-right',
        icon: 'error',
        actions: [{label:'Dismiss',color:'white'}]
      })
    },
    AddPlannerFailedwithoutPOST(err) {
      Notify.create({
        message: 'Failed to create planner, '+err,
        color: 'accent',
        textColor: 'white',
        timeout: 3000,
        position: 'top-right',
        icon: 'error',
        actions: [{label:'Dismiss',color:'white'}]
      })
    },
    showLoading () {
      this.$q.loading.show()

      // hiding in 2s
      this.timer = setTimeout(() => {
        this.$q.loading.hide()
        this.timer = void 0
      }, 2000)
    }
  },
  beforeDestroy () {
    if (this.timer !== void 0) {
      clearTimeout(this.timer)
      this.$q.loading.hide()
    }
  }
}
</script>
