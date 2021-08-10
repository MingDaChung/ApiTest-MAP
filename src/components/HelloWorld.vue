<template>
  <div class="hello">
    <div class="search_box">
      <span>Please Input Address：</span>
      <input
        type="text"
        v-model.lazy.trim="keyword"
        class="input"
        @change="page = 1"
      />
    </div>
    <div class="showmap">
      <button @click="showMap = !showMap">
        Toggle map
      </button>
    </div>
    <table class="table table_border" id="idData">
      <thead>
        <tr>
          <td>map</td>
          <td>sno</td>
          <td>sna</td>
          <td>tot</td>
          <td>sarea</td>
          <td>mday</td>
          <td>lat</td>
          <td>lng</td>
          <td>ar</td>
          <td>sareaen</td>
          <td>snaen</td>
          <td>aren</td>
          <td>bemp</td>
          <td>srcUpdateTime</td>
          <td>updateTime</td>
          <td>infoTime</td>
          <td>infoDate</td>
        </tr>
      </thead>
      <tr v-for="data in mergeDatas" :key="data">
        <td>
          <botton class="botton_icon" @click="ShowTheWay(data)"
            ><i class="bx bx-map"></i
          ></botton>
        </td>
        <td>{{ data.sno }}</td>
        <td>{{ data.sna }}</td>
        <td>{{ data.tot }}</td>
        <td>{{ data.sarea }}</td>
        <td>{{ data.mday }}</td>
        <td>{{ data.lat }}</td>
        <td>{{ data.lng }}</td>
        <td>{{ data.ar }}</td>
        <td>{{ data.sareaen }}</td>
        <td>{{ data.snaen }}</td>
        <td>{{ data.aren }}</td>
        <td>{{ data.bemp }}</td>
        <td>{{ data.srcUpdateTime }}</td>
        <td>{{ data.updateTime }}</td>
        <td>{{ data.infoTime }}</td>
        <td>{{ data.infoDate }}</td>
      </tr>
    </table>

    <l-map v-if="showMap" :zoom="zoom" :center="center" class="Map">
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker :lat-lng="marker" />
      <l-icon-default :image-path="path" />
    </l-map>

    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item">
          <button type="button" class="page-link" @click="page--">
            Previous
          </button>
        </li>
        <li class="page-item">
          <!-- <button 
            type="button"
            class="page-link"
            v-for="pageNumber in pages.slice(page - 1, page + 5)"
            @click="page = pageNumber"
          >
            {{ pageNumber }}
          </button>-->
          {{ page }}
        </li>
        <li class="page-item">
          <button type="button" @click="page++" class="page-link">
            Next
          </button>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
export default {
  name: "HelloWorld",
  components: {
    LMap,
    LTileLayer,
    LMarker,
  },
  data() {
    return {
      keyword: "",
      totaldata1: [],
      totaldata2: [],
      // mergeData: [],
      page: 1,
      perPage: 5,
      pages: [],
      //For the Map
      zoom: 16,
      path: "/images/",
      center: [],
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker: latLng(),
      showMap: false,
    };
  },
  created() {
    // Simple GET request using fetch
    fetch("https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.gz")
      .then((response) => response.json())
      .then((data) => (this.totaldata1 = data.retVal));

    fetch(
      "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
    )
      .then((response) => response.json())
      .then((data) => (this.totaldata2 = data));
  },
  methods: {
    /* setPages() {
      let numberOfPages = Math.ceil(this.mergeData.length / this.perPage);
      for (let index = 1; index <= numberOfPages; index++) {
        this.pages.push(index);
      }
    },*/
    paginate(mergeData) {
      let max = Math.ceil(mergeData.length / this.perPage);
      // console.log(mergeData.length / this.perPage);
      // console.log(`max:${max}`);
      if (this.page <= 0) {
        this.page = 1;
      }
      if (this.page >= max) {
        this.page = max;
      }

      let page = this.page;
      let perPage = this.perPage;
      let from = page * perPage - perPage;
      let to = page * perPage;
      return mergeData.slice(from, to);
    },
    ShowTheWay(data) {
      console.log(data);
      if (this.showMap == 0) {
        this.showMap = !this.showMap;
      }
      // return data.lat, data.lng;
      this.center = [data.lat, data.lng];
      this.marker = latLng(data.lat, data.lng);
    },
  },
  computed: {
    mergeDatas() {
      this.totaldata2.forEach((data) => {
        // console.log(data);
        data.mday = data.mday.replace(/[-|:]/g, "");
        data.sna = data.sna.replaceAll("YouBike2.0_", "");
        data.snaen = data.snaen.replaceAll("YouBike2.0_", "");
      });

      let mergeData = Object.values(this.totaldata1).concat(this.totaldata2);
      return this.paginate(
        mergeData.filter((data) => data.ar.indexOf(this.keyword) > -1)
      );
      // return mergeData.filter((data) => data.ar.indexOf(this.keyword) > -1);
    },
  },
};
</script>

<style scoped>
.search_box {
  font-size: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 10vh;
}

.showmap {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
table tr td {
  border: 1px solid black;
}
.table {
  width: 100%;
  height: 70vh;
  border-collapse: collapse;
}
button.page-link {
  display: inline-block;
}
button.page-link {
  font-size: 20px;
  color: #29b3ed;
  font-weight: 500;
}
.offset {
  width: 500px !important;
  margin: 20px auto;
}
.botton_icon {
  cursor: pointer;
}
nav {
  width: 100%;
  height: 10vh;
  /* bottom: 0; */
  display: flex;
  align-items: center;
  justify-content: center;
}
nav ul {
  list-style: none;
  width: 20%;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.Map {
  width: 50%;
  height: 500px;
  position: absolute;
  top: 10vh;
  left: 28%;
}
.botton_icon {
  font-size: 20px;
}
</style>
