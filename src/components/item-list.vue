<template>
  <div class="item-list">
    <span class="main-head">
      Item List
    </span>
    <div class="list-tab">
      <button class="mdl-button mdl-js-button mdl-js-ripple-effect" v-on:click="seeUnpurchased">
        <span v-bind:class="classObject">Current</span>
      </button>
      <button class="mdl-button mdl-js-button mdl-js-ripple-effect" v-on:click="seePurchased">
        <span v-bind:class="classObject">Purchased</span>
      </button>
    </div>
    <div class="" v-if='showUnpurchased' v-for="(item, key) in unpurchasedItems">
      <Item
      :item="item"
      :itemKey="key"
      :hasBeenLiked="hasBeenLiked(key)"
      :likeCount="likeCount(item.likes || 0)"
      />
    </div>
    <div class="" v-else v-for="(item, key) in purchasedItems">
      <Item
      :item="item"
      :itemKey="key"
      :hasBeenLiked="hasBeenLiked(key)"
      :likeCount="likeCount(item.likes || 0)"
      />
    </div>

  </div>
</template>

<script>
import Item from './item.vue'
import _ from 'lodash'
export default {
  name: 'item-list',
  components: {
    Item
  },
  data () {
    return ({
      items: {},
      showUnpurchased: true,
    })
  },
  mounted () {
    const itemList = firebase.database().ref().child(`items`)
    itemList.on('value', (snapshot) => {
      console.log(snapshot.val())
      this.items = snapshot.val()
    })
  },
  methods: {
    markAsPurchased (key) {
      firebase.database().ref(`items/${key}`).update({purchased: true})
    },
    deleteItem (key) {
      firebase.database().ref(`items/${key}`).remove()
    },
    hasBeenLiked (key) {
      if (this.items[key].likes) {
        return this.items[key].likes[document.cookie]
        ? false
        : true
      } else {
        return true
      }
    },
    likeCount (likes) {
      return Object.keys(likes).length
    },
    seePurchased () {
      this.showUnpurchased = false
    },
    seeUnpurchased () {
      this.showUnpurchased = true
    }
  },
  computed: {
    unpurchasedItems () {
      let unpurchased = _.pickBy(this.items, function(value, key) {
        return !value.purchased
      })
      return unpurchased
    },
    purchasedItems () {
      let purchased = _.pickBy(this.items, function(value, key){
        return value.purchased
      })
      return purchased
    },
    classObject () {
      return {
        activeTab: this.showUnpurchased
      }
    }
  }
}
</script>

<style lang='scss' scoped>
.item-list {
  display: flex;
  flex-direction: column;
  width: 100%;
  min-height: 350px;
  padding: 15px;
}
.main-head {
  font-size: 20px;
  font-weight: bold;
  color: #008cc7;
},
.activeTab {
  color: orange
}
</style>
