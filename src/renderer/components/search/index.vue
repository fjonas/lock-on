<style lang="scss">
    .search {
        .search-wrap {
            padding-left: 110px;
            margin-top: 30px;
            .info {
                color: #bbbec4;
            }
            .result {
                font-size: 15px;
            }
        }
    }
</style>
<template>
    <div class="search">
        <div class="search-wrap">
            <i-input v-model="word" style="width: 50%" @on-change="inputting" @click.native="$event.target.select()"></i-input>
            <div class="info">
                {{info}}
            </div>
            <div class="result">
                {{result}} <span v-if="!result">no results</span>
            </div>
        </div>
        <div>
            <Form ref="form" :model="form" :label-width="110" :rules="rules">
                <Form-item label="word" prop="word">
                    <Input v-model="form.word" placeholder="请输入"></Input>
                </Form-item>
                <Form-item label="definition" prop="definition">
                    <Input v-model="form.definition" placeholder="请输入"></Input>
                </Form-item>
                <Form-item label="ranking" prop="rank">
                    <Rate allow-half v-model="form.rank"></Rate>
                </Form-item>
                <Form-item label="source sentence" prop="sourceSentence">
                    <Input v-model="form.sourceSentence" placeholder="请输入"></Input>
                </Form-item>
                <Form-item label="source url" prop="sourceUrl">
                    <Input v-model="form.sourceUrl" placeholder="请输入"></Input>
                </Form-item>
                <Form-item>
                    <Button type="ghost" @click.native="post">record this word</Button>
                </Form-item>
            </Form>
        </div>
    </div>
</template>
<script type="text/ecmascript-6">
    import webview from '../common/webview.vue'
    import { getUrl } from './baiduConfig'
    import { mapActions } from 'vuex'

    let inputHandle

    let word = ''
    let definition = ''
    let rank = 0
    let recognized = false
    let like = false
    let sourceSentence = ''
    let sourceUrl = ''
    let finded = 0

    export default {
        name: 'search',
        components: {webview},
        data () {
            return {
                word: '',
                dict: null,
                fromLang: 'auto',
                toLang: 'auto',
                info: 'input please',
                result: '',
                form: {
                    word,
                    definition,
                    rank,
                    recognized,
                    like,
                    createTime: new Date(),
                    sourceSentence,
                    sourceUrl,
                    finded
                },
                rules: {
                    word: {
                        required: true,
                        message: 'word was required'
                    },
                    definition: {
                        required: true,
                        message: 'definition was required'
                    }
                }
            }
        },
        methods: {
            ...mapActions({
                postWord: 'words/post'
            }),
            resetForm (reset) {
                this.form = {
                    word: reset ? '' : this.word,
                    definition: reset ? '' : this.result,
                    rank,
                    recognized,
                    like,
                    createTime: new Date(),
                    sourceSentence,
                    sourceUrl,
                    finded
                }
                this.$refs.form.resetFields()
            },
            search () {
                if (!this.word) {
                    this.info = 'input please'
                    this.result = ''
                    return
                }
                let url = getUrl(this.word, this.fromLang, this.toLang)
                this.info = 'searching...'
                this.$http.get(url)
                .then(res => {
                    this.result = res.translation[0]
                    this.info = 'done'
                    this.resetForm(false)
                })
            },
            inputting () {
                this.info = 'inputting...'
                if (inputHandle) {
                    clearTimeout(inputHandle)
                }
                inputHandle = setTimeout(() => {
                    this.search()
                }, 550)
            },
            post () {
                this.$refs.form.validate((valid) => {
                    if (valid) {
                        this.form.createTime = new Date()
                        this.postWord(this.form)
                        .then(() => {
                            this.$Message.success('succeed')
                            this.resetForm(true)
                        }, () => {
                            this.$Message.error('error')
                        })
                    } else {
                        this.$Message.error('invalid form')
                    }
                })
            }
        }
    }
</script>
