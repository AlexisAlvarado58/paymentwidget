<template>
    <v-app>
        <div class="pa-5">
            <v-form
                lazy-validation
                ref="form">
                <v-select
                outlined
                dense
                label="Type of Payment"
                :items="PaymentTypes"
                item-text="text"
                item-value="value"
                @change="renderTable()"
                v-model="type"
                ></v-select>
                <v-data-table
                id="element-to-convert"
                :items-per-page="20"
                :headers="headers"
                :items="items"
                class="elevation-10 tableCss"
                hide-default-footer
                disable-pagination
                >
                <template v-slot:top>
                <v-toolbar class="toolbarr"  dense flat>
                    <h3>
                        Processing Schedule  <a color="orange"> {{ recordName }}</a>
                    </h3>
                    <v-spacer></v-spacer>
                        <vue-json-to-csv
                            :v-if="showExport"
                            :json-data="newArray"
                            :labels="headersObj"
                            :csv-title="'Processing Schedule '+recordName">
                            <v-btn :disabled="!showExport" color="success" class="mr-6 teal button__custom">
                            Export <i class="mdi mdi-export-variant" aria-hidden="true"></i>
                            </v-btn>
                        </vue-json-to-csv>
                </v-toolbar>
                </template>
                  <!------------------------------------------------------------ Period Number # ------------------------------------------------------------------- -->
                <template v-slot:item.id="{item}">
                    <p>{{ item.id }}</p>
                </template>
                <!------------------------------------------------------------ Period Endind Date Field-------------------------------------------------------------- -->
                <template v-slot:item.periodEnd="{item}">
                    <v-menu
                        v-model="item.menu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field 
                            dense
                            v-model="item.periodEnd"
                            label=""
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                        v-model="item.periodEnd"
                        @change="changePeriod(item,item.periodEnd)"
                        @input="item.menu = false"
                        ></v-date-picker>
                    </v-menu>
                </template>
                <!-- -------------------------------------------Client Cut ------------------------------------------------------------------------------------ -->
                <template v-slot:item.clientCut="{item}">
                    <v-menu
                        v-model="item.menu2"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field dense
                            :rules="[rules.required]"
                            v-model="item.clientCut"
                            label=""
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                        v-model="item.clientCut"
                        @input="item.menu2 = false"
                        ></v-date-picker>
                    </v-menu>
                </template>
                <!------------------------------------------------------------ Funds to be in OPCS-------------------------------------------------------------- -->
                <template v-slot:item.FundTo="{item}">
                    <v-menu
                        v-model="item.menu3"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field dense
                            v-model="item.FundTo"
                            :rules="[rules.required]"
                            label=""
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                        v-model="item.FundTo"
                        @input="item.menu3 = false"
                        ></v-date-picker>
                    </v-menu>
                </template>
                 <!------------------------------------------------------------ Run Finals------------------------------------------------------------------------- -->
                <template v-slot:item.runFinals="{item}">
                    <v-text-field
                    v-model="item.runFinals"
                    label=""
                    ></v-text-field>
                </template>
                 <!------------------------------------------------------------ Finalize Payroll------------------------------------------------------------------------- -->
                 <template v-slot:item.finalize="{item}">
                    <v-menu
                        v-model="item.menu5"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field dense
                            v-model="item.finalize"
                            :rules="[rules.required]"
                            label=""
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                        v-model="item.finalize"
                        @input="item.menu5 = false"
                        ></v-date-picker>
                    </v-menu>
                </template>

                <!------------------------------------------------------------ PMT Date------------------------------------------------------------------------- -->
                <template v-slot:item.pmtDate="{item}">
                    <v-menu
                        v-model="item.menu4"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field dense
                            v-model="item.pmtDate"
                            :rules="[rules.required]"
                            label=""
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                        v-model="item.pmtDate"
                        @change="changePMT(item,item.pmtDate)"
                        @input="item.menu4 = false"
                        ></v-date-picker>
                    </v-menu>
                </template>
                <!-- --------------------------------------------------------Comments------------------------------------------------------------------------------------ -->
                <template v-slot:item.comments="{item}">
                    <v-text-field
                    v-model="item.comments"
                    label=""
                    ></v-text-field>
                </template>
                </v-data-table>
                <div class="text-center pt-5">
                    <v-btn
                    color="orange"
                    @click="saveData()"
                    width="400"
                    :loading="loadingBtn"
                    >
                        Save
                    </v-btn>
                </div>
            </v-form>
        </div>
        <v-snackbar
            v-model="snackbar"
            :timeout="2000"
            color="green"
            elevation="24"
            >
            <strong> Dates Saved Correctly</strong>.
        </v-snackbar>
    </v-app>
