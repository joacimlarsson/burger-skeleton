<template>

<!-- STARTINGPAGE -->

<div v-if="category===0">

  <Startingpage ref="startingpage" v-on:randburg="randomBurger(ingredients)" v-on:bytsprak="switchLang()" v-on:gavidare="startOrdering()" :category="category" :ui-labels="uiLabels" :lang="lang">

  </Startingpage>

</div>

<!-- ORDERINGPAGE -->

<div v-else>

<div id="ordering" class=container>
  <img class="example-panel" src="@/assets/kitchen2.jpeg">

  <div id="heady">
    <button class="buttons" id="switchlangbutton" v-on:click="switchLang()">
      {{ uiLabels.language }}
    </button>

    <div>
      <button id="cancelbutton" class="buttons" v-on:click="cancelOrdering()">{{uiLabels.cancelOrder}}</button>
    </div>
  </div>

  <div id="huvudmeny">

    <button id="btn1" :class="['btn', {'active': category===1}]" v-on:click="redirect(1)">{{uiLabels.burger}}</button>
    <button id="btn2" :class="['btn', {'active': category===2}]" v-on:click="redirect(2)">{{uiLabels.bread}}</button>
    <button id="btn3" :class="['btn', {'active': category===3}]" v-on:click="redirect(3)">{{uiLabels.topping}}</button>
    <button id="btn4" :class="['btn', {'active': category===4}]" v-on:click="redirect(4)">{{uiLabels.sauce}}</button>
    <button id="btn5" :class="['sides-btn','btn', {'active': category===5}]" v-on:click="redirect(5)">{{uiLabels.sideorders}}</button>
    <button id="btn6" :class="['sides-btn','btn', {'active': category===6}]" v-on:click="redirect(6)">{{uiLabels.drinks}}</button>
    <button id="btn7" :class="['btn', {'active': category===7}]" v-on:click="redirect(7)">{{uiLabels.checkout}}</button>

  </div>

  <div class="limittext" v-if="category != 5 && category != 6 && category != 7">
    {{uiLabels.choose}} max {{maxIngredients()}} </div>
  <div class="limittext" v-else-if="category == 5 || category == 6">
    {{uiLabels.noMaxLimit}} </div>


  <div class="orderSummary">
    <div id="order-table">
      <h2>{{ uiLabels.currentMenu }}</h2>
      <table style="width:100%">
        <tr v-for="item in countAllChosenIngredients" :key="countAllChosenIngredients.indexOf(item)">
          <td> <button class="plusMinus" id="removeButton" v-on:click="removeFromOrder(item.id)"> - </button></td>
          <td>
            {{item.count}}
          </td>
          <td> <button class="plusMinus" id="addButton" v-on:click="IsOkToAdd(item.id)"> + </button></td>
          <td>{{getItemById(item.id)["ingredient_"+lang]}}</td>
          <td id="price">{{getItemById(item.id).selling_price * item.count}}:-</td>
        </tr>
      </table>
    </div>
  </div>

  <div id="price-summary">

    <table style="width:100%">
      <td>{{uiLabels.total}}</td>
      <td>{{ price }} :-</td>
    </table>
    <div v-if="category!=7">
      <button class="buttons" id="checkoutbutton" v-on:click="redirect(7)">{{ uiLabels.checkout }}</button>
    </div>
    <div v-else-if="category==7">
      <button class="buttons" id="pobutton" v-on:click="placeOrder()">{{ uiLabels.placeOrder }}</button>
    </div>

  </div>

  <div class="menuDisplay">

    <div id="ingredient-choice">
      <Ingredient ref="ingredient" v-for="item in ingredients" v-on:increment="IsOkToAdd(item.ingredient_id)" v-on:decrement="removeFromOrder(item.ingredient_id)" v-show="item.category===category"
        v-bind:itemCount="countNumberOfChosenIngredients(item.ingredient_id)" v-bind:item="item" :lang="lang" :key="item.ingredient_id">

      </Ingredient>
    </div>

    <div id="finalsummary" v-if="this.category == 7">
      <h1 id="review"> {{uiLabels.review}} </h1>
      <h2>
        <table style="width:100%">
          <tr id="review" v-for="item in countAllChosenIngredients" :key="item.ingredient_id">
            <td>
              {{item.count}}
            </td>
            <td>
              x
            </td>
            <td>{{getItemById(item.id)["ingredient_"+lang]}}</td>
            <td id="price">{{getItemById(item.id).selling_price * item.count}}:-</td>
          </tr>
        </table>
      </h2>
      <div v-if="chosenIngredients.length>0">
        <button class="buttons" id="donebutton" v-on:click="addMenu()">{{uiLabels.done}}</button>
        <button id = "random-button2" class="random-button" v-if="category == 7 && randomBurgerBool()==true" v-on:click="newRandomBurger(ingredients)">{{uiLabels.goToRandomMenu2}}</button>
      </div>
      <br>
      <div>
        <table id="order-summary" style="width:100%">
          <tr id="tablerow" v-for="(menu,key) in currentOrder.menus" :key="key">
            <td id="menunr">{{uiLabels.menu}} {{key+1}}</td>
            <div v-for="(item,key2) in menu.ingredients" :key="key2">
              <td id="ing-display">{{item["ingredient_"+lang]}}</td>
            </div>
            <td id="editbtn" ><button v-on:click="changeMenu(menu)">{{uiLabels.edit}}</button> </td>
          </tr>
        </table>
        <br>
        <button id="addanothermenubutton" class="buttons" v-if="category == 7 && chosenIngredients.length===0 && currentOrder.menus.length > 0" v-on:click="redirect(1)">{{uiLabels.newMenu}}</button>
          <button id="random-button3" class="random-button" v-if="category == 7 && chosenIngredients.length===0 && currentOrder.menus.length > 0" v-on:click="newRandomBurger(ingredients)">{{uiLabels.goToRandomMenu3}}</button>
      </div>
      </div>
  </div>


  <div id="footer-buttons">
    <button class="previous-button" v-if="category!==1" v-on:click="previousCategory"><span>{{ uiLabels.previous }}</span></button>
    <button class="next-button" v-if="category!==7" v-on:click="nextCategory"><span>{{ uiLabels.next }}</span></button>
  </div>

  <div id="footer">
    <img src="@/assets/gluten.png" height="20"> - {{uiLabels.glutenFree}}
    <img src="@/assets/dairy.png" height="20"> - {{uiLabels.dairyFree}}
    <img src="@/assets/vegan.png" height="20"> - {{uiLabels.vegan}}
  </div>

