<template>
  <div class="panel">
    <div v-for="(item, idx) in markAry" :key="idx" class="item">
      <div v-for="(subItem, subIdx) in item" :key="subIdx" class="subItem">
        <div class="content" :class="{ 'cover': subItem === COVERD || subItem === SETTLED }"></div>
      </div>
    </div>
    <div>得分：{{ score }}</div>
  </div>
</template>

<script>
const WIDTH = 12;
const HEIGHT = 14;
const NO_COVERD = -1;
const COVERD = 1;
const SETTLED = 2;
const KEY = {
  UP: 38,
  DOWN: 40,
  LEFT: 37,
  RIGHT: 39
}
// 90°旋转矩阵
const ROTATE_MATRIX = [
  [0, 1],
  [-1, 0]
]
export default {
  name: 'HelloWorld',
  data() {
    return {
      markAry: [],
      brickAry: [],
      NO_COVERD,
      COVERD,
      SETTLED,
      fallInterval: null,
      speed: 500,
      score: 0
    }
  },
  created() {
    document.addEventListener('keydown', this.onKeydown, false);
    this.initArray();
    this.generateBrick();
    this.fall();
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.onKeydown, false);
  },
  methods: {
    reset() {
      clearInterval(this.fallInterval);
      this.fallInterval = null;
      this.speed = 500;
      this.brickAry = [];
    },
    reStart() {
      this.initArray();
      this.generateBrick();
      this.fall();
    },
    initArray() {
      for (let i = 0; i < HEIGHT; i++) {
        this.markAry[i] = [];
        for (let j = 0; j < WIDTH; j++) {
          this.markAry[i].push(NO_COVERD);
        }
      }
    },
    // 生成方框函数
    generateBrick() {
      let timeStamp = Number(new Date());
      let initLoc = timeStamp % 10;
      this.brickAry = [];
      this.brickAry.push({ x: 3, y: initLoc });
      let j = 3;
      while(j > 0) {
        let orientation  = timeStamp % this.brickAry.length;
        timeStamp = Math.floor(timeStamp / this.brickAry.length);
        let pos = this.brickAry[orientation];
        let posOrientationArr = [{x: pos.x + 1, y: pos.y}, {x: pos.x - 1, y: pos.y}, {x: pos.x, y: pos.y + 1}, {x: pos.x, y: pos.y - 1}]
        let isInvalid;
        let lastArr = [];
        for (let j = 0; j < posOrientationArr.length; j++) {
          let itemPos = posOrientationArr[j];
          for (let i = 0; i < this.brickAry.length; i++) {
            let item = this.brickAry[i];
            isInvalid = false;
            if (item.x === itemPos.x && item.y === itemPos.y || itemPos.x < 0 || itemPos.x >= HEIGHT || itemPos.y < 0 || itemPos.y >= WIDTH) {
              isInvalid = true;
              break;
            }
          } 
          if (!isInvalid) lastArr.push(itemPos);
        }
        let lastArrRandomIdx  = timeStamp % lastArr.length;
        timeStamp = Math.floor(timeStamp / lastArr.length);
        this.brickAry.push(lastArr[lastArrRandomIdx]);
        j--;
      }
      this.fillSquare(COVERD);
    },
    // 定时下坠函数
    fall() {
      this.fallInterval = setInterval(() => {
        for (let i = 0; i < this.brickAry.length; i++) {
          let item = this.brickAry[i];
          if (item.x + 1 >= HEIGHT || this.markAry[item.x + 1][item.y] === SETTLED) {
            clearInterval(this.fallInterval);
            this.fallInterval = null;
            break;
          }
        }
        if (this.fallInterval) {
          this.fillSquare(NO_COVERD);
          for (let i = 0; i < this.brickAry.length; i++) {
            let item = this.brickAry[i];
            item.x++;
            this.markAry[item.x][item.y] = COVERD;
          }
        } else {
          this.fillSquare(SETTLED);
          this.judgeIsSuccess();
          this.generateBrick();
          this.fall();
        }
      }, this.speed);
    },
    // 判断是否成功，并计分
    judgeIsSuccess() {
      let isFull;
      for (let i = this.markAry.length - 1; i >= 0; i--) {
        let item = this.markAry[i];
        isFull = true;
        for (let j = 0; j < item.length; j++) {
          if (item[j] !== SETTLED) { 
            isFull = false;
            break;
          }
        }
        if (isFull) {
          this.score += WIDTH;
          for (let k = i; k >= 0; k--) {
            for (let l = 0; l < this.markAry[i].length; l++) {
              if (k === 0) {
                this.markAry[k][l] === NO_COVERD;
              } else {
                this.markAry[k][l] = JSON.parse(JSON.stringify((this.markAry[k - 1][l])));
              }
            }
          }
          i++;
        }
      }
      for (let i = 0; i <= 3; i++) {
        for (let j = 0; j < this.markAry[i].length; j++) {
          if (this.markAry[i][j] === SETTLED) {
            this.reset();
            alert('游戏结束');
            return;
          }
        }
      }
    },
    // 监听键盘事件
    onKeydown(e) {
      switch(e.keyCode) {
        case KEY.DOWN: 
          this.down();
          break;
        case KEY.LEFT: 
          this.move(e.keyCode);
          break;
        case KEY.RIGHT: 
          this.move(e.keyCode);
          break;
        case KEY.UP: 
          this.rotate();
      }
    },
    // 下键动作函数
    down() {
      let isDown = false;
      while(!isDown) {
        for (let i = 0; i < this.brickAry.length; i++) {
          let item = this.brickAry[i];
          if (item.x + 1 === HEIGHT || this.markAry[item.x + 1][item.y] === SETTLED) {
            isDown = true;
            break;
          }
        }
        if (!isDown) {
          this.fillSquare(NO_COVERD);
          for (let i = 0; i < this.brickAry.length; i++) {
            let item = this.brickAry[i];
            item.x++;
            this.markAry[item.x][item.y] = COVERD;
          }
        }
      }
      this.fillSquare(SETTLED);
      clearInterval(this.fallInterval);
      this.fallInterval = null;
      this.judgeIsSuccess();
      this.generateBrick();
      this.fall();
    },
    // 左右移动函数
    move(keyCode) {
      let delta = keyCode === KEY.LEFT ? -1 : 1;
      for (let i = 0; i < this.brickAry.length; i++) {
        let item = this.brickAry[i];
        if (item.y + delta >= WIDTH || item.y + delta < 0 || this.markAry[item.x][item.y + delta] === SETTLED) return;
      }
      this.fillSquare(NO_COVERD);
      for (let i = 0; i < this.brickAry.length; i++) {
        let item = this.brickAry[i];
        item.y += delta;
      }
      this.fillSquare(COVERD);
    },
    // 填充方块函数
    fillSquare(type = NO_COVERD) {
      for (let i = 0; i < this.brickAry.length; i++) {
        let item = this.brickAry[i];
        this.markAry[item.x][item.y] = type;
      }
    },
    // 旋转函数
    rotate() {
      let centerX = 0, centerY = 0;
      for (let i = 0; i < this.brickAry.length; i++) {
        let item = this.brickAry[i];
        centerX += item.x;
        centerY += item.y;
      }
      centerX = Math.round(centerX / this.brickAry.length);
      centerY = Math.round(centerY / this.brickAry.length);
      let rotateArr = [];
      for (let i = 0; i < this.brickAry.length; i++) {
        let item = this.brickAry[i];
        let tempX = ((item.x - centerX) * ROTATE_MATRIX[0][0] + (item.y - centerY) * ROTATE_MATRIX[0][1]) + centerX;
        let tempY = ((item.x - centerX) * ROTATE_MATRIX[1][0] + (item.y - centerY) * ROTATE_MATRIX[1][1]) + centerY;
        if (tempX < 0 || tempX >= HEIGHT || tempY < 0 || tempY >= WIDTH || this.markAry[tempX][tempY] === SETTLED) {
          return;
        }
        rotateArr.push({x: tempX, y: tempY});
      }
      this.fillSquare(NO_COVERD);
      this.brickAry = rotateArr;
      this.fillSquare(COVERD);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.panel {
  display: inline-block;
  border: 10px solid black;
  border-top: 0px;
}
.item {
  display: flex;
}
.wall{
  background-color: black;
}
.content {
  width: 30px;
  height: 30px;
  background-color: gray;
}
.cover {
  background-color: red;
}
</style>
