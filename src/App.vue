<template lang="pug">
div(class="stripe" id="stripe" :style="gotResult ? {backgroundColor: '#343BAA'} : {backgroundColor: '#5858B9'} ")
  div(class="stripe-wellDone" v-if="gotResult")
    h1 WELL
    h1 DONE!
  div(class="stripe-result" v-if="gotResult")
    p YOU GOT...
    p # {{ outputPrize.name }}!
svg(id="wheel" class="wheel" width="560" height="560")
img(id="wheel-outside" :src="require('@/assets/svgs/wheelOutside.svg')")
div(id="wheel-btn")
  p PRESS
img(id="wheel-pointer" :src="require('@/assets/svgs/wheelPointer.svg')")
</template>

<script>
import * as d3 from 'd3';
import { onMounted, ref } from 'vue';

export default {
  name: 'App',
  components: {
  },
  setup() {
    const prizeTotal = ref(0);
    const outputPrize = ref(null);
    const data = ref([]);
    const gotResult = ref(false);
    let isSpinng = false;
    data.value.length = 20;

    for (let i = 0; i < data.value.length; i += 1) {
      data.value[i] = {};
    }

    for (let i = 0; i < 20; i += 1) {
      data.value[i].name = i + 1;
      data.value[i].value = 1;
      if (i + 1 === 20) {
        data.value[i].total = 69;
      } else if (i + 1 === 19) {
        data.value[i].total = 20;
      } else if (i + 1 === 18) {
        data.value[i].total = 10;
      } else if (i + i === 17) {
        data.value[i].total = 5;
      } else if (i + 1 >= 2 && i + 1 <= 16) {
        data.value[i].total = 1;
      } else if (i + 1 === 1) {
        data.value[i].total = 1;
      }
    }

    onMounted(() => {
      const wheel = d3.select('#wheel');
      const wheelRadius = 250;
      const width = wheel.attr('width');
      const height = wheel.attr('height');
      const textLabel = d3.arc().outerRadius(wheelRadius).innerRadius(wheelRadius - 160);
      const totalLabel = d3.arc().outerRadius(wheelRadius).innerRadius(wheelRadius - 200);
      const textRotateOffset = (360 / data.value.length) / 2;
      let rotateOffset = -((360 / data.value.length) / 2);

      const g = wheel.append('g')
        .attr('id', 'g-wrapper')
        .attr('transform', `translate(${width / 2}, ${height / 2})`);
      const pie = d3.pie().value((d) => d.value);

      const arcPath = d3.arc()
        .outerRadius(wheelRadius)
        .innerRadius(0);
      const pies = g.selectAll('.arc')
        .data(pie(data.value))
        .enter()
        .append('g')
        .attr('class', (dataObject) => `arc prize-${dataObject.data.name}`);

      const offsetTheWheelRotate = () => {
        wheel.node().style.transform = `translate(-50%, -50%) rotate(${rotateOffset}deg)`;
      };

      const drawPies = () => {
        pies.selectAll('.piePath').remove();

        pies.append('path')
          .attr('d', arcPath)
          .attr('stroke', '#1F1579')
          .attr('stroke-width', '2px')
          .attr('class', 'piePath')
          .attr('fill', (dataObject, index) => {
            const isEven = index % 2 === 0;
            if (isEven) return '#343BAA';
            return '#F0BEFF';
          });
      };

      const drawPrizeTitle = () => {
        pies.selectAll('.text-title').remove();

        pies.append('text')
          .attr('transform', (dataObject, index) => {
            const halfAngleOfEachPie = (dataObject.endAngle * 57.32) / 2; // convert radian to angle
            const titleRotateOffset = textRotateOffset;
            const titleRotateAngle = halfAngleOfEachPie + titleRotateOffset * index;
            return `translate(${textLabel.centroid(dataObject)}) rotate(${titleRotateAngle})`;
          })
          .attr('text-anchor', 'middle')
          .attr('font-family', 'Roboto Condensed')
          .attr('font-wieght', 'bolder')
          .attr('font-size', '24')
          .attr('class', 'text-title')
          .attr('fill', (dataObject, index) => {
            const isEven = index % 2 === 0;
            if (isEven) return '#F0BEFF';
            return '#343BAA';
          })
          .text((dataObject) => `#${dataObject.data.name}`);
      };

      const drawPrizeRemainTotal = () => {
        pies.selectAll('.text-total').remove();

        pies.append('text')
          .attr('transform', (dataObject, index) => {
            const halfAngleOfEachPie = (dataObject.endAngle * 57.32) / 2; // convert radian to angle
            const titleRotateOffset = textRotateOffset;
            const titleRotateAngle = halfAngleOfEachPie + titleRotateOffset * index;
            return `translate(${totalLabel.centroid(dataObject)}) rotate(${titleRotateAngle})`;
          })
          .attr('text-anchor', 'middle')
          .attr('font-family', 'Roboto Condensed')
          .attr('font-wieght', 'bolder')
          .attr('font-size', '16')
          .attr('class', 'text-total')
          .attr('fill', (dataObject, index) => {
            const isEven = index % 2 === 0;
            if (isEven) return '#F0BEFF';
            return '#343BAA';
          })
          .text((dataObject) => dataObject.data.total);
      };

      const initWheel = () => {
        offsetTheWheelRotate();
        drawPies();
        drawPrizeTitle();
        drawPrizeRemainTotal();
      };

      const activePie = (className) => {
        const activePies = d3.selectAll(`.prize-${className}`);
        activePies.selectAll('path')
          .attr('fill', '#FF00BA');
        activePies.selectAll('text')
          .attr('fill', '#fff');
      };

      const removeAllRandomIcon = () => {
        d3.select('#stripe')
          .selectAll('text')
          .remove();
      };

      const appendRadomIcon = (prize) => {
        const stripe = d3.select('#stripe');
        const stripeHeight = stripe.node().clientHeight;
        const interval = 130;
        let lasPosition = -100;

        removeAllRandomIcon();

        for (let i = 0; i < 30; i += 1) {
          const isEven = i % 2 === 0;
          stripe.append('text')
            .style('left', `${lasPosition + interval}px`)
            .style('top', () => `${Math.random() * (stripeHeight - 20)}px`)
            .style('transform', () => {
              if (isEven) return 'rotate(20deg)';
              return 'rotate(-20deg)';
            })
            .attr('class', 'randomIcon')
            .text(() => prize.icon);
          lasPosition += interval;
        }
      };

      const updatePrizeTotal = () => {
        prizeTotal.value = 0;
        data.value.forEach(({ total }) => {
          prizeTotal.value += total;
        });
      };

      const getAngleList = () => {
        const angleList = [];
        const startAngle = -((360 / data.value.length) / 2);
        const eachPieAngle = 360 / data.value.length;

        data.value.forEach((p, index) => {
          if (startAngle + (index * eachPieAngle) < 0) {
            angleList.push(360);
          } else {
            angleList.push((startAngle + (index * eachPieAngle) + 360 + (-startAngle)));
          }
        });

        return angleList;
      };

      let lastPrizeIndex = null;
      const getPrize = () => {
        if (prizeTotal.value <= 0) return false;
        if (isSpinng) return false;

        const prize = data.value[Math.floor(Math.random() * data.value.length)];
        const prizeIndex = data.value.findIndex((p) => p.name === prize.name);
        const angleList = getAngleList();
        initWheel();
        removeAllRandomIcon();
        gotResult.value = false;
        if (prize.total === 0) return getPrize();
        isSpinng = true;
        prize.total -= 1;

        setTimeout(() => {
          gotResult.value = true;
          outputPrize.value = prize;
          appendRadomIcon(prize);
          initWheel();
          activePie(prize.name);
          isSpinng = false;
        }, 10000);

        if (lastPrizeIndex) {
          rotateOffset -= (angleList[prizeIndex] - angleList[lastPrizeIndex]) - 3600;
          lastPrizeIndex = prizeIndex;
        } else {
          rotateOffset -= angleList[prizeIndex] - 3600;
          lastPrizeIndex = prizeIndex;
        }

        wheel.node().style.transform = `translate(-50%, -50%) rotate(${rotateOffset}deg)`;
        updatePrizeTotal();
        return true;
      };

      document.getElementById('wheel-btn').addEventListener('click', () => {
        getPrize();
      });

      initWheel();
      updatePrizeTotal();
    });

    return {
      gotResult,
      outputPrize,
    };
  },
};
</script>

