<template>
  <div 
    class="color-picker"
    @mouseup="
      dragUp($event);
      pointer1up();
      pointer2up();
      circleUp();
      degUp();" 
    @mousemove="
      dragMove($event);
      pointer1move($event);
      pointer2move($event);
      circleMove($event)
      degTurn($event)"
  >
    <div class="container">
      <h1>Color Picker</h1>
      <div class="show">
        <div v-show="!bgcProperty" class="bg" :style="`background-image: linear-gradient(${deg}deg, ${showColor})`"></div>
        <div v-show="bgcProperty" class="bg" :style="`background-image: radial-gradient(circle, ${showColor})`"></div>
      </div>
      <div class="controls">
        <!-- %拖拉區 -->
        <div 
          ref="drag-list" 
          @mousedown="add"
          class="drag-list" 
          :style="`background: linear-gradient(90deg, ${dragColor});`"
        >
          <div
            class="drag-item-container" 
            v-for="(item, index) in colorList" 
            :key="item.id"
            :ref="item.position"
            :class="{activeDrag: active == index}"
            @mousedown.stop="dragDown($event, index)"
            :style="`transform: translateX(${dragWidth * item.position * 0.01}px);`"
          >
            <div class="drag-position">
              <input type="text" v-model="item.position">
            </div>
            <div class="drag-item" :style="`background: ${item.hex}`"></div>
          </div>
        </div>
        <!-- 左邊 -->
        <div class="panel">
          <div class="color-panel">
            <div 
              class="palette" 
              ref="palette"
              :style="`background: linear-gradient(225deg, ${rgbList[active]} 0%, rgba(255,255,255,0) 100%);`"
              >
              <div class="bgleft"></div>
              <div 
                @mousedown="circleDown"
                class="select-circle" 
                ref="circle"
              ></div>
            </div>
          </div>
          <!-- 中間 -->
          <div class="color-panel">
            <h3>color code</h3>
            <input @keyup="hexToRgba" type="text" id="hex" v-model="colorList[active].hex">
            <input @keyup="rgbaToHex" type="text" id="r" v-model.number="colorList[active].rgba.red">
            <input @keyup="rgbaToHex" type="text" id="g" v-model.number="colorList[active].rgba.green">
            <input @keyup="rgbaToHex" type="text" id="b" v-model.number="colorList[active].rgba.blue">
            <input @keyup="rgbaToHex" type="text" id="a" v-model.number="colorList[active].rgba.alpha">
            <br/>
            <span>hex</span>
            <span>r</span>
            <span>g</span>
            <span>b</span>
            <span>a</span>
            <!-- <input type="text" id="rgba" v-model="rgbaList[active]"> -->
            <!-- 中間拖拉區 -->
            <div class="pointer-1" ref="pointer1">
              <div 
                @mousedown.stop="pointer1Down"
                class="control-1"
                ref="control1" 
                :style="`background: ${rgbList[active]}`"
              ></div>
            </div>
            <div class="pointer-2" ref="pointer2">
              <div 
                @mousedown.stop="pointer2Down"
                class="control-2" 
                ref="control2"
                :style="`background-color: ${rgbList[active]}`"
              ></div>
              <!-- style currentColor -->
              <div class="bg" :style="`background-image: linear-gradient(90deg, ${rgbList[active]} 20%, rgba(255,255,255,0) 100%);`"></div>
            </div>
          </div>
          <!-- 右邊 -->
          <div class="color-panel">
            <ul class="console">
              <li 
                v-for="(item, index) in colorList" 
                :class="{active: index === active}"
                @click="active = index"
                :key="item.id"
              >
                <div class="color-box" :style="`background: ${item.hex}`"></div>
                <input type="text" @keyup="hexToRgba" v-model="colorList[index].hex">
                <input type="text" v-model="colorList[index].position">
                <img src="../assets/delete.png" alt="" width="30px"
                  @click.stop="del(index)">
              </li>
            </ul>
          </div>
        </div>
        <div class="other">
          <div class="change-property">
            <button :class="{active: bgcProperty == 0}" @click="bgcProperty = 0">Linear</button>
            <button :class="{active: bgcProperty == 1}" @click="bgcProperty = 1">Radial</button>
          </div>
          <div class="set-deg">
            <div 
              class="deg-circle" 
              ref="deg-circle"
              @mousedown="degDown"
              :style="`transform: rotateZ(${deg}deg);`">
              <div class="dot"></div>
            </div>
            <input type="text" v-model="deg">
          </div>
          <div class="code">
            <h3 v-show="!bgcProperty">background: linear-gradient({{deg}}deg, {{showColor}})</h3>
            <h3 v-show="bgcProperty">background: radial-gradient(circle, {{showColor}})</h3>
            <button
              v-show="!bgcProperty" 
              @click="copy('copy1')"
              class="copy" 
              ref="copy1"
              :data-clipboard-text="`background: linear-gradient(${deg}deg, ${showColor})`"
            ></button>
            <button
              v-show="bgcProperty" 
              @click="copy('copy2')"
              class="copy" 
              ref="copy2"
              :data-clipboard-text="`background: radial-gradient(circle, ${showColor})`"
            ></button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Clipboard from 'clipboard';

