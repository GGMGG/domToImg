<template>
  <div class="common-layout">
    <el-container class="common-container">
      <el-header class="common-header">
        <h1>{{ msg }}</h1>
        <el-button type="primary" @click="createImage('png')" class="create-btn">生成图片(PNG)</el-button>
        <el-button type="primary" @click="createImage('jpg')" class="create-btn">生成图片(JPG)</el-button>
        <el-button type="primary" @click="createImage('svg')" class="create-btn">生成图片(SVG)</el-button>
        <el-button v-if="showDownload" type="primary" @click="downLoadImg" class="create-btn">下载图片</el-button>
      </el-header>
      <el-container class="common-container">
        <el-main class="common-main">
          <el-steps :active="stepActive" align-center finish-status="success">
            <el-step title="编写" @click="steActive(0)" :description="editDescription" />
            <el-step title="预览" @click="steActive(1)" :description="prewDescription" />
            <el-step title="生成" @click="steActive(2)" :description="createEditDescription" :status="createSuccessStr" />
          </el-steps>
          <div>
            <el-row v-show="stepActive === 0">
              <el-col :span="24" class="common-col">
                <htmlCodeEdit v-model="htmlCode" editor-width="75" editor-height="500" style="margin-left: 12.5%" />
              </el-col>
            </el-row>
            <el-row v-show="stepActive === 1">
              <el-col :span="24" class="common-col">
                <div id="domNode" class="common-content" v-html="htmlCode"></div>
              </el-col>
            </el-row>
            <el-row v-show="stepActive === 2">
              <el-col v-show="createSuccess" :span="24" class="common-col common-img">
                <div class="common-content">
                  <img :src="dataUrl" />
                </div>
              </el-col>
            </el-row>
          </div>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import domtoimage from "dom-to-image";
import { ElMessage } from "element-plus";
import htmlCodeEdit from "@/components/htmlCodeEdit/htmlCodeEdit.vue";

/**
 * props
 */
defineProps<{ msg: string }>();

// 步骤条index
const stepActive = ref(0);
// html代码
const htmlCode = ref(`<!-- example: please wirte css under .common-content; if you want to change img size, please resize .common-content -->
<article class='article'>
  <h1 class='h1'>DomToImg</h1>
  <p class='p'>GGMGG</p>
</article>

<style>
.common-content {
  .article {
    background: linear-gradient(to right, hsl(98 100% 62%), hsl(204 100% 59%) );
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-align: center;
  }

  .h1 {
    font-size: 10vmin;
    line-height: 1.1;
  }

  .p {
    font-family: "Dank Mono", ui-monospace, monospace;
    font-size: 16px;
    text-align: right;
    margin-right:10px;
    margin-top:10px;
  }
}
</style>`);
// 生成的图片地址
const dataUrl = ref("");
// 生成的图片类型
const fileType = ref("");
// 是否生成成功
const createSuccess = ref(false);
// 下载按钮
const showDownload = ref(false);
// 步骤条描述
const editDescription = ref("");
const prewDescription = ref("");
const createEditDescription = ref("");
const createSuccessStr = ref("");

/**
 * 设置步骤条
 * @param actived
 */
const steActive = (actived: number) => {
  if (actived === 0 || actived === 1) {
    createSuccessStr.value = "";
    showDownload.value = false;
  }

  if (actived === 1 && !htmlCode.value) {
    showError("请输入html代码");
    return;
  }

  if (actived === 2) {
    showError("请点击按钮生成图片");
    return;
  }

  if (actived === 1 && htmlCode.value) {
    editDescription.value = "编写成功";
    prewDescription.value = "预览成功";
  }

  stepActive.value = actived || 0;
};

/**
 * 生成图片
 * @param type
 */
const createImage = (type: string) => {
  if (!htmlCode.value) {
    showError("请输入html代码");
    return;
  }

  if (stepActive.value !== 1) {
    showError("请先点击预览步骤");
    return;
  }

  createEditDescription.value = "生成类型：" + type;
  if (type === "png") {
    createImagePng();
    return;
  }

  if (type === "jpg") {
    createImageJpg();
    return;
  }

  if (type === "svg") {
    createImageSvg();
    return;
  }

  showError("无效的生成类型");
  return;
};

/**
 * 生成png
 * @param node
 */
const createImagePng = () => {
  const node = document.getElementById("domNode");
  domtoimage
    .toPng(node)
    .then(function (value: any) {
      createSuccessFn(value, "png");
    })
    .catch(function (error: any) {
      createFailFn(error);
    });
};

/**
 * 生成jpg
 * @param node
 */
const createImageJpg = () => {
  const node = document.getElementById("domNode");
  domtoimage
    .toJpeg(node)
    .then(function (value: any) {
      createSuccessFn(value, "jpeg");
    })
    .catch(function (error: any) {
      createFailFn(error);
    });
};

/**
 * 生成svg
 * @param node
 */
const createImageSvg = () => {
  const node = document.getElementById("domNode");
  domtoimage
    .toSvg(node)
    .then(function (value: any) {
      createSuccessFn(value, "svg");
    })
    .catch(function (error: any) {
      createFailFn(error);
    });
};

/**
 * 生成成功
 * @param value
 */
const createSuccessFn = (value: any, type: string) => {
  dataUrl.value = value;
  fileType.value = type;
  createSuccess.value = true;
  createSuccessStr.value = "finish";
  stepActive.value = 2;
  showDownload.value = true;
};

/**
 * 生成失败
 * @param error
 */
const createFailFn = (error: any) => {
  createSuccess.value = false;
  showError("生成失败：" + error.toString());
};

/**
 * 下载生成的图片
 */
const downLoadImg = () => {
  if (!dataUrl.value) {
    showError("未找到生成的图片");
    return;
  }

  // 获取文件
  const file = base64toFile();
  if (!file) {
    showError("未找到生成的图片");
    return;
  }

  // 创建一个a标签
  const aTag = document.createElement("a");
  aTag.style.display = "none";
  aTag.download = file.name;
  // 获取url
  const href = URL.createObjectURL(file);
  aTag.href = href;
  aTag.click();
  // 释放url
  URL.revokeObjectURL(href);
};

/**
 * base64转文件
 * @param fileType
 */
const base64toFile = () => {
  // 截取
  const arr = dataUrl.value.split(",");
  const mime = arr[0].match(/:(.*?);/)[1];
  // base64编码
  const bstr = window.atob(arr[1]);
  let n = bstr.length;
  let u8arr = new Uint8Array(n);
  while (n--) {
    // unicode 编码
    u8arr[n] = bstr.charCodeAt(n);
  }

  return new File([u8arr], `image.` + fileType.value, {
    type: mime,
  });
};

/**
 * 错误提示
 * @param msg
 */
const showError = (msg: string) => {
  ElMessage({
    showClose: true,
    message: msg,
    type: "error",
  });
};
</script>

<style scoped>
.common-layout {
  height: 100%;
  width: 100%;

  .common-container,
  .common-main {
    height: 100%;
    overflow: hidden;
  }

  .common-container .el-header {
    height: 100%;
  }

  .create-btn {
    margin-left: 20px;
    border-radius: 5px;
  }

  .common-col {
    margin: 10px;
    padding: 20px;
  }

  .common-img {
    border: 1px solid rgb(232, 163, 163);
    border-radius: 5px;
  }

  .common-content {
    /* width: 500px; */
    height: 500px;
  }
}
</style>
