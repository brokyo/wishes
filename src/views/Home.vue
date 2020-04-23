<template>
  <div>
    <span @click="fullscreen">awakening.systems</span>
  </div>
</template>

<script>
const p5 = require("p5");
var W3CWebSocket = require("websocket").w3cwebsocket;
var client = new W3CWebSocket(
  "ws://ec2-35-166-31-113.us-west-2.compute.amazonaws.com:8080/",
  "echo-protocol"
);

class Signal {
  constructor(request, noiseVal) {
    this.show = true;
    this.x = 0;
    this.y = 10;
    this.d = 20;
    this.color = noiseVal * 255;
  }

  diameterChange(noiseVal) {
    this.d += noiseVal / 1.4;
  }

  display(sketch) {
    sketch.fill(this.color);
    sketch.circle(this.x, this.y, this.d);
  }
}

export default {
  data() {
    return {
      xOff: 0,
      yOff: 0,
      signalId: 0,
      signals: [],
      p5canvas: {},
    };
  },
  methods: {
    fullscreen() {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen();
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        }
      }
    },
    addSignal(request, noise) {
      let signal = new Signal(request, noise);

      this.signals.push(signal);
      setTimeout(this.removeSignal, 45000, signal);
    },
    removeSignal(signal) {
      signal.show = false;
      // signals.splice(signals.findIndex(function(i) {
      //   return i.id === id;
      // }), 1);
    },
    initWS() {
      let vue = this;
      client.onerror = function() {
        console.log("Connection Error");
      };

      client.onopen = function() {
        console.log("WebSocket Client Connected");
        // client.send('foo')
      };

      client.onclose = function() {
        console.log("echo-protocol Client Closed");
      };

      client.onmessage = function(e) {
        let response = JSON.parse(e.data);
        vue.addSignal(response);
        console.table(response);
      };
    },
    initp5() {
      // function addSignal(request) {
      // }

      const s = (sketch) => {
        let rotation = 0;

        sketch.setup = () => {
          sketch.createCanvas(sketch.windowWidth, sketch.windowHeight);
          sketch.frameRate(30);
          sketch.angleMode(sketch.DEGREES);
        };

        sketch.draw = () => {
          let angle = 360 / signals.length;
          let bg = sketch.noise(this.yOff) * 255;
          sketch.background(bg);

          sketch.fill(255, 255, 255, 1);
          sketch.translate(sketch.width / 2, sketch.height / 2);

          sketch.rotate(rotation++);
          this.signals.forEach((signal, index) => {
            // if(signal.show) {
            sketch.push();
            sketch.rotate(angle * index);
            signal.display(sketch);
            signal.y += 0.0001;
            signals[index].diameterChange(sketch.noise(this.xOff));
            sketch.pop();
            // }
          });
          this.xOff += 0.001;
          this.yOff += 0.0005;
        };
        sketch.mousePressed = () => {
          sketch.fullscreen();
        };

        sketch.windowResized = () => {
          sketch.resizeCanvas(sketch.windowWidth, sketch.windowHeight);
        };
      };

      this.p5canvas = new p5(s);
    },
  },
  mounted() {
    this.initWS();
    this.initp5();
  },
};
</script>

<style lang="sass" scoped>
 html
   overflow: hidden

span
  position: absolute
  bottom: 0
  right: 0
  color: grey
</style>
