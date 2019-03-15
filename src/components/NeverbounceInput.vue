<template>
  <div class="container">
    <input type="email" v-model.lazy="email" v-debounce="delay" size="30" :class="inputClass">
    <transition name="fade">
      <span v-if="status === 'sending'" class="spinner"></span>
    </transition>
    <div v-if="status !== undefined">
      {{ status.toString() }}
    </div>
  </div>

</template>

<script>
  import axios from 'axios'
  import jsonpAdapter from 'axios-jsonp'
  import debounce from 'v-debounce'
  let CancelToken = axios.CancelToken;
  let cancel;
  export default {
    props: {
      nbKey: {
        type: String,
        required: true
      },
      inputClass: {
        type: String,
        required: false
      },
    },
    data(){
      return{
        email:null,
        status:undefined,
        delay: 500,
      }
    },
    directives: {
      debounce
    },
    watch: {
      email () {
        if(this.isValidRegexEmail(this.email)){
          console.log('regex valid')
          if (cancel !== undefined){
            cancel('Request anterior cancelado')
          }
          this.fetchNeverbounceApi()
        }
      }
    },
    methods:{
      isValidRegexEmail(email){
        return /^([a-zA-Z0-9_.-])+@(([a-zA-Z0-9-])+\.)+([a-zA-Z0-9]{2,4})+$/.test(email)
      },
      fetchNeverbounceApi(){
        this.status = 'sending'
        axios.get('https://api.neverbounce.com/v4/poe/check',{
            cancelToken: new CancelToken(function executor(c) {
              // An executor function receives a cancel function as a parameter
              cancel = c;
            }),
            params:{
              key:this.key,
              email:this.email,
              // timeout:1
            },
            adapter: jsonpAdapter,

          }
        ).then(resp => {
          console.log(resp)
          this.status = resp.data.result
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

<style scoped>
  input {
    padding: 5px;
  }
  .container {
    display: inline-block;
    position: relative;
  }
  @keyframes spinner {
    to {transform: rotate(360deg);}
  }

  .spinner:before {
    content: '';
    box-sizing: border-box;
    position: absolute;
    right: 0;
    width: 15px;
    height: 15px;
    border-radius: 50%;
    margin: 6px;
    border: 2px solid #ccc;
    border-top-color: red;
    animation: spinner .6s linear infinite;
  }
  /* ANIMATIONS */
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active em versões anteriores a 2.1.8 */ {
    opacity: 0;
  }
</style>
