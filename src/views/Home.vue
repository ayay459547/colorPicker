<template>
  <div class="color-picker">
    <div class="container">
      <h1>Color Picker</h1>
      <div class="show" style="background: linear-gradient(87deg, rgba(219,66,66,1) 45%, rgba(195,58,58,0) 100%);"></div>
      <div class="controls"
        @mousemove="dragMove"
        @mouseup="dragUp">

        <div ref="drag-list" 
          @mousedown="add"
          class="drag-list" 
          style="background: linear-gradient(87deg, rgba(219,66,66,1) 45%, rgba(195,58,58,0) 100%);">

          <div
            class="drag-item-container" 
            v-for="(item, index) in colorList" 
            :key="item.id"
            :ref="item.position"
            :class="{activeDrag: active == index}"
            @mousedown.stop="dragDown(index)"
            :style="`transform: translateX(${dragWidth * item.position * 0.01}px);`"
          >

            <div class="drag-position"><input type="text" v-model="item.position"></div>
            <div class="drag-item" :style="`background: ${item.hex}`"></div>
          </div>
        </div>

        <div class="panel">
          <div class="color-panel">
            <div class="palette">
              <div class="select-circle"></div>
            </div>
          </div>

          <div class="color-panel">
            <h3>color code</h3>
            <label for="hex">hex : </label>
            <input type="text" id="hex" v-model="colorList[active].hex">
            <br/>
            <label for="rgba">rgba : </label>
            <input type="text" id="rgba" v-model="colorList[active].rgba">
            
            <div class="pointer-1">
              <div class="control-1"></div>
            </div>
            <div class="pointer-2">
              <div class="control-2"></div>
              <!-- style currentColor -->
            <div class="bg"></div>
            </div>
          </div>

          <div class="color-panel">
            <ul class="console">
              <li v-for="(item, index) in colorList" 
                :class="{active: index === active}"
                @click="active = index"
                :key="item.id">
                <div class="color-box" :style="`background: ${item.hex}`"></div>
                <input type="text" v-model="colorList[index].hex">
                <input type="text" v-model="colorList[index].position">
                <img src="../assets/delete.png" alt="" width="30px"
                  @click.stop="del(index)">
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      bgcProperty: 0, // 0: liner-gradient, 1: radial-gradient
      colorList: [
        {
          id: 0,
          hex: '#c33a3a',
          rgba: 'rgba(219,66,66,1)',
          position: 0
        },
        {
          id: 1,
          hex: '#c33a3a',
          rgba: 'rgba(195,58,58,0)',
          position: 100
        }
      ],
      active: 0,
      globalId: 2, //

      dragWidth: null,
      dragCanMove: false
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
          hex: '#c33a3a',
          rgba: 'rgba(219,66,66,1)',
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

    dragDown(index) {
      this.dragCanMove = true
      this.changeActive(index)
    },
    dragUp() {
      this.dragCanMove = false
      this.listSort()
    },
    dragMove(e) {
      if(this.dragCanMove) {
        let pos = e.clientX - this.$refs['drag-list'].offsetLeft
        pos = Math.floor(pos * 100 / this.dragWidth)
        pos = pos <= 0 ? 0 : pos
        pos = pos >= 100 ? 100 : pos
        this.colorList[this.active].position = pos
      }
    },


    listSort() {
      this.colorList.sort((a, b) => {
        return a.position - b.position
      })
    },
    changeActive(index) {
      this.active = index
    }
  },
  mounted() {
    this.dragWidth = this.$refs['drag-list'].clientWidth
  }
}
</script>

<style lang="scss" scoped>
.color-picker { 
  width: 100%;
  height: 100vh;
  *{
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
      border: 2px solid black;
      border-radius: 20px;
    }

    .controls{
      width: 95%;
      height: 50%;
      margin: -100px auto;
      border: 2px solid black;
      background-color: #fff;
      border-radius: 20px;

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
            transition: all 0.3s;
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

          .palette {
            width: 90%;
            height: 200px;
            border-radius: 10px;
            border: 1px solid black;

            .select-circle {
              width: 30px;
              height: 30px;
              border: 2px solid black;
              border-radius: 50%;
            }
          }

          .pointer-1 {
            margin-top: 20px;
            width: 100%;
            height: 30px;
            border-radius: 5px;
            background-image: linear-gradient(to right, red 0%, #ff0 17%, lime 33%, cyan 50%, blue 66%, #f0f 83%, red 100%);;
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
          }

          .console {
            width: 100%;
            list-style: none;

            li {
              width: 100%;
              // border: 2px solid black;
              padding: 5px;
              display: flex;
              justify-content: space-around;
              transition: all 0.3s;
              border-radius: 5px;

              input:nth-child(2) {
                text-align: center;
                width: 80px;
              }
              input:nth-child(3) {
                text-align: center;
                width: 40px;
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

    }
  }
}
</style>
