<template>
  <div class="vue-cropper">
    <div
      class="vue-cropper-card"
      @mouseup="
        clipDown = false;
        dotDown = false;
      "
      @mousemove="cardMouseMove"
      :style="{
        width: `${width}px`,
        height: `${height}px`,
      }"
    >
      <canvas class="vue-cropper-cardbg" ref="cardBg"></canvas>
      <canvas class="vue-cropper-modal" ref="cardModal"> </canvas>
      <div
        class="vue-cropper-clip"
        ref="clip"
        :style="clipStyle"
        @mousedown="clipDown = true"
        @mouseup="clipDown = false"
      >
        <div class="dot top-left" @mousedown="register('topleft')"></div>
        <div class="dot top-right" @mousedown="register('topright')"></div>
        <div class="dot top-center" @mousedown="register('topcenter')"></div>
        <div class="dot bottom-left" @mousedown="register('bottomleft')"></div>
        <div
          class="dot bottom-center"
          @mousedown="register('bottomcenter')"
        ></div>
        <div
          class="dot bottom-right"
          @mousedown="register('bottomright')"
        ></div>
        <div class="dot left-center" @mousedown="register('leftcenter')"></div>
        <div
          class="dot right-center"
          @mousedown="register('rightcenter')"
        ></div>
      </div>
    </div>
    <canvas class="vue-cropper-clip-image" ref="clipImage"></canvas>

    <div>
      <button @click="exportImg">save</button>
    </div>
  </div>
</template>