</div>
</div>
</template>


<script>

import Ingredient from '@/components/Ingredient.vue'
import OrderItem from '@/components/OrderItem.vue'
import Startingpage from '@/components/Startingpage.vue'
import sharedVueStuff from '@/components/sharedVueStuff.js'

export default {
  name: 'Ordering',
  components: {
    Ingredient,
    OrderItem,
    Startingpage
  },
  mixins: [sharedVueStuff],
  data: function() {
    return {
      chosenIngredients: [],
      price: 0,
      category: 0,
      orderNumber: "",
      okToAdd: true,
      addedToMenu: false,
      randomBurgerBoolean: false,
      isOrderedbool: false,
      currentOrder: {
        menus: []
      }
    }
  },
  computed: {
    chosenIngredientsSet: function() {
      return [...new Set(this.chosenIngredients)]
    },
    countAllChosenIngredients: function() {
      let chosenIngredientTuples = [];
      for (let i = 0; i < this.chosenIngredientsSet.length; i += 1) {
        chosenIngredientTuples[i] = {};
        chosenIngredientTuples[i].id = this.chosenIngredientsSet[i].ingredient_id;
        chosenIngredientTuples[i].count = this.countNumberOfChosenIngredients(this.chosenIngredientsSet[i].ingredient_id);
      }
      return chosenIngredientTuples;
    }
  },
  created: function() {
    this.$store.state.socket.on('orderNumber', function(data) {
      this.orderNumber = data;
    }.bind(this));
  },
  methods: {
    getItemById(id) {
      for (let i = 0; i < this.ingredients.length; i += 1) {
        if (this.ingredients[i].ingredient_id === id)
          return this.ingredients[i];
      }
      return null
    },
    countNumberOfChosenIngredients: function(id) {
      let counter = 0;
      for (let i = 0; i < this.chosenIngredients.length; i += 1) {
        if (this.chosenIngredients[i].ingredient_id === id) {
          counter += 1;
        }
      }
      return counter;
    },
    addToOrder: function(id) {
      if (this.okToAdd) {
        this.chosenIngredients.push(this.getItemById(id));
        this.price += this.getItemById(id).selling_price;
      }
    },
    changeMenu: function(menu) {
      this.chosenIngredients = [];
      for (let i = 0; i < menu.ingredients.length; i += 1) {
        this.chosenIngredients.push(menu.ingredients[i]);
      }
      this.currentOrder.menus.splice(this.currentOrder.menus.indexOf(menu), 1);
    },
    addMenu: function() {
      this.randomBurgerBoolean = false;
      this.addedToMenu = true;
      this.currentOrder.menus.push({
        ingredients: this.chosenIngredients.splice(0),
        price: this.price
      });
      this.chosenIngredients = [];
      for (var i = 0; i < this.$refs.ingredient.length; i += 1) {
        this.$refs.ingredient[i].resetCounter();
      }
    },
    IsOkToAdd: function(id) {
      var i;
      let chosen = 0;
      this.okToAdd = true;
      let cat = this.getItemById(id).category
      let lim;
      if (cat == 1) {
        lim = 2;
      }
      if (cat == 2) {
        lim = 1;
      }
      if (cat == 3) {
        lim = 4;
      }
      if (cat == 4) {
        lim = 2;
      }
      for (i = 0; i < this.chosenIngredients.length; i += 1) {
        if (this.chosenIngredients[i].category == cat) {
          chosen += 1;
        }
      }
      if (chosen >= lim) {
        this.okToAdd = false
      } else if (this.getItemById(id).stock <= this.countNumberOfChosenIngredients(id)) {
        this.okToAdd = false;
        alert(this.uiLabels.alertNoIngredients)
      } else {
        this.addToOrder(id);
      }
      return this.okToAdd;
    },
    maxIngredients: function() {
      let cat = this.category
      let max = 0;
      if (cat == 1) {
        max = 2;
      }
      if (cat == 2) {
        max = 1;
      }
      if (cat == 3) {
        max = 4;
      }
      if (cat == 4) {
        max = 2;
      }
      return max;
    },
    removeFromOrder: function(id) {
      for (let i = this.chosenIngredients.length - 1; i >= 0; --i) {
        if (this.chosenIngredients[i].ingredient_id === id) {
          this.chosenIngredients.splice(i, 1);
          this.price += -this.getItemById(id).selling_price;
          break;
        }
      }
    },
    placeOrder: function() {
      if(this.addedToMenu == false ){
        alert(this.uiLabels.alertAdd)
}
      else if(this.chosenIngredients.length != 0){
        alert(this.uiLabels.alertUnfinished)
      }
      else if (confirm(this.uiLabels.instructions)) {
        alert(this.uiLabels.alertThankYou)

        this.$store.state.socket.emit('order', this.currentOrder);
        this.currentOrder = {
          menus: []
        };
        this.category = 0;
        this.price = 0;

      }
    },


    nextCategory: function() {
      this.category += 1;
      var btns = document.getElementsByClassName("btn");
      for (var i = btns.length - 1; i >= 0; i--)
        if (btns[i].className === "btn active") {
          btns[i].className = "btn";
          btns[i + 1].className += " active";
        }
    },
    previousCategory: function() {
      this.category -= 1;
      var btns = document.getElementsByClassName("btn");
      for (var i = 1; i < btns.length; i++)
        if (btns[i].className === "btn active") {
          btns[i].className = "btn";
          btns[i - 1].className += " active";
        }
    },
    randomBurger: function(ingredients) {

      for (let i = 0; i < this.chosenIngredients.length; i += 1) {
        this.price -= this.chosenIngredients[i].selling_price;
      }
      this.chosenIngredients = [];
      this.randomBurgerBoolean = true;
      let burgmax = 9;
      let toppingmax = 34;
      let saucemax = 48;
      let breadmax = 52;
      let sidesmax = 55;
      let drinkmax = 60;
      for (let i = 0; i < 2; i++) {
        let randburg = Math.floor(Math.random() * (burgmax));
        this.IsOkToAdd(ingredients[randburg].ingredient_id)
      }
      let randbread = Math.floor(Math.random() * (breadmax - saucemax)) + saucemax;
      this.IsOkToAdd(ingredients[randbread].ingredient_id)
      for (let i = 0; i < 4; i++) {
        let randtopping = Math.floor(Math.random() * (toppingmax - burgmax)) + burgmax;
        this.IsOkToAdd(ingredients[randtopping].ingredient_id)
      }
      for (let i = 0; i < 2; i++) {
        let randsauce = Math.floor(Math.random() * (saucemax - toppingmax)) + toppingmax;
        this.IsOkToAdd(ingredients[randsauce].ingredient_id)
      }
      let randsides = Math.floor(Math.random() * (sidesmax - breadmax)) + breadmax;
      this.addToOrder(ingredients[randsides].ingredient_id)
      let randdrink = Math.floor(Math.random() * (drinkmax - sidesmax)) + sidesmax;
      this.addToOrder(ingredients[randdrink].ingredient_id)
      this.category = 7;
    },
    newRandomBurger: function(ingredients) {
      for (let i = 0; i < this.chosenIngredients.length; i += 1) {
        this.price -= this.chosenIngredients[i].selling_price;
      }
      this.chosenIngredients = [];
      this.randomBurger(ingredients);
    },
    randomBurgerBool: function() {
      if (this.randomBurgerBoolean == true) {
        return true
      } else if (this.randomBurgerBoolean == false) {
        return false
      }
    },
    redirect: function(num) {

      this.category = num;
    },
    startOrdering: function() {
      this.category = 1;
    },
    cancelOrdering: function() {
      if (confirm(this.uiLabels.cancelins)){
      this.chosenIngredients = [];

      this.currentOrder = {
        menus: []
      };
      if (this.randomBurgerBool== true ){
        this.randomBurgerBool= false;
      }
      this.category = 0;
      this.price = 0;
    }
    }
  }
}
</script>
<style scoped>
@import "https://fonts.googleapis.com/css?family=Quicksand&display=swap";


