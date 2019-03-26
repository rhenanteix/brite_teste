<template>
    <div id="table-component">
      
        <div id="table-header" class="header-table">
            <span>
                <label >Amount Range</label>
                <input type='text' placeholder="Min" v-model="filterValues.minAmount" v-on:change="onFilterDataChanged"/>
                <input type='text' placeholder="Max" v-model="filterValues.maxAmount" v-on:change="onFilterDataChanged"/>
            </span>
            <span>
                <label >Duration</label>
                <input type='date' placeholder="Min" v-model="filterValues.minDate" v-on:change="onFilterDataChanged"/>
                <input type='date' placeholder="Max" v-model="filterValues.maxDate" v-on:change="onFilterDataChanged"/>
            </span>
            <span>
                <label>Display Per Page</label>
                <input type='number' min="5" max="50" value="5" v-on:change="onDisplayPerChanged" v-model="pagenation.displayPerPage"/>
            </span>
            <br/>
            <br/>
        </div>
        <div id="table-content">
            <b-table :tbody-tr-class="rowClass">
                <thead>
                    <tr>
                        <th v-for="item in header" v-on:click="orderChanged" v-bind:key="item">{{item}}</th>
                    </tr>
                </thead>
                <tbody>
                    <template v-for="item in sortedData">
                        <table-row :rowData="item" v-bind:key="item.ID" @updated="onRowDataUpdated"></table-row>
                    </template>
                </tbody>
            </b-table>
        </div>
        <div id="table-footer">
            <b-pagination class="page-number" v-bind:key="number" v-for="number in pagenation.pageNumber" v-on:click="onPageChanged">{{number}}</b-pagination>
        </div>
    </div>
    
</template>

<script>
import TableRow from './TableRow'

export default {
    props : ['data','header'],
    name: 'table-component',
    components: {
        TableRow
    },
    data: function(){
        return{
            sortedData : this.data,
            filteredData : this.data,
            sortKey : "",
            sortOrder : "ASC",

            pagenation:{
                currentPage : 1,
                pageNumber : [],
                displayPerPage : 5
            },

            filterValues : {
                minAmount : undefined,
                maxAmount : undefined,
                minDate : undefined,
                maxDate : undefined
            }
        }
    },
    created : function(){
        console.log('data is ', this.data,this.header)
        console.log(this.data.length);
        let cnt = Math.ceil(this.data.length / this.pagenation.displayPerPage);
        console.log(cnt);
        for(let i = 1 ; i <= cnt ;i++)
            this.pagenation.pageNumber.push(i);
        this.filterData();
    },
    methods : {
        filterData : function(){
            console.log('filter data is ',this.filterValues);
            let _this = this;
            this.sortedData = this.filteredData;

            //SORTING
            this.sortedData = this.sortedData.sort(function(a,b){
                let x = a[_this.sortKey];
                let y = b[_this.sortKey];
                if(_this.sortOrder == "ASC")
                {
                    if(x < y) return -1;
                    else return 1;
                }
                else{
                    if(x < y) return 1;
                    else return -1;
                }
            })

            //PAGINATION
            this.sortedData = this.sortedData.slice((this.pagenation.currentPage-1) * this.pagenation.displayPerPage,
                                                    this.pagenation.currentPage * this.pagenation.displayPerPage);
            console.log(this.pagenation);
        },
        orderChanged : function(e){
            this.sortKey = e.target.innerHTML;
            if(this.sortOrder == "ASC")
                this.sortOrder = "DEC";
            else
                this.sortOrder = "ASC";
            this.filterData();
        },
        onDisplayPerChanged : function(){
            let cnt = Math.ceil(this.filteredData.length / this.pagenation.displayPerPage);
            this.pagenation.currentPage = 1;
            this.pagenation.pageNumber = [];
            for(let i = 1 ; i <= cnt ;i++)
                this.pagenation.pageNumber.push(i);
            this.filterData();
        },
        onPageChanged : function(e){
            this.pagenation.currentPage = parseInt(e.target.innerHTML);
            this.filterData();
        },
        onRowDataUpdated : function(e){
            console.log('updated data is ',e);
            for(let i in this.sortedData)
            {
                if(this.data[i].ID == e.ID)
                {
                    this.data[i].Description = e.Description;
                    break;
                }
            }
        },
        onFilterDataChanged : function(){
            this.filteredData = this.data;
            let _this = this;

            //FILTERING

            //AMOUNT
            if(this.filterValues.minAmount !== undefined && this.filterValues.minAmount !== "")
            {
                this.filteredData = this.filteredData.filter(function(item){
                    return item.Amount >= parseInt(_this.filterValues.minAmount)
                })
            }

            if(this.filterValues.maxAmount !== undefined && this.filterValues.maxAmount !== "")
            {
                this.filteredData = this.filteredData.filter(function(item){
                    return item.Amount <= parseInt(_this.filterValues.maxAmount)
                })
            }

            // DATE
            if(this.filterValues.minDate !== undefined && this.filterValues.minDate !== "")
            {
                this.filteredData = this.filteredData.filter(function(item){
                    // return item.Amount >= parseInt(_this.filterValues.minAmount)
                    var x = new Date(item.Date);
                    var y = new Date(_this.filterValues.minDate);
                    return x >= y;
                })
            }

            if(this.filterValues.maxDate !== undefined && this.filterValues.maxDate !== "")
            {
                this.filteredData = this.filteredData.filter(function(item){
                    // return item.Amount >= parseInt(_this.filterValues.minAmount)
                    var x = new Date(item.Date);
                    var y = new Date(_this.filterValues.maxDate);
                    return x <= y;
                })
            }
            this.onDisplayPerChanged();
        }
    }
}
</script>

<style>
    #table-footer{
        padding-top: 2rem;
    }
    .page-number{
        padding: 1rem;
        margin: .3rem;
        background-color: #007bff;
        color: #fff;
        border-radius: .3rem;
        cursor: pointer;
    }
    table {
        font-family: arial, sans-serif;
        border-collapse: collapse;
        width: 100%;
    }

    td, th {
        border: 1px solid #dddddd;
        text-align: left;
        padding: 8px;
    }
    th{
        cursor: pointer;
    }
    label {
       color: #007bff; 
       margin: 1.5em 0 1em;
       font-size: 22px;
       text-align: center;
        
    }
       input{
        width: 25%;
        height: 120%;
        padding: .375rem .75rem;
        font-size: 1rem;
        font-weight: 400;
        line-height: 1.5;
        color: #495057;
        background-color: #fff;
        background-clip: padding-box;
        border: 1px solid #ced4da;
        border-radius: .25rem;
        display: flex;
        
    }
    
</style>
