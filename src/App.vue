<template lang="pug">
  #app
    el-container
      el-header
        img(src="https://d30y9cdsu7xlg0.cloudfront.net/png/1258372-200.png", alt="")
        h1 台北市颱風災害動態資訊
      el-main
        .container
          el-row(type="flex" justify="center")
            el-col.filter-box(:span="4")
              h4 受災地區:
              el-select(v-model="areaValue" placeholder="請選擇" clearable @change="filterData('area')")
                el-option(
                  v-for="(item, index) in areaOptions"
                  :key="index"
                  :label="item"
                  :value="item"
                )
            el-col.filter-box(:span="4")
              h4 災害類型:
              el-select(v-model="pNameValue" placeholder="請選擇" clearable @change="filterData('pName')")
                el-option(
                  v-for="(item, index) in pNameOptions"
                  :key="index"
                  :label="item"
                  :value="item"
                )
            el-col.filter-box(:span="4")
              h4 是否處理完成:
              el-select(v-model="CaseCompleteValue" placeholder="請選擇" clearable @change="filterData('CaseComplete')")
                el-option(
                  v-for="(item, index) in CaseCompleteOptions"
                  :key="index"
                  :label="item.CaseComplete"
                  :value="item.value"
                )
          el-row
            el-col.info-total(:span="24")
              p 共有
                span {{ total }}
                |筆資料
          el-row.typhoon-main(type="flex" justify="space-between" v-loading="loading")
            el-col(:span="11" v-for="tyData in pageData[currentPage]"  :key="tyData.CaseID")
              el-card.box-card
                .typhoon-name(slot='header')
                  span {{ tyData.DPName }}
                  .checkPoint(:class="{OK: tyData.CaseComplete, notOK: !tyData.CaseComplete}")
                .typhoon-info
                  .info-item.time
                    span.title 發生時間:
                    span.content {{ tyData.CaseTime }}
                  .info-item.area
                    span.title 受災地區:
                    span.content {{ tyData.CaseLocationDistrict }}
                  .info-item.type
                    span.title 災害類型:
                    span.content {{ tyData.PName }}
                  .info-item.address
                    span.title 詳細地址:
                    span.content {{ tyData.CaseLocationDescription }}
                  .info-item.detail
                    span.title 災情描述:
                    span.content {{ tyData.CaseDescription }}
          el-pagination(
            background
            layout="prev, pager, next"
            :page-size="10"
            :total="pageData.length*10"
            @current-change="currentChange"
            )

      el-footer
        h5 台北市颱風災害動態資訊
        h5 資料來源：臺北市政府 消防局EOC 開放資料專區

</template>

