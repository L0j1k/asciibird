<template>
  <div>
    <t-card class="overflow-x-scroll h-full">
      <div :style="`height: ${blocksWidthHeight.h}px;width: ${blocksWidthHeight.w}px;`">
        <canvas
          :ref="canvasName"
          :id="canvasName"
          class="previewcanvas"
          :width="blocksWidthHeight.w"
          :height="blocksWidthHeight.h"
        />
      </div>
    </t-card>
  </div>
</template>


<script>
import { mircColours99, blockWidth, blockHeight, cyrb53, getBlocksWidth, filterNullBlocks  } from "../../ascii";

export default {
  name: "BrushCanvas",
  created() {
    window.addEventListener('load', () => {
         // Fixes the font on load issue
         this.delayRedrawCanvas();
    })
  },
  mounted() {
    this.ctx = this.$refs[this.canvasName].getContext("2d");
    this.delayRedrawCanvas();
  },
  props: {
    blocks: {
      type: [Array, Boolean],
      required: false,
      default: false,
    },
  },
  data: () => ({
    ctx: null,
    redraw: true,
  }),
  computed: {
    blockWidth() {
      return blockWidth * this.blockSizeMultiplier;
    },
    blockHeight() {
      return blockHeight * this.blockSizeMultiplier;
    },
    blockSizeMultiplier() {
      return this.$store.getters.blockSizeMultiplier
    },
    currentAscii() {
      return this.$store.getters.currentAscii;
    },
    toolbarState() {
      return this.$store.getters.toolbarState;
    },
    isTargettingBg() {
      return this.$store.getters.isTargettingBg;
    },
    isTargettingFg() {
      return this.$store.getters.isTargettingFg;
    },
    isTargettingChar() {
      return this.$store.getters.isTargettingChar;
    },
    currentFg() {
      return this.$store.getters.currentFg;
    },
    currentBg() {
      return this.$store.getters.currentBg;
    },
    currentChar() {
      return this.$store.getters.currentChar;
    },
    brushSizeHeight() {
      return this.$store.getters.brushSizeHeight;
    },
    brushSizeWidth() {
      return this.$store.getters.brushSizeWidth;
    },
    brushSizeType() {
      return this.$store.getters.brushSizeType;
    },
    options() {
      return this.$store.getters.options;
    },
    canvasName() {
      let hash = cyrb53(JSON.stringify(this.getBlocks))
      return `${hash}-brush-canvas`;
    },
    getBlocks() {
      return this.blocks === false
        ? this.$store.getters.brushBlocks
        : this.blocks;
    },
    isMainCanvas() {
      return this.blocks === false;
    },
    blocksWidthHeight() {
      return {
          w: getBlocksWidth(this.getBlocks) * this.blockWidth,
          h: this.getBlocks.length * this.blockHeight
        }
    },
    mircColours() {
      return mircColours99;
    },
    canvasRef() {
      return this.$refs[this.canvasName];
    }
  },
  watch: {
    blockSizeMultiplier() {
      this.delayRedrawCanvas();
    },
    getBlocks() {
      this.delayRedrawCanvas();
    },
    blocks() {
      this.delayRedrawCanvas();
    },
    currentAscii() {
      this.delayRedrawCanvas();
    },
    brushSizeHeight() {
      this.delayRedrawCanvas();
    },
    brushSizeWidth() {
      this.delayRedrawCanvas();
    },
    isTargettingBg() {
      this.delayRedrawCanvas();
    },
    isTargettingFg() {
      this.delayRedrawCanvas();
    },
    isTargettingChar() {
      this.delayRedrawCanvas();
    },
    currentFg() {
      this.delayRedrawCanvas();
    },
    currentBg() {
      this.delayRedrawCanvas();
    },
    currentChar() {
      this.delayRedrawCanvas();
    },
  },
  methods: {
    getBlocksWidth(blocks) {
      return getBlocksWidth(blocks)
    },
    filterNullBlocks(blocks) {
      return filterNullBlocks(blocks)
    },
    drawPreview() {
      this.ctx.clearRect(0, 0, this.canvasRef.width, this.canvasRef.height);
      this.ctx.fillStyle = this.mircColours[1];

      // hack font for ascii shout outs 2 beenz
      this.ctx.font = "13px Hack";

      let y = 0;
      let x = 0;

      // Get middle block
      if (this.getBlocks) {
        let blocksWidth = this.getBlocksWidth(this.getBlocks)
        for (y = 0; y < this.getBlocks.length; y++) {
          for (x = 0; x < blocksWidth; x++) {
            if (this.getBlocks[y] && this.getBlocks[y][x]) {
              const curBlock = this.getBlocks[y][x];

              if (curBlock.bg !== null) {
                this.ctx.fillStyle = this.mircColours[curBlock.bg];

                this.ctx.fillRect(
                  x * this.blockWidth,
                  y * this.blockHeight,
                  this.blockWidth,
                  this.blockHeight
                );
              }

              if (curBlock.fg !== null) {
                this.ctx.fillStyle = this.mircColours[curBlock.fg];
              }

              if (curBlock.char !== null) {
                this.ctx.fillStyle = this.mircColours[curBlock.fg];
                this.ctx.fillText(
                  curBlock.char,
                  x * this.blockWidth,
                  y * this.blockHeight + this.blockHeight - 3
                );
              }
            }
          }
        }
      }
    },
    delayRedrawCanvas() {
      if (this.redraw) {
        this.redraw = false;

        requestAnimationFrame(() => {
          this.redraw = true;
          this.drawPreview();
        });
      }
    },
  },
};
</script>
