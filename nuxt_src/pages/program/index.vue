<i18n>
## language=yaml
en:
  title: Program
  day1_header: Conference Day
  day2_header: Unconference Day
  unconference_title: What is an unconference？
  to_candidates: To Proposals
  bookmark_only: BookMark Only
  day1_description: |
    Conference DAY in conference format (3 parallel sessions). Doors open at 10:30, scheduled to end at 21:00 in JST.<br>
    Simultaneous interpretation will be provided via Zoom Webinar for all Track A or B sessions.
  day2_description: |
    Unconference DAY <a href="https://github.com/scalamatsuri/2022.unconference/projects/1" target="_blank" rel="noopener">Timetable</a>.<br>
    Doors open at 10:30, and scheduled to end at 20:00 in JST.<br>
    Please put your sessions ideas to <a href="https://github.com/scalamatsuri/2022.unconference" target="_blank" rel="noopener">scalamatsuri/2022.unconference</a> Github repository.<br><br>

ja:
  title: プログラム
  day1_header: カンファレンス Day
  day2_header: アンカンファレンス Day
  unconference_title: アンカンファレンスとは？
  to_candidates: 応募セッション一覧を表示する
  bookmark_only: ブックマークのみ表示
  day1_description: |
    カンファレンス DAY カンファレンス形式(3パラレルセッション) 10時30分入場開始 21時終了予定 (JST)。<br>
    Track AおよびBの全セッションについて、Zoom Webinarを利用した同時通訳がつきます。<br><br>
    さらに、ScalaMatsuriスポンサー企業によるバーチャルブースコンテンツTrackも追加予定です。どうぞお楽しみに！
  day2_description: |
    アンカンファレンス DAY <a href="https://github.com/scalamatsuri/2022.unconference/projects/1" target="_blank" rel="noopener">タイムテーブル</a> 10時30分入場開始 20時終了予定(JST）<br>
    セッションのアイディアは、<a href="https://github.com/scalamatsuri/2022.unconference" target="_blank" rel="noopener">scalamatsuri/2022.unconference</a> Githubリポジトリに投稿してください。<br><br>
</i18n>

<template>
  <!-- TODO Table構造なので上手にComponentとして切り出したりしたいぞ！！！ -->
  <!-- 開始時刻が正となるため、例えば、13:00〜14:00のものと13:30〜14:00があったら、左の列が区切られるImage -->
  <!-- google io 2018 参考 https://events.google.com/io2018/schedule/?section=may-8 -->
  <div id="program">
    <div class="main js-subNav">
      <div class="main_inner">
        <h1 class="main_title">
          {{ $t('title') }}
        </h1>
        <ul class="main_index">
          <li class="main_item">
            <a href="#day1">{{ getWholeDayStrOf(sessionsIn17) + $t('day1_header') }}</a>
          </li>
          <li class="main_item">
            <a href="#day2">{{ getWholeDayStrOf(sessionsIn18) + $t('day2_header') }}</a>
          </li>
        </ul>
      </div>
    </div>
    <!-- <div class="btnArea programCtrl">
      <p class="content_link candidatesBtn">
        <nuxt-link :to="localePath('proposals')">
          {{ $t('to_candidates') }}
          <img v-lazy="require('~/assets/img/common/arrow-next-b.svg')" alt>
        </nuxt-link>
      </p>
      <a class="js-changeView favBtn" href="javascript:void(0) ">{{ $t('bookmark_only') }}</a>
    </div> -->

    <div id="day1" class="program">
      <h2 class="program_title">
        {{ getWholeDayStrOf(sessionsIn17) + $t('day1_header') }}
      </h2>
      <p class="program_text">
        <span v-html="$t('day1_description')" />
      </p>
      <div class="schedule">
        <div v-for="[startAt, sessions] in Object.entries(sessionsIn17)" :key="startAt">
          <div class="schedule_content">
            <p class="schedule_time">
              {{ getTimeStr(parseInt(startAt)) }}
            </p>
            <div class="schedule_events">
              <div v-for="session in sessions" :key="session.title || session.proposal.id" @click="openModal(session.proposal)">
                <schedule :schedule="session" :locale="$i18n.locale" :style="{ 'pointer-events': isSessionWellDetailed(session.proposal) ? 'auto' : 'none' }" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div id="day2" class="program">
      <h2 class="program_title">
        {{ getWholeDayStrOf(sessionsIn18) + $t('day2_header') }}
      </h2>
      <p class="program_text">
        <span v-html="$t('day2_description')" />
      </p>
      <p>
        <nuxt-link :to="localePath('unconference')">
          {{ $t('unconference_title') }}
        </nuxt-link>
      </p>
      <div class="schedule">
        <div v-for="[startAt, sessions] in Object.entries(sessionsIn18)" :key="startAt">
          <div class="schedule_content">
            <p class="schedule_time">
              {{ getTimeStr(parseInt(startAt)) }}
            </p>
            <div class="schedule_events">
              <div v-for="session in sessions" :key="session.title || session.proposal.id" @click="openModal(session.proposal)">
                <schedule :schedule="session" :locale="$i18n.locale" :style="{ 'pointer-events': isSessionWellDetailed(session.proposal) ? 'auto' : 'none' }" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <transition name="fade">
      <div v-if="showModal" class="modal is_active fadeIn animated" tabindex="0" @click.self="closeModal()" @keyup.escape="closeModal()">
        <modal :program="selectProgram" @close="closeModal" />
      </div>
    </transition>
  </div>
</template>
<script>
import { mapGetters } from 'vuex'
import Modal from '@/components/parts/SessionDetailModal.vue'
import Schedule from '@/components/sections/program/schedule.vue'
import { DateTime } from 'luxon'

export default {
  components: {
    Modal,
    Schedule
  },
  data() {
    return {
      selectProgram: null,
      showModal: false
    }
  },
  computed: {
    ...mapGetters({
      filterByDateAndGroupByStartAt: 'sessions/filterByDateAndGroupByStartAt'
    }),
    sessionsIn17() {
      return this.filterByDateAndGroupByStartAt(17)
    },
    sessionsIn18() {
      return this.filterByDateAndGroupByStartAt(18)
    }
  },
  methods: {
    getTimeStr(time) {
      return DateTime.fromSeconds(time).toFormat('HH:mm')
    },
    getDateStr(time) {
      return DateTime.fromSeconds(time).toFormat('d')
    },
    getWholeDayStr(fromTime, toTime) {
      const from = this.getDateStr(fromTime)
      const to = this.getDateStr(toTime)
      if (from === to) { return '10/' + from + ' ' } else { return '10/' + from + ' - ' + to + ' ' }
    },
    getWholeDayStrOf(sessionTimeObj) {
      const sessionTimes = Object.keys(sessionTimeObj).map(v => parseInt(v))
      return this.getWholeDayStr(sessionTimes[0], sessionTimes[sessionTimes.length - 1])
    },
    isSessionWellDetailed(session) {
      return session && session[this.$i18n.locale] && session[this.$i18n.locale].detail
    },
    openModal(item) {
      if (!item || !this.isSessionWellDetailed(item)) return
      this.selectProgram = item
      this.showModal = true
    },
    closeModal() {
      this.showModal = false
    }
  },
  head() {
    const $t = this.$t.bind(this)
    return {
      title: $t('title'),
      meta: [
        { name: 'og:title', content: `${$t('title')} | ScalaMatsuri 2022` }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .2s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
