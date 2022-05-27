<template>
    <div class="hello">
        <div class="bar">
            <span>主页</span>
            <span>愿望选择</span>
            <span>游戏学习</span>
            <span>详细介绍</span>
        </div>
        <div class="left">
            <div
                class="item" v-for="(item,index) in imgArr"
                :key="index"
                @click="changeCurrent(index)"
            >
                <img :src="item.url" alt="" >
                <div class="btn" >{{item.name}}</div>
            </div>
        </div>
        <div class="mid" @mousemove="move($event)">
            <div
                class="content"
                v-for="(item, index) in imgArr" :key="index"
                v-show="current === index"
            >
                <canvas
                    class='origin' :ref="'canvas' + index"
                    :width="canvasWidth" :height="canvasHeight"
                    :style="{width: canvasWidth + 'px', height: canvasHeight + 'px'}"
                />
                <div class="border" :style="{
                    width: penWidth + 'px',
                    height: penWidth + 'px',
                    top: borderY + 'px',
                    left: borderX + 'px',
                    borderRadius: erasering ? '0' : '50%'}"></div>
                <canvas
                    :ref="'canvas_' + index "
                    :width="canvasWidth" :height="canvasHeight"
                    @mousedown="mousedown($event)"
                    @mousemove="mousemove($event)"
                    :style="{width: canvasWidth + 'px', height: canvasHeight + 'px'}"
                />
            </div>
        </div>
        <div class="right">
            <div class="pen-color">
                线条颜色：
                <input class="color-input" type="color" name="color" ref="color" v-model="color">
            </div>
            <div class="pen-width">
                线条粗细：
                <div class="xi" @click="xi">-</div>
                {{penWidth}}
                <div class="cu" @click="cu">＋</div>
            </div>
            <button class="get-score" @click="getScore">
                点我出成绩
            </button>
            <div class="score">
                我的得分：
                <span class="span">{{score}}</span>
            </div>
            <div class="btn-group">
                <button class="eraser" @click="eraser">
                    {{erasering ? '画笔' : '橡皮'}}
                </button>
                <button class="clear" @click="clear">
                    清除全部
                </button>
            </div>
            <div class="save" @click="save">
                <!-- <img src="../assets/save.png" alt=""> -->
                <span>保存图片</span>
            </div>
        </div>
    </div>
</template>

