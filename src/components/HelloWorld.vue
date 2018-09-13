<template>
  <v-container fluid>
    <v-slide-y-transition mode="out-in">
        <v-card>
          <v-card-title>
            Time Clock Log
            <v-spacer></v-spacer>
            <v-text-field
                    v-model="search"
                    append-icon="search"
                    label="Search"
                    single-line
                    hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table
                  :headers="headers"
                  :items="times"
                  :search="search"
          >
            <template slot="items" slot-scope="props">
              <td>{{ props.item.started | moment('MMM D, YYYY hh:mm:ss A')}}</td>
              <td>{{ props.item.stopped === "" ? "Working" : props.item.stopped | moment('MMM D, YYYY hh:mm:ss A')}}</td>
              <td>{{ props.item.difference/60/60 }}</td>
              <td><v-icon
                      small
                      @click="deleteItem(props.item)"
              >
                delete
              </v-icon></td>
            </template>
            <v-alert slot="no-results" :value="true" color="error" icon="warning">
              Your search for "{{ search }}" found no results.
            </v-alert>
          </v-data-table>

          <v-btn block color="green" v-if="!isClockedIn" v-on:click="handleClockEvent" dark>Clock In</v-btn>
          <v-btn block color="red" v-else v-on:click="handleClockEvent" dark>Clock Out</v-btn>

          <v-layout justify-center row wrap ma-2>
            <v-flex xs12 sm6 md5 ma-2>
              <v-btn block color="blue"  v-on:click="exportToFile" dark>Export .TIMESALAD</v-btn>

            </v-flex>

            <v-flex xs12 sm6 md5 ma-2>

              <v-btn block color="blue"  v-on:click="clickFileImport" dark>Import .TIMESALAD</v-btn>

            </v-flex>
          </v-layout>




          <v-layout justify-center row wrap ma-2>

            <v-flex xs12 sm6 md3 ma-2>
              <v-btn block color="blue"  v-on:click="uploadOrUpdate" dark>Upload Data</v-btn>
            </v-flex>

            <v-flex xs12 sm6 md3>
              <v-text-field
                      v-model="myjsonid"
                      label="Unique Key"
              ></v-text-field>
            </v-flex>

            <v-flex xs12 sm6 md3 ma-2>
              <v-btn block color="blue"  v-on:click="loadFromMyJSON" dark>Download Data</v-btn>

            </v-flex>


          </v-layout>

          <v-layout justify-center row wrap ma-2>
          <v-flex xs12 sm6 md1>
            <v-switch v-model="autoupload" @change="autoUploadChanged" label="Auto Synchronize"></v-switch>

          </v-flex>
          </v-layout>


          <!-- REAL FILE INPUT (HIDDEN) -->
          <input type="file" id="file" ref="myFiles" @change="importFile" style="display:none" />
        </v-card>
    </v-slide-y-transition>

    <v-snackbar
            v-model="snackbar"
            :bottom="y === 'bottom'"
            :left="x === 'left'"
            :multi-line="mode === 'multi-line'"
            :right="x === 'right'"
            :timeout="timeout"
            :top="y === 'top'"
            :vertical="mode === 'vertical'"
    >
      {{ text }}
      <v-btn
              color="pink"
              flat
              @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>
  </v-container>
</template>

