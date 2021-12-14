<template>
    <div class="hello">
        <div class="left">
            <div 
                class="item" v-for="(item,index) in imgArr" 
                :key="index"
            >
                <img :src="item.url" alt="">
                <div class="btn" @click="changeCurrent(index)">{{item.name}}</div>
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
            <button class="eraser" @click="eraser">
                {{erasering ? 'PEN' : 'ERASER'}}
            </button>
            <button class="clear" @click="clear">
                CLEAR ALL
            </button>
            <button @click="getScore">
                CLICK ME GOT SCORE
            </button>
            <div>
                YOUR SCORE IS 
                <span class="red">
                    {{score}}
                </span>
            </div>
            <div class="pen-color">
                LINE COLOR
                <input type="color" name="color" ref="color" v-model="color">
            </div>
            <div class="pen-width">
                BRUSH SIZE
                <div class="xi" @click="xi">-</div>
                {{penWidth}}
                <div class="cu" @click="cu">＋</div>
            </div>
            <div class="save" @click="save">
                <img src="../assets/save.png" alt="">
                <span>CLICK ME TO SAVE</span> 
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
            canvasWidth: 200,
            canvasHeight: 200,
            ctx: null,
            ctx2: null,
            erasering: false,
            penWidth: 8,
            color: "#ff0000",
            imgArr: [
                {url: img1_, name: 'TRVAL SAFE'}, 
                {url: img2_, name: 'HOUSE PROTECTION'}, 
                {url: img3_, name: 'PROMOTION OPPORTUNITY'}, 
                {url: img4_, name: 'GOOD FORTUNE'}
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
                this.borderX = e.offsetX + 246.5
                this.borderY = e.offsetY + 50
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
.hello {
    display: flex;
    color: #fff;
    background-image: url('../assets/background.png');
    justify-content: center;
    font-size: 18px;
    .left {
        flex: 0 0 344px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10%;
        .item {
            flex: 0 0 50%;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0 0 15px;
            img {
                width: 62px;
                height: 228px;
                background: #ffae00;
                margin-bottom: 10px;
            }
            .btn {
                width: 138px;
                height: 60px;
                color: #fff;
                display: flex;
                justify-content: center;
                align-items: center;
                background: #ee0000;
                border-radius: 10px;
                text-align: center;
            }
        }
    }
    .mid {
        flex: 0 0 694px;
        .border {
            box-sizing: border-box;
            position: absolute;
            transform: translate(-50%, -50%);
            border: 1px solid #e00
        }
        .content {
            margin: 0 auto;
            width: 694px;
            height: 900px;
            background-image: url(../assets/fubg.png);
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
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-end;
        >* {
            margin-bottom: 18px;
        }
        button {
            height: 50px;
            background: #e00;
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            border: none;
            border-radius: 10px;
            padding: 0 10px;
            font-size: 18px;
        }
        .red {
            color: #e00;
        }
        .pen-width {
            display: flex;
            align-items: center;
        }
        .xi,.cu {
            background: #fff;
            border-radius: 5px;
            color: #e00;
            width: 30px;
            font-weight: bold;
            text-align: center;
            margin: 0 3px;
        }
        .save {
            margin-top: 50px;
            display: flex;
            flex-direction: column;
            color: #e00;
            align-items: center;
            img {
                width: 100px;
                margin-bottom: 14px;
            }
        }
    }
    .origin {
        opacity: 0.2;
    }
}
</style>