export default {
  data() {
    return {
      bgcProperty: 0, // 0: liner-gradient, 1: radial-gradient
      colorList: [
        {
          id: 0,
          hex: '#ff0000',
          rgba: {
            red: 255,
            green: 0,
            blue: 0,
            alpha: 100
          },
          position: 0
        },
        {
          id: 1,
          hex: '#00ffff',
          rgba: {
            red: 0,
            green: 255,
            blue: 255,
            alpha: 100
          },
          position: 50
        },
        {
          id: 2,
          hex: '#c800ff',
          rgba: {
            red: 200,
            green: 0,
            blue: 255,
            alpha: 50
          },
          position: 100
        }
      ],
      active: 0,
      globalId: 3, //

      //拖拉區
      dragWidth: null,
      dragCanMove: false,
      //算速度
      startTime: null,
      endTime: null,
      startX: null,
      endX: null,
      //圓圈區
      paletteWidth: null,
      paletteHeight: null,
      circleCanMove: false,
      //rbga左右拖拉區
      pointer1Width: null,
      pointer1CanMove: false,
      pointer2Width: null,
      pointer2CanMove: false,
      //bg-img角度
      deg: 90,
      degCircleWidth: null,
      circleCanTurn: false,
      //pointer六塊中哪一塊
      areaStateList: [],

      //copy
      clipboard: null
    }
  },
  computed: {
    showColor() {
      let res = ''
      this.colorList.forEach((item, index) => {
        res += `rgba(${item.rgba.red},${item.rgba.green},${item.rgba.blue},${item.rgba.alpha/100}) ${item.position}%`
        if(index != this.colorList.length - 1) {
          res += ','
        }
      })
      return res
    },
    dragColor() {
      let res = ''
      this.colorList.forEach((item, index) => {
        res += `rgba(${item.rgba.red},${item.rgba.green},${item.rgba.blue}) ${item.position}%`
        if(index != this.colorList.length - 1) {
          res += ','
        }
      })
      return res
    },
    rgbaList() {
      let res = []
      this.colorList.forEach(item => {
        res.push(`rgba(${item.rgba.red},${item.rgba.green},${item.rgba.blue},${item.rgba.alpha/100})`)
      })
      return res
    },
    rgbList() {
      let res = []
      this.colorList.forEach(item => {
        res.push(`rgba(${item.rgba.red},${item.rgba.green},${item.rgba.blue})`)
      })
      return res
    }
  },
  methods: {
    log() {
      console.log(123)
    },
    add(e) {
      //滑鼠位置 - 拖拉元件的左邊
      let pos = e.clientX - this.$refs['drag-list'].offsetLeft
      pos = Math.floor(pos * 100 / this.dragWidth)

      //如果已存在不會加入
      let judgment = true      
      this.colorList.forEach(item => {
        if(item.position === pos) {
          judgment = false
        }
      })

      if(judgment) {
        const obj = {
          id: this.globalId++,
          hex: this.colorList[this.active].hex,
          rgba: this.colorList[this.active].rgba,
          position: pos
        }
        this.colorList.push(obj)
      }
      //排序
      this.listSort()
      //換active
      let newActive = this.colorList.findIndex(item => {
        return item.position === pos
      })
      this.changeActive(newActive)
      
    },

    del(index) {
      if(this.colorList.length > 2) {
        this.colorList.splice(index,1)
        this.changeActive(0)
      }else {
        console.log('至少兩個顏色')
      }
    },
    //上半部拖拉控制
    dragDown(e, index) {
      this.dragCanMove = true
      this.changeActive(index)

      //算速度
      this.startTime = Date.now()
      let pos = this.nowPosition(e)
      this.startX = pos
    },
    dragUp(e) {
      this.dragCanMove = false
      this.listSort()

      //算速度
      this.endTime = Date.now()
      let pos = this.nowPosition(e)
      this.endX = pos

      let x = this.endX - this.startX
      let t = this.endTime - this.startTime
      // console.log(x)
      // console.log(t)
      console.log("速度 => ",Math.floor(x*400/t))
      //滑動功能保留
      // setTime
      // this.colorList[this.active].position += Math.floor(x*400/t)
    },
    dragMove(e) {
      if(this.dragCanMove) {
        let pos = this.nowPosition(e)
        this.colorList[this.active].position = pos
      }
    },
    nowPosition(e) {
      let pos = e.clientX - this.$refs['drag-list'].offsetLeft
      pos = Math.floor(pos * 100 / this.dragWidth)
      pos = pos <= 0 ? 0 : pos
      pos = pos >= 100 ? 100 : pos
      return pos
    },
    //圓圈拖拉控制
    circleDown() {
      this.circleCanMove = true
    },
    circleUp() {
      this.circleCanMove = false
    },
    circleMove(e) {
      if(this.circleCanMove) {
        let newLeft = e.clientX - this.$refs['palette'].offsetLeft
        let newTop = e.clientY - this.$refs['palette'].offsetTop
        //不超過範圍
        newLeft = newLeft < 0 ? 0 : newLeft
        newTop = newTop < 0 ? 0 : newTop
        newLeft = newLeft > this.paletteWidth ? this.paletteWidth : newLeft
        newTop = newTop > this.paletteHeight ? this.paletteHeight : newTop
        //移動
        this.$refs['circle'].style.left = `${newLeft}px`
        this.$refs['circle'].style.top = `${newTop}px`
        //rgb附值
        let newRgbX = Math.floor(Math.floor(newLeft * 100 / this.paletteWidth) * 255 / 100)
        let newRgbY = Math.floor(Math.floor(newTop * 100 / this.paletteHeight) * 255 / 100)
        console.log(newRgbX,newRgbY)
        // this.colorList[this.active].rgba.red = 255 - newRgbY
        //小於0就等於0
        // this.colorList[this.active].rgba.green = this.colorList[this.active].rgba.green >= 0 ? 255 - newRgbY - newRgbX : 0
        // this.colorList[this.active].rgba.blue = this.colorList[this.active].rgba.blue >= 0 ? 255 - newRgbY - newRgbX : 0
      }
    },
    //中間區拖拉控制
    pointer1Down() {
      this.pointer1CanMove = true
    },
    pointer2Down() {
      this.pointer2CanMove = true
    },
    pointer1up() {
      this.pointer1CanMove = false
    },
    pointer2up() {
      this.pointer2CanMove = false
    },
    pointer1move(e) {
      if(this.pointer1CanMove) {
        let newpos = e.clientX - this.$refs['pointer1'].offsetLeft
        //不超過範圍
        newpos = newpos < 0 ? 0 : newpos
        newpos = newpos > this.pointer1Width ? this.pointer1Width : newpos
        //移動
        this.$refs['control1'].style.left = `${newpos}px`
        //rgb附值 分六塊
        let area = Math.floor(this.pointer1Width / 6)
        if(newpos <= area) {
          this.colorList[this.active].rgba.red = 255
          this.colorList[this.active].rgba.green = Math.floor(newpos/area * 255)
          this.colorList[this.active].rgba.blue = 0
        }else if(newpos <= area * 2) {
          this.colorList[this.active].rgba.red = 255 - Math.floor((newpos - area)/area * 255)
          this.colorList[this.active].rgba.green = 255
          this.colorList[this.active].rgba.blue = 0
        }else if(newpos <= area * 3) {
          this.colorList[this.active].rgba.red = 0
          this.colorList[this.active].rgba.green = 255
          this.colorList[this.active].rgba.blue = Math.floor((newpos - area * 2)/area * 255)
        }else if(newpos <= area * 4) {
          this.colorList[this.active].rgba.red = 0
          this.colorList[this.active].rgba.green = 255 - Math.floor((newpos - area * 3)/area * 255)
          this.colorList[this.active].rgba.blue = 255
        }else if(newpos <= area * 5) {
          this.colorList[this.active].rgba.red = Math.floor((newpos - area * 4)/area * 255)
          this.colorList[this.active].rgba.green = 0
          this.colorList[this.active].rgba.blue = 255
        }else if(newpos <= area * 6) {
          this.colorList[this.active].rgba.red = 255
          this.colorList[this.active].rgba.green = 0
          this.colorList[this.active].rgba.blue = 255 - Math.floor((newpos - area * 5)/area * 255)
        }
        this.rgbaToHex()
      }
    },
    pointer2move(e) {
      if(this.pointer2CanMove) {
        let newpos = e.clientX - this.$refs['pointer2'].offsetLeft
        //不超過範圍
        newpos = newpos < 0 ? 0 : newpos
        newpos = newpos > this.pointer2Width ? this.pointer2Width : newpos
        //移動
        this.$refs['control2'].style.left = `${newpos}px`
        //透明度
        this.colorList[this.active].rgba.alpha = 100 - Math.floor(newpos * 100/this.pointer2Width)
      }
    },

    //角度控制
    degUp() {
      this.circleCanTurn = false
    },
    degDown() {
      this.circleCanTurn = true
    },
    degTurn(e) {
      if(this.circleCanTurn) {
        //取圓心
        let centerX = this.$refs['deg-circle'].offsetLeft + this.degCircleWidth/2
        let centerY = this.$refs['deg-circle'].offsetTop + this.degCircleWidth/2
        let x = e.clientX - centerX
        //因為滑鼠往上 y 會遞減 所以相反
        let y = centerY - e.clientY
        let angle = Math.atan2(x, y) / Math.PI * 180
        angle = angle < 0 ? angle + 360 : angle
        this.deg = Math.floor(angle)
      }
    },


    listSort() {
      this.colorList.sort((a, b) => {
        return a.position - b.position
      })
    },
    changeActive(index) {
      //控制器換位置
      this.active = index
    },

    hexToRgba(e, hex = this.colorList[this.active].hex, opacity = this.colorList[this.active].rgba.alpha) {
      // let RGBA = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt( "0x" + hex.slice(5, 7)) + "," + opacity + ")";
      this.colorList[this.active].rgba = {
        red: parseInt("0x" + hex.slice(1, 3)),
        green: parseInt("0x" + hex.slice(3, 5)),
        blue: parseInt("0x" + hex.slice(5, 7)),
        alpha: opacity,
      }
    },
    rgbaToHex(e, r = this.colorList[this.active].rgba.red, g = this.colorList[this.active].rgba.green, b = this.colorList[this.active].rgba.blue) {
      this.colorList[this.active].hex = `#${this.toHex(r)}${this.toHex(g)}${this.toHex(b)}`
    },
    toHex(color) {
      let hex = parseInt(color).toString(16)
      return hex.length == 1 ? `0${hex}` : hex
    },

    copy(btn) {
      console.log(btn)
      // this.desrtoy()
      this.clipboard = new Clipboard(this.$refs[btn]);
      this.clipboard.on('success', () => {
        // console.log('copy success')
        // this.clipboard = new Clipboard(this.$refs['copy'])
      })
      this.clipboard.on('error', () => {
        console.log('copy error')
      })
    }
  },
  mounted() {
    this.dragWidth = this.$refs['drag-list'].clientWidth
    //方形調色盤
    this.paletteWidth = this.$refs['palette'].clientWidth
    this.paletteHeight = this.$refs['palette'].clientHeight
    //拖拉調色
    this.pointer1Width = this.$refs['pointer1'].clientWidth
    this.pointer2Width = this.$refs['pointer2'].clientWidth
    //角度圓圈
    this.degCircleWidth = this.$refs['deg-circle'].clientWidth
  }
}
</script>

