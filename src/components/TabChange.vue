<template>
  <div>
    <el-table v-parent
      :data="tableData"
      style="width: 100%">
      <el-table-column prop="date"
        label="日期"
        width="180">
        <template scope="scope">
          <span @click="click(scope)"
            v-chilrd-default-show="{scope, order: 1}">{{scope.row.date}}</span>
          <el-input v-chilrd-default-hidden="{scope, order: 1}"
            v-model="scope.row.date"
            placeholder=""></el-input>
        </template>
      </el-table-column>
      <el-table-column prop="name"
        label="姓名"
        width="180">
        <template scope="scope">
          <span @click="click2(scope)"
            v-chilrd-default-show="{scope, order: 2}">{{scope.row.name}}</span>
          <el-input v-chilrd-default-hidden="{scope, order: 2}"
            v-model="scope.row.name"
            placeholder=""></el-input>
        </template>
      </el-table-column>
      <el-table-column prop="address"
        label="地址">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  directives: {
    parent: {
      inserted: function (el, binding, vnode) {
        // let currentRow = -1
        el.addEventListener('swich-row-show', ev => {
          const { row, showDefaultHidden = false, showDefaultShow = false } = ev.detail
          const rowElement = document.getElementsByClassName(`row_${row}`)
          // currentRow = row
          for (let element of rowElement) {
            let { dataset: { isDefaultHidden = false, isDefaultShow = false } } = element
            isDefaultHidden = Boolean(isDefaultHidden).valueOf()
            isDefaultShow = Boolean(isDefaultShow).valueOf()
            if (showDefaultShow && isDefaultShow) {
              element.style = {}
            } else {
              element.style.display = 'none'
            }
            if (showDefaultHidden && isDefaultHidden) {
              element.style = {}
            } else {
              element.style.display = 'none'
            }
          }
        })
      }
    },
    parent2: {
      inserted: function (el, bingding) {
        const chilrdList = []
        el.addEventListener('chilrd-inserted', ev => {
          const { chilrd } = ev.details
          chilrdList[chilrd.row][chilrd.column]
        })

      }
    },
    chilrd: {
      inserted: function (el, bingding) {
        const { scope, order, defaultShow } = bingding.value
        const chilrdInserted = new CustomEvent('chilrd-inserted', {
          details: {
            chilrd: {
              el,
              row: scope.$index,
              column: order,
              defaultShow
            }
          }
        })
        scope.store.table.$el.dispatchEvent(chilrdInserted)
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

</style>