<script lang="ts">
  import { ref, onMounted } from "vue";

  interface shrinkImage {
    imageWidth: number;
    imageHeight: number;
    width: number;
    height: number;
    base?: number;
  }

  interface clipStyle extends Record<any, any> {
    width: string;
    height: string;
    top: string;
    left: string;
  }

  // https://stackoverflow.com/questions/12168909/blob-from-dataurl
  // const dataURItoBlob = async (url:string) => await (await fetch(url)).blob();
  function dataURItoBlob(dataURI: string) {
    // convert base64 to raw binary data held in a string
    // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
    var byteString = atob(dataURI.split(",")[1]);

    // separate out the mime component
    var mimeString = dataURI.split(",")[0].split(":")[1].split(";")[0];

    // write the bytes of the string to an ArrayBuffer
    var ab = new ArrayBuffer(byteString.length);

    // create a view into the buffer
    var ia = new Uint8Array(ab);

    // set the bytes of the buffer to the correct values
    for (var i = 0; i < byteString.length; i++) {
      ia[i] = byteString.charCodeAt(i);
    }

    // write the ArrayBuffer to a blob, and you're done
    var blob = new Blob([ab], { type: mimeString });
    return blob;
  }
  const saveData = (function () {
    const a = document.createElement("a");
    document.body.appendChild(a);
    return function (blob: Blob, fileName: string) {
      let url = window.URL.createObjectURL(blob);
      a.href = url;
      a.download = fileName;
      a.click();
      window.URL.revokeObjectURL(url);
    };
  })();

  export default {
    props: {
      image: {
        type: String,
        required: true,
      },
      width: {
        type: Number,
        required: false,
        default: 500,
      },
      height: {
        type: Number,
        required: false,
        default: 300,
      },
    },
    setup(props: any, { emit }: any) {
      let defaultCanvas = document.createElement("canvas");

      let cardBg = ref<HTMLCanvasElement>(defaultCanvas);
      let cardModal = ref<HTMLCanvasElement>(defaultCanvas);
      let clip = ref<HTMLDivElement>();
      let clipStyle = ref<clipStyle>({
        width: "0",
        height: "0",
        left: "0",
        top: "0",
        border: "1px solid #1E9EFB",
      });
      let clipImage = ref<HTMLCanvasElement>(defaultCanvas);

      let dotDown = ref(false);
      let dotType = ref<string>("topcenter");
      let clipDown = ref(false);

      let basePaintParams = {
        baseOffsetX: 0,
        baseOffsetY: 0,
        w: 180,
        h: 180,
      };

      const clearModal = () => {
        let ctx = cardModal.value.getContext("2d") as CanvasRenderingContext2D;
        ctx.clearRect(0, 0, props.width, props.height);
        ctx.fillStyle = "rgba(0,0,0,0.5)";
        ctx.fillRect(0, 0, props.width, props.height);
      };

      const fillAndSet = () => {
        // 设置clip的style
        clipStyle.value.width = `${Math.abs(basePaintParams.w)}px`;
        clipStyle.value.height = `${Math.abs(basePaintParams.h)}px`;
        clipStyle.value.left = `${basePaintParams.baseOffsetX}px`;
        clipStyle.value.top = `${basePaintParams.baseOffsetY}px`;

        clearModal();

        let ctx = cardModal.value.getContext("2d") as CanvasRenderingContext2D;

        ctx.save();
        ctx.beginPath();
        ctx.globalCompositeOperation = "destination-out";
        ctx.fillStyle = "black";
        ctx.fillRect(
          basePaintParams.baseOffsetX,
          basePaintParams.baseOffsetY,
          basePaintParams.w,
          basePaintParams.h
        );
        ctx.fill();
        ctx.closePath();
        ctx.restore();

        save();
      };

      const save = () => {
        let bgCtx = cardBg.value.getContext("2d") as CanvasRenderingContext2D;

        let imageData = bgCtx.getImageData(
          basePaintParams.w < 0
            ? basePaintParams.baseOffsetX + basePaintParams.w
            : basePaintParams.baseOffsetX,
          basePaintParams.h < 0
            ? basePaintParams.baseOffsetY + basePaintParams.h
            : basePaintParams.baseOffsetY,
          Math.abs(basePaintParams.w),
          Math.abs(basePaintParams.h)
        );

        let targetCanvas = clipImage.value.getContext(
          "2d"
        ) as CanvasRenderingContext2D;
        clipImage.value.width = basePaintParams.w;
        clipImage.value.height = basePaintParams.h;
        targetCanvas.putImageData(
          imageData,
          0,
          0,
          0,
          0,
          basePaintParams.w,
          basePaintParams.h
        );
      };

      const exportImg = async () => {
        let MIME_TYPE = "image/png";
        let imgURL = clipImage.value.toDataURL(MIME_TYPE);
        emit("dataURIChange", imgURL);
        let res = await dataURItoBlob(imgURL);
        emit("blobChange", res);
        saveData(res, "data.png");
      };

      const register = (_class: string) => {
        dotDown.value = true;
        dotType.value = _class;
      };

      const cardMouseMove = (e: MouseEvent) => {
        if (!dotDown.value && !clipDown.value) {
          return;
        }

        const { offsetX, offsetY } = e;

        const topYChange = () => {
          if (e.target === cardModal.value) {
            if (offsetY < basePaintParams.baseOffsetY) {
              basePaintParams.h += basePaintParams.baseOffsetY - offsetY;
              basePaintParams.baseOffsetY = offsetY;
            }
          } else if (e.target === clip.value) {
            basePaintParams.h -= offsetY;
            basePaintParams.baseOffsetY += offsetY;
          }
        };

        const bottomYChange = () => {
          // 对于下边的点来说，鼠标移动到画板上高度变为 offsetY - 上坐标的Y。
          if (e.target === cardModal.value) {
            if (offsetY > basePaintParams.baseOffsetY) {
              basePaintParams.h = offsetY - basePaintParams.baseOffsetY;
            }
          } else if (e.target === clip.value) {
            basePaintParams.h = offsetY;
          }
        };

        const leftXChange = () => {
          if (e.target === cardModal.value) {
            if (offsetX < basePaintParams.baseOffsetX) {
              basePaintParams.w += basePaintParams.baseOffsetX - offsetX;
              basePaintParams.baseOffsetX = offsetX;
            }
          } else if (e.target === clip.value) {
            basePaintParams.w -= offsetX;
            basePaintParams.baseOffsetX += offsetX;
          }
        };

        const rightXChange = () => {
          if (e.target === cardModal.value) {
            if (offsetX > basePaintParams.baseOffsetX) {
              basePaintParams.w = offsetX - basePaintParams.baseOffsetX;
            }
          } else if (e.target === clip.value) {
            basePaintParams.w = offsetX;
          }
        };

        // const shrink = (type) => {
        //   let cpp = this.clipPaintParams
        //   // 都是数字，可以浅拷贝
        //   let backup = {
        //     ...cpp
        //   }
        //   // 鼠标向下移动时Y数值为正
        //   // 鼠标向上移动时Y数值为负
        //   if (type === 'topleft') {
        //     // 对于左上角来说，基础XY(左上角的点)应该根据与movement相同
        //     // 裁剪框的大小在向上移动时需要增加。
        //     cpp.baseOffsetY += movementY
        //     cpp.baseOffsetX += movementY
        //     cpp.w -= movementY
        //     cpp.h -= movementY
        //   }

        //   if (type === 'topright') {
        //     cpp.baseOffsetY += movementY
        //     cpp.w -= movementY
        //     cpp.h -= movementY
        //   }

        //   if (type === 'bottomleft') {
        //     cpp.baseOffsetX -= movementY
        //     cpp.w += movementY
        //     cpp.h += movementY
        //   }

        //   if (type === 'bottomright') {
        //     cpp.w += movementY
        //     cpp.h += movementY
        //   }

        //   if (type === 'rightcenter') {
        //     cpp.baseOffsetY -= movementX
        //     cpp.w += 2 * movementX
        //     cpp.h += 2 * movementX
        //   }

        //   if (type === 'leftcenter') {
        //     cpp.baseOffsetX += 2 * movementX
        //     cpp.baseOffsetY += movementX
        //     cpp.w -= 2 * movementX
        //     cpp.h -= 2 * movementX
        //   }

        //   if (type === 'bottomcenter') {
        //     cpp.baseOffsetX -= movementY
        //     cpp.w += 2 * movementY
        //     cpp.h += 2 * movementY
        //   }

        //   if (type === 'topcenter') {
        //     cpp.baseOffsetY += 2 * movementY
        //     cpp.baseOffsetX += movementY
        //     cpp.w -= 2 * movementY
        //     cpp.h -= 2 * movementY
        //   }

        //   if (cpp.h < 10 || cpp.w < 10) {
        //     this.clipPaintParams = backup
        //   }

        // }

        const dotRunMap: Record<any, Function> = {
          topcenter: () => {
            topYChange();
          },
          bottomcenter: () => {
            bottomYChange();
          },
          leftcenter: () => {
            leftXChange();
          },
          rightcenter: () => {
            rightXChange();
          },
          bottomright: () => {
            rightXChange();
            bottomYChange();
          },
          bottomleft: () => {
            leftXChange();
            bottomYChange();
          },
          topright: () => {
            rightXChange();
            topYChange();
          },
          topleft: () => {
            leftXChange();
            topYChange();
          },
        };

        if (dotDown.value) {
          dotRunMap[dotType.value]();
        }

        if (clipDown.value && !dotDown.value) {
          basePaintParams.baseOffsetX =
            basePaintParams.baseOffsetX + e.movementX;
          basePaintParams.baseOffsetY =
            basePaintParams.baseOffsetY + e.movementY;
        }
        fillAndSet();
      };

      const shrinkImage = ({
        imageWidth,
        imageHeight,
        width,
        height,
        base = 1,
      }: shrinkImage): Record<any, number> => {
        // 根据height和width 返回一个合适的 imageWidth和imageHeight。

        if (imageWidth / base < width && imageHeight / base < height) {
          return {
            imageWidth: imageWidth / base,
            imageHeight: imageHeight / base,
          };
        }

        return shrinkImage({
          imageWidth,
          imageHeight,
          width,
          height,
          base: base + 0.1,
        });
      };

      const getImage = () => {
        let img = new Image();

        img.src = props.image;

        img.onload = initCanvas;
      };

      const initCanvas = (e: Event) => {
        let img = e.target as HTMLImageElement;

        cardBg.value.width = props.width;
        cardBg.value.height = props.height;
        cardModal.value.width = props.width;
        cardModal.value.height = props.height;

        let cardBgCtx: any = cardBg.value.getContext("2d");

        let { imageWidth, imageHeight } = shrinkImage({
          imageWidth: img.width,
          imageHeight: img.height,
          width: props.width,
          height: props.height,
        });

        cardBgCtx.drawImage(
          img,
          0,
          0,
          img.width,
          img.height,
          (props.width - imageWidth) / 2,
          (props.height - imageHeight) / 2,
          imageWidth,
          imageHeight
        );

        basePaintParams = {
          baseOffsetX: (props.width - imageWidth) / 2,
          baseOffsetY: (props.height - imageHeight) / 2,
          w: imageWidth,
          h: imageHeight,
        };
        fillAndSet();
      };

      const init = () => {
        if (!props.image) {
          return;
        }

        getImage();
      };

      onMounted(() => {
        init();
      });

      return {
        cardBg,
        cardModal,
        clip,
        clipStyle,
        clipImage,
        dotDown,
        clipDown,
        cardMouseMove,
        register,
        exportImg,
      };
    },
  };
