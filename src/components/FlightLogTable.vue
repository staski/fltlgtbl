<template>
  <div>
    <b-table small striped hover :fields="fields" :items="flightSegments" responsive="sm">

      <template v-slot:cell(date)="data">
        {{ showDate(data.item.takeoffTime) }}
      </template>

    <template v-slot:cell(plane)="data">
      {{ data.value}}
    </template>

    <template v-slot:cell(pilot)="data">
      {{ data.value}}
    </template>
    
    <template v-slot:cell(departureAirport)="data">
      {{ data.value }}
    </template>

    <template v-slot:cell(landingAirport)="data">
      {{ data.value }}
    </template>

    <template v-slot:cell(takeoffTime)="data">
      {{ showTime(data.item.takeoffTime) }}
    </template>

    <template v-slot:cell(landingTime)="data">
      {{ showTime(data.item.landingTime) }}
    </template>

    <template v-slot:cell(duration)="data">
      {{ showAirborneTime(data.item) }}
    </template>

    <template v-slot:cell(landingCount)="data">
      {{ showTotalLandings(data.item) }}
    </template>

    <template v-slot:cell(props)="data">
        <b-badge variant="success" v-if="data.item.new"> New</b-badge>
        <b-badge variant="warning" v-if="data.item.duplicate">Duplicate</b-badge>
    </template>
       
    <template v-slot:cell(edit)="data">
        <b-button size="sm" @click="showModal(data.item, data.index)" class="mr-2">
        Edit
        </b-button>
        <b-button size="sm" @click="entryDelete(data.item, data.index)" class="mr-2">
        Delete
        </b-button>
    </template>


    <!-- Optional default data cell scoped slot -->
    <template v-slot:cell()="data">
      <i>{{ data.value }}</i>
    </template>
    </b-table>


      <b-modal
          size="lg"
          id="modal-edit-segment"
          ref="modal-edit-segment"
          title="Edit Flight Segment"
          ok-title="Submit"
          cancel-title="Discard"
          @ok="handleOk"
      >
      
    <b-form>
        <b-form-row>
            <b-form-group class="px-1" id="input-group-plane" label="Plane" label-for="input-plane">
               <b-form-input
                 id="input-plane"
                 v-model="form.registration"
                 :state="registrationState"
                  lazy-formatter
                 :formatter="formatterRegistration"
                 required
               ></b-form-input>
             </b-form-group>

            <b-form-group class="px-1" id="input-group-pilot" label="Pilot" label-for="input-pilot">
               <b-form-input
                 id="input-pilot"
                 v-model="form.pilot"
                 :state="pilotState"
                 lazy-formatter
                 :formatter="formatterPilot"
                 required
               ></b-form-input>
             </b-form-group>

             <b-form-group class="px-1" id="input-group-date" label="Date" label-for="input-date">
                 <b-form-datepicker
                     id="input-date"
                     v-model="form.date"
                     required
                     placeholder="Date"
                     :date-format-options="{ year: 'numeric', month: 'numeric', day: 'numeric' }"
                 ></b-form-datepicker>
             </b-form-group>
         </b-form-row>
    </b-form>
      
    <b-card class="mt-3" header="Flight Segment" bg-variant="light">
    
    <b-form-row>
        <b-form-group class="px-1" id="input-group-from" label="From" label-for="input-from">
            <b-form-input
               id="input-from"
               v-model="form.from"
               required
               placeholder="From"
             ></b-form-input>
       </b-form-group>

        <b-form-group class="px-1" id="input-group-offblock" label="Block Off" label-for="input-offblock">
            <b-form-timepicker
                id="input-offblock"
                v-model="form.offblock"
                required
                placeholder="Off Block Time"
            ></b-form-timepicker>
        </b-form-group>
       
       <b-form-group class="px-1" id="input-group-5" label="Takeoff" label-for="input-5">
           <b-form-timepicker
             v-model="form.takeoff"
             locale="de"
           ></b-form-timepicker>
         </b-form-group>
    
    <b-form-group class="px-1" id="input-group-hobbs-start" label="Hobbs Start" label-for="input-hobbs-start">
        <b-form-input
            type="number"
            id="input-hobbs-start"
            v-model="form.hobbsstart"
            required
            placeholder="Hobbs Start"
            step='0.01'
        ></b-form-input>
    </b-form-group>

    </b-form-row>

    <b-form-row>

      <b-form-group class="px-1" id="input-group-to" label="To" label-for="input-to">
         <b-form-input
           id="input-to"
           v-model="form.to"
           required
           placeholder="To"
         ></b-form-input>
       </b-form-group>

        <b-form-group class="px-1" id="input-group-onblock" label="Block On" label-for="input-onblock">
            <b-form-timepicker
                id="input-onblock"
                v-model="form.onblock"
                required
                placeholder="On Block Time"
            ></b-form-timepicker>
        </b-form-group>
    
        <b-form-group class="px-1" id="input-group-landing" label="Landing" label-for="input-landing">
            <b-form-timepicker
                id="input-landing"
                v-model="form.landing"
                required
                placeholder="Landing"
            ></b-form-timepicker>
        </b-form-group>

    
        <b-form-group class="px-1" id="input-group-hobbs-end"
            label="Hobbs End" label-for="input-hobbs-end">
              <b-form-input
                  type="number"
                  id="input-hobbs-end"
                  v-model="form.hobbsend"
                  required
                  placeholder="Hobbs End"
                  step='0.01'
              ></b-form-input>
        </b-form-group>

    </b-form-row>


    <b-form-row class="my-3">
      
    <b-form-group class="px-1" id="input-group-landings-day"
          label="Landings (Day)" label-for="input-landings-day">
            <b-form-input
                type="number"
                id="input-landings-day"
                v-model="form.landingcount"
                required
                placeholder="Landings Day"
                min="0" max="30"
            ></b-form-input>
        </b-form-group>
        
    <b-form-group classe="px-3" id="input-group-landings-night"
          label="Landings (Night)" label-for="input-landings-night">
            <b-form-input
                type="number"
                id="input-landings-night"
                v-model="form.nightlandings"
                required
                placeholder="0"
                min="0" max="30"
            ></b-form-input>
        </b-form-group>

    <b-form-group class="px-3" label="Flight Rules">
        <b-form-radio-group
            v-model="form.rules"
            :options=rules
            @change="changeRules"
        ></b-form-radio-group>
    </b-form-group>
    
    <b-form-group class="px-3" label="Function">
        <b-form-select
            v-model="form.function"
            :options=functions
        ></b-form-select>
    </b-form-group>

    <b-form-group class="px-3" id="timer-flight" label="Total Time of Flight" label-for="timer-flight">
        <b-form-timepicker
            id="timer-flight"
            v-model="form.totaltime"
            required
            placeholder="Flight Time"
            disabled
        ></b-form-timepicker>
    </b-form-group>
    
    </b-form-row>
    </b-card>
    
    <b-button v-b-toggle="'collapse-stats'" class="m-1">Statistics</b-button>
    <b-button v-b-toggle="'collapse-timers'" class="m-1">Timers</b-button>
    
    <b-collapse id="collapse-stats">
      <flight-stats v-bind:stats="form.stats" v-if="form.stats"> </flight-stats>
    </b-collapse>

    <b-collapse id="collapse-timers">
    
    <b-form-row class="my-3">
    <b-form-group class="px-3" id="timer-night" label="Night Time" label-for="timer-night">
        <b-form-timepicker
            id="timer-night"
            v-model="form.nighttime"
            required
            placeholder="Night Time"
        ></b-form-timepicker>
    </b-form-group>
    <b-form-group class="px-3" id="timer-ifr" label="IFR Time" label-for="timer-flight">
        <b-form-timepicker
            id="timer-ifr"
            v-model="form.ifrtime"
            required
            placeholder="IFR Time"
            @input="ifrTimeInput"
        ></b-form-timepicker>
    </b-form-group>
    <b-form-group class="px-3" id="timer-pic" label="PIC Time" label-for="timer-pic">
        <b-form-timepicker
            id="timer-pic"
            v-model="form.pictime"
            required
            placeholder="PIC Time"
            disabled
        ></b-form-timepicker>
    </b-form-group>
    <b-form-group class="px-3" id="timer-dual" label="DUAL Time" label-for="timer-dual">
        <b-form-timepicker
            id="timer-dual"
            v-model="form.dualtime"
            required
            placeholder="DUAL Time"
            disabled
        ></b-form-timepicker>
    </b-form-group>
    <b-form-group class="px-3" id="timer-fi" label="FI Time" label-for="timer-fi">
        <b-form-timepicker
            id="timer-fi"
            v-model="form.fitime"
            required
            placeholder="DUAL Time"
            disabled
        ></b-form-timepicker>
    </b-form-group>
    </b-form-row>
    </b-collapse>

    </b-modal>
  </div>
