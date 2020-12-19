<template>
  <b-navbar type="dark" variant="dark">
    <b-navbar-nav class="header-nav">
      <b-nav-item>IS HACK CONTEST 2020 回答者ページ</b-nav-item>
      <p class="clock-msg">残り:{{ time }}</p>
    </b-navbar-nav>
  </b-navbar>
</template>

<script>
const dayjs = require('dayjs')

export default {
  data() {
    return {
      time: '',
    }
  },
  async mounted() {
    const settings = await this.$axios.$get(
      'https://ishc2020.microcms.io/api/v1/settings',
      {
        headers: {
          'X-API-KEY': 'd4810716-5137-44cc-83a4-44f1bc6da121',
        },
      }
    )

    setInterval(() => {
      const now_date = dayjs()
      const finish_date = dayjs(Number(settings['finish-time'] * 1000))
      const diff = finish_date.diff(now_date)
      this.time = dayjs(diff).subtract(9, 'hour').format('hh:mm.ss')
    }, 1000)
  },
}
</script>

<style>
.header-nav {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.clock-msg {
  color: #fff;
  font-size: 1.2rem;
}
</style>