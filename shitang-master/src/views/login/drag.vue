<template>
  <div id="dragContainer" ref="dragContainer" v-show="showVerify">
    <div id="dragBg" ref="dragBg"></div>
    <div id="dragText" ref="dragText">{{ text }}</div>
    <div
      id="dragHandler"
      class="dragHandlerBg"
      ref="dragHandler"
      @mousedown="onDragHandlerMouseDown()"
      @mousemove="onDragHandlerMouseDown($event)"
    ></div>
  </div>
</template>
<script>

export default {
  
  props: ["showVerify"],
  data() {
    return {
      maxHandlerOffset: 0, //验证的最长距离
      isVertifySucc: false,
      text: null,
    };
  },
  methods: {
    initDrag() {
      this.text = "拖动滑块验证";
    },
    //选中滑块
    onDragHandlerMouseDown() {
      this.maxHandlerOffset =
        this.$refs.dragContainer.clientWidth -
        this.$refs.dragHandler.clientWidth;
      //鼠标移动监听
      document.addEventListener(
        "mousemove",
        this.onDragHandlerMouseMove
      );
      //鼠标松开监听
      document.addEventListener("mouseup", this.onDragHandlerMouseUp);
    },
    //滑块移动
    onDragHandlerMouseMove(event) {
      /*
        html元素不存在width属性,只有clientWidth
        offsetX是相对当前元素的,clientX和pageX是相对其父元素的
        */
      //滑块移动量 event是mouseevent
      var left = event.clientX - this.$refs.dragHandler.clientWidth / 2;
      //
      if (left < 0) {
        left = 0;
        //如果滑块移动量   > 滑块的最大偏移量 ，则调用验证成功函数
      } else if (left > this.maxHandlerOffset) {
        left = this.maxHandlerOffset;
        this.verifySucc();
      }
      //滑块移动量
      this.$refs.dragHandler.style.left = left + "px";
      //绿色背景的长度
      this.$refs.dragBg.style.width = this.$refs.dragHandler.style.left;
    },

    //松开滑块函数
    onDragHandlerMouseUp() {
      //移除鼠标移动监听
      document.removeEventListener("mousemove", this.onDragHandlerMouseMove);
      //移除鼠标松开监听
      document.removeEventListener("mouseup", this.onDragHandlerMouseUp);
      //初始化滑块移动量
      this.$refs.dragHandler.style.left = 0;
      //初始化绿色背景
      this.$refs.dragBg.style.width = 0;
    },

    //验证成功
    verifySucc() {
      //成功标记，不可回弹
      this.isVertifySucc = false;
      //容器文本的文字改为白色“验证通过”字体
      this.text = "验证通过";
      this.$refs.dragText.style.color = "white";
      //验证通过的滑块背景
      this.$refs.dragHandler.setAttribute("class", "dragHandlerOkBg");
      //移除鼠标按下监听
      this.$refs.dragHandler.removeEventListener(
        "mousedown",
        this.onDragHandlerMouseDown
      );
      //移除 鼠标移动监听
      document.removeEventListener("mousemove", this.onDragHandlerMouseMove);
      //移除鼠标松开监听
      document.removeEventListener("mouseup", this.onDragHandlerMouseUp);
      this.$emit("update:showVerify", false);
      this.$router.go(-1); //返回上一页
    },
  },
  mounted() {
    this.initDrag();
  },
};
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
#dragContainer {
  position: relative;
  display: flex;
  justify-items: center;
  align-items: center;
  background: #e8e8e8;
  width: 300px;
  height: 33px;
  border: 2px solid #333;

  #dragBg {
    position: absolute;
    background-color: #7ac23c;
    width: 0px;
    height: 100%;
  }

  #dragText {
    position: absolute;
    width: 100%;
    height: 100%;
    text-align: center;
    line-height: 33px;
  }

  #dragHandler {
    position: absolute;
    width: 40px;
    height: 100%;
    cursor: move;
  }

  /* 滑块初始背景 */
  .dragHandlerBg {
    background: #333 no-repeat center
      url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA3hpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNS1jMDIxIDc5LjE1NTc3MiwgMjAxNC8wMS8xMy0xOTo0NDowMCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDo0ZDhlNWY5My05NmI0LTRlNWQtOGFjYi03ZTY4OGYyMTU2ZTYiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NTEyNTVEMURGMkVFMTFFNEI5NDBCMjQ2M0ExMDQ1OUYiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6NTEyNTVEMUNGMkVFMTFFNEI5NDBCMjQ2M0ExMDQ1OUYiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTQgKE1hY2ludG9zaCkiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDo2MTc5NzNmZS02OTQxLTQyOTYtYTIwNi02NDI2YTNkOWU5YmUiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6NGQ4ZTVmOTMtOTZiNC00ZTVkLThhY2ItN2U2ODhmMjE1NmU2Ii8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+YiRG4AAAALFJREFUeNpi/P//PwMlgImBQkA9A+bOnfsIiBOxKcInh+yCaCDuByoswaIOpxwjciACFegBqZ1AvBSIS5OTk/8TkmNEjwWgQiUgtQuIjwAxUF3yX3xyGIEIFLwHpKyAWB+I1xGSwxULIGf9A7mQkBwTlhBXAFLHgPgqEAcTkmNCU6AL9d8WII4HOvk3ITkWJAXWUMlOoGQHmsE45ViQ2KuBuASoYC4Wf+OUYxz6mQkgwAAN9mIrUReCXgAAAABJRU5ErkJggg==");
  }

  /* 验证成功时的滑块背景 有√*/
  .dragHandlerOkBg {
    background: #333 no-repeat center
      url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA3hpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNS1jMDIxIDc5LjE1NTc3MiwgMjAxNC8wMS8xMy0xOTo0NDowMCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDo0ZDhlNWY5My05NmI0LTRlNWQtOGFjYi03ZTY4OGYyMTU2ZTYiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NDlBRDI3NjVGMkQ2MTFFNEI5NDBCMjQ2M0ExMDQ1OUYiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6NDlBRDI3NjRGMkQ2MTFFNEI5NDBCMjQ2M0ExMDQ1OUYiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTQgKE1hY2ludG9zaCkiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDphNWEzMWNhMC1hYmViLTQxNWEtYTEwZS04Y2U5NzRlN2Q4YTEiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6NGQ4ZTVmOTMtOTZiNC00ZTVkLThhY2ItN2U2ODhmMjE1NmU2Ii8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+k+sHwwAAASZJREFUeNpi/P//PwMyKD8uZw+kUoDYEYgloMIvgHg/EM/ptHx0EFk9I8wAoEZ+IDUPiIMY8IN1QJwENOgj3ACo5gNAbMBAHLgAxA4gQ5igAnNJ0MwAVTsX7IKyY7L2UNuJAf+AmAmJ78AEDTBiwGYg5gbifCSxFCZoaBMCy4A4GOjnH0D6DpK4IxNSVIHAfSDOAeLraJrjgJp/AwPbHMhejiQnwYRmUzNQ4VQgDQqXK0ia/0I17wJiPmQNTNBEAgMlQIWiQA2vgWw7QppBekGxsAjIiEUSBNnsBDWEAY9mEFgMMgBk00E0iZtA7AHEctDQ58MRuA6wlLgGFMoMpIG1QFeGwAIxGZo8GUhIysmwQGSAZgwHaEZhICIzOaBkJkqyM0CAAQDGx279Jf50AAAAAABJRU5ErkJggg==");
  }
}
</style>