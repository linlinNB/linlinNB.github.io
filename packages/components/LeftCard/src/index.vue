<template>
  <div
    class='left-card-body'
    v-resize='resetPagination'
  >
    <div
      class='left-card-container'
      ref='leftCardContainer'
    >
      <a-empty
        style='margin-top: 50px'
        v-if='dataSource.length === 0 && showEmpty'
        :description='emptyDescription'
      >
        <template>
          <slot name='image' />
        </template>
      </a-empty>
      <div
        v-else
        class='left-card-list'
        :style="{
          'grid-template-columns': `repeat(auto-fit, ${itemWidth})`,
          'grid-auto-rows': itemHeight
        }"
      >
        <LeftCardItem
          v-for='(feature, index) in dataSource'
          :key='feature[rowKey]'
          :feature='feature'
          :action='action'
          :config='getConfig(feature)'
          :main-img-key='mainImgKey'
          :sub-img-key='subImgKey'
          :copyable='copyable'
          v-bind='$attrs'
          v-on='$listeners'
          @selectChange='selectItem'
        >
          <template
            slot='action'
            slot-scope='scope'
          >
            <slot
              name='action'
              v-bind='scope'
            />
          </template>
          <template
              slot='itemLeft'
              slot-scope='scope'
          >
            <slot
                name='itemLeft'
                v-bind='scope'
            />
          </template>
          <template
            slot='item'
            slot-scope='scope'
          >
            <slot
              name='item'
              v-bind='scope'
              :index='index'
            />
          </template>
          <template
            slot='content'
            slot-scope='scope'
          >
            <slot
              name='content'
              v-bind='scope'
              :index='index'
            />
          </template>
          <template
            slot='actionLeft'
            slot-scope='scope'
          >
            <slot
              name='actionLeft'
              v-bind='scope'
            />
          </template>
          <!--          <template v-for='item in config' :slot='`${item.key}Name`' slot-scope='{feature}'>-->
          <!--            <slot :name='`${item.key}Name`' :feature='feature'></slot>-->
          <!--          </template>-->
          <template
            v-for='item in config'
            :slot='`${item.key}Value`'
            slot-scope='scope'
          >
            <slot
              :name='`${item.key}Value`'
              v-bind='scope'
            />
          </template>
        </LeftCardItem>
      </div>
    </div>
  </div>
</template>

<script>
import LeftCardItem from './LeftCardItem'
import resize from './utils/resize'
export default {
  name: 'LeftCard',
  components: { LeftCardItem },
  directives:{
    resize:resize
  },
  props: {
    rowKey: {
      type: String,
      default: () => 'featureId'
    },
    dataSource: {
      type: Array,
      default: () => []
    },
    mainImgKey: {
      type: String,
      default: () => ''
    },
    subImgKey: {
      type: String,
      default: () => ''
    },
    /**
     * ?????????????????????
     *hover      //hover
     preview    //??????
     operate    //????????????
     select     //??????????????????
     edit       //??????
     confirm    //????????????
     info       //????????????
     ???????????? ??????
     <slot >
     ??????1
     {
        hover:true
     }
     ??????2
     {
        hover:(feature)=>{
          //?????????????????????
          return true
        }
     }
     * */
    action: {
      type: Object,
      default: () => ({})
    },
    // ???????????????Option,
    // ?????????????????????????????????????????????
    changeConfig: {
      type: Function,
      default: undefined
    },
    /**
     * ?????????????????????
     *{
          label: '??????',   // ??????
          key: 'time',    // key
          click: (item) => {        //????????????
            console.log('??????')
          },
          style:{               //???????????????
            color:'#b32626',
            cursor:'pointer'
          } //??????????????????
        }
     */
    config: {
      type: Array,
      default: () => []
    },
    cardWidth: {
      // eslint-disable-next-line vue/require-prop-type-constructor
      type: Number,
      default: () => 350
    },
    cardHeight: {
      // eslint-disable-next-line vue/require-prop-type-constructor
      type: Number,
      default: () => 195
    },
    emptyDescription: {
      type: String,
      default: undefined
    },
    showEmpty: {
      type: Boolean,
      default: () => true
    },
    copyable: {
      // ????????????????????????
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      itemWidth: `${this.cardWidth}px`,
      itemHeight: `${this.cardHeight}px`,
      oldWidth: 0,
      oldHeight: 0,
      // resetPagination: debounce(this.dealSizeChange, 100),
      pageSize: 0,
      isFullScreen: false // ???????????????????????????
    }
  },
  watch: {
    // ??????????????? - ???????????????
    dataSource: {
      handler: function (val, oldVal) {
        if (val !== oldVal) {
          this.scrollToTop()
        }
      },
      deep: true
    }
  },
  methods: {
    // ???????????????????????????card?????????????????????????????????????????????changeOption????????????
    getConfig (feature) {
      if (this.changeConfig) {
        return this.changeConfig(feature)
      } else {
        return this.config
      }
    },
    saveItem (feature) {
      this.$emit('saveItem', feature)
    },
    selectItem (feature) {
      this.$emit('selectItem', feature)
    },
    confirmItem (feature) {
      this.$emit('confirmItem', feature)
    },
    // resetPagination(width, height) {
    //
    // },
    resetPagination (width, height) {
      const isFullScreen = document.fullscreen || document.mozFullScreen || document.webkitIsFullScreen
      if (isFullScreen) {
        this.isFullScreen = isFullScreen
        return
      }
      if (this.isFullScreen) {
        this.isFullScreen = false
        return
      }
      // ????????????????????????????????????????????????????????????????????????
      // ??????????????????????????????????????????????????????
      if (Math.abs(height - this.oldHeight) < this.cardHeight && width === this.oldWidth) {
        return
      }
      // console.log(width, height)
      this.oldWidth = width
      this.oldHeight = height
      // -----end------
      if (width && height) {
        // ?????????
        let count = width / this.cardWidth
        if (count - Math.floor(count) > 0.8) {
          count = count + 1
        }
        const columns = Math.floor(count)
        const itemWidth = (width - (columns - 1) * 10) / columns
        this.itemWidth = `${itemWidth}px`
        // console.log(itemWidth)
        // ?????????
        const rows = Math.round(height / this.cardHeight)
        if (rows * columns === this.pageSize) {
          return
        }
        this.pageSize = rows * columns
        // console.log(rows * columns)
        this.$emit('resetPagination', this.pageSize)
      }
    },
    // ???????????????
    scrollToTop () {
      this.$refs.leftCardContainer.scrollTop = 0
    }
  }
}
</script>

<style lang='less' scoped>
.left-card-body {
  height: 100%;
  width: 100%;
  overflow: hidden;
}

.left-card-container {
  height: 100%;
  width: 100%;
  overflow-y: auto;

  &::-webkit-scrollbar {
    width: 0px;
  }
}

.left-card-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  grid-gap: 10px 10px;
  grid-auto-rows: 185px;
}
</style>
