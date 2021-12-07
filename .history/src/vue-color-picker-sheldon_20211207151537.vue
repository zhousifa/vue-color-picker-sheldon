<template>
  <div :class="['ColorPicker',draggable?'draggable':'']" id="color-picker">
    <!-- 标题区域 -->
    <div class="title">取色器</div>
    <!-- 调色板区域 -->
    <div class="colors stable">
      <div @click="getColorValue(index)" v-show="index !== 447" :style="'background:' + item + hexOpcity" class="color" v-for="(item,index) in colors.hex" :key="index"></div>
    </div>
    <!-- 透明度区域 -->
    <div class="block stable">
      <!-- <span class="demonstration">自定义初始值</span> -->
      <el-slider v-model="opcity" :step="0.01" :max="1"></el-slider>
    </div>
    <!-- 选中颜色展示区域 -->
    <div v-show="!colorType" class="currentColor stable" :style="'background:'+colorValue.hex"></div>
    <div v-show="colorType" class="currentColor stable" :style="`background:rgba(${colorValue.rgb[0]},${colorValue.rgb[1]},${colorValue.rgb[2]},${rgbOpcity})`"></div>
    <!-- switch区域 -->
    <div class="colorType">
      <span>颜色格式：</span>
      <el-switch @change="changeColorType" v-model="colorType" active-text="rgb" inactive-text="hex"></el-switch>
    </div>
    
    <!-- hex颜色输入框区域 -->
    <el-input v-show="!colorType" class="colorValue stable" v-model="colorValue.hex" maxlength="9" placeholder="请选择或者输入颜色"></el-input>
    <!-- rgba颜色输入框区域 -->
    <el-row v-show="colorType" type="flex" :gutter="10"  class="rgbaValue stable">
      <el-col :span="6" v-for="(item,index) in colorValue.rgb" :key="index">
        <el-input maxlength="3" v-model="colorValue.rgb[index]" max="9"></el-input>
      </el-col>
      <el-col :span="6">
        <el-input v-model="rgbOpcity"></el-input>
      </el-col>
    </el-row>
    <!-- 按钮区域 -->
    <div>
      <el-button class="stable" @click="sendColor" type="primary">确定</el-button>
    </div>
  </div>
</template>