<style lang="scss">
$primary: #1F1172;
$secondary: #5858B9;
$third: #343BAA;
$alert: #FF00BA;
$light: #F0BEFF;
$white: #fff;

html {
  background-color: $secondary;
  overflow: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.stripe {
  overflow: hidden;
    display: flex;
    position: absolute;
    padding: 0 160px;
    width: 100%;
    height: 240px;
    top: 50%;
    justify-content: space-between;
    align-items: center;
    font-family: 'Roboto Condensed', 'Material Icons', Arial, Helvetica, sans-serif;
    font-weight: bold;
    text-align: left;
    color: #fff;
    background-color: $third;
    transform: translateY(-50%);
    user-select: none;
    .randomIcon {
      position: absolute;
      font-family: 'Material Icons', Arial, Helvetica, sans-serif;
      font-size: 48px;
      color: $secondary;
      z-index: -1;
    }
}

.stripe-wellDone {
  font-size: 72px;
}

.stripe-result {
  p:first-of-type {
    font-size: 32px;
  }
  p:last-of-type {
    font-size: 72px;
    text-decoration: underline;
    color: $alert;
  }
}

.wheel {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  transition: all 10s ease-in-out;
}

#wheel-outside {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

#wheel-pointer {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-46%, -66%) rotate(190deg);
}

#wheel-btn {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 104px;
  height: 104px;
  font-weight: 800;
  font-size: 32px;
  font-family: 'Roboto Condensed', 'Material Icons', Arial, Helvetica, sans-serif;
  color: $alert;
  background-color: $primary;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  z-index: 9999;
  user-select: none;
  cursor: pointer;
}
</style>
