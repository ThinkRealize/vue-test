<template>
  <div>
    <el-table v-parent2="hiddenOccultationCallback"
      :data="tableData"
      style="width: 100%">
      <el-table-column prop="date"
        label="日期"
        width="180">
        <template scope="scope">
          <p v-chilrd:display="{scope: scope, order: 0}">{{scope.row.date}}</p>
          <el-input v-chilrd:occultation="{scope: scope, order: 0}"
            v-model="scope.row.date"
            placeholder=""></el-input>
        </template>
      </el-table-column>
      <el-table-column prop="name"
        label="姓名"
        width="180">
        <template scope="scope">
          <p v-chilrd:display.last="{scope: scope, order: 1}">{{scope.row.name}}</p>
          <el-input v-chilrd:occultation.last="{scope: scope, order: 1}"
            v-model="scope.row.name"
            placeholder=""></el-input>
        </template>
      </el-table-column>
      <el-table-column prop="address"
        label="地址">
      </el-table-column>
    </el-table>
    <span></span>
    <span></span>
  </div>
</template>

<script>
const TYPE_DISPLAY = 'display'
const TYPE_OCCULTATION = 'occultation'
const EVENT_DISPLAY_CLICK = `${TYPE_DISPLAY}-click`
const EVENT_OCCULTATION_FOCUS = `${TYPE_OCCULTATION}-focus`
const EVENT_INSERTED = 'inserted'
const KEY_ENTER = 13
const KEY_TAB = 9

// const focus = (el) => {
//   const input = getInput(el)
//   if (input) input.focus()
// }

