<template>
    <span>
        <img v-if="from === 'bot'" style="float: left; padding-top: 10px; padding-left: 5px; margin-right: 10px; width: 30px; height: 30px; border-radius:15px;" src="https://asldemo.s3-eu-west-1.amazonaws.com/Screenshot+2019-06-10+at+19.53.45.png" alt="">

        <span class="bubble" :class="{'me': from === 'me', 'loading': loading}">
        <template v-for="line in text.split('\n')">{{line}}<br></template>

        <button name="playButton" v-if="from == 'bot' && mp3url != false" class="btn btn-primary btn-sm" @click.prevent="audio.isPlaying ? pause(audio) : play(audio)" v-for="audio in audios" :key="audio.id"><span class="fa fa-play-circle-o"></span>

            <img v-if="mp3url!=false && from == 'bot'" style="width:15px; height:15px;" src="https://cdn3.iconfinder.com/data/icons/iconic-1/32/play_alt-512.png" alt="">
            <!--<audio id="audio" src=""></audio>-->
        </button>
        <br v-if="imageUrl != false && from =='bot'"/>
        <img style="margin-top: 15px; width: 100%; max-width: 200px;" v-if="imageUrl != false && from =='bot'" v-bind:src="imageUrl" alt="">

        <span style="font-size: 0.8em" v-if="pdfUrl != false && from =='bot'">
            <br>
            <a v-bind:href="pdfUrl" target="_blank">클릭해서 PDF 보기</a>
        </span>
        <span v-if="videoUrl != false && from =='bot'">
            <br>
           <video v-bind:src="videoUrl" id="video-container" style="width:100%;max-width: 300px;" controls></video>
            <br>
        </span>

    </span>
        <img v-if="from == 'me'" style="float: right; padding-top: 30px; position: absolute; right: 5%;  width: 30px; height: 30px; border-radius:15px;" src="https://cdn2.iconfinder.com/data/icons/business-management-52/96/Artboard_20-512.png">

    </span>

</template>

<style lang="sass" scoped>

.bubble
    padding: 12px
    border-radius: 40px
    color: #202124
    border: 1px solid #E8EAED
    display: inline-block
    word-break: break-all
    word-wrap: break-word
    position: relative
    background-color: white
    max-width: 60%

    &::before
        content: url("data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjRweCIgaGVpZ2h0PSIyOHB4IiB2aWV3Qm94PSIwIDAgMjQgMjgiIHZlcnNpb249IjEuMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayI+CiAgICA8ZyBzdHJva2U9Im5vbmUiIHN0cm9rZS13aWR0aD0iMSIgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj4KICAgICAgICA8ZyB0cmFuc2Zvcm09InRyYW5zbGF0ZSgtMjQuMDAwMDAwLCAtMTAxLjAwMDAwMCkiPgogICAgICAgICAgICA8ZyB0cmFuc2Zvcm09InRyYW5zbGF0ZSgyNC4wMDAwMDAsIDEwMS4wMDAwMDApIj4KICAgICAgICAgICAgICAgIDxwYXRoIGQ9Ik0wLjUsMS4yMTMzNzE0OCBDMC41LDcuNjA2NjkwNjcgMC41LDguNzU1Nzk0NzggMC41LDI2LjkzMjQ3OTIgTDIyLjU4NjkzMDIsOC41MjY3MDQwNCBDMTMuMDQwODkxNSw4LjU4NTgzODUzIDUuNjY5NjIyMDQsNi4xNTI1ODA2MyAwLjUsMS4yMTMzNzE0OCBaIiBzdHJva2U9IiNFOEVBRUQiPjwvcGF0aD4KICAgICAgICAgICAgICAgIDxwb2x5Z29uIGZpbGw9IiNGRkZGRkYiIGZpbGwtcnVsZT0ibm9uemVybyIgcG9pbnRzPSIxIDkgMjQgOSAxIDI4Ij48L3BvbHlnb24+CiAgICAgICAgICAgIDwvZz4KICAgICAgICA8L2c+CiAgICA8L2c+Cjwvc3ZnPg==")
        position: absolute
        left: 0
        top: 0
        margin-top: -9px
        margin-left: -1px

    &:not(.me)
        margin-bottom: 10px
        float: left

    &.me
        @media(min-width: 370px)
            position: absolute
            right: 10%

        @media (min-width: 1300px)
            position: absolute
            right: 8%


        background-color: #F1F3F4
        border: 1px solid #F1F3F4
        color: #5F6368
        margin-top: 24px

        &::before
            content: ''

        &::after
            content: url("data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMzNweCIgaGVpZ2h0PSIyOHB4IiB2aWV3Qm94PSIwIDAgMzMgMjgiIHZlcnNpb249IjEuMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayI+CiAgICA8ZyBzdHJva2U9Im5vbmUiIHN0cm9rZS13aWR0aD0iMSIgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj4KICAgICAgICA8ZyB0cmFuc2Zvcm09InRyYW5zbGF0ZSgtMzU1LjAwMDAwMCwgLTE2NS4wMDAwMDApIiBmaWxsPSIjRjFGM0Y0Ij4KICAgICAgICAgICAgPGcgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoMC4wMDAwMDAsIDE2NS4wMDAwMDApIj4KICAgICAgICAgICAgICAgIDxwYXRoIGQ9Ik0zNTUsMCBDMzU1LDMgMzU1LDEyLjMzMzMzMzMgMzU1LDI4IEwzODgsOCBDMzczLjMzMzMzMyw4LjMzMzMzMzMzIDM2Mi4zMzMzMzMsNS42NjY2NjY2NyAzNTUsMCBaIiB0cmFuc2Zvcm09InRyYW5zbGF0ZSgzNzEuNTAwMDAwLCAxNC4wMDAwMDApIHNjYWxlKC0xLCAxKSB0cmFuc2xhdGUoLTM3MS41MDAwMDAsIC0xNC4wMDAwMDApIj48L3BhdGg+CiAgICAgICAgICAgIDwvZz4KICAgICAgICA8L2c+CiAgICA8L2c+Cjwvc3ZnPg==")
            position: absolute
            right: 0
            top: 0
            margin-top: -9px
            margin-right: -1px

    &.loading
        width: 20px
        height: 16px
        animation: loading 1.5s ease infinite

@keyframes loading
    0%
        opacity: 0
    50%
        opacity: 1
    100%
        opacity: 0
</style>

<script>
export default {
    name: 'Bubble',
    props: ['text', 'from', 'loading', 'mp3url', 'imageUrl', 'pdfUrl', 'videoUrl', 'componentName'],
    data: function() {
        return {
            hey: 'hey',
            isPlaying: false,
            file: null
        }
    },
    computed: {
        audios: function() {
            return [
                {
                    id: 'mp3',
                    name: 'mp3name',
                    file: new Audio(this.mp3url),
                    isPlaying: false
                }
            ]
        }
    },
    methods: {
        play(audio) {
            audio.isPlaying = true;
            audio.file.play();
        },
        pause (audio) {
            audio.isPlaying = false;
            audio.file.pause();
        }
     }
}
</script>