.container {
  font-family: 'Quicksand', sans-serif;
  display: grid;
  grid-template-areas:
    "header header"
    "nav nav"
    "chooseMax side"
    "content side"
    "buttons empty"
    "footer footer";
  grid-template-columns: 1fr 300px;
  grid-template-rows: 25px auto auto 1fr 5em;
  grid-gap: 1em;
  height: 100vh;
}
.limittext {
  grid-area: chooseMax;
  font-size: 1.4em;
  font-weight: bold;
}
#footer {
  grid-area: footer;
}
#menunr {
  font-weight: bold;
  vertical-align: top;
}
#order-summary {
  border-collapse: collapse;
}
#tablerow {
  border-bottom: solid 0.08em black;
}
#editbtn {
  text-align: right;
}
#price {
  text-align: right;
}
#footer-buttons {
  grid-area: buttons;
  position: relative;
}
.plusMinus {
  background-color: pink;
  border: none;
  color: red;
  font-size: 1.3em;
}


#addButton {
  color: green;
}
/*PREVIOUS BUTTON*/
.previous-button {
  border: solid black;
  text-align: center;
  font-size: 1.3em;
  padding: 0.4em;
  width: 7em;
  transition: all 0.5s;
  cursor: pointer;
  position: absolute;
  left: 0;
}
.previous-button span {
  cursor: pointer;
  display: inline-block;
  position: relative;
  transition: 0.5s;
}
.previous-button span:after {
  content: '\00ab';
  position: absolute;
  opacity: 0;
  top: 0;
  left: 0;
  transition: 0.5s;
}
.previous-button:hover span {
  padding-left: 0.9em;
}
.previous-button:hover span:after {
  opacity: 1;
  left: 0;
}
/*NEXT BUTTON*/
.next-button {
  border: solid black;
  text-align: center;
  font-size: 1.3em;
  padding: 0.4em;
  width: 7em;
  transition: all 0.5s;
  cursor: pointer;
  position: absolute;
  right: 0;
}
.next-button span {
  cursor: pointer;
  display: inline-block;
  position: relative;
  transition: 0.5s;
}
.next-button span:after {
  content: '\00bb';
  position: absolute;
  opacity: 0;
  top: 0;
  right: 0;
  transition: 0.5s;
}
.next-button:hover span {
  padding-right: 1.2em;
}
.next-button:hover span:after {
  opacity: 1;
  right: 0;
}