</template>

<script>

import FlUtils from '../flutils.js'
import FlightStats from './FlightStats.vue';
//Vue.component('flight-stats', FlightStats);

export default {
    name: 'Flight-Log-Table',
    components : {
        'flight-stats' : FlightStats
    },
    props: {
        debug : Number,
        flightSegments : Array,
        pilots: Array,
        planes: Array,
        rules: Array,
        functions: Array,
        },

    computed: {
        pilotState() {
            let p = this.form.pilot
            let l = p.length
            if (l <= 2 || l > 50) {
                return false
            }


            for(let i = 0; i < l; i++){
               if (FlUtils.validateCharPilot(p.charAt(i)) == false) {
                   return false
               } 
            }
            return true;
        },

        registrationState() {
            let p = this.form.registration
            let l = p.length
            if (l <= 2 || l >= 10) {
                return false
            }

            for(let i = 0; i < l; i++){
               if (FlUtils.validateCharRegistration(p.charAt(i)) == false) {
                   return false
               } 
            }
            return true;
        }
    },

    data() {
      return {
        functionoptions : [
                { text: 'PIC', value: 'PIC'},
                { text: 'DUAL', value: 'DUAL' },
                { text: 'FI', value: 'FI' },
        ],
        rulesoptions : [
                { text: 'VFR', value: 'VFR'},
                { text: 'IFR', value: 'IFR' }
        ],
        planeoptions : [
          { text: 'DEEBU', value: 'DEEBU'},
          { text: 'DEKAL', value: 'DEKAL'}
        ],
        form : {
            type: '',
            registration : '',
            pilot: '',
            date : '',
            from : '',
            to : '',
            offblock : '',
            takeoff : '',
            landing : '',
            onblock : '',
            duration : '',
            hobbsstart : '0.00',
            hobbsend : '0.00',
            landingcount : '',
            nightlandings : '',
            rules : '',
            function : '',
            totaltime : 0,
            
            ifrtime : '',
            ifrtime_s : 0,
            
            airbornetime : 0,
            pictime : 0,
            dualtime : 0,
            fitime : 0,
            nighttime : 0,
            stats : null
        },
        show : true,
        selectedItem: null,
        fields: [
          // A virtual column that doesn't exist in items
            'date',
            'plane',
            'pilot',
          { key: 'departureAirport', label: 'From' },
          { key: 'landingAirport', label: 'To' },
          { key: 'takeoffTime', label: 'Takeoff' },
          { key: 'landingTime', label: 'Landing' },
          { key: 'duration', label: 'Airborne Time'},
            'landingCount',
          { key: 'props', label : ''},
          { key: 'edit', label: ''}
        ]
      }
    },
    methods : {
        changeRules ( rules ){
            if (rules === "IFR"){
                var offBlock = new Date(this.form.date), onBlock = new Date(this.form.date)
                offBlock = this.setTimeFromForm(this.form.offblock, offBlock).getTime() / 1000
                onBlock = this.setTimeFromForm(this.form.onblock, onBlock).getTime() / 1000

                this.form.ifrtime_s = onBlock - offBlock
                this.form.ifrtime = FlUtils.showTime ( this.form.ifrtime_s )
            } else {
                console.log( rules )
                this.form.ifrtime_s = 0
                this.form.ifrtime = FlUtils.showTime ( this.form.ifrtime_s )
            }
        },
        
        formatterPilot(value){
            return FlUtils.formatterPilot(value)
        },

        formatterRegistration(value){
            return FlUtils.formatterRegistration(value)
        },

        showModal(item, index) {
            this.selectedItem=item
            this.index = index
            this.form.type = 'SEP'
            this.form.registration = item.plane
            this.form.pilot = item.pilot
            this.form.from = item.departureAirport
            this.form.to = item.landingAirport
            this.form.date = new Date (item.takeoffTime * 1000)
           
            this.form.offblock = FlUtils.showTime(item.offBlock)
            this.form.takeoff = FlUtils.showTime(item.takeoffTime)
            this.form.landing = FlUtils.showTime(item.landingTime)
            this.form.onblock = FlUtils.showTime(item.onBlock)
            this.form.landingcount = item.landingCount
            this.form.nightlandings = item.nightLandings
            this.form.duration = this.showAirborneTime(item)
            this.form.rules = item.rules
            this.form.function = item.function
            this.form.totaltime = FlUtils.showTime(item.onBlock - item.offBlock)
            
            this.form.nighttime = FlUtils.showTime(item.nightTime)
            this.form.ifrtime_s = item.ifrtime_s ? item.ifrtime_s :
                item.rules == "IFR" ? item.onBlock - item.offBlock : 0
            this.form.ifrtime = FlUtils.showTime(this.form.ifrtime_s)
            
            this.form.pictime = item.function === "PIC" ?
                FlUtils.showTime(item.onBlock - item.offBlock) : FlUtils.showTime(0)
            this.form.dualtime = item.function === "DUAL" ?
                FlUtils.showTime(item.onBlock - item.offBlock) : FlUtils.showTime(0)
            this.form.fitime = item.function === "FI" ?
                FlUtils.showTime(item.onBlock - item.offBlock) : FlUtils.showTime(0)
            
            this.form.stats = item.stats
            console.log(this.form.stats)
            this.$refs['modal-edit-segment'].show()
        },

        handleOk(evt) {
          var litem = this.selectedItem
          var lform = this.form

          var takeoff = lform.date
          //console.log("HALLO: " + takeoff)
          var year = takeoff.getUTCFullYear()
          var month = takeoff.getUTCMonth()
          var day = takeoff.getUTCDate()
          
          var offBlock, takeoffTime, landingTime, onBlock;

          offBlock = takeoffTime = landingTime = onBlock =
            new Date(Date.UTC(year, month, day));

          //console.log("DATE: " + offBlock )
          litem.plane = lform.registration
          litem.type = lform.type
          litem.pilot = lform.pilot
          litem.departureAirport = lform.from
          litem.landingAirport = lform.to
          
          
          litem.offBlock = this.setTimeFromForm(lform.offblock, offBlock).getTime() / 1000
          litem.takeoffTime = this.setTimeFromForm(lform.takeoff, takeoffTime).getTime()/1000
          litem.landingTime = this.setTimeFromForm(lform.landing, landingTime).getTime() / 1000
          litem.onBlock = this.setTimeFromForm(lform.onblock, onBlock).getTime() / 1000


          litem.landingCount = lform.landingcount
          litem.nightLandings = lform.nightlandings
          litem.rules = lform.rules
          litem.function = lform.function
          litem.ifrtime_s = lform.ifrtime_s
          litem.nighttime = lform.nighttime
          this.$parent.entrySave (litem, this.index)

          //alert(JSON.stringify(this.selectedItem))
        },
        
        getSecondsfromForm(form){
                var secs = parseInt(form.slice(0,2))*3600 + parseInt(form.slice(3,5)) * 60
                return secs
        },
        
        ifrTimeInput(data) {
            this.form.ifrtime_s = this.getSecondsfromForm(data)
        },
        
        setTimeFromForm(form, item){
          
          //console.log("SET: " + item)
          item.setUTCHours(form.slice(0,2))
          item.setUTCMinutes(form.slice(3,5))
          
          return item
        },

        entryDelete(item, index)
        {
          this.$parent.entryDelete(item,index)
        },
        
        // some utility functions
        showDate : function (timer){
            return FlUtils.showDate(timer)
        },

        showTime : function (timer){
            return FlUtils.showTime( timer )
        },
        
        showTimeSeconds : function (timer){
                return timer
        },

        showAirborneTime : function (line) {
            var duration = line.landingTime - line.takeoffTime
            return FlUtils.showTime(duration)
        },

        showTotalLandings : function (line) {
            return Number(line.landingCount) + Number(line.nightLandings)
        },

        showTotalFlightTime : function (line) {
            var duration = line.onBlock - line.offBlock
            return FlUtils.showTime(duration)
        }


    }
  }
</script>
