<template>
  <div>
    <input type="email" v-model="email" @input="isValidRegexEmail" >
    {{ valid.toString() }}
  </div>

</template>

<script>
  import axios from 'axios'
  import jsonpAdapter from 'axios-jsonp'
  let CancelToken = axios.CancelToken;
  let cancel;
  export default {
    data(){
      return{
        email:null,
        valid:false
      }
    },
    methods:{
      isValidRegexEmail(){
        const regex = /^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/
        this.valid = regex.test(this.email)
        if (!this.valid) return
        if (cancel !== undefined){
          cancel()
        }
        setTimeout( this.fetchNeverbounceApi(), 5000);


      },
      fetchNeverbounceApi(){

        axios.get('https://api.neverbounce.com/v4/poe/check',{
          cancelToken: new CancelToken(function executor(c) {
            // An executor function receives a cancel function as a parameter
            cancel = c;
          }),
          params:{
              key:'public_f33e84bb50465c18065e6b0ec0ac19ca',
              email:this.email,
              // timeout:1
            },
            adapter: jsonpAdapter
          }
        ).then(resp => {
          console.log(resp)
        }).catch( error => {
          if (axios.isCancel(error)) {
            return
          } else {
            console.log(error)
          }
        })
      }
    }

  }
</script>

<style>
</style>
