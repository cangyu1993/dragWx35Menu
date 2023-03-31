<template>
  <div class="root">
    <div class="home">
      <ul class="menu_list">
        <ul v-for="(btn, i) in menu.buttons" :key="i" class="sub_buttons">
          <draggable v-model="btn.sub_button" tag="transition-group" v-bind="dragOptions" draggable=".sub_name"
            :animation="100" @change="dragChangeEvent($event, i)">
            <li v-for="(sub, i2) in btn.sub_button" :key="i2" class="sub_name"
              :class="{ 'current': isCurrentClick(2, i, i2) }" @click.stop="selectSubMenu(i, i2)">{{
                sub.name
              }}
            </li>
            <li class="add_menu_2" v-if="btn.sub_button.length < 5" @click="addMenu(2, i)" slot="footer" key="footer">+
            </li>
          </draggable>
          <li class="menu_name" :class="{ 'current': isCurrentClick(1, i, 0) }" @click.stop="selectMenu(i)">{{ btn.name }}
          </li>
        </ul>
        <li class="add_menu_1" :class="[menuLevel1Width]" v-if="menu.buttons.length < 3" @click="addMenu(1)">+</li>
      </ul>
      <ul class="area_other">
        <draggable v-model="areaBtn" tag="transition-group" v-bind="dragOptions" draggable=".sub_name" :animation="100"
          class="area_other_data" @change="dragChangeAreaEvent($event)">
          <li class="sub_name" v-for="(item, index) in areaBtn" :key="index" :class="{ 'current': isCurrentArea(index) }"
            @click.stop="selectMenuArea(index)">{{
              item.name }}</li>
          <li slot="footer" key="footer" class="area_add" @click="addAreaBtn">新增+</li>
        </draggable>
      </ul>
    </div>
    <div class="weixin-menu-detail" v-if="selectedMenuLevel > 0">
      <wx-menu-button-editor :button="selectedButton" :selectedMenuLevel="selectedMenuLevel"
        @delMenu="delMenu"></wx-menu-button-editor>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable"

export default {
  name: "HomeView",
  components: { draggable, wxMenuButtonEditor: () => import('./wx-menu-button-editor') },
  data() {
    return {
      menu: {
        buttons: [],
      },//35菜单
      areaBtn: [],
      reduction: false,//元素是否需要还原（目标位置数量超了）
      areaIndex: '',//待启用区域点击索引
      selectedMenuIndex: '', //当前选中菜单索引
      selectedSubMenuIndex: '', //当前选中子菜单索引
      selectedMenuLevel: 0, //选中菜单级别
      selectedButton: '', //选中的菜单按钮
    };
  },
  computed: {
    //底部菜单宽度
    menuLevel1Width() {
      let length = this.menu.buttons.length
      return `menu_1_${length}`
    },
    dragOptions() {
      return {
        animation: 0,
        group: "description",
        disabled: false,
        ghostClass: "ghost"
      };
    }
  },
  methods: {
    //待启用区域current样式
    isCurrentArea(index) {
      return this.areaIndex === index
    },
    //然后current样式
    isCurrentClick(level = 0, i = 0, i2 = 0) {
      if (level == '1') {
        return this.selectedMenuLevel == '1' && this.selectedMenuIndex === i
      } else {
        return this.selectedMenuLevel == '2' && this.selectedMenuIndex === i && this.selectedSubMenuIndex === i2
      }
    },
    //change事件
    dragChangeEvent(event, i) {
      if (event.hasOwnProperty('added')) {
        let { added: { newIndex } } = event
        if (this.menu.buttons[i].sub_button.length > 5) {
          this.menu.buttons[i].sub_button.splice(newIndex, 1)
          this.reduction = true
        }
      }
      if (event.hasOwnProperty('removed') && this.reduction) {
        let { removed: { element, oldIndex } } = event
        this.menu.buttons[i].sub_button.splice(oldIndex, 0, element)
        this.reduction = false
      }
    },
    //areaChange事件
    dragChangeAreaEvent(event) {
      if (event.hasOwnProperty('removed') && this.reduction) {
        let { removed: { element, oldIndex } } = event
        this.areaBtn.splice(oldIndex, 0, element)
        this.reduction = false
      }
    },
    //新增菜单
    addMenu(level, i) {
      if (level == 1 && this.menu.buttons.length < 3) {
        this.menu.buttons.push({
          type: 'view',
          name: '菜单名称',
          sub_button: [],
          url: ''
        })
      }
      if (level == 2 && this.menu.buttons[i].sub_button.length < 5) {
        this.menu.buttons[i].sub_button.push({
          type: 'view',
          name: `子菜单`,
          url: ''
        })
      }
    },
    //区域内新增菜单
    addAreaBtn() {
      this.areaBtn.push({
        type: 'view',
        name: `待启用-${this.areaBtn.length + 1}`,
        url: ''
      })
    },
    //待启用区域选择按一级菜单处理
    selectMenuArea(i) {
      this.selectedMenuLevel = 1;
      this.selectedSubMenuIndex = '';
      this.areaIndex = i;
      this.selectedButton = this.areaBtn[i];
    },
    //选中主菜单
    selectMenu(i) {
      this.selectedMenuLevel = 1;
      this.selectedSubMenuIndex = '';
      this.areaIndex = '';
      this.selectedMenuIndex = i;
      this.selectedButton = this.menu.buttons[i];
    },
    //选中子菜单
    selectSubMenu(i, i2) {
      this.areaIndex = '';
      this.selectedMenuLevel = 2;
      this.selectedMenuIndex = i;
      this.selectedSubMenuIndex = i2;
      this.selectedButton = this.menu.buttons[i].sub_button[i2];
    },
    unSelectMenu() {
      //不选中任何菜单
      this.areaIndex = '';
      this.selectedMenuLevel = 0;
      this.selectedMenuIndex = '';
      this.selectedSubMenuIndex = '';
      this.selectedButton = '';
    },
    //删除菜单
    delMenu() {
      if (this.selectedMenuLevel == 1) {
        if (this.areaIndex) {
          this.areaBtn.splice(this.areaIndex, 1);
        } else {
          this.menu.buttons.splice(this.selectedMenuIndex, 1);
        }
        this.unSelectMenu();
      } else if (this.selectedMenuLevel == 2) {
        this.menu.buttons[this.selectedMenuIndex].sub_button.splice(
          this.selectedSubMenuIndex,
          1
        );
        this.unSelectMenu();
      }
    },
  }
};
</script>

