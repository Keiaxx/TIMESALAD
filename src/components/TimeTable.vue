<template>
  <v-container fluid>
    <v-slide-y-transition mode="out-in">
        <v-card>
          <v-card-title>
            Time Clock Log
            <v-spacer></v-spacer>
            <v-btn icon @click.native="dialog = true">
              <v-icon>picture_as_pdf</v-icon>
            </v-btn>
            <v-btn icon @click.native="addTimeDialog = true">
              <v-icon>add_circle</v-icon>
            </v-btn>
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
                  :items="timesBackwards"
                  :search="search"
          >
            <template slot="items" slot-scope="props">
              <td>{{ props.item.started | moment('MMM D, YYYY hh:mm:ss A')}}</td>
              <td>{{ props.item.stopped === "" ? "Working" : props.item.stopped | moment('MMM D, YYYY hh:mm:ss A')}}</td>
              <td>{{calcDiff(props.item.started, props.item.stopped)}}</td>
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


    <v-dialog v-model="addTimeDialog" persistent max-width="300px">
      <v-card>
        <v-card-title>
          <span class="headline">Add Time</span>
        </v-card-title>
        <v-card-text>

          <v-container grid-list-md>
            <v-layout row wrap>
              <!--begin date-->
              <v-flex xs12 lg6>
                  <v-datetime-picker
                          label="Select Start Time"
                          v-model="startDateTime12">
                  </v-datetime-picker>
              </v-flex>
              <!--end date-->
              <v-flex xs12 lg6>
                <v-datetime-picker
                        label="Select End Time"
                        v-model="endDateTime12">
                </v-datetime-picker>
              </v-flex>
            </v-layout>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" flat @click.native="addTimeDialog = false">Close</v-btn>
          <v-btn color="blue darken-1" flat @click.native="addTime">Add Time</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialog" persistent max-width="800px">
      <v-card>
        <v-card-title>
          <span class="headline">Export Timesheet PDF: Date Range</span>
        </v-card-title>
        <v-card-text>
          <v-container grid-list-md>
            <v-layout row wrap>
              <!--begin date-->
              <v-flex xs12 lg6>
                <v-text-field
                        v-model="preferredname"
                        label="Your Name"
                ></v-text-field>
              </v-flex>
              <v-flex xs12 lg6>
                <v-menu
                        :close-on-content-click="false"
                        v-model="menu1"
                        :nudge-right="40"
                        lazy
                        transition="scale-transition"
                        offset-y
                        full-width
                        max-width="290px"
                        min-width="290px"
                >
                  <v-text-field
                          slot="activator"
                          v-model="beginDateFormat"
                          label="Begin Date"
                          hint="MM/DD/YYYY format"
                          persistent-hint
                          prepend-icon="event"
                          readonly
                  ></v-text-field>
                  <v-date-picker v-model="date1" no-title @input="menu1 = false"></v-date-picker>
                </v-menu>
              </v-flex>
              <!--end date-->
              <v-flex xs12 lg6>
                <v-menu
                        :close-on-content-click="false"
                        v-model="menu2"
                        :nudge-right="40"
                        lazy
                        transition="scale-transition"
                        offset-y
                        full-width
                        max-width="290px"
                        min-width="290px"
                >
                  <v-text-field
                          slot="activator"
                          v-model="endDateFormat"
                          label="End Date"
                          hint="MM/DD/YYYY format"
                          persistent-hint
                          prepend-icon="event"
                          readonly
                  ></v-text-field>
                  <v-date-picker v-model="date2" no-title @input="menu2 = false"></v-date-picker>
                </v-menu>
              </v-flex>

            </v-layout>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" flat @click.native="resetDateRange">Close</v-btn>
          <v-btn color="blue darken-1" flat @click.native="exportPdf(date1, date2)">Export</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="pdfdialog" fullscreen hide-overlay transition="dialog-bottom-transition">

      <v-card>

        <v-toolbar color="primary">
          <v-btn icon @click.native="pdfdialog = false">
            <v-icon class="white--text">close</v-icon>
          </v-btn>
          <v-toolbar-title class="white--text">{{openedfilename}}</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn flat @click.native="pdfdialog = false" class="white--text">Close</v-btn>
          </v-toolbar-items>
        </v-toolbar>

        <vue-pdf-viewer :url='pdfurl' style="height: 90vh;">
        </vue-pdf-viewer>
      </v-card>
    </v-dialog>

  </v-container>
</template>

