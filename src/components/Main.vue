<template>
<div>
  <NavBar
  v-on:resetFromButton="resetFromButton"></NavBar>
  <div class="container">
      <div id="frontCard">
        <b-card title="Dictionnaire JeuxDeMots" sub-title="Université de Montpellier">
          <b-card-text>
            Ce site internet permet d'explorer le réseau lexico-sémantique de <a href="http://www.jeuxdemots.org/">JeuxDeMots</a>
          </b-card-text>
        </b-card>
      </div>
      <Search
        :requestAnswer.sync="requestAnswer"
        :infoData.sync="infoData"
        :infoListeMotsFromSearch.sync="infoListeMotsFromSearch"
        :showCardFromSearch.sync="showCardFromSearch"
        :showListeMotsFromSearch.sync="showListeMotsFromSearch"
        :displayLoader.sync="displayLoader"
        v-on:sortRequestAnswer="sortRequestAnswer"
        v-on:waitingRequestAnswer="waitingRequestAnswer"
        v-on:resetAllVariable="resetAllVariable"
      ></Search>
    </div>
  </div>
</template>

<script>
import NavBar from "./Navbar.vue";
import Search from "./Search.vue";

export default {
  name: "Main",
  components: {
    NavBar,
    Search,
  },

  data() {
    return {
      eid: "",
      requestAnswer: "",
      def: Array,
      relationsTriees: Object,
      relationsTypes: Array,
      relationsSizes: Object,
      infoData: [],
      headerMot: Object,
      //"information potentielle"
      relationsHeaderMot: ["POS", "r_lemma", "r_data", "équivalent masc", "équivalent fem", "homophone"],
      relationsToRemove: ["r_flpot","  	r_flpot"," 	r_flpot", "information potentielle", "r_node2relnode", "r_context", "r_link", "r_cooccurrence", "r_termgroup"],
      showCardFromSearch: false,
      showListeMotsFromSearch: false,
      infoListeMotsFromSearch: new Object(),
      displayLoader: false,
    };
  },

  methods: {
    waitingRequestAnswer: function () {
      setTimeout(() => {
        this.sortRequestAnswer();
        this.$emit("Ready");
      }, 1);
    },

    sortRequestAnswer: function () {
      // LECTURE JSON
      var objectJSON = JSON.parse(this.requestAnswer);
      if(Object.prototype.hasOwnProperty.call(objectJSON.data, 'requete')){    
        this.infoListeMotsFromSearch.listeMots = objectJSON.data.listeMots;
        this.infoListeMotsFromSearch.requete = objectJSON.data.requete;   
        this.showListeMotsFromSearch = true;   
      }else{
      // console.log(objectJSON["data"]); // Check JSON FILE
        this.def = objectJSON["data"]["definitions"];
        this.relationsTriees = objectJSON["data"]["relationsTriees"];
        this.eid = objectJSON["data"]["eid"];
        this.relationsTypes = objectJSON["data"]["relationsTypes"];
        this.relationsSizes = objectJSON["data"]["relationsSizes"];
        this.headerMot = new Object();
        for(var key in this.relationsTriees){
          if(this.relationsToRemove.includes(key)){
            delete this.relationsTriees[key]
          }
          if(this.relationsHeaderMot.includes(key)){
            this.headerMot[key] = ""
            for(var t of this.relationsTriees[key]){
                this.headerMot[key] += t[0] + ", "
            }
            this.headerMot[key] = this.headerMot[key].substring(0, this.headerMot[key].length - 2) 
            delete this.relationsTriees[key]
            if(key === "r_data"){
              this.headerMot["Information lexicale"] = this.headerMot[key]
              delete this.headerMot[key]
            }else if(key === "r_lemma"){
              this.headerMot["Lemme"] = this.headerMot[key]
              delete this.headerMot[key]
            }else if(key === "POS"){
              this.headerMot["Catégorie grammaticale"] = this.headerMot[key]
              delete this.headerMot[key]
            }else if(key === "équivalent fem"){
              this.headerMot["Équivalent féminin"] = this.headerMot[key]
              delete this.headerMot[key]
            }else if(key === "homophone"){
              this.headerMot["Homophone"] = this.headerMot[key]
              delete this.headerMot[key]
            }
          }
        }
        this.infoData.push(this.def);
        this.infoData.push(this.relationsTriees);
        this.infoData.push(this.headerMot);
        this.infoData.push(this.relationsTypes);
        this.infoData.push(this.relationsSizes)
        this.showCardFromSearch = true;
      }      
    },

    resetAllVariable: function () {
      this.requestAnswer = "";
      this.def = Array;
      this.relationsTriees = Object;
      this.relationsTypes = Array;
      this.infoData.length = 0;
      this.showCardFromSearch = false;
      this.displayLoader = true;
      this.relationsSizes = Object;
    },

    resetFromButton: function () {
      this.$emit('reset-historique')
      this.resetAllVariable();
      this.displayLoader = false;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#frontCard{
  padding-top: 16px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