<script>
    const moment = require('moment')
    import axios from 'axios';

    export default {
        data () {
            return {
                search: '',
                headers: [
                    {
                        text: 'Time Started',
                        sortable: false,
                        value: 'started'
                    },
                    {
                        text: 'Time Stopped',
                        sortable: true,
                        value: 'stopped'
                    },
                    {
                        text: 'Hours (Decimal)',
                        sortable: true,
                        value: 'difference'
                    }, {
                    text: "Actions",
                    }
                ],
                times: [],
                files: [],
                myjsonid: "",
                autoupload: false,
                snackbar: false,
                y: 'top',
                x: 'right',
                mode: 'multi-line',
                timeout: 5000,
                text: 'Hello, I\'m a snackbar'
            }
        },
        methods: {
            autoUploadChanged() {
              console.log("Setting auto upload")
                if(this.autoupload){
                    localStorage.autoupload = "yes"
                }else{
                    localStorage.autoupload = "no"
                }
            },
            saveLocalStorage() {
                console.log("Saving Local Storage");
                localStorage.times = JSON.stringify(this.times);

                if(this.autoupload){
                    this.uploadOrUpdate();
                }
            },
            clickFileImport: function() {
                this.$refs.myFiles.click();
            },
            importFile: function() {
                // Check for the various File API support.
                if (window.File && window.FileReader && window.FileList && window.Blob) {
                    // Great success! All the File APIs are supported.
                    console.log("FILE API SUPPORTED");

                    var fr = new FileReader();
                    var vue_this = this;

                    fr.onload = function (e) {
                        console.log(e.target.result)

                        try {
                            vue_this.times = JSON.parse(e.target.result);
                        }catch(e){
                            alert('The .TIMESALAD file you imported is unreadable!');
                        }

                        vue_this.saveLocalStorage();
                    };

                    var extension = this.$refs.myFiles.files[0].name.split('.').pop().toLowerCase();  //file extension from input file

                    if (extension === "timesalad") {
                        fr.readAsText(this.$refs.myFiles.files[0]);
                    }else{
                        alert('You must import .TIMESALAD files.');
                    }


                } else {
                    alert('The File APIs are not fully supported in this browser.');
                }
            },
            exportToFile:function() {
                if(!this.times.length > 0){
                    alert("Nothing to export")
                    return;
                }

                const date = moment().format("MMM_Do_YY");

                const data = JSON.stringify(this.times)
                const blob = new Blob([data], {type: 'text/plain'})
                const e = document.createEvent('MouseEvents'),
                    a = document.createElement('a');
                a.download = "timeclock_"+date+".timesalad";
                a.href = window.URL.createObjectURL(blob);
                a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
                e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
                a.dispatchEvent(e);
                this.saveLocalStorage();
            },
            deleteItem (item) {
                const index = this.times.indexOf(item)
                confirm('Are you sure you want to delete this item?') && this.times.splice(index, 1)
                this.saveLocalStorage();
            },
            clockIn: function() {
                var r = confirm("Are you sure you want to clock in?!");
                if (!r) {
                    return;
                }

                var logitem = {
                    started: moment(),
                    stopped: "",
                    difference: ""
                }
                this.times.unshift(logitem);
                this.saveLocalStorage();
            },
            clockOut: function() {
                var r = confirm("Are you sure you want to clock out?!");
                if (!r) {
                    return;
                }

                var lastitem = this.times[0];
                lastitem.started = lastitem.started;

                lastitem.stopped = moment();
                this.times[this.times.length-1] = lastitem;

                lastitem.difference = lastitem.stopped.diff(lastitem.started, "seconds")
                this.saveLocalStorage();

            },
            handleClockEvent: function () {
                // `this` inside methods point to the Vue instance

                if(this.times.length > 0){
                    var lastitem = this.times[0];
                    if(lastitem.stopped){
                        //User has clocked out
                        this.clockIn();
                    }else{
                        //User is still working
                        this.clockOut();
                    }
                }else{
                    this.clockIn();
                }
            },
            uploadOrUpdate: function(){
                if(!this.times.length > 0){
                    this.showSnack("Nothing to upload")
                    return;
                }

                axios.put('https://api.myjson.com/bins/'+this.myjsonid, this.times)
                    .then(response => {
                        // JSON responses are automatically parsed.
                        console.log(response.data);


                        this.showSnack("Upload success!")
                    })
                    .catch(e => {
                        axios.post('https://api.myjson.com/bins', this.times)
                            .then(response => {
                                // JSON responses are automatically parsed.
                                console.log(response.data);

                                var createdid = response.data.uri.split("bins/")[1];

                                this.myjsonid = createdid;

                                this.showSnack("Uploaded! Unique Key is: "+this.myjsonid+" . Keep this key to load your records from the cloud!");
                            })
                            .catch(e => {
                                this.showSnack("Error uploading!")
                            })
                    })
            },
            loadFromMyJSON: function() {
                if(!this.myjsonid){
                    this.showSnack("You must enter a Unique Key!")
                }

                axios.get('https://api.myjson.com/bins/'+this.myjsonid)
                    .then(response => {
                        // JSON responses are automatically parsed.
                        console.log(response.data);
                        this.times = response.data;
                        this.showSnack("Loaded data from cloud");
                    })
                    .catch(e => {
                        this.showSnack("Unable to load that Unique Key!")
                    })
            },
            showSnack(message) {
                this.text = message;
                this.snackbar  = true;
            },
        },
        computed: {
            // a computed getter

            isClockedIn: function () {
                // `this` points to the vm instance
                if(this.times.length > 0){
                    var lastitem = this.times[0];
                    if(lastitem.stopped){
                        //User has clocked out
                        return false;
                    }else{
                        //User is still working
                        return true;
                    }
                }else{
                    return false;
                }
            }
        },
        mounted: function() {
            if (localStorage.times) {
                this.times = JSON.parse(localStorage.times);
            }
            if (localStorage.myjsonid) {
                this.myjsonid = localStorage.myjsonid;

                if(localStorage.autoupload === "yes"){
                    this.autoupload = true;
                    this.loadFromMyJSON();
                }else{
                    this.autoupload = false;
                }

            }

            console.log("AUTO UPLOAD: " + this.autoupload)
        },
        watch: {
            myjsonid: function(val){
                console.log(val);
                localStorage.myjsonid = val;
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
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
  color: #42b983;
}
</style>
