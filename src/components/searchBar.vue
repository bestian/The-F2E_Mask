<template>
  <div class="category" :class="{'close' : !isOpen}">
    <div class="search city-selector-set">
      <select name="country" class="country" v-model="selected.country"
      @change="selected.dist = null">
        <option
          :value="item"
          v-for="(item, index) in country" :key="index"
        >
          {{ item }}
        </option>
      </select>
      <select name="dist" class="dist" v-model="selected.dist"
      @change="passLatLng(false)">
        <option value="null" selected>-- 請選擇區域 --</option>
        <option
          v-for="(item, index) in dist"
          :key="index"
          :value="item"
        >
          {{ item }}
        </option>
      </select>
      <input type="radio" name="target" v-model = "target" v-bind:value="'成人'" />成人
      <input type="radio" name="target" v-model = "target" v-bind:value="'兒童'" />兒童
      <p>
        尚能取得{{target}}口罩的藥局有
        <span>
          {{ getLength(list) }}
        </span>
        家
      </p>
    </div>
    <ul class="list">
      <li v-for="(item, index) in list" :key="index">
        <h3>{{item.name}}</h3>
        <p><i class="icon_pin"></i>{{item.address}}</p>
        <p><i class="icon_tel"></i>{{item.phone}}</p>
        <div class="btnWrap">
          <button :class="{'soldout': !item.mask_adult}">
            成人：{{item.mask_adult | mask}}
          </button>
          <button :class="{'soldout': !item.mask_child}">
            兒童：{{item.mask_child | mask}}
          </button>
        </div>
      </li>
    </ul>
    <button class="toggleBtn" @click="isOpen = !isOpen">
      <i class="icon_arrow" :class="{'close' : !isOpen}"></i>
    </button>
  </div>
</template>

<script>
/* eslint-disable camelcase */
export default {
  name: 'searchBar',
  props: {
    properties: Array,
  },
  data() {
    return {
      target: '成人',
      isOpen: true,
      twCity: [],
      api: [],
      selected: {
        country: '新北市',
        dist: null,
      },
    };
  },
  created() {
    this.getAPI();
    this.getCountry();
  },
  methods: {
    getLength(list) {
      let ans;
      if (this.target === '成人') {
        ans = list.filter(item => item.mask_adult > 10).length;
      }
      if (this.target === '兒童') {
        ans = list.filter(item => item.mask_child > 10).length;
      }
      return ans;
    },
    async getAPI() {
      const { data } = await this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json?fbclid=IwAR14GsJ3L_SUQSTO1F9ru1pydJrv2t9KJA5xQqqhw5Fode7Y7VGubLGjnBM');
      // console.log('api =>', data);
      this.api = data.features;
    },
    async getCountry() {
      try {
        const { data } = await this.$http.get('./latlng.json');
        this.twCity = data;
        // console.log('tw =>', data);
      } catch (err) {
        console.log(err);
      }
    },
    passLatLng() {
      const apiPassArr = this.api
        .filter(item => item.properties.address.includes(this.selected.country)
        && item.properties.address.includes(this.selected.dist));
      this.$emit('pass-evt', apiPassArr);
    },
  },
  computed: {
    country() {
      return this.twCity
        .map(item => item.city)
        .filter((item, index, arr) => arr.indexOf(item) === index);
    },
    dist() {
      return this.twCity
        .filter(item => item.city === this.selected.country)
        .map(item => item.district)
        .filter((item, index, arr) => arr.indexOf(item) === index);
    },
    list() {
      return this.api.map(item => item.properties).filter((item) => {
        if (!this.selected.dist) {
          return item.address.includes(this.selected.country);
        }
        return item.address.includes(this.selected.country)
        && item.address.includes(this.selected.dist);
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import '@/assets/scss/index';
</style>