<script>
import moment from 'moment'
export default {
  name: 'App',
  data () {
    return {
      areaOptions: ['中正區', '大同區', '中山區', '松山區', '大安區', '萬華區', '信義區', '士林區', '北投區', '內湖區', '南港區'],
      areaValue: '',
      pNameOptions: ['民生、基礎設施災情', '積淹水災情', '道路、隧道災情', '路樹災情', '土石災情', '環境汙染', '橋梁災情', '車輛及交通事故', '廣告招牌災情', '其他災情'],
      pNameValue: '',
      CaseCompleteOptions: [
        {CaseComplete: '已經處理完成', value: true},
        {CaseComplete: '尚未處理完成', value: false}
      ],
      CaseCompleteValue: '',
      currentPage: 0,
      loading: true,
      typhoonData: [],
      tempData: [],
      total: ''
    }
  },
  created () {
    this.getTyphoonData()
  },
  methods: {
    getTyphoonData () {
      const vm = this
      this.axios.post('https://typhoonapi.yosgo.com/graphql', {
        query: `
        {
          getDisasterSummary {  
          items {
            DPID
            DPName
            CaseID
            CaseTime
            PName
            CaseLocationDistrict
            CaseLocationDescription
            CaseDescription
            CaseComplete
          }
          total
        }
      }`
      }).then((res) => {
        vm.loading = false
        res.data.data.getDisasterSummary.items.forEach((item) => {
          item.CaseTime = moment(item.CaseTime).locale('zh-tw').format('LLL')
        })
        vm.typhoonData = res.data.data.getDisasterSummary.items
        vm.tempData = vm.typhoonData
        vm.total = res.data.data.getDisasterSummary.total
        console.log(res)
      })
    },
    currentChange (index) {
      const vm = this
      // 直接把當前頁面丟進去
      vm.currentPage = index - 1
    },
    filterData (type) {
      const vm = this
      let filterValue = ''
      let filterType = ''

      switch (type) {
        case 'area':
          filterValue = vm.areaValue
          filterType = 'CaseLocationDistrict'

          vm.tempData = vm.typhoonData.filter((item) => {
            return item[filterType].match(filterValue)
          })

          // 處理另外兩個選項
          if (vm.pNameValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.PName.match(vm.pNameValue)
            })
          }

          if (vm.CaseCompleteValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.CaseComplete === vm.CaseCompleteValue
            })
          }
          break
        case 'pName':
          filterValue = vm.pNameValue
          filterType = 'PName'

          vm.tempData = vm.typhoonData.filter((item) => {
            return item[filterType].match(filterValue)
          })

          // 處理另外兩個選項
          if (vm.areaValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.CaseLocationDistrict.match(vm.areaValue)
            })
          }

          if (vm.CaseCompleteValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.CaseComplete === vm.CaseCompleteValue
            })
          }
          break
        case 'CaseComplete':
          filterValue = vm.CaseCompleteValue
          filterType = 'CaseComplete'

          // 處理清空後變成空字串就全部顯示
          vm.tempData = vm.typhoonData.filter((item) => {
            if (typeof filterValue !== 'string') {
              return item[filterType] === filterValue
            } else {
              return true
            }
          })

          // 處理另外兩個選項
          if (vm.areaValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.CaseLocationDistrict.match(vm.areaValue)
            })
          }

          if (vm.pNameValue !== '') {
            vm.tempData = vm.tempData.filter((item) => {
              return item.PName.match(vm.pNameValue)
            })
          }
          break
      }
    }
  },
  computed: {
    pageData () {
      const vm = this
      const newData = []
      vm.tempData.forEach((item, i) => {
        // 每10筆推一個空陣列
        if (i % 10 === 0) {
          newData.push([])
        }
        // 控制頁數
        const page = parseInt(i / 10)
        newData[page].push(item)
      })
      console.log(newData)
      return newData
    }
  }
}
</script>

<style lang="sass">
  @import '/assets/style/main.sass'
  .el-container
    .el-header,.el-footer
      display: flex
      justify-content: center
      align-items: center
      background-color: $color_header
      color: white
      height: 250px
      img
        width: 50px
        filter: invert(1)
    .el-footer
      flex-direction: column
      h5:first-child
        margin-bottom: 5px
      h5
        margin: 0
    .el-main
      display: flex
      justify-content: center
      .container
        max-width: 1170px
        width: 80%
        .el-row:not(:last-child)
          margin-bottom: 20px
        .el-row
          flex-wrap: wrap
        .filter-box:not(:last-child)
          border-right: 1px solid #eee
        .filter-box
          background-color: #fff
          padding: 0px 20px
          padding-bottom: 10px
          .el-input__inner
            border: none
            padding-left: 0
        .info-total
          letter-spacing: 1.2px
          span
            color: $color_header
        .typhoon-main
          .el-col:not(:nth-child(-1n+3))
            margin-top: 20px
          .el-card
            min-height: 270px
            .typhoon-name
              display: flex
              align-items: center
              justify-content: space-between
              .checkPoint
                width: 12px
                height: 12px
                border-radius: 50%
                &.OK
                  background-color: $color_OK
                &.notOK
                  background-color: $color_notOK
            .typhoon-info
              .info-item:not(:last-child)
                margin-bottom: 10px
              .info-item
                display: flex
                .title
                  flex: 2
                  color: #ACB2BA
                .content
                  flex: 10
              .area
                .content
                  color: $color_OK
              .type
                .content
                  color: $color_blue
        .el-pagination
          display: flex
          justify-content: center
</style>
