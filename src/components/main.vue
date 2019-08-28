<template>
  <div class="main-component">
    <h2>入力</h2>
    <select v-model="fromLang">
      <option v-for="lang in langs" v-bind:value="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <!-- <input type="button" value="音声入力"> -->
    <input type="button" v-on:click="translate();" value="翻訳">
    <input type="button" v-on:click="addFavorite();" value="★">
    <input type="button" v-on:click="clearTexts();" value="消">
    <textarea id="input" v-model="input"></textarea>
    <h2>翻訳結果</h2>
    <select v-model="toLang">
      <option v-for="lang in langs" v-bind:value="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" v-on:click="swapTexts();" value="▲">
    <!-- <input type="button" onclick="speak();" value="読" id="TTSSButton"> -->
    <!-- <input type="button" id="repRecButton" value="返答音声入力" onclick="replyMicRecStart()"> -->
    <textarea v-model="translated" readonly></textarea>
    
    <h2>再翻訳結果</h2>
    <!-- <input type="button" onclick="tweet();" value="Tweet"> -->
    <textarea v-model="retranslated" readonly></textarea>
  </div>
</template>

<script>
export default {
  name: 'main-component',
  data () {
    return {
      langs: [
        {code: "en",
         char: "英"},
        {code: "ja",
         char: "日"},
        {code: "zh",
         char: "中"},
        {code: "ko",
         char: "韓"},
      ],
      input   : "",
      fromLang: "",
      toLang      : "en",
      translated  : "",
      retranslated: "",
      isWaiting   : false,
      lastInput   : "hoge",
      transCnt    : 0,
      FavoData    : [],
    }
  },
  props: {
    initInput   : String,
    initFromLang: String,
  },
  methods: {
    translate: function(){
      // 同じ言語には翻訳できない
      if(this.fromLang == this.toLang){
        if(this.toLang == "ja")this.toLang = "en";
        else                   this.toLang = "ja";
      } 
      // inputに変更がなければ早期リターン
      let thisInput = this.fromLang + this.input;
      if(thisInput == this.lastInput){
        return 0;
      } 

      console.log(`trans cnt: ${this.transCnt++}`);
      var transRequest = new XMLHttpRequest();
      transRequest.responseType = 'json';

      this.lastInput = thisInput;
      let url = this.buildUrl();
      transRequest.open("GET",url, true);
      transRequest.send();
      
      transRequest.onload = () => {
        this.showTranslateResult(transRequest.response);
      };
    },
    showTranslateResult: function(data){
        console.log(data);
        this.translated   = data.translatedText;
        this.retranslated = data.retranslatedText;
        console.log(this.input);
        if(this.isWaiting){
          this.translate();
        }
        this.isWaiting = false;
    },
    autoTrans: function(){
      setInterval(()=>{this.translate()},1000);
    },
    buildUrl: function(){
      let apiUrl = "https://script.google.com/macros/s/" +
                   "AKfycbz1e0JapH5dWB0LNI58Rs6xKhIBjmihigdFomU7HH61VLabbVs/"
      let input  = encodeURIComponent(this.input);
      return `${apiUrl}exec?text=${input}&fromLang=${this.fromLang}&toLang=${this.toLang}`;
    },
    clearTexts: function(){
      this.input        = "";
      this.translated   = "";
      this.retranslated = "";
    },
    swapTexts: function(){
      var tmp       = this.fromLang;
      this.fromLang = this.toLang;
      this.toLang   = tmp;
      this.input    = this.translated;
      this.translate();
    },
    addFavorite: function(){
      this.loadFavorite();
      
      this.deleteFavorite(this.input); // 被りは消す
      if(!this.input){
        alert("何も入力されていません\n");
        return -1;  
      }
      alert("お気に入り登録しました\n"+this.input);
      // this.deleteFavorite(this.input); // 被りは消す
      this.FavoData.unshift({
        lang:     this.fromLang,
        sentence: this.input,
      });
      this.saveFavorite();
      this.clearTexts();
    },
    deleteFavorite: function(sentence){
      this.FavoData = this.FavoData.filter(function (item) {
        return item.sentence !== sentence;
      });
      this.saveFavorite();
    },
    loadFavorite: function(){
      this.FavoData = JSON.parse( localStorage.getItem('FavoData') );
      if( !this.FavoData ){
        this.FavoData = [];
      }
    },
    saveFavorite: function(){
      localStorage.setItem('FavoData', JSON.stringify(this.FavoData));
    }
  },created(){
    // 自動翻訳
    setInterval(()=>{this.translate()},1000);
    this.input    = this.initInput;
    this.fromLang = this.initFromLang;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main-component{
  margin-top: 2.5em;
}
</style>