.random-button {
  color: #fff !important;
  text-transform: uppercase;
  text-decoration: none;
  background: #ed3330;
  padding: 1em;
  border-radius:  0.5em;
  display: inline-block;
  border: none;
  transition: all 0.3s ease 0s;
}
.random-button:hover {
  cursor: pointer;
  background: darkred;
  letter-spacing: 0.03em;
  -webkit-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  -moz-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  box-shadow: 5px 40px -10px rgba(0, 0, 0, 0.57);
  transition: all 0.15s ease 0s;
}
.menuDisplay {
  grid-area: content;
  overflow-y: scroll;
}

#ingredient-choice {
  display: grid;
  grid-column-gap: 0.5em;
  grid-row-gap: 0.5em;
  grid-template-columns: repeat(auto-fill, 9em);
  text-align: center;
  font-size: 1.1em;
  overflow-y: scroll;
}
#finalsummary {
  width: 100%;
  position: relative;
}
.buttons {
  padding: 0.5em;
  cursor: pointer;
  border-radius: 0.5em;
  text-align: center;
}
#donebutton {
  background-color: darkgreen;
  color: white;
  font-size: 1em;
}
#pobutton {
  background-color: darkgreen;
  color: white;
  font-size: 1.2em;
}
#checkoutbutton {
  background-color: #f1f1f1;
  font-size: 1.2em;
}
#cancelbutton {
  background-color: darkred;
  font-size: 0.9em;
  color: floralwhite;
  position: absolute;
  top: 0.5em;
  right: 0.5em;
}
#switchlangbutton {
  background-color: royalblue;
  color: gold;
  font-size: 0.9em;
  border: 0.15em solid crimson;
}
.example-panel {
  position: fixed;
  left: 0;
  top: 0;
  z-index: -2;
  opacity: 0.2;
}
.ingredient {
  border: 0.2em solid black;
  padding: 0.3em;
  background-color: pink;
  color: black;
}
/*allt nedan gäller menyn "burgare bröd osv"*/

