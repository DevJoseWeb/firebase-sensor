<template>
  <div>
    <div class="row">
        <div class="col-lg-12">
          <button class="btn btn-default pull-right" v-on:click="loadData()"><span class="glyphicon glyphicon-refresh"></span></button>
        </div>
    </div>
    <div class="row" v-for="category in categories" :key="category.id">
      <div class="col-lg-12">
        <chart-detail v-if="isReadyChart" :rooms="rooms" :category="category" :readings="data" :numberOfHours="numberOfHours"></chart-detail>
      </div>
    </div>
  </div>
</template>

<script>
import ChartDetail from '@/components/home/chart-detail'

export default {
  name: 'chart-summary',
  data () {
    return {
      data: {},
      chartReadyCount: 0,
      numberOfHours: 24
    }
  },
  props: ['rooms', 'categories'],
  computed: {
    isReadyChart () {
      return this.chartReadyCount === Object.keys(this.rooms).length
    }
  },
  methods: {
    loadData () {
      this.data = {}
      this.chartReadyCount = 0
      var lowerTimeLimit = (new Date() / 1000) - 3600 * this.numberOfHours
      Object.keys(this.rooms).forEach((roomId) => {
        this.DevLog('load readings data for room:', roomId)
        this.$firebase.database().ref('readings/' + roomId).limitToLast(4 * this.numberOfHours).once('value', (newValue) => {
          var basicArray = this.ObjectToArray(newValue.val())

          // Remove too old readings
          basicArray = basicArray.filter((item) => {
            return item.time > lowerTimeLimit
          })
          this.data[roomId] = basicArray
          this.chartReadyCount++
        })
      })
    }
  },
  created () {
    this.loadData()
  },
  components: { ChartDetail: ChartDetail }
}
</script>
