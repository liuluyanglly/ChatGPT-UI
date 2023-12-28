<template>
<!--    <div>-->
<!--        <img src="/logo.jpg" style="width: 130px;margin: 0;float: left;">-->
<!--    </div>-->
    <LeftDrawer />
    <div class="chat-container" style="width: 70%">
        <div class="chat-list">
            <el-auto-resizer>
                <template #default="{ height }">
                    <el-scrollbar ref="scrollContainer" :height="height">
                        <div class="welcome-wrapper">
                            <span class="title">联通大模型</span>
                            <span class="label animate__animated animate__bounceInDown">你的AI助手</span>
                        </div>

                        <template v-for="(item, index) in chatList">
                            <div class="chat-list__item" v-if="item.role !== 'system'">
                                <div class="message-card animate__animated animate__fadeInUp"
                                     :class="{ 'is-right': item.role === 'user' }">
                                    <section class="list-item__text">
                                        <MarkdownIt :model-value="item.content"></MarkdownIt>
                                    </section>
                                    <div class="extend-wrapper" v-if="!pending" @click.stop="deleteMessage(index)">
                                        <el-icon color="#F56C6C" size="18px">
                                            <CircleClose/>
                                        </el-icon>
                                    </div>
                                </div>
                            </div>
                        </template>
                    </el-scrollbar>
                </template>
            </el-auto-resizer>
        </div>

        <div class="chat-enter animate__animated animate__fadeInUp">
            <div class="clear-wrapper" :class="{ mini: isMiniClear }" @click="clearHandle">
                <div class="icon">
                    <i-game-icons-magic-broom/>
                </div>
                <div class="tips">新对话</div>
            </div>

            <div class="enter-wrapper">
                <div class="enter-icon">
                    <i-ph-chat-circle-text-light/>
                </div>
                <el-input type="textarea" placeholder="有问题尽管问我..." resize="none" maxlength="2000"
                          enterkeyhint="send"
                          autocorrect="off" show-word-limit :autosize="{ minRows: 1, maxRows: 8 }" v-model="keyword"
                          @focus="focusHandle"
                          @blur="blurHandle" @keydown.enter.prevent="sendMessage"></el-input>

                <div class="is-pending" v-show="pending"></div>
            </div>
        </div>


<!--        <div class="setting-button animate__animated animate__fadeInRight" @click="openSetting">-->
<!--            高级设置-->
<!--            <el-icon size="18px">-->
<!--                <Setting/>-->
<!--            </el-icon>-->
<!--        </div>-->

    </div>
</template>

<script setup>
import {computed, ref} from 'vue'
import {useWindowSize} from '@vueuse/core'
import {useOpenAi} from '@/hooks/useOpenAi'
import {CircleClose, Setting} from '@element-plus/icons-vue'
import {useModal} from '@/hooks/useModal'
import {useSettingStore} from '@/store/setting'
import {storeToRefs} from "pinia";
import MarkdownIt from './markdown-it.vue';
import settingDrawer from './setting-drawer.vue';
import LeftDrawer from './left-drawer.vue';

function openSetting() {
    useModal(settingDrawer)
}

const {height} = useWindowSize()
const clientHeight = computed(() => `${height.value}px`)

const settingStore = useSettingStore()
const {chatList} = storeToRefs(settingStore)

const {
    scrollContainer,
    keyword,
    pending,
    deleteMessage,
    sendMessage,
} = useOpenAi({openSetting})

const isMiniClear = ref(false)

function clearHandle() {
    pending.value = false
    settingStore.clearMessage()
}

function focusHandle() {
    isMiniClear.value = true
}

function blurHandle() {
    isMiniClear.value = false
}
</script>

<style lang="scss" scoped>
@keyframes pending-animation {
  0% {
    transform: translateX(-100%);
  }

  100% {
    transform: translateX(100%);
  }
}

@keyframes blend-animation {
  0% {
    letter-spacing: -30px;
    filter: blur(10px);
  }

  100% {
    letter-spacing: 0px;
    filter: blur(0px);
  }
}

.chat-container {
  display: flex;
  flex-direction: column;
  max-width: 1200px;
  height: v-bind(clientHeight);
  overflow: hidden;
  padding: 30px;
  margin: 0 auto;
  font-size: 16px;
}

