<template>
    <div style="width: 20%; height: calc(100vh );background-color: #ffffff;box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5); float: left;">
        <div style="margin-left: 10px;display: flex;flex-direction: column;">
<!--            <img src="/logo.jpg" style="width: 130px;margin: 10px 10px 10px 10px;float: left;">-->
        </div>
        <div style="margin-left: 10px;display: flex;flex-direction: column;margin-top: 10px">
            <label style="font-size: 24px;margin-left: 7px; font-weight: bold;">Chat-Unicom</label>
<!--            <label >最好用的AI</label>-->
        </div>
        <div class="example-wrapper ">
            <div class="item-wrapper custom-style animate__animated animate__bounceInDown"
                 style="animation-delay: .3s;">
                <div class="title" style="margin-top: 10px">🧐 提出复杂问题</div><br>
                <div class="message-card" style="margin-bottom: 10px">"我可以为我挑剔的只吃橙色食物的孩子做什么饭?"</div>
            </div>
            <div class="item-wrapper custom-style animate__animated animate__bounceInDown"
                 style="animation-delay: .5s;" >
                <div class="title" style="margin-top: 10px">🙌 获取更好的答案</div><br>
                <div class="message-card" style="margin-bottom: 10px">"销量最高的 3 种宠物吸尘器有哪些优点和缺点?"</div>
            </div>
            <div class="item-wrapper custom-style animate__animated animate__bounceInDown"
                 style="animation-delay: .7s;">
                <div class="title" style="margin-top: 10px">🎨 获得创意灵感</div><br>
                <div class="message-card" style="margin-bottom: 10px">"以海盗的口吻写一首关于外太空鳄鱼的俳句?"</div>
            </div>
        </div>
        <div style="margin-top: 50%;margin-left: 60%;margin-bottom: 5%" @click="openSetting">
            <label class="setting-lab" style="margin-right: 5px">高级设置</label>
            <el-icon size="18px">
                <Setting/>
            </el-icon>
        </div>

    </div>


</template>

<script setup>
import {computed, ref} from 'vue'
import {storeToRefs} from 'pinia';
import {useSettingStoreWithOut} from '@/store/setting';
import {ElMessage} from 'element-plus';
import {Setting} from "@element-plus/icons-vue";
import {useModal} from "@/hooks/useModal.js";
import settingDrawer from "@/components/setting-drawer.vue";

const settingStore = useSettingStoreWithOut()
const {systemInfo, apiKey, modelMap, currentModel, chatList} = storeToRefs(settingStore)

const drawerVisible = ref(true)
const drawerSize = computed(() => {
    if (window.innerWidth > 750) return '400px'
    return '100%'
})

const promptVal = ref('')
function openSetting() {
    useModal(settingDrawer)
}
function choosePrompt(val) {
    promptVal.value = val
    systemInfo.value.content = val
}

function changeApiKey() {
    settingStore.setApiKey(apiKey.value)
}

function chooseJsonFile() {
    const input = document.createElement('input')
    input.type = 'file'
    input.accept = '.json'
    input.multiple = false
    input.onchange = (event) => {
        const [file] = event.target.files
        readFile(file)
        input.remove()
    }
    input.click()

    function readFile(file) {
        const reader = new FileReader(); // 创建 FileReader 对象
        reader.onload = function (e) {
            try {
                const content = e.target.result; // 获取文件内容
                const jsonData = JSON.parse(content); // 将内容解析为 JSON 对象

                settingStore.initChatList(jsonData)
                ElMessage.success('导入成功')
            } catch (err) {
                ElMessage.success('导入失败')

            }

        }
        reader.readAsText(file); // 以文本格式读取文件内容
    }
}

// 导出Markdown
function downloadMdHandle() {
    // 生成 markdown 字符串
    const messages = [systemInfo.value, ...chatList.value]
    const markdown = messages.reduce((acc, item) => `${acc}> ${item.role}\n\n${item.content}\n\n`, '');
    const filename = `chat-history-${new Date().valueOf()}.md` // 保存文件的名称
    const blob = new Blob([markdown], {type: 'text/markdown'}) // 将 markdown 文本转化为 Blob 对象
    downloadFn(URL.createObjectURL(blob), filename)
}

// 导出JSON
function downloadJsonHandle() {
    const messages = [systemInfo.value, ...chatList.value]
    // 将 JSON 数据转换为字符串并创建 Blob 对象
    const blob = new Blob([JSON.stringify(messages)], {type: "application/json"});
    // 保存文件的名称
    const filename = `chat-history-${new Date().valueOf()}.json`
    downloadFn(URL.createObjectURL(blob), filename)
}

// 下载文件
function downloadFn(href, fileName) {
    const link = document.createElement('a') // 创建下载链接
    link.style.display = 'none'
    link.download = fileName
    link.href = href
    link.click() // 模拟点击下载链接
    link.remove
}

</script>

<style lang="scss">
.drawer-wrapper {
  background-image: url('/static/bg-sprite.png');
  background-position: 0% 0%;
  background-size: 300% 200%;

  .el-drawer__header {
    margin-bottom: 10px;
    --el-drawer-padding-primary: 16px 20px 0;
  }

  .el-drawer__body {
    --el-drawer-padding-primary: 16px 20px;
  }
}
</style>

<style lang="scss" scoped>
:deep(.el-alert) {
  margin-bottom: 12px;
}

:deep(.el-alert__content) {
  margin: 0 auto;
}

.custom-style {
    border: 2px solid skyblue;
    border-radius: 10px;
    margin-top: 20px;
    margin-left: 4%;
    width: 88%;
}

.setting-lab:hover {
    cursor: pointer; /* 设置手指样式 */
    color: skyblue; /* 设置字体颜色变化 */
    font-weight: bold;
}
</style>