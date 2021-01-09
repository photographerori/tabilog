<template>
  <div class="container">
    <h2 class="top">位置検索</h2>
    <div class="content">
      <!-- Google Mapから位置情報追加 -->
      <form id="search-form" @submit.prevent>
        <h2 class="title">Google Mapから位置情報追加する</h2>
        <input
          id="inputAddress"
          type="textbox"
          value
          placeholder="地名を入力"
        />
        <input
          id="search-button"
          type="button"
          value="検索"
          @click="codeAddress()"
        />
      </form>
      <div id="map">
        <!-- ここに地図が表示される -->
      </div>

      <div>
        <button class="btn submit-btn" @click="publish">
          Submit
        </button>
      </div>
    </div>
  </div>
</template>
<script>
// import firebase from "firebase";
import { auth } from "@/firebase"
import { db } from "@/firebase"
// import GoogleMap from "../components/GoogleMap";
let GoogleMapsApiLoader = require("google-maps-api-loader")

export default {
  data() {
    return {
      currentUser: {},
      google: null,
      map: null,
      geocoder: null,
      marker: [],
      currentlatlng: {
        lat: 35.6809591,
        lng: 139.7673068
      },
      createUser: {}
    }
  },
  // components: {
  //   GoogleMap
  // },
  created() {
    auth.onAuthStateChanged(user => {
      this.currentUser = user
    })
  },
  firestore() {
    return {
      createUser: db.collection("users").doc(this.$route.params.uid)
    }
  },
  methods: {
    initMap: function() {
      this.geocoder = new this.google.maps.Geocoder()
      this.map = new this.google.maps.Map(document.getElementById("map"), {
        center: this.currentlatlng,
        zoom: 15
      })
    },
    // 検索、マーカーの設置
    codeAddress: function() {
      let inputAddress = document.getElementById("inputAddress").value
      // 地図の移動や座標の取得
      this.geocoder.geocode({ address: inputAddress }, (results, status) => {
        if (status == "OK") {
          let latitude = results[0].geometry.location.lat()
          let longitude = results[0].geometry.location.lng()
          this.currentlatlng.lat = latitude
          this.currentlatlng.lng = longitude
          let latlng = new this.google.maps.LatLng(latitude, longitude)

          // マーカー設置
          this.marker = new this.google.maps.Marker({
            map: this.map,
            position: {
              lat: this.currentlatlng.lat,
              lng: this.currentlatlng.lng
            },
            animation: this.google.maps.Animation.DROP
          })

          // googleMapの中心座標をずらす
          this.map.setCenter(latlng)
        } else {
          alert("該当する結果がありませんでした：" + status)
        }
      })
    },
    publish() {
      const ref = db.collection("posts").doc()
      const date = this.$date(new Date())
      ref
        .set({
          createUsername: this.createUser.displayName,
          createdAt: date,
          uid: this.currentUser.uid,
          id: ref.id,
          lat: this.currentlatlng.lat,
          lng: this.currentlatlng.lng
        })
        .then(() => {
          this.$router.push("/create/" + this.currentUser.uid)
          alert("The post got published!")
        })
    }
  },
  async mounted() {
    this.google = await GoogleMapsApiLoader({
      apiKey: "AIzaSyCgHlOo-1ywOyvD5AIjjmuSQg5RFtzTlXw"
    })
    this.initMap()
  }
}
</script>
<style scoped>
body {
  margin: 0;
  padding: 0;
}
.container {
  width: 60%;
  border: 3px solid black;
  margin: 30px auto;
}
.top {
  font-size: 36px;
  text-decoration: underline;
}
.content {
  position: flex;
  justify-content: flex-start;
  text-align: left;
  padding: 0px 150px;
}
#search-form {
  margin: 15px 0;
}
#inputAddress {
  width: 300px;
  height: 30px;
  font-size: 18px;
}
#search-button {
  height: 30px;
  background: none;
  color: #666;
  border: none;
  font-size: 20px;
  cursor: pointer;
}
#search-button:hover {
  color: #7fbfff;
}
#map {
  height: 450px;
  width: 600px;
  margin: 0 auto;
}
.btn {
  display: inline-block;
  background-color: white;
  border: 1px solid black;
  font-weight: 600;
  padding: 10px 40px;
  margin: 10px auto;
  cursor: pointer;
  -webkit-transition: all 0.3s;
  transition: all 0.3s;
  color: black;
  text-decoration: none !important;
}
.submit-btn {
  box-shadow: 2px 2px 0 0 black;
  border-radius: 7px;
}
.submit-btn:hover {
  transform: translate(2px, 2px);
  box-shadow: none;
}
</style>