.chat-enter {
  display: flex;
  align-items: flex-end;

  .clear-wrapper {
    display: flex;
    align-items: center;
    width: 112px;
    height: 48px;
    color: #fff;
    font-weight: bold;
    background: linear-gradient(90deg, #2870EA 10.79%, #1B4AEF 87.08%);
    border-radius: 24px;
    overflow: hidden;
    cursor: pointer;
    transition: all .3s;
    margin-right: 10px;


    .icon {
      width: 48px;
      height: 48px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
      flex-shrink: 0;

    }

    .tips {
      width: max-content;
      font-size: 16px;
      white-space: nowrap;
    }

    &.mini {
      width: 48px;
    }
  }

  .enter-wrapper {
    position: relative;
    flex: 1;
    padding: 9px 12px;
    border-radius: 24px !important;
    background-color: #fff;
    display: flex;
    overflow: hidden;

    .is-pending {
      position: absolute;
      left: 0;
      bottom: 0;
      width: 100%;
      height: 2px;
      background: linear-gradient(90deg, #2870EA 10.79%, #1B4AEF 87.08%);
      opacity: 0.75;
      animation-name: pending-animation;
      animation-duration: 1.3s;
      animation-timing-function: ease-in-out;
      animation-iteration-count: infinite;
    }

    .enter-icon {
      width: 24px;
      margin-top: 3px;
      font-size: 18px;
    }

    :deep(.el-textarea__inner) {
      min-height: auto !important;
      line-height: 20px;
      box-shadow: none;
      background-color: transparent;
      padding-right: 80px;
      font-size: 16px;


    }

    :deep(.el-input__count) {
      line-height: 20px;
    }
  }
}

.chat-list {
  flex: 1;
  width: 100%;
  overflow: hidden;
  margin-bottom: 40px;

  :deep(.el-scrollbar__view) {
    padding-right: 20px;
  }

  :deep(.chat-list__item) {
    margin-bottom: 20px;


    .message-card {
      width: max-content;
      padding: 10px 16px;
      border-radius: 8px;
    }

    .extend-wrapper {
      position: absolute;
      top: 0;
      right: 0;
      width: max-content;
      height: 24px;
      padding: 0 8px;
      transform: translateX(30px);
      justify-content: center;
      align-items: center;
      display: none;
      cursor: pointer;
    }

    &:hover .extend-wrapper {
      display: flex;
    }

    .is-right {
      color: #fff;
      margin-left: auto;
      background-image: linear-gradient(90deg, #2870EA 10.79%, #1B4AEF 87.08%);

      .extend-wrapper {
        left: 0;
        transform: translateX(-30px);
      }
    }


    p,
    li {
      line-height: 26px;
    }
  }
}

.welcome-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 12vh;
  filter: contrast(30);

  .title {
    color: #111;
    font-weight: 600;
    font-size: 36px;
    margin-bottom: 12px;
    animation-name: blend-animation;
    animation-duration: .9s;
    animation-timing-function: ease-in-out;
  }

  .label {
    font-size: 18px;
    margin-bottom: 4vh;
  }
}

.example-wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-column-gap: 20px;
  margin-bottom: 4vh;

  .item-wrapper {
    flex: 1;
    text-align: center;

    .title {
      margin-bottom: 24px;
    }
  }
}

.tips-wrapper {
  text-align: center;
  margin-bottom: 4vh;
}

.message-card {
  position: relative;
  border-radius: 6px;
  text-align: left;
  outline: transparent solid 1px;
  padding: 20px;
  background-color: #fff;
  max-width: 90%;
  font-size: 16px;
  box-shadow: 0px 0.3px 0.9px rgba(0, 0, 0, 0.12), 0px 1.6px 3.6px rgba(0, 0, 0, 0.16);

}

.setting-button {
  position: fixed;
  right: -1px;
  top: 6vh;
  width: 132px;
  height: 32px;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(90deg, #2870EA 10.79%, #1B4AEF 87.08%);
  border-radius: 4px 0 0 4px;
  overflow: hidden;
  cursor: pointer;
}

@media (max-width: 750px) {

  .chat-container {
    padding: 30px 18px;
  }

  .chat-enter {
    .clear-wrapper {
      width: 48px;
    }

    :deep(.el-textarea__inner) {
      padding-right: 11px !important;
    }

    :deep(.el-input__count) {
      display: none;
    }
  }

  .welcome-wrapper {
    padding-top: 3vh;
  }

  .example-wrapper {
    grid-template-columns: repeat(1, 1fr);

    .message-card {
      margin: 0 auto 20px;
    }
  }

  :deep(.chat-list) {
    margin-bottom: 20px;

    .el-scrollbar__view {
      padding-right: 0;
    }
  }
}
</style>
