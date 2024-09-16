<template>
    <v-app>
      <v-container fluid class="container">
        <div v-if="showAnimation">
          <div class="square"></div>
          <div class="square"></div>
          <div class="square"></div>
          <div class="square"></div>
          <div class="square"></div>

          <div class="circle"></div>
          <div class="circle"></div>
          <div class="circle"></div>
          <div class="circle"></div>
          <div class="circle"></div>

          <div class="triangle"></div>
          <div class="triangle"></div>
          <div class="triangle"></div>
          <div class="triangle"></div>
          <div class="triangle"></div>
        </div>
  <div v-if="loadings" class="loader-overlay">
    <div class="loader-circle"></div>
    <div class="loader-text">
      <p>
        <span :class="{'checked': isDataLoaded}">✔</span> 
        <span v-if="!isDataLoaded">Getting Company Data...<span class="dots"></span></span>
        <span v-else>Company Data</span>
      </p>
      <p>
        <span :class="{'checked': areFinancialsLoaded}">✔</span> 
        <span v-if="!areFinancialsLoaded">Getting Financial Metrics...<span class="dots"></span></span>
        <span v-else>Financial Metrics</span>
      </p>
      <p>
        <span :class="{'checked': isSentimentLoaded}" class="unchecked">✔</span> 
        <span v-if="!isSentimentLoaded">Getting Sentiment Score<span class="dots"></span></span>
        <span v-else>Sentiment Loaded</span>
      </p>
    </div>
  </div>
 
        <v-row align="center">
          <div v-if='showAnimation' class="three-js-background"></div>

            <v-col cols="auto" class="logo-container">
                <img src="@/assets/Success.png" height="100"> 
            </v-col>
            <v-col cols="auto">
                <span class="tool-name">StockSift</span>
            </v-col>
        </v-row>
        <v-row justify="center" align="center" class="text-center">
                <v-col cols="12">
                <div class="typewriter">
                    <h1>{{ animatedText }}</h1>
                </div>
                <div>
                    <v-col cols="12">
                        <v-card class="widget-format1">
                          <!-- <v-skeleton-loader
                            v-if="!isDataLoaded & !isFirstSearchComplete"
                            type="card, list-item, avatar"
                            height="200"
                          /> -->
                          <canvas id="stockChart" width="400" height="200" v-if="showWidgets"></canvas>
                        </v-card>
                    </v-col>
                </div>
                <div>
                </div>
                    <div class="second-row">
                    <v-row justify="center">
                      <v-col cols="12" sm="7" md="10" lg="6">
                          <v-text-field
                              dark
                              height="30"
                              v-model="searchQuery"
                              :class="{ 'search-field': searchInitiated }"
                              label="Search by Ticker Symbol"
                              append-icon="mdi-magnify"
                              @click:append="getNews()"
                              @keyup.enter="getNews()"
                              :loading="loadings"
                              outlined
                              dense
                              class="text-field white-text"
                          ></v-text-field>
                      </v-col>
                          <div class="slider-container bg-surface-variant rounded-pill" v-if="showWidgets">
                          <v-progress-linear
                            class="buy-rate bg-surface-variant rounded-pill"
                            v-model="sliderValue"
                            height="40"
                            color="teal"
                          >
                            <strong>{{ Math.ceil(sliderValue) }}% Buying rate</strong>
                          </v-progress-linear>
                          </div>
                    </v-row>

                </div>
                </v-col>
            </v-row>

        <!-- Container pentru widget-uri, afișat doar după căutare -->
        <v-row v-if="showWidgets">
          <v-col cols="12" md="6" lg="4" >
            <v-card class="widget-format">
              <v-card-title>Company Description</v-card-title>
                <v-card-text class="white--text text-left">
                  <span v-if="!showFullDescription">
                    {{ truncatedDescription }}
                    <a href="#" @click.prevent="toggleDescription">... Read more</a>
                  </span>
                  <span v-else>
                    {{ description }}
                  </span>
                </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12" md="6" lg="4">
            <v-card class="widget-format">
              <v-card-title>Sentiment Analysis</v-card-title>
              <v-progress-circular
                  :size="300"
                  :width="40"
                  :value="this.sentiment_score"
                  color="teal"
                  rotate="-90"
                >
                <div class="progress-content">
                  <span class="sentiment-score">{{ this.sentiment_score }}%</span>
                  <span class="positive-news">Positive News</span>
                </div>
              </v-progress-circular>
              <!-- <v-card-text>News from x-date</v-card-text> -->
            </v-card>
          </v-col>
          <v-col cols="12" md="6" lg="4">
            <v-card class="widget-format">
              <v-card-title>Metrics Evaluation</v-card-title>
              <v-card-text class="white--text text-left">
                <p>Market Cap: {{ financialMetrics.marketCap.toLocaleString() }} 💲</p>
                <!-- <p>Dividend Yield: {{ financialMetrics.dividendYield }}%</p> -->
                <p>Dividend Yield: 
                  <span v-if="financialMetrics.dividendYield !== null">
                    {{ financialMetrics.dividendYield }}%
                  </span>
                  <span v-else>
                    No Dividend
                  </span>
                </p>
                <p>Trailing PE RATIO: {{ financialMetrics.peRatio }}</p>
                <p>Forward PE RATIO: {{ financialMetrics.fwRatio }}</p>
                <p>Revenue: {{ financialMetrics.revenue.toLocaleString() }} 💲</p>
                <p>Net Income: {{ financialMetrics.netIncome.toLocaleString() }} 💲</p>
                <p>EBITDA: {{ financialMetrics.ebitda.toLocaleString() }} 💲</p>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-app>
  </template>
  
  <script>
