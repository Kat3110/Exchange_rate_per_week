<template>
  <v-app>
    <v-app-bar app v-if="isLoading === false">
      <v-switch v-model="$vuetify.theme.dark" hide-details="false" inset prepend-icon="mdi-theme-light-dark"></v-switch>
      <v-spacer></v-spacer>
      <v-switch v-if="isPercent === true" v-model="isPercent" hide-details="false" inset
        prepend-icon="mdi-percent-outline"></v-switch>
      <v-switch v-if="isPercent === false" v-model="isPercent" hide-details="false" inset
        prepend-icon="mdi-currency-rub"></v-switch>
    </v-app-bar>



    <v-main>
      <div class="d-flex mt-10" v-if="isLoading === true">
        <v-spacer></v-spacer>
        <v-progress-circular :size="100" :width="10" color="purple" indeterminate></v-progress-circular>
        <v-spacer></v-spacer>
      </div>

      <div v-if="isLoading === false">
        <v-card class="mx-auto mt-3 d-flex flex-column" max-width="444" v-for="currency in currencies"
          :key="currency.ID">
          <div class='pt-4 pl-4 pr-4 d-flex align-start justify-space-between'>
            <div>
              <div class="subtitle-1 font-weight-medium text--primary">{{ currency.CharCode }}</div>
              <p class="subtitle-1 text--primary" v-if="currency.Nominal === 1">
                {{ currency.Name }}
              </p>
              <p class="subtitle-1 text--primary" v-if="currency.Nominal !== 1">
                {{ currency.Nominal }} {{ currency.Name }}
              </p>
            </div>
            <v-btn color="#2196f3" text @click="openPopup(currency.CharCode)">
              <v-icon large>
                mdi-chart-timeline-variant-shimmer
              </v-icon>
            </v-btn>
          </div>
          <div class='pb-4 pl-4 pr-4 d-flex align-center justify-space-between'>
            <p class="text-h5 mb-0">
              {{ currency.Value }} &#8381;
            </p>
            <div class="d-flex flex-column">
              <div v-if="isPercent === false">
                <div class="red--text text-h6 d-flex align-center" v-if="currency.Value < currency.Previous">
                  <v-icon color="red" large>
                    mdi-menu-down
                  </v-icon> {{ (currency.Previous - currency.Value).toFixed(2) }}
                  &#8381;
                </div>
                <div class="green--text text-h6 d-flex align-center" v-if="currency.Value > currency.Previous">
                  <v-icon color="green" large>
                    mdi-menu-up
                  </v-icon> {{ (currency.Value - currency.Previous).toFixed(2) }}
                  &#8381;
                </div>
              </div>
              <div v-if="isPercent === true">
                <div class="red--text text-h6 d-flex align-center" v-if="currency.Value < currency.Previous">
                  <v-icon color="red" large>
                    mdi-menu-down
                  </v-icon> {{ (100 - ((currency.Value / currency.Previous) * 100)).toFixed(2) }}
                  %
                </div>
                <div class="green--text text-h6 d-flex align-center" v-if="currency.Value > currency.Previous">
                  <v-icon color="green" large>
                    mdi-menu-up
                  </v-icon> {{ (-100 + ((currency.Value / currency.Previous) * 100)).toFixed(2) }}
                  %
                </div>
              </div>


            </div>
          </div>

        </v-card>
        <v-dialog v-model="dialog" persistent max-width="600">
          <v-card>
            <div class="d-flex align-center">
              <v-btn color="primary" text @click="dialog = false">
                <v-icon>mdi-window-close</v-icon>
              </v-btn>
              <div class="text-body-1 font-weight-thin text-center text-sm-h6">
                Данные за семь дней от {{ days[1] }} по {{ days[0] }}
              </div>
            </div>
            <v-sheet class="v-sheet--offset mx-auto" elevation="12" max-width="600">
              <v-sparkline :labels="value" :value="value" padding="20" height="150" stroke-linecap="round" smooth>
              </v-sparkline>
            </v-sheet>
          </v-card>
        </v-dialog>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import axios from 'axios';
export default {
  name: 'App',

  components: {
  },

  data () {
    return {
      days: [],
      value: [],
      dialog: false,
      currencies: [],
      url: '',
      isPercent: false,
      isLoading: true,
      counter: 1,
      responseNextDay: null,
      response: null,
      temporary: [],
    }
  },
  methods: {
    async fetchFirstDay () {
      try {
        this.isLoading = true
        this.response = await axios.get('https://www.cbr-xml-daily.ru/daily_json.js')
        this.days.push(`${this.response.data.Date.substring(8, 10)}.${this.response.data.Date.substring(5, 7)}`)
        this.currencies = this.response.data.Valute
        this.url = this.response.data.PreviousURL
      } catch {
        console.log('error 1')
      } finally {
        for (let name in this.currencies) {
          this.currencies[name].Values = []
          this.currencies[name].Values.push(Number(this.currencies[name].Value.toFixed(2)))
        }
        await this.fetchNextDay()
      }
    },
    async fetchNextDay () {
      try {
        this.responseNextDay = await axios.get(this.url)
        this.url = this.responseNextDay.data.PreviousURL
        this.temporary = this.responseNextDay.data.Valute
        for (let name in this.temporary) {
          this.currencies[name].Values.push(Number(this.temporary[name].Value.toFixed(2)))
        }
      } catch {
        console.log('error 2')
      } finally {
        this.counter = this.counter + 1
        if (this.counter === 7) {
          this.days.push(`${this.responseNextDay.data.Date.substring(8, 10)}.${this.responseNextDay.data.Date.substring(5, 7)}`)
          this.isLoading = false
        } else {
          await this.fetchNextDay()
        }
      }
    },
    openPopup (code) {
      this.value = this.currencies[code].Values.reverse()
      this.dialog = true
    }
  },
  mounted () {
    this.fetchFirstDay()
  }
};
</script>
