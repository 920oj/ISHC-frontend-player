<template>
  <div class="answer">
    <Header />
    <div class="wrapper">
      <div class="questionarea-wrapper">
        <a :href="backLink" style="margin: 16px 0 16px 0">← 問題一覧に戻る</a>
        <h2>{{ question.title }}</h2>
        <b-badge variant="info">{{ question.level[0] }}</b-badge>
        <div v-html="question.body" style="margin: 16px 0 16px 0"></div>
      </div>
      <div class="editorarea-wrapper">
        <monaco-editor
          class="editorarea"
          v-model="code"
          :language="language"
          ref="editor"
          :theme="theme"
        ></monaco-editor>
        <div class="controller">
          <b-form inline style="margin-top: 16px; margin-bottom: 16px">
            <label for="select-language" style="margin-right: 16px"
              >言語を選択:
            </label>
            <b-form-select
              id="select-language"
              v-model="language"
              :options="language_list"
            ></b-form-select>
          </b-form>
          <b-button block variant="primary" @click="submit()">提出</b-button>
        </div>
      </div>
    </div>
    <b-modal
      id="confirm-modal"
      hide-footer
      no-close-on-esc
      no-close-on-backdrop
      hide-header-close
    >
      <template #modal-title> {{ modalTitle }} </template>
      <div class="d-block text-center" v-if="!submit_result.is_loaded">
        <h3>現在採点中です。<br />そのままでお待ち下さい。</h3>
      </div>
      <div
        class="d-block text-center"
        v-if="submit_result.is_loaded && submit_result.is_corrected"
      >
        <h3>正解です！</h3>
        <p>点数が加算されます！</p>
        <p>メモリ使用量: {{ submit_result.memory }}</p>
        <p>処理時間: {{ submit_result.time }}</p>
        <b-button :href="backLink">問題一覧に戻る</b-button>
      </div>
      <div
        class="d-block text-center"
        v-if="submit_result.is_loaded && !submit_result.is_corrected"
      >
        <h3>不正解です……</h3>
        <p>もう一度リトライ！</p>
        <b-button @click="closeModal()">閉じる</b-button>
        <div class="text-left d-block" style="margin-top: 16px">
          <a :href="backLink">問題一覧に戻る</a>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script>
import Header from '@/components/Header.vue'
import MonacoEditor from 'vue-monaco'
export default {
  components: {
    Header,
    MonacoEditor,
  },
  data() {
    return {
      code: '',
      theme: 'vs-dark',
      language: 'c',
      language_list: [
        { value: 'c', text: 'C' },
        { value: 'cpp', text: 'C++' },
        { value: 'javascript', text: 'JavaScript(Node.js)' },
        { value: 'python', text: 'Python3' },
        { value: 'php', text: 'PHP' },
        { value: 'ruby', text: 'Ruby' },
        { value: 'shell', text: 'ShellScript(bash)' },
      ],
      submit_result: {
        is_loaded: false,
        is_corrected: null,
        msg: '',
        memory: '',
        time: '',
      },
    }
  },
  mounted() {
    console.log('リダイレクト')
    console.log(this.done_list)
    console.log(this.$route.params.pid)
    if (
      this.done_list.includes(Number(this.$route.params.pid.replace('q-', '')))
    ) {
      alert('この問題は回答済みです。問題一覧ページにリダイレクトします。')
      this.$router.push(`/${this.$route.params.id}/questions/`)
    }
  },
  computed: {
    backLink() {
      return `/${this.$route.params.id}/questions/`
    },
    modalTitle() {
      return this.submit_result.is_loaded ? '結果' : '採点中です……'
    },
  },
  methods: {
    async submit() {
      this.$bvModal.show('confirm-modal')
      let submit_language = this.language
      if (this.language == 'javascript') {
        submit_language = 'js'
      }
      if (this.language == 'shell') {
        submit_language = 'sh'
      }
      const body = {
        team_id: this.$route.params.id,
        answer: this.code,
        language: submit_language,
      }
      const question_id = this.$route.params.pid.replace('q-', '')
      const post_result = await this.$axios.$post(
        `https://ishc-api.920oj.net/submit/${question_id}`,
        body
      )
      if (post_result.msg) {
        this.submit_result.is_loaded = true
        this.submit_result.is_corrected = true
        this.submit_result.msg = post_result.msg
        this.submit_result.memory = post_result.memory
        this.submit_result.time = post_result.time
      }
      if (post_result.err) {
        this.submit_result.is_loaded = true
        this.submit_result.is_corrected = false
        this.submit_result.msg = post_result.err
      }
    },
    closeModal() {
      this.$bvModal.hide('confirm-modal')
      this.submit_result = {
        is_loaded: false,
        is_corrected: null,
        msg: '',
        memory: '',
        time: '',
      }
    },
  },
  async asyncData({ app, params }) {
    const question = await app.$axios.$get(
      `https://ishc2020.microcms.io/api/v1/questions/${params.pid}`,
      {
        headers: {
          'X-API-KEY': 'd4810716-5137-44cc-83a4-44f1bc6da121',
        },
      }
    )
    const done_list = await app.$axios.$get(
      `https://ishc-api.920oj.net/judge/teams/${params.id}/correct`
    )
    return { question, done_list }
  },
}
</script>

<style>
.wrapper {
  display: flex;
  width: 100%;
  height: calc(100vh - 56px);
}

.questionarea-wrapper {
  flex: 1;
  padding: 16px;
  height: calc(100vh - 56px);
  overflow: scroll;
}

.editorarea-wrapper {
  flex: 1;
  width: 100%;
  height: calc(100vh - 56px);
  display: flex;
  flex-direction: column;
}
.editorarea {
  flex: 6;
  width: 100%;
}

.controller {
  flex: 1;
}

div > pre {
  background-color: #f9f9f9;
  padding: 16px;
}
</style>