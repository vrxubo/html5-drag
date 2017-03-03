<template>
  <div class="drag-container">
    <div
      class="left"
      ref="parentContainer"
    >
      <div
        class="left-sub"
        ref="subContainer"
      >
        <div class="left-sub-sub" @drop="handleDrop"></div>
      </div>
    </div>
    <div class="right">
      <span @dragstart="handleDragStart" draggable>拿走吧</span>
    </div>
  </div>
</template>
<script>
  /**
   * 只在chrome和firefox做过测试 完全没有考虑过IE
   *
   * 1。drag事件进入子元素时会触发父元素的 dragleave 事件，
   * 如果子元素没有在 dragenter 事件里调用 event.stopPropagation()
   * 则进入子元素的时候 会同时触发父元素的 dragleave 和 dragenter 事件
   * 并且会先触发 enter 事件 后触发 leave 事件
   * 如果不做任何处理会造成 拖拽元素已经离开父元素的假象 事实上并没有  只是进入了子元素
   * 因此加了一个计数器， 每次触发enter事件时令计数器加1， 触发leave事件时 计数器-1
   * 计数器===0时，说明真的离开了当前节点
   *
   * 2。 如果子元素在dragenter事件里调用了 event.stopPropagation()
   * 导致在进入子元素的时候只触发了父元素的leave事件 而没有触发enter事件
   * 计数器归零，同样会造成已经离开父元素的假象，
   * 这种情况因为子元素调用了 event.stopPropagation() 说明并不想与父元素事件有关联
   * 因此默认正常
   * @param elem 触发元素
   * @returns {Function}
   */
  function bindDrag (elem) {
    // 计数器，
    let dragStatus = 0
    return function (handleDragEnter, handleDragLeave, handleDrop) {
      /**
       * 进入元素时 阻止事件冒泡
       * 计数器加1 调用 dragenter 处理函数
       */
      elem.addEventListener('dragenter', (event) => {
        event.stopPropagation()
        // 如果只想触发一次 enter 事件
        // 可以写成这样
        dragStatus === 0 && handleDragEnter && handleDragEnter(event)
        // 无所谓的话这样也可以 handleDragEnter && handleDragEnter(event)
        dragStatus++
      })
      /**
       * 触发dragleave事件时 阻止事件冒泡
       * 计数器减1 判断计数器是否小于等于0 true 则 调用dragleave 处理函数
       * 否则 什么也不做
       */
      elem.addEventListener('dragleave', (event) => {
        event.stopPropagation()
        dragStatus -= 1
        if (dragStatus <= 0) {
          handleDragLeave && handleDragLeave(event)
        }
      })
      /**
       * 触发drop事件 阻止事件冒泡 阻止默认事件 这两个必须同时调用
       * 在firefox里，如果没有同时调用 preventDefault 和 stopPropagation
       * firefox会弹出一个新窗口
       * 令计数器归零
       * 如果有 drop 处理函数 调一下
       */
      elem.addEventListener('drop', (event) => {
        event.preventDefault()
        event.stopPropagation()
        dragStatus = 0
        handleDrop && handleDrop(event)
      })
    }
  }

  export default {
    data () {
      return {}
    },
    mounted () {
      bindDrag(this.$refs.parentContainer)(() => {
        console.log('drag enter')
      }, () => {
        console.log('drag leave')
      })
      bindDrag(this.$refs.subContainer)(() => {
        console.log('drag sub enter')
      }, () => {
        console.log('drag sub leave')
      })
    },
    methods: {
      handleDrop () {
        // 如果需要在子元素里触发 drop 事件
        // 不要调event.stopPropagation()
        // 否则父元素不能触发drop事件 计数器无法归零 后果自负
        console.log('drop')
      },
      handleDragStart (event) {
        event.dataTransfer.effectAllowed = 'move'
        // firefox 必须setData元素才能拖拽
        event.dataTransfer.setData('text/plain', 'test')
        return true
      }
    }
  }
</script>
<style>
  .drag-container {
    height: 100%;
    display: flex;
    .left, .right {
      border: 1px solid #eee;
      flex: 1;
    }
    .left {
      .left-sub {
        width: 400px;
        height: 400px;
        border: 1px solid #ddd;
        background-color: #f2f2f2;
        .left-sub-sub {
          width: 200px;
          height: 200px;
          border: 1px solid #ddd;
          background-color: #fff;
        }
      }
    }
  }
</style>
