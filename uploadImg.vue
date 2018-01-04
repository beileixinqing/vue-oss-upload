<template>
    <div id="uploadFile">
        <input type="file" :id="name" @change="doUpload" v-if="multiple==false" accept="image/gif,image/jpeg,image/jpg,image/png,image/svg">
        <input type="file" :id="name" @change="doUpload" :multiple="multiple" v-if="multiple==true" accept="image/gif,image/jpeg,image/jpg,image/png,image/svg">
        <button type="button" class="ivu-btn ivu-btn-ghost"><Icon type="ios-cloud-upload-outline"></Icon><span>上传文件</span></button>
        <span class="size">建议尺寸：{{size}}，图片大小不要超过100kb，否则上传失败</span>
        <div class="uploadView" v-if="uploadImg">
            <div v-for="(item,index) in uploadImg" :key="index">
                <div>
                  <img :src="item" :style="{width:width+'px',height:height+'px'}">
                  <div class="handleBtn">
                    <Icon type="ios-eye-outline" v-if="defaultView" @click.native="handleView(item)"></Icon>
                    <Icon type="ios-eye-outline" v-if="currentView" @click.native="handleView(item.url)"></Icon>
                    <Icon type="ios-trash-outline" @click.native="handleRemove(index)"></Icon>
                  </div>
                </div>
            </div>
        </div>
        <Modal title="查看图片" v-model="visible">
          <img :src="viewUrl" style="width: 100%">
        </Modal>
    </div>
</template>
<script>
    import axios from 'axios'
    export default {
        props:{
            width:Number,
            height:Number,
            name:String,
            size:String,
            multiple:Boolean
        },
        data () {
            return {
                region: '',
                bucket: '',
                uploadImg:[],
                visible:false,
                viewUrl:'',
                defaultView:false,
                currentView:false
            }
        },
        mounted(){
          this.setData();
        },
        methods: {
            setData(){
              if(process.env.NODE_ENV=="development"){
                this.region='oss-cn-shenzhen';
                this.bucket='x-test';
              }else{
                this.region='oss-cn-zhangjiakou';
                this.bucket='zwtt';
              }
            },
            doUpload(){
                let userId = localStorage.getItem('userId');
                let token = localStorage.getItem('token');
                const _this = this;
                axios({
                    url: "/internal/oss/get_token",
                    method: 'GET',
                    headers: {'w-auth-token': token}
                }).then((res) => {
                    var client = new OSS.Wrapper({
                        accessKeyId: res.data.accessKeyId,
                        accessKeySecret: res.data.accessKeySecret,
                        stsToken: res.data.securityToken,
                        region: _this.region,
                        bucket: _this.bucket
                    });
                    const inputFiles = document.getElementById(_this.name);
                    if (inputFiles.files) {
                        const fileLen = inputFiles.files.length;
                        const currentImgLength=_this.uploadImg.length;
                        const restLength=10-currentImgLength;
                        if(currentImgLength>10){
                          this.$Message.error('图片最多上传十张');
                        }else{
                          if(fileLen<=restLength){
                            for (let i = 0; i < fileLen; i++) {
                              const file = inputFiles.files[i];
                              let date = new Date();
                              let path="merchant/"+userId+"/"+date.getFullYear()+(date.getMonth()+1)+date.getDate()+date.getHours()+date.getMinutes()+date.getSeconds()+date.getMilliseconds()+ '.' + file.name.split('.').pop();
                              let size=file.size;
                              if(Math.round(size/(1024*1024)*100)/100<=2){
                                client.multipartUpload(path, file).then((results) => {
                                  if(size>=100*1024){
                                    _this.uploadImg.push(results.res.requestUrls[0].split("?")[0]);
                                  }else{
                                    _this.uploadImg.push(results.url);
                                  }
                                }).catch((err) => {
                                  this.$Message.error('上传图片失败，请重试!');
                                });
                              }else{
                                this.$Message.error('上传图片不能超过2M，请重试!');
                              }
                            }
                          }else{
                            this.$Message.error('图片最多上传十张');
                          }
                        }
                    }
                });
            },
            handleView(url){
              this.visible = true;
              this.viewUrl=url;
            },
            handleRemove(index){
              this.uploadImg.splice(index,1);
            },
            getSrcList(val){
              this.uploadImg=val;
            }
        }
    }
</script>
<style lang="less">
    #uploadFile{
        width:300px;
    }
    #uploadFile  input {
        position: relative;
        z-index: 100;
        opacity: 0;
        filter: alpha(opacity=0);
        cursor: pointer;
        width: 100%;
    }
    #uploadFile  .size {
        position: absolute;
        left:40%;
    }
    #uploadFile  .ivu-btn{
      margin-top: -62px;
    }
    .uploadView{
        width:100%;
        height:auto;
        img{
            width:100%;
            height:auto;
        }
    }
  .handleBtn .ivu-icon{
      font-size:25px;
  }
</style>