<style scoped lang="less">
.current {
  border-color: #44b549 !important;
  background: #fff;
  color: #44b549;
}

.root {
  display: flex;

  .weixin-menu-detail {
    margin-left: 660px;
  }
}

.home {
  width: 375px;
  height: 667px;
  box-sizing: border-box;
  border: 1px solid #eee;
  position: relative;

  .menu_list {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    align-items: flex-end;
    flex-shrink: 0;

    .sub_buttons {
      width: calc(375px / 3);
      box-sizing: border-box;

      .menu_name {
        height: 60px;
        text-align: center;
        padding: 20px;
        border: 1px solid #eee;
        margin-top: 10px;
        position: relative;

        &::after {
          display: block;
          content: ' ';
          position: absolute;
          left: 50%;
          top: -20px;
          transform: translate(-50%, 100%);
          width: 0;
          height: 0;
          border-style: solid;
          border-width: 8.7px 5px 0 5px;
          border-color: #eee transparent transparent transparent;
        }
      }
    }
  }

  .menu_1_3 {
    width: calc(375px / 3 * 0);
  }

  .menu_1_2 {
    width: calc(375px / 3 * 1);
  }

  .menu_1_1 {
    width: calc(375px / 3 * 2);
  }

  .menu_1_0 {
    width: calc(375px / 3 * 3);
  }

  .add_menu_1 {
    height: 60px;
    box-sizing: border-box;
    padding: 20px;
    border: 1px solid #eee;
    text-align: center;
  }

  .add_menu_2,
  .sub_name {
    height: 60px;
    text-align: center;
    padding: 20px;
    border: 1px solid #eee;
    cursor: pointer;
  }
}

.area_other {
  width: 600px;
  min-height: 667px;
  box-sizing: border-box;
  border: 1px solid #eee;
  margin-left: calc(375px + 30px);
  overflow-y: scroll;

  .area_other_data {
    padding: 25px;
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }

  .sub_name {
    margin: 5px;
    cursor: pointer;
  }

  .sub_name,
  .area_add {
    width: calc(375px / 3);
    height: 60px;
    text-align: center;
    padding: 20px;
    border: 1px solid #eee;
    margin: -1px;
    cursor: pointer;
  }
}

.flip-list-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}

.area_other::-webkit-scrollbar {
  display: none;
}

/* 隐藏 IE、Edge 和 Firefox 的滚动条 */
.area_other {
  -ms-overflow-style: none;
  /* IE and Edge */
  scrollbar-width: none;
  /* Firefox */
}

@import url(../style/wx-menu.css);
</style>
