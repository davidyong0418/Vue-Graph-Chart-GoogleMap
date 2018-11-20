<template>
<div class="app">

  <div class="graph">
    <apexcharts width="400" type="bar" :options="options" :series="series"></apexcharts>
  </div>

  <div class="map">
    <div :style="{opacity: opacity}"><slot/></div>
  </div>

</div>
</template>
<script>
import Vue from 'vue'
import * as VueGoogleMaps from 'vue2-google-maps'
import VueApexCharts from 'vue-apexcharts'
Vue.use(VueApexCharts);
export default {
  mixins: [VueGoogleMaps.MapElementMixin],
  props: {
    delayRepaint: {
      type: Number,
      default: undefined
    },
    marker: {
      type: Object,
      default: undefined
    },
    offsetX: {
      type: Number,
      default: 0
    },
    offsetY: {
      type: Number,
      default: 0
    },
    alignment: {
      type: String,
      default: "top"
    },
    options: {
        chart: {
            id: 'coinchart'
        },
        yaxis: {
            seriesName: undefined,
            opposite: false,
            tickAmount: 2,
        },
        xaxis: {
            categories: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ,11, 12],
            labels:{
                show: false
            }
        },
        dataLabels: {
            enabled: false
        },
    },
    fill : { color: "#aada2b"},
  },
  data () {
    return {
      opacity: 0.01
    }
  },
  watch: {
    marker (val) {
      this.$mapPromise.then(map => this.$overlay.setPosition())
    },
  },
  provide () {
    const self = this
    this.$mapPromise.then(map => {
      class Overlay extends google.maps.OverlayView {
        constructor (map) {
          super()
          this.setMap(map)
          this.draw = () => this.repaint()
          this.setPosition = () => this.repaint()
        }
        repaint () {
          const div = self.$el
          const projection = this.getProjection()
          if (projection && div) {
            const posPixel = projection.fromLatLngToDivPixel(self.position)
            let x, y
            switch (self.alignment) {
              case "top":
                x = posPixel.x - (div.offsetWidth / 2)
                y = posPixel.y - div.offsetHeight
                break
              case "bottom":
                x = posPixel.x - (div.offsetWidth / 2)
                y = posPixel.y
                break
              case "left":
                x = posPixel.x - div.offsetWidth
                y = posPixel.y - (div.offsetHeight / 2)
                break
              case "right":
                x = posPixel.x
                y = posPixel.y - (div.offsetHeight / 2)
                break
              case "center":
                x = posPixel.x - (div.offsetWidth / 2)
                y = posPixel.y - (div.offsetHeight / 2)
                break
              case "topleft":
              case "lefttop":
                x = posPixel.x - div.offsetWidth
                y = posPixel.y - div.offsetHeight
                break
              case "topright":
              case "righttop":
                x = posPixel.x
                y = posPixel.y - div.offsetHeight
                break
              case "bottomleft":
              case "leftop":
                x = posPixel.x - div.offsetWidth
                y = posPixel.y
                break
              case "bottomright":
              case "rightbottom":
                x = posPixel.x
                y = posPixel.y
                break
              default:
                throw new Error("Invalid alignment type of custom marker!")
                break
              }
            div.style.left = (x + self.offsetX) + 'px'
            div.style.top = (y + self.offsetY) + 'px'
          }
        }
        onAdd () {
          const div = self.$el
          const panes = this.getPanes()
          div.style.position = 'absolute'
          div.style.display = 'inline-block'
          div.style.zIndex = 50
          panes.overlayLayer.appendChild(div)
          panes.overlayMouseTarget.appendChild(div)
        }
        onRemove () {
          self.$el.remove()
        }
      }
      this.$overlay = new Overlay(map)
      if (this.delayRepaint) {
        setTimeout(() => {
          this.$overlay.repaint()
          this.opacity = 1
        }, this.delayRepaint)
      } else {
        this.opacity = 1
      }
    })
  },
  computed: {
    lat () {
      return parseFloat(this.marker.lat || this.marker.latitude)
    },
    lng () {
      return parseFloat(this.marker.lng || this.marker.longitude)
    },
    position () {
      const self = this
      return {
        lat () {
          return self.lat
        },
        lng () {
          return self.lng
        }
      }
    }
  },
  destroyed () {
    this.$overlay.setMap(null)
    this.$overlay = undefined
  }
}
</script>
