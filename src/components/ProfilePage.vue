<template>
<div class="content container">

    <div class="row">
      
      <div class=preContent id="userStuff">
      
        <div class="col s4">
          <div class=userCollectionWishlistBox>
            <h5>{{username}}'s Stuff</h5>
            <a class="waves-effect waves-light btn">Wish List</a>            
            <a class="waves-effect waves-light btn">Collection</a>
          </div>
        </div>

        <!--<div class="col s8">
          <form>
            <p>
              <input type="checkbox" id="check1" />
              <label for="test5">You</label>
            </p>
              <p>
              <input type="checkbox" id="check2" />
              <label for="test5">There</label>
            </p>
              <p>
              <input type="checkbox" id="check3" />
              <label for="test5">Hey</label>
            </p>
          </form></div> -->
          <div class="col s12">
            <form>   
              <div class="input-field col s4">
                <input type=text name=filterSearch placeholder="Filter Search">
              </div>
              <div id="brand-selector" class="input-field col s4">
                <select v-model="selectedBrand" class="browser-default">
                  <option v-for="option in brandOptions" v-bind:value="option">
                    {{option}}
                  </option>
                </select>
              </div>
              <div id="sort-selector" class="input-field col s4">
                <select class="browser-default">
                  <option value="price">Price</option>
                  <option value="name">Name</option>
                </select>
              </div>
            
            </form>

          </div>      
    </div>
      
    <div v-if="dataLoaded" class="userMainContent row">
      <CirclePop v-for="(item, key) in items"
                 v-bind:item="item" 
                 v-on:remove="deleteItem(key)"
                 :key="key">
      </CirclePop>
    </div>

    </div>

  </div>
</template>

<script>
  import CirclePop from '@/components/ProfileComponents/CirclePop'
  import store from '../storage'
  import UserService from '../userservice'

  export default {
    components: { CirclePop },
    name: 'ProfilePage',
    data () {
      return {
        sharedState: store,
        selectedBrand: '',
        username: store.state.auth.user.displayName,
        dataLoaded: false,
        items: {}
      }
    },
    computed: {
      brandOptions: function () {
        UserService.fillData(this.sharedState.firebase)
        this.selectedBrand = 'All Brands'
        return ['All Brands', 'Adventure Time', 'TMNT']
      }
    },
    watch: {
      selectedBrand: function (value) {
        console.log('in watch function')
        console.log('value ' + value)
        var vm = this
        console.log
        UserService.getPops(this.sharedState.firebase, value).then(function (data) {
          console.log('got the pops\n' + JSON.stringify(data.val()))
          var pops = data.val()
          if (pops !== null) {
            // TODO add something if there are no pops
            vm.items = pops
            vm.dataLoaded = true
          } else {
            vm.dataLoaded = false
          }
        })
      }
    },
    methods: {
      deleteItem: function (itemUid) {
        console.log('in this function')
        // var userService = new UserService()
        // userService.deleteItemDB(this.sharedState.firebase, itemUid)
        var firebase = this.sharedState.firebase
        var user = firebase.auth().currentUser
        console.log('/users/' + user.uid + '/pops/' + itemUid)

        // delete from database
        /* firebase.database().ref('/users/' + user.uid + '/pops/' + this.selectedBrand + '/' + itemUid).set(null) */
        UserService.deleteItemDB(firebase, this.selectedBrand, itemUid)

        // remove from view
        this.$delete(this.items, itemUid)
      }
    }
}
</script>