</template>
<script>
import moment from 'moment';
import html2pdf from "html2pdf.js";
import VueJsonToCsv from 'vue-json-to-csv'

export default {
    components: {
        VueJsonToCsv,
     },
    data(){
        return{
            recordID:'',
            showExport: false,
            recordName: '',
            snackbar: false,
            PaymentTypes: [
                {text:'Semi-Monthly',value: 24},
                {text:'Bi-Weekly',value: 26},
                {text:'Monthly',value: 12}
            ],
            type:'',
            selected: '',
            payment_type: '',
            arrayAUX: [],
            items: [],
            headersObj:{
            "Period_No": {title: 'Period #'},
            "Period_Ending_Date":{title: 'Period Ending Date'},
            "Client_Cut_Off":{title: 'Client Cut-Off 10:00 am AST & Client receives invoice'},
            "Funds_To_be_in_OPC_S_Act":{title: 'Funds to be in BTI ACT'},
            "Run_Finals":{title: 'Run Finals'},
            "Finalize_Payroll":{title: 'Finalize Payroll'},
            "Pmt_Date":{title: 'Pmt Date'},
            "Comments":{title: 'Comments'},
            },
            newArray:[],
            headers:[
                { text: 'Period #', value: 'id', sortable: false, class: 'FixCols'},
                { text: 'Period Ending Date',value: 'periodEnd', sortable: false},
                { text: 'Client Cut-Off 10:00 am AST & Client receives invoice', value: 'clientCut'},
                { text: 'Funds to be in BTI ACT', value: 'FundTo'},
                { text: 'Run Finals',value:'runFinals'},
                { text: 'Finalize Payroll', value:'finalize'},
                { text: 'Pmt Date',value: 'pmtDate'},
                { text: 'Comments', value: 'comments'}
            ],
            loadingBtn: false,
            periodNum: 0,
            rules:{
                required: value => !!value || 'Field Can´t be empty',
            }
        }
    },
    async  mounted(){
        const that = this;
        await ZOHO.embeddedApp.on("PageLoad",function(data){      
            that.recordID = data.EntityId[0];   
            ZOHO.CRM.UI.Resize({height:"100vh",width:"1200"})    
            ZOHO.CRM.API.getRecord({Entity:"Accounts",RecordID:that.recordID})
                .then(function(data){
                    let hasPaymentSchedule = data.data[0].Processing_Schedule_Data;
                    that.recordName = data.data[0].Account_Name;
                    that.periodNum = data.data[0].Period_Position;
                    if(that.periodNum == null){
                        that.periodNum = 0;
                    }
                if(hasPaymentSchedule.length == 0){
                        that.type = 26;
                        for (let index = 0; index < that.type; index++) {
                            that.items.push(
                                {"id": index+1,"menu":false,"menu2":false,"menu3":false,"menu4":false,"menu5":false,"finalize":"","periodEnd": "","clientCut":  "","FundTo": "","pmtDate": "","comments":"","runFinals":""}
                            )
                        }
                    }else{
                        if(hasPaymentSchedule.length == 26){
                            that.type = 26;
                        }else if(hasPaymentSchedule.length == 24){
                            that.type = 24;
                        }else if(hasPaymentSchedule.length == 12){
                            that.type = 12;
                        }
                        hasPaymentSchedule.forEach(element => { 
                            that.items.push(
                                {"id": element.Period_No,"menu":false,"menu2":false,"menu3":false,"menu4":false,"menu5":false,"finalize":element.Finalize_Payroll,"periodEnd": element.Period_Ending_Date,"clientCut": element.Client_Cut_Off,"FundTo": element.Funds_To_be_in_OPC_S_Act,"pmtDate": element.Pmt_Date,"comments":element.Comments,"runFinals":element.Run_Finals}
                            )  
                        });
                    }
                    that.newArray = [];
                    that.items.forEach(element => {
                        var subFormData = {
                            "Period_No": element.id.toString(),
                            "Period_Ending_Date":element.periodEnd,
                            "Client_Cut_Off":element.clientCut,
                            "Funds_To_be_in_OPC_S_Act":element.FundTo,
                            "Run_Finals":element.runFinals,
                            "Finalize_Payroll":element.finalize,
                            "Pmt_Date":element.pmtDate,
                            "Comments":element.comments,
                        }
                        that.newArray.push(subFormData);
                        });      
                })
        })
        ZOHO.embeddedApp.init();
   },
    methods:{
        Print(){
            var element = document.getElementById("element-to-convert");
            var opt = {
                margin: 10,
                filename: "Processing Schedule"+this.recordName+".pdf",
                //  image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2 },
                // jsPDF:        { unit: 'in', format: 'letter', orientation: 'portrait' }
            }
            html2pdf().set(opt).from(element).save();
        },
       async saveData(){
        if(this.$refs.form.validate()){
            const that = this;
                this.loadingBtn = true;
                that.newArray = [];
                this.items.forEach(element => {
                    var subFormData = {
                        "Period_No": element.id.toString(),
                        "Period_Ending_Date":element.periodEnd,
                        "Client_Cut_Off":element.clientCut,
                        "Funds_To_be_in_OPC_S_Act":element.FundTo,
                        "Run_Finals":element.runFinals,
                        "Finalize_Payroll":element.finalize,
                        "Pmt_Date":element.pmtDate,
                        "Comments":element.comments,
                    }
                    this.newArray.push(subFormData);
                });
                if(this.type == 26){
                    this.payment_type = "Bi-Weekly"
                }else if(this.type == 24){
                    this.payment_type = "Semi-Monthly"
                }else if(this.type == 12){
                    this.payment_type = "Monthly"
                }
                var config={
                    Entity:"Accounts",
                    APIData:{
                        "id": this.recordID,
                        "Processing_Schedule_Data": this.newArray,
                        "Client_Cut_Off": this.items[this.periodNum].clientCut,
                        "Finalize_Payroll": this.items[this.periodNum].finalize,
                        "Funds_To_be_in_BTI_Act": this.items[this.periodNum].FundTo,
                        "Pmt_Date": this.items[this.periodNum].pmtDate,
                        "Period_Position": this.periodNum,
                        "Period_No": this.periodNum+1,
                        "Payment_Type": this.payment_type
                    },
                    Trigger:["workflow"]
                }
                await  ZOHO.CRM.API.updateRecord(config)
                .then(function(data){
                    that.showExport = true;
                    that.loadingBtn = false;
                    that.snackbar = true;
                })

        }
    },
        renderTable(){
            if(this.type == 24){
            this.arrayAUX = this.items.slice(24,26);
            this.items= this.items.slice(0,24);
            }else if (this.type == 26){ 
                if(this.arrayAUX.length == 0){
                    for (let index = 24; index < 26; index++) {
                    this.items.push({"id": index+1,"menu":false,"menu2":false,"menu3":false,"menu4":false,"menu5":false,"finalize":"","periodEnd": "","clientCut":  "","FundTo": "","pmtDate": "","comments":"","runFinals":""})
                    }
                }else{
                    let count = 24;
                    this.arrayAUX.forEach(element => {
                        count = count+1;
                        this.items.splice(count,0,element);
                        
                    });
                }
            }else{
                //Added to Render Monthly OPTION
                this.items = [];
                for (let index = 0; index < 12; index++) {
                    this.items.push({"id": index+1,"menu":false,"menu2":false,"menu3":false,"menu4":false,"menu5":false,"finalize":"","periodEnd": "","clientCut":  "","FundTo": "","pmtDate": "","comments":"","runFinals":""})
                    }
            }
          
        },
        changePMT(item,date){
            if(this.type != 12){
                let id = item.id;
                if(id === "1" || id === 1){
                    if(this.type === 26){ 
                        let count = 0;
                        let newDate; 
                        for (let index = 0; index < this.items.length; index++) {
                            newDate = moment(date).add(count, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day()== 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].pmtDate = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].pmtDate =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].pmtDate =  newDate
                            }
                            count = count + 14;
            //------------------------------------------------------------------------------------------------------------------------------------                 
                            //Change finalize
                            newDate = moment(newDate).subtract(3, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].finalize = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].finalize =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].finalize = newDate
                            }
            //------------------------------------------------------------------------------------------------------------------------------------ 
                            //Change Fund To
                            newDate = moment(newDate).subtract(3, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].FundTo = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].FundTo =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].FundTo = newDate
                            }
            //---------------------------------------------------------------------------------------------------------------------------------------                
                            //Change Client Cut
                            newDate = moment(newDate).subtract(4, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].clientCut = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].clientCut =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].clientCut = newDate
                            }
                        }
    //====================================================FIN IF==============================================================================================================
                    }else{
                        let count = 0;
                        let newDate; 
                        // let index2 = 0;
                        for (let index = 0; index < this.items.length; index++) {
                            newDate = moment(date).add(count, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day()== 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].pmtDate = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].pmtDate =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].pmtDate =  newDate
                            }
                            count = count + 15;
            //------------------------------------------------------------------------------------------------------------------------------------                 
                            //Change finalize
                            newDate = moment(newDate).subtract(3, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].finalize = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].finalize =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].finalize = newDate
                            }
            //------------------------------------------------------------------------------------------------------------------------------------ 
                            //Change Fund To
                            newDate = moment(newDate).subtract(3, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].FundTo = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].FundTo =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].FundTo = newDate
                            }
            //---------------------------------------------------------------------------------------------------------------------------------------                
                            //Change Client Cut
                            newDate = moment(newDate).subtract(4, "days").format("YYYY-MM-DD");
                            if(moment(newDate).day() == 0 || moment(newDate).day() == 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].clientCut = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].clientCut =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].clientCut = newDate
                            }        
                        }
                    }
                }
                else{
                    console.log("else");
                }
            }
        },
        changePeriod(item,date){
            //Change Period End Date
            if(this.type !=  12){    
                let id = item.id;
                if(id === "1" || id === 1){
                    let count = 0;      
                    let newDate;
                    for (let index = 0; index < this.items.length; index++) {
                        newDate = moment(date).add(count, "days").format("YYYY-MM-DD");
                        this.items[index].periodEnd =  newDate
                        /* if(moment(newDate).day() == 0 || moment(newDate).day()== 6){
                                if(moment(newDate).day()==0){
                                    //if sunday
                                    this.items[index].periodEnd = newDate = moment(newDate).subtract(2, "days").format("YYYY-MM-DD");
                                }else{
                                    //if saturday
                                    this.items[index].periodEnd =  newDate = moment(newDate).subtract(1, "days").format("YYYY-MM-DD");
                                }
                            }else{
                                this.items[index].periodEnd =  newDate
                            }*/
                        count = count + 14;
                    }
                }
            }
        },
    },
}
</script>
<style>
</style>