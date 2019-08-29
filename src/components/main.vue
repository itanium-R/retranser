<template>
  <div class="main-component">
    <h2>入力</h2>
    <select v-model="fromLang">
      <option v-for="lang in langs" v-bind:value="lang.code" v-bind:key="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" v-on:click="voiceInput();" class="voiceInputButton" 
           v-model="voiceInputButtonMsg"> 
    <input type="button" v-on:click="translate();" value="翻訳">
    <input type="button" v-on:click="addFavorite();" value="★">
    <input type="button" v-on:click="clearTexts();" value="消">
    <textarea id="input" v-model="input"></textarea>
    <h2>翻訳結果</h2>
    <select v-model="toLang">
      <option v-for="lang in langs" v-bind:value="lang.code" v-bind:key="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" v-on:click="swapTexts();" value="▲">
    <input type="button" v-on:click="speak();"   value="読">
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
      // transCnt    : 0,
      FavoData    : [],
      voiceInputButtonMsg : "音声入力",
      speaks      : false
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

      // console.log(`trans cnt: ${this.transCnt += 1}`);
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
      // console.log(data);
      this.translated   = data.translatedText;
      this.retranslated = data.retranslatedText;
      // console.log(this.input);
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
    voiceInput: function(){
      try{
        var recognition = new webkitSpeechRecognition() || 
                          new SpeechRecognition();
        recognition.onresult = (event) => {
          if(event.results.length > 0){
            this.input = event.results[0][0].transcript;
            this.voiceInputButtonMsg = "音声入力";
            this.speaks = true;
          }else{
            this.voiceInputButtonMsg = "音声認識失敗";
          }
          recognition.stop();
        }
        recognition.onaudiostart = () => {
          this.voiceInputButtonMsg = "音声入力中";
        }
        recognition.onspeechstart = () =>{
          this.voiceInputButtonMsg = "音声認識中";
        }
        recognition.onend = () =>{
          if(this.voiceInputButtonMsg == "音声入力中" ||
             this.voiceInputButtonMsg == "音声認識中"){
             this.voiceInputButtonMsg = "音声認識失敗";
          }
        }
        recognition.lang = this.fromLang;
        recognition.start();
      }catch(e){
        alert("このブラウザは音声認識に非対応です．Web Speech API をサポートするブラウザをご利用ください．" + e);
      }
    },
    speak: function(){
      try{
        this.speaks = false;
        var uttr = new SpeechSynthesisUtterance();
        uttr.text = this.translated;
        uttr.lang = this.toLang;
        speechSynthesis.speak(uttr);
      }catch(e){
        alert("このブラウザは音声合成に非対応です．Web Speech API をサポートするブラウザをご利用ください．" + e);
      }
    },
    addFavorite: function(){
      this.loadFavorite();
      
      if(!this.input){
        alert("何も入力されていません\n");
        return -1;  
      }
      this.deleteFavorite(this.input); // 被りは消す
      this.FavoData.unshift({
        lang:     this.fromLang,
        sentence: this.input,
      });
      alert("お気に入り登録しました\n"+this.input);
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
  },created: function(){
    // 自動翻訳
    setInterval(()=>{this.translate()},1000);
    this.input    = this.initInput;
    this.fromLang = this.initFromLang;
    if(this.input)this.translate();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .main-component{
    margin-top: 2.5em;
  }
  input[type="button"].voiceInputButton{
    width:8em !important;
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
</style>