<style lang="scss" scoped>
::-webkit-scrollbar {
  width: 10px;
}
::-webkit-scrollbar-track {
  border-radius: 5px;
  box-shadow: inset 0 0 10px rgba(0,0,0,0.25);
}
::-webkit-scrollbar-thumb {
  border-radius: 5px;
  background-color: #1F2667;

  &:hover {
    background-color: #384088;
  }
}

.color-picker { 
  width: 100%;
  height: 100vh;
  *{
    user-select: none;
    box-sizing: border-box;
  }
  
  .container {
    width: 95%;
    height: 100%;
    margin: 0 auto;
    text-align: center;
    box-sizing: border-box;
    h1 {
      padding: 20px;
    }

    .show{
      width: 100%;
      height: 40%;
      border-radius: 20px;
      position: relative;
      z-index: -1;
      background-image: 
        linear-gradient(45deg, #ccc 25%, transparent 25%),
        linear-gradient(-45deg, #ccc 25%, transparent 25%),
        linear-gradient(45deg, transparent 75%, #ccc 75%),
        linear-gradient(-45deg, transparent 75%, #ccc 75%);
      background-size: 16px 16px;
      background-position: 0 0, 0 8px, 8px -8px, -8px 0px; 
      .bg {
        width: 100%;
        height: 100%;
        border-radius: 20px;
        border: 2px solid black;
        position: absolute; 
      }
    }

    .controls{
      width: 95%;
      height: 50%;
      min-width: 1230px;
      margin: -100px auto;
      border: 2px solid black;
      background-color: #fff;
      border-radius: 20px;
      z-index: 10;
      //上半部控制面版
      .drag-list {
        width: 95%;
        height: 40px;
        margin: 60px auto 0px;
        border-radius: 10px;
        border: 2px solid black;
        display: flex;
        position: relative;
        cursor: copy;

        .drag-item-container {
          width: 50px;
          height: 100px;
          margin-top: -40px;
          left: -25px;
          position:absolute;
          cursor: move;

          .drag-position {
            border-radius: 5px;
            transition: all 0.3s;

            input {
              width: 50px;
              font-size: 20px;
              border-radius: 5px;
              background-color: #fff;
              border: 2px solid black;
              text-align: center;
            }
          }

          .drag-item {
            width: 15px;
            height: 70px;
            margin: 0 auto;
            border-radius: 5px;
            border: 2px solid black;
          }

          &:hover {
            .drag-position,
            .drag-item {
              box-shadow: 3px 3px 2px 1px rgba(0, 0, 0, 0.2);
            }
          }
          &.activeDrag {
            .drag-position,
            .drag-item {
              box-shadow: 4px 4px 3px 2px rgba(0, 0, 0, 0.2);
            }
          }
        }
      }

      //下半部控制面版
      .panel {
        display: flex;
        width: 95%;
        margin: 0 auto;

        .color-panel {
          flex: 1;
          margin-top: 30px;
          height: 230px;
          // border: 2px solid black;
          font-size: 25px;
          padding: 10px;
          
          //左邊區
          .palette {
            width: 90%;
            height: 200px;
            border-radius: 5px;
            position: relative;
            // border: 1px solid black;
            
            .bgleft {
              width: 100%;
              height: 200px;
              background-image: linear-gradient(180deg, rgba(255,255,255,0) 0%, rgba(0,0,0,1) 100%);
            }

            .select-circle {
              width: 30px;
              height: 30px;
              position: absolute;
              left: 0;
              top: 0;
              transform: translate(-15px, -15px);
              background-image: radial-gradient(circle, rgba(255,255,255,0) 35%, rgba(255,255,255,1) 40%);
              border: 4px solid black;
              border-radius: 50%;
            }
          }

          //中間區
          input {
            width: 50px;
            height: 30px;
            padding: 0 10px;
            margin: 0 5px;
          }
          input:nth-child(2) {
            width: 80px;
          }
          span {
            display: inline-block;
            padding: 5px 25px;
            color: rgb(177, 177, 177);
          }

          .pointer-1 {
            margin-top: 20px;
            width: 100%;
            height: 30px;
            position: relative;
            border-radius: 5px;
            background-image: linear-gradient(to right, red 0%, #ff0 17%, lime 33%, cyan 50%, blue 66%, #f0f 83%, red 100%);;

            .control-1 {
              width: 10px;
              height: 40px;
              border-radius: 5px;
              border: 2px solid black;
              position: absolute;
              cursor: move;
              transform: translateX(-5px);
              top: -5px;
            }
          }

          .pointer-2 {
            margin-top: 20px;
            width: 100%;
            height: 30px;
            border-radius: 5px;
            position: relative;
            background-image: 
              linear-gradient(45deg, #ccc 25%, transparent 25%),
              linear-gradient(-45deg, #ccc 25%, transparent 25%),
              linear-gradient(45deg, transparent 75%, #ccc 75%),
              linear-gradient(-45deg, transparent 75%, #ccc 75%);
            background-size: 16px 16px;
            background-position: 0 0, 0 8px, 8px -8px, -8px 0px;
            
            .control-2 {
              width: 10px;
              height: 40px;
              border-radius: 5px;
              border: 2px solid black;
              position: absolute;
              cursor: move;
              transform: translateX(-5px);
              top: -5px;
              z-index: 10;
            }

            .bg {
              width: 100%;
              height: 30px;
              border-radius: 5px;
              position: absolute;
              }
            }
          
          //右邊區
          .console {
            width: 100%;
            height: 200px;
            list-style: none;
            overflow-y: scroll;
            overflow-y: overlay;
            
            li {
              width: 100%;
              // border: 2px solid black;
              padding: 5px;
              display: flex;
              justify-content: space-around;
              align-items: center;
              transition: all 0.3s;
              border-radius: 5px;

              input:nth-child(2) {
                text-align: center;
                width: 80px;
              }
              input:nth-child(3) {
                text-align: center;
                width: 50px;
              }

              &.active {
                background: #e0e0e0;
              }
            }
            // input:first-child {
            //   width: 50px;
            // }
            // input:last-child {
            //   width: 30px;
            // }
            .color-box {
              width: 30px;
              height: 30px;
              border-radius: 5px;
              display: inline-block;
            }
          }
        }
      }
      //底部區
      .other {
        width: 100%;
        height: 100px;
        margin-top: 5px;
        display: flex;

        .change-property {
          flex: 1;
          display: flex;
          align-items: center;
          justify-content: center;

          button {
            padding: 8px;
            margin: 10px;
            font-size: 25px;
            border-radius: 10px;
            background: black;
            color: #fff;
            transition: all 0.3s;
            &.active {
              box-shadow: 5px 5px 5px 2px gray;
            }
          }
        }
        .set-deg {
          flex: 1;
          display: flex;
          align-items: center;
          justify-content: space-evenly;

          .deg-circle {
            width: 60px;
            height: 60px;
            border: 3px solid black;
            border-radius: 50%;

            .dot {
              width: 10px;
              height: 10px;
              border-radius: 50%;
              margin: 5px auto;
              background: rgb(0, 0, 0);
            }
          }
          input {
            width: 80px;
            height: 30px;
            font-size: 20px;
            text-align: center;
          }
        }
        .code {
          padding: 40px 10px;
          margin-right: 10px;
          position: relative;
          font-family: monospace, serif;
          font-size: 15px;
          flex: 4;
          text-align: left;
          border-radius: 5px;
          background: #1f2667;
          color: #cbcbd4;

          .copy {
            width: 30px;
            height: 30px;
            right: 5px;
            top: 5px;
            position: absolute;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            background-color: #1f2667;
            background-image: url('../assets/copy.svg');
            background-size: cover;
          }
        }
      }
    }
  }
}
</style>