import Chart from 'chart.js/auto';
import anime from 'animejs';
import * as THREE from 'three';
import axios from 'axios';
import Cookies from 'js-cookie';

  export default {

    data() {
      return {
        searchQuery: '',
        sentiment_score: null,
        showWidgets: false,
        searchInitiated: false,
        showAnimation: true,
        years: [],
        prices: [],
        description: '',
        isFirstSearchComplete: false,
        financialMetrics: {
          marketCap: null,
          peRatio: null,
          dividendYield: null,
          fwRatio: null,
          revenue: null,
          netIncome: null,
          ebitda: null,
        },
        isDataLoaded: false,
        isSentimentLoaded: false,
        areFinancialsLoaded: false,
        showFullDescription: false,
        loadings: false,
        animatedText: 'Search and analyse any stock...',
        loading: false,
        sliderValue: 0,
        trackColor: 'purple',
        thumbColor: 'white',
        chart: null,
        sentimentValue: 0,
        cubes: [],
        time: 0,
        color: {
          r: 0, g: 0, b: 255, rs: 0, gs: 0, bs: 0, rt: 0, gt: 0, bt: 255
        }
        };
    },
    methods: {
      getFinancials() {
        axios.get(`http://127.0.0.1:8000/trends/financialmetrics/${this.searchQuery}`)  
          .then(response => {
            console.log('financialmetrics', response.data)
            this.financialMetrics = {
              marketCap: response.data.marketCap,
              dividendYield: response.data.dividendYield,
              peRatio: response.data.peRatio,
              fwRatio: response.data.fwRatio,
              revenue: response.data.revenue,
              netIncome: response.data.netIncome,
              ebitda: response.data.ebitda,
            }
            console.log('marketCap thisss', this.financialMetrics.marketCap)
            this.areFinancialsLoaded = true;
          })     
          .catch(error => {
          console.log('Error here: ', error)
        })
      },
      toggleDescription() {
      this.showFullDescription = !this.showFullDescription;
    },
      getDescription() {
        axios.get(`http://127.0.0.1:8000/trends/companydescription/${this.searchQuery}`)  
          .then(response => {
            console.log('company_description', response.data)
            this.description = response.data;
            this.isDataLoaded = true;
          })     
          .catch(error => {
          console.log('Error here: ', error)
        })
      },

      getNews(){
        this.isDataLoaded = false;
        this.isSentimentLoaded = false;
        this.areFinancialsLoaded = false;
        this.loadings = true;
        console.log('getnews post', this.searchQuery)
        let formData = new FormData();
        formData.append('ticker', this.searchQuery)
        const csrfToken = Cookies.get('csrftoken')

        this.getGraph();
        this.getDescription();
        this.getFinancials();
        axios({
          method: 'POST',
          url: 'http://127.0.0.1:8000/trends/stocknews/',
          data: formData,
          headers: {
            'X-CSRFToken': csrfToken,
            'Content-Type': 'multipart/form-data'
          },
          withCredentials: true,
        })
        .then(response => {
          this.isSentimentLoaded = true;
          this.sentiment_score = response.data.sentiment_score;
          console.log('sentiment', this.sentiment_score)
          console.log('Reponse data: ', response.data)
          this.onSearch();
          this.loadings = false;
          this.isFirstSearchComplete = true
        })
        .catch(error => {
          console.log('Error here: ', error)
        })
      },
      getGraph() {
        axios.get(`http://127.0.0.1:8000/trends/averageprices/${this.searchQuery}`) 
          .then(response => {
            console.log('sunt in getGraph');
            console.log(response.data);
            this.years = Object.keys(response.data).map(year => parseInt(year));
            this.prices = Object.values(response.data).map(price => parseInt(price))
            console.log('years', this.years)

          })
          .catch(error => {
            console.log(error)
          })
      },
    addLights() {
      for (let x = -5; x <= 5; x += 5) {
        for (let z = -5; z <= 5; z += 5) {
          const light = new THREE.PointLight('white', 1, 7.5);
          light.position.set(x, 2, z);
          this.scene.add(light);
        }
      }
    },
      onSearch() {
        this.searchInitiated = true;
        this.showWidgets = true;
        this.loading = true;
        this.showAnimation = false;
      setTimeout(() => {
        this.renderHardcodedChart();
        this.animateToBuySellPercentage(80);
        this.loading = false;
      }, 1);
      },

    animateToBuySellPercentage(percentage) {
      this.sliderValue = percentage;
    },

    renderHardcodedChart() {
      const ctx = document.getElementById('stockChart').getContext('2d');
      if (this.chart) {
        this.chart.destroy(); // Destroy the existing chart if it exists
      }
      this.chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: this.years,
          datasets: [{
            label: 'Stock Price',
            data: this.prices,
            borderColor: 'rgb(75, 192, 192)',
            tension: 0.1
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: false
            }
          },
          responsive: true,
          maintainAspectRatio: false
        }
      });
    },


    randomValues() {
  anime({
    targets: '.square, .circle, .triangle',
    translateX: function() {
      return anime.random(-1000, 1000);
    },
		translateY: function() {
      return anime.random(-500, 500);
    },
		rotate: function() {
			return anime.random(0, 360);
		},
		scale: function() {
			return anime.random(.2, 2);
		},
    duration: 1000,
		easing: 'easeInOutQuad',
    complete: this.randomValues,
	});
}

    },

    watch: {
  },
  computed: {
    truncatedDescription() {
    if (this.description.length > 700) {
      return this.description.substring(0, 700) + '...';
    }
    return this.description;
  }
  },
    mounted() {
  this.$nextTick(() => {
    this.renderHardcodedChart();
  });
  this.randomValues();
  window.addEventListener('resize', this.onWindowResize);

  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onWindowResize);
  }
  };
  </script>
  
  <style scoped>

