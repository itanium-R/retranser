<template>
  <div class="main-component">
    <h2>入力</h2>
    <select v-model="fromLang" v-on:click="showLangButton('入力');" 
                               v-on:blur="hideLangButton();">
      <option v-for="lang in langs" v-bind:value="lang.code" 
              v-on:change="translate();" v-bind:key="lang.code">
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
    <select v-model="toLang" v-on:click="showLangButton('翻訳先');"
                             v-on:blur="hideLangButton();">
      <option v-for="lang in langs" v-bind:value="lang.code" 
              v-on:change="translate();" v-bind:key="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" v-on:click="swapTexts();" value="▲">
    <input type="button" v-on:click="speak();"   value="読">
    <input type="button" value="返答音声入力" v-on:click="replyByVoice();">
    <textarea v-model="translated" readonly></textarea>
    
    <h2>再翻訳結果</h2>
    <input type="button" v-on:click="tweet();" value="Tweet">
    <textarea v-model="retranslated" readonly></textarea>
  
    <div v-if="showsLangButton" class="langSelector">
    <div class="center">
      <h2>{{ langButtonMode }}言語を選択してください．</h2>
      <span v-for="lang in langs" v-bind:value="lang.code"
            v-on:change="translate();" v-bind:key="lang.code">
        <button v-on:click="setLang(lang.code)">{{lang.description}}</button>
      </span>
    </div></div>
  </div>
  
</template>

<script>
export default {
  name: 'main-component',
  data () {
    return {
      langs: [
        {code: "en",
         char: "英",
         description: "English"},
        {code: "ja",
         char: "日",
         description: "日本語"},
        {code: "zh",
         char: "中",
         description: "中文"},
        {code: "ko",
         char: "韓",
         description: "한국어"},
      ],
      input   : "",
      fromLang: "",
      toLang      : "en",
      translated  : "",
      retranslated: "",
      lastInput   : "hoge",
      // transCnt    : 0,
      FavoData    : [],
      voiceInputButtonMsg : "音声入力",
      speaks      : false,
      showsLangButton: false,
      langButtonMode : ""
    }
  },
  props: {
    initInput   : String,
    initFromLang: String,
  },
  methods: {
    showLangButton: function(mode){
      setTimeout(() => {
        this.showsLangButton = true;
        this.langButtonMode  = mode;
      },250);
    },
    hideLangButton: function(){
      setTimeout(() => {
        this.showsLangButton = false},
      200);
    },
    setLang: function(lang){
      if(this.langButtonMode == "入力"){
        this.fromLang = lang;
      }
      if(this.langButtonMode == "翻訳先"){
        this.toLang   = lang;
      }
      this.showsLangButton = false;
    },
    translate: function(){
      // 同じ言語には翻訳できない
      if(this.fromLang == this.toLang){
        if(this.toLang == "ja")this.toLang = "en";
        else                   this.toLang = "ja";
      } 
      // inputに変更がなければ早期リターン
      let thisInput = this.fromLang + this.toLang + this.input;
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
      this.translated   = data.translatedText;
      this.retranslated = data.retranslatedText;
      if(this.speaks)this.speak();
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
      try{                                                 // eslint-disable-next-line
        var recognition = new webkitSpeechRecognition() || // eslint-disable-next-line
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
    replyByVoice: function(){
      this.swapTexts();
      this.voiceInput();
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
    },tweet(){
      if(this.input==""){
          alert("何も入力されていません");
          return -1;
      }
      let url = "https://itanium-r.github.io/retranser/index.html";
      // url += "?fromLang=" + ElmId("fromLang").value;
      // url += "&toLang="   + ElmId("toLang").value;
      // url += "&input="    + encodeURIComponent(ElmId("input").value);
      // url = encodeURIComponent(url);
      let message = this.retranslated;
      message += "←";
      message += this.translated;
      message += "←";
      message += this.input;
      message += encodeURIComponent( " #再翻やくん" ); 

      document.open("https://twitter.com/share?url=" + url + 
                    "&text=" + message + "&count=none&lang=ja",
                    "_blank",
                    "toolbar=no,location=no,menubar=no,status=no");
      }
  },created: function(){
    // 自動翻訳
    setInterval(()=>{this.translate()},1000);
    if(this.initInput)this.input    = this.initInput;
    if(this.initFromLang)this.fromLang = this.initFromLang;
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
    width: 7em !important;
  }
  textarea{
    font-family: 'Sawarabi Mincho','Noto Serif JP', sans-serif;
    resize: none;
    width : 24rem;
    height: 6em;
    max-width: 100%;
    border: solid 1px #AAA;
    border-radius: .5em;
    font-size: 1.2em;
    -ms-overflow-style: none;
  }
  textarea::-webkit-scrollbar{
    display:none;
  }
  .langSelector{
    z-index: 9;
    background: #EEE;
    position: fixed;
    padding: 1em;
    bottom:0;
    right: 0;
    left: 0;
    margin: 0;
  }
  .center{
    width: 12em;
    margin: auto;
  }
  
  button{
    font-size: 1.2rem;
    width:     5rem;
    height:    5rem;
    margin:   .5rem;
    display: inline-block;
    border-radius: 50%;
    background: #FFF;
    border: solid 1px #AAA;
  }
</style>
