<template>
  <div id="app">
    <span class="title">Koronawirus (COVID-19) w Polsce</span>
    <div class="info">
      <span class="zarazenia" v-show="!loading">Zarażenia: <span>{{cases[cases.length -1]}}</span></span>
      <span class="zgony" v-show="!loading">Zgony: <span>{{deaths[deaths.length -1]}}</span></span>
    </div>
      <canvas id="myChart" height="120rem" v-show="!loading"></canvas>
    <span v-show="loading">
      <div class="lds-hourglass"></div>
    </span>
    <span class="author">powered by <a href="https://cholodymedia.pl" target="_blank">cholodymedia</a></span>
  </div>
</template>

<script>
import Chart from 'chart.js'
import axios from 'axios'
import { isBrowser } from 'mobile-device-detect';

export default {
  name: 'App',
  data() {
    return {
      database: [],
      stepSize: null,
      dates: [],
      cases: [],
      deaths: [],
      loading: true
    }
  },
  methods: {
    draw() {
        let ctx = document.getElementById('myChart').getContext('2d');
        new Chart(ctx, {
        type: 'line',
        data: {
          datasets: [{
              label: 'Zarażenia',
              data: this.cases,
              type: 'line',
              borderColor: '#F9813B',
              backgroundColor: 'rgba(249, 129, 59, 0.39)',
              pointBorderWidth: 5,
          }, {
              label: 'Zgony',
              data: this.deaths,
              type: 'line',
              borderColor: '#CA1551',
              backgroundColor: 'rgba(202, 21, 81, 0.616)',
              pointBorderWidth: 5,
          }],
          labels: this.dates
        },
        options: {
          animation: {
            duration: 2000
          },
          legend: {
            labels: {
              fontColor: 'white',
              fontSize: 14,
            }
          },
          scales: {
                yAxes: [{
                  ticks: {
                    fontColor: "white",
                    fontSize: 14,
                    stepSize: this.stepSize,
                  },
                  gridLines: {
                    display: true ,
                    color: "grey"
                  },
                }],
                xAxes: [{
                  ticks: {
                    fontColor: "white",
                    fontSize: 14,
                    stepSize: 1,
                  },
                  gridLines: {
                    display: true ,
                    color: "grey"
                  },
                }]
            }
        }
      });
    },
    getDates() {
      this.database.forEach(element => {
        this.dates.push(element.title);
      });
    },
    getCases() {
      this.database.forEach(element => {
        this.cases.push(element.zarazenia);
      });
    },
    getDeaths() {
      this.database.forEach(element => {
        this.deaths.push(element.zgony);
      });
    }
  },
  mounted() {
    setTimeout(() => {location.reload()}, 60000);
    window.addEventListener("orientationchange", () => {
      if(this.stepSize) {
        this.loading = true;
        location.reload();
      }
    });
    window.addEventListener('resize', () => {
      if(this.stepSize && isBrowser) {
        this.loading = true;
        location.reload();
      }
    });
    axios({
      method: "POST",
      url: "https://graphql.datocms.com",
      headers: {
        Authorization: 'bearer ' + '45b5198426e3cea0281b963f5e8cf7'
      },
      data: {
        query: `
          {
            allInformation {
              title,
              zarazenia,
              zgony
            }
            stepsize {
              number
            }
          }
        `
      }
    }).then(response => {
      this.database = response.data.data.allInformation;
      this.stepSize = response.data.data.stepsize.number;
      this.getDates();
      this.getCases();
      this.getDeaths();
      this.draw();
      this.loading = false;
    });
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Montserrat:500&display=swap');
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Montserrat;
}
#app {
  max-width: 100vw;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #1c242c;

  .title {
    position: fixed;
    color: white;
    font-size: 2rem;
    top: 2rem;
    @media (max-width: 1000px) {
      top: 1rem;
      font-size: 1.2rem;
      position: relative;
    }
  }

  .info {
    width: 100%;
    display: flex;
  }

  .zarazenia {
    position: fixed;
    color: white;
    font-size: 2rem;
    top: 4rem;
    left: 3rem;
    animation: 2s fade-in forwards;
    span {
      color: #F9813B;
    }
    @media (max-width: 1000px) {
      top: 2rem;
      left: 0rem;
      font-size: 1rem;
      position: relative;
      margin-left: 3rem;
    }
  }

  .zgony {
    position: fixed;
    color: white;
    font-size: 2rem;
    top: 4rem;
    right: 3rem;
    animation: 1.5s fade-in forwards;
    span {
      color: #CA1551;
    }
    @media (max-width: 1000px) {
      top: 2rem;
      font-size: 1rem;
      position: relative;
      right: 0rem;
      margin-left: auto;
      margin-right: 3rem;
    }
  }

  .author {
    color: white;
    position: fixed;
    bottom: 2rem;
    @media (max-width: 1000px) {
      bottom: 1rem;
      font-size: 0.8rem;
      position: relative;
    }

    a {
      color: #0099b6;
      text-decoration: none;
      transition: 0.4s;
      &:hover {
        color: #ffc850;
      }
    }
  }
}

//Transitions
#myChart {
  animation: 1.5s fade-in forwards;
  margin-top: 3rem;
  margin-bottom: 2rem;
  padding-left: 1rem;
  padding-right: 1rem;
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

// Loading Circle
.lds-hourglass {
  display: inline-block;
  position: relative;
  width: 10rem;
  height: 10rem;
  display: flex;
  justify-content: center;
  align-items: center;
}
.lds-hourglass:after {
  content: " ";
  display: block;
  border-radius: 50%;
  width: 0;
  height: 0;
  margin: 8px;
  box-sizing: border-box;
  border: 50px solid #fff;
  border-color: #fff transparent #fff transparent;
  animation: lds-hourglass 1.2s infinite;
}
@keyframes lds-hourglass {
  0% {
    transform: rotate(0);
    animation-timing-function: cubic-bezier(0.55, 0.055, 0.675, 0.19);
  }
  50% {
    transform: rotate(450deg);
    animation-timing-function: cubic-bezier(0.215, 0.61, 0.355, 1);
  }
  100% {
    transform: rotate(900deg);
  }
}
</style>
