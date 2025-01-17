<script setup lang="ts">
import axios from 'axios'
import InputBar from './components/InputBar.vue'
import ValuesDisplay from './components/ValuesDisplay.vue'
import TypesDisplay from './components/TypesDisplay.vue'
import FilterDropdown from './components/FilterDropdown.vue'
import SortButton from './components/SortButton.vue'
import DatePicker from './components/DatePicker.vue' // Hier hinzugefügt
import { ValueType } from './scripts/value_type'
import { Value } from './scripts/value'
</script>

<script lang="ts">
export default {
  data() {
    return {
      values: new Array<Value>(),
      value_types: new Array<ValueType>(),
      filter_start: '',
      filter_end: '',
      filter_type: '', // <--- The missing comma
      filter: ''
    }
  },

  mounted() {
    this.get_types()
    this.get_values().then((data) => {
      this.values = data
    })
  },
  methods: {
    getTypeId(type_name: string) {
      var return_value = ''
      for (var i = 0; i < this.value_types.length; i++) {
        if (this.value_types[i].type_name.toUpperCase() == type_name.toUpperCase()) {
          return_value = '' + this.value_types[i].id
          console.log('Found matching type', this.value_types[i])
        }
      }
      return return_value
    },


    sortValues(order: 'asc' | 'desc') {
      this.values.sort((a, b) => {
        const aValue = a.value.toString(); // Convert to string
        const bValue = b.value.toString(); // Convert to string

        if (order === 'asc') {
          return parseFloat(aValue) - parseFloat(bValue); // Compare numerically
        } else {
          return parseFloat(bValue) - parseFloat(aValue); // Compare numerically
        }
      });
    },

    
    update_search(args: string[]) {
      console.log('New search arguemnts', args)
      this.filter_end = ''
      this.filter_start = ''
      this.filter_type = ''
      for (var i = 0; i < args.length; i++) {
        const command = args[i]
        console.log('handling command', command)
        const command_and_args = args[i].split(':')
        if (command_and_args.length == 2) {
          const key = command_and_args[0]
          const value = command_and_args[1]
          if (key == 'type') {
            this.filter_type = this.getTypeId(value)
            console.log('Update typeid', this.filter_type)
            continue
          } else if (key == 'start') {
            this.filter_start = value
            continue
          } else if (key == 'end') {
            this.filter_end = value
            continue
          }
        }
        console.log('Ignoring command', command)
      }
      this.get_values().then((result) => {
        this.values = result
      })
    },
    get_types() {
      axios
        .get('/api/type/')
        .then((result) => {
          this.value_types = result.data
        })
        .catch((error) => {
          console.error(error)
        })
    },
    get_values() {
     const promise = new Promise<Value[]>((accept, reject) => {
       const url = '/api/value/';
       var params: { [key: string]: string } = {};
       if (this.filter !== '') {
         params['type_id'] = this.filter;
       }
       if (this.sortOrder === 'asc') {
         params['sort'] = 'asc';
       } else if (this.sortOrder === 'desc') {
         params['sort'] = 'desc';
       }
       if (this.filter_type != '') {
         params['type_id'] = this.filter_type;
       }
       if (this.filter_end != '') {
         params['end'] = this.filter_end;
       }
       if (this.filter_start != '') {
         params['start'] = this.filter_start;
       }
       console.log('Trying to get url', url);
       axios
         .get(url, { params: params })
         .then((result) => {
           accept(result.data);
         })
         .catch((error) => {
           console.error(error);
           reject(error);
         });
     });
     return promise;
   },

    updateFilter(selectedType) {
      this.filter = selectedType;
      this.get_values().then((result) => {
        this.values = result;
      });
    },

    updateDateFilter(selectedDateRange) {
       this.filter_start = selectedDateRange.start;
       this.filter_end = selectedDateRange.end;
       this.get_values().then((result) => {
         this.values = result;
       });
     },

     resetFilters() {
      location.reload(); // Reloads the current page
    },
  }
}
</script>

<template>
  <div class="container p-1">
    <h1 class="row">RDP</h1>
    <FilterDropdown :types="value_types" @filter="updateFilter" />
    <DatePicker @filter="updateDateFilter" />
    <SortButton label="Sort Lowest to Highest" direction="asc" @sort="sortValues" />
    <SortButton label="Sort Highest to Lowest" direction="desc" @sort="sortValues" />
    <InputBar @search="update_search" />
    <div class="row dark-red-bg text-center mb-2 d-flex justify-content-center">
      <button @click="resetFilters" class="btn btn-danger">Reset all Filters</button>
    </div>

    <TypesDisplay :value_types="value_types" @update_type="get_types" />
    <ValuesDisplay :values="values" :value_types="value_types" />
  </div>
</template>