<script>
import img1_ from "../assets/fu1.png";
import img2_ from "../assets/fu2.png";
import img3_ from "../assets/fu3.png";
import img4_ from "../assets/fu4.png";
export default {
    name: "HelloWorld",
    data() {
        return {
            isDown: false,
            lock: true,
            x: 0,
            y: 0,
            score: 0,
            canvasWidth: 100,
            canvasHeight: 200,
            ctx: null,
            ctx2: null,
            erasering: false,
            penWidth: 8,
            color: "#ff0000",
            imgArr: [
                {url: img1_, name: '出行平安符'},
                {url: img2_, name: '家宅安宁符'},
                {url: img3_, name: '好感增加符'},
                {url: img4_, name: '好运降临符'}
            ],
            current: 0,
            borderX: 0,
            borderY: 0,
        };
    },
    mounted() {
        document.addEventListener("mouseup", this.mouseup);
        for (let i = 0; i < this.imgArr.length; i++) {
            // 原图
            let canvas = this.$refs["canvas" + i][0];
            // 画稿
            let canvas2 = this.$refs["canvas_" + i][0];
            this["canvas_" + i] = canvas2;
            let ctx = canvas.getContext("2d");
            let ctx2 = canvas2.getContext("2d");
            this["ctx" + i] = ctx;
            this["ctx_" + i] = ctx2;

            let img = new Image();
            img.onload = () => {
                // 调整画布大小
                this.$refs["canvas" + i][0].width = img.width;
                this.$refs["canvas" + i][0].height = img.height;
                this.canvasWidth = img.width;
                this.canvasHeight = img.height;
                this.$refs["canvas" + i][0].clientWidth;
                setTimeout(() => {
                    this["ctx" + i].drawImage(img, 0, 0, img.width, img.height);
                }, 10);
            };
            img.src = this.imgArr[i].url;
        }
    },
    beforeDestroy() {
        document.removeEventListener("mouseup", this.mouseup);
    },
    methods: {
        mousedown(e) {
            this.isDown = true;
            this.x = e.offsetX;
            this.y = e.offsetY;
        },
        mousemove(e) {
            if (!this.erasering) {
                if (!this.isDown) {
                    return;
                }
                if (!this.lock) {
                    return;
                }
                this.lock = false;
                setTimeout(() => {
                    this.lock = true;
                }, 5);
                this["ctx_" + this.current].beginPath();
                // 画笔粗细
                this["ctx_" + this.current].lineWidth = this.penWidth;
                // 画笔颜色
                this["ctx_" + this.current].strokeStyle = this.color;
                // 画笔边缘样式（圆角）
                this["ctx_" + this.current].lineCap = "round";
                this["ctx_" + this.current].moveTo(this.x, this.y);
                this["ctx_" + this.current].lineTo(e.offsetX, e.offsetY);
                this["ctx_" + this.current].stroke();
            } else {
                if (!this.isDown) {
                    return;
                }
                this["ctx_" + this.current].clearRect(
                    this.x - this.penWidth / 2,
                    this.y - this.penWidth / 2,
                    this.penWidth,
                    this.penWidth
                );
            }
            this.x = e.offsetX;
            this.y = e.offsetY;
        },
        mouseup() {
            this.isDown = false;
        },
        move(e) {
            if (e.target.classList[0] == 'border') {
                return
            }
            if (e.target.nodeName === "CANVAS") {
                this.borderX = e.offsetX + 186.5
                this.borderY = e.offsetY + 66
            } else {
                this.borderX = e.offsetX
                this.borderY = e.offsetY
            }
        },
        getScore() {
            // 获得两个canvas的像素数组[255, 255, 255, 0]
            let arr1 = this.toRgbArr(
                this["ctx" + this.current].getImageData(
                    0,
                    0,
                    this.canvasWidth,
                    this.canvasHeight
                ).data
            );
            let arr2 = this.toRgbArr(
                this["ctx_" + this.current].getImageData(
                    0,
                    0,
                    this.canvasWidth,
                    this.canvasHeight
                ).data
            );
            // 正确的像素
            let rightpx = 0;
            // 计算原图中带颜色的像素
            let colorArr1 = arr1.filter((i) => i[3] !== 0);
            console.log("原图中有" + colorArr1.length + "个像素带颜色");
            // 计算画稿中带颜色的像素
            let colorArr2 = arr2.filter((i) => i[3] !== 0);
            console.log("画稿中有" + colorArr2.length + "个像素带颜色");
            // 找到所有正确的像素
            for (let i = 0; i < arr2.length; i++) {
                if (this.isRight(arr1[i], arr2[i])) {
                    rightpx++;
                }
            }
            let allpx =
                arr2.filter(
                    (item, index) =>
                        !this.isWhite(item) || !this.isWhite(arr1[index])
                ).length || 1;
            let tempScore = parseInt((rightpx * 100) / allpx);
            this.score =
                tempScore * 1.2 > 100 ? 100 : parseInt(tempScore * 1.2);
            console.log("画正确的像素有" + rightpx);
            console.log("两幅画中共有" + allpx + "个像素带颜色");
        },
        toRgbArr(arr) {
            let res = [];
            for (let i = 0; i < arr.length; i += 4) {
                res.push([arr[i], arr[i + 1], arr[i + 2], arr[i + 3]]);
            }
            return res;
        },
        isRight(arr1, arr2) {
            return arr1[3] !== 0 && arr2[3] !== 0;
        },
        isWhite(arr) {
            return arr[3] === 0;
        },
        clear() {
            this["ctx_" + this.current].clearRect(
                0,
                0,
                this.canvasWidth,
                this.canvasHeight
            );
        },
        eraser() {
            this.erasering = !this.erasering;
        },
        changeCurrent(index) {
            this.current = index;
        },
        save() {
            let url = this["canvas_" + this.current].toDataURL();
            var a = document.createElement("a");
            a.href = url;
            // download 属性定义了下载链接的地址
            a.download = this.imgArr[this.current].name + ".png";
            a.click();
        },
        xi() {
            this.penWidth--;
            if (this.penWidth === 1) {
                this.penWidth = 1;
            }
        },
        cu() {
            this.penWidth++;
        },
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
@font-face {
     font-family: 'zf';
     src:  url('../assets/font.TTF');
}
.hello {
    display: flex;
    color: #fff;
    width: 100vw;
    height: 100vh;
    background-image: url('../assets/bg.png');
    justify-content: center;
    align-items: flex-end;
    font-family: 'zf';
    position: relative;
    .bar {
        width: 100%;
        position: absolute;
        top: 30px;
        height: 50px;
        background: linear-gradient(90deg, #ca7534 40%, #4c62d9 60%);
        display: flex;
        justify-content: space-around;
        align-items: center;
        span {
            font-size: 30px;
        }
    }
    .left {
        flex: 0 0 344px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10%;
        padding-bottom: 20px;
        .item {
            flex: 0 0 50%;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0 0 15px;
            img {
                width: 62px;
                height: 228px;
                // background: #fff;
                margin-bottom: 10px;
            }
            .btn {
                width: 138px;
                height: 50px;
                color: #0d3ced;
                display: flex;
                justify-content: center;
                align-items: center;
                background: #ff4a47;
                border-radius: 10px;
                text-align: center;
                font-size: 24px;
            }
        }
    }
    .mid {
        box-sizing: border-box;
        padding-top: 50px;
        flex: 0 0 500px;
        height: 700px;
        margin: 0 70px;
        .border {
            box-sizing: border-box;
            position: absolute;
            transform: translate(-50%, -50%);
            border: 1px solid #e00
        }
        .content {
            margin: 0 auto;
            width: 500px;
            height: 635px;
            align-self: baseline;
            background-image: url(../assets/mid.png);
            background-size: contain;
            background-repeat: no-repeat;
            position: relative;
            canvas {
                position: absolute;
                left: 50%;
                top: 50%;
                transform: translate(-50%, -50%);
            }
        }
    }
    .right {
        flex: 0 0 300px;
        font-size: 30px;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        color: #a6b9ff;
        >* {
            margin-bottom: 38px;
        }
        .pen-color {
            .color-input {
                border: none;
                outline: none;
                padding: 0;
            }
        }
        button {
            height: 50px;
            background: #86f;
            color: #000;
            display: flex;
            justify-content: center;
            align-items: center;
            border: none;
            padding: 0 10px;
            font-size: 30px;
        }
        .pen-width {
            display: flex;
            align-items: center;
        }
        .get-score {
            width: 200px;
            border-radius: 10px;
            font-family: 'zf';
            background: #ff4a47;
            color: #0d3ced;
        }
        .score {
            .span {
                color: #ef5454;
            }
        }
        .xi,.cu {
            background: #ff7e00;
            color: #576695;
            width: 38px;
            font-weight: bold;
            text-align: center;
            margin: 0 3px;
            line-height: 30px;
        }
        .btn-group {
            display: flex;
            >* {
                background: #ff7e00;
                border-radius: 10px;
                color: #3761ff;
                font-family: 'zf';
            }
            .eraser {
                margin-right: 16px;
            }
        }
        .save {
            background: #ff4a47;
            color: #0d3ced;
            height: 50px;
            display: flex;
            justify-content: center;
            align-items: center;
            width: 200px;
            border-radius: 10px;
            margin-bottom: 120px;
        }
    }
    .origin {
        opacity: 0.4;
    }
}
</style>
