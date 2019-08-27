<template>
  <div class="favo">
    <ul>
      <li v-for="favo in FavoData" :key="favo">
        <input value="←" type="button" class="radius50"
               v-on:click="writeInput(favo)">
        <input value="✖"  type="button" class="radius50"
               v-on:click="deleteFavorite(favo.sentence)">
        {{ favo.lang }}:{{ favo.sentence }}        
      </li>
    </ul>     
  </div>
</template>

<script>
export default {
  name: 'favo',
  data () {
    return {
      FavoData: [
        {lang:"en",
        sentence:"I have a pen."},
        {lang:"en",
        sentence:"I have an apple."}
      ]
    }
  },
  props: {
    msg: String
  },
  methods: {
    addFavorite: function(newLang,newSentence){
      if(!newSentence){
        alert("何も入力されていません\n");
        return -1;  
      }
      alert("お気に入り登録しました\n"+newSentence);
      this.deleteFavorite(newSentence); // 被りは消す
      this.FavoData.unshift({
        lang:     newLang,
        sentence: newSentence,
      });
      this.saveFavorite();
    },
    deleteFavorite: function(sentence){
      this.FavoData = this.FavoData.filter(function (item) {
        return item.sentence !== sentence;
      });
      this.saveFavorite();
    },
    saveFavorite: function(){
      localStorage.setItem('FavoData', JSON.stringify(this.FavoData));
    },
    loadFavorite: function(){
      this.FavoData = JSON.parse( localStorage.getItem('FavoData') );
      if( !this.FavoData ){
        this.FavoData = [];
      }
    },
    writeInput: function(favo){
      clearTextareas();
      ElmId("fromLang").value = favo.lang;
      ElmId("input").value    = favo.sentence;
      execTrans();
      switchSection("main");
    }
  }/*,
  mounted: function(){
    this.loadFavorite();
  }*/
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.favo{
  margin-top: 2.5em;
}
</style>
