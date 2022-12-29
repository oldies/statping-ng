<template>
    <div class="onlinestat">
      <div v-observe-visibility="{callback: visibleChart, once: true}" v-if="!loaded" class="row">
        <div class="col-12 text-center mt-3">
          <font-awesome-icon icon="circle-notch" class="text-dim" size="2x" spin/>
        </div>
      </div>
      <transition name="fade">
        <div v-if="loaded">
        <div class="mt-3" style="margin-top: 0!important;">
            <div class="service_day" v-for="(d, index) in failureData" @mouseover="mouseover(d)" @mouseout="mouseout" :class="{'day-error': d.amount > 0, 'day-success': d.amount === 0}">
                <span v-if="d.amount !== 0" class="d-none d-md-block text-center small"></span>
            </div>
            <div class="percent">
            <small>({{this.service.online_365_days}}%)</small>
            </div>
        </div>
      <div class="daily-failures small text-right text-dim">{{hover_text}}</div>
      </div>
  </transition>
    </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.service_day {
    height: 1em;
    margin-right: 2px;
    border-radius: 2px;
    width: 5px;
    cursor: pointer;
    float:left;
}
.percent {
  float: left;
  width: 4rem;
  margin-left: .5rem;
  line-height: 1em;
}
.onlinestat {
  float: right;
}
.basge {
  line-height: 1em;
}
</style>

<script>
    import Api from '../../API';

const isToday = function (date) {
  const today = new Date();

  if (today.toDateString() === date.toDateString()) {
    return true;
  }

  return false;
}

const getTodaysTotalSeconds = function(){
    let date = new Date();
    return +(date.getHours() * 60 * 60) + (date.getMinutes() * 60) + date.getSeconds();
}

export default {
  name: 'GroupServiceFailures',
  components: {

  },
    data() {
        return {
            failureData: [],
          hover_text: "",
          loaded: false,
          visible: false,
        }
    },
  props: {
      service: {
          type: Object,
          required: true
      }
  },
  computed: {
    service_txt() {
      return this.smallText(this.service)
    }
  },
  mounted () {

    },
    methods: {
      visibleChart(isVisible, entry) {
        if (isVisible && !this.visible) {
          this.visible = true
          this.lastDaysFailures().then(() =>  this.loaded = true)
        }
      },
      mouseout() {
        this.hover_text = ""
      },
    mouseover(e) {
      let txt = `${e.amount} Fehler (${e.probes} Aufrufe, ${e.quote}% online)`
      if (e.amount === 0) {
        txt = `keine Probleme`
      }
      this.hover_text = `${e.date.toLocaleDateString()} - ${txt}`
    },
      async lastDaysFailures() {
        const start = this.beginningOf('day', this.nowSubtract(86400 * 14))
        const end = this.endOf('tomorrow')
        const data = await Api.service_failures_data(this.service.id, this.toUnix(start), this.toUnix(end), "24h", true)
        data.forEach((d) => {
          let date = this.parseISO(d.timeframe)
          let hits = (24*60*60);
          if (isToday(date)) {
            hits = getTodaysTotalSeconds();
          }
          const probes = Math.round(hits / this.service.check_interval);
          const quote = Math.round((1 - (d.amount/probes))*10000)/100;
          this.failureData.push({month: date.getMonth(), day: date.getDate(), date: date, amount: d.amount, probes: probes, quote: quote})
        })
        this.$emit('event_child', {'failureData': this.failureData, 'online': this.service.online})
      }
    }
}
</script>
