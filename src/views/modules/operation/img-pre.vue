<template>
    <div class="img-model" >
      <div class="img-btn">
        <i @click="transformImg" class="el-icon-refresh-right">旋转</i>
        <i @click="big" class="el-icon-zoom-in">放大</i>
        <i @click="small" class="el-icon-zoom-out">缩小</i>
        <i @click="close" class="el-icon-circle-close">关闭</i>
      </div>
      <img @mousewheel="rollImg(this)" class="show-img" ref="bigImage" @click="close" :src="src" alt="">
      <!--<div class="img-box">-->
        <!--<div class="img-btn">-->
          <!--<i @click="transformImg" class="el-icon-refresh-right">旋转</i>-->
          <!--<i @click="big" class="el-icon-zoom-in">放大</i>-->
          <!--<i @click="small" class="el-icon-zoom-out">缩小</i>-->
          <!--<i @click="close" class="el-icon-circle-close">关闭</i>-->
        <!--</div>-->
        <!--<div  @mousewheel="rollImg(this)" class="img-box-show">-->
          <!--<img class="show-img" ref="bigImage" @click="close" :src="src" alt="">-->
        <!--</div>-->
      <!--</div>-->

      <!--<img @mousewheel="rollImg(this)" ref="bigImage" @click="close" :src="src" alt="">-->
    </div>
</template>

<script>
    export default {
        name: "img-pre",
        data(){
          return{
            src:'',
            current:0,
            zoom:1
          }
        },
        methods:{
          init (src) {
            document.body.style.overflow='hidden';
            document.body.style.height='100%';
            this.src=src;
          },
          close(){
            document.body.style.overflow='auto';
            document.body.style.height='auto';
            this.$emit('refreshClose');
            this.$refs.bigImage.style.maxWidth="70%";
          },
          rollImg() {
            /* 获取当前页面的缩放比
                  若未设置zoom缩放比，则为默认100%，即1，原图大小
              */
            this.$refs.bigImage.style.maxWidth="100%";
            var delta = (event.wheelDelta && (event.wheelDelta > 0 ? 1 : -1)) ||  // chrome & ie
              (event.detail && (event.detail > 0 ? -1 : 1));
            var zoom = parseInt(this.$refs.bigImage.style.zoom) || 100;
            if (delta > 0) {
              // 向上滚
              // zoom += event.wheelDelta / 12;
              // this.$refs.bigImage.style.zoom = zoom + "%";
              this.zoom += 0.1;
              this.$refs.bigImage.style.transform = "rotate("+this.current+"deg)scale("+this.zoom+","+this.zoom+")";

            } else if (delta < 0) {
              // 向下滚
              // zoom += event.wheelDelta / 12;
              // this.$refs.bigImage.style.zoom = zoom + "%";
              this.zoom -= 0.1;
              this.$refs.bigImage.style.transform = "rotate("+this.current+"deg)scale("+this.zoom+","+this.zoom+")";
            }
            // if (zoom >= 100 && zoom <250) {
            //   this.$refs.bigImage.style.zoom = zoom + "%";
            // }
            return false;
          },
          transformImg(){
            this.current = (this.current+90)%360;
            this.$refs.bigImage.style.transform = "rotate("+this.current+"deg)scale("+this.zoom+","+this.zoom+")";
          },
          big(){
            this.$refs.bigImage.style.maxWidth="100%";
            // var zoom = parseInt(this.$refs.bigImage.style.zoom) || 100;
            // console.log(zoom)
            this.zoom += 0.1;
            this.$refs.bigImage.style.transform = "rotate("+this.current+"deg)scale("+this.zoom+","+this.zoom+")";
          },
          small(){
            // this.$refs.bigImage.style.maxWidth="100%";
            // var zoom = parseInt(this.$refs.bigImage.style.zoom) || 100;
            // console.log(zoom)
            this.zoom -= 0.1;
            this.$refs.bigImage.style.transform = "rotate("+this.current+"deg)scale("+this.zoom+","+this.zoom+")";
          }
        }
    }
</script>

<style scoped>
  .img-model{
    width: 100%;
    height: 100vh;
    background: rgba(0,0,0,.3);
    position: fixed;
    top:0;
    left:0;
    z-index: 9999;
    overflow: auto;
    text-align: center;
    padding:70px 0;
    align-items: center;
  }
  .show-img{
    max-width: 70%;
    width: auto;
    height: auto;
    z-index: 999999;
  }
  .img-btn{
    width: 100%;
    height: 60px;
    line-height: 60px;
    position: fixed;
    top:0;
    left:0;
    color:#fff;
    margin-bottom: 10px;
    z-index: 999;
    text-align: center;
    background: rgba(0,0,0,.4);
  }
  .img-btn>i{
    display: inline-block;
    font-size: 24px;
    margin-right: 20px;
    cursor: pointer;
  }
  .img-box{
    width: 60%;
    height: 80%;
    background: #fff;
    border-radius: 10px;
    position: relative;
    margin: auto;
    padding:10px;
    box-sizing: border-box;
  }
  .img-box-show{
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    padding:10px;
    position: absolute;
    top:60px;
    left:0;
    overflow: auto;
  }
</style>
