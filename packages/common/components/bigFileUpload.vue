<!--
 * @Description:大文件上传
 * @Author: 王天琦
 * @Date: 2023-10-25 11:48:58
-->
<template>
  <div>
    <input type="file" @change="handleFileChange" />
    <el-button @click="handleUpload">上传</el-button>
  </div>
</template>

<script>
 // 切片大小 the chunk size
 const SIZE = 10 * 1024 * 1024;

export default {
  data: () => ({
   container: {
    file: null
   },
  data: []
  }),
  methods: {
   request() {},
   handleFileChange() {},
   // 生成文件切片
   createFileChunk(file, size = SIZE) {
   const fileChunkList = [];
    let cur = 0;
    while (cur < file.size) {
     fileChunkList.push({ file: file.slice(cur, size) });
     cur = size;
    }
    return fileChunkList;
   },
  // 上传切片
   async uploadChunks() {
    const requestList = this.data
     .map(({ chunk,hash }) => {
      const formData = new FormData();
      formData.append("chunk", chunk);
      formData.append("hash", hash);
      formData.append("filename", this.container.file.name);
      return { formData };
     })
     .map(({ formData }) =>
      this.request({
       url: "http://localhost:3000",
       data: formData
      })
     );
      // 并发请求
    await Promise.all(requestList);
   },
   async handleUpload() {
    if (!this.container.file) return;
    const fileChunkList = this.createFileChunk(this.container.file);
    this.data = fileChunkList.map(({ file },index) => ({
     chunk: file,
     hash: this.container.file.name + "-" + index // 文件名  数组下标
    }));
    await this.uploadChunks();
   }
  }
};
</script>