</script>

<style lang="less">
  .vue-cropper {
    display: flex;
    flex-direction: column;
    &-card {
      position: relative;
      flex-shrink: 0;
      overflow: hidden;
      margin: auto;
      &bg {
        position: absolute;
        top: 0;
        left: 0;
        background: linear-gradient(
            45deg,
            #ccc 25%,
            transparent 0,
            transparent 75%,
            #ccc 0
          ),
          linear-gradient(
            45deg,
            #ccc 25%,
            transparent 0,
            transparent 75%,
            #ccc 0
          );
        background-position: 0 0, 4px 4px;
        background-size: 8px 8px;
        z-index: -1;
      }
    }

    &-modal {
      position: absolute;
      top: 0;
      left: 0;
    }

    &-clip {
      position: absolute;
      cursor: all-scroll;
    }

    &-clip-image {
      margin: auto;
    }

    .dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background: #1e9efb;
      position: absolute;
    }

    .top-left {
      top: -5px;
      left: -5px;
      cursor: nwse-resize;
    }

    .top-right {
      top: -5px;
      right: -5px;
      cursor: nesw-resize;
    }

    .top-center {
      top: -5px;
      left: 50%;
      transform: translateX(-50%);
      cursor: ns-resize;
    }

    .bottom-center {
      bottom: -5px;
      left: 50%;
      transform: translateX(-50%);
      cursor: ns-resize;
    }

    .bottom-left {
      bottom: -5px;
      left: -5px;
      cursor: nesw-resize;
    }

    .bottom-right {
      bottom: -5px;
      right: -5px;
      cursor: nwse-resize;
    }

    .left-center {
      top: 50%;
      transform: translateY(-50%);
      left: -5px;
      cursor: ew-resize;
    }

    .right-center {
      top: 50%;
      transform: translateY(-50%);
      right: -5px;
      cursor: ew-resize;
    }
  }
</style>
