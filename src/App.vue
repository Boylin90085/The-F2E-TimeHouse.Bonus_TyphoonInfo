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
              el-select(v-model="areaValue" placeholder="請選擇")
                el-option(
                  v-for="item in areaOptions"
                  :key="item.value"
                  :label="item.locale"
                  :value="item.value"
                )
            el-col.filter-box(:span="4")
              h4 災害類型:
              el-select(v-model="pNameValue" placeholder="請選擇")
                el-option(
                  v-for="item in pNameOptions"
                  :key="item.value"
                  :label="item.pName"
                  :value="item.value"
                )
            el-col.filter-box(:span="4")
              h4 是否處理完成:
              el-select(v-model="CaseCompleteValue" placeholder="請選擇")
                el-option(
                  v-for="item in CaseCompleteOptions"
                  :key="item.value"
                  :label="item.CaseComplete"
                  :value="item.value"
                )
          el-row
            el-col.info-total(:span="24")
              p 共有
                span {{ total }}
                |筆資料
          el-row.typhoon-main(type="flex" justify="space-between" v-loading="loading")
            el-col(:span="11" v-for="tyData in filterData[currentPage]"  :key="tyData.CaseID")
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
            page-size="10"
            :total="filterData.length*10"
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
      areaOptions: [
        {locale: '中正區', value: '1'},
        {locale: '大同區', value: '2'},
        {locale: '中山區', value: '3'},
        {locale: '松山區', value: '4'},
        {locale: '大安區', value: '5'},
        {locale: '萬華區', value: '6'},
        {locale: '信義區', value: '7'},
        {locale: '士林區', value: '8'},
        {locale: '北投區', value: '9'},
        {locale: '內湖區', value: '10'},
        {locale: '南港區', value: '11'}],
      areaValue: '',
      pNameOptions: [
        {pName: '民生、基礎設施災情', value: '1'},
        {pName: '積淹水災情', value: '2'},
        {pName: '道路、隧道災情', value: '3'},
        {pName: '路樹災情', value: '4'},
        {pName: '土石災情', value: '5'},
        {pName: '環境汙染', value: '6'},
        {pName: '橋梁災情', value: '7'},
        {pName: '車輛及交通事故', value: '8'},
        {pName: '廣告招牌災情', value: '9'},
        {pName: '其他災情', value: '10'}
      ],
      pNameValue: '',
      CaseCompleteOptions: [
        {CaseComplete: '已經處理完成', value: '1'},
        {CaseComplete: '尚未處理完成', value: '2'}
      ],
      CaseCompleteValue: '',
      currentPage: 0,
      loading: true,
      typhoonData: [],
      total: ''
    }
  },
  created () {
    this.getTyphoonData()
  },
  methods: {
    getTyphoonData () {
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
        this.loading = false
        res.data.data.getDisasterSummary.items.forEach((item) => {
          item.CaseTime = moment(item.CaseTime).locale('zh-tw').format('LLL')
        })
        this.typhoonData = res.data.data.getDisasterSummary.items
        this.total = res.data.data.getDisasterSummary.total
        console.log(res)
      })
    },
    currentChange (index) {
      const vm = this
      // 直接把當前頁面丟進去
      vm.currentPage = index - 1
    }
  },
  computed: {
    filterData () {
      const vm = this
      const newData = []
      vm.typhoonData.forEach((item, i) => {
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