<script>
    const moment = require('moment')
    import axios from 'axios';
    import VuePDFViewer from 'vue-instant-pdf-viewer'

    export default {
        components: {
            'vue-pdf-viewer': VuePDFViewer
        },
        data () {
            return {

                pdfdialog: false,
                search: '',
                headers: [
                    {
                        text: 'Time Started',
                        sortable: false,
                        value: 'started'
                    },
                    {
                        text: 'Time Stopped',
                        sortable: false,
                        value: 'stopped'
                    },
                    {
                        text: 'Hours (Decimal)',
                        sortable: false,
                        value: 'difference'
                    }, {
                    text: "Actions",
                        sortable: false,
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
                text: 'Hello, I\'m a snackbar',
                beginDate: null,
                endDate: null,
                date1: null,
                date2: moment().format('YYYY-MM-DD'),
                dateFormatted: null,
                appVersion: "v1.0",
                menu1: false,
                menu2: false,
                openedfilename: null,
                dialog: false,
                dialogloading: false,
                valueDeterminate: 0,
                pdfurl: 'https://bitcoin.org/bitcoin.pdf',
                preferredname: '',
                addTimeDialog: false,
                startDateTime12: moment().toDate(),
                endDateTime12: moment().toDate(),
                startDateTimeMenu: false,
                endDateTimeMenu: false
            }
        },
        methods: {
            addTime() {
                if(!this.startDateTime12){
                    alert("You must specify a start datetime")
                    return;
                }

                if(!this.endDateTime12){
                    alert("You must specify a end datetime")
                    return;
                }

                var logitem = {
                    started: moment(this.startDateTime12),
                    stopped: moment(this.endDateTime12),
                    difference: ""
                };

                this.times.unshift(logitem);
                this.addTimeDialog = false;
            },
            calcDiff(start, end) {
                return (moment(end).diff(moment(start), 'seconds')/60/60).toFixed(2)
            },
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

                lastitem.stopped = moment();
                this.times[0] = lastitem;

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
                        localStorage.times = JSON.stringify(response.data);
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
            resetDateRange() {
                this.date1 = this.beginDate;
                this.date2 = this.endDate;
                this.dialog = false;
            },
            dateFormatUTC(rawDate) {
                var date_Raw = new Date(Date.parse(rawDate));
                var formattedDate = (date_Raw.getUTCMonth() + 1) + "-" + (date_Raw.getUTCDate()) + "-" + date_Raw.getUTCFullYear();
                return formattedDate;
            },
            formatDate(date) {
                if (!date) return null

                const [year, month, day] = date.split('-')
                return `${month}/${day}/${year}`
            },
            parseDate(date) {
                if (!date) return null

                const [month, day, year] = date.split('/')
                return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
            },
            exportPdf(date1, date2) {
                if(!this.times.length > 0){
                    alert("Nothing to export")
                    return;
                }

                if(!date1){
                    alert("You must choose a begin date!")
                    return;
                }

                if(!date2){
                    alert("You must choose an end date!")
                    return;
                }

                var start = moment(date1, 'YYYY-MM-DD');
                var end = moment(date2, 'YYYY-MM-DD').endOf('day');

                this.times.sort(function (left, right) {
                    return moment.utc(left.started).diff(moment.utc(right.started))
                });

                var filteredTimes = this.times.filter(function(item) {
                    var current = moment(item.started);
                    return (current > start && current < end);
                });

                console.log(filteredTimes);


                let transformedMRData = filteredTimes.map((textRow) => {
                    return {
                        'memberName': textRow.started,
                        'memberEmail': textRow.stopped,
                        'loanUnits': new Big(moment(textRow.stopped).diff(moment(textRow.started), "seconds")/60/60)
                    };
                });


                const asMoney = (rawMoney) => rawMoney.round(2, 1).toFixed(2).toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, "$1,");


                const makeCell = (content, rowIndex = -1, options = {}) => {
                    return Object.assign({text: content, fillColor: rowIndex % 2 ? 'white' : '#e8e8e8'}, options);
                }

                const thl = (content, rowIndex = -1, options = {}) => {
                    return makeCell(content, rowIndex, Object.assign({ bold: true, alignment: 'left', fontSize: 9 }, options));
                }
                const thr = (content, rowIndex = -1, options = {}) => {
                    return makeCell(content, rowIndex, Object.assign({ bold: true, alignment: 'right', fontSize: 9 }, options));
                }
                const tdl = (content, rowIndex = -1, options = {}) => {
                    return makeCell( content, rowIndex, Object.assign({ bold: false, alignment: 'left', fontSize: 9 }, options));
                }
                const tdr = (content, rowIndex = -1, options = {}) => {
                    return makeCell( content, rowIndex,Object.assign({ bold: false, alignment: 'right', fontSize: 9 }, options));
                }

// -- Set the report date for display only.
                const reportDate = () => moment().format('MMMM Do YYYY')

// --  Calculate the total of the current balances.
                const sumCurrentBalance = (data) => {
                    return asMoney(data.reduce((sum, current) => sum.plus(current['currentBalance']), new Big(0.0)));
                }

                const truncateContent = (content, maxLength = 17) => {
                    return ''.concat(content.slice(0, maxLength), content.length > maxLength ? '…' : '');
                }


// -- Create a base document template for the reports.
                const createDocumentDefinition = (reportDate, subHeading, ...contentParts) => {
                    const baseDocDefinition = {
                        pageSize: 'A4',
                        footer: (currentPage, pageCount) => {
                            return {
                                text: `${reportDate} : Page ${currentPage.toString()} of ${pageCount.toString()} generated by timeclock.gose.pw ${this.appVersion}`,
                                alignment: 'center',
                                fontSize: 7
                            }
                        },

                        styles: {
                            title: {
                                fontSize: 20
                            },
                            titleSub: {
                                fontSize: 18
                            },
                            titleDate: {
                                fontSize: 14,
                                alignment: 'right',
                                bold: false
                            }
                        },

                        content: [
                            {
                                columns: [
                                    {text: this.preferredname + ' Time Sheet', style: 'title', width: '*'},
                                    {text: "Approved By: _________________", style: 'titleDate', width: '200'},
                                ]
                            },
                            {text: moment(start).format("MMM Do YYYY")+" to "+moment(end).format("MMM Do YYYY")+"\n\n", style: 'titleSub'},
                        ],
                    };
                    const docDefinition = JSON.parse(JSON.stringify(baseDocDefinition));
                    docDefinition.footer = baseDocDefinition.footer;
                    docDefinition.content.push(...contentParts);
                    return docDefinition;
                };



                // -- Generate the body of the document table, with headings
                const tableBody = (dataRows) => {
                    const body = [
                        [
                            thl(' ', -1, {colSpan: 2}),
                            thl(' '),
                            thr('Hours (Decimal Hours)'),
                        ]
                    ];

                    let cumLoanUnits = new Big("0.0");
                    let cumCashUnits = new Big("0.0");
                    let cumTotalUnits = new Big("0.0");

                    dataRows.forEach((row, index) => {
                        const tableRow = [];
                        tableRow.push(tdl(moment(row['memberName']).format('MMMM Do YYYY, h:mm:ss a'),  index));
                        tableRow.push(tdl(moment(row['memberEmail']).format('MMMM Do YYYY, h:mm:ss a'), index));
                        tableRow.push(tdr(asMoney(row['loanUnits']), index));
                        body.push(tableRow);

                        cumLoanUnits = cumLoanUnits.plus(row['loanUnits']);
                    });
                    body.push([
                        tdl(`Total hours (Decimal Hours)`, -1, {colSpan: 2, fillColor: 'black', color: 'white'}),
                        tdl(' '),
                        tdr(asMoney(cumLoanUnits), -1, {fillColor: 'black', color: 'white'}),
                    ]);
                    return body;
                }

                // -- The main report table, with the table body.
                const tableData = {
                    table: {
                        headerRows: 1,
                        widths: [200, 200, '*'],

                        body: tableBody(transformedMRData),
                    }
                };
                const docDefinition = createDocumentDefinition(reportDate(), 'Member Register', tableData);
                var _this = this;
                pdfMake.createPdf(docDefinition).getDataUrl(function(dataURL) {
                    _this.pdfurl = dataURL;
                    _this.pdfdialog = true;
                    _this.dialog = false;
                });
            },
            openPDF(name) {
                this.openedfilename = name
                this.dialogloading = true
                this.valueDeterminate = 0

                this.$http.get('/panel/review/' + name, {
                    responseType: 'arraybuffer',
                    headers: {
                        'Authorization': localStorage.getItem('jwtToken')
                    },
                    onDownloadProgress: function (progressEvent) {
                        let percentCompleted = Math.round((progressEvent.loaded * 100) / progressEvent.total)
                        this.updateProgressBarValue(percentCompleted)
                    }.bind(this)
                })
                    .then(response => {
                        // Create a Blob from the PDF Stream

                        const file = new Blob(
                            [response.data], {
                                type: 'application/pdf'
                            })
                        // Build a URL from the file

                        const fileURL = window.URL.createObjectURL(file)
                        // Open the URL on new Window
                        this.pdfurl = fileURL
                        this.dialogloading = false
                        this.dialog = true
                    })
                    .catch(e => {
                        console.log(e)
                        this.showSnackbar = true
                        this.snackMessage = 'Error getting pdf list!'
                        this.dialogloading = false
                    })
            }




        },
        computed: {
            // a computed getter
            timesBackwards() {
                this.times.sort(function (right, left) {
                    return moment.utc(left.started).diff(moment.utc(right.started))
                });

                return this.times;
            },
            beginDateFormat() {
                return this.formatDate(this.date1)
            },
            endDateFormat() {
                return this.formatDate(this.date2)
            },

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

            if(localStorage.preferredname){
                this.preferredname = localStorage.preferredname;
            }

            console.log("AUTO UPLOAD: " + this.autoupload)
        },
        watch: {
            myjsonid: function(val){
                console.log(val);
                localStorage.myjsonid = val;
            },
            preferredname: function(val){
                console.log(val);
                localStorage.preferredname = val;
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
