<template>
  <div class="main">
    <h2>入力</h2>
    <select v-model="fromLang">
      <option v-for="lang in langs" :key="lang" v-bind:value="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" id="recButton" value="音声入力" onclick="micRecStart()">
    <input type="button" onclick="execTrans();" value="翻訳">
    <input type="button" value="★"
           onclick="vm.addFavorite(ElmId('fromLang').value,ElmId('input').value);
                    clearTextareas();">
    <input type="button" onclick="clearTextareas();" value="消">
    <textarea id="input" oninput="autoTrans()"  
    onkeydown="transByEnter()" v-model="input"></textarea>
    <h2>翻訳結果</h2>
    <select v-model="toLang">
      <option v-for="lang in langs" :key="lang" v-bind:value="lang.code">
        {{ lang.char }}
      </option>
    </select>
    <input type="button" onclick="swapLang(true);" value="▲">
    <input type="button" onclick="speak();" value="読" id="TTSSButton">
    <input type="button" id="repRecButton" value="返答音声入力" onclick="replyMicRecStart()">
    <textarea id="translated"   readonly></textarea>
    
    <h2>再翻訳結果</h2>
    <input type="button" onclick="tweet();" value="Tweet">
    <textarea id="retranslated" readonly></textarea>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      langs: [
        {code: "en",
         char:"英"},
        {code: "ja",
         char:"日"},
        {code: "zh",
         char:"中"},
        {code: "ko",
         char:"韓"},
      ],
      fromLang:"ja",
      toLang:"ko"
    }
  },
  props: {
    msg: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main{
  margin-top: 2.5em;
}
</style>
