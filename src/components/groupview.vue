<template>
  <div class="groupview">
    <new-meeting :team="currentGroup" :token="token"
                 @finish="addMeetingFinish"
                 @cancel="currentGroup=''" ref="newMeeting"
                 v-if="currentGroup && currentGroup.toString().length>0"></new-meeting>
    <new-group v-if="addNewGroup" ref="newGroup" :token="token"
               @finishGroup="addGroupFinish" @cancel="addNewGroup=false"></new-group>
    <header>
      <svg class="router-icon-s" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 28.89 28.89">
        <title>icon</title>
        <g>
          <rect class="icon-s" x="10.29" y="1.73" width="8.35" height="8.35"
                transform="translate(8.41 -8.5) rotate(45)"></rect>
          <rect class="icon-s" x="18.81" y="10.25" width="8.35" height="8.35"
                transform="translate(16.93 -12.03) rotate(45)"></rect>
          <rect class="icon-s" x="10.25" y="18.81" width="8.35" height="8.35"
                transform="translate(20.48 -3.47) rotate(45)"></rect>
          <rect class="icon-s" x="1.73" y="10.29" width="8.35" height="8.35"
                transform="translate(11.96 0.06) rotate(45)"></rect>
        </g>
      </svg>
      <h2>我的小组</h2>
      <button type="button" @click="addNewGroup=true">+ 添加小组</button>
    </header>
    <div class="groups">
      <section v-for="group in groups">
        <div class="col">
          <thumbnail :seed="group.team_id" width="72px" height="72px" radius="999em" :alt="group.name"></thumbnail>
        </div>
        <div class="col">
          <h3><a :href='"/group.html#/groupmanagement/" + group.team_id'>{{group.name}}</a></h3>
          <p>{{group.description.length < 45 ? group.description : (group.description.substr(0, 44) + '……')}}</p>
          <ul class="btn-list">
            <li><a :href="'/meeting.html#/' + group.team_id" target="_blank"><i class="icon-play"></i>进入会议</a></li>
            <li><a :href='"/group.html#/groupmanagement/" + group.team_id'><i class="icon-conf"></i>小组管理</a></li>
            <li><a @click="currentGroup=group.team_id"><i class="icon-add"></i>新建预约</a></li>
          </ul>
        </div>
        <corner v-if="group.corner" :content="group.corner"></corner>
      </section>
      <section class="no-groups" v-if="!groups || groups.length===0">
        <p>你还没有小组~　创建或加入一个小组吧！</p>
      </section>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import urlconf from 'assets/url.conf'
  import thumbnail from './thumbnail'
  import corner from './corner'
  import newMeeting from './popupNewMeeting'
  import newGroup from './popupNewGroup'
  export default {
    props: ['token'],
    data () {
      return {
        groups: [],
        currentGroup: '',
        addNewGroup: false
      }
    },
    methods: {
      addMeetingFinish () {
        this.currentGroup = ''
      },
      addGroupFinish () {
        this.$http.get(urlconf.group(this.token)).then(resp => {
          this.groups = resp.body
        }, resp => {

        })
        this.addNewGroup = false
      }
    },
    components: {
      thumbnail,
      corner,
      newMeeting,
      newGroup
    },
    created () {
      this.$http.get(urlconf.group(this.token)).then(resp => {
        this.groups = resp.body
      }, resp => {

      })
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .router-icon-s
    width 30px
    height 30px
    .icon-s
      fill #a7c158

  header
    display flex
    align-items center
    padding-left 30px
    svg
      margin-right 20px
    button
      color #fff
      background #FFCA4C
      border-radius 9px
      border none
      font-size 16px
      width 125px
      height 34px
      line-height 34px
      margin-left 50px
      cursor pointer
      transition-duration .3s
      &:hover
        box-shadow 0 2px 6px rgba(0, 0, 0, 0.16)

  .groups
    display flex
    flex-wrap wrap
    padding-left 24px
    section
      display flex
      align-items center
      background #F2F2F2
      width 400px
      margin 0 30px 30px 30px
      transition-duration .3s
      &:hover
        box-shadow 0 0 10px #aaa
      .col
        display flex
        flex-direction column
        justify-content space-around
        padding 0 20px
        &:first-child
          height 120px
          border-right solid .5px #A7C158
      .thumbnail
        width 72px
        height 72px
      h3
        font-size 16px
        margin-bottom 0
      p
        font-size 14px
        font-weight 200
        color #939880
        margin 6px auto
        width 100%
      ul
        display flex
        flex-wrap wrap
        justify-content space-between
        font-size 12px
        font-weight 200
        li
          transition-duration .3s
          padding 4px 8px
          border-radius 4px
          cursor pointer
          &:hover, &:active
            background #fff
        i
          color #A7C158
          margin-right 4px

  section.no-groups
    width 100%
    height 500px
    text-align center
    &:hover
      box-shadow none
    p
      font-size 24px
</style>
