<template>
  <div>
    test
    <tinymce-editor ref="editor" v-model="newFrom.contents" @onClick="onClick"></tinymce-editor>
  </div>
</template>

<script>
import TinymceEditor from '@/components/tiny-editor'
export default {
  components: { TinymceEditor },
  data() {
    return {
      newFrom: {
        contents: ''
      }
    }
  },
  methods: {
    onClick(e, tinymce) {
      console.log(this.newFrom.contents)
    }
  }
}
</script>
<style lang="sass" scoped>

</style>