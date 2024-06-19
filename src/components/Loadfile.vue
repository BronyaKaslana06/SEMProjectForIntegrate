<script setup lang="ts">
import { ref } from 'vue'
import {
  ElMessage,
  ElMessageBox,
  genFileId,
  UploadRequestHandler,
  UploadRequestOptions,
  UploadUserFile,
  ElLoading
} from 'element-plus'
import { useRouter } from "vue-router";
import { Plus } from '@element-plus/icons-vue'
import type { UploadInstance, UploadProps, UploadRawFile } from 'element-plus'
import axiosInstance from "~/Axios/request";

const backToMain = () => {
  window.location.href = "http://111.231.168.12";
};

const upload = ref<UploadInstance>()
const fileList = ref<UploadUserFile[]>([])
const router = useRouter()

const exceedHandler: UploadProps['onExceed'] = (files) => {
  console.log(files)
  ElMessageBox.alert('一次只能上传一个文件.', 'Warning', {
    confirmButtonText: 'OK',
  })

  upload.value!.clearFiles()
  const file = files[0] as UploadRawFile
  file.uid = genFileId()
  upload.value!.handleStart(file)
}

// 用户确定上传
const submitConfirm = () => {
  console.log('submitUpload')
  upload.value!.submit()
}

// 上传前的文件格式检查
const beforeDataUpload: UploadProps['beforeUpload'] = (rawFile) => {
  if (rawFile.type !== 'text/plain') {
    ElMessage.error('数据必须以txt格式进行上传！')
    return false
  }

  // TODO：其他文件的格式限制可以写在这里

  return true
}

// 自定义上传
const fileUpload = (options: UploadRequestOptions) => {
  // 准备上传文件
  // debugger
  console.log(options)
  const file = options.file;
  const formData = new FormData();
  formData.append("file", file);

  // 加载动画
  const loading = ElLoading.service({
    lock: true,
    text: '正在上传与计算中',
    background: 'rgba(0, 0, 0, 0.7)',
  })
  axiosInstance.post('/data/', formData, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  }).then(res => {
    // 关闭加载动画
    loading.close()
    console.log(res)
    // 上传成功后的处理
    if (res.data.code === 200) {
      ElMessage.success({
        message: '上传成功!',
        type: 'warn',
      })
      // 跳转到结果页面
      let batchID = res.data.data

      router.push(`/calculating/${batchID}`)
    } else if (res.data.code === 400) {
      ElMessage.error({
        message: res.data.message + '-错误码：' + res.data.code,
        type: 'error',
      })
    }

  }, err => {
    // 关闭加载动画
    loading.close();
    console.log(err)
    // 出现错误时的处理
    ElMessage.error({
      message: err.data.message + '错误码：' + err.data.code,
      type: 'error',
    })
  })
}
</script>

<template>
  <div style="position: fixed; right: 10px; top: 15px; z-index: 1000;">
    <el-button type="primary" @click="backToMain" style="position: absolute; right: 0; top: 0;">返回主页</el-button>
  </div>
  <div id="contentContainer" class="flex-content-around h-full">
    <div id="upLoaderContainer" class="flex-col" style="margin: auto; padding: 0 500px">
      <el-upload ref="upload" class="avatar-uploader" drag accept="text/plain" action="/data/"
        v-model:file-list="fileList" :show-file-list="true" :on-exceed="exceedHandler" :auto-upload="false"
        :before-upload="beforeDataUpload" :http-request="fileUpload" :limit="1">
        <template #trigger>
          <el-icon class="avatar-uploader-icon">
            <Plus />
          </el-icon>
        </template>
        <template #tip>
          <div class="el-upload__tip">
            仅支持txt文件
          </div>
        </template>
      </el-upload>
      <div id="tip" class="bg-blend-color-dodge" style="margin: 20px 0px">
        <el-card style="width: 100%" shadow="hover">
          <el-text class="mx-1" size="large">txt文件注意事项：<br /><br /></el-text>
          1. 首行必须为各个属性的属性名，可以复制粘贴下面的内容<br />
          <el-card style="margin: 5px; background: rgba(207,212,213,0.63)" shadow="always">
            elasticityModulus structuralAdhesiveStress panelDamageArea structuralAdhesiveDamageLength connectorsNumber
            backBoltsNumber panelVerticality stitchingWidth panelSize Offset_x Offset_y Offset_z flatness stains cracks
          </el-card>
          2. 第二行开始每一行为一块幕墙的数据<br />
          3. 每一行包括若干数字，对应上述的属性，数字之间用空格隔开<br />
          4. 数字可以是正数、小数和负数
        </el-card>
      </div>
      <div class="flex">
        <div class="grow"></div>
        <el-button type="primary" round @click="submitConfirm">
          上传文件
        </el-button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 48px;
  color: #8c939d;
  width: 378px;
  height: 378px;
  text-align: center;
}
</style>