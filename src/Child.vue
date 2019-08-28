<template>
  <div class="item">
　　<span>{{ value }}</span>
    <button @click="updateEvt">追加</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      value: 0
    }
  },
  methods: {
    plusOne () {
      this.value += 1
      this.$emit('updated', this.value)
    }
  }
}
</script>