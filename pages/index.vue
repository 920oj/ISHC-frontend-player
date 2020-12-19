<template>
  <div class="index">
    <Header />
    <b-jumbotron header="IS HACK CONTEST 2020" lead="回答者用ページ">
      <p>ここに説明を入力</p>
      <b-form inline>
        <b-form-select v-model="selected" :options="teams"></b-form-select>
        <b-button variant="primary" @click="jumpPage()">移動</b-button>
      </b-form>
    </b-jumbotron>
  </div>
</template>

<script>
import Header from '@/components/Header.vue'
export default {
  components: {
    Header,
  },
  data() {
    return {
      selected: null,
      teams: [
        { value: '1', text: '大谷研究室' },
        { value: '2', text: '岩野研究室' },
        { value: '3', text: '宮地研究室' },
        { value: '4', text: '市野研究室' },
      ],
    }
  },
  methods: {
    jumpPage() {
      if (this.selected) {
        window.location.href = `/${this.selected}/questions`
      }
    },
  },
}
</script>

<style>
</style>
