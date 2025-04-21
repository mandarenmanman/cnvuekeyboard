<template>
    <div class="simple-keyboard-container">
        <div class="candidates" v-if="candidates.length">
            <div class="candidates-row">
                <button v-if="currentPage > 1" @click="prevPage" class="candidate-page-btn pretty-btn">
                    <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M10 12L6 8L10 4" stroke="#666" stroke-width="2" stroke-linecap="round"
                            stroke-linejoin="round" />
                    </svg>
                    上一页
                </button>
                <span v-for="(item, index) in pagedCandidates" :key="index" class="candidate-item"
                    @click="selectCandidate(item)">
                    {{ item }}
                </span>
                <button v-if="currentPage < totalPages" @click="nextPage" class="candidate-page-btn pretty-btn">
                    下一页
                    <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M6 4L10 8L6 12" stroke="#666" stroke-width="2" stroke-linecap="round"
                            stroke-linejoin="round" />
                    </svg>
                </button>
                <span class="candidate-page-info">{{ currentPage }} / {{ totalPages }}</span>
            </div>
        </div>
        <div :class="keyboardClass"></div>

    </div>
</template>

<script>
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";
import cnchar from "cnchar";
import "cnchar-input";
import "cnchar-words";
import "cnchar-poly";
import "cnchar-idiom";
export default {
    name: "SimpleKeyboard",
    props: {
        keyboardClass: {
            type: String,
            default: "simple-keyboard"
        },
        input: {
            type: String,
            default: ""
        },
        preventMouseDownDefault: {
            type: Boolean,
            default: true
        }
    },
    data: () => ({
        keyboard: null,
        currentLayout: 'default',
        rawInput: '',
        spInput: '',//拼音输入的内容
        crInput: '',//汉字输入的内容
        candidates: [],
        currentPage: 1,      // 新增：当前页
        pageSize: 10         // 新增：每页显示数量
    }),
    computed: {
        keyboardDisplayOptions() {

            //记录当前input的内容
            //如果是中文，就显示拼音
            //如果是英文，就显示英文
            //如果是符号，就显示符号
            if (this.currentLayout === 'chinese') {
                this.spInput = this.getInput();
                console.log('spInput:', this.spInput);
            }

            return {
                '{bksp}': '⌫',
                '{enter}': '⏎',
                '{space}': '空格',
                '{clear}': '清空',
                '{lang}': this.currentLayout !== 'chinese' && this.currentLayout !== 'chineseShift' ? 'En' : '中',
                '{symbol}': '符号',
                '{shift}': '⇧'
            };
        },
        pagedCandidates() {   // 新增：当前页候选词
            const start = (this.currentPage - 1) * this.pageSize;
            return this.candidates.slice(start, start + this.pageSize);
        },
        totalPages() {        // 新增：总页数
            return Math.ceil(this.candidates.length / this.pageSize) || 1;
        }
    },
    mounted() {
        this.initializeKeyboard();
        if (this.input) {
            this.keyboard.setInput(this.input);
        }
        // 延迟强制更新布局
        this.$nextTick(() => {
            this.keyboard.setOptions({
                layoutName: this.currentLayout,
                display: this.keyboardDisplayOptions
            });
        });
    },
    methods: {
        initializeKeyboard() {
            console.log('initializeKeyboard:', this.currentLayout);
            this.keyboard = new Keyboard(`.${this.keyboardClass}`, {
                display: this.keyboardDisplayOptions,
                layout: {
                    'default': [
                        "1 2 3 4 5 6 7 8 9 0",
                        "q w e r t y u i o p",
                        "a s d f g h j k l",
                        "z x c v b n m",
                        "{shift} {lang} {symbol} {space} {clear} {bksp} {enter}"
                    ],
                    'shift': [
                        "1 2 3 4 5 6 7 8 9 0",
                        "Q W E R T Y U I O P",
                        "A S D F G H J K L",
                        "Z X C V B N M",
                        "{shift} {lang} {symbol} {space} {clear} {bksp} {enter}"
                    ],
                    'chinese': [
                        "1 2 3 4 5 6 7 8 9 0",
                        "q w e r t y u i o p",
                        "a s d f g h j k l",
                        "z x c v b n m",
                        "{shift} {lang} {symbol} {space} {clear} {bksp} {enter}"
                    ],
                    'chineseShift': [
                        "1 2 3 4 5 6 7 8 9 0",
                        "Q W E R T Y U I O P",
                        "A S D F G H J K L",
                        "Z X C V B N M",
                        "{shift} {lang} {symbol} {space} {clear} {bksp} {enter}"
                    ],
                    'symbols': [
                        "~ ! @ # $ % ^ & * ( )",
                        "- _ = + [ ] { } | \\",
                        ": ; ' \" < > , . / ?",
                        "` · ￥ ！ ， 。 《 》 ～",
                        "{lang} {symbol} {space} {clear} {bksp} {enter}"
                    ]
                },
                layoutName: this.currentLayout,
                onChange: this.onChange,
                onKeyPress: this.onKeyPress,
                preventMouseDownDefault: this.preventMouseDownDefault
            });
        },

        onChange(input) {
            this.$emit("onChange", input);
            if (this.currentLayout === 'chinese') {
                this.rawInput = input;
                this.updateCandidates(input);
            } else {
                console.log('updateCandidates:', input);
            }
        },

        onKeyPress(button) {
            this.$emit("onKeyPress", button);
            if (button === "{enter}") {
                this.$emit("onEnter");
            } else if (button === "{clear}") {
                this.clearInput();
            } else if (button === "{lang}") {
                this.toggleLanguage();
            } else if (button === "{symbol}") {
                this.toggleSymbols();
            } else if (button === "{shift}") {
                this.handleShift();
            }
        },

        handleShift() {
            // 判断当前布局并切换大小写
            if (this.currentLayout === 'default') {
                this.currentLayout = 'shift';
            } else if (this.currentLayout === 'shift') {
                this.currentLayout = 'default';
            } else if (this.currentLayout === 'chinese') {
                this.currentLayout = 'chineseShift';
            } else if (this.currentLayout === 'chineseShift') {
                this.currentLayout = 'chinese';
            }
            this.keyboard.setOptions({
                layoutName: this.currentLayout,
                display: this.keyboardDisplayOptions
            });
        },

        toggleLanguage() {
            if (this.currentLayout === 'symbols') {
                this.currentLayout = 'default';
            } else {
                this.currentLayout = this.currentLayout === 'chinese' ? 'default' : 'chinese';
            }
            this.keyboard.setOptions({
                layoutName: this.currentLayout,
                display: this.keyboardDisplayOptions
            });
            this.$emit("onLangChange", this.currentLayout);
            this.rawInput = '';
            this.candidates = [];
        },

        toggleSymbols() {
            this.currentLayout = this.currentLayout === 'symbols' ? 'default' : 'symbols';
            this.keyboard.setOptions({
                layoutName: this.currentLayout,
                display: this.keyboardDisplayOptions
            });
            this.$emit("onLangChange", this.currentLayout);
            this.rawInput = '';
            this.candidates = [];
        },

        updateCandidates(pinyinInput) {

            pinyinInput = pinyinInput.substring(this.spInput.length, pinyinInput.length);
            console.log('updateCandidates:', pinyinInput);

            if (!pinyinInput || !/^[a-z\s]+$/i.test(pinyinInput)) {
                this.candidates = [];
                this.currentPage = 1; // 重置页码
                return;
            }



            try {
                const result = cnchar.input(pinyinInput);
                const assoc = result[0]?.association?.[0]?.split(' ') || [];
                const words = result[0]?.words?.[0]?.split('') || [];

                this.crInput = pinyinInput;

                this.candidates = [...assoc, ...words];
                this.currentPage = 1; // 新增：每次更新重置到第一页
            } catch (e) {
                this.candidates = [];
                this.currentPage = 1;
            }
        },
        prevPage() { // 新增
            if (this.currentPage > 1) this.currentPage--;
        },
        nextPage() { // 新增
            if (this.currentPage < this.totalPages) this.currentPage++;
        },
        selectCandidate(candidate) {

            console.log('selectCandidate:', candidate);
            console.log('this.crInput:', this.crInput);
            console.log('this.spInput:', this.spInput);
            console.log('this.rawInput:', this.rawInput);

            this.spInput = this.spInput + candidate;
            this.keyboard.setInput(this.spInput);
            this.$emit("onChange", this.spInput);
            this.rawInput = '';
            this.candidates = [];
            this.currentPage = 1; // 重置
        },
        clearInput() {
            if (this.keyboard) {
                this.keyboard.clearInput();
                this.candidates = [];
                this.currentPage = 1;
            }
        },

        getInput() {
            return this.keyboard ? this.keyboard.getInput() : '';
        },

        setInput(input) {
            if (this.keyboard) {
                this.keyboard.setInput(input);
            }
        }
    },
    watch: {
        input(newInput) {
            if (this.keyboard && this.keyboard.getInput() !== newInput) {
                this.keyboard.setInput(newInput);
            }
        }
    },
    beforeUnmount() {
        if (this.keyboard) {
            this.keyboard.destroy();
            this.keyboard = null;
        }
    }
};
</script>

