<template>
  <div id="app">
    <div class="columns is-multiline is-mobile">
      <div class="column is-6">
        <line-chart :data="temp"/>
      </div>
       <div class="column is-6">
        <line-chart :data="cpu"/>
      </div>
      <div class="column is-6">
        <line-chart :data="memory"/>
      </div>
      <div class="column is-6">
        <line-chart :data="inB"/>
      </div>
      <div class="column is-6">
        <line-chart :data="out"/>
      </div>
    </div>
    <div class="columns is-multiline is-mobile">
      <div class="column is-3" style="border: 1px solid;"  v-for="int in interfaces" :key="int.index">
        <interface @change-interface="chageInterface" :data="int"/>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
/* global io */
import moment from 'moment'
import LineChart from './components/LineChart.vue'
import Interface from './components/Interface'
export default {
  name: 'App',
  components: {
    LineChart,
    Interface
  },
  data () {
    return {
      socket: '',
      devices: [],
      interfaces: []
    }
  },
  mounted () {
    this.socket = io.connect()
    this.socket.on('init device', (data) => {
      console.log('1', data)
      this.devices = data
    })
    this.socket.on('update device', (data) => {
      console.log('2', data)
      this.devices.push(data)
    })
    this.socket.on('all interface', (data) => {
      console.log('3', data)
      this.interfaces = data
    })
    this.socket.on('update interface', (data) => {
      console.log('3', data)
      let int = this.interfaces.find(int => int.index === data.index)
      if (int) {
        int.status = data.status
      }
    })
    this.socket.on('update interface link status', (data) => {
      console.log('4', data)
      let int = this.interfaces.find(int => int.index === data.index.toString())
      if (int) {
        int.linkStatus = data.linkStatus
        this.linkStatus(int)
      }
    })
  },
  methods: {
    chageInterface (index, value) {
      this.socket.emit('change interface', {index, value})
    },
    linkStatus(int) {
      const message = `LINK ${int.description}  status ${int.linkStatus === 1 ? 'UP' : 'DOWN'}`
      this.$snackbar.open({
        message,
        type: 'is-warning',
        position: 'is-top',
        actionText: null,
        queue: false
      })
    }
  },
  computed: {
    labels () {
      return this.devices.map(device => {
        return moment(device.time).format('HH:mm')
      })
    },
    temp () {
      return {
        labels: this.labels,
        datasets: [
          {
            label: 'temp C',
            borderColor: '#f87979',
            data: this.devices.map(device => device.temp)
          }
        ]
      }
    },
    cpu () {
      return {
        labels: this.labels,
        datasets: [
          {
            label: 'cpu %',
            borderColor: '#f87979',
            data: this.devices.map(device => device.cpu)
          }
        ]
      }
    },
    memory () {
      return {
        labels: this.labels,
        datasets: [
          {
            label: 'memory %',
            borderColor: '#f87979',
            data: this.devices.map(device => device.memory)
          }
        ]
      }
    },
    inB () {
      return {
        labels: this.labels,
        datasets: [
          {
            label: 'in %',
            borderColor: '#f87979',
            data: this.devices.map(device => device.in)
          }
        ]
      }
    },
    out () {
      return {
        labels: this.labels,
        datasets: [
          {
            label: 'out %',
            borderColor: '#f87979',
            data: this.devices.map(device => device.out)
          }
        ]
      }
    }
  }
}
</script>

<style>
</style>
