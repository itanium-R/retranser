<template>

  <div class="favo-component">

    <p v-if="FavoData.length==0">
      お気に入りは登録されていません．<br>
      ★ボタンで登録するとここに表示されます．
    </p>
    <ul>
      <li v-for="favo in FavoData" v-bind:key="favo.sentence">
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
  name: 'favo-component',
  data () {
    return {
      FavoData: []
    }
  },
  props: {
    msg: String
  },
  methods: {
    emitEventTwo () {
      this.$emit('inputFromFavo', 'This is an argument')
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
      this.$emit('inputFromFavo', favo);
    }
  },
  created: function(){
    this.loadFavorite();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.favo-component{
  margin-top: 4em;
}
</style>