<script>
export default {
  name:'ColorPicker',
  props:{
    draggable:{
      type:Boolean,
      default:true
    }
  },
  data () {
    return {
      colors:{
        // #000000、#ffffff
        hex:['#000000'],
        // rgb(0,0,0)、rgb(255,255,255)
        // rgb:['rbg(0,0,0)']
        rgb:[[0,0,0]]
      },
      opcity: 1,
      colorIndex:0,
      colorValue:{
        hex:'#000000',
        rgb:[0,0,0]
      },
      colorType:false,
      rgbOpcity:1
    }
  },
  computed:{
    hexOpcity:function(){
      const value = parseInt(this.opcity * 255) > 255?255:parseInt(this.opcity * 255)
      return this.convert(value)
    }
  },
  created () {
    // 初始化调色板
    this.getColors()
  },
  mounted(){
    // 让取色器可拖动
    this.makerPickerDraggle()
  },
  methods:{
    // 让取色器可拖动
    makerPickerDraggle(){
      const picker = document.getElementById('color-picker')
      let stableArr = document.getElementsByClassName('stable')
      let x = 0
      let y = 0
      let l = 0
      let t = 0
      let isDown = false
      picker.onmousedown = (e)=>{
        // 阻止拖拽打开网页的默认事件
        e.preventDefault()
        // 获取x,y坐标
        x = e.clientX
        y = e.clientY
        //获取左部和顶部的偏移量
        l = picker.offsetLeft;
        t = picker.offsetTop;
        //开关打开
        isDown = true;
         //设置样式  
        picker.style.cursor = 'move'
      }
      // 设置不可拖动部分
      for(let i = 0;i < stableArr.length;i++){
        let stable = stableArr[i]
        stable.onmousedown = (e)=>{
          // 阻止冒泡事件
          e.stopPropagation()
          //开关关闭
          isDown = false;
          picker.style.cursor = 'default'
        }
      }
      //鼠标移动
      window.onmousemove = (e)=> {
        if (isDown == false) {
            return;
        }
        //获取x和y
        let nx = e.clientX;
        let ny = e.clientY;
        //计算移动后的左偏移量和顶部的偏移量
        let nl = nx - (x - l);
        let nt = ny - (y - t);

        picker.style.left = nl + 'px';
        picker.style.top = nt + 'px';
      }
      //鼠标抬起事件
      picker.onmouseup = function() {
        //开关关闭
        isDown = false;
        picker.style.cursor = 'default';
      }
    },
    // 发送颜色
    sendColor(){
      let color = ''
      // 发送hex
      if(!this.colorType){
        let hex = this.colorValue.hex
        let len = hex.length
        // 符合以下几种条件时，颜色值有效，对应的rgb值也要发生变化
        if(len === 4 || len === 5 || len === 7 || len === 9){
          let matchStr = hex.match(/^#[0-9a-f]*/g)
          // 正则匹配到有效的颜色
          if(matchStr && matchStr[0] && matchStr[0] === hex){
            color = hex
          }
        }
        else{
          return this.$message.warning('当前颜色值不符合格式，请修改后进行切换')
        }
      }
      // 发送rgb
      else{
         // 是否都是有效数字
        let flag = true
        let rgb = this.colorValue.rgb
        for(let i = 0;i < rgb.length;i++){
          if(!(parseInt(rgb) > -1 && parseInt(rgb) < 256)){
            flag = false
          }
        }
        if(!(this.rgbOpcity >= 0 && this.rgbOpcity <= 1)){
          flag = false
        }
        if(flag){
          color = `rgba(${rgb[0]},${rgb[1]},${rgb[2]},${this.rgbOpcity})`
        }
        else{
          this.$message('当前颜色值不符合格式，请修改后进行切换')
        }
      }
      this.$emit('receiveColor',color)
    },
    // 点击色块，获取当前颜色
    getColorValue(index){
      this.colorIndex = index
      this.rgbOpcity = this.opcity
      this.colorValue.hex = this.colors.hex[index] + (this.hexOpcity === 'ff'?'':this.hexOpcity)
      this.colorValue.rgb = this.colors.rgb[index]
    },
    // hex转换器
    convert(num){
      num = parseInt(num)
      return num > 15?num.toString(16):'0' + num.toString(16)
    },
    // 取色器是由三原色组成，按道理来说是需要三个轴，但是电脑平面只有两个轴，按道理来说是无法完成的
    // 但是我看了下谷歌浏览器的取色器变化曲线
    // ff0000->ff00ff->0000ff->00ffff->00ff00->ffff00->ff0000
    // 它是将每种原色的最大值和最小值当成一个极端情况，相当于一个硬币的正反两面——ff和00，
    // 投掷三枚硬币，如果连顺序也计入变量，有8中情况：000000->0000ff->00ff00->00ffff->ff0000->ff00ff->ffff00->ffffff
    // 考虑到颜色的渐变，让ff向波浪一样传播，重新排下顺序：000000->ff0000->ffff00->00ff00->00ffff->0000ff->ff00ff->ffffff
    // 变化顺序下标是0，1，0，2，1，0，2
    // 可以发现000000 和 ffffff 是三种原色的极致，特地被排除在外了。
    getColors(){
      // 三原色初始值
      const colorPoints = [0,0,0]
      // 三原色渐变顺序
      const changeOrder = [0,1,0,2,1,0,1]
      for(const order of changeOrder){
        const hexObj = {
          hex0:this.convert(colorPoints[0]),
          hex1:this.convert(colorPoints[1]),
          hex2:this.convert(colorPoints[2])
        }

        const rgbObj = {
          rgb0:colorPoints[0],
          rgb1:colorPoints[1],
          rgb2:colorPoints[2],
        }

        if(colorPoints[order] === 0){
          while(colorPoints[order] < 255){
            colorPoints[order] = colorPoints[order] + 4 > 255?255:colorPoints[order] + 4
            hexObj['hex' + order] = this.convert(colorPoints[order])
            const hexStr = '#' + hexObj['hex0'] + hexObj['hex1'] + hexObj['hex2']
            this.colors.hex.push(hexStr)

            rgbObj['rgb' + order] = colorPoints[order]
            let rbg = [rgbObj.rgb0,rgbObj.rgb1,rgbObj.rgb2]
            this.colors.rgb.push(rbg)
          }
        }
        else if(colorPoints[order] === 255){
          while(colorPoints[order] > 0){
            colorPoints[order] = colorPoints[order] - 4 < 0?0:colorPoints[order] - 4
            hexObj['hex' + order] = this.convert(colorPoints[order])
            const hexStr = '#' + hexObj['hex0'] + hexObj['hex1'] + hexObj['hex2']
            this.colors.hex.push(hexStr)

            rgbObj['rgb' + order] = colorPoints[order]
            let rbg = [rgbObj.rgb0,rgbObj.rgb1,rgbObj.rgb2]
            this.colors.rgb.push(rbg)
          }
        }
  
      }
      // 一共有449个颜色，是质数，需要去掉一个，否则不好排版，448 = 16 * 28
      // console.log('hex',this.colors.hex)
      // console.log('rgba',this.colors.rgb)
    },
    // 改变颜色格式
    changeColorType(){
      const index = this.colorIndex
      // hex->rgb
      if(this.colorType){
        let hex = this.colorValue.hex
        let len = hex.length
        // 符合以下几种条件时，颜色值有效，对应的rgb值也要发生变化
        if(len === 4 || len === 5 || len === 7 || len === 9){
          let matchStr = hex.match(/^#[0-9a-f]*/g)
          // 正则匹配到有效的颜色
          if(matchStr && matchStr[0] && matchStr[0] === hex){
            let value = hex.slice(1)
            len = value.length
            // 统一转换为8位
            let value8 = ''
            if(len === 3 || len === 4){
              let opcity = value[3]?value[3]+value[3]:'ff'
              value8 = value[0]+value[0]+value[1]+value[1]+value[2]+value[2]+opcity
            }
            else{
              let opcity = value[7]?value[6]+value[7]:'ff'
              value8 = value[0]+value[1]+value[2]+value[3]+value[4]+value[5]+opcity
            }
            // console.log(value8)
            this.rgbOpcity = (parseInt(value8.slice(6,8),16) / 255).toFixed(2)
            this.colorValue.rgb[0] = parseInt(value8.slice(0,2),16)
            this.colorValue.rgb[1] = parseInt(value8.slice(2,4),16)
            this.colorValue.rgb[2] = parseInt(value8.slice(4,6),16)
          }
        }
        else{
          this.$message('当前颜色值不符合格式，请修改后进行切换')
        }
      }
      // rgb->hex
      else{
        // 是否都是有效数字
        let flag = true
        let rgb = this.colorValue.rgb
        for(let i = 0;i < rgb.length;i++){
          if(!(parseInt(rgb) >= 0 && parseInt(rgb) <= 255)){
            flag = false
          }
        }
        if(!(this.rgbOpcity >= 0 && this.rgbOpcity <= 1)){
          flag = false
        }
        if(flag){
          console.log('rgb->hex',rgb)
          this.colorValue.hex = '#' + this.convert(rgb[0]) + this.convert(rgb[1]) + this.convert(rgb[2]) + this.convert(parseInt(this.rgbOpcity * 255))
        }
        else{
          this.$message('当前颜色值不符合格式，请修改后进行切换')
        }
      }
    }
  }
}
</script>
<style lang="less">
.draggable{
  z-index: 10;
  position: absolute;
}
.ColorPicker{
  width: 392px;
  border: 1px solid #ccc;
  padding: 15px;
  background: #ffffff52;
  // z-index: 10;
  // position: absolute;
  .title{
    text-align: center;
    font-size: 14px;
    font-weight: bold;
    margin-bottom: 10px;
  }
  .colors{
    display: flex;
    flex-wrap: wrap;
    width: 392px;
    height: 230px;
    .color{
      width: 10px;
      height: 10px;
      border:1px solid #000;
      margin: 1px;
      border-radius: 2px;
    }
  }
  .currentColor{
    height: 40px;
    border-radius: 4px;
    border: 1px solid #DCDFE6;
  }
  .colorType{
    font-size: 14px;
    height: 50px;
    line-height: 50px;
    padding-left: 5px;
  }
  .colorValue{
    input{
      text-align: center;
      letter-spacing: 1px;
    }
  }
  .rgbaValue{
    // margin-top: 10px;
    input{
      text-align: center;
      letter-spacing: 1px;
    }
  }
  .el-button{
    margin: 10px auto 0;
    display: block;
  }
}
</style>
