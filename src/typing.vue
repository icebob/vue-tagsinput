<template>
    <input v-if="typing"
        :class="klass.input"
        v-el:input
        type="text"
        v-model="text"
        @blur="finish"
        @keydown="keyPress"
        :style="{width: 2 + charLen(text) + 'ch'}" />
    <span v-else
        :class="klass.gap" @click="begin">&nbsp;</span>
</template>
<style scoped>
input.input {
    outline: none;
    box-shadow: none;
    border: none;
    background-color: transparent;
    font-family: monospace;
    padding: 0.5ch;
}
.gap {
    flex-basis: 7px;
}
</style>
<script>
import Vue from 'vue'
import {KEY_CODE, E} from './lib'
export default {
    props: {
        index: {
            type: Number,
            required: true
        }
    },
    data() {
        return {
            typing: false,
            text: ''
        }
    },
    computed: {
        klass() {
            return this.$parent.klass
        }
    },
    watch: {
        typing(val) {
            val && Vue.nextTick(_ => this.$els.input.focus())
        }
    },
    events: {
        [E`active`](index) {
            this.typing = index === this.index
        }
    },
    methods: {
        begin() {
            this.typing = true
        },
        finish() {
            let result = this.text.trim()
            result && this.$dispatch(E`insert`, this.index, result)
            this.typing = false
            this.text = ''
        },
        charLen(str) {
            let charNum = 0
            for (let i = 0; i < str.length; ++i) {
                charNum += str.charCodeAt(i) > 127 ? 2 : 1
            }
            return charNum
        },
        keyPress(e) {
            let $input = this.$els.input
            let cursor = $input.selectionStart
            let valLen = $input.value.length
            let key = e.keyCode
            let native = false

            if (key === KEY_CODE.RIGHT && cursor === valLen) {
                valLen === 0 && this.$dispatch(E`activeOther`, this.index + 1)
            } else if (key === KEY_CODE.LEFT && cursor === 0) {
                valLen === 0 && this.$dispatch(E`activeOther`, this.index - 1)
            } else if (key === KEY_CODE.BACKSPACE && cursor === 0) {
                this.$dispatch(E`remove`, this.index - 1)
            } else if (key === KEY_CODE.TAB) {
                this.finish()
                this.$dispatch(E`activeOther`, this.index)
            } else native = true

            !native && e.preventDefault()
        }
    }
}
</script>