#heady {
  grid-area: header;
}
#huvudmeny {
  grid-area: nav;
  display: grid;
  grid-column-gap: 0.5em;
  grid-row-gap: 0.5em;
  grid-template-columns: repeat(auto-fill, 11.2em);
  justify-content: center;
}
#price-summary {
  z-index: 1;
  width: inherit;
  position: fixed;
  bottom: 0.5em;
  right: 0.5em;
  padding: 1em;
  background-color: pink;
  border: 0.2em dashed black;
}
.btn {
  background-color: #f1f1f1;
  cursor: pointer;
  font-size: 1.3em;
  border-radius: 0.1em;
  border: 0.12em solid;
  margin: 0.4em;
  padding: 1em;
  width: 9em;
  position: relative;
}
.sides-btn {
  background-color: #d9d9d9;
}
/* Style the active class, and buttons on mouse-over */
.active,
.btn:hover {
  background-color: pink;
  /* denna hade vi: #bfbfbf*/
}

.orderSummary {
  grid-area: side;
  border: 0.2em solid black;
  background-color: pink;
  padding: 1em;
  overflow-y: scroll;
}
#order-table {
  width: 100%;
  position: relative;
}
#random-button2 {
  top: 0;
  right: 0;
  position: absolute;
}
#random-button3 {
  bottom: 0;
  left: 0;
  position: relative;
}
#addanothermenubutton {
  bottom: 0;
  left: 0;
  position: relative;
  color: #fff !important;
  text-transform: uppercase;
  text-decoration: none;
  background: green;
  padding: 1em;
  border-radius:  0.5em;
  display: inline-block;
  transition: all 0.3s ease 0s;
}
#addanothermenubutton:hover{
  cursor: pointer;
  background: darkgreen;
  letter-spacing: 0.03em;
  -webkit-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  -moz-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
  box-shadow: 5px 40px -10px rgba(0, 0, 0, 0.57);
  transition: all 0.15s ease 0s;
}

@media (max-width: 600px) {
  .container {
    display: grid;
    grid-template-areas:
      "header header header"
      "nav nav nav"
      "chooseMax chooseMax side"
      "content content side"
      "buttons buttons buttons"
      "footer footer footer";
    grid-template-columns: 1fr 150px;
    align-content: start;
    grid-template-rows: auto auto auto auto auto;
    grid-gap: 1em;
    height: 85vh;
  }
  #ingredient-choice {
    grid-column-start: 1;
    grid-column-end: 5;
    display: grid;
    text-align: center;
    font-size: 1.15em;
    overflow-y: scroll;
  }
  .next-button {
    border: solid black;
    text-align: center;
    font-size: 1.1em;
    padding: 0.3em;
    width: 6em;
    grid-gap: 2em;
    transition: all 0.5s;
    cursor: pointer;

    position: absolute;

    right: 0;
    grid-area: content content side;
    display: inline-block;
  }
  .previous-button {
    font-size: 1.1em;
    padding: 0.3em;
    width: 6em;
    left: 0;
    display: inline-block;
    position: relative;
  }
  #buttons {
    position: relative;
  }
  #price-summary {
    z-index: 1;
    width: inherit;
    position: fixed;
    bottom: 0.25em;
    right: 0.25em;
    padding: 0em;
    background-color: pink;
    border: 0.2em dashed black;
  }
  #huvudmeny {
    display: flex;

    flex-wrap: nowrap;
    justify-content: flex-start;
    height: 6em;
    width: 100%;
    overflow-x: scroll;
}
.btn{
height: 1em;
display: inline-block;
}

.btn{
height: 4em;

}


  #random-button2{
    font-size: 0.6em;
    top: 0;
    left: 0;
    position: relative;
    grid-area:buttons;
  }

  #review {
    font-size: 1em;
  }

  .ingredient {
    font-size: 1em;
    grid-area:buttons buttons;
    height:7em;

  }


}
</style>
