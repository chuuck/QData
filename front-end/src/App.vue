<template>
  
  <div id = "center_layout">
  
    <HelloWorld @change="fileChanged" msg="Welcome to Your Vue.js App"/>
    <InputPrompt ref="input_prompt" @submitPrompt="update_prompt"/>
    <SQLAccordion ref="accordion" :query="query"/>
    <ResponseTable :people="people" :columns="columns"/>

  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import InputPrompt from './components/InputPrompt.vue'
import SQLAccordion from './components/SQLAccordion.vue'
import ResponseTable from './components/ResponseTable.vue'
import axios from 'axios'

export default {
  name: 'App',
  components: {
    HelloWorld, InputPrompt, SQLAccordion, ResponseTable
  },
  data(){
    return{
      files: '',

      columns: [],

      people: [],

      prompt: '',

      query: ''
    }
  },
  methods: {
      onClickChild () {
        this.$refs.input_prompt.switch_loading_state(true)
        
        let formData = new FormData()

        console.log(this.files);

        for (let index = 0; index < this.files.length; index++) {
          formData.append('file', this.files[index])
        }
      
        formData.append('prompt', this.prompt)
        

        axios.post('http://127.0.0.1:8000/upload-csv-test',
          formData, {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
        }).then((response) => {
          this.$refs.input_prompt.switch_loading_state(false)
          this.query = response.data['query']
          this.$refs.accordion.highlight_text()
          let table_object = JSON.parse(response.data['table']);

          this.columns = Object.keys(table_object[0]).reduce((obj, key) => {
            obj[key] = key;
            return obj;
          }, {});

          console.log(this.columns)
          this.people = table_object
          console.log(this.people)
          
          
          
        }, (error) => {
          this.$refs.input_prompt.switch_loading_state(false)
          console.log(error);
        });

      },

      fileChanged(uploaded_files){

        this.files = uploaded_files.target.files
      },

      update_prompt(prompt){
        this.prompt = prompt;
        this.onClickChild();
      }
 
  }
}
</script>

<style>


#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}


#center_layout{

  margin-left: 15%;
  margin-right: 15%;

}


</style>