a {
  color: #42b983;
  cursor: pointer;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.text-left {
  text-align: left;
}

.loader-overlay {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: fixed;  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 9999;
}

.loader-circle {
  border: 5px solid #f3f3f3;
  border-radius: 50%;
  border-top: 5px solid #3498db;
  width: 50px;
  height: 50px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loader-text {
  margin-top: 20px;
  text-align: center;
  color: white;
}

.checked {
  margin-right: 3%;
  color: green;
  font-weight: bold;
}
.unchecked{
  margin-right: 5px;
}

.container {
  background: rgb(0,0,0);
background: linear-gradient(62deg, rgba(0,0,0,1) 0%, rgba(19,15,64,1) 71%);
    height: 100%;
}

.typewriter h1 {
    color: #FFFFFF; 
  text-shadow: 
    1px 1px 0px #ddd, 
    2px 2px 0px #d4d4d4, 
    2px 2px 0px #ccc; 
  font-size: 2em; /* Adjust the size of the text */
  font-weight: bold;
  overflow: hidden;
  border-right: .15em solid black;
  white-space: nowrap;
  margin: 0 auto;
  letter-spacing: .15em;
  font-family: Courier, monospace;
  animation: typing 7s steps(40, end), blink-caret .75s step-end infinite;
  z-index: 9999; /* Adjust the value as needed */

}

@keyframes typing {
  from { width: 0 }
  to { width: 100% }
}

@keyframes blink-caret {
  from, to { border-color: transparent }
  50% { border-color: black }
}

  
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
  
header {
    background-color: #42b983;
    color: white;
    padding: 10px;
  }
  
main {
    margin: 20px;
  }
  
footer {
    color: #586069;
    padding: 10px;
    position: absolute;
    bottom: 0;
    width: 100%;
    background: rgb(2,0,36);
  }

.widget-format{
    background-color: transparent;
    color: white;
    margin: 1vh;
    padding: 3%;
    border: 2px solid linear-gradient(62deg, rgba(0,0,0,1) 0%, rgba(19,15,64,1) 71%);
    border-image-slice: 1;
    box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.3); /* Add shadow for raised effect */
    z-index: 10; /* Bring the card in front */
    position: relative; /* Ensure z-index works */
}

.widget-format1{
    background-color: transparent;
    color: white;
    margin: 1vh;
    border: 1px solid linear-gradient(62deg, rgba(0,0,0,1) 0%, rgba(19,15,64,1) 71%);
    padding: 3%;
}

.slider-container {
  width: 45vw;
  padding: 20px;
  background-image: linear-gradient(to bottom, #2c3e50, #2d3141, #292632, #221c23, #181316, #120e10, #0a0708, #000000, #000000, #000000, #000000, #000000);
}

.second-row {
    margin-top: 3vh;
    margin-left: 1vw;
    margin-right: 1vw;
}

.logo-container {
    padding-right: 0vw;
    padding-left: 1vw;
}

.tool-name {
  font-size: 1.2rem; /* Adaptează dimensiunea fontului după necesități */
  font-weight: bold;
  font-size: 3vh;
  color: #FFFFFF; /* Schimbă culoarea dacă este necesar */
}

.text-field {
  margin-top: 1.3vh;
  margin-left: 0.5vw;
    /* margin-right: 3vw; */
  color: white;
}

* {
	margin: 0;
	padding: 0;
}

body {
	background: #222222;
	overflow: hidden;
}

.square {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 50px;
	height: 50px;
	background: linear-gradient(#303030, #757575);
	z-index: 3;
}

.circle {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 50px;
	height: 50px;
	background: #1cd99d;
	border-radius: 50%;
}

.triangle {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 50px;
	height: 50px;
	background: #f5f5f5;
	clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
	-webkit-clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}

@keyframes slideToLeft {
  0% {
    transform: translateX(0%);
  }
  100% {
    transform: translateX(-4%); /* Adjust this value based on the desired final position */
  }
}

.search-field {
  position: relative; 
  animation: slideToLeft 1s forwards;
}

.three-js-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
}

.progress-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.sentiment-score {
  margin-bottom: 10px; /* Adjust this value to control the spacing */
}

.positive-news {
  font-size: 16px; /* Adjust font size if needed */
}

.dots::after {
  content: ' ';
  display: inline-block;
  width: 1em;
  text-align: left;
  animation: ellipsis 1.5s infinite;
}

@keyframes ellipsis {
  0% {
    content: ' ';
  }
  25% {
    content: '.';
  }
  50% {
    content: '..';
  }
  75% {
    content: '...';
  }
  100% {
    content: ' ';
  }
}
  </style>