export default {
  directives: {
    parent2: {
      inserted: function (el, bingding) {
        const hiddenOccultationCallback = bingding.value
        const rowCellList = []
        let focusChilrd = {}
        const switchRowCellShow = ({ row, type, isShow }) => {
          if (row === undefined || row >= rowCellList.length) {
            console.log('row invalid', row)
            return
          }
          rowCellList[row].forEach((cell) => {
            const element = cell[type]
            if (isShow) {
              element.el.style = {}
              element.isShow = true
            } else {
              element.el.style.display = 'none'
              element.isShow = false
            }
          })
        }
        const switchRowDisplayShow = (row, isShow = false) => {
          switchRowCellShow({
            row,
            type: TYPE_DISPLAY,
            isShow: isShow
          })
          switchRowCellShow({
            row,
            type: TYPE_OCCULTATION,
            isShow: !isShow
          })
          if (isShow) {
            hiddenOccultationCallback(row)
          }
        }
        const focuesRowCell = (row, column) => {
          if (row >= rowCellList.length) {
            console.log('row invalid', row)
            return
          }
          const cell = rowCellList[row][column]
          const { input } = cell[TYPE_OCCULTATION]
          if (input) input.focus()
        }
        const focusRowFirstCell = (row) => {
          focuesRowCell(row, 0)
        }
        el.addEventListener(EVENT_INSERTED, ev => {
          const { chilrd, chilrd: { row, column, isDefaultShow, isDefaultHidden } } = ev.detail
          const cellList = rowCellList[row] ? rowCellList[row] : []
          const cell = cellList[column] ? cellList[column] : {}
          chilrd.isLastRow = row === length - 1
          chilrd.isLastCell = chilrd.isLastColumn && chilrd.isLastRow
          if (isDefaultShow) {
            cell[TYPE_DISPLAY] = chilrd
          }
          if (isDefaultHidden) {
            cell[TYPE_OCCULTATION] = chilrd
          }
          cellList[column] = cell
          rowCellList[row] = cellList
        })
        el.addEventListener(EVENT_DISPLAY_CLICK, ev => {
          const { chilrd: { row, column } } = ev.detail
          if (focusChilrd.row !== undefined) {
            switchRowDisplayShow(focusChilrd.row, true)
          }
          switchRowDisplayShow(row, false)
          focuesRowCell(row, column)
        })
        el.addEventListener('keydown', ev => {
          const { keyCode } = ev
          const { el, isLastColumn, row, isShow } = focusChilrd
          if (((keyCode === KEY_TAB && isLastColumn) || keyCode === KEY_ENTER) && el && isShow) {
            switchRowDisplayShow(row, true)
            switchRowDisplayShow(row + 1, false)
            if (keyCode === KEY_ENTER) {
              focusRowFirstCell(row + 1)
            }
          }
        }, true)
        el.addEventListener(EVENT_OCCULTATION_FOCUS, ev => {
          const { chilrd } = ev.detail
          focusChilrd = chilrd
        })
        document.addEventListener('click', ev => {
          let hidden = false
          for (let cellList of rowCellList) {
            for (let cell of cellList) {
              if (cell[TYPE_DISPLAY].el === ev.target) {
                return
              }
              if (cell[TYPE_OCCULTATION].input !== ev.target) {
                hidden = true
              }
            }
          }
          if (hidden && focusChilrd.row !== undefined) {
            switchRowDisplayShow(focusChilrd.row, true)
            focusChilrd = {}
          }
        }, true)
      },
      unbind: function (el) {
      }
    },
    chilrd: {
      inserted: function (el, bingding) {
        const { arg, value: { scope, order }, modifiers: { last = false } } = bingding
        const parent = scope.store.table
        const getInput = (el) => {
          if (el.tagName === 'INPUT') {
            return el
          } else {
            for (let childNode of el.childNodes) {
              if (childNode.tagName === 'INPUT') {
                return childNode
              }
            }
          }
        }
        let chilrd = {
          el,
          row: scope.$index,
          column: order,
          type: arg,
          isDefaultShow: arg === TYPE_DISPLAY,
          isDefaultHidden: arg === TYPE_OCCULTATION,
          isShow: arg === TYPE_DISPLAY,
          isLastColumn: last,
          input: getInput(el)
        }
        if (chilrd.isDefaultHidden) {
          el.style.display = 'none'
        }
        const displayClick = new CustomEvent(EVENT_DISPLAY_CLICK, {
          detail: {
            chilrd
          }
        })
        if (chilrd.isDefaultShow) {
          el.addEventListener('click', ev => {
            parent.$el.dispatchEvent(displayClick)
          })
        }
        const occultationFocus = new CustomEvent(EVENT_OCCULTATION_FOCUS, {
          detail: {
            chilrd
          }
        })
        if (chilrd.isDefaultHidden && chilrd.input) {
          chilrd.input.addEventListener('focus', ev => {
            parent.$el.dispatchEvent(occultationFocus)
          })
        }
        const chilrdInserted = new CustomEvent(EVENT_INSERTED, {
          detail: {
            chilrd
          }
        })
        parent.$el.dispatchEvent(chilrdInserted)
      },
      unbind: function (el) {
      }
    },
    chilrdDefaultHidden: {
      inserted: function (el, bingding, vnode) {
        const { scope, order } = bingding.value
        el.className += ` row_${scope.$index}`
        el.style.display = 'none'
        el.dataset.row = scope.$index
        el.dataset.column = order
        el.dataset.isDefaultHidden = true
      }
    },
    chilrdDefaultShow: {
      inserted: function (el, bingding, vnode) {
        const { scope, order } = bingding.value
        el.className += ` row_${scope.$index}`
        el.dataset.row = scope.$index
        el.dataset.column = order
        el.dataset.isDefaultShow = true
        const custom = new CustomEvent('swich-row-show', {
          detail: {
            row: scope.$index,
            showDefaultHidden: true
          }
        })
        el.addEventListener('click', ev => {
          scope.store.table.$el.dispatchEvent(custom)
        })
      }
    }
  },
  methods: {
    hiddenOccultationCallback (row) {
    },
    onTab (ev) {
      console.log('onTab', ev)
    },
    click (row) {
      console.log(row)
    },
    click2 (scope) {
      console.log(scope)
    }
  },
  data () {
    return {
      tableData: [{
        date: '2016-05-02',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        date: '2016-05-04',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1517 弄'
      }, {
        date: '2016-05-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1519 弄'
      }, {
        date: '2016-05-03',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1516 弄'
      }]
    }
  }
}
</script>

<style>
p {
  margin: 0;
}
</style>
