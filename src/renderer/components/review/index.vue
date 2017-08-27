<style lang="scss">
    .review {
        .display {
            td {
                border: 1px solid black;
                &.desc:after {
                    content: "⬆";
                }
                &.asc:after {
                    content: "⬇";
                }
            }
        }
    }
</style>
<template>
    <div class="review">
        <div class="conditions">
            <i-form inline ref="filter" :model="filter">
                <form-item prop="word">
                    <i-input v-model="filter.word" placeholder="word" @on-change="timer">
                        <Icon type="ios-person-outline" slot="prepend"></Icon>
                    </i-input>
                </form-item>
                <form-item prop="word">
                    <rate v-model="filter.rankMin" allow-half placeholder="rank-min" @on-change="timer"></rate>
                </form-item>
                <form-item prop="word">
                    <rate v-model="filter.rankMax" allow-half placeholder="rank-max" @on-change="timer"></rate>
                </form-item>
                <form-item prop="word">
                    <Checkbox v-model="filter.recognized" placeholder="recognized" @on-change="timer"></Checkbox>
                </form-item>
                <form-item prop="word">
                    <Date-picker type="date" placeholder="startTime" v-model="filter.startTime"
                                 @on-change="timer"></Date-picker>
                </form-item>
                <form-item prop="word">
                    <Date-picker type="date" placeholder="endTime" v-model="filter.endTime"
                                 @on-change="timer"></Date-picker>
                </form-item>
                <form-item prop="word">
                    <i-input v-model="filter.sourceUrl" placeholder="sourceUrl" @on-change="timer">
                        <Icon type="ios-person-outline" slot="prepend"></Icon>
                    </i-input>
                </form-item>
                <form-item prop="word">
                    <i-input v-model="filter.sourceSentence" placeholder="sourceSentence" @on-change="timer">
                        <Icon type="ios-person-outline" slot="prepend"></Icon>
                    </i-input>
                </form-item>
            </i-form>
        </div>
        <div class="display">
            <table>
                <tr>
                    <td @click="sort('word')" :class="{asc: sortStatus.word === 1, desc: sortStatus.word === -1}">word(sortable)</td>
                    <td>definition</td>
                    <td @click="sort('rank')" :class="{asc: sortStatus.rank === 1, desc: sortStatus.rank === -1}">rank(sortable)</td>
                    <td @click="sort('recognized')" :class="{asc: sortStatus.recognized === 1, desc: sortStatus.recognized === -1}">recognized(sortable)</td>
                    <td @click="sort('createTime')" :class="{asc: sortStatus.createTime === 1, desc: sortStatus.createTime === -1}">createtime(sortable)</td>
                    <td>sourceurl</td>
                    <td>sourceSentence</td>
                    <td @click="sort('finded')" :class="{asc: sortStatus.finded === 1, desc: sortStatus.finded === -1}">finded(sortable)</td>
                </tr>
                <tr v-for="item of list">
                    <td>{{item.word}}</td>
                    <td>{{item.definition}}</td>
                    <td>
                        <Rate allow-half v-model="item.rank"></Rate>
                    </td>
                    <td>
                        <Checkbox v-model="item.recognized"></Checkbox>
                    </td>
                    <td>{{item.createTime.toLocaleString()}}</td>
                    <td>{{item.sourceUrl}}</td>
                    <td>{{item.sourceSentence}}</td>
                    <td>{{item.finded}}</td>
                </tr>
            </table>
        </div>
        <div class="pagination">
            <Page :total="pageTotal" size="small" :page-size="pageSize" :current="curPage"
                  @on-change="changePage"></Page>
        </div>
    </div>
</template>
<script>
  import { mapState, mapActions } from 'vuex'

  let handle

  export default {
    data () {
      return {
        filter: {
          rankMin: 1,
          rankMax: 5
        },
        sortStatus: {
          word: 0,
          rank: 0,
          recognized: 0,
          createTime: 0,
          finded: 0
        },
        pageTotal: 1,
        pageSize: 10,
        curPage: 1
      }
    },
    computed: {
      ...mapState({
        list: state => state.words.words
      })
    },
    mounted () {
      this.timer()
    },
    watch: {
      'filter.rankMin': function (value) {
        if (value > this.filter.rankMax) {
          this.filter.rankMax = value
        }
      },
      'filter.rankMax': function (value) {
        if (value < this.filter.rankMin) {
          this.filter.rankMin = value
        }
      }
    },
    methods: {
      ...mapActions({
        search: 'words/search',
        count: 'words/count'
      }),
      sort (field) {
        this.sortStatus[field] = this.sortStatus[field] === 0 ? 1 : this.sortStatus[field] === 1 ? -1 : 0
        this.findWords()
      },
      changePage (number) {
        this.curPage = number
        this.findWords()
      },
      findWords (getTotal = false) {
        let conds = {}
        let has = (key) => {
          if (this.filter[key]) {
            return true
          }
          return false
        }
        if (has('recognized')) {
          conds.recognized = this.filter.recognized
        }
        if (has('word')) {
          conds.word = new RegExp(this.filter.word)
        }
        if (has('sourceUrl')) {
          conds.sourceUrl = new RegExp(this.filter.sourceUrl)
        }
        if (has('sourceSentence')) {
          conds.sourceSentence = new RegExp(this.filter.sourceSentence)
        }
        let vm = this
        if (getTotal) {
          this.count({
            find: {
              ...conds,
              $where: function () {
                return (has('rankMin') ? this.rank >= vm.filter.rankMin && this.rank <= vm.filter.rankMax : true) &&
                  (has('startTime') ? this.createTime.valueOf() > vm.filter.startTime : true) &&
                  (has('endTime') ? this.createTime.valueOf() < vm.filter.endTime : true)
              }
            }
          })
            .then(res => {
              this.pageTotal = res
            })
        }
        let sort = {}
        for (let [key, value] of Object.entries(this.sortStatus)) {
          if (value !== 0) {
            sort[key] = value
          }
        }
        this.search({
          find: {
            ...conds,
            $where: function () {
              return (has('rankMin') ? this.rank >= vm.filter.rankMin && this.rank <= vm.filter.rankMax : true) &&
                (has('startTime') ? this.createTime.valueOf() > vm.filter.startTime : true) &&
                (has('endTime') ? this.createTime.valueOf() < vm.filter.endTime : true)
            }
          },
          sort,
          skip: this.pageSize * (this.curPage - 1),
          limit: this.pageSize
        })
      },
      timer () {
        if (handle) {
          clearTimeout(handle)
        }
        handle = setTimeout(() => {
          this.curPage = 1
          this.findWords(true)
        }, 350)
      }
    }
  }
</script>