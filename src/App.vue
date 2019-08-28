<template>
  <div id="app">
    <div class="header"><div class="headerInner">
      <h1 class="headerL">再翻やくん</h1>
      <input class="headerR" type="button" value="★List"
             @click="showFavo()" v-if="showsMain">
      <input class="headerR" type="button" value="TOP"
             @click="showMain()" v-if="showsFavo"> 
    </div></div><br>
    <div v-if="showsMain">
      <Main v-bind:initInput="initialInput" v-bind:initFromLang="initialLang"/>
    </div>
    
    <div v-if="showsFavo">
      <Favo @inputFromFavo="inputFromFavo"/>
    </div>
  </div>
</template>

<script>
import Favo from './components/favo.vue'
import Main from './components/main.vue'

export default {
  name: 'app',
  components: {
    Favo,
    Main
  },
  data () {
    return {
      showsMain : true,
      showsFavo : false,
      initialInput : "",
      initialLang  : "ja"
    }
  },
  methods: {
    showMain: function(){
      this.showsMain = true;
      this.showsFavo = false;
    },
    showFavo: function(){
      this.showsMain = false;
      this.showsFavo = true;
    },
    inputFromFavo: function(favo) {
      console.log(favo.sentence);
      console.log(favo.lang);
      this.initialInput = favo.sentence;
      this.initialLang  = favo.lang;
      this.showMain();
    },
  }

}
</script>

<style>

  @import url('https://fonts.googleapis.com/css?family=Noto+Serif+JP|Sawarabi+Mincho&display=swap');

  body{
    margin:0;
    padding:0;
    font-family: 'Sawarabi Mincho','Noto Serif JP', sans-serif;
    max-width:100%;
  }
  .header{
    position: fixed;
    top   : 0;
    right : 0;
    left  : 0;
    background: #EEE;
    color: #000;
    height:4em;
    text-align:center;
    z-index:9;
  }
  .headerInner{
    max-width: 24em;
    margin:auto;
  }
  .headerR{
    float:right;
    margin-top:1rem;
    height:2rem !important;
    width :4em;
  }
  .headerL{
    float: left;
  }
  .header::after {
    content: "";
    display: block;
    clear: both;
    overflow: hidden;
  }
  
  #main{
    display: none;
    animation:         fadeIn .5s ease 0s 1 normal forwards;
    -webkit-animation: fadeIn .5s ease 0s 1 normal forwards;
  }
  #favorite{
    display: none;
    animation:         fadeIn .5s ease 0s 1 normal forwards;
    -webkit-animation: fadeIn .5s ease 0s 1 normal forwards;
  }
  @keyframes fadeIn {
    0% {opacity: 0}
    100% {opacity: 1}
    }
  h1{
    margin: .6em 0 .6em 0;
    padding:0;
    font-size:1.5em;
    display:inline-block;
  }
  h2{
    margin: .5em .5em .5em 0;
    padding:0;
    font-size:1.2em;
    display:inline-block;
  }
  input[type="button"],select{
    font-family: 'Sawarabi Mincho','Noto Serif JP', sans-serif;
    background:#FEFEFE;
    border:solid 1px #AAA;
    border-radius: .2em;
    height:2.4em;
  }
  #recButton,#repRecButton{
    width:7.5em;
    max-width:90%;
    /* display:none; */
  }
  textarea{
    font-family: 'Sawarabi Mincho','Noto Serif JP', sans-serif;
    resize: none;
    width : 24rem;
    height: 6em;
    max-width:100%;
    border:solid 1px #AAA;
    border-radius: .5em;
    font-size:1.2em;
    -ms-overflow-style:none;
  }
  textarea::-webkit-scrollbar{
    display:none;
  }
  div.tArea{
    width : 24em;
    max-width:90%;
    height: 6em;
    
    background:#FFF;
    border:solid 1px #AAA;
    border-radius: .5em;
  }
  #app{
    margin:auto;
    padding:0;
    width:24rem;
    max-width:80%;
  }
  ul{
    margin:.5em 0 .5em 0;
    padding:0;
  }
  li{
    list-style-type: none;
    padding:.3em 0 .3em 0;
  }
  li:nth-child(2n-1){
    background: #EEE;
  }
  .radius50{
    border-radius:50% !important;
  }
  .redBack{
    background: #F40 !important;
    color: #FFF;
  }
  #tofavorite{
    display: none;
  }
  /* #TTSSButton{
    display: none;
  } */
</style>