<style scoped>
.simple-keyboard-container {
    width: 100%;
    box-sizing: border-box;
    padding: 10px;
    background: #f7f7f7;
    border-top: 1px solid #ddd;
}

:deep(.simple-keyboard) {
    width: 100%;
}

.candidates {
    margin-top: 8px;
    padding: 6px;
    background: #ffffff;
    border: 1px solid #ddd;
    border-radius: 6px;
    /* 移除 display:flex，改为子元素控制布局 */
}

.candidates-row {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 10px 8px;
    width: 100%;
}

.candidate-item {
    padding: 10px 20px;
    background: #eee;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.2s;
    font-size: 20px;
    min-width: 48px;
    min-height: 44px;
    display: flex;
    align-items: center;
    justify-content: center;
    user-select: none;
    margin-bottom: 2px;
}

.candidate-item:hover {
    background: #dcdcdc;
}

.candidate-page-btn.pretty-btn {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    padding: 8px 18px;
    background: linear-gradient(90deg, #f0f0f0 0%, #e0e7ff 100%);
    border: none;
    border-radius: 20px;
    box-shadow: 0 2px 8px rgba(120, 120, 180, 0.08);
    color: #333;
    font-weight: 500;
    font-size: 18px;
    cursor: pointer;
    transition: background 0.2s, box-shadow 0.2s, color 0.2s;
    outline: none;
}

.candidate-page-btn.pretty-btn:hover {
    background: linear-gradient(90deg, #e0e7ff 0%, #f0f0f0 100%);
    color: #2a4fa2;
    box-shadow: 0 4px 16px rgba(80, 120, 200, 0.13);
}

.candidate-page-btn.pretty-btn svg {
    vertical-align: middle;
    margin: 0 2px;
}

.candidate-page-info {
    margin-left: 12px;
    color: #888;
    font-size: 18px;
    font-weight: 500;
    min-width: 60px;
    text-align: center;
    letter-spacing: 1px;
}

:deep(.simple-keyboard .hg-button:active),
:deep(.simple-keyboard .hg-button.hg-activeButton) {
    background: #4caf50 !important;
    color: #fff !important;
    border-color: #388e3c !important;
    box-shadow: 0 2px 8px rgba(76, 175, 80, 0.15) !important;
    transition: background 0.1s;
}
:deep(.simple-keyboard .hg-button[data-skbtn="{shift}"]) {
    background: #e0f7fa !important;
    color: #00796b !important;
}
